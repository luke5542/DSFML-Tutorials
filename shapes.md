Shapes
=====

Introduction
---

DSFML provides a set of classes that represent simple shape entities. Each type of shape is a separate class, but they all derive from the same base class so that they provide the same subset of common features. Each class then adds its own specificities: a radius property for the circle class, a size for the rectangle class, points for the polygon class, etc.

Common Shape Properties
---

Transformation (Position, Rotation, Scale)
-

These properties are common to all the DSFML graphical classes, so they are explained in a separate tutorial: [Transforming Entities](https://github.com/luke5542/DSFML-Tutorials/blob/master/transforms.md).

Color
-

One of the most basic property of a shape is its color, that you can change with the `setFillColor` function.

```
auto shape = new CircleShape(50);

// set the shape color to green
shape.fillColor = new Color(100, 250, 50);
```

![A Colored Shape](http://www.sfml-dev.org/tutorials/2.0/images/graphics-shape-color.png "A Colored Shape")

Outline
-

Shapes can have an outline. You can select the thickness and color of the outline with the `setOutlineThickness` and `setOutlineColor` functions.

```
auto shape = new CircleShape(50);
shape.fillColor = new Color(100, 250, 50);

// set a 10-pixel wide orange outline
shape.outlineThickness = 10;
shape.outlineColor = new Color(250, 150, 100);
```

![An Outlined Shape](http://www.sfml-dev.org/tutorials/2.0/images/graphics-shape-color.png "An Outlined Shape")

By default, the outline expands outside the shape (if you have a circle with a radius of 10 and an outline thickness of 5, the total radius of the circle will be 15). You can make it expand towards the center of the shape instead, by giving a negative thickness.

To disable the outline, set its thickness to 0. If you want the outline only, you can set the fill color to sf::Color::Transparent.
Texture

Shapes can also be textured, like sprites. To specify which part of the texture must be mapped to the shape, you must use the setTextureRect function. It takes the texture rectangle to map to the bounding rectangle of the shape. This method doesn't offer maximum flexibility, but it is much easier to use than individually setting the texture coordinates of each point of the shape.

sf::CircleShape shape(50);

// map a 100x100 textured rectangle to the shape
shape.setTexture(&texture); // texture is a sf::Texture
shape.setTextureRect(sf::IntRect(10, 10, 100, 100));

A textured shape

Note that the outline is not textured.
In case the shape has a fill color, the texture is modulated (multiplied) with it.
To disable texturing, call setTexture(NULL).
Drawing a shape

Drawing a shape is as simple as drawing any other SFML entity:

window.draw(shape);

Built-in shape types
Rectangles

To draw rectangles, you must use the sf::RectangleShape class. It has only one attribute: the size of the rectangle.

// define a 120x50 rectangle
sf::RectangleShape rectangle(sf::Vector2f(120, 50));

// change the size to 100x100
rectangle.setSize(sf::Vector2f(100, 100));

A rectangle shape
Circles

Circles are represented by the sf::CircleShape class. It has two attributes: the radius and the number of sides. The number of sides is an optional attribute, it allows you to adjust the "quality" of the circle: circles have to be simulated by polygons with many sides (the graphics card is unable to draw a perfect circle directly), and this attribute defines how many sides your circle will have. If you draw small circles, you'll probably need only a few sides. If you draw big circles, or zoom on regular circles, you'll most likely need more sides.

// define a circle with radius = 200
sf::CircleShape circle(200);

// change the radius to 40
circle.setRadius(40);

// change the number of sides (points) to 100
circle.setPointCount(100);

A circle shape
Regular polygons

There's no dedicated class for regular polygons, in fact you can get a regular polygon of any number of sides with the sf::CircleShape class: indeed, since circles are simulated by polygons with many sides, you just have to play with the number of sides to get the desired polygons. A sf::CircleShape with 3 points is a triangle, with 4 points it's a square, etc.

// define a triangle
sf::CircleShape triangle(80, 3);

// define a square
sf::CircleShape square(80, 4);

// define an octagon
sf::CircleShape octagon(80, 8);

Regular polygons
Convex shapes

The sf::ConvexShape class is the ultimate shape class: it allows you to define any shape, as long as it stays convex. Indeed, SFML is unable to draw concave shapes; if you need to draw a concave shape, you'll have to split it into multiple convex polygons (if possible).

To define a convex shape, you must first set the total number of points, and then define these points.

// create an empty shape
sf::ConvexShape convex;

// resize it to 5 points
convex.setPointCount(5);

// define the points
convex.setPoint(0, sf::Vector2f(0, 0));
convex.setPoint(1, sf::Vector2f(150, 10));
convex.setPoint(2, sf::Vector2f(120, 90));
convex.setPoint(3, sf::Vector2f(30, 100));
convex.setPoint(4, sf::Vector2f(0, 50));

It is very important to define the points of a convex shape either in clockwise or anticlockwise order. If you define them in a random order, the result will be undefined.
A convex shape

Officially, sf::ConvexShape can only create convex shapes. But in fact, its requirements are a little more relaxed. In fact, the only technical constraint that your shape must follow, is that if you draw a line from its center of gravity to any of its point, you mustn't cross an edge. With this relaxed definition, you can for example draw stars.
Lines

There's no shape class for lines. The reason is simple: if your line has a thickness, it is a rectangle; if it doesn't, it can be drawn with a line primitive.

Line with thickness:

sf::RectangleShape line(sf::Vector2f(150, 5));
line.rotate(45);

A line shape drawn as a rectangle

Line without thickness:

sf::Vertex line[] =
{
    sf::Vertex(sf::Vector2f(10, 10)),
    sf::Vertex(sf::Vector2f(150, 150))
};

window.draw(line, 2, sf::Lines);

A line shape drawn as a primitive

To learn more about vertices and primitives, you can read the Vertex arrays tutorial.
Custom shape types

You can extend the set of shape classes with your own shape types. To do so, you must derive from sf::Shape and override two functions:

    getPointCount: return the number of points of the shape
    getPoint: return a point of the shape

You must also call the update() protected function whenever the points of your shape change, so that the base class knows about it and can update its internal state.

Here is a complete example of a custom shape class: EllipseShape.

class EllipseShape : public sf::Shape
{
public :

    explicit EllipseShape(const sf::Vector2f& radius = sf::Vector2f(0, 0)) :
    m_radius(radius)
    {
        update();
    }

    void setRadius(const sf::Vector2f& radius)
    {
        m_radius = radius;
        update();
    }

    const sf::Vector2f& getRadius() const
    {
        return m_radius;
    }

    virtual unsigned int getPointCount() const
    {
        return 30; // fixed, but could be an attribute of the class if needed
    }

    virtual sf::Vector2f getPoint(unsigned int index) const
    {
        static const float pi = 3.141592654f;

        float angle = index * 2 * pi / getPointCount() - pi / 2;
        float x = std::cos(angle) * m_radius.x;
        float y = std::sin(angle) * m_radius.y;

        return sf::Vector2f(m_radius.x + x, m_radius.y + y);
    }

private :

    sf::Vector2f m_radius;
};

An ellipse shape
Antialiased shapes

There's no option to antialias a single shape. If you want to get antialiased shapes (ie. shapes with smooth edges), you must enable antialiasing globally when you create the window, with the corresponding attribute of the sf::ContextSettings structure.

sf::ContextSettings settings;
settings.antialiasingLevel = 8;

sf::RenderWindow window(sf::VideoMode(800, 600), "SFML shapes", sf::Style::Default, settings);

Aliased vs antialiased shape

Remember that antialiasing depends on the graphics card: it may either not support it, or have it forced to off in the driver settings.
