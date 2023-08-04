---
layout: default
title: Quick Start
nav_order: 5
parent: iOS
---

# Quick Start
## Sample Codes
### Run SDK on MTKView

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

### Run SDK on VideoViewController

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