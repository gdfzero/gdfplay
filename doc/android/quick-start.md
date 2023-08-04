---
layout: default
title: Quick Start
nav_order: 5
parent: Android
---

# Quick Start
## Sample Codes
### Usage of this class with MediaPlayer:

```java
class VideoPlayActivity extends AppCompatActivity {
    // this ImageView must be placed in layout, where the video will be showed.
    private ImageView displayView;
    // play button should be placed in layout also.
    private Button btnPlay;

    MediaPlayer player = null;
    GDFSRProcessor videoSR = null;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        // media resolution should be one of:
        // 320x180, 426x240, 480x270, 640x360, 854x480
        String mediaPath = "https://test-videos.co.uk/vids/bigbuckbunny/mp4/h264/360/Big_Buck_Bunny_360_10s_10MB.mp4";
        player = new MediaPlayer()
        player.setDataSource(mediaPath);
        player.prepareAsync();
        player.setOnPreparedListener(mp -> {
            videoSR = new GDFSRProcessor(activity, player.getVideoWidth(), player.getVideoHeight());
            videoSR.setDisplayView(displayView);
            player.setSurface(videoSR.getPlayerSurface());
            player.setOnCompletionListener(p -> {
                videoSR.release();
                player.release();
                videoSR = null;
                player = null;
            });
            player.start();
        });
    }
  ...
}
```

### Usage of this class with ExoPlayer:
```java
class VideoPlayActivity extends AppCompatActivity {
    // this ImageView must be placed in layout, where the video will be showed.
    private ImageView displayView;
    // play button should be placed in layout also.
    private Button btnPlay;

    ExoPlayer player = null;
    GDFSRProcessor videoSR = null;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        // media resolution should be one of:
        // 320x180, 426x240, 480x270, 640x360, 854x480
        String mediaPath = "https://test-videos.co.uk/vids/bigbuckbunny/mp4/h264/360/Big_Buck_Bunny_360_10s_10MB.mp4";
        Uri uri = Uri.parse(mediaPath);
        DataSource.Factory dataSourceFactory = new DefaultDataSource.Factory(this);
        MediaSource mediaSource;
        @C.ContentType int type = Util.inferContentType(uri);
        if (type == C.CONTENT_TYPE_DASH) {
            mediaSource =
                new DashMediaSource.Factory(dataSourceFactory)
                    .createMediaSource(MediaItem.fromUri(uri));
        } else if (type == C.CONTENT_TYPE_OTHER) {
            mediaSource =
                new ProgressiveMediaSource.Factory(dataSourceFactory)
                    .createMediaSource(MediaItem..fromUri(uri));
        } else {
            throw new IllegalStateException();
        }

        int videoWidth, videoHeight, videoDuration;
        try (MediaMetadataRetriever retriever = new MediaMetadataRetriever()) {
            retriever.setDataSource(uri.toString(), new HashMap<>());
            String height = retriever.extractMetadata(MediaMetadataRetriever.METADATA_KEY_VIDEO_HEIGHT);
            String width = retriever.extractMetadata(MediaMetadataRetriever.METADATA_KEY_VIDEO_WIDTH);
            String duration = retriever.extractMetadata(MediaMetadataRetriever.METADATA_KEY_DURATION);
            videoWidth = Integer.parseInt(width);
            videoHeight = Integer.parseInt(height);
            videoDuration = Integer.parseInt(duration);
        } catch (IOException e) {
            throw new RuntimeException("cannot get video information: "+uri.toString(), e);
        }
        player = new ExoPlayer.Builder(this).build();
        player.setMediaSource(mediaSource);
        player.prepare();
        videoSR = new GDFSRProcessor(this, videoWidth, videoHeight);
        videoSR.setDisplayView(displayView);
        player.setVideoSurface(videoSR.getPlayerSurface());
        player.addListener(new Player.Listener() {
            @Override
            public void onPlaybackStateChanged(int playbackState) {
                Player.Listener.super.onPlaybackStateChanged(playbackState);
                if (playbackState == Player.STATE_ENDED) {
                    videoSR.release();
                    player.release();
                    videoSR = null;
                    player = null;
                }
            }
        });
        player.play();
    }
    ...
}


```
