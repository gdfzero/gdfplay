# Android
[gdfplay로 돌아가기](https://gdfplay.io)

## Summary
Android 기기를 위한 [GDFLab](https://gdflab.com)의 초해상도 기술 SDK.

[GDFPlay](https://gdfplay.io)는 애플리케이션, `GDFSR`이 실제 SDK 라이브러리 모듈이며, `gdfplayer`는 라이브러리 사용 예시 프로젝트 입니다.

GDFSRProcessor is wrapper of GDFSR library dedicated to Video Super Resolution.

This class can be applied to
* <a href="https://developer.android.com/guide/topics/media/mediaplayer">MediaPlayer</a> or
* <a href="https://exoplayer.dev">ExoPlayer</a>.


입력 영상의 해상도를 알게된 시점에 아래의 코드를 이용해 GDFSRProcessor 객체를 생성합니다.

영상의 모든 프레임에 대해 아래처럼 업스케일을 반복적으로 진행합니다.

---

## Requirements
- 안드로이드 SDK : 28 /29
- 안드로이드 컴파일 SDK : 33
- 안드로이드 NDK : 21.4
- 퀄컴 스냅드래곤 865 이상의 프로세서가 탑재된 기기에서 잘 동작합니다

----------
## Recommendations
<table>
<tr><th style="text-align:center">해상도</th><th style="text-align:center">업스케일</th></tr>
<tr><td style="text-align:center">270p ~ 480p</td><td style="text-align:center">3x</td></tr>
<tr><td style="text-align:center">480p ~ 640p</td><td style="text-align:center">2x</td></tr>
</table>
위 표는 해상도에 따라 권장하는 배율입니다. SDK 발급 시 고객의 요청에 따라 해상도를 맞춰드립니다. 스트리밍하시는 환경에 맞춰 설정하시면 더욱 효율적으로 사용하실 수 있습니다. 만약 해상도 설정에 관한 문의가 있으시다면 언제든지 문의해주시기 바랍니다.

----------
## Flowchart

<p align="center">
<img src="../../../img/flowchart.png">
</p>


----------
## Method 1: Usage With Video
### Method 1.1: Usage of this class with MediaPlayer:

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

### Method 1.2: Usage of this class with ExoPlayer:
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


----------
## Method 2 : Usage With Image (Frame by Frame SR)
This methods gives SDK users power to upscale Frame by Frame regardless of player they Use
### 예제

다른 API 호출전에 아래의 코드를 통해 초기화가 선행되어야 합니다. 최초 Activity 생성시 호출 가능합니다.

```java
// initialize GDFUpscaler
GDFUpscaler.initialize(getApplicationContext());
```

입력 영상의 해상도를 알게된 시점에 아래의 코드를 이용해 GDFUpsacler 객체를 생성합니다.

```java
// create GDFUpscaler instance
GDFUpscaler upscaler = GDFUpscaler.getUpscaler(width, height);
```
영상의 모든 프레임에 대해 아래처럼 업스케일을 반복적으로 진행합니다.

### ByteBuffer to Bitmap

```java
ByteBuffer input;	// input video frame(ARGB_8888)
Bitmap result: 	// output video frame(must be allocated)  
//i.e result =  Bitmap.createBitmap(width * scale, height * scale, Bitmap.Config.ARGB_8888);
//where  scale = upscaler.getScale();

// do upscale inference
upscaler.upscaleToBitmap(input, result);

// use result Bitmap object in your code
...

// clearBuffers() must be called after each time inference (upscaleToBitmap) is called, or may cause memory leak.
upscaler.clearBuffers();



//When video/stream finished
`upscaler.close()`를 통해 객체를 해제합니다.
```

### More Information
For More information and API list guide Please visit 
<a href="https://gdfplay.io/developer/doc/sdk/get-started/quick-start#gdfsdk">GDFPlay.io</a>

