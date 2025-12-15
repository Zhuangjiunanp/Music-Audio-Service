# Music Audio Service

A **native Android Media Playback Service** for **audio and video** (local + network),
designed to **gradually replace Download Manager–based playback**, while **WebView playback still coexists**.

Official Website: https://zhuangjiunanp.github.io

---

## Scope (Important)

- ✅ Local media playback
- ✅ Network media playback (HTTP / HTTPS)
- ❌ NOT WebView-based playback
- ❌ NOT a browser player
- ⚠ WebView playback still exists, but **not replaced**

---

## Required Permissions

```xml
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_MEDIA_PLAYBACK"/>
```

---

## Service Architecture

```text
UI (Activity / Fragment)
        |
        v
MusicAudioService (Foreground)
        |
        v
ExoPlayer
        |
        v
Local File / Network URL
```

---

## MusicAudioService (FULL Kotlin Code)

```kotlin
package com.example.player

import android.app.*
import android.content.*
import android.os.*
import androidx.core.app.NotificationCompat
import androidx.media3.common.MediaItem
import androidx.media3.exoplayer.ExoPlayer

class MusicAudioService : Service() {

    companion object {
        const val ACTION_SET_SOURCE = "action_set_source"
        const val ACTION_PLAY = "action_play"
        const val ACTION_PAUSE = "action_pause"
        const val ACTION_STOP = "action_stop"
        const val EXTRA_URL = "extra_url"
        const val CHANNEL_ID = "media_playback"
    }

    private lateinit var player: ExoPlayer

    override fun onCreate() {
        super.onCreate()
        createNotificationChannel()
        player = ExoPlayer.Builder(this).build()
    }

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        when (intent?.action) {
            ACTION_SET_SOURCE -> {
                val url = intent.getStringExtra(EXTRA_URL) ?: return START_NOT_STICKY
                val item = MediaItem.fromUri(url)
                player.setMediaItem(item)
                player.prepare()
            }
            ACTION_PLAY -> {
                startForeground(1, buildNotification())
                player.play()
            }
            ACTION_PAUSE -> player.pause()
            ACTION_STOP -> {
                player.stop()
                stopForeground(true)
                stopSelf()
            }
        }
        return START_STICKY
    }

    override fun onDestroy() {
        player.release()
        super.onDestroy()
    }

    override fun onBind(intent: Intent?): IBinder? = null

    private fun createNotificationChannel() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel(
                CHANNEL_ID,
                "Media Playback",
                NotificationManager.IMPORTANCE_LOW
            )
            getSystemService(NotificationManager::class.java).createNotificationChannel(channel)
        }
    }

    private fun buildNotification(): Notification {
        return NotificationCompat.Builder(this, CHANNEL_ID)
            .setContentTitle("Playing Media")
            .setContentText("Music Audio Service")
            .setSmallIcon(android.R.drawable.ic_media_play)
            .build()
    }
}
```

---

## Start Playback from Activity (FULL Code)

```kotlin
val setSource = Intent(this, MusicAudioService::class.java).apply {
    action = MusicAudioService.ACTION_SET_SOURCE
    putExtra(
        MusicAudioService.EXTRA_URL,
        "https://example.com/video.mp4"
    )
}
startService(setSource)

val play = Intent(this, MusicAudioService::class.java).apply {
    action = MusicAudioService.ACTION_PLAY
}
startService(play)
```

---

## AndroidManifest.xml Registration

```xml
<service
    android:name=".player.MusicAudioService"
    android:exported="false"
    android:foregroundServiceType="mediaPlayback"/>
```

---

## Why This Replaces Download Manager Playback

| Download Manager | Music Audio Service |
|------------------|---------------------|
| Download first   | Stream or play now  |
| File-only logic  | Media-aware logic   |
| No MediaSession  | Media-ready         |
| Indirect control | Direct playback     |

---

## What Is NOT Replaced

- WebView video playback
- Browser-based streaming
- Embedded website players

---

## License

MIT
