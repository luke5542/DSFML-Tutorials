Handling Time
=====

Time in DSFML
---

Just like the SFML implementation of time, DSFML doesn't impose a specific unit or type for time values, like always being a float for seconds or long for milliseconds and leaving it up to the programmer to convert as needed. It instead provides a struct, [Time](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/system/time.d), that wraps this functionality.

The [Time](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/system/time.d) struct contains a relative time value or time span. It's exclusively a value used to represent a certain amount of time, as opposed to a full-on date-time class.

Converting Time
---

A [Time](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/system/time.d) object can be created from any of the source units: seconds, milliseconds, or microseconds. There is a (non-member) function to turn each of them into a [Time](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/system/time.d):

```D
Time t1 = microseconds(42000000);
auto t2 = milliseconds(42000);
auto t3 = seconds(42);
```

Notice how all three of these times are equal.

Similarly, a [Time](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/system/time.d) object can be converted back to either seconds, milliseconds, or microseconds:

```D
Time  time = ...;

long  usec = time.asMicroseconds();
int   msec = time.asMilliseconds();
float sec  = time.asSeconds();
```

Playing With Time Values
---

[Time](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/system/time.d) is just an amount of time, so it supports arithmetic operations such as addition, subtraction, comparison, etc. Times can also be negative.

```D
Time t1 = seconds(42);
auto t2 = t1 * 2;
auto t3 = t1 + t2;
auto t4 = -t3;

bool b1 = (t1 == t2);
bool b2 = (t3 > t4);
```

Measuring Time
---

Now that we've seen how to manipulate time values with DSFML, let's see how to do something that almost every program needs: measuring the time elapsed.

DSFML, like SFML, has a very simple class for measuring time: [Clock](https://github.com/Jebbs/DSFML/blob/master/src/dsfml/system/clock.d). It only has two methods: `getElapsedTime`, to retrieve the time elapsed since the clock started, and `restart`, to restart the clock.

```D
Clock clock = new Clock(); // starts the clock
...
Time elapsed1 = clock.getElapsedTime();
writeln(elapsed1.asSeconds());
clock.restart();
...
Time elapsed2 = clock.getElapsedTime();
writeln(elapsed2.asSeconds());
```

Note that `restart` also returns the elapsed time, so that you can avoid the slight gap that would exist if you had to call `getElapsedTime` explicitly before `restart`.

Here is an example that uses the time elapsed at each iteration of the game loop to update the game logic:

```D
Clock clock = new Clock();
while (window.isOpen())
{
    Time elapsed = clock.restart();
    updateGame(elapsed);
    ...
}
``` 