Refer to the original discord docs to make friendly names for the opcode numbers

1. First you are informed of the websocket url of a server in the region the voice channel is set to. This will be something in the format `wss://region-idXXXX.discord.media/?v=<version>` where X is a server number/id. At the time of testing I got a 4 digit number and I'm not sure if the number is padded with 0s. I can't check by entering a single digit server url into my browser due to cloudflare answering even if the domain is invalid. 
2. The client establishes a websocket connection using the above information. 
3. An message with opcode 0 is sent to identify the user associated with the connection. Note: A voice connection was made but for some reason the below data contains the word video. 
```json
{
    "op":0,
    "d":{
        "server_id": "1111111111",
        "session_id": "xxxxxxxx... 32 characters total",
        "streams":[
           {
               "type": "video",
               "rid": 100,
               "quality": 100
           }
        ],
        "token": "not your discord token, 16 characters hashlike",
        "user_id": "1111111111",
        "video": true
    }
}
```
The normal heartbeating also occurs
```
{"op":8,"d":{"v":6,"heartbeat_interval":13750}}
```
4. The server replies with stream details with opcode 2. 
```json
{
  "op": 2,
  "d": {
    "streams": [
      {
        "type": "video",
        "ssrc": 61802,
        "rtx_ssrc": 61803,
        "rid": "100",
        "quality": 100,
        "active": false
      }
    ],
    "ssrc": 61801,
    "port": 1234,
    "modes": [
      "aead_aes256_gcm_rtpsize",
      "aead_aes256_gcm",
      "xsalsa20_poly1305_lite_rtpsize",
      "xsalsa20_poly1305_lite",
      "xsalsa20_poly1305_suffix",
      "xsalsa20_poly1305"
    ],
    "ip": "ipv4 ip",
    "experiments": [
        "a/b testing stuff"
    ]
  }
}
```
5. The client then appears to respond with opcode 16 with `{}` in the `d` for data field. 
6. The server responds with opcode 16 which appears to list protocol versions. Data as of November of 2021:
```json
{
  "voice": "0.8.23",
  "rtc_worker": "0.3.14"
}
```
7. The client sends both opcode 12 and opcode 1 which still say video in their fields. 
Opcode 12 data:
```json
{
  "audio_ssrc": 0,
  "video_ssrc": 0,
  "rtx_ssrc": 0,
  "streams": [
    {
      "type": "video",
      "rid": "100",
      "ssrc": 0,
      "active": false,
      "quality": 100,
      "rtx_ssrc": 0,
      "max_bitrate": 2500000,
      "max_framerate": 30,
      "max_resolution": {
        "type": "fixed",
        "width": 1280,
        "height": 720
      }
    }
  ]
}
```
(on the first testing session the opcode 12 was sent twice)
Opcode 1 data:
```json
{
  "protocol": "webrtc",
  "data": "a=extmap-allow-mixed\na=ice-ufrag:6nHO\na=ice-pwd:Z7zvZzj3GlMaxxYwHG3tnd0X\na=ice-options:trickle\na=extmap:1 urn:ietf:params:rtp-hdrext:ssrc-audio-level\na=extmap:2 http://www.webrtc.org/experiments/rtp-hdrext/abs-send-time\na=extmap:3 http://www.ietf.org/id/draft-holmer-rmcat-transport-wide-cc-extensions-01\na=extmap:4 urn:ietf:params:rtp-hdrext:sdes:mid\na=extmap:5 urn:ietf:params:rtp-hdrext:sdes:rtp-stream-id\na=extmap:6 urn:ietf:params:rtp-hdrext:sdes:repaired-rtp-stream-id\na=rtpmap:111 opus/48000/2\na=extmap:14 urn:ietf:params:rtp-hdrext:toffset\na=extmap:13 urn:3gpp:video-orientation\na=extmap:12 http://www.webrtc.org/experiments/rtp-hdrext/playout-delay\na=extmap:11 http://www.webrtc.org/experiments/rtp-hdrext/video-content-type\na=extmap:7 http://www.webrtc.org/experiments/rtp-hdrext/video-timing\na=extmap:8 http://www.webrtc.org/experiments/rtp-hdrext/color-space\na=rtpmap:96 VP8/90000\na=rtpmap:97 rtx/90000",
  "sdp": "a=extmap-allow-mixed\na=ice-ufrag:6nHO\na=ice-pwd:Z7zvZzj3GlMaxxYwHG3tnd0X\na=ice-options:trickle\na=extmap:1 urn:ietf:params:rtp-hdrext:ssrc-audio-level\na=extmap:2 http://www.webrtc.org/experiments/rtp-hdrext/abs-send-time\na=extmap:3 http://www.ietf.org/id/draft-holmer-rmcat-transport-wide-cc-extensions-01\na=extmap:4 urn:ietf:params:rtp-hdrext:sdes:mid\na=extmap:5 urn:ietf:params:rtp-hdrext:sdes:rtp-stream-id\na=extmap:6 urn:ietf:params:rtp-hdrext:sdes:repaired-rtp-stream-id\na=rtpmap:111 opus/48000/2\na=extmap:14 urn:ietf:params:rtp-hdrext:toffset\na=extmap:13 urn:3gpp:video-orientation\na=extmap:12 http://www.webrtc.org/experiments/rtp-hdrext/playout-delay\na=extmap:11 http://www.webrtc.org/experiments/rtp-hdrext/video-content-type\na=extmap:7 http://www.webrtc.org/experiments/rtp-hdrext/video-timing\na=extmap:8 http://www.webrtc.org/experiments/rtp-hdrext/color-space\na=rtpmap:96 VP8/90000\na=rtpmap:97 rtx/90000",
  "codecs": [
    {
      "name": "opus",
      "type": "audio",
      "priority": 1000,
      "payload_type": 111,
      "rtx_payload_type": null
    },
    {
      "name": "H264",
      "type": "video",
      "priority": 1000,
      "payload_type": 102,
      "rtx_payload_type": 121
    },
    {
      "name": "VP8",
      "type": "video",
      "priority": 2000,
      "payload_type": 96,
      "rtx_payload_type": 97
    },
    {
      "name": "VP9",
      "type": "video",
      "priority": 3000,
      "payload_type": 98,
      "rtx_payload_type": 99
    }
  ],
  "rtc_connection_id": "a uuidv4?"
}
```
8. The server then makes an agreement to this using opcode 15 with data
```json
{
    "any": 100
}
```
9. Some fingerprint sha thing is sent
**TODO**
