---
created: 2024-06-22T05:17
updated: 2024-07-07T22:09
---

First time doing anything RTP related, fun challenge.

## Setup

- `Analyze > Decode as > Add+ > RTP` It should display the RTP type
- `Edit > Preferences > Protocols > H.264` Set type to 96 to reflect the type in decoded RTP
- Install https://github.com/volvet/h264extractor
- `Menu > Tools > Extract h264 stream from RTP`
- `ffmpeg -f h264 -i video_20240622-051346.264 -c copy out.mp4`

The capture has two RTP streams.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1719047926/2024/06/e5e5694546af86523c9c42af9021c9f4.png)

![vlcsnap-2024-06-22-05h18m28s311.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1719048210/2024/06/e2f6aac77a5fe580b1e9f48e4f395802.png)

```
FLAG{Th4nk_y0u_f0r_W4tching}
```
