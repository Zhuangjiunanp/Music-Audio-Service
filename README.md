# Music Audio Service

## Overview

Music Audio Service is a native Android background service designed for **local audio and video playback**.  
It is planned to **replace the current Download Manager–based video playback pipeline**.  
This service **does not rely on WebView or web streaming** and works entirely with local media files.

> Current App Language Support: **Chinese (zh-CN only)**  
> Documentation Language: **English**

---

## Goals

- Replace Download Manager video playback logic
- Provide a unified **Audio / Video playback service**
- Support background playback
- Support system-level media controls
- Prepare infrastructure for future multi-language UI

---

## Key Features

- 🎵 Local audio playback (MP3, AAC, FLAC)
- 🎬 Local video playback (MP4, MKV)
- 🔊 MediaSession integration
- ⏯ Notification controls (Play / Pause / Next)
- 🔁 Playlist support
- 🧠 Service-based architecture (no Activity dependency)

---

## Architecture

```text
+---------------------+
|   UI Layer (CN)     |
|  Activity / View   |
+----------+----------+
           |
           v
+---------------------+
| MusicAudioService   |
| (Foreground)       |
+----------+----------+
           |
           v
+---------------------+
| MediaPlayer /      |
| ExoPlayer           |
+---------------------+
```

---

## Service Lifecycle

1. UI sends Intent command
2. Service initializes MediaPlayer / ExoPlayer
3. Service enters foreground mode
4. MediaSession publishes playback state
5. Notification controls playback

---

## Intent Actions

```text
ACTION_PLAY
ACTION_PAUSE
ACTION_STOP
ACTION_SEEK
ACTION_SET_SOURCE
```

---

## Kotlin Service Example

```kotlin
class MusicAudioService : Service() {

    private lateinit var player: ExoPlayer

    override fun onCreate() {
        super.onCreate()
        player = ExoPlayer.Builder(this).build()
    }

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        when (intent?.action) {
            "ACTION_PLAY" -> player.play()
            "ACTION_PAUSE" -> player.pause()
            "ACTION_STOP" -> stopSelf()
        }
        return START_STICKY
    }

    override fun onBind(intent: Intent?): IBinder? = null
}
```

---

## Setting Media Source

```kotlin
val mediaItem = MediaItem.fromUri(fileUri)
player.setMediaItem(mediaItem)
player.prepare()
```

---

## Foreground Notification

```kotlin
startForeground(
    1,
    NotificationCompat.Builder(this, CHANNEL_ID)
        .setContentTitle("Playing Media")
        .setSmallIcon(R.drawable.ic_music)
        .build()
)
```

---

## Why Replace Download Manager?

| Download Manager | Music Audio Service |
|------------------|---------------------|
| File-oriented    | Playback-oriented   |
| No media control | Full media control  |
| No background UX | Foreground service  |
| Not extensible   | Modular & scalable  |

---

## Future Plan

- ⏩ Video playback surface binding
- 🌍 Multi-language UI support
- 📡 Streaming protocol extension (optional)
- 🧩 Plugin-based decoder support

---

## License

MIT License
