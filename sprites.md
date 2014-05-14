Sprites and Textures
=====

Vocabulary
---

Most (if not all) of you are already familiar with these two very common objects, so let's define them very briefly.

A texture is an image. But we call it "texture" because it has a very specific role: being mapped to a 2D entity.

A sprite is nothing more than a textured rectangle.

![Rectangular Entity + Texture = Sprite](http://www.sfml-dev.org/tutorials/2.0/images/graphics-sprites-definition.png "Rectangular Entity + Texture = Sprite")

Ok, that was short but if you really don't understand what sprites and textures are, then you'll find a much better description on wikipedia.

Loading a Texture
---

So, before creating any sprite, we need a valid texture. The class that encapsulates textures in SFML is, surprisingly, Texture. Since the (only) role of a texture is to be loaded and mapped to graphical entities, almost all its functions are about loading and updating it.

The most common way of loading a texture is from an image file on disk, which is done with the loadFromFile function.

```
Texture texture = new Texture();
if (!texture.loadFromFile("image.png"))
{
    // error...
}
```

> The `loadFromFile` method sometimes fails with no obvious reason. First, check the error message printed by DSFML in the standard output (check the console). If the message is "unable to open file", make sure that the working directory (which is the directory any file path will be interpreted relatively to) is what you think it is: when you run the application from the explorer, the working directory is the executable folder, but when you launch your program from your IDE (Visual Studio, Code.Blocks, ...) the working directory is sometimes set to the project directory instead. This can generally be changed easily in the project settings.

You can also load an image file from memory (`loadFromMemory`), from a custom input stream (`loadFromStream`), or from an already loaded image (`loadFromImage`). The latter loads the texture from a [Image](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/image.d), which is a utility class that helps to manipulate images (modify pixels, create transparency channel, etc.). The pixels of an [Image](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/image.d) stay in system memory, which ensures that operations on them will be as fast as possible, as opposed to the pixels of a texture which reside in video memory and are therefore slow to retrieve or update -- but very fast to draw.

DSFML supports the most common file formats. The full list is available in the API documentation.

All these loading functions have an optional argument, which can be used if you want to load a smaller part of the image.

```
// load a 32x32 rectangle that starts at (10, 10)
if (!texture.loadFromFile("image.png", IntRect(10, 10, 32, 32)))
{
    // error...
}
```

The `IntRect` struct is a simple utility type that represents a rectangle, and is merely an alias for `Rect!(int)`. Its constructor takes the coordinates of the left-top corner, and the size of the rectangle.

If you don't want to load a texture from an image, but rather want to update it directly from an array of pixels, you can create it empty and update it later:

```
// create an empty 200x200 texture
if (!texture.create(200, 200))
{
    // error...
}
```

Note that the contents of the texture are undefined at this point.

To update the pixels of an existing texture, you must use the update function. It has overloads for many sources:

```
// update a texture from an array of pixels
ubyte[] pixels pixels = new ubyte[width * height * 4]; // * 4 because pixels have 4 components (RGBA)
...
texture.update(pixels);

// update a texture from a Image
Image image = new Image();
...
texture.update(image);

// update the texture from the current contents of the window
RenderWindow window;
...
texture.update(window);
```

These examples all assume that the source has the same size as the texture. If this is not the case, i.e. if you want to update only a part of the texture, then you can specify the coordinates of the sub-rectangle that you want to update. You can refer to the documentation for more details.

Additionally, a texture has two properties that change how it is rendered.

The first property allows one to smooth the texture. Smoothing a texture makes its pixels less visible (but a little more blurry), which can be important if it is scaled.

```
texture.setSmooth(true);
```

![Smooth vs Not Smooth](http://www.sfml-dev.org/tutorials/2.0/images/graphics-sprites-smooth.png "Smooth vs Not Smooth")


Since smoothing interpolates adjacent pixels in the texture, it can have the unwanted side effect of showing pixels outside the selected texture area. This can happen when your sprite is located at non-integer coordinates.

The second property allows one to repeat a texture within a single sprite.

```
texture.setRepeated(true);
```

![Repeated vs not repeated](http://www.sfml-dev.org/tutorials/2.0/images/graphics-sprites-repeated.png "Repeated vs not repeated")

This works only if your sprite is configured to show a rectangle which is bigger than the texture. Otherwise this property has no effect.

Ok, Can I Have My Sprite Now?
---

Yes, you can now create your sprite.

```
Sprite sprite = new Sprite();
sprite.setTexture(texture);

... and finally draw it.

// inside the main loop, between window.clear() and window.display()
window.draw(sprite);
```

If you don't want your sprite to show the full texture, you can set its texture rectangle.

```
sprite.setTextureRect(IntRect(10, 10, 32, 32));
```

You can also change the color of a sprite. The color that you set is modulated (multiplied) with the texture of the sprite. This can also be used to change the global transparency (alpha) of the sprite.

```
sprite.setColor(Color(0, 255, 0)); // green
sprite.setColor(Color(255, 255, 255, 128)); // half transparent
```

These sprites all use the same texture, but have a different color:

![Coloring Sprites](http://www.sfml-dev.org/tutorials/2.0/images/graphics-sprites-color.png "Coloring Sprites")

Sprites can also be transformed: they have a position, an orientation and a scale.

```
// position
sprite.position(Vector2f(10, 50)); // absolute position

// rotation
sprite.rotation(90); // absolute angle
sprite.rotation(sprite.rotation() + 15); // offset relative to the current angle

// scale
sprite.scale(Vector2f(0.5f, 2f)); // absolute scale factor
```

By default, the origin for these three transformations is the top-left corner of the sprite. If you want to set the origin to a different point (for example the center of the sprite, or another corner), you can use the `setOrigin` function.

```
sprite.origin(Vector2f(25, 25));
```

Since transformation functions are common to all DSFML entities, they are explained in a separate tutorial: [Transforming Entities](https://github.com/luke5542/DSFML-Tutorials/blob/master/transforms.md).

The White Square Problem
---

You successfully loaded a texture, correctly defined a sprite, and... all you see on screen now is a white square. What happened?

This is a common mistake. When you set the texture of a sprite, all it does internally is to keep a pointer to the texture instance. Therefore, if the texture is destroyed or moves elsewhere in memory, the sprite ends up with an invalid texture pointer.

You must correctly manage the lifetime of your textures, so that they live as long as they are used by sprites.

The Importance of Using as few Textures as Possible
---

Using as few textures as possible is a good strategy, and the reason is simple: changing the current texture is an expensive operation for the graphics card. Drawing many sprites that use the same texture will give you the best performances.

Additionally, using a single texture allows you to group static geometry into a single entity (you can only use one texture per draw call), which will be much faster to draw than a set of many entities. Batching static geometry involves other classes and is therefore beyond the scope of this tutorial, for more details see the [Vertex Array Tutorial](https://github.com/luke5542/DSFML-Tutorials/blob/master/vertex-arrays.md).

So, keep this in mind when you create your animation sheets or your tilesets: use a single texture if possible.

Using Texture With OpenGL Code
---

**TO BE COMPLETED**