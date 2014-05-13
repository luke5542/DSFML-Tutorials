Events Explained
=====

Introduction
---

This tutorial is a detailed list of window events. It describes them, and shows how to (and not to) use them.

The Event Type
---

Before dealing with events, it is important to understand what the [Event](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/window/event.d) type is, and how to correctly use it. [Event](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/window/event.d) is a union, which means that only one of its member is valid at a time (remember that, in D, all the members of a union share the same memory space). The valid member is the one that matches the event type, for example `event.key` for a `KeyPressed` event. Trying to read any other member will result in an undefined behaviour (most likely: random or invalid values). So never try to use an event member that doesn't match its type.

[Event](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/window/event.d) instances are filled by the `pollEvent` (or `waitEvent`) function of the [Window](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/window/window.d) class. Only these two functions can produce valid events, any attempt to use a [Event](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/window/event.d) without first a successful call to `pollEvent` (or `waitEvent`) will result in the same undefined behaviour that I mentioned above.

To be clear, here is what a typical event loop looks like:

```
Event event;

while(window.pollEvent(event))
{

	// check the type of the event
    switch(event.type)
    {
    	// window closed
        case Event.EventType.Closed:
            window.close();
            break;

        // key pressed
        case Event.EventType.KeyPressed:
            ...
            break;

		// we don't process other types of events
        default:
            break;
    }
}
```

> Read the above paragraph once again and make sure that it's printed in your head, the [Event](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/window/event.d) union causes too many problems to inadvertent programmers.

Alright, now we can see what events DSFML supports, what they mean and how to use them properly. 

The Closed Event
---

The Event.Closed event is triggered when the user wants to close the window, by all the possible ways that the OS provides ("close" button, keyboard shortcut, etc.). This event only represents a close request, the window is not physically closed.

A typical code will just call window.close() in reaction to this event, to actually close the window. But you may also want to do something, like saving the current application state or asking the user what to do. If you don't do anything, the window remains open.

There's no member associated to this event in the Event union.

```
if (event.type == Event.EventType.Closed)
    window.close();
```

The Resized Event
---

The Event.Resized event is triggered when the window is resized, either by a user action or programmatically by calling window.setSize.

You can use this event to adjust the rendering settings: the viewport if you use OpenGL directly, or the current view if you use Dsfml-graphics.

The member associated to this event is event.size, it contains the new size of the window.

```
if (event.type == Event.EventType.Resized)
{
    writeln("new width: ", event.size.width);
    writeln("new height: ", event.size.height);
}
```

The LostFocus and GainedFocus Events
---

The Event.LostFocus and Event.GainedFocus events are triggered when the window loses/gains the focus, which happens when you switch the currently active window. When the window is out of focus, it doesn't receive keyboard events.

This event can be used if you want to pause your game when the window is inactive.

There's no member associated to these events in the Event union.

```
if (event.type == Event.EventType.LostFocus)
    myGame.pause();

if (event.type == Event.EventType.GainedFocus)
    myGame.resume();
```

The TextEntered Event
---

The Event.TextEntered event is triggered when a character is typed. This must not be confused with the KeyPressed event: TextEntered interprets the user input and produces the appropriate printable character. For example, pressing '^' then 'e' on a french keyboard will produce two KeyPressed events, but a single TextEntered event containing the 'ê' character. It works with all the input methods provided by the OS, even the most specific or complex ones.

This event is typically used to catch user input in a text field.

The member associated to this event is event.text, it contains the Unicode value of the entered character. You can either put it directly in a String, or cast it to a char after making sure that it is in the ASCII range (0 - 127).

```
if (event.type == Event.EventType.TextEntered)
{
    if (event.text.unicode < 128)
        writeln("ASCII character typed: ", event.text.unicode);
}
```

Note that, since they are part of the Unicode standard, some non-printable characters such as backspace are generated by this event. In most cases you'll need to filter them out.

> Many programmers use the KeyPressed event to get user input, and start to implement crazy algorithms that try to interpret all the possible key combinations to produce correct characters. Don't do that!


The KeyPressed and KeyReleased Events
---

The Event.KeyPressed and Event.KeyReleased events are triggered when a keyboard key is pressed/released.

If a key is held, multiple KeyPressed events will be generated, at the default OS delay (ie. the same delay that applies when you hold a letter in a text editor). To disable repeated KeyPressed events, you can call window.setKeyRepeatEnabled(false). On the contrary, obviously, KeyReleased events can never be repeated.

This event is the one to use if you want to trigger an action exactly once when a key is pressed or released, like making a character jump with space, or exiting something with escape.

> Sometimes, people try to use the KeyPressed event to implement smooth movement. Doing so will not produce smooth movements, because when you hold a key you only get a few events (remember, the repeat delay). Smooth movements can only be achieved by using real-time keyboard input with Keyboard (see the dedicated tutorial).

The member associated to these events is event.key, it contains the code of the pressed/released key, as well as the current state of modifier keys (alt, control, shift, system).

```
if (event.type == Event.EventType.KeyPressed)
{
    if (event.key.code == Keyboard.Key.Escape)
    {
        writeln("the escape key was pressed");
        writeln("control:", event.key.control);
        writeln("alt:", event.key.alt);
        writeln("shift:", event.key.shift);
        writeln("system:", event.key.system);
    }
}
```

Note that some keys have a special meaning for the OS, and will produce unexpected behaviours. An exemple is the F10 key on Windows, which "steals" the focus, or the F12 key which starts the debugger when using Visual Studio. This will probably be solved in a future version of DSFML.

The MouseWheelMoved Event
--- 

The `Event.EventType.MouseWheelMoved` event is triggered when the mouse wheel moves up or down.

The member associated to this event is `event.mouseWheel`, it contains the amount of ticks the wheel has moved, as well as the current position of the mouse cursor.

```
if (event.type == Event.EventType.MouseWheelMoved)
{
    writeln("wheel movement: ", event.mouseWheel.delta);
    writeln("mouse x: ", event.mouseWheel.x);
    writeln("mouse y: ", event.mouseWheel.y);
}
```

The MouseButtonPressed and MouseButtonReleased Events
---

The `Event.EventType.MouseButtonPressed` and `Event.EventType.MouseButtonReleased` events are triggered when a mouse button is pressed/released.

DSFML supports 5 mouse buttons: left, right, middle (wheel), extra #1 and extra #2 (side buttons).

The member associated to these events is `event.mouseButton`, it contains the code of the pressed/released button, as well as the current position of the mouse cursor.

```
if (event.type == Event.EventType.MouseButtonPressed)
{
    if (event.mouseButton.button == Mouse.Button.Right)
    {
        writeln("the right button was pressed");
        writeln("mouse x: ", event.mouseButton.x);
        writeln("mouse y: ", event.mouseButton.y);
    }
}
```

The MouseMoved Event
---

The `Event.EventType.MouseMoved` event is triggered when the mouse moves within the window.

This event is triggered even if the window doesn't have the focus. However, it is triggered only when the mouse moves within the inner area of the window, not when it moves over the titlebar or borders.

The member associated to this event is `event.mouseMove`, it contains the current position of the mouse cursor relatively to the window.

```
if (event.type == Event.EventType.MouseMoved)
{
    writeln("new mouse x: ", event.mouseMove.x);
    writeln("new mouse y: ", event.mouseMove.y);
}
```

The MouseEntered and MouseLeft Event
---

The `Event.EventType.MouseEntered` and `Event.EventType.MouseLeft` events are triggered when the mouse cursor enters/leaves the window.

There's no member associated to these events in the Event union.

```
if (event.type == Event.EventType.MouseEntered)
    writeln("the mouse cursor has entered the window");

if (event.type == Event.EventType.MouseLeft)
    writeln("the mouse cursor has left the window");
```

The JoystickButtonPressed and JoystickButtonReleased Events
---

The `Event.EventType.JoystickButtonPressed` and `Event.EventType.JoystickButtonReleased` events are triggered when a joystick button is pressed/released.

DSFML supports up to 8 joysticks and 32 buttons.

The member associated to these events is `event.joystickButton`, it contains the identifier of the joystick and the index of the pressed/released button.

```
if (event.type == Event.EventType.JoystickButtonPressed)
{
    writeln("joystick button pressed!");
    writeln("joystick id: ", event.joystickButton.joystickId);
    writeln("button: ", event.joystickButton.button);
}
```

The JoystickMoved Event
---

The `Event.EventType.JoystickMoved` event is triggered when a joystick axis moves.

Joystick axes are typically very sensitive, that's why DSFML uses a detection threshold to avoid spamming your event loop with tons of JoystickMoved events. This threshold can be changed with the `Window.setJoystickThreshold` method, in case you want to receive more or fewer joystick move events.

DSFML supports 8 joystick axes: X, Y, Z, R, U, V, POV X and POV Y. How they map to your joystick depends on its driver.

The member associated to this event is `event.joystickMove`, it contains the identifier of the joystick, the name of the axis, and its current position (in the range [-100, 100]).

```
if (event.type == Event.EventType.JoystickMoved)
{
    if (event.joystickMove.axis == Joystick.X)
    {
        writeln("X axis moved!");
        writeln("joystick id: ", event.joystickMove.joystickId);
        writeln("new position: ", event.joystickMove.position);
    }
}
```

The JoystickConnected and JoystickDisconnected Events
---

The Event.JoystickConnected and Event.JoystickDisconnected events are triggered when a joystick is connected/disconnected.

The member associated to this event is event.joystickConnect, it contains the identifier of the connected/disconnected joystick.

if (event.type == Event.JoystickConnected)
    writeln("joystick connected: ", event.joystickConnect.joystickId);

if (event.type == Event.JoystickDisconnected)
    writeln("joystick disconnected: ", event.joystickConnect.joystickId);
