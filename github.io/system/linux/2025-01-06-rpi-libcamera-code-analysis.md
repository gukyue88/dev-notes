---
layout: single
title: "[linux] libcamera 라즈베리파이 코드 분석"
categories: system/linux
published: false
---

#### 주요 코드

- repository : [raspberrypi libcamera](https://github.com/raspberrypi/libcamera)
- src/libcamera/pipeline/rpi : 라즈베리파이용 파이프라인 핸들러 정의 폴더
- src/ipa/rpi : 라즈베리파이용 ipa 알고리즘 관련 코드

### 초기화 시퀀스 다이어그램

#### 초기화 및 객체간 구조

- class RPiCamApp : `rpicam-apps/rpicam_app.hpp`
- class RPiCamStillApp : public RPiCamApp : `rpicam-apps/rpicam_still.cpp`
- class CameraManager : `libcamera/include/libcamera/camera_manager.h`
- class CamearaManager::Private : `libcamera/include/libcamera/internal/camera_manager.h`

`rpicam-apps/rpicam_still.cpp`

```cpp
class RPiCamStillApp : public RPiCamApp
{
public:
	RPiCamStillApp() : RPiCamApp(std::make_unique<StillOptions>()) {}

	StillOptions *GetOptions() const { return static_cast<StillOptions *>(RPiCamApp::GetOptions()); }
};

int main(int argc, char *argv[])
{
    RPiCamStillApp app;
    StillOptions *options = app.GetOptions();
    if (options->Parse(argc, argv))
    {
        if (options->Get().verbose >= 2)
            options->Get().Print();

        event_loop(app);
    }

	return 0;
}

static void event_loop(RPiCamStillApp &app)
{
	StillOptions const *options = app.GetOptions();
	// output requested?
	bool output = !options->Get().output.empty() || options->Get().datetime || options->Get().timestamp;
	// "signal" mode is much like "keypress" mode
	bool keypress = options->Get().keypress || options->Get().signal;
	unsigned int still_flags = RPiCamApp::FLAG_STILL_NONE;
	if (options->Get().encoding == "rgb24" || options->Get().encoding == "png")
		still_flags |= RPiCamApp::FLAG_STILL_BGR;
	if (options->Get().encoding == "rgb48")
		still_flags |= RPiCamApp::FLAG_STILL_BGR48;
	else if (options->Get().encoding == "bmp")
		still_flags |= RPiCamApp::FLAG_STILL_RGB;
	if (options->Get().raw)
		still_flags |= RPiCamApp::FLAG_STILL_RAW;

	app.OpenCamera();

	// Monitoring for keypresses and signals.
	signal(SIGUSR1, default_signal_handler);
	signal(SIGUSR2, default_signal_handler);
	pollfd p[1] = { { STDIN_FILENO, POLLIN, 0 } };

	if (options->Get().immediate)
	{
		app.ConfigureStill(still_flags);
		while (keypress)
		{
			int key = get_key_or_signal(options, p);
			if (key == 'x' || key == 'X')
				return;
			else if (key == '\n')
				break;
			std::this_thread::sleep_for(10ms);
		}
	}
	else if (options->Get().zsl)
		app.ConfigureZsl();
	else
		app.ConfigureViewfinder();
	app.StartCamera();
	auto start_time = std::chrono::high_resolution_clock::now();
	auto timelapse_time = start_time;
	int timelapse_frames = 0;
	constexpr int TIMELAPSE_MIN_FRAMES = 6; // at least this many preview frames between captures
	bool keypressed = false;
	enum
	{
		AF_WAIT_NONE,
		AF_WAIT_SCANNING,
		AF_WAIT_FINISHED
	} af_wait_state = AF_WAIT_NONE;
	int af_wait_timeout = 0;

	bool want_capture = options->Get().immediate;
	for (unsigned int count = 0;; count++)
	{
		RPiCamApp::Msg msg = app.Wait();
		if (msg.type == RPiCamApp::MsgType::Timeout)
		{
			LOG_ERROR("ERROR: Device timeout detected, attempting a restart!!!");
			app.StopCamera();
			app.StartCamera();
			continue;
		}
		if (msg.type == RPiCamApp::MsgType::Quit)
			return;
		else if (msg.type != RPiCamApp::MsgType::RequestComplete)
			throw std::runtime_error("unrecognised message!");

		CompletedRequestPtr &completed_request = std::get<CompletedRequestPtr>(msg.payload);
		auto now = std::chrono::high_resolution_clock::now();
		int key = get_key_or_signal(options, p);
		if (key == 'x' || key == 'X')
			return;
		if (key == '\n')
			keypressed = true;

		// In viewfinder mode, run until the timeout or keypress. When that happens,
		// if the "--autofocus-on-capture" option was set, trigger an AF scan and wait
		// for it to complete. Then switch to capture mode if an output was requested.
		if (app.ViewfinderStream() && !want_capture)
		{
			LOG(2, "Viewfinder frame " << count);
			timelapse_frames++;

			bool timed_out = options->Get().timeout && (now - start_time) > options->Get().timeout.value;
			bool timelapse_timed_out = options->Get().timelapse &&
									   (now - timelapse_time) > options->Get().timelapse.value &&
									   timelapse_frames >= TIMELAPSE_MIN_FRAMES;

			if (af_wait_state != AF_WAIT_NONE)
			{
				FrameInfo fi(completed_request);
				bool scanning = fi.af_state == libcamera::controls::AfStateScanning;
				if (scanning || (af_wait_state == AF_WAIT_SCANNING && ++af_wait_timeout >= 16))
					af_wait_state = AF_WAIT_FINISHED;
				else if (af_wait_state == AF_WAIT_FINISHED)
					want_capture = true;
			}
			else if (timed_out || keypressed || timelapse_timed_out)
			{
				// Trigger a still capture, unless we timed out in timelapse or keypress mode
				if ((timed_out && options->Get().timelapse) || (!keypressed && keypress))
					return;

				keypressed = false;
				if (options->Get().af_on_capture)
				{
					libcamera::ControlList cl;
					cl.set(libcamera::controls::AfMode, libcamera::controls::AfModeAuto);
					cl.set(libcamera::controls::AfTrigger, libcamera::controls::AfTriggerStart);
					app.SetControls(cl);
					af_wait_state = AF_WAIT_SCANNING;
					af_wait_timeout = 0;
				}
				else
					want_capture = true;
			}
			if (want_capture)
			{
				if (!output)
					return;
				keypressed = false;
				af_wait_state = AF_WAIT_NONE;
				timelapse_time = std::chrono::high_resolution_clock::now();
				if (!options->Get().zsl)
				{
					app.StopCamera();
					app.Teardown();
					app.ConfigureStill(still_flags);
				}
				if (options->Get().af_on_capture)
				{
					libcamera::ControlList cl;
					cl.set(libcamera::controls::AfMode, libcamera::controls::AfModeAuto);
					cl.set(libcamera::controls::AfTrigger, libcamera::controls::AfTriggerCancel);
					app.SetControls(cl);
				}
				if (!options->Get().zsl)
					app.StartCamera();
			}
			else
				app.ShowPreview(completed_request, app.ViewfinderStream());
		}
		// In still capture mode, save a jpeg. Go back to viewfinder if in timelapse mode,
		// otherwise quit.
		else if (app.StillStream() && want_capture)
		{
			want_capture = false;
			if (!options->Get().zsl)
				app.StopCamera();
			LOG(1, "Still capture image received");
			save_images(app, completed_request);
			if (!options->Get().metadata.empty())
				save_metadata(options, completed_request->metadata);
			timelapse_frames = 0;
			if (!options->Get().immediate &&
				(options->Get().timelapse || options->Get().signal || options->Get().keypress))
			{
				if (!options->Get().zsl)
				{
					app.Teardown();
					app.ConfigureViewfinder();
				}
				if (options->Get().af_on_capture && options->Get().afMode_index == -1)
				{
					libcamera::ControlList cl;
					cl.set(libcamera::controls::AfMode, libcamera::controls::AfModeAuto);
					cl.set(libcamera::controls::AfTrigger, libcamera::controls::AfTriggerCancel);
					app.SetControls(cl);
				}
				if (!options->Get().zsl)
					app.StartCamera();
				af_wait_state = AF_WAIT_NONE;
			}
			else
				return;
		}
	}
}
```

`rpicam-apps/rpicam_app.cpp`

```cpp
void RPiCamApp::OpenCamera()
{
	// Make a preview window.
	preview_ = std::unique_ptr<Preview>(make_preview(RPiCamApp::GetOptions()));
	preview_->SetDoneCallback(std::bind(&RPiCamApp::previewDoneCallback, this, std::placeholders::_1));

	LOG(2, "Opening camera...");

	if (!camera_manager_)
		initCameraManager();

	std::vector<std::shared_ptr<libcamera::Camera>> cameras = GetCameras();
	if (cameras.size() == 0)
		throw std::runtime_error("no cameras available");

	if (options_->Get().camera >= cameras.size())
		throw std::runtime_error("selected camera is not available");

	std::string const &cam_id = cameras[options_->Get().camera]->id();
	camera_ = camera_manager_->get(cam_id);
	if (!camera_)
		throw std::runtime_error("failed to find camera " + cam_id);

	if (camera_->acquire())
		throw std::runtime_error("failed to acquire camera " + cam_id);
	camera_acquired_ = true;

	LOG(2, "Acquired camera " << cam_id);

	if (!options_->Get().post_process_file.empty())
	{
		post_processor_.LoadModules(options_->Get().post_process_libs);
		post_processor_.Read(options_->Get().post_process_file);
	}
	// The queue takes over ownership from the post-processor.
	post_processor_.SetCallback(
		[this](CompletedRequestPtr &r) { this->msg_queue_.Post(Msg(MsgType::RequestComplete, std::move(r))); });

	// We're going to make a list of all the available sensor modes, but we only populate
	// the framerate field if the user has requested a framerate (as this requires us actually
	// to configure the sensor, which is otherwise best avoided).

	std::unique_ptr<CameraConfiguration> config = camera_->generateConfiguration({ libcamera::StreamRole::Raw });
	const libcamera::StreamFormats &formats = config->at(0).formats();

	bool log_env_set = getenv("LIBCAMERA_LOG_LEVELS");
	// Suppress log messages when enumerating camera modes.
	if (!log_env_set)
	{
		libcamera::logSetLevel("RPI", "ERROR");
		libcamera::logSetLevel("Camera", "ERROR");
	}

	for (const auto &pix : formats.pixelformats())
	{
		for (const auto &size : formats.sizes(pix))
		{
			double framerate = 0;
			if (options_->Get().framerate)
			{
				SensorMode sensorMode(size, pix, 0);
				config->at(0).size = size;
				config->at(0).pixelFormat = pix;
				config->sensorConfig = libcamera::SensorConfiguration();
				config->sensorConfig->outputSize = size;
				config->sensorConfig->bitDepth = sensorMode.depth();
				config->validate();
				camera_->configure(config.get());
				auto fd_ctrl = camera_->controls().find(&controls::FrameDurationLimits);
				framerate = 1.0e6 / fd_ctrl->second.min().get<int64_t>();
			}
			sensor_modes_.emplace_back(size, pix, framerate);
		}
	}

	if (!log_env_set)
	{
		libcamera::logSetLevel("RPI", "INFO");
		libcamera::logSetLevel("Camera", "INFO");
	}
}

void RPiCamApp::initCameraManager()
{
	camera_manager_.reset();
	camera_manager_ = std::make_unique<CameraManager>();
	int ret = camera_manager_->start();
	if (ret)
		throw std::runtime_error("camera manager failed to start, code " + std::to_string(-ret));
}
```

`libcamera/src/libcamera/camera_manager.cpp`

```cpp
CameraManager::CameraManager()
	: Extensible(std::make_unique<CameraManager::Private>())
{
	if (self_)
		LOG(Camera, Fatal)
			<< "Multiple CameraManager objects are not allowed";

	self_ = this;
}

int CameraManager::start()
{
	LOG(Camera, Info) << "libcamera " << version_;

	int ret = _d()->start(); // CameraManager::Private 의 start() 호출
	if (ret)
		LOG(Camera, Error) << "Failed to start camera manager: "
				   << strerror(-ret);

	return ret;
}
```

- CameraManager가 생성될 때, 부모 객체인 Extensible에 CameraManager::Private를 생성함.
  - 이렇게 하면 CameraManager::\_d 가 CameraManager::Private을 가리키게 됨

```cpp
CameraManager::Private::Private()
	: initialized_(false)
{
	ipaManager_ = std::make_unique<IPAManager>();
}

class CameraManager::Private : public Extensible::Private, public Thread
{
}

int CameraManager::Private::start()
{
	int status;

	/* Start the thread and wait for initialization to complete. */
	Thread::start();

	{
		MutexLocker locker(mutex_);
		cv_.wait(locker, [&]() LIBCAMERA_TSA_REQUIRES(mutex_) {
			return initialized_;
		});
		status = status_;
	}

	/* If a failure happened during initialization, stop the thread. */
	if (status < 0) {
		exit();
		wait();
		return status;
	}

	return 0;
}

void CameraManager::Private::run()
{
	LOG(Camera, Debug) << "Starting camera manager";

	int ret = init();

	mutex_.lock();
	status_ = ret;
	initialized_ = true;
	mutex_.unlock();
	cv_.notify_one();

	if (ret < 0) {
		cleanup();
		return;
	}

	/* Now start processing events and messages. */
	exec();

	cleanup();
}
```

- CameraManager::Private는 생성시, ipaManager를 생성하여 `ipaManager_`에 넣어둠
- CameraManager::Private는 public Extensible::Private와 public Thread를 상속받음
- CameraManager->start()는 CameraManager::Private->start()를 호출하고, CameraManager::Private은 Thread의 자식 객체로, start()가 호출되면 run()함수가 실행됨
- CameraManager::Private::run() 에서는 CameraManager::Private::init() 함수 및 exec() 함수를 호출함
