Handling Time
=====

Time in DSFML
===
Just like the SFML implementation of time, DSFML doesn't impose a specific unit or type for time values, like always being a float for seconds or long for milliseconds and leaving it up to the programmer to convert as needed. It instead provides a struct, dsfml.system.time.Time, that wraps this functionality.

The Time struct contains a relative time value or time span. It's exclusively a value used to represent a certain amount of time, as opposed to a full-on date-time class.