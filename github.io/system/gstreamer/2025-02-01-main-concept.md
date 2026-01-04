---
layout: single
title: "[GStreamer] 주요 개념 및 사용법"
categories: system/gstreamer
---

## 주요 개념

![gstreamer overview](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FkE8vi%2Fbtrbkf6ZejW%2FAAAAAAAAAAAAAAAAAAAAAEVqO0ppi6zT1oHKkzzEW0D0kJcFHWePKN_G6semSuE_%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1759244399%26allow_ip%3D%26allow_referer%3D%26signature%3DGTt7sQN1AJa5RXxrE7B2aSwrTKI%253D)

#### 구성요소

- Elements
  - Pipeline을 구성하는 요소
  - Source element, filters(plugins), sink elements가 있음
    - source element : 데이터 생성, src pad만 존재
    - fliters(plugins) : 데이터를 받아 처리후 내보냄
    - sink element : 파이프라인의 끝 지점, sink pad만 존재
  - properties
    - element들은 속성을 지정할 수 있는 properties를 가짐
    - 주로 초기 설정을 지정하는데 사용하나, 동작중 설정 가능한 것들도 있음
- pads
  - element의 입력/출력
  - element 사이를 연결하는 역할
  - source pad를 통해 나가고 sink pad를 통해 받음
- bin
  - elements의 컨테이너(그룹)
  - element의 상태를 한 번에 관리 가능
  - 상태 : NULL(기본상태), READY(준비상태), PAUSED(멈춘상태), PLAYING(재생상태)
- pipeline
  - 최상위 bin
  - application을 위한 버스 제공
  - 자식에 대한 동기화 관리(?)

#### Application과 pipeline 통신

- buffers : pads를 통해 전달되는 데이터
- bus : application은 버스를 통해 pipeline과 통신함
- message : 각 element에서 application으로 보내는 데이터 (예. 비디오 파일 끝)
- event : Elemnets간 또는 application과 Element간 전송 데이터, 비동기방식, 목적: 상황알림 (예. 스트림 끝 도달, 특정위치로 이동 등)
- query : Elements간 또는 application과 Element간 전송 데이터, 동기방식, 목적: 정보 요청 (예. 현재 재생 위치 확인, 미디어의 전체길이 확인 등)

#### 주요 plugins

- filesrc : 파일을 src element로 사용
- filesink : 파일로 저장 (sink element)
- videosrc : 테스트를 위한 video 생성 src element
- audiosrc : 테스트를 위한 audio 생성 src element
- videoconvert : 색공간변환 element (예. RGB -> YUV)
- videorate : 비디오 프레임을 가져와 새 스트림 생성 (사용 예. 다른 framerate로 비디오 생성 가능)
- videoscale : 비디오 프레임 크기 조정
- queue : 데이터 임시 저장, 상류 엘리먼트의 생산속도가 하류 엘리먼트 소비속도보다 빠를 경우, 하류 엘리먼트가 느리게 소비할 수 있도록 도와줌
- queue2 : queue의 개선된 버전, (queue보다 queue2를 사용하도록 권장됨)
- multiqueue : 여러 개의 입력 pad와 여러 개의 출력 pad를 가진 queue element
- tee : 하나의 데이터 스트림을 여러 개의 독립적인 출력 스트림으로 복제
- gst-nvvideocodes, gst-nvstreammux, gst-nvinfer, gst-nvtracker, gst-nvosd, gst-tilre, gst-nvvidconv는 nvidia로 가속화된 플러그인

#### 데이터 흐름

- upstream element : 앞 쪽에 위치한 element를 가리킴
- downstream element : 뒷 쪽에 위치한 element를 가리킴
- upstream element는 준비된 데이터를 downstream element에 push함
- 만약 downstream element가 준비가 안된 경우, upstream에게 잠시 멈춰달라고 요청하는 형태임 (백프레셔)
- 이 신호는 파이프 라인을 거슬러 올라가 결국 가장 느린 엘리먼트의 속도에 맞춰 전체 파이니라인의 생산속도를 늦추게 됨
- queue를 사용하면 이런 백레셔를 중간에서 흡수하여 각 엘리먼트가 독립적으로 작동할수 있도록 도와줌

## 사용방법

```bash
gst-launch-1.0 videotestsrc ! videoconvert ! autovideosink
```

![](https://miro.medium.com/max/255/0*BvyTC-oGbl7vVFf7.png)

#### properties 설정

```bash
gst-launch-1.0 videotestsrc pattern=11 ! videoconvert ! autovideosink
```

![](https://miro.medium.com/max/529/0*_0Dg1UmR4ShYgNm0.png)

![](https://miro.medium.com/max/254/0*m0gNOM5Jwm0UGVIq.png)

#### named elements 사용

![](https://miro.medium.com/max/700/0*moiY7BI2J0CPrgCM.png)

```bash
gst-launch-1.0 videotestsrc ! videoconvert ! tee name=t ! queue ! autovideosink t. ! queue ! autovideosink
```

- queue : 데이터를 쌓아두고 필요할 때 다음 엘리먼트로 보내는 저장소 역할
- tee : element를 두 개로 분기해주는 역할을 함
  - 출력(src)에서 가져가는 속도가 다르면 느린쪽에 맞추게 됨
  - queue를 이용해 가져가서 한쪽이 느려서 tee가 느리게 되는 걸 방지함
- `tee name=t` : tee 라는 element에 t라는 이름을 부여함
- `t.` : 부여한 이름을 사용 (변수(나중에 다시 사용하기 위해 이름을 지정해놓음) 의역할을 하는듯)

![](https://miro.medium.com/max/529/0*_0Dg1UmR4ShYgNm0.png)
