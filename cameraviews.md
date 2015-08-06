Controlling the 2D Camera with Views
=====

What is a View?
---

In games, it is not uncommon to have levels which are much bigger than the window itself. You only see is a small part of them. This is typically the case in RPGs, platform games, and many other genres. What developers might tend to forget is that they define entities in a 2D world, not directly in the window. The window is just a view, it shows a specific area of the whole world. It is perfectly fine to draw several views of the same world in parallel, or draw the world to a texture rather than to a window. The world itself remains unchanged, what changes is just the way it is seen.

Since what is seen in the window is just a small part of the entire 2D world, you need a way to specify which part of the world is shown in the window. Additionally, you may also want to define where/how this area will be shown within the window. These are the two main features of DSFML views.

To summarize, views are what you need if you want to scroll, rotate or zoom your world. They are also the key to creating split screens and mini-maps.

Defining What the View Views
---

The class which encapsulates views in DSFML is [View](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/view.d). It can be constructed directly with a definition of the area to view:

```D
// create a view with the rectangular area of the 2D world to show
View view1 = new View(FloatRect(200, 200, 300, 200));

// create a view with its center and size
View view2 = new View(Vector2f(350, 300), Vector2f(300, 200));
```

These two definitions are equivalent: Both views will show the same area of the 2D world, a 300x200 rectangle centered on the point (350, 300).

![A View](http://www.sfml-dev.org/tutorials/2.3/images/graphics-view-initial.png "A View")

If you don't want to define the view upon construction or want to modify it later, you can use the equivalent setters:

```D
View view1 = new View();
view1.reset(FloatRect(200, 200, 300, 200));

View view2 = new View();
view2.center = Vector2f(350, 300);
view2.size = Vector2f(200, 200);
```

Once your view is defined, you can transform it to make it show a translated/rotated/scaled version of your 2D world.

**Moving (Scrolling) the View**

Unlike drawable entities, such as sprites or shapes whose positions are defined by their top-left corner (and can be changed to any other point), views are always manipulated by their center -- this is simply more convenient. That's why the property to change the position of a view is named center, and not position.

```D
// move the view at point (200, 200)
view.center = Vector2f(200, 200);

// move the view by an offset of (100, 100) (so its final position is (300, 300))
view.move(Vector2f(100, 100));
```

![A Translated View](http://www.sfml-dev.org/tutorials/2.3/images/graphics-view-translated.png "A Translated View")

**Rotating the View**

To rotate a view, use the `rotation` property.

```D
// rotate the view at 20 degrees
view.rotation = 20;

// rotate the view by 5 degrees relatively to its current orientation (so its final orientation is 25 degrees)
view.rotation = view.rotation + 5;
```

![A Rotated View](http://www.sfml-dev.org/tutorials/2.3/images/graphics-view-rotated.png "A Rotated View")

**Zooming (Scaling) the View**

Zooming in (or out) a view is done through to resizing it, so the function to use is setSize.

```D
// resize the view to show a 1200x800 area (we see a bigger area, so this is a zoom out)
view.size = Vector2f(1200, 800);

// zoom the view relatively to its current size (apply a factor 0.5, so its final size is 600x400)
view.zoom(0.5f);
```

![A Scaled View](http://www.sfml-dev.org/tutorials/2.3/images/graphics-view-scaled.png "A Scaled View")

Defining How the View is Viewed
---

Now that you've defined which part of the 2D world is seen in the window, let's define where it is shown. By default, the viewed contents occupy the full window. If the view has the same size as the window, everything is rendered 1:1. If the view is smaller or larger than the window, everything is scaled to fit in the window.

This default behavior is suitable for most situations, but it might need to be changed sometimes. For example, to split the screen in a multiplayer game, you may want to use two views which each only occupy half of the window. You can also implement a minimap by drawing your entire world to a view which is rendered in a small area in a corner of the window. The area in which the contents of the view is shown is called the viewport.

To set the viewport of a view, you can use the `viewport` property.

```D
// define a centered viewport, with half the size of the window
view.viewport = FloatRect(0.25f, 0.25, 0.5f, 0.5f);
```
![A Viewport](http://www.sfml-dev.org/tutorials/2.3/images/graphics-view-viewport.png "A Viewport")

You might have noticed something very important: The viewport is not defined in pixels, but instead as a ratio of the window size. This is more convenient: it allows you to not have to track resize events in order to update the size of the viewport every time the size of the window changes. It is also more intuitive: you would probably define your viewport as a fraction of the entire window area anyway, not as a fixed-size rectangle.

Using a viewport, it is straightforward to split the screen for multiplayer games:

```D
// player 1 (left side of the screen)
player1View.viewport = FloatRect(0, 0, 0.5f, 1);

// player 2 (right side of the screen)
player2View.viewport = FloatRect(0.5f, 0, 0.5f, 1);
```

![A Split Screen](http://www.sfml-dev.org/tutorials/2.3/images/graphics-view-split-screen.png "A Split Screen")

... or a mini-map:

```D
// the game view (full window)
gameView.viewport = FloatRect(0, 0, 1, 1);

// mini-map (upper-right corner)
minimapView.viewport = FloatRect(0.75f, 0, 0.25f, 0.25f);
```

![A Minimap](http://www.sfml-dev.org/tutorials/2.3/images/graphics-view-minimap.png "A Minimap")

Using a View
---

To draw something using a view, you must draw it after setting the `view` property of the target to which you are drawing ([RenderWindow](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/renderwindow.d) or [RenderTexture](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/rendertexture.d)).

```D
// let's define a view
View view = new View(FloatRect(0, 0, 1000, 600));

// activate it
window.view = view;

// draw something to that view
window.draw(some_sprite);

// want to do visibility checks? retrieve the view
View currentView = window.view;
...
```

The view remains active until you set another one. This means that there is always a view which defines what appears in the target, and where it is drawn. If you don't explicitly set any view, the render-target uses its own default view, which matches its size 1:1. You can get the default view of a render-target with the `getDefaultView()` method. This can be useful if you want to define your own view based on it, or restore it to draw fixed entities (like a GUI) on top of your scene.

```D
// create a view half the size of the default view
View view = window.getDefaultView().dup;
view.zoom(0.5f);
window.view = view;

// restore the default view
window.view = window.getDefaultView();
```

> When you use the `view` property, the render-target makes a copy of the view, and doesn't store a pointer to the one that is passed. This means that whenever you update your view, you need to set `view` again to apply the modifications. Don't be afraid to copy views or create them on the fly, they aren't expensive objects (they just hold a few floats).

Showing More When the Window is Resized
---

Since the default view never changes after the window is created, the viewed contents are always the same. So when the window is resized, everything is squeezed/stretched to the new size.

If, instead of this default behavior, you'd like to show more/less stuff depending on the new size of the window, all you have to do is update the size of the view with the size of the window.

```D
// the event loop
Event event;
while (window.pollEvent(event))
{
    ...

    // catch the resize events
    if (event.type == Event.EventType.Resized)
    {
        // update the view to the new size of the window
        FloatRect visibleArea = FloatRect(0, 0, event.size.width, event.size.height);
        window.view = new View(visibleArea);
    }
}
```

Coordinates conversions
---

When you use a custom view, or when you resize the window without using the code above, pixels displayed on the target no longer match units in the 2D world. For example, clicking on pixel (10, 50) may hit the point (26.5, -84) of your world. You end up having to use a conversion function to map your pixel coordinates to world coordinates: `mapPixelToCoords()`.

```D
// get the current mouse position in the window
Vector2i pixelPos = Mouse.getPosition(window);

// convert it to world coordinates
Vector2f worldPos = window.mapPixelToCoords(pixelPos);
```

By default, `mapPixelToCoords()` uses the current view. If you want to convert the coordinates using view which is not the active one, you can pass it as an additional argument to the function.

The opposite, converting world coordinates to pixel coordinates, is also possible with the `mapCoordsToPixel()` function.
