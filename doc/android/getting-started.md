---
layout: default
title: Getting Started
nav_order: 1
parent: Android
---

# Getting Started

Android 기기를 위한 [GDFLab](https://gdflab.com)의 초해상도 기술 SDK.

[GDFPlay](https://gdfplay.io)는 애플리케이션, `GDFSR`이 실제 SDK 라이브러리 모듈이며, `gdfplayer`는 라이브러리 사용 예시 프로젝트 입니다.

GDFSRProcessor is wrapper of GDFSR library dedicated to Video Super Resolution.

This class can be applied to
* <a href="https://developer.android.com/guide/topics/media/mediaplayer">MediaPlayer</a> or
* <a href="https://exoplayer.dev">ExoPlayer</a>.
* <a href="https://docs.aws.amazon.com/ivs/latest/userguide/player.html">IVS Player</a>.


입력 영상의 해상도를 알게된 시점에 아래의 코드를 이용해 GDFSRProcessor 객체를 생성합니다.

영상의 모든 프레임에 대해 아래처럼 업스케일을 반복적으로 진행합니다