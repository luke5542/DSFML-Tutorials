Drawing 2D stuff
=====

Introduction
---

As you learnt in the previous tutorials, the window module of DSFML provides an easy way to open an OpenGL window and handle its events, but it doesn't help when it comes to drawing something. The only option which is left to you is to use the powerful-yet-complex-and-low-level OpenGL API.

Fortunately, DSFML provides a graphics module which will help you draw 2D entities in a simpler way than with OpenGL.

The Drawing Window
---

To draw the entities provided by the graphics module, you must use a specialized window class: [RenderWindow](http://dsfml.com/dsfml/graphics/renderwindow.html). This class derives from [Window](http://dsfml.com/dsfml/window/window.html), and inherits all its functions. Everything that you've learnt about [Window](http://dsfml.com/dsfml/window/window.html) (creation, event handling, controlling the framerate, mixing with OpenGL, etc.) is applicable to [RenderWindow](http://dsfml.com/dsfml/graphics/renderwindow.html).

On top of that, [RenderWindow](http://dsfml.com/dsfml/graphics/renderwindow.html) adds high-level functions to help you draw things easily. In this tutorial we'll focus on two of these functions: `clear()` and `draw()`. They are as simple as their name: `clear()` clears the whole window with the chosen color, and `draw()` draws whatever object you give to it.

Here is what a typical main loop looks like with a render window:

```D
import dsfml.graphics;

void main()
{
    // create the window
    auto window = new RenderWindow(VideoMode(800, 600), "My window");

    // run the program as long as the window is open
    while (window.isOpen())
    {
        // check all the window's events that were triggered since the last iteration of the loop
        Event event;
        while (window.pollEvent(event))
        {
            // "close requested" event: we close the window
            if (event.type == Event.EventType.Closed)
                window.close();
        }

        // clear the window with black color
        window.clear(Color.Black);

        // draw everything here...
        // window.draw(...);

        // end the current frame
        window.display();
    }
}
```

Calling `clear()` before drawing anything is mandatory, otherwise you may keep undefined pixels from previous frames. The only exception is when you cover the entire window with what you draw, so that no pixel is left undrawn; in this case you can avoid calling `clear()` (although it won't make a big difference on performance).

Calling `display()` is also mandatory, it takes what was drawn since the last call to `display()` and displays it on the window. Indeed, things are not drawn directly to the window, but to a hidden buffer. This buffer is then copied to the window when you call `display()` -- this is called double-buffering.

> This `clear()`/`draw()`/`display()` cycle is the only good way to draw things. Don't try other strategies, such as keeping pixels from the previous frame, "erasing" pixels, or drawing once and calling `display()` multiple times. You'll get strange results due to double-buffering.
> Modern graphics chipsets and APIs are really made for repeated `clear()`/`draw()`/`display()` cycles where everything is completely refreshed at each iteration of the main loop. Don't be scared to draw 1000 sprites 60 times per second, you're far below the millions of triangles that your computer can handle.

What can I draw now?
---

Now that you have a main loop which is ready to draw, let's see what, and how, you can actually draw there.

DSFML provides four kinds of drawable entities: three of them are ready to be used (sprites, texts and shapes), the last one is the building block that will help you to create your own drawable entities (vertex arrays).

Although they share some common properties, these entities have their own specificities and deserve their own tutorials:

* [Sprite tutorial](https://github.com/luke5542/DSFML-Tutorials/blob/master/sprites.md)
* [Text tutorial](https://github.com/luke5542/DSFML-Tutorials/blob/master/text.md)
* [Shape tutorial](https://github.com/luke5542/DSFML-Tutorials/blob/master/shapes.md)
* [Vertex array tutorial](https://github.com/luke5542/DSFML-Tutorials/blob/master/vertex-arrays.md)

Off-screen drawing
---

DSFML also provides a way to draw to a texture instead of directly to a window. To do so, use a [RenderTexture](http://dsfml.com/dsfml/graphics/rendertexture.html) instead of a [RenderWindow](http://dsfml.com/dsfml/graphics/renderwindow.html). It has the same functions for drawing, inherited from their common base [RenderTarget](http://dsfml.com/dsfml/graphics/rendertarget.html).

```D
// create a 500x500 render-texture
RenderTexture renderTexture = new RenderTexture();
if (!renderTexture.create(500, 500))
{
    // error...
}

// drawing uses the same functions
renderTexture.clear();
renderTexture.draw(sprite); // or any other drawable
renderTexture.display();

// get the target texture (where the stuff has been drawn)
Texture texture = renderTexture.getTexture();

// draw it to the window
Sprite sprite = new Sprite(texture);
window.draw(sprite);
```

The `getTexture()` function returns a read-only texture, which means that you can only use it, not modify it. If you need to modify it before using it, you can copy it to your own [Texture](http://dsfml.com/dsfml/graphics/texture.html) instance.

[RenderTexture](http://dsfml.com/dsfml/graphics/rendertexture.html) also has the same functions as [RenderWindow](http://dsfml.com/dsfml/graphics/renderwindow.html) for handling views and OpenGL (corresponding tutorials to be done later). If you use OpenGL to draw to the render-texture, you can have a depth buffer by using the third optional argument of the create function.

```D
renderTexture.create(500, 500, true); // enable depth buffer
```

Drawing from Threads
---

**TO BE DONE LATER**
