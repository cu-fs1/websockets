## Socket.IO (for students)

Socket.IO is a JavaScript library that helps build **real-time, two-way communication** between a browser/client and a server.

It's often used when you want features like:
- chat apps (messages instantly appear to everyone)
- multiplayer games (frequent updates)
- live dashboards (events update the UI continuously)
- notifications (server pushes updates without the client polling)

> Note: Socket.IO is not "raw WebSocket." It is built on top of the idea of persistent connections, but it provides an **event-based API**, plus extra features like automatic reconnection and fallback transports.

---

## How Socket.IO feels (the mental model)

With Socket.IO, you typically write code like:
- The **server** waits for connections and listens for events.
- The **client** connects and listens for events from the server.
- Both sides can **emit events** (send messages) and **handle events** (receive messages).

Under the hood, Socket.IO manages the connection details (how messages travel), so you mostly focus on the events and data.

---

## Key terms

### 1) Client and Server
- **Client**: your browser app or Node process that connects to the server.
- **Server**: your Node/Express app that accepts connections and broadcasts events.

### 2) Socket (a single connection)
When a client connects, Socket.IO creates a **`socket`** object representing that connection.

### 3) Events (the main feature)
Instead of sending "one fixed message type," you send named events:
- `socket.emit("eventName", data)` sends an event
- `socket.on("eventName", handler)` listens for an event

### 4) `namespaces` and `rooms`
- **Namespaces** let you split functionality (e.g. `/chat`, `/admin`).
- **Rooms** let you group sockets (e.g. `room = "room-123"`) so you can message only those users.

### 5) Transports and fallback
Socket.IO can use different "transport" methods to get data to the other side.
If WebSocket isn't available, it can fall back to alternatives (like long-polling), which helps it work in more environments.

---

## Minimal example: broadcast a message

These examples use **ES modules** (`import ...`).
If you're running this with Node.js, either name files with `.mjs` (example: `server.mjs`) or set `"type": "module"` in `package.json`.

### 1) Install packages

Server:
```bash
pnpm add express socket.io
```

Client:
```bash
pnpm add socket.io-client
```

### 2) Server (`server.js`) - Express + Socket.IO

```js
import { createServer } from "http";
import express from "express";
import { Server } from "socket.io";

const app = express();
const server = createServer(app);
const io = new Server(server, {
  cors: { origin: "*" }, // For learning only. Restrict in production.
});

app.get("/", (req, res) => {
  res.send("Server is running. Open the client to connect to Socket.IO.");
});

io.on("connection", (socket) => {
  console.log("New client connected:", socket.id);

  // Send something to just this client
  socket.emit("welcome", "Connected to Socket.IO!");

  // Listen for a custom event from the client
  socket.on("chat message", (msg) => {
    console.log("Message received:", msg);

    // Broadcast to everyone (including the sender)
    io.emit("chat message", msg);
  });

  socket.on("disconnect", () => {
    console.log("Client disconnected:", socket.id);
  });
});

server.listen(3000, () => {
  console.log("Socket.IO server listening on http://localhost:3000");
});
```

### 3) Client (`client.js`)

```js
import { io } from "socket.io-client";

const socket = io("http://localhost:3000");

// Listen for server events
socket.on("welcome", (text) => {
  console.log(text);
});

socket.on("chat message", (msg) => {
  console.log("Chat:", msg);
});

// Send an event to the server
socket.emit("chat message", "Hello everyone!");
```

---

## Rooms exercise (send to a subset)

Often you don't want to broadcast to all users - only users in the same "room".

Server ideas:
- when a client joins `roomName`, add them with `socket.join(roomName)`
- send to only that room with `io.to(roomName).emit(...)`

Pseudo-example:

```js
socket.on("join", (roomName) => {
  socket.join(roomName);
});

socket.on("room update", (roomName, data) => {
  io.to(roomName).emit("room update", data);
});
```

---

## When to use Socket.IO

Use Socket.IO when you need:
- real-time updates (server pushes immediately)
- easy event-based messaging
- grouping users (rooms)
- resilient connections (reconnect + fallback behavior)

If you only need very simple, standards-only WebSocket behavior, you might choose **raw WebSocket** instead, but for most learning and app-building, Socket.IO is easier to work with.

