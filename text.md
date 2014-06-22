
Text and Fonts
=====

Loading a Font
---

Before drawing text, you need a character font -- like in any other program that prints text. Fonts are encapsulated in the [Font](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/font.d) class, which provides three main features: loading a font, getting glyphs (i.e. visual characters) from it, and reading its attributes. In a regular program, you'll only deal with the first feature, loading the font. So let's focus on this first.

The most common way of loading a font is from a file on disk, which is done with the `loadFromFile` function.

```D
Font font = new Font();
if (!font.loadFromFile("arial.ttf"))
{
    // error...
}
```

Note that DSFML won't load your system fonts automatically, i.e. `font.loadFromFile("Courier New")` won't work. First because DSFML requires file names, not font names, and secondly because DSFML doesn't have a magic access to your system's font folder. So, if you want to load a font, you need to have the font file with your application, like other resources (images, sounds, ...).

> The `loadFromFile` function sometimes fails with no obvious reason. First, check the error message printed by DSFML in the standard output (check the console). If the message is "unable to open file", make sure that the working directory (which is the directory any file path will be interpreted relatively to) is what you think it is: when you run the application from the explorer, the working directory is the executable folder, but when you launch your program from your IDE (Visual Studio, Code::Blocks, ...) the working directory is sometimes set to the project directory instead. This can generally be changed easily in the project settings.

You can also load a font file from memory (`loadFromMemory`), or from a custom input stream (`loadFromStream`).

DSFML supports the most common font formats. The full list is available in the API documentation.

That's all. Once your font is loaded, you can start drawing text.

Drawing Text
---

To draw text, you need to use the Text class. It's very simple to use:

```D
Text text = new Text();

// select the font
text.setFont(font); // font is a Font

// set the string to display
text.setString("Hello world");

// set the character size
text.setCharacterSize(24); // in pixels, not points!

// set the color
text.setColor(Color.Red);

// set the text style
text.setStyle(Text.Bold | Text.Underlined);

...

// inside the main loop, between window.clear() and window.display()
window.draw(text);
```

![Drawing Text](http://www.sfml-dev.org/tutorials/2.0/images/graphics-text-draw.png "Drawing Text")

Texts can also be transformed: they have a position, an orientation and a scale. The functions involved are the same as for the [Sprite](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/sprite.d) class and other DSFML entities, and are explained in the [Transforming Entities](https://github.com/luke5542/DSFML-Tutorials/blob/master/transforms.md) tutorial.

Making your own text class
---

If Text is too limited, or if you want to do something else with pre-rendered glyphs, [Font](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/font.d) provides everything that you need.

First, you can retrieve the texture which contains all the pre-rendered glyphs of a certain size:

```D
const(Texture) texture = font.getTexture(characterSize);
```

Note that glyphs are added to the texture when they are requested. There are so many characters (remember, more than 100000) that they can't all be generated when you load the font. Instead, they are rendered on the fly when you call the `getGlyph` function (see below).

To make something useful of the glyph texture, you must then get the texture coordinates of glyphs that are contained in it:

```D
Glyph glyph = font.getGlyph(character, characterSize, bold);
```

`character` is the `dchar` (UTF-32 code) of the glyph that you want to get. You must also specify the character size, and whether you want the bold or the regular version of the glyph.

The Glyph structure contains three members:

+ textureRect is the texture coordinates of the glyph within the texture
+ bounds is the bounding rectangle of the glyph, which helps to position it relatively to the base line of the text
+ advance is the horizontal offset to apply to get the start position of the next glyph in the text

Finally, you can get some other metrics of the font, such as the line height or the kerning (always for a certain character size):

```D
int lineHeight = font.getLineHeight(characterSize);

int kerning = font.getKerning(character1, character2, characterSize);
```
