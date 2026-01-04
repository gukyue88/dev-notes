---
layout: single
title: "[GStreamer] 입문용 코드 분석"
categories: system/gstreamer
---

```cpp
// GStreamer API 전체를 포함하는 헤더 파일
#include <gst/gst.h>

int main(int argc, char* argv[]) {
  // GStreamer 라이브러리 초기화
  // arg를 통한 커맨드라인 옵션 가능, 예. --gst-debug, --help 등
  gst_init(&argc, &argv);

  // 문자열 형태의 파이프라인 구문을 파싱해 실제 GstElement 트리를 만들고 pipeline 객체를 반환함
  GError* error = NULL;
  pipeline = gst_parse_launch(
      "videotestsrc pattern=ball ! videoconvert ! autovideosink",
      &error
  );
  if (!pipeline) {
    g_printerr("파이프라인 생성 실패 : %s\n", error->message);
    return -1;
  }

  // 파이프라인 실행
  // gst_element_set_state는 파이프라인 혹은 개별 요소의 상태를 전환함
  // GST_STATE_PLAYING은 데이터 흐름을 시작하도록 명령하는 상태이고, 내부적으로 READY -> PAUSED -> PLAYING 순서로 전이됨
  gst_element_set_state(pipeline, GST_STATE_PLAYING);

  // 파이프라인에 연결된 GstBus 객체를 얻음
  GstBus* bus = gst_element_get_bus(pipeline);

  // gst_bus_timed_pop_filtered : 버스에서 일정 시간 혹은 무한정 시간 동안 기다렸다가 특정 종류의 메시지를 꺼내오는 함수
  GstMessage *msg = gst_bus_timed_pop_filtered(
    bus,
    GST_CLOCK_TIME_NONE, // 지정한 메시지 타입이 올 때까지 무한정 대기
    // GST_MESSAGE_ERROR : 에러, GST_MESSAGE_EOS : End of Stream
    static_cast<GstMessageType>(GST_MESSAGE_ERROR | GST_MESSAGE_EOS)
  );

  // msg 출력
  if (msg) {
    GError* err;
    gchar* debug_info;
    if (GST_MESSAGE_TYPE(msg) == GST_MESSAGE_ERROR) {
      gst_message_parse_error(msg, &err, &debug_info);
      g_printerr("Error: %s\n", err->message);
      g_printerr("Debug: %s\n", debug_info ? debug_info : "none");
      g_clear_error(%err);
      g_free(debug_info);
    }
    gst_message_unref(msg);
  }

  // 자원 정리
  gst_element_set_state(pipeline, GST_STATE_NULL);
  gst_object_unref(bus);
  gst_object_unref(pipeline);

  return 0;
}
```
