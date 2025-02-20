# WebRTC

You can have real-time audio/video communication with a remote peer. You can send and receive multimedia sources and generic data with a remote peer. The multimedia sources include audio/video streams from a microphone, camera, or media file. The generic data includes string or byte data.

The main features of the WebRTC API include the following:

- Managing media sources

  You can [manage media sources](#media_source) which can be transferred to a remote peer. You can add/remove/mute/pause media sources.

- Controlling data channels

  You can [create/destroy data channels and send/receive generic data](#data_channel) to a remote peer.

- Manipulating state and establishing connection

  You can [use functions for session description and ICE candidates](#establish_connection) to establish network connection with a remote peer properly.

- Rendering audio/video data

  You can [decide how to render audio/video streaming data](#media_render).

## Prerequisites

To use the functions and data types of the WebRTC API (in [mobile](../../api/mobile/latest/group__CAPI__MEDIA__WEBRTC__MODULE.html), [wearable](../../api/wearable/latest/group__CAPI__MEDIA__WEBRTC__MODULE.html) and [IoT headed](../../api/iot-headed/latest/group__CAPI__MEDIA__WEBRTC__MODULE.html) applications), include the `<webrtc.h>` header file in your application:

```c
#include <webrtc.h>
```

<a name="media_source"></a>
## Manage media sources

You can add media sources to a webrtc handle. Once you get source id of the media source, you can manage various functions of the media source with the source id:

1. To add a media source to the webrtc handle, use `webrtc_add_media_source()` before calling `webrtc_start()`:

    ```c
    int ret;
    webrtc_h webrtc;
    unsigned int a_src_id;
    unsigned int v_src_id;

    ret = webrtc_create(&webrtc);
    ret = webrtc_add_media_source(webrtc, WEBRTC_MEDIA_SOURCE_TYPE_AUDIOTEST, &a_src_id);
    ret = webrtc_add_media_source(webrtc, WEBRTC_MEDIA_SOURCE_TYPE_VIDEOTEST, &v_src_id);
    ...
    ret = webrtc_start(webrtc);
    ```

2. To set or get the direction of the media source, use `webrtc_media_source_set_transceiver_direction()` or `webrtc_media_source_get_transceiver_direction()`:
   
    Default transceiver direction of a media source is `WEBRTC_TRANSCEIVER_DIRECTION_SENDRECV`.

    In the following example code, it tries to change the direction to `WEBRTC_TRANSCEIVER_DIRECTION_SENDONLY` before it starts:

    ```c
    ret = webrtc_media_source_set_transceiver_direction(webrtc, v_src_id, WEBRTC_MEDIA_TYPE_VIDEO, WEBRTC_TRANSCEIVER_DIRECTION_SENDONLY);
    ...
    ret = webrtc_start(webrtc);
    ```

3. To pause a media source, use the `webrtc_media_source_set_pause()`:

    ```c
    ret = webrtc_media_source_set_pause(webrtc, a_src_id, WEBRTC_MEDIA_TYPE_AUDIO, true);
    ```
    > [!NOTE]
    >
    > Pausing a media source means it does not send the data to a remote peer. Pause or resume of a media source is also possible in `WEBRTC_STATE_PLAYING`.

4. To mute a media source, use the `webrtc_media_source_set_mute()`:

    ```c
    ret = webrtc_media_source_set_mute(webrtc, a_src_id, WEBRTC_MEDIA_TYPE_AUDIO, true);
    ret = webrtc_media_source_set_mute(webrtc, v_src_id, WEBRTC_MEDIA_TYPE_VIDEO, true);
    ```
    > [!NOTE]
    >
    > Muting a media source means it sends black video frames or silent audio frames to a remote peer. Mute or un-mute of a media source is also possible in `WEBRTC_STATE_PLAYING`.
    > Some types of media sources do not support this functionality. For example, `WEBRTC_MEDIA_SOURCE_TYPE_FILE` and `WEBRTC_MEDIA_SOURCE_TYPE_MEDIA_PACKET`.

5. To set or get the video resolution to a media source, use the `webrtc_media_source_set_video_resolution()` or `webrtc_media_source_get_video_resolution()`:

    ```c
    ret = webrtc_media_source_set_video_resolution(webrtc, v_src_id, 640, 480);
    ```
    > [!NOTE]
    >
    > Some types of media sources support dynamic resolution change while streaming. Otherwise `WEBRTC_ERROR_INVALID_OPERATION` error will be returned.

<a name="data_channel"></a>
## Control data channels

You can create a data channel to a webrtc handle. It is also possible to get notified when you have new data channel requested by a remote peer. You can send or receive data to or from these data channels by using the functions below:

1. To create a data channel, use `webrtc_create_data_channel()` before calling `webrtc_start()`:

    ```c
    int ret;
    webrtc_h webrtc;
    webrtc_data_channel_h channel;

    ret = webrtc_create(&webrtc);
    ret = webrtc_create_data_channel(webrtc, "data_channel_label", NULL, &channel);
    ...
    ret = webrtc_start(webrtc);
    ```

2. To get notified when a data channel is created by a remote peer, use `webrtc_set_data_channel_cb()`. The callback function will be invoked when it is created after negotiation:

    ```c
    void _data_channel_cb(webrtc_h webrtc, webrtc_data_channel_h channel, void *user_data)
    {
        /* New data channel is created by remote peer request */
    }

    void webrtc_func(void)
    {
        int ret;
        webrtc_h webrtc;
        webrtc_data_channel_h channel;

        ret = webrtc_create(&webrtc);
        ret = webrtc_set_data_channel_cb(webrtc, _data_channel_cb, user_data);
        ...
        ret = webrtc_start(webrtc);
    }
    ```

3. Once you get a data channel as above, it is recommended to set callbacks to handle events from the data channel:

    ```c
    void _data_channel_open_cb(webrtc_data_channel_h channel, void *user_data)
    {
        /* Data channel is opened */
    }

    void _data_channel_message_cb(webrtc_data_channel_h channel, webrtc_data_channel_type_e type, void *message, void *user_data)
    {
        /* Message arrived */
        switch (type) {
        case WEBRTC_DATA_CHANNEL_TYPE_STRING:
            printf("%s\n", (char *)message);
            break;

        case WEBRTC_DATA_CHANNEL_TYPE_BYTES: {
            webrtc_bytes_data_h *data = message;
            const char *data_ptr;
            unsigned long size;

            webrtc_get_data(data, &data_ptr, &size);
            printf("bytes message[%p, size:%lu]\n", data_ptr, size);
            break;
        }
        }
    }

    void _data_channel_close_cb(webrtc_data_channel_h channel, void *user_data)
    {
        /* Data channel is closed */
    }

    void _data_channel_error_cb(webrtc_data_channel_h channel, webrtc_error_e error, void *user_data)
    {
        /* An error occurs on the data channel */
    }

    void data_channel_set_callbacks(webrtc_data_channel_h channel, void *user_data)
    {
        webrtc_data_channel_set_open_cb(channel, _data_channel_open_cb, user_data);
        webrtc_data_channel_set_message_cb(channel, _data_channel_message_cb, user_data);
        webrtc_data_channel_set_error_cb(channel, _data_channel_error_cb, user_data);
        webrtc_data_channel_set_close_cb(channel, _data_channel_close_cb, user_data);
    }
    ```

4. To send string data to the data channel, use `webrtc_data_channel_send_string()`:

    ```c
    ret = webrtc_data_channel_send_string(channel, "string_to_send");
    ```

5. To send bytes data to the data channel, use `webrtc_data_channel_send_bytes()`:

    ```c
    char buffer[BUFFER_SIZE] = {0, };
    unsigned int data_size;

    /* Some works to fill the buffer with data */

    ret = webrtc_data_channel_send_bytes(channel, buffer, data_size);
    ```

<a name="establish_connection"></a>
## Manipulate state and establish connection

You can change state of the webrtc handle. If you are ready for media sources that need to be sent to a remote peer, you can start the handle. Once you get the state of negotiation, you can utilize functions to create an offer or answer description, to set a local or remote description, and to add ICE candidates from the remote peer. Finally, you can get the playing state of the handle as well as a connection between peers is established.

1. To get the state of negotiation, use `webrtc_start()`:

    ```c
    void _ice_candidate_cb(webrtc_h webrtc, const char *candidate, void *user_data)
    {
        /* Gather candidates or send candidate to the remote peer via signaling server */
    }

    void _state_changed_cb(webrtc_h webrtc, webrtc_state_e previous, webrtc_state_e current, void *user_data)
    {
        /* After webrtc_start(), current state will be changed to WEBRTC_STATE_NEGOTIATING */
    }

    void webrtc_func(void)
    {
        int ret;
        webrtc_h webrtc;
        unsigned int a_src_id;
        unsigned int v_src_id;

        ret = webrtc_create(&webrtc);
        ret = webrtc_add_media_source(webrtc, WEBRTC_MEDIA_SOURCE_TYPE_AUDIOTEST, &a_src_id);
        ret = webrtc_add_media_source(webrtc, WEBRTC_MEDIA_SOURCE_TYPE_VIDEOTEST, &v_src_id);
        ...
        ret = webrtc_set_ice_candidate_cb(webrtc, _ice_candidate_cb, user_data);
        ret = webrtc_set_state_changed_cb(webrtc, _state_changed_cb, user_data);
        ...
        ret = webrtc_start(webrtc);
    }
    ```

2. If the handle is an offerer, to create offer description, use `webrtc_create_offer()` or `webrtc_create_offer_async()`:

    ```c
    int ret;
    webrtc_h webrtc;
    char *offer_desc;
    ...
    ret = webrtc_start(webrtc);
    ret = webrtc_create_offer(webrtc, NULL, &offer_desc);
    ...
    /* After sending this offer description to the remote peer */
    free(offer_desc);
    ```

3. If the handle is an answerer, to create answer description, use `webrtc_create_answer()` or `webrtc_create_answer_async()`:

    ```c
    int ret;
    webrtc_h webrtc;
    char *answer_desc;
    ...
    ret = webrtc_start(webrtc);
    ...
    /* After receiving an offer description from the remote peer */
    ret = webrtc_set_remote_description(webrtc, offer_desc_from_remote);
    ret = webrtc_create_answer(webrtc, NULL, &answer_desc);
    ...
    /* After sending this answer description to the remote peer */
    free(offer_desc);
    ```

4. To gather ICE candidates, use `webrtc_set_local_description()`:

    ```c
    void _ice_candidate_cb(webrtc_h webrtc, const char *candidate, void *user_data)
    {
        /* Gather candidates or send candidate to the remote peer via signaling server */
    }

    void _ice_gathering_state_change_cb(webrtc_h webrtc, webrtc_ice_gathering_state_e state, void *user_data)
    {
        /* Until the state is changed to WEBRTC_ICE_GATHERING_STATE_COMPLETE, _ice_candidate_cb() will be called */
    }

    void webrtc_func(void)
    {
        int ret;
        webrtc_h webrtc;
        char *local_desc;
        ...
        ret = webrtc_set_ice_candidate_cb(webrtc, _ice_candidate_cb, user_data);
        ret = webrtc_set_state_changed_cb(webrtc, _state_changed_cb, user_data);
        ret = webrtc_set_ice_gathering_state_change_cb(webrtc, _ice_gathering_state_change_cb, user_data);
        ...
        ret = webrtc_start(webrtc);
        ...
        /* local_desc can be obtained from calling webrtc_create_offer() or webrtc_create_answer() */
        ret = webrtc_set_local_description(webrtc, local_desc);
    }
    ```

5. To finish the negotiation, use `webrtc_add_ice_candidate()`, `webrtc_set_local_description()` or `webrtc_set_remote_description()`:

    ```c
    /* After receiving all of ICE candidates from the remote peer */
    ret = webrtc_add_ice_candidate(webrtc, candidate);
    ...
    /* In case of an answerer */
    ret = webrtc_set_local_description(webrtc, answer_desc);
    ... or ...
    /* In case of an offerer */
    ret = webrtc_set_remote_description(webrtc, answer_desc);
    ...
    /* If the connection is established successfully, you'll get notified of WEBRTC_STATE_PLAYING by _state_changed_cb() */
    ```
6. To get notified of various negotiation states, set callbacks by using `webrtc_set_peer_connection_state_change_cb()`, `webrtc_set_signaling_state_change_cb()`, `webrtc_set_ice_gathering_state_change_cb()` and `webrtc_set_ice_connection_state_change_cb()`:

    ```c
    void _peer_connection_state_change_cb(webrtc_h webrtc, webrtc_peer_connection_state_e state, void *user_data)
    {
        /* After setting both description and ICE candidates from the remote peer, it'll be changed from WEBRTC_PEER_CONNECTION_STATE_CONNECTING to WEBRTC_PEER_CONNECTION_STATE_CONNECTED. */
    }

    void _signaling_state_change_cb(webrtc_h webrtc, webrtc_signaling_state_e state, void *user_data)
    {
        /* After setting local or remote description it'll be changed to WEBRTC_SIGNALING_STATE_HAVE_LOCAL_OFFER or WEBRTC_SIGNALING_STATE_HAVE_REMOTE_OFFER respectively. Both descriptions are set, it'll be changed to WEBRTC_SIGNALING_STATE_STABLE. */
    }

    void _ice_gathering_state_change_cb(webrtc_h webrtc, webrtc_ice_gathering_state_e state, void *user_data)
    {
        /* When setting local description, it'll be changed to WEBRTC_ICE_GATHERING_STATE_GATHERING. If the gathering work is done, it'll be changed to WEBRTC_ICE_GATHERING_STATE_COMPLETE. */
    }

    void _ice_connection_state_change_cb(webrtc_h webrtc, webrtc_ice_connection_state_e state, void *user_data)
    {
        /* After setting both description and ICE candidates from the remote peer, it'll be changed from WEBRTC_ICE_CONNECTION_STATE_CHECKING to WEBRTC_ICE_CONNECTION_STATE_COMPLETED. */
    }

    void webrtc_func(void)
    {
        int ret;
        webrtc_h webrtc;
        ...
        ret = webrtc_set_peer_connection_state_change_cb(webrtc, _peer_connection_state_change_cb, user_data);
        ret = webrtc_set_signaling_state_change_cb(webrtc, _signaling_state_change_cb, user_data);
        ret = webrtc_set_ice_gathering_state_change_cb(webrtc, _ice_gathering_state_change_cb, user_data);
        ret = webrtc_set_ice_connection_state_change_cb(webrtc, _ice_connection_state_change_cb, user_data);
        ...
        ret = webrtc_start(webrtc);
        /* Do negotiation */
    }
    ```

<a name="media_render"></a>
## Render audio/video data

You can decide how to handle audio/video streaming data received from a remote peer by using functions provided in this API set. You can also render sending audio/video data on the local target device.

1. To get notified of creation of audio or video track from a remote peer, use `webrtc_set_track_added_cb()`:

    ```c
    void _track_added_cb(webrtc_h webrtc, webrtc_media_type_e type, unsigned int track_id, void *user_data)
    {
        int ret;
        some_app_data_s *data = (some_app_data_s *)user_data;

        if (type == WEBRTC_MEDIA_TYPE_AUDIO) {
            ret = sound_manager_create_stream_information(SOUND_STREAM_TYPE_MEDIA, NULL, NULL, &data->stream_info);
            ret = webrtc_set_sound_stream_info(webrtc, track_id, data->stream_info);

        } else if (type == WEBRTC_MEDIA_TYPE_VIDEO) {
            /* To render video data to video overlay, use window id */
            ret = webrtc_set_display(webrtc, id, WEBRTC_DISPLAY_TYPE_OVERLAY, data->win_id);
            ... or ...
            /* To render video data to EVAS image object */
            ret = webrtc_set_display(webrtc, id, WEBRTC_DISPLAY_TYPE_EVAS, data->evas_image_object);
        }
    }

    void webrtc_func(void)
    {
        int ret;
        webrtc_h webrtc;
        ...
        ret = webrtc_set_track_added_cb(webrtc, _track_added_cb, data);
        ...
        ret = webrtc_start(webrtc);
        ...
        /* After finishing negotiation, _track_added_cb() could be called if receiving audio/video data from the remote peer exists */
    }
    ```
    > [!IMPORTANT]
    >
    > `webrtc_set_sound_stream_info()` or `webrtc_set_display()` must be called inside of the callback set by `webrtc_set_track_added_cb()` if you want to output the audio or video track from the remote peer to the local target device's audio device or video display.

2. To get media packet handle which packs the audio or video data from a remote peer, use `webrtc_set_encoded_audio_frame_cb()` or `webrtc_set_encoded_video_frame_cb()`:

    ```c
    void _encoded_frame_cb(webrtc_h webrtc, webrtc_media_type_e type, unsigned int track_id, media_packet_h packet, void *user_data)
    {
        some_app_data_s *data = (some_app_data_s *)user_data;

        if (type == WEBRTC_MEDIA_TYPE_AUDIO) {
            /* Use media packet - copy it's data or pass it to another API */
        } else if (type == WEBRTC_MEDIA_TYPE_VIDEO) {
            /* Use media packet - copy it's data or pass it to another API */
        }

        /* media packet should be unreferenced after use */
	    media_packet_unref(packet);
    }

    void webrtc_func(void)
    {
        int ret;
        webrtc_h webrtc;
        ...
        ret = webrtc_set_encoded_audio_frame_cb(webrtc, _encoded_frame_cb, data);
        ret = webrtc_set_encoded_video_frame_cb(webrtc, _encoded_frame_cb, data);
        ...
        ret = webrtc_start(webrtc);
        ...
        /* After finishing negotiation, _encoded_frame_cb() could be called if receiving audio/video data from the remote peer exists */
    }
    ```

3. To render sending audio/video data on the local target device, use `webrtc_media_source_set_audio_loopback()` or `webrtc_media_source_set_video_loopback()`:

    ```c
    void webrtc_func(some_app_data_s *data)
    {
        int ret;
        webrtc_h webrtc;
        unsigned int a_src_id, v_src_id;
        unsigned int a_track_id, v_track_id;
        ...
        ret = webrtc_add_media_source(webrtc, WEBRTC_MEDIA_SOURCE_TYPE_MIC, &a_src_id);
        ret = webrtc_add_media_source(webrtc, WEBRTC_MEDIA_SOURCE_TYPE_CAMERA, &v_src_id);

        /* To set audio source data loopback and to output it to audio device on the local target device */
        ret = webrtc_media_source_set_audio_loopback(webrtc, a_src_id, data->stream_info, &track_id);
        /* To set video source data loopback and to render it to video overlay, use window id */
        ret = webrtc_media_source_set_video_loopback(webrtc, v_src_id, WEBRTC_DISPLAY_TYPE_OVERLAY, data->win_id, &track_id);
        ... or ...
        /* To set video source data loopback and to render it to EVAS image object */
        ret = webrtc_media_source_set_video_loopback(webrtc, v_src_id, WEBRTC_DISPLAY_TYPE_EVAS, data->evas_image_object, &track_id);
        ...
        ret = webrtc_start(webrtc);
        /* Do negotiation */
    }
    ```

## Related information
- Dependencies
  - Tizen 6.5 and Higher for Mobile
  - Tizen 6.5 and Higher for Wearable
  - Tizen 6.5 and Higher for IoT Headed
