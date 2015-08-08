Communicating with sockets
=====

Sockets
---

A socket is the interface between your application and the outside world: through a socket, you can send and receive data. Therefore, any network program will most likely have to deal with sockets, they are the central element of network communication.

There are several kinds of sockets, each providing specific features. DSFML implements the most common ones: TCP sockets and UDP sockets.

TCP vs UDP
---

It is important to know what TCP and UDP sockets can do, and what they can't do, so that you can choose the best socket type according to the requirements of your application.

The main difference is that TCP sockets are connection-based. You can't send or receive anything until you are connected to another TCP socket on the remote machine. Once connected, a TCP socket can only send and receive to/from the remote machine. This means that you'll need one TCP socket for each client in your application. UDP is not connection-based, you can send and receive to/from anyone at any time with the same socket.

The second difference is that TCP is reliable unlike UDP. It ensures that what you send is always received, without corruption and in the same order. UDP performs fewer checks, and doesn't provide any reliability: what you send might be received multiple times (duplication), or in a different order, or be lost and never reach the remote computer. However, UDP does guarantee that data which is received is always valid (not corrupted). UDP may seem scary, but keep in mind that *almost all the time*, data arrives correctly and in the right order.

The third difference is a direct consequence of the second one: UDP is faster and more lightweight than TCP. Because it has fewer requirements, thus less overhead.

The last difference is about the way data is transported. TCP is a *stream* protocol: there's no message boundary, if you send "Hello" and then "DSFML", the remote machine might receive "HelloDSFML", "Hel" + "loDSFML", or even "He" + "loDS" + "FML". UDP is a *datagram* protocol. Datagrams are packets that can't be mixed with each other. If you receive a datagram with UDP, it is guaranteed to be exactly the same as it was sent.

Oh, and one last thing: since UDP is not connection-based, it allows broadcasting messages to multiple recipients, or even to an entire network. The one-to-one communication of TCP sockets doesn't allow that.

Connecting a TCP socket
---

As you can guess, this part is specific to TCP sockets. There are two sides to a connection: the one that waits for the incoming connection (let's call it the server), and the one that triggers it (let's call it the client).

On client side, things are simple: the user just needs to have a [TcpSocket](http://dsfml.com/dsfml/network/tcpsocket.html) and call its `connect()` function to start the connection attempt.

```D
import dsfml.network;

TcpSocket socket = new TcpSocket();
Socket.Status status = socket.connect("192.168.0.5", 53000);
if (status != Socket.Status.Done)
{
    // error...
}
```

The first argument is the address of the host to connect to. It is an [IpAddress](http://dsfml.com/dsfml/network/ipaddress.html), which can represent any valid address: a URL, an IP address, or a network host name. See its documentation for more details.

The second argument is the port to connect to on the remote machine. The connection will succeed only if the server is accepting connections on that port.

There's an optional third argument, a time out value. If set, and the connection attempt doesn't succeed before the time out is over, the function returns an error. If not specified, the default operating system time out is used.

Once connected, you can retrieve the address and port of the remote computer if needed, with the `getRemoteAddress()` and `getRemotePort()` functions.

> All functions of socket classes are blocking by default. This means that your program (more specifically the thread that contains the function call) will be stuck until the operation is complete. This is important because some functions may take very long: For example, trying to connect to an unreachable host will only return after a few seconds, receiving will wait until there's data available, etc. You can change this behavior and make all functions non-blocking by using the `setBlocking()` function of the socket. See the next chapters for more details.

On the server side, a few more things have to be done. Multiple sockets are required: One that listens for incoming connections, and one for each connected client.

To listen for connections, you must use the special [TcpListener](http://dsfml.com/dsfml/network/tcplistener.html) class. Its only role is to wait for incoming connection attempts on a given port, it can't send or receive data.

```D
TcpListener listener = new TcpListener();

// bind the listener to a port
if (listener.listen(53000) != Socket.Status.Done)
{
    // error...
}

// accept a new connection
TcpSocket client = new TcpSocket();
if (listener.accept(client) != Socket.Status.Done)
{
    // error...
}

// use "client" to communicate with the connected client,
// and continue to accept new connections with the listener
```

The `accept()` function blocks until a connection attempt arrives (unless the socket is configured as non-blocking). When it happens, it initializes the given socket and returns. The socket can now be used to communicate with the new client, and the listener can go back to waiting for another connection attempt.

After a successful call to `connect()` (on client side) and `accept()` (on server side), the communication is established and both sockets are ready to exchange data.

Binding a UDP socket
---

UDP sockets need not be connected, however you need to bind them to a specific port if you want to be able to receive data on that port. A UDP socket cannot receive on multiple ports simultaneously.

```D
UdpSocket socket = new UdpSocket();

// bind the socket to a port
if (socket.bind(54000) != Socket.Status.Done)
{
    // error...
}
```

After binding the socket to a port, it's ready to receive data on that port. If you want the operating system to bind the socket to a free port automatically, you can pass `Socket.AnyPort`, and then retrieve the chosen port with `socket.getLocalPort()`.

UDP sockets that send data don't need to do anything before sending.

Sending and receiving data
---

Sending and receiving data is done in the same way for both types of sockets. The only difference is that UDP has two extra arguments: the address and port of the sender/recipient. There are two different functions for each operation: the low-level one, that sends/receives a raw array of bytes, and the higher-level one, which uses the [Packet](http://dsfml.com/dsfml/network/packet.html) class. See the tutorial on packets for more details about this class. In this tutorial, we'll only explain the low-level functions.

To send data, you must call the send function with a pointer to the data that you want to send, and the number of bytes to send.

```D
char[100] data = ...;

// TCP socket:
if (socket.send(data) != Socket.Status.Done)
{
    // error...
}

// UDP socket:
IpAddress recipient = new IpAddress("192.168.0.5");
unsigned short port = 54000;
if (socket.send(data, recipient, port) != Socket.Status.Done)
{
    // error...
}
```

The `send()` functions take a `const(void)[]`, so you can pass an array of anything. However, it is generally a bad idea to send something other than an array of bytes because native types with a size larger than 1 byte are not guaranteed to be the same on every machine: Types such as `int` or `long` may have a different size, and/or a different endianness. Therefore, such types cannot be exchanged reliably across different systems. This problem is explained (and solved) in the [tutorial on packets](https://github.com/luke5542/DSFML-Tutorials/blob/master/packets.md).

With UDP you can broadcast a message to an entire sub-network in a single call: to do so you can use the special address `IpAddress.Broadcast`.

There's another thing to keep in mind with UDP: Since data is sent in datagrams and the size of these datagrams has a limit, you are not allowed to exceed it. Every call to `send()` must send less than `UdpSocket.maxDatagramSize` bytes -- which is a little less than 2^16 (65536) bytes.

To receive data, you must call the `receive()` function:

```D
char[] data;
size_t received;

// TCP socket:
if (socket.receive(data, 100, received) != Socket.Status.Done)
{
    // error...
}
writeln("Received ", received, " bytes");

// UDP socket:
IpAddress sender;
ushort port;
if (socket.receive(data, 100, received, sender, port) != Socket.Status.Done)
{
    // error...
}
writeln("Received ", received, " bytes from ", sender, " on port ", port);
```

It is important to keep in mind that if the socket is in blocking mode, `receive()` will wait until something is received, blocking the thread that called it (and thus possibly the whole program).

The first two arguments specify the buffer to which the received bytes are to be copied, along with its maximum size. The third argument is a variable that will contain the actual number of bytes received after the function returns.
With UDP sockets, the last two arguments will contain the address and port of the sender after the function returns. They can be used later if you want to send a response.

These functions are low-level, and you should use them only if you have a very good reason to do so. A more robust and flexible approach involves using [packets](https://github.com/luke5542/DSFML-Tutorials/blob/master/packets.md).

Blocking on a group of sockets
---

Blocking on a single socket can quickly become annoying, because you will most likely have to handle more than one client. You most likely don't want socket A to block your program while socket B has received something that could be processed. What you would like is to block on multiple sockets at once, i.e. waiting until any of them has received something. This is possible with socket selectors, represented by the [SocketSelector](http://dsfml.com/dsfml/network/socketselector.html) class.

A selector can monitor all types of sockets: [TcpSocket](http://dsfml.com/dsfml/network/tcpsocket.html), [UdpSocket](http://dsfml.com/dsfml/network/udpsocket.html), and [TcpListener](http://dsfml.com/dsfml/network/tcplistener.html). To add a socket to a selector, use its `add()` function:

```D
TcpSocket socket = new TcpSocket();

SocketSelector selector = new Selector();
selector.add(socket);
```

> A selector is not a socket container. It only references (points to) the sockets that you add, it doesn't store them. There is no way to retrieve or count the sockets that you put inside. Instead, it is up to you to have your own separate socket storage (like an array or a DList).

Once you have filled the selector with all the sockets that you want to monitor, you must call its `wait()` function to wait until any one of them has received something (or has triggered an error). You can also pass an optional time out value, so that the function will fail if nothing has been received after a certain period of time -- this avoids staying stuck forever if nothing happens.

```D
if (selector.wait(seconds(10)))
{
    // received something
}
else
{
    // timeout reached, nothing was received...
}
```

If the `wait()` function returns true, it means that one or more socket(s) have received something, and you can safely call receive on the socket(s) with pending data without having them block. If the socket is a [TcpListener](http://dsfml.com/dsfml/network/tcplistener.html), it means that an incoming connection is ready to be accepted and that you can call its accept function without having it block.

Since the selector is not a socket container, it cannot return the sockets that are ready to receive. Instead, you must test each candidate socket with the `isReady()` function:

```D
if (selector.wait(seconds(10)))
{
    // for each socket... <-- pseudo-code because I don't know how you store your sockets :)
    {
        if (selector.isReady(socket))
        {
            // this socket is ready, you can receive (or accept if it's a listener)
            socket.receive(...);
        }
    }
}
```

You can have a look at the API documentation of the [SocketSelector](http://dsfml.com/dsfml/network/socketselector.html) class for a working example of how to use a selector to handle connections and messages from multiple clients.

As a bonus, the time out capability of `Selector.wait()` allows you to easily implement a receive-with-timeout function, which is not directly available in the socket classes:

```D
Socket.Status receiveWithTimeout(TcpSocket socket, Packet packet, Time timeout)
{
    SocketSelector selector = new SocketSelector();
    selector.add(socket);
    if (selector.wait(timeout))
        return socket.receive(packet);
    else
        return Socket.Status.NotReady;
}
```

Non-blocking sockets
---

All sockets are blocking by default, but you can change this behavior at any time with the `setBlocking()` function.

```D
TcpSocket tcpSocket = ...;
tcpSocket.setBlocking(false);

TcpListener listenerSocket = ...;
listenerSocket.setBlocking(false);

UdpSocket udpSocket = ...;
udpSocket.setBlocking(false);
```

Once a socket is set as non-blocking, all of its functions always return immediately. For example, `receive()` will return with status `Socket.Status.NotReady` if there's no data available. Or, `accept()` will return immediately, with the same status, if there's no pending connection.

Non-blocking sockets are the easiest solution if you already have a main loop that runs at a constant rate. You can simply check if something happened on your sockets in every iteration, without having to block program execution.
