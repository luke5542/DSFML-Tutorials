User Data Streams
=====

Introduction
---

DSFML has several resource classes: images, fonts, sounds, etc. In most programs, these resources will be loaded from files, with the help of their `loadFromFile()` method. In a few other situations, resources will be packed directly into the executable or in a big data file, and loaded from memory with `loadFromMemory()`. These methods cover *almost* all the possible use cases -- but they are not without their limits.

Sometimes you want to load files from unusual places, such as a compressed/encrypted archive, or a remote network location, for example. For these special situations, DSFML provides a third loading method: `loadFromStream()`. This method reads data from an [InputStream](http://dsfml.com/dsfml/system/inputstream.html) interface, which allows you to define your own implementations.

In this tutorial you'll learn how to write and use your own derived input stream.


And Standard Streams?
---

Like many other languages, D already has classes for various input data streams. Unfortunately, these classes are not very user friendly, and can even become very complicated if you want to implement more than trivial stuff. 

More importantly, to stick to the original SFML design, DSFML provides its own stream interface, which is hopefully a lot more *simple and fast*.

Input Stream
---

The [InputStream](http://dsfml.com/dsfml/system/inputstream.html) interface declares four methods:

```D
interface InputStream
{
    long read(void[] data);

    long seek(long position);

    long tell();

    long getSize();
}
```

`read()` must extract size bytes of data from the stream, and copy them to the supplied data address; it returns the number of bytes read, or -1 on error.

`seek()` must change the current reading position in the stream; its position argument is the absolute byte offset to jump to (so it is relative to the beginning of the data, not to the current position); it returns the new position, or -1 on error.

`tell()` must return the current reading position (in bytes) in the stream, or -1 on error.

`getSize()` must return the total size (in bytes) of the data which is contained in the stream, or -1 on error.

To create your own working stream, you must implement all of these four methods according to their requirements.

An Example
---

Below is a complete and working implementation of a custom input stream. It's not very useful: we'll write a stream that reads data from a file, `FileStream`. But it is simple enough so that you can focus on how the code works, and not get lost in implementation details.

In this example we'll use D's file API, so we have a [File](http://dlang.org/phobos/std_stdio.html#.File) member. We also add a default constructor, a destructor, and a function to open the file.

An interesting note, while D has many links back to the C/C++ APIs, in this specific example when we use [File](http://dlang.org/phobos/std_stdio.html#.File), we are actually using an encapsulation of `FILE*`` that is automatically cleaned up for us when our m_file reference leaves scope.

Here is the implementation:

```D
class FileStream : InputStream
{
    File m_file;

    this()
    {
        // Constructor code
    }

    bool open(string fileName)
    {
        try
        {
            m_file.open(fileName);
        }
        catch(Exception e)
        {
            writeln(e.msg);
        }

        return m_file.isOpen;
    }

    long read(void[] data)
    {

        if(m_file.isOpen)
        {
            return m_file.rawRead(data).length;
        }
        else
        {
            return -1;
        }
    }

    long seek(long position)
    {
        if(m_file.isOpen)
        {
            m_file.seek(position);
            return tell();
        }
        else
        {
            return -1;
        }
    }

    long tell()
    {

        if(m_file.isOpen)
        {
            return m_file.tell;
        }
        else
        {
            return -1;
        }
    }

    long getSize()
    {
        if(m_file.isOpen)
        {
            long position = m_file.tell;

            m_file.seek(0,SEEK_END);

            long size = tell();

            seek(position);

            return size;
        }
        else
        {
            return -1;
        }
    }
}
```

Note that, as explained above, all functions return -1 on error.

Don't forget to check the forum and wiki (including the standard SFML forum and wiki), chances are that another user already wrote an [InputStream](http://dsfml.com/dsfml/system/inputstream.html) class that matches your needs. And if you write a new one and feel like it could be useful outside your project, don't hesitate to share!

Using Your Stream
---

Using a custom stream class is straight-forward: instantiate it, and pass it to the `loadFromStream()` (or `openFromStream()`) method of the object that you want to load.

```D
auto stream = new FileStream();
stream.open("image.png");

auto streamTexture = new Texture();
if(streamTexture.loadFromStream(stream))
{
    writeln("Texture loaded!");
}
```

Common Mistakes
---

Some resource classes are not loaded completely after `loadFromStream` has been called. Instead, they continue to read from their data source as long as they are used. This is the case for [Music](http://dsfml.com/dsfml/audio/music.html), which streams audio samples as they are played, and for [Font](http://dsfml.com/dsfml/graphics/font.html), which loads glyphs on the fly depending on the texts content.

As a consequence, the stream instance that you used to load a music or a font, as well as its data source, must remain alive as long as the resource uses it. If it is destroyed while still being used, the behavior is undefined (can be a crash, corrupt data, or nothing visible).

Another common mistake is to return whatever the internal functions return directly, but sometimes it doesn't match what DSFML expects. For example, a function you use internally for `seek()` could return 0 for success instead of the position.
