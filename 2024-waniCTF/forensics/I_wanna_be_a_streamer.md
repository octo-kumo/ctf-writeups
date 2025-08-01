---
ai_date: 2025-04-27 05:23:21
ai_summary: Extracted h264 stream from RTP led to flag discovery, likely involving RTP analysis or media exploitation.
ai_tags:
  - rtp
  - h264
  - media-exploitation
created: 2024-06-22T05:17
points: 169
solves: 144
tags:
  - RTP
  - pcap
updated: 2025-07-14T09:46
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

```flag
FLAG{Th4nk_y0u_f0r_W4tching}
```