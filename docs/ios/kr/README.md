# IOS
[gdfplay로 돌아가기](https://gdfplay.io)

## Summary 

iOS 기기를 위한 [GDFLab](https://gdflab.com)의 초해상도 기술 SDK.

[GDFPlay](https://gdfplay.io)는 애플리케이션, `GDFSR`이 실제 SDK 라이브러리 모듈이며, `gdfplayer`는 라이브러리 사용 예시 프로젝트 입니다.

GDFSRProcessor is wrapper of GDFSR library dedicated to Video Super Resolution.

This class can be applied to
* [AVPlayer](https://developer.apple.com/documentation/avfoundation/avplayer/) 


입력 영상의 해상도를 알게된 시점에 아래의 코드를 이용해 GDFSRProcessor 객체를 생성합니다.

영상의 모든 프레임에 대해 아래처럼 업스케일을 반복적으로 진행합니다.

---
## Requirements
- SDK: iOS 11 / Storyboard 11
- SDK 컴파일 iOS 11
- laguege: swift
- Cocoapod install(TensorFlowLiteSwift/Metal, CryptoSwift)
- x86 기반 iphone simulator에서는 정상 동작하지 않습니다.(실제 device에서 테스트 요망)
- tensorflow lite제안으로 인해 SDK는 arm64기반 모바일에서만 작동합니다.
- iOS 11 지원기기

---
## Recommendations
<table>
<tr><th style="text-align:center">해상도</th><th style="text-align:center">업스케일</th></tr>
<tr><td style="text-align:center">270p ~ 480p</td><td style="text-align:center">3x</td></tr>
<tr><td style="text-align:center">480p ~ 640p</td><td style="text-align:center">2x</td></tr>
</table>
위 표는 해상도에 따라 권장하는 배율입니다. SDK 발급 시 고객의 요청에 따라 해상도를 맞춰드립니다. 스트리밍하시는 환경에 맞춰 설정하시면 더욱 효율적으로 사용하실 수 있습니다. 만약 해상도 설정에 관한 문의가 있으시다면 언제든지 문의해주시기 바랍니다.

---
## Flowchart

<p align="center">
<img src="../../../img/flowchart.png">
</p>

---
## Usage
### Storyboard - Usage of this class with AVPlayer:

```swift
import MetalKit
import AVKit
import VideoToolbox
import GDFSR

class VideoMetalView: MTKView {
    private var videoSR: GDFSRVideo?
    private var player: AVPlayer?

    deinit {
        stop()
    }
    func play(stream: URL) throws {
        let item: AVPlayerItem = AVPlayerItem(url: stream)
        let maxResSize: CGSize = CGSize(width: 854, height: 480)
        let scale: Int = 2
        device = device ?? MTLCreateSystemDefaultDevice()
        framebufferOnly = false
        layer.isOpaque = true
        self.player = AVPlayer(playerItem: item)
        self.videoSR = GDFSRVideo(metalView: self, videoItem: item, maxResSize: maxResSize, scale: scale, completionHandler: {
            // now video can be played
            self.play()
        })
    }
    func play() { player?.play() }
    func pause() { player?.pause() }
    func stop() {
        player?.rate = 0
        videoSR?.release()
        videoSR = null
        player = null
    }
}
```

```swift
import Foundation
import UIKit
import OSLog
import GDFSR

class VideoViewController: UIViewController, VideoMetalViewDelegate {

    @IBOutlet weak var videoView: VideoMetalView!
    @IBOutlet weak var videoControls: UIStackView!
    @IBOutlet weak var sliProgress: UISlider!
    @IBOutlet weak var lblElapsed: UILabel!
    @IBOutlet weak var lblRemain: UILabel!
    @IBOutlet weak var closeControl: UIStackView!
    @IBOutlet weak var btnPlay: UIButton!
    @IBOutlet weak var txtFileName: UITextField!
    @IBOutlet weak var segBtns: UISegmentedControl!
    @IBOutlet weak var btnInfo: UIButton!
    @IBOutlet weak var btnRepeat: UIButton!
    @IBOutlet weak var noSDKText: UILabel!
    
    var fileItem: FileItem!

    private var showState = false
    private var isAuto = false
    private var isPlaying = true
    private let playImage = UIImage(systemName: "play.fill")
    private let pauseImage = UIImage(systemName: "pause.fill")
    private var showInfo = false
    private var repeatVideo = false
    private var prevPoint: CGPoint = .zero

    
    override func viewDidLoad() {
        super.viewDidLoad()

        videoView?.videoDelegate = self
    
        // set video file name
        txtFileName.text = fileItem.name
        showControls(true)
        
        // examplevideo
        String mediaPath = "https://test-videos.co.uk/vids/bigbuckbunny/mp4/h264/360/Big_Buck_Bunny_360_10s_10MB.mp4"
        
        // video play
        DispatchQueue.main.async {
            if let vv = self.videoView {
                do {
                    self.btnPlay?.setBackgroundImage(self.pauseImage, for: .normal)
                    try vv.play(stream: URL(string:String mediaPath)!)
                    print("[VideoViewControrller] play")
                } catch {
                    GDFLog.error("\(error)")
                    self.onClose(self)
                }
            } else {
                self.onClose(self)
            }
        }
    }
```



--- 
## More Information
For More information and API list guide Please visit 
<a href="https://gdfplay.io/developer/doc/sdk/get-started/quick-start#gdfsdk">GDFPlay.io</a>
