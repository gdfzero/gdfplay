---
layout: default
title: Getting Started
nav_order: 1
parent: Android
---

# Getting Started

Super Resolution Technology SDK from [GDFLab](https://gdflab.com) for Android devices.

[GDFPlay](https://gdfplay.io) is an application, `GDFSR` is an actual SDK library module, and `gdfplayer` is an example project for using the library.

GDFSRProcessor is wrapper of GDFSR library dedicated to Video Super Resolution.

This class can be applied to
* <a href="https://developer.android.com/guide/topics/media/mediaplayer">MediaPlayer</a> or
* <a href="https://exoplayer.dev">ExoPlayer</a>.
* <a href="https://docs.aws.amazon.com/ivs/latest/userguide/player.html">IVS Player</a>.


When the resolution of the input video is known, create a GDFSRProcessor object using the code below.

Repeat upscaling for every frame of the video as shown below.