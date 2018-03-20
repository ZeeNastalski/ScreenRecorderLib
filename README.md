# ScreenRecorderLib
A .NET library for screen recording in Windows, using native Microsoft Media Foundation for realtime encoding to h264 video or PNG images.

Available on [NuGet](https://www.nuget.org/packages/ScreenRecorderLib/).

**Basic usage:**

This will start a video recording using the default settings:
* 4000kbps video bitrate
* 30fps
* no audio
* mouse pointer enabled

**For audio recording to work, system audio must be set to 16 bit, 44100hz or 48000hz**

```csharp
        using ScreenRecorderLib;
        
        Recorder _rec;
		Stream _outStream;
        void CreateRecording()
        {
            _rec = Recorder.CreateRecorder();
            _rec.OnRecordingComplete += Rec_OnRecordingComplete;
            _rec.OnRecordingFailed += Rec_OnRecordingFailed;
            _rec.OnStatusChanged += Rec_OnStatusChanged;
			//Record to a file
			string videoPath = Path.Combine(Path.GetTempPath(), "test.mp4");
            _rec.Record(videoPath);
			//..Or to a stream
			//_outStream = new MemoryStream();
			//_rec.Record(_outStream);
        }
        void EndRecording()
        {
            _rec.Stop(); 
        }
        private void Rec_OnRecordingComplete(object sender, RecordingCompleteEventArgs e)
        {
			//Get the file path if recorded to a file
            string path = e.FilePath;	
			//or do something with your stream
			//... something ...
			_outStream?.Dispose();
        }
        private void Rec_OnRecordingFailed(object sender, RecordingFailedEventArgs e)
        {
            string error = e.Error;
			_outStream?.Dispose();
        }
        private void Rec_OnStatusChanged(object sender, RecordingStatusEventArgs e)
        {
            RecorderStatus status = e.Status;
        }
```

To change the options, pass a RecorderOptions when creating the Recorder:

```csharp
            RecorderOptions options = new RecorderOptions
            {
                RecorderMode = RecorderMode.Video,
                IsThrottlingDisabled = false, //If throttling is disabled, out of memory exceptions may eventually crash the program, depending on how much RAM your system has.
				IsHardwareEncodingEnabled = true, //Enabled by default.
                IsLowLatencyEnabled = false, //Provides faster encoding, but can reduce quality.
                IsMp4FastStartEnabled = false, //Fast start writes the mp4 header at the beginning of the file, to facilitate streaming.
                AudioOptions = new AudioOptions { Bitrate = AudioBitrate.bitrate_128kbps, Channels = AudioChannels.Stereo, IsAudioEnabled = true },
                VideoOptions = new VideoOptions { Bitrate = 8000 * 1000, Framerate = 60, IsMousePointerEnabled = true, IsFixedFramerate = true, EncoderProfile = H264Profile.Main }
            };
            _rec = Recorder.CreateRecorder(options);
```

To only record a portion of the screen, or a different monitor than the main monitor, you can also set DisplayOptions:
```csharp
            //crop to a 400x400 pixel square at x=400,y=400. Passing 0 for these values will default to full screen recording.
            int left = 400;
            int top = 400;
            int right = 800;
            int bottom=800;
            int monitorIndex= 0;//0 is always primary monitor.
            RecorderOptions options = new RecorderOptions
            {
               DisplayOptions = new DisplayOptions(monitorIndex, left, top, right, bottom)
            }
```

See the sample project for a full implementation, including multimonitor.
