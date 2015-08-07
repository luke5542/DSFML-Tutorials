Recording audio
=====

Recording to a sound buffer
---

The most common use for captured audio data is for it to be saved to a sound buffer ([SoundBuffer](https://github.com/Jebbs/DSFML/tree/master/src/dsfml/audio/soundbuffer.d)) so that it can either be played or saved to a file.

This can be achieved with the very simple interface of the [SoundBufferRecorder](https://github.com/Jebbs/DSFML/tree/master/src/dsfml/audio/soundbufferrecorder.d) class:

```D
// first check if an input audio device is available on the system
if (!SoundBufferRecorder.isAvailable())
{
    // error: audio capture is not available on this system
    ...
}

// create the recorder
auto recorder = new SoundBufferRecorder();

// start the capture
recorder.start();

// wait...

// stop the capture
recorder.stop();

// retrieve the buffer that contains the captured audio data
const(SoundBuffer) buffer = recorder.getBuffer();
```

The `SoundRecorder.isAvailable()` static method checks if audio recording is supported by the system. It if returns false, you won't be able to use the [SoundBufferRecorder](https://github.com/Jebbs/DSFML/tree/master/src/dsfml/audio/soundbufferrecorder.d) class at all.

The start and stop functions are self-explanatory. The capture runs in its own thread, which means that you can do whatever you want between start and stop. After the end of the capture, the recorded audio data is available in a sound buffer that you can get with the `getBuffer()` method.

With the recorded data, you can then:

+ Save it to a file
```
buffer.saveToFile("my_record.ogg");
```

+ Play it directly
```
Sound sound = new Sound(buffer);
sound.play();
```
+ Access the raw audio data and analyze it, transform it, etc.
```
const(short[]) samples = buffer.getSamples();
doSomething(samples);
```

> If you want to use the captured audio data after the recorder is destroyed or restarted, don't forget to make a copy of the buffer.

Custom recording
---

If storing the captured data in a sound buffer is not what you want, you can write your own recorder. Doing so will allow you to process the audio data while it is captured, (almost) directly from the recording device. This way you can, for example, stream the captured audio over the network, perform real-time analysis on it, etc.

To write your own recorder, you must inherit from the [SoundRecorder](https://github.com/Jebbs/DSFML/tree/master/src/dsfml/audio/soundrecorder.d) abstract base class. In fact, [SoundBufferRecorder](https://github.com/Jebbs/DSFML/tree/master/src/dsfml/audio/soundbufferrecorder.d) is just a built-in specialization of this class.

You only have a single virtual function to override in your derived class: `onProcessSamples()`. It is called every time a new chunk of audio samples is captured, so this is where you implement your specific stuff.

Audio samples are provided to the `onProcessSamples()` function every 100 ms. This is currently hard-coded into DSFML and you can't change that (unless you modify DSFML itself). This may change in the future.

There are also two additional virtual functions that you can optionally override: `onStart()` and `onStop()`. They are called when the capture starts/stops respectively. They are useful for initialization/cleanup tasks.

Here is the skeleton of a complete derived class:

```D
class MyRecorder : SoundRecorder
{
    override bool onStart() // optional
    {
        // initialize whatever has to be done before the capture starts
        ...

        // return true to start the capture, or false to cancel it
        return true;
    }

    override bool onProcessSamples(const(short)[] samples)
    {
        // do something useful with the new chunk of samples
        ...

        // return true to continue the capture, or false to stop it
        return true;
    }

    override void onStop() // optional
    {
        // clean up whatever has to be done after the capture is finished
        ...
    }
}
```

The `isAvailable()`/`start()`/`stop()` functions are defined in the [SoundRecorder](https://github.com/Jebbs/DSFML/tree/master/src/dsfml/audio/soundrecorder.d) base, and thus inherited in every derived classes. This means that you can use any recorder class exactly the same way as the [SoundBufferRecorder](https://github.com/Jebbs/DSFML/tree/master/src/dsfml/audio/soundbufferrecorder.d) class above.

```D
if (!MyRecorder.isAvailable())
{
    // error...
}

MyRecorder recorder;
recorder.start();
...
recorder.stop();
```

Threading issues
---

Since recording is done in a separate thread, it is important to know what exactly happens, and where.

`onStart()` will be called directly by the start function, so it is executed in the same thread that called it. However, `onProcessSample()` and `onStop()` will always be called from the internal recording thread that DSFML creates.

If your recorder uses data that may be accessed concurrently in both the caller thread and in the recording thread, you have to protect it (with a mutex for example) in order to avoid concurrent access, which may cause undefined behavior -- corrupt data being recorded, crashes, etc.

If you're not familiar enough with threading, you can refer to the corresponding tutorial for more information.
