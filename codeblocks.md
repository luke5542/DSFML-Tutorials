Code::Blocks (Windows)
=====

This assumes that you have already either [built DSFML-C](https://github.com/Jebbs/DSFML/wiki/Building-The-Library-And-Your-First-DSFML-Program) from source, or [downloaded](https://github.com/Jebbs/DSFML/wiki/Getting-Started-in-Windows) the pre-built libraries for your system. It's also assumed that you have read and built the DSFML library files as described [here](https://github.com/Jebbs/DSFML/wiki/Building-Your-First-DSFML-Program%28Windows%29).

As a result of the above process, you should have the following ready to integrate with Code::Blocks:
* DSFML-C library files (named something like dsfml-graphics.lib)
* DSFML library files (named something like dsfml-graphics-2.lib)
* DSFML-C binary files (note, you'll also need the dependency .dll's that can be found in the download)
* DSFML D source

To get D working with Code::Blocks, you can follow their guide [here](http://wiki.codeblocks.org/index.php?title=Installing_a_supported_compiler#Digital_Mars_D_Compiler_for_Windows).

Dependency Setup
===

If you have Code::Blocks setup correctly, you should be able to easily create a new D project from the `File -> New -> Project` window.

After the project has been created, right click the project name and select `Build Options`. Under the `Linker Settings` tab, click `Add` at the bottom to add in the DSFML-C and DSFML library files from where you have them located on disk.

Next, under the `Search Directories` tab, select the `Compiler` tab and add the directory of your DSFML source directory. Under the `Linker` tab, add the directories of your:
* DSFML-C binary files,
* DSFML-C library files,
* and your DSFML binary files.

Now for some code
===

In the main D file of your project, place the following code:

```D
module main;

import dsfml.graphics;

void main(string[] args)
{
    auto window = new RenderWindow(VideoMode(800,600),"Hello DSFML!");

    auto head = new CircleShape(100);
    head.fillColor = Color.Green;
    head.position = Vector2f(300,100);

    auto leftEye = new CircleShape(10);
    leftEye.fillColor = Color.Blue;
    leftEye.position = Vector2f(350,150);

    auto rightEye = new CircleShape(10);
    rightEye.fillColor = Color.Blue;
    rightEye.position = Vector2f(430,150);

    auto smile = new CircleShape(30);
    smile.fillColor = Color.Red;
    smile.position = Vector2f(368,200);

    auto smileCover = new RectangleShape(Vector2f(60,30));
    smileCover.fillColor = Color.Green;
    smileCover.position = Vector2f(368,200);

    while (window.isOpen())
    {
        Event event;

        while(window.pollEvent(event))
        {
            if(event.type == event.EventType.Closed)
            {
                window.close();
            }
        }

        window.clear();

        window.draw(head);
        window.draw(leftEye);
        window.draw(rightEye);
        window.draw(smile);
        window.draw(smileCover);

        window.display();
    }
}
```

Now just build and run the project and you should be done!

![A Running DSFML Program!](https://camo.githubusercontent.com/c72a4470bb37fbb39c770aaefec8f964953d1f24/687474703a2f2f692e696d6775722e636f6d2f686274314942482e706e67 "A Running DSFML Program!")
