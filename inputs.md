
Keyboard, Mouse, and Joystick
=====

Introduction
---

This tutorial explains how to access global inputs: keyboard, mouse and joysticks. This must not be confused with events: real-time inputs allow you to query the global state of keyboard, mouse and joysticks at any time (*"is this button currently pressed?"*, *"where is the mouse?"*) while events notify you when something happens (*"this button was pressed"*, *"the mouse has moved"*).

Keyboard
---

The class that gives access to the keyboard state is [Keyboard](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/window/keyboard.d). It contains only one function, `isKeyPressed`, which checks the current state of a key (pressed or released). It is a static function, so you don't need to instanciate [Keyboard](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/window/keyboard.d) to use it.

This function directly reads the keyboard state, ignoring the focus state of your window. This means that `isKeyPressed` may return true even if your window is inactive.

```D
if (Keyboard.isKeyPressed(Keyboard.Key.Left))
{
    // left key is pressed: move our character
    character.move(1, 0);
}
```

Key codes are defined in the `Keyboard.Key` enum.

> Some key codes may be missing, or interpreted incorrectly depending on your OS and keyboard layout. This is something that will be improved in a future version of SFML.

Mouse
---

The class that gives access to the mouse state is [Mouse](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/window/mouse.d). Like its friend [Keyboard](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/window/keyboard.d), [Mouse](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/window/mouse.d) contains only static functions and is not meant to be instanciated (DSFML only handles a single mouse for the time being).

You can check if buttons are pressed:

```D
if (Mouse.isButtonPressed(Mouse.Button.Left))
{
    // left mouse button is pressed: shoot
    gun.fire();
}
```

Mouse button codes are defined in the `Mouse.Button` enum. DSFML supports up to 5 buttons: left, right, middle (wheel), plus two extra buttons whatever they are.

You can also get and set the current position of the mouse, either relatively to the desktop or to a window:

```D
// get the global mouse position (relatively to the desktop)
Vector2i globalPosition = Mouse.getPosition();

// get the local mouse position (relatively to a window)
Vector2i localPosition = Mouse.getPosition(window); // window is a Window

// set the mouse position globally (relatively to the desktop)
Mouse.setPosition(Vector2i(10, 50));

// set the mouse position localy (relatively to a window)
Mouse.setPosition(Vector2i(10, 50), window); // window is a Window
```

There's no function for reading the current state of the mouse wheel. Indeed, the wheel can only be moved relatively, and has no absolute state. By looking at a key you can tell whether it's pressed or released; by looking at the mouse cursor you can tell where it is located on the screen; but by looking at the mouse wheel you can't tell which "tick" it is on. You can only be notified when it moves (`MouseWheelMoved` event).

Joystick
---

The class that gives access to the joysticks state is [Joystick](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/window/joystick.d). Like the other classes of this tutorial, it contains static functions only.

Joysticks are identified by their index (0 to 7, since DSFML supports up to 8 joysticks). Therefore, the first argument of every function of [Joystick](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/window/joystick.d) is the index of the joystick that you want to query.

You can check whether a joystick is connected or not:

```D
if (Joystick.isConnected(0))
{
    // joystick number 0 is connected
    ...
}
```

You can also get the capabilities of a connected joystick:

```D
// check how many buttons joystick number 0 has
uint buttonCount = Joystick.getButtonCount(0);

// check if joystick number 0 has a Z axis
bool hasZ = Joystick.hasAxis(0, Joystick.Axis.Z);
```

Joystick axes are defined in the `Joystick.Axis` enum. Since they have no special meaning, buttons are simply numbered from 0 to 31.

Finally, you can of course query the state of a joystick's axes and buttons:

```D
// is button 1 of joystick number 0 pressed?
if (Joystick.isButtonPressed(0, 1))
{
    // yes: shoot!
    gun.fire();
}

// what's the current position of the X and Y axes of joystick number 0?
float x = Joystick.getAxisPosition(0, Joystick.Axis.X);
float y = Joystick.getAxisPosition(0, Joystick.Axis.Y);
character.move(x, y);
```

> Joystick states are automatically updated by the event loop. If you don't have one, or need to read a joystick state (for example, check which joysticks are connected) before starting your game loop, you'll have to call the `Joystick.update()` function to make sure that the joystick states are up to date.
