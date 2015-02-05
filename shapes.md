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

```D
auto shape = new CircleShape(50);

// set the shape color to green
shape.fillColor = Color(100, 250, 50);
```

![A Colored Shape](http://www.sfml-dev.org/tutorials/2.0/images/graphics-shape-color.png "A Colored Shape")

Outline
-

Shapes can have an outline. You can select the thickness and color of the outline with the `setOutlineThickness` and `setOutlineColor` functions.

```D
auto shape = new CircleShape(50);
shape.fillColor = Color(100, 250, 50);

// set a 10-pixel wide orange outline
shape.outlineThickness = 10;
shape.outlineColor = Color(250, 150, 100);
```

![An Outlined Shape](http://www.sfml-dev.org/tutorials/2.0/images/graphics-shape-color.png "An Outlined Shape")

By default, the outline expands outside the shape (if you have a circle with a radius of 10 and an outline thickness of 5, the total radius of the circle will be 15). You can make it expand towards the center of the shape instead, by giving a negative thickness.

To disable the outline, set its thickness to 0. If you want the outline only, you can set the fill color to `Color.Transparent`.

Texture
-

Shapes can also be textured, like sprites. To specify which part of the texture must be mapped to the shape, you must specify the texture rectangle with the `textureRect` property. It takes the texture rectangle to map to the bounding rectangle of the shape. This method doesn't offer maximum flexibility, but it is much easier to use than individually setting the texture coordinates of each point of the shape.

```D
CircleShape shape = new CircleShape(50);

// map a 100x100 textured rectangle to the shape
shape.setTexture(texture); // texture is a Texture
shape.textureRect = IntRect(10, 10, 100, 100);
```

![A Textured Shape](http://www.sfml-dev.org/tutorials/2.0/images/graphics-shape-texture.png "A Textured Shape")

Note that the outline is not textured.
In case the shape has a fill color, the texture is modulated (multiplied) with it.
To disable texturing, call `setTexture(null)`.

Drawing a shape
---

Drawing a shape is as simple as drawing any other DSFML entity:

```D
window.draw(shape);
```

Built-in shape types
---

Rectangles
-

To draw rectangles, you must use the [RectangleShape](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/rectangleshape.d) class. It has only one attribute: the size of the rectangle.

```D
// define a 120x50 rectangle
RectangleShape rectangle = new RectangleShape(new Vector2f(120, 50));

// change the size to 100x100
rectangle.size = Vector2f(100, 100);
```

![A Rectangle Shape](http://www.sfml-dev.org/tutorials/2.0/images/graphics-shape-rectangle.png "A Rectangle Shape")

Circles
-

Circles are represented by the [CircleShape](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/circleshape.d) class. It has two attributes: the radius and the number of sides. The number of sides is an optional attribute, it allows you to adjust the "quality" of the circle: circles have to be simulated by polygons with many sides (the graphics card is unable to draw a perfect circle directly), and this attribute defines how many sides your circle will have. If you draw small circles, you'll probably need only a few sides. If you draw big circles, or zoom on regular circles, you'll most likely need more sides.

```D
// define a circle with radius = 200
CircleShape circle = new CircleShape(200);

// change the radius to 40
circle.radius = 40;

// change the number of sides (points) to 100
circle.pointCount = 100;
```

![A Circle Shape](http://www.sfml-dev.org/tutorials/2.0/images/graphics-shape-circle.png "A Circle Shape")

Regular polygons
-

There's no dedicated class for regular polygons, in fact you can get a regular polygon of any number of sides with the [CircleShape](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/circleshape.d) class: indeed, since circles are simulated by polygons with many sides, you just have to play with the number of sides to get the desired polygons. A [CircleShape](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/circleshape.d) with 3 points is a triangle, with 4 points it's a square, etc.

```D
// define a triangle
CircleShape triangle = new CircleShape(80, 3);

// define a square
CircleShape square = new CircleShape(80, 4);

// define an octagon
CircleShape octagon = new CircleShape(80, 8);
```

![Regular Polygons](http://www.sfml-dev.org/tutorials/2.0/images/graphics-shape-regular.png "Regular Polygons")

Convex shapes
-

The [ConvexShape](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/convexshape.d) class is the ultimate shape class: it allows you to define any shape, as long as it stays convex. Indeed, DSFML is unable to draw concave shapes; if you need to draw a concave shape, you'll have to split it into multiple convex polygons (if possible).

To define a convex shape, you must first set the total number of points, and then define these points.

```D
// create an empty shape
ConvexShape convex = new ConvexShape();

// resize it to 5 points
convex.pointCount = 5;

// define the points
convex.setPoint(0, new Vector2f(0, 0));
convex.setPoint(1, new Vector2f(150, 10));
convex.setPoint(2, new Vector2f(120, 90));
convex.setPoint(3, new Vector2f(30, 100));
convex.setPoint(4, new Vector2f(0, 50));
```

> It is very important to define the points of a convex shape either in clockwise or anticlockwise order. If you define them in a random order, the result will be undefined.

![A Convex Shape](http://www.sfml-dev.org/tutorials/2.0/images/graphics-shape-convex.png "A Convex Shape")

Officially, [ConvexShape](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/convexshape.d) can only create convex shapes. But in fact, its requirements are a little more relaxed. In fact, the only technical constraint that your shape must follow, is that if you draw a line from its center of gravity to any of its point, you mustn't cross an edge. With this relaxed definition, you can for example draw stars.

Lines
-

There's no shape class for lines. The reason is simple: if your line has a thickness, it is a rectangle; if it doesn't, it can be drawn with a line primitive.

Line with thickness:

```D
RectangleShape line = new RectangleShape(Vector2f(150, 5));
line.rotate(45);
```

![A Line Shape Drawn as a Rectangle](http://www.sfml-dev.org/tutorials/2.0/images/graphics-shape-line-rectangle.png "A Line Shape Drawn as a Rectangle")

Line without thickness:

```D
Vertex[2] line =
[
    Vertex(Vector2f(10, 10)),
    Vertex(Vector2f(150, 150))
];

window.draw(line, 2, PrimitiveType.Lines);
```

![A Line Shape Drawn as a Primitive](http://www.sfml-dev.org/tutorials/2.0/images/graphics-shape-line-primitive.png "A Line Shape Drawn as a Primitive")

To learn more about vertices and primitives, you can read the [Vertex Arrays](https://github.com/luke5542/DSFML-Tutorials/blob/master/vertex-arrays.md) tutorial.

Custom shape types
---

You can extend the set of shape classes with your own shape types. To do so, you must derive from [Shape](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/shape.d) and override two functions:

+ `getPointCount`: return the number of points of the shape
+ `getPoint`: return a point of the shape

You must also call the `update` protected function whenever the points of your shape change, so that the base class knows about it and can update its internal state.

Here is a complete example of a custom shape class: EllipseShape.

```D
class EllipseShape : Shape
{

    private {
        Vector2f m_radius;
    }

    public :

        this(Vector2f radius = Vector2f(0, 0)) {
            m_radius = radius;
            update();
        }

        @property
        {
            Vector2f radius(Vector2f newRadius)
            {
                m_radius = newRadius;
                update();
                return newRadius;
            }

            Vector2f radius()
            {
                return m_radius;
            }
        }

        override uint getPointCount()
        {
            return 30; // fixed, but could be an attribute of the class if needed
        }

        override Vector2f getPoint(uint index) const
        {

            float angle = index * 2 * PI / getPointCount() - PI / 2;
            float x = cos(angle) * m_radius.x;
            float y = sin(angle) * m_radius.y;

            return Vector2f(m_radius.x + x, m_radius.y + y);
        }

}
```

![An Ellipse Shape](http://www.sfml-dev.org/tutorials/2.0/images/graphics-shape-ellipse.png "An Ellipse Shape")

Antialiased Shapes
---

There's no option to antialias a single shape. If you want to get antialiased shapes (ie. shapes with smooth edges), you must enable antialiasing globally when you create the window, with the corresponding attribute of the [ContextSettings](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/window/contextsettings.d) structure.

```D
ContextSettings settings = new ContextSettings();
settings.antialiasingLevel = 8;

RenderWindow window(VideoMode(800, 600), "SFML shapes", Style.Default, settings);
```

![Aliased vs Antialiased Shape](http://www.sfml-dev.org/tutorials/2.0/images/graphics-shape-antialiasing.png "Aliased vs Antialiased Shape")

Remember that antialiasing depends on the graphics card: it may either not support it, or have it forced to off in the driver settings.
