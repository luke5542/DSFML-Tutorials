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

Alright, now we can see what events SFML supports, what they mean and how to use them properly. 

The Closed Event
---

The sf::Event::Closed event is triggered when the user wants to close the window, by all the possible ways that the OS provides ("close" button, keyboard shortcut, etc.). This event only represents a close request, the window is not physically closed.

A typical code will just call window.close() in reaction to this event, to actually close the window. But you may also want to do something, like saving the current application state or asking the user what to do. If you don't do anything, the window remains open.

There's no member associated to this event in the sf::Event union.

```
if (event.type == Event.EventType.Closed)
    window.close();
```

The Resized Event
---

The sf::Event::Resized event is triggered when the window is resized, either by a user action or programmatically by calling window.setSize.

You can use this event to adjust the rendering settings: the viewport if you use OpenGL directly, or the current view if you use sfml-graphics.

The member associated to this event is event.size, it contains the new size of the window.

```
if (event.type == Event.EventType.Resized)
{
    writeln("new width: ", event.size.width);
    writeln("new height: ", event.size.height);
}
```