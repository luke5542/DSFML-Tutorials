Position, rotation, scale: transforming entities
=====

Transforming SFML entities
---

All SFML classes (sprites, text, shapes) use the same interface for transformations: [Transformable](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/transformable.d). This base class defines a simple API to position, rotate, and scale your entities. It doesn't provide maximum flexibility, but it rather defines an interface which is easy to understand and to use, and which covers 99% of use cases -- for the other 1%, see the last chapters.

[Transformable](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/transformable.d) defines four properties: position, rotation, scale and origin; they are standard D `@property` fields. These transformation components are all independent from each other: if you want to change the orientation of the entity, you just have to set its rotation property, you don't have to care about the current position and scale.

Another note about [Transformable](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/graphics/transformable.d) that's specific to DSFML: all of the built-in transformable types (i.e. Sprite, Text, etc) use a `NormalTransformable` mixin that provides a standard implementation for classes that wish to implement the `Transformable` interface.

Position
--

The position is the... position of the entity in the 2D world. I don't think it deserves more explanations :)

```D
// 'entity' can be a Sprite, a Text, a Shape or any other transformable class

// set the absolute position of the entity
entity.position = Vector2f(10, 50);

// move the entity relatively to its current position
// This may change later...
entity.position = entity.position + Vector2f(5, 5);

// retrieve the absolute position of the entity
Vector2f position = entity.position; // = (15, 55)
```

![A Translated Entity](http://sfml-dev.org/tutorials/2.1/images/graphics-transform-position.png "A Translated Entity")

By default, entities are positioned relatively to their top-left corner; we'll see later how to change that with the 'origin' property.

Rotation
--

The rotation is the orientation of the entity in the 2D world. It is defined in degrees, in clockwise order (because the Y axis is pointing down in DSFML).

```D
// 'entity' can be a Sprite, a Text, a Shape or any other transformable class

// set the absolute rotation of the entity
entity.rotation = 45;

// rotate the entity relatively to its current orientation
// This may change later...
entity.rotation = entity.rotation + 10;

// retrieve the absolute rotation of the entity
float rotation = entity.rotation(); // = 55
```

![A Rotated Entity](http://sfml-dev.org/tutorials/2.1/images/graphics-transform-rotation.png "A Rotated Entity")

Note that DSFML always returns an angle in range [0, 360] when you call `rotation`.

Like for the position, the rotation is done around the top-left corner by default, but this can be changed with the origin.

Scale
--

The scale factor allows to resize the entity. The default scale is 1, less than 1 makes the entity smaller, more than 1 makes it bigger. Negative scales are also allowed, so that you can mirror the entity.

```D
// 'entity' can be a Sprite, a Text, a Shape or any other transformable class

// set the absolute scale of the entity
entity.scale = Vector2f(4.0f, 1.6f);

// scale the entity relatively to its current scale
// This may change later...
entity.scale = entity.scale + Vector2f(0.5f, 0.5f);

// retrieve the absolute scale of the entity
Vector2f scale = entity.scale; // = (2, 0.8)
```

![A Scaled Entity](http://sfml-dev.org/tutorials/2.1/images/graphics-transform-scale.png "A Scaled Entity")

Origin
--

The origin is the center point of the three other transformations. The position is the position of the origin, the rotation is made around the origin, and the scale is applied around the origin too. By default it is the top-left corner of the entity (point (0, 0)), but you can change it so that it is its center, or another corner for example.

To keep things simple, there's only one origin for the three transformation components. Which means that you can't, for example, position an entity relatively to its top-left corner while rotating it around its center. If you need to do such things, have a look at the next chapters.

```D
// 'entity' can be a Sprite, a Text, a Shape or any other transformable class

// set the origin of the entity
entity.origin = Vector2f(10, 20);

// retrieve the origin of the entity
Vector2f origin = entity.origin; // = (10, 20)
```

Note that changing the origin also changes the visual position of the entity, although its position property is still the same. If you don't understand why, read this tutorial one more time!

Transforming your own classes

sf::Transformable is not only made for SFML classes, it can also be a base (or member) of you own classes.

class MyGraphicalEntity : public sf::Transformable
{
    // ...
};

MyGraphicalEntity entity;
entity.setPosition(10, 30);
entity.setRotation(110);
entity.setScale(0.5f, 0.2f);
To make use of the final transform of the entity (most likely to draw it), call the getTransform function. This function returns a sf::Transform; see below for more explanations about it, and how to use it to transform an SFML entity.

If you don't need/want the complete set of functions provided by the sf::Transformable interface, don't hesitate to use it as a member instead, and to provide your own functions on top of it. It is not abstract so there's no problem to instanciate it instead of using it as a base class.

Custom transforms

The sf::Transformable class is easy to use, but it is also limited. Some users need more power, they need to specify a final transformation as a custom combination of individual transformations. For this kind of users, a lower-level class is available: sf::Transform. It is nothing more than a 3x3 matrix, so it can represent any transformation in the 2D spaaaaaace.

There are many ways to construct a sf::Transform:

by using the predefined functions for the most common transformations (translation, rotation, scale)
by combining two transforms
by specifying its 9 elements directly
Here are a few examples:

// the identity transform (does nothing)
sf::Transform t1 = sf::Transform::Identity;

// a rotation transform
sf::Transform t2;
t2.rotate(45);

// a custom matrix
sf::Transform t3(2, 0, 20,
                 0, 1, 50,
                 0, 0, 1);

// a combined transform
sf::Transform t4 = t1 * t2 * t3;
You can of course apply several predefined transformations to the same transform, they will all be combined sequentially:

sf::Transform t;
t.translate(10, 100);
t.rotate(90);
t.translate(-10, 50);
t.scale(0.5f, 0.75f);
Back to the point: how can a custom transform be applied to a graphical entity? It's easy: pass it to the draw function.

window.draw(entity, transform);
... which is in fact a shortcut for:

sf::RenderStates states;
states.transform = transform;
window.draw(entity, states);
If your entity is a sf::Transformable (sprite, text, shape), with its own internal transform, then both are combined to produce the final transform.

Bounding boxes

After transforming entities and drawing them, maybe you'd like to do some calculations with them, like checking collisions.

SFML entities can give you their bounding box. The bounding box is the minimal rectangle that contains the entity, with sides aligned on the X and Y axes.

Bounding box of entities
The bounding box is very useful to implement collision detection: it is very fast to check against a point or another axis-aligned rectangle, and it is close enough to the real entity to provide a good approximation.

// get the bounding box of the entity
sf::FloatRect boundingBox = entity.getGlobalBounds();

// check collision with a point
sf::Vector2f point = ...;
if (boundingBox.contains(point))
{
    // collision!
}

// check collision with another box (like the bounding box of another entity)
sf::FloatRect otherBox = ...;
if (boundingBox.intersects(otherBox))
{
    // collision!
}
The function is named getGlobalBounds because it returns the bounding box of the entity in the global coordinates system, ie. with all its transformations (position, rotation, scale) applied.

There's another function that returns the bounding box of the entity in its local coordinates system (without transformations applied): getLocalBounds. This function can be used to get the initial size of an entity, for example, or to perform more specific calculations.

Object hierarchies (scene graph)

With the custom transforms seen previously, it's now easy to implement a hierarchy of objects, where children are transformed relatively to their parent. All you have to do is to pass the combined transform from parent to children when you draw them, until you reach the final drawable entities (sprites, texts, shapes, vertex arrays or your own drawables).

// the abstract base class
class Node
{
public:

    // ... functions to transform the node

    // ... functions to manage the node's children

    void draw(sf::RenderTarget& target, const sf::Transform& parentTransform) const
    {
        // combine the parent transform with the node's one
        sf::Transform combinedTransform = parentTransform * m_transform;

        // let the node draw itself
        onDraw(target, combinedTransform)

        // draw its children
        for (std::size_t i = 0; i < m_children.size(); ++i)
            m_children[i]->draw(target, combinedTransform);
    }

private:

    virtual void onDraw(sf::RenderTarget& target, const sf::Transform& transform) const = 0;

    sf::Transform m_transform;
    std::vector<Node*> m_children;
};

// a simple derived class: a node that draws a sprite
class SpriteNode : public Node
{
public:

    // .. functions to define the sprite

private:

    virtual void onDraw(sf::RenderTarget& target, const sf::Transform& transform) const
    {
        target.draw(m_sprite, transform);
    }

    sf::Sprite m_sprite;
};