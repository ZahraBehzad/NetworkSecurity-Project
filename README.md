# encryptedChatFlask 🔒💬

A real-time, room-based chat application built with **Flask** and **Flask-SocketIO**, created as a course project for **Network Security**. It demonstrates the fundamentals of live client-server communication over WebSockets combined with a classical cipher (the **Rail Fence cipher**) used to encrypt messages before they are broadcast and stored.

> ⚠️ **Educational purpose only.** The Rail Fence cipher is a classical transposition cipher and is **not cryptographically secure** by modern standards — it is trivially breakable and is used here purely to illustrate encryption/decryption concepts in a network security course. Do not use this project to protect sensitive or real-world communications.

---

## Features

- 🔑 **Create or join chat rooms** using a randomly generated 4-letter room code
- ⚡ **Real-time messaging** powered by WebSockets (`Flask-SocketIO`)
- 🔐 **Message encryption** — every message is encrypted with the Rail Fence cipher before being sent/stored, and decrypted client-side via an API call before being displayed
- 🗄️ **Persistent storage** of rooms and messages using **MongoDB**
- 👥 **Live member tracking** — join/leave events are broadcast to the room
- 🖥️ Minimal, framework-free front end (vanilla JS + jQuery + Jinja templates)

---

## Tech Stack

| Layer          | Technology                                  |
|----------------|----------------------------------------------|
| Backend        | Python, Flask, Flask-SocketIO                 |
| Real-time comm | WebSockets (Socket.IO)                        |
| Database       | MongoDB (via `pymongo`)                       |
| Encryption     | Custom Rail Fence cipher implementation       |
| Frontend       | HTML, CSS, JavaScript, jQuery, Jinja2         |
| Deployment     | Configured for Vercel and Liara               |

---

## Project Structure

```
encryptedChatFlask/
├── functions/
│   └── railFence.py       # Rail Fence cipher: encryptRailFence() / decryptRailFence()
├── static/
│   └── css/
│       └── style.css      # App styling
├── templates/
│   ├── base.html           # Base layout, loads Socket.IO client
│   ├── home.html           # Landing page: create / join a room
│   └── room.html           # Chat room UI + client-side socket + decrypt logic
├── main.py                 # Flask app, routes, and Socket.IO event handlers
├── requirements.txt        # Python dependencies
├── liara.json                # Liara deployment config
└── vercel.json               # Vercel deployment config
```

---

## How It Works

1. **Home page (`/`)** — A user enters a name and either creates a new room (generates a unique 4-character code) or joins an existing one using a room code. Room state is stored in the `rooms_collection` MongoDB collection.
2. **Room page (`/room`)** — Once inside a room, the client connects via Socket.IO.
   - When a user sends a message, the server **encrypts** it with the Rail Fence cipher (`railFenceKey = 3`) before broadcasting it to everyone in the room and saving it to MongoDB.
   - Each connected client receives the encrypted message over the socket and calls the **`/decrypt`** endpoint (a POST request with the cipher text and key) to retrieve the plaintext before rendering it in the chat window.
3. **Join/Leave events** — When a user connects or disconnects, an (encrypted) system message is broadcast to the room and the member count in MongoDB is updated.
4. **Message history** — When a user loads a room, previously stored (encrypted) messages are fetched from MongoDB and decrypted server-side before being rendered into the page.

### The Rail Fence Cipher

Implemented in `functions/railFence.py`:

- `encryptRailFence(text, key)` — writes the plaintext in a zig-zag pattern across `key` "rails" and reads it off row by row to produce the ciphertext.
- `decryptRailFence(cipher, key)` — reconstructs the zig-zag pattern and reads the original message back out in the correct order.

---

## Getting Started

### Prerequisites

- Python 3.9+
- A MongoDB instance (local or a cloud cluster, e.g. MongoDB Atlas)

### Installation

```bash
# Clone the repository
git clone https://github.com/ZahraBehzad/NetworkSecurity-Project.git
cd encryptedChatFlask

# (Recommended) create a virtual environment
python -m venv venv
source venv/bin/activate   # on Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### Configuration

In `main.py`, update the MongoDB connection string with your own:

```python
client = MongoClient("your-mongodb-connection-string")
```

It's recommended to load this (and the Flask `SECRET_KEY`) from an environment variable rather than hardcoding it, e.g.:

```python
import os
client = MongoClient(os.environ.get("MONGODB_URI"))
app.config["SECRET_KEY"] = os.environ.get("SECRET_KEY", "a-random-secret-key")
```

### Running the App

```bash
python main.py
```

By default, this starts the Flask-SocketIO development server (with debug mode on). Open your browser at:

```
http://127.0.0.1:5000
```

Open the same URL in a second browser tab/window, join the same room code, and start chatting to see the encryption/decryption pipeline in action (check your browser console / Network tab to see the raw ciphertext being exchanged before decryption).

---

## Deployment

This repository includes ready-made configs for two deployment targets:

- **Vercel** — `vercel.json` routes all traffic to `main.py` via `@vercel/python`.
- **Liara** — `liara.json` points to the Flask app module (`main:app`).

Remember to set your MongoDB connection string and Flask secret key as environment variables on whichever platform you deploy to, rather than committing them to source control.

---

## Course Context

This project was developed as part of the **Network Security** course to explore:

- Client-server real-time communication (WebSockets vs. HTTP polling)
- Symmetric encryption concepts using a classical cipher
- The security trade-offs of transposition ciphers (small key space, no diffusion of individual characters, susceptibility to brute-force/frequency analysis)
- Practical considerations for storing and transmitting "encrypted" data in a web application
