# Pro Assignment 2: Real-Time Features (WebSockets)

## Learning Objectives

By completing this assignment, you will:
- Understand WebSocket technology and real-time communication
- Learn to set up WebSocket servers
- Practice bidirectional client-server communication
- Implement real-time updates without page refresh
- Handle WebSocket connections, events, and errors
- Understand the difference between HTTP and WebSocket protocols
- Gain experience with Socket.io (Node.js) or Flask-SocketIO (Python)

---

## Prerequisites

- Completed at least Assignment 2 (E2E Hello World)
- Working server and client application
- Understanding of your chosen stack (Node.js or Python)
- Cursor AI installed and configured
- Basic understanding of events and callbacks

---

## Overview

This is a **pro-level bonus assignment** worth **10 bonus points**. It's optional but highly recommended for students who want to learn real-time web features.

**Goal:** Add real-time functionality to your application using WebSockets, enabling live updates without page refresh.

**Use Cases:**
- Live chat messages
- Real-time notifications
- Live data updates (e.g., stock prices, scores)
- Collaborative editing
- Live user presence

---

## Instructions

### Step 1: Choose Your WebSocket Library

Select a WebSocket library based on your stack:

**For Node.js:**
- **Socket.io** (recommended) - Most popular, easy to use, automatic fallback
- **ws** - Lightweight, native WebSocket implementation
- **uWebSockets** - High performance, advanced

**For Python:**
- **Flask-SocketIO** (recommended) - Easy integration with Flask
- **python-socketio** - Standalone Socket.IO server
- **websockets** - Native WebSocket library

**Recommendation:** Start with **Socket.io** (Node.js) or **Flask-SocketIO** (Python) - they're the easiest to use.

### Step 2: Install WebSocket Dependencies

1. **Install WebSocket library:**

   **Node.js (Socket.io):**
   ```bash
   npm install socket.io
   ```

   **Node.js (ws - lightweight):**
   ```bash
   npm install ws
   ```

   **Python (Flask-SocketIO):**
   ```bash
   pip install flask-socketio
   ```

   **Python (python-socketio):**
   ```bash
   pip install python-socketio
   ```

2. **Update package files:**

   **Node.js (package.json):**
   ```json
   {
     "dependencies": {
       "socket.io": "^4.7.0"
     }
   }
   ```

   **Python (requirements.txt):**
   ```txt
   flask-socketio==5.3.6
   ```

### Step 3: Set Up WebSocket Server

1. **Create WebSocket server:**

   **Node.js (Socket.io):**
   ```javascript
   // server/index.js
   const express = require('express');
   const http = require('http');
   const { Server } = require('socket.io');

   const app = express();
   const server = http.createServer(app);
   const io = new Server(server, {
     cors: {
       origin: "http://localhost:3000", // Your client URL
       methods: ["GET", "POST"]
     }
   });

   // WebSocket connection handler
   io.on('connection', (socket) => {
     console.log('User connected:', socket.id);

     // Send welcome message
     socket.emit('message', {
       type: 'welcome',
       text: 'Connected to server!',
       timestamp: new Date().toISOString()
     });

     // Handle incoming messages
     socket.on('message', (data) => {
       console.log('Received:', data);
       // Broadcast to all clients
       io.emit('message', {
         type: 'broadcast',
         text: data.text,
         user: data.user || 'Anonymous',
         timestamp: new Date().toISOString()
       });
     });

     // Handle disconnection
     socket.on('disconnect', () => {
       console.log('User disconnected:', socket.id);
     });
   });

   const PORT = process.env.PORT || 3001;
   server.listen(PORT, () => {
     console.log(`Server running on port ${PORT}`);
     console.log(`WebSocket server ready`);
   });
   ```

   **Python (Flask-SocketIO):**
   ```python
   # server/app.py
   from flask import Flask
   from flask_socketio import SocketIO, emit

   app = Flask(__name__)
   socketio = SocketIO(app, cors_allowed_origins="*")

   @socketio.on('connect')
   def handle_connect():
       print('User connected')
       emit('message', {
           'type': 'welcome',
           'text': 'Connected to server!',
           'timestamp': datetime.now().isoformat()
       })

   @socketio.on('message')
   def handle_message(data):
       print('Received:', data)
       # Broadcast to all clients
       socketio.emit('message', {
           'type': 'broadcast',
           'text': data.get('text', ''),
           'user': data.get('user', 'Anonymous'),
           'timestamp': datetime.now().isoformat()
       })

   @socketio.on('disconnect')
   def handle_disconnect():
       print('User disconnected')

   if __name__ == '__main__':
       socketio.run(app, host='0.0.0.0', port=3001, debug=True)
   ```

### Step 4: Set Up Client WebSocket Connection

1. **Install client library:**

   **Node.js (Socket.io client):**
   ```bash
   npm install socket.io-client
   ```

   **Or use CDN in HTML:**
   ```html
   <script src="https://cdn.socket.io/4.7.0/socket.io.min.js"></script>
   ```

   **Python (JavaScript client):**
   ```html
   <script src="https://cdn.socket.io/4.7.0/socket.io.min.js"></script>
   ```

2. **Create client connection:**

   **HTML/JavaScript:**
   ```html
   <!-- client/index.html -->
   <!DOCTYPE html>
   <html>
   <head>
     <title>Real-Time App</title>
     <script src="https://cdn.socket.io/4.7.0/socket.io.min.js"></script>
   </head>
   <body>
     <div id="messages"></div>
     <input type="text" id="messageInput" placeholder="Type a message...">
     <button onclick="sendMessage()">Send</button>

     <script>
       // Connect to WebSocket server
       const socket = io('http://localhost:3001');

       // Listen for connection
       socket.on('connect', () => {
         console.log('Connected to server');
         addMessage('System', 'Connected to server!', 'system');
       });

       // Listen for messages
       socket.on('message', (data) => {
         addMessage(data.user || 'Server', data.text, data.type);
       });

       // Listen for disconnection
       socket.on('disconnect', () => {
         console.log('Disconnected from server');
         addMessage('System', 'Disconnected from server', 'system');
       });

       // Send message function
       function sendMessage() {
         const input = document.getElementById('messageInput');
         const text = input.value.trim();
         
         if (text) {
           socket.emit('message', {
             text: text,
             user: 'You'
           });
           input.value = '';
         }
       }

       // Add message to UI
       function addMessage(user, text, type) {
         const messagesDiv = document.getElementById('messages');
         const messageDiv = document.createElement('div');
         messageDiv.className = `message ${type}`;
         messageDiv.innerHTML = `
           <strong>${user}:</strong> ${text}
           <small>${new Date().toLocaleTimeString()}</small>
         `;
         messagesDiv.appendChild(messageDiv);
         messagesDiv.scrollTop = messagesDiv.scrollHeight;
       }

       // Send on Enter key
       document.getElementById('messageInput').addEventListener('keypress', (e) => {
         if (e.key === 'Enter') {
           sendMessage();
         }
       });
     </script>
   </body>
   </html>
   ```

### Step 5: Implement Real-Time Features

1. **Live message updates:**

   **Server (Node.js):**
   ```javascript
   // Broadcast new message to all clients
   io.on('connection', (socket) => {
     socket.on('new_message', (data) => {
       // Broadcast to all connected clients
       io.emit('message_received', {
         id: Date.now(),
         text: data.text,
         user: data.user,
         timestamp: new Date().toISOString()
       });
     });
   });
   ```

   **Client:**
   ```javascript
   // Listen for new messages
   socket.on('message_received', (data) => {
     displayMessage(data);
   });
   ```

2. **User presence (who's online):**

   **Server (Node.js):**
   ```javascript
   const users = new Set();

   io.on('connection', (socket) => {
     socket.on('user_joined', (username) => {
       users.add(username);
       io.emit('users_updated', Array.from(users));
     });

     socket.on('disconnect', () => {
       // Remove user when they disconnect
       io.emit('user_left', socket.id);
     });
   });
   ```

3. **Typing indicators:**

   **Server (Node.js):**
   ```javascript
   socket.on('typing', (data) => {
     // Broadcast to all except sender
     socket.broadcast.emit('user_typing', {
       user: data.user,
       isTyping: data.isTyping
     });
   });
   ```

   **Client:**
   ```javascript
   let typingTimeout;
   const input = document.getElementById('messageInput');

   input.addEventListener('input', () => {
     socket.emit('typing', { user: 'You', isTyping: true });
     
     clearTimeout(typingTimeout);
     typingTimeout = setTimeout(() => {
       socket.emit('typing', { user: 'You', isTyping: false });
     }, 1000);
   });
   ```

### Step 6: Handle Errors and Reconnection

1. **Error handling:**

   **Client:**
   ```javascript
   socket.on('connect_error', (error) => {
     console.error('Connection error:', error);
     addMessage('System', 'Failed to connect to server', 'error');
   });

   socket.on('error', (error) => {
     console.error('Socket error:', error);
   });
   ```

2. **Automatic reconnection:**

   **Socket.io automatically handles reconnection**, but you can customize it:

   ```javascript
   const socket = io('http://localhost:3001', {
     reconnection: true,
     reconnectionDelay: 1000,
     reconnectionAttempts: 5
   });

   socket.on('reconnect', (attemptNumber) => {
     console.log('Reconnected after', attemptNumber, 'attempts');
   });

   socket.on('reconnect_failed', () => {
     console.error('Failed to reconnect');
   });
   ```

### Step 7: Add Real-Time Features to Your App

1. **Live chat:**
   - Messages appear instantly for all users
   - No page refresh needed
   - Typing indicators
   - User presence

2. **Live updates:**
   - Real-time data updates (e.g., scores, prices)
   - Live notifications
   - Status changes

3. **Collaborative features:**
   - Live cursors
   - Shared editing
   - Real-time collaboration

### Step 8: Test Your WebSocket Implementation

1. **Test connection:**
   - Open multiple browser tabs
   - Send messages from one tab
   - Verify messages appear in all tabs

2. **Test reconnection:**
   - Disconnect server
   - Reconnect server
   - Verify client reconnects automatically

3. **Test error handling:**
   - Send invalid data
   - Verify error handling works
   - Check error messages

### Step 9: Document Your WebSocket Implementation

1. **Update README.md:**
   - Add "Real-Time Features" section
   - Document WebSocket setup
   - Explain events and messages
   - Include example usage
   - Document connection details

2. **Create WebSocket documentation:**
   ```markdown
   ## Real-Time Features (WebSockets)

   ### Connection
   - Server: `ws://localhost:3001`
   - Library: Socket.io

   ### Events
   - `connect` - Client connected
   - `message` - New message received
   - `disconnect` - Client disconnected
   - `typing` - User typing indicator
   ```

---

## Requirements

### Required

- [ ] WebSocket server set up and running
- [ ] Client WebSocket connection established
- [ ] Real-time message updates working
- [ ] Live updates without page refresh
- [ ] Connection/disconnection handling
- [ ] Error handling implemented
- [ ] WebSocket documented in README.md

### Optional

- [ ] User presence (who's online)
- [ ] Typing indicators
- [ ] Private messages (rooms)
- [ ] Message history
- [ ] Reconnection handling
- [ ] Authentication for WebSocket connections

---

## Acceptance Criteria

Your submission will be evaluated based on:

- [ ] WebSocket server successfully set up
- [ ] Client connects to WebSocket server
- [ ] Real-time updates working (no page refresh)
- [ ] Messages broadcast to all clients
- [ ] Connection/disconnection handled properly
- [ ] Error handling implemented
- [ ] WebSocket features documented
- [ ] Example use case demonstrated
- [ ] Changes committed and pushed to GitHub

---

## Submission Requirements

1. **WebSocket code:** All WebSocket code in server and client
2. **Documentation:** README.md updated with WebSocket section
3. **Example:** Working example of real-time features
4. **Commit:** All changes committed and pushed to GitHub

**Commit message example:**
```bash
git commit -m "Pro Assignment 2: Real-Time Features (WebSockets)"
```

---

## Grading Rubric

See [Grading Rubrics](../materials/grading-rubrics.md) for detailed criteria.

**Total Points:** 10 bonus points

- **WebSocket setup:** 3 points
  - Server set up: 1 point
  - Client connection: 1 point
  - Basic events working: 1 point
- **Real-time features:** 4 points
  - Live updates working: 2 points
  - No page refresh needed: 1 point
  - Multiple clients supported: 1 point
- **Error handling:** 2 points
  - Connection errors handled: 1 point
  - Disconnection handled: 1 point
- **Documentation:** 1 point
  - WebSocket documented: 0.5 points
  - Example use case: 0.5 points

---

## Tips for Success

- **Start with Socket.io/Flask-SocketIO:** Easiest to set up and use
- **Test with multiple clients:** Open multiple browser tabs to test
- **Handle errors gracefully:** WebSocket connections can fail
- **Use Cursor AI:** Ask Cursor to generate WebSocket code
- **Keep it simple:** Start with basic message broadcasting
- **Test reconnection:** Verify automatic reconnection works
- **Document as you go:** Write down what you learn

---

## Example WebSocket Structure

```
your-project/
├── server/
│   ├── index.js (or app.py)
│   └── socket.js (or socket.py) - WebSocket handlers
├── client/
│   └── index.html (with Socket.io client)
├── package.json (or requirements.txt)
└── README.md
```

**Example WebSocket Events:**

**Server Events:**
- `connection` - New client connected
- `disconnect` - Client disconnected
- `message` - Message received from client
- `typing` - User typing indicator

**Client Events:**
- `connect` - Connected to server
- `disconnect` - Disconnected from server
- `message` - Message received from server
- `user_typing` - User typing indicator

---

## Common WebSocket Issues

### Issue: Connection refused

**Solutions:**
- Check server is running
- Verify port number is correct
- Check CORS settings
- Ensure firewall allows connection

### Issue: Messages not received

**Solutions:**
- Verify event names match (case-sensitive)
- Check client is connected
- Verify server is broadcasting correctly
- Check browser console for errors

### Issue: Multiple connections

**Solutions:**
- Close previous connections before creating new ones
- Use singleton pattern for socket connection
- Check for existing connections before creating

### Issue: Reconnection not working

**Solutions:**
- Check reconnection settings
- Verify server is accessible
- Check network connectivity
- Review Socket.io reconnection options

---

## Getting Help

- Ask questions in the help channel
- Review WebSocket documentation:
  - [Socket.io Documentation](https://socket.io/docs/v4/)
  - [Flask-SocketIO Documentation](https://flask-socketio.readthedocs.io/)
  - [WebSocket API (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)
- Check [FAQ](../materials/faq.md)
- Review [Student Guide](../materials/student-guide.md)
- Contact your teacher if needed

---

## Resources

**WebSocket Libraries:**
- [Socket.io](https://socket.io/) - Node.js WebSocket library
- [Flask-SocketIO](https://flask-socketio.readthedocs.io/) - Python WebSocket library
- [ws](https://github.com/websockets/ws) - Lightweight Node.js WebSocket library

**Learning Resources:**
- [WebSocket Tutorial](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API/Writing_WebSocket_client_applications)
- [Socket.io Tutorial](https://socket.io/get-started/chat)
- [Real-Time Web Apps](https://www.html5rocks.com/en/tutorials/websockets/basics/)

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-25 | RepodIn Education Team | Initial version |

---

**Next Review Date:** 2026-03-20

