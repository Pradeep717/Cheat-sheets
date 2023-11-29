---

# Socket.IO Communication Guide

Socket.IO facilitates real-time bidirectional communication between the server and clients. This guide covers the `emit` and `on` methods, essential for communication in Socket.IO.

## `emit` Method

The `emit` method sends data or events from the server to connected clients.

### Examples:

#### Sending to Sender Only:
```javascript
socket.emit('message', 'This is a test');
```

#### Broadcasting to All Clients:
```javascript
io.emit('message', 'This is a test');
```

#### Broadcasting to All Clients Except Sender:
```javascript
socket.broadcast.emit('message', 'This is a test');
```

#### Sending to All Clients in a Specific Room (Channel):
```javascript
socket.broadcast.to('game').emit('message', 'Nice game');
```

#### Sending to All Clients in a Room, Including Sender:
```javascript
io.in('game').emit('message', 'Cool game');
```

#### Sending to Sender Only if in a Specific Room:
```javascript
socket.to('game').emit('message', 'Enjoy the game');
```

#### Broadcasting to All Clients in a Namespace:
```javascript
io.of('myNamespace').emit('message', 'GG');
```

#### Sending to an Individual Socket ID (Server-side):
```javascript
socket.broadcast.to(socketid).emit('message', 'For your eyes only');
```

#### Joining a Room (Server-side):
```javascript
socket.join('some room');
```

#### Leaving a Room (Server-side):
```javascript
socket.leave('some room');
```

## `on` Method

The `on` method listens for events emitted by the server or other clients.

### Examples:

#### Listening for an Event:
```javascript
socket.on('event', callback);
```

#### Handling Connection Event:
```javascript
io.on('connection', socket => { ... });
```

#### Handling Disconnection Event:
```javascript
socket.on('disconnect', () => { ... });
```

#### Handling Room Joining Event:
```javascript
socket.on('join', room => { ... });
```

#### Handling Room Leaving Event:
```javascript
socket.on('leave', room => { ... });
```

## Server-Side Implementation

### Socket.IO Server Code:
```javascript
io.on('connection', (socket) => {
  // Handle connection logic
  
  socket.on('joinRoom', (room) => {
    // Logic for joining a room
  });

  socket.on('leaveRoom', (room) => {
    // Logic for leaving a room
  });

  socket.on('listenEvent', (topic) => {
    // Logic for listening to a specific event/topic
  });

  socket.on('disconnect', () => {
    // Handle disconnection logic
  });
});
```

## Client-Side Implementation

### Connecting to Socket.IO Server:
```javascript
const socket = io('http://localhost:3004');

socket.on('connect', () => {
  // Handle connection logic
});

socket.on('disconnect', () => {
  // Handle disconnection logic
});

socket.emit('joinRoom', roomName);
socket.emit('listenEvent', eventName);

socket.on('receiveMessage', (data) => {
  // Handle received message
});
```

---

---

# More details

In the context of Socket.IO, `emit` and `on` are two fundamental methods used for communication between the server and clients. They work together to establish a real-time connection, enabling the exchange of data and events.

**`emit`** is used to send data or events from the server to connected clients. It's like broadcasting information from a central point to multiple recipients. The `emit` method takes two arguments: the event name and the data to send. For example:

```javascript
socket.emit('message', 'Hello from the server!');
```

In this example, the server emits an event named `message` and sends the string `'Hello from the server!'` as data. This event will be received by all connected clients.

**`on`** is used to listen for events emitted by the server or other clients. It's like subscribing to a channel to receive updates. The `on` method takes two arguments: the event name and a callback function. For example:

```javascript
socket.on('message', (data) => {
  console.log('Received message:', data);
});
```

In this example, the client listens for the `message` event and defines a callback function that will be executed whenever the event is emitted. The callback function receives the data sent with the event, which is logged to the console in this case.

Together, `emit` and `on` enable a bi-directional communication channel between the server and clients. The server can send information to clients using `emit`, and clients can respond or react to events using `on`. This real-time communication is what makes Socket.IO a powerful tool for building web applications that require dynamic and interactive user experiences.

## Server-Side Implementation

```javascript
io.on("connection", (socket) => {
  console.log("A user connected");

  socket.on("joinRoom", (room) => {
    socket.join(room);
    console.log(`User joined room: ${room}`);
    console.log(
      `Number of users in room ${room}: ${
        io.sockets.adapter.rooms.get(room).size
      }`
    );
  });

  socket.on("leaveRoom", (room) => {
    socket.leave(room);
    console.log(`User left room: ${room}`);
    // Check if the room exists
    let roomDetails = io.sockets.adapter.rooms.get(room);
    if (!roomDetails || roomDetails.size === 0) {
      console.log(`No users in room ${room}, closing changeStream`);
      if (changeStreams[room]) {
        changeStreams[room].close();
        delete changeStreams[room];
      }
    }
  });

  socket.on("listenEvent", (topic) => {
    console.log(`User listening to topic: ${topic}`);
    if (!changeStreams[topic]) {
      console.log("Creating new changeStream");
      changeStreams[topic] = connection.connection.db
        .collection(topic)
        .watch();
    }

    changeStreams[topic].on("change", async (change) => {
      const updateDocument = await connection.connection.db
        .collection(topic)
        .findOne({ _id: change.documentKey._id });

      console.log(
        `Topic name : ${topic}, id : ${updateDocument._id}, createdAt : ${updateDocument.createdAt}`
      );
      //broadcast to all clients in the given topic
      io.to(topic).emit("receiveMessage", updateDocument);
    });
  });

  socket.on("disconnect", () => {
    console.log("A user disconnected");
  });
});
```

## Client-Side Implementation

```javascript
import { io } from "socket.io-client";

export default {
  name: "HomeView",
  data() {
    return {
      socket: null,
      connected: false,
      value: "",
      isLoading: false,
      fooEvents: [],
      roomName: window.location.pathname.split('/')[2],
    };
  },
  created() {
    // Connect to the Socket.IO server
    this.socket = io("http://localhost:3004", {
      transports: ['websocket'],
    });

    this.socket.on("connect", () => {
      console.log("Connected!");
      this.connected = true;
    });

    this.socket.on("disconnect", () => {
      this.socket.emit('leaveRoom', this.roomName);
      console.log("Disconnected!");
      this.connected = false;
    });

    this.socket.emit('joinRoom', this.roomName);
    this.socket.emit('listenEvent', this.roomName);

    this.socket.on('receiveMessage', data => {
      console.log(data);
      this.fooEvents.push(data);
    });
  },
  beforeUnmount() {
    this.socket.emit('leaveRoom', this.roomName);
    this.socket.disconnect();
  },
};
```

---
![](https://res.cloudinary.com/dj76d2css/image/upload/v1701257391/hLVCykeTBk_klel0g.png)
![](https://res.cloudinary.com/dj76d2css/image/upload/v1701257401/Code_mDY1Ih0gfZ_jly1jc.png)
[![Video Title](https://your-video-thumbnail-url.jpg)][(https://link-to-your-video.com](https://res.cloudinary.com/dj76d2css/video/upload/v1701257654/bandicam_2023-11-29_16-52-16-472_vwcu7k.mp4)https://res.cloudinary.com/dj76d2css/video/upload/v1701257654/bandicam_2023-11-29_16-52-16-472_vwcu7k.mp4)]

