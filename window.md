Opening and Managing a DSFML Window
=====

Introduction
---

This tutorial only describes how to open and manage a window. Drawing stuff is out of the scope of the dsfml-window module: it is handled by the dsfml-graphics module. However, the window management remains exactly the same so reading this tutorial is important in any case.

Opening a Window
---

Windows in SFML are defined by the [Window](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/window/window.d) class. A window can be created and opened directly upon construction:

```
import dsfml.window;


void main()
{
	auto window = new Window(VideoMode(800, 600), "My Window");

	...

}
```

The first argument, the video mode, defines the size of the window (the inner size, without the titlebar and borders). Here, we create a window of 800x600 pixels.
The [VideoMode](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/window/videomode.d) class has some interesting static functions to get the desktop resolution, or the list of valid video modes for fullscreen mode. Don't hesitate to have a look at its documentation.

The second argument is simply the title of the window.

This constructor accepts a third optional argument: a style, which allows to choose which decorations and features you want. You can use any combination of the following styles: 

--- | --- | ---
*Still* | `renders` | **nicely**
1 | 2 | 3

| Style | Description |
| --- | --- |
| sf::Style::None | No decoration at all (useful for splash screens, for example); this style cannot be combined with others |
| sf::Style::Titlebar | The window has a titlebar |
| sf::Style::Resize | The window can be resized and has a maximize button |
| sf::Style::Close | The window has a close button |
| sf::Style::Fullscreen | The window is shown in fullscreen mode; this style cannot be combined with others, and requires a valid video mode |
| sf::Style::Default | The default style, which is a shortcut for `Titlebar OR Resize OR Close` |

Bringing the Window to Life
---

```
import dsfml.window;


void main()
{
	auto window = new Window(VideoMode(800, 600), "My Window");

	// run the program as long as the window is open
	while (window.isOpen())
    {
    	// check all the window's events that were triggered since the last iteration of the loop
        Event event;
        while(window.pollEvent(event))
        {
        	// "close requested" event: we close the window
            if(event.type == event.EventType.Closed)
            {
                window.close();
            }
        }
    }
}
```

Playing With the Window
---

Controlling the Framerate
---

Things to Know About Windows
---