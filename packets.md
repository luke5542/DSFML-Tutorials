Using and extending packets
=====

Problems that need to be solved
---

Exchanging data on a network is more tricky than it seems. The reason is that different machines, with different operating systems and processors, can be involved. Several problems arise if you want to exchange data reliably between these different machines.

The first problem is the endianness. The endianness is the order in which a particular processor interprets the bytes of primitive types that occupy more than a single byte (integers and floating point numbers). There are two main families: "big endian" processors, which store the most significant byte first, and "little endian" processors, which store the least significant byte first. There are other, more exotic byte orders, but you'll probably never have to deal with them. The problem is obvious: If you send a variable between two computers whose endianness doesn't match, they won't see the same value. For example, the 16-bit integer "42" in big endian notation is 00000000 00101010, but if you send this to a little endian machine, it will be interpreted as "10752".

The second problem is specific to how the TCP protocol works. Because it doesn't preserve message boundaries, and can split or combine chunks of data, receivers must properly reconstruct incoming messages before interpreting them. Otherwise bad things might happen, like reading incomplete variables, or ignoring useful bytes.

You may of course face other problems with network programming, but these are the lowest-level ones, that almost everybody will have to solve. This is the reason why DSFML provides some simple tools to avoid them.

Packets
---

These two main problems are solved by using a specific class to pack your data: [Packet](http://dsfml.com/dsfml/network/packet.html). As a bonus, it provides a much nicer interface than plain old byte arrays.

Reading to and writing from a packet utilizes methods with the following pattern: `readType()` where Type is any primitive type (i.e. to read a bool you would use `readBool()`, an unsigned short would be read with `readUshort()`, etc). See the documentation for more information.

```D
// on sending side
short x = 10;
string s = "hello";
double d = 0.6;

Packet packet = new Packet();
packet.writeShort(x);
packet.writeString(s);
packet.writeDouble(d);

// on receiving side
short x;
string s;
double d;

x = packet.readShort();
s = packet.readString();
d = packet.readDouble();
```

> The interface described above is likely to change in future versions of the library, so be careful when upgrading.

Unlike writing, reading from a packet can fail if you try to extract more bytes than the packet contains. If a reading operation fails, the packet error flag is set. To check whether or not the last read operation succeeded, use the method `canRead()`:

```D
x = packet.readByte();
if (packet.canRead())
{
    // ok
}
else
{
    // error, failed to read 'x' from the packet
}
```

Sending and receiving packets is as easy as sending/receiving an array of bytes: sockets have an overload of send and receive that directly accept a [Packet](http://dsfml.com/dsfml/network/packet.html).

```D
// with a TCP socket
tcpSocket.send(packet);
tcpSocket.receive(packet);

// with a UDP socket
udpSocket.send(packet, recipientAddress, recipientPort);
udpSocket.receive(packet, senderAddress, senderPort);
```

Packets solve the "message boundaries" problem, which means that when you send a packet on a TCP socket, you receive the exact same packet on the other end, it cannot contain less bytes, or bytes from the next packet that you send. However, it has a slight drawback: To preserve message boundaries, [Packet](http://dsfml.com/dsfml/network/packet.html) has to send some extra bytes along with your data, which implies that you can only receive them with a [Packet](http://dsfml.com/dsfml/network/packet.html) if you want them to be properly decoded. Simply put, you can't send an DSFML packet to a non-DSFML packet recipient, it has to use an DSFML packet for receiving too. Note that this applies to TCP only, UDP is fine since the protocol itself preserves message boundaries.

Extending packets to handle user types
---

Packets have overloads of their operators for all the primitive types and the most common standard types, but what about your own classes? Two methods have been supplied to support the insertion and removal of non-primitive data: `const(void)[] getData()` for reading and `void append(const(void)[] data)` for writing.


Custom packets
---

Packets provide nice features on top of your raw data, but what if you want to add your own features such as automatically compressing or encrypting the data? This can easily be done by deriving from [Packet](http://dsfml.com/dsfml/network/packet.html) and overriding the following functions:

+ `onSend()`: called before the data is sent by the socket
+ `onReceive()`: called after the data has been received by the socket

These functions provide direct access to the data, so that you can transform them according to your needs.

Here is a mock-up of a packet that performs automatic compression/decompression:

```D
class ZipPacket : Packet
{
    override const(void)[] onSend()
    {
        const(void)[] srcData = getData();
        return compressTheData(srcData); // this is a fake function, of course :)
    }
    override void onReceive(const(void)[] data)
    {
        const(void)[] dstData = uncompressTheData(data, size, &dstSize); // this is a fake function, of course :)
        append(dstData);
    }
}
```

Such a packet class can be used exactly like [Packet](http://dsfml.com/dsfml/network/packet.html). All your method overrides and overloads will apply to them as well.

```D
ZipPacket packet = new ZipPacket();
packet.writeByte(x);
socket.send(packet);
```
