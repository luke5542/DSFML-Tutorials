Designing your own entities with vertex arrays
=====

Introduction
---

DSFML provides simple classes for the most common 2D entities. And while more complex entities can easily be created from these building blocks, it is not always the most efficient solution. For example, you'll quickly reach your graphics card's limits if you draw many individual sprites. The reason is that the performances directly depend on the number of calls to the draw function. Indeed, each call involves setting a set of OpenGL states, resetting matrices, changing textures, etc. And all this stuff only to draw two triangles (a sprite). This is really not what your graphics card expects: today's GPUs are designed to process large batches of triangles, typically several thousands or millions.

To fill this gap, DSFML provides a lower-level mechanism to draw things: vertex arrays. In fact, vertex arrays are used internally by all other DSFML classes. They allow a more flexible definition of 2D entities, containing as many triangles as you need. They even allow drawing points or lines.

What is a Vertex, and Why are They Always in Arrays?
---

A vertex is the smallest graphical entity that you can manipulate. In short, it is a graphical point: it contains of course a 2D position (x, y), but also a color, and a pair of texture coordinates. We'll see later the role of these attributes.

Vertices alone don't do much. They are always grouped in primitives: either points (1 vertex), lines (2 vertices), triangles (3 vertices) or quads (4 vertices). Then, you put one or more primitives together to create the final geometry of the entity.

Now you understand why we always talk about vertex arrays, and not vertices alone.

A Simple Vertex Array
---

First, let's have a look at the [Vertex](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/vertex.d) class. It's a simple container which contains three public members, and no function except constructors. These constructors allow to create vertices with only the attributes that you need -- you don't always need to color or texture your entity.

```D
// create a new vertex
auto vertex = Vertex();

// set its position
vertex.position = Vector2f(10, 50);

// set its color
vertex.color = Color.Red;

// set its texture coordinates
vertex.texCoords = Vector2f(100, 100);
```

... or, using the correct constructor:

```D
auto vertex = Vertex(Vector2f(10, 50), Color.Red, Vector2f(100, 100));
```

Now, let's define a primitive. Remember, a primitive is made of several vertices, therefore we need a vertex array. DSFML provides a simple wrapper for this: [VertexArray](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/vertexarray.d). It provides the semantics of an array (with a few differences where methods are used instead), and it also stores the primitive type directly.

```D
// create an array of 3 vertices that define a triangle primitive
auto triangle = VertexArray(PrimitiveType.Triangles, 3);

// define the position of the triangle's points
triangle[0].position = Vector2f(10, 10);
triangle[1].position = Vector2f(100, 10);
triangle[2].position = Vector2f(100, 100);

// define the color of the triangle's points
triangle[0].color = Color.Red;
triangle[1].color = Color.Blue;
triangle[2].color = Color.Green;

// no texture coordinates here, we'll see that later
```

Your triangle is ready, you can now draw it. Drawing a vertex array is similar to drawing other DSFML entities, with the draw function:

```D
window.draw(triangle);
```

![A Triangle Made With Vertices](http://www.sfml-dev.org/tutorials/2.0/images/graphics-vertex-array-triangle.png "A Triangle Made With Vertices")

You can see that the vertices' color is interpolated to fill the primitive; this is a nice way to create gradients.

Note that you don't have to use the [VertexArray](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/vertexarray.d) class. It's just defined for convenience, it's nothing more than a `Vertex[]` with a [PrimitiveType](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/primitivetype.d). If you need more flexibility, or a static array, you can use your own storage. You must then use the overload of the draw function which takes a pointer to the vertices, the vertex count and the primitive type.

```D
Vertex[] vertices;
vertices ~= Vertex(...);
...

window.draw(vertices, PrimitiveType.Triangles);

Vertex vertices[2] =
[
    Vertex(...),
    Vertex(...)
];

window.draw(vertices, PrimitiveType.Lines);
```

Primitive types
---

Let's stop for a while, and see which primitives you can create with vertices. As explained above, you can define the most basic 2D primitives: point, line, triangle and quad (actually, this one exists for convenience, internally the graphics card breaks it into two triangles). There are also "chained" variants of these primitive types, that allow to share vertices between two adjacent primitives -- which is really useful because primitives are often connected.

Let's have a look at the full list:

| Primitive Type | Description | Example |
| --- | --- | --- |
| Points | A set of unconnected points. These points have no thickness: they will always take one pixel, regardless of the current transform and view. | ![The Points primitive type](http://www.sfml-dev.org/tutorials/2.0/images/graphics-vertex-array-points.png) |
| Lines | A set of unconnected lines. These lines have no thickness: they will always be one pixel wide, regardless of the current transform and view. | ![The Lines primitive type](http://www.sfml-dev.org/tutorials/2.0/images/graphics-vertex-array-lines.png) |
| LinesStrip | A set of connected lines. The end vertex of one line is used as the start vertex of the next one. | ![The LinesStrip primitive type](http://www.sfml-dev.org/tutorials/2.0/images/graphics-vertex-array-lines-strip.png) |
| Triangles | A set of unconnected triangles. | ![The Triangles primitive type](http://www.sfml-dev.org/tutorials/2.0/images/graphics-vertex-array-triangles.png) |
| TrianglesStrip | A set of connected triangles. Each triangle shares its two last vertices with the next one. | ![The TrianglesStrip primitive type](http://www.sfml-dev.org/tutorials/2.0/images/graphics-vertex-array-triangles-strip.png) |
| TrianglesFan | A set of triangles connected to a central point. The first vertex is the center, then each new vertex defines a new triangle, using the center and the next vertex. | ![The TrianglesFan primitive type](http://www.sfml-dev.org/tutorials/2.0/images/graphics-vertex-array-triangles-fan.png) |
| Quads | A set of unconnected quads. The 4 points of each quad must be defined either in clockwise or counterclockwise order. | ![The Quads primitive type](http://www.sfml-dev.org/tutorials/2.0/images/graphics-vertex-array-quads.png) |

Texturing
---

Like other DSFML entities, vertex arrays can be textured. To do so, you'll need to play with the `texCoords` attribute of the vertices. This attribute defines which pixel of the texture is mapped to the vertex.

```D
// create a quad
auto quad = VertexArray(Quads, 4);

// define it as a rectangle, located at (10, 10) and with size 100x100
quad[0].position = Vector2f(10, 10);
quad[1].position = Vector2f(110, 10);
quad[2].position = Vector2f(110, 110);
quad[3].position = Vector2f(10, 110);

// define its texture area to be a 25x50 rectangle starting at (0, 0)
quad[0].texCoords = Vector2f(0, 0);
quad[1].texCoords = Vector2f(25, 0);
quad[2].texCoords = Vector2f(25, 50);
quad[3].texCoords = Vector2f(0, 50);
```

> Texture coordinates are defined in pixels (just like the `textureRect` of sprites and shapes). They are not normalized (between 0 and 1), as people who are used to OpenGL programming would expect.

Vertex arrays are low-level entities, they only deal with geometry and do not store additional attributes like a texture. To draw a vertex array with a texture, you must pass it directly to the draw function using a [RenderStates](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/renderstates.d) instance:

```D
auto vertices = VertexArray();
auto texture = Texture();

...

auto states = RenderStates(texture);

window.draw(vertices, states);
```

Transforming a Vertex Array
---

Transforming is similar to texturing. The transform is not stored in the vertex array, you must pass it to the draw function, again using [RenderStates](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/renderstates.d):

```D
auto vertices = VertexArray();
auto transform = Transform();

...

auto states = RenderStates(texture);

window.draw(vertices, states);
```

To know more about transformations and the [Transform](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/transform.d) class, you can read the [Transforming Entities](https://github.com/luke5542/DSFML-Tutorials/blob/master/transforms.md) tutorial.

Creating a DSFML-like Entity
---

Now that you know how to define your own textured/colored/transformed entity, wouldn't it be nice to wrap it in a DSFML-style class? Fortunately, DSFML makes this easy for you, with the [Drawable](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/drawable.d) and [Transformable](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/transformable.d) base interfaces. These two interfaces are the base of the built-in DSFML entities -- [Sprite](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/sprite.d), [Text](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/text.d) and [Shape](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/shape.d).

[Drawable](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/drawable.d) only declares one virtual function; no member nor concrete function. Inheriting from [Drawable](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/drawable.d) allows to draw instances of your class the same way as DSFML classes:

```
class MyEntity : Drawable
{
    override void draw(RenderTarget& target, RenderStates states)
    {
        ...
    }
}

MyEntity entity = ...;
window.draw(entity); // internally calls entity.draw
```

Note that doing this is not mandatory, you could just have a similar draw function in your class, and call it with `entity.draw(window)`. But the other way, by implementing [Drawable](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/drawable.d), is nicer and more consistent. And in case you need to store an array of drawable objects, you can do it without extra work since all drawable objects (DSFML's and yours) use the same known interface.

The other interface, [Transformable](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/transformable.d), shouldn't be implemented directly; that functionality is encapsulated for you with the `NormalTransformable` mixin. Including this mixin automatically adds to your class the same transformation functions and properties as other DSFML classes (position, rotation, scale, ...). You can learn more about this class in the [Transforming Entities](https://github.com/luke5542/DSFML-Tutorials/blob/master/transforms.md) tutorial.

So, using these two interfaces and a vertex array (in this example we'll also add a texture), here is what a typical DSFML-like graphical class would look like:

```D
class MyEntity : Drawable, Transformable
{

    // add functions to play with the entity's geometry / colors / texturing...

    override void draw(RenderTarget target, RenderStates states)
    {
        // apply the entity's transform -- combine it with the one that was passed by the caller
        states.transform *= getTransform(); // getTransform() is defined by Transformable

        // apply the texture
        states.texture = m_texture;

        // you may also override states.shader or states.blendMode if you want

        // draw the vertex array
        target.draw(m_vertices, states);
    }

    private 
    {
        VertexArray m_vertices;
        Texture m_texture;
    }
}
```

You can then play with this class as if it was a built-in SFML class:

```D
auto entity = MyEntity();

// you can transform it
entity.position = Vector2f(10, 50);
entity.rotation = Vector2f(45);

// you can draw it
window.draw(entity);
```

Example: Tile Map
---

With what we've seen above, let's create a class that encapsulates a tile map. The whole map will be contained in a single vertex array, therefore it will be super fast to draw. Note that we can apply this strategy only if the whole tile set can fit into a single texture. Otherwise, we would have to use at least one vertex array per texture.

```D
class TileMap : Drawable, Transformable
{
    private
    {
        VertexArray m_vertices;
        Texture m_tileset;
    }

    this()
    {
        m_tileSet = Texture();
    }

    bool load(const(string) tileset, Vector2u tileSize, const(int) tiles, uint width, uint height)
    {
        // load the tileset texture
        if (!m_tileset.loadFromFile(tileset))
            return false;

        // resize the vertex array to fit the level size
        m_vertices = VertexArray(PrimitiveType.Quads, width * height * 4);

        // populate the vertex array, with one quad per tile
        for (uint i = 0; i < width; ++i)
            for (uint j = 0; j < height; ++j)
            {
                // get the current tile number
                int tileNumber = tiles[i + j * width];

                // find its position in the tileset texture
                int tu = tileNumber % (m_tileset.size.x / tileSize.x);
                int tv = tileNumber / (m_tileset.size.x / tileSize.x);

                // get a pointer to the current tile's quad
                Vertex quad = m_vertices[(i + j * width) * 4];

                // define its 4 corners
                quad[0].position = Vector2f(i * tileSize.x, j * tileSize.y);
                quad[1].position = Vector2f((i + 1) * tileSize.x, j * tileSize.y);
                quad[2].position = Vector2f((i + 1) * tileSize.x, (j + 1) * tileSize.y);
                quad[3].position = Vector2f(i * tileSize.x, (j + 1) * tileSize.y);

                // define its 4 texture coordinates
                quad[0].texCoords = Vector2f(tu * tileSize.x, tv * tileSize.y);
                quad[1].texCoords = Vector2f((tu + 1) * tileSize.x, tv * tileSize.y);
                quad[2].texCoords = Vector2f((tu + 1) * tileSize.x, (tv + 1) * tileSize.y);
                quad[3].texCoords = Vector2f(tu * tileSize.x, (tv + 1) * tileSize.y);
            }

        return true;
    }

    override void draw(RenderTarget& target, RenderStates states)
    {
        // apply the transform
        states.transform *= getTransform();

        // apply the tileset texture
        states.texture = m_tileset;

        // draw the vertex array
        target.draw(m_vertices, states);
    }

}
```

And now, the application that uses it:

```D
void main()
{
    // create the window
    RenderWindow window = RenderWindow(VideoMode(512, 256), "Tilemap");

    // define the level with an array of tile indices
    const(int) level[] =
    {
        0, 0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,
        0, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 2, 0, 0, 0, 0,
        1, 1, 0, 0, 0, 0, 0, 0, 3, 3, 3, 3, 3, 3, 3, 3,
        0, 1, 0, 0, 2, 0, 3, 3, 3, 0, 1, 1, 1, 0, 0, 0,
        0, 1, 1, 0, 3, 3, 3, 0, 0, 0, 1, 1, 1, 2, 0, 0,
        0, 0, 1, 0, 3, 0, 2, 2, 0, 0, 1, 1, 1, 1, 2, 0,
        2, 0, 1, 0, 3, 0, 2, 2, 2, 0, 1, 1, 1, 1, 1, 1,
        0, 0, 1, 0, 3, 2, 2, 2, 0, 0, 0, 0, 1, 1, 1, 1,
    };

    // create the tilemap from the level definition
    TileMap map = TileMap();
    if (!map.load("tileset.png", Vector2u(32, 32), level, 16, 8))
        exit(-1);

    // run the main loop
    while (window.isOpen())
    {
        // handle events
        Event event;
        while (window.pollEvent(event))
        {
            if(event.type == Event.EventType.Closed)
                window.close();
        }

        // draw the map
        window.clear();
        window.draw(map);
        window.display();
    }
}
```

![The Tilemap Example](http://www.sfml-dev.org/tutorials/2.0/images/graphics-vertex-array-tilemap.png "The Tilemap Example")

Example: Particle System
---

This second example implements another common entity: the particle system. This one is a very simple one, with no texture and as few parameters as possible. It demonstrates the use of the `Points` primitive type with a dynamic vertex array which changes every frame.

```D
class ParticleSystem : Drawable, Transformable
{

    private
    {
        Particle[] m_particles;
        VertexArray m_vertices;
        Time m_lifetime;
        Vector2f m_emitter;
    }


    this(unsigned int count)
    {
        m_particles = new Particle[count];
        m_vertices(PrimitiveType.Points, count);
        m_lifetime(seconds(3));
        m_emitter = Vector2f(0, 0);
    }

    void setEmitter(Vector2f position)
    {
        m_emitter = position;
    }

    void update(Time elapsed)
    {
        for (uint i = 0; i < m_particles.size(); ++i)
        {
            // update the particle lifetime
            Particle p = m_particles[i];
            p.lifetime -= elapsed;

            // if the particle is dead, respawn it
            if (p.lifetime <= Time.Zero)
                resetParticle(i);

            // update the position of the corresponding vertex
            m_vertices[i].position += p.velocity * elapsed.asSeconds();

            // update the alpha (transparency) of the particle according to its lifetime
            float ratio = p.lifetime.asSeconds() / m_lifetime.asSeconds();
            m_vertices[i].color.a = to!ubyte(ratio * 255);
        }
    }

    override void draw(RenderTarget target, RenderStates states)
    {
        // apply the transform
        states.transform *= getTransform();

        // our particles don't use a texture
        states.texture = null;

        // draw the vertex array
        target.draw(m_vertices, states);
    }

    void resetParticle(int index)
    {
        // give a random velocity and lifetime to the particle
        float angle = (uniform!(uint)() % 360) * 3.14f / 180.f;
        float speed = (uniform!(uint)() % 50) + 50.f;
        m_particles[index].velocity = Vector2f(cos(angle) * speed, sin(angle) * speed);
        m_particles[index].lifetime = milliseconds((uniform!(uint)() % 2000) + 1000);

        // reset the position of the corresponding vertex
        m_vertices[index].position = m_emitter;
    }
}


struct Particle
{
    Vector2f velocity;
    Time lifetime;
}
```

And the little demo that uses it:

void main()
{
    // create the window
    auto window = RenderWindow(VideoMode(512, 256), "Particles");

    // create the particle system
    auto particles = ParticleSystem(1000);

    // create a clock to track the elapsed time
    Clock clock = new Clock();

    // run the main loop
    while (window.isOpen())
    {
        // handle events
        Event event;
        while (window.pollEvent(event))
        {
            if(event.type == Event.EventType.Closed)
                window.close();
        }

        // make the particle system emitter follow the mouse
        Vector2i mouse = Mouse.getPosition(window);
        particles.setEmitter(window.mapPixelToCoords(mouse));

        // update it
        Time elapsed = clock.restart();
        particles.update(elapsed);

        // draw it
        window.clear();
        window.draw(particles);
        window.display();
    }

}

![The Particles Example](http://www.sfml-dev.org/tutorials/2.0/images/graphics-vertex-array-particles.png "The Particles Example")
