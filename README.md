# Encrypted Chat Flask

A real-time chat room application built with Flask and Flask-SocketIO. Users can create or join chat rooms with a short room code, and all messages are encrypted with a Rail Fence cipher before being broadcast and stored, then decrypted client-side for display.

## Features

- **Create or join rooms** — generate a unique 4-character room code or join an existing one
- **Real-time messaging** via WebSockets (Flask-SocketIO)
- **Rail Fence cipher encryption** — messages are encrypted on the server before being sent to clients, and decrypted in the browser via a `/decrypt` endpoint
- **Persistent chat history** — rooms and messages are stored in MongoDB, so message history reloads when a room is revisited
- **Join/leave notifications** and live member counts per room

## Tech Stack

- **Backend:** Flask, Flask-SocketIO
- **Database:** MongoDB (via PyMongo)
- **Frontend:** Jinja2 templates, vanilla JS, jQuery, Socket.IO client
- **Deployment:** configured for both Vercel (`vercel.json`) and Liara (`liara.json`)

## Project Structure

```
encryptedChatFlask/
├── main.py                  # Flask app, routes, and SocketIO event handlers
├── functions/
│   └── railFence.py         # Rail Fence cipher encrypt/decrypt implementation
├── templates/
│   ├── base.html            # Base layout
│   ├── home.html            # Landing page (create/join room)
│   └── room.html            # Chat room UI + client-side socket logic
├── static/
│   └── css/style.css        # Styling
├── requirements.txt         # Python dependencies
├── vercel.json               # Vercel deployment config
└── liara.json                 # Liara deployment config
```

## Setup

### Prerequisites

- Python 3.11+
- A MongoDB instance (e.g. a free [MongoDB Atlas](https://www.mongodb.com/atlas) cluster)

### Installation

1. Clone the repository:
   ```bash
   git clone <your-repo-url>
   cd encryptedChatFlask
   ```

2. Create a virtual environment and install dependencies:
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   pip install -r requirements.txt
   ```

3. Configure environment variables. The app currently reads its MongoDB URI and Flask secret key directly from `main.py` — **move these to environment variables before deploying anywhere public** (see [Security Notes](#security-notes) below). For example:
   ```bash
   export MONGODB_URI="your-mongodb-connection-string"
   export FLASK_SECRET_KEY="a-random-secret-key"
   ```
   and update `main.py` to read them with `os.environ.get(...)`.

4. Run the app:
   ```bash
   python main.py
   ```

5. Open `http://localhost:5000` in your browser.

## How It Works

1. A user enters a name and either creates a new room (generating a random 4-letter code) or joins an existing room by code.
2. Messages sent in a room are encrypted server-side with the Rail Fence cipher before being broadcast over the socket connection and saved to MongoDB.
3. The client receives the encrypted message and calls the `/decrypt` endpoint, which returns the plaintext for display.
4. Room membership counts and message history are tracked per room in the `rooms` MongoDB collection.

## Security Notes

This project is a learning/demo implementation, not production-hardened. Before deploying it or sharing the repo publicly, consider:

- **Move all secrets out of source code.** `main.py` currently contains a hardcoded MongoDB connection string (with credentials) and a hardcoded Flask `SECRET_KEY`. Rotate that database password immediately if this code has ever been pushed to a public repo, and load both values from environment variables instead.
- **Rail Fence is a classical/educational cipher**, not cryptographically secure — it's easily broken and shouldn't be relied on to protect sensitive data. It's well suited for demonstrating the encrypt/decrypt flow, not for real confidentiality.
- Consider adding input validation/sanitization on messages and room codes before rendering them, to avoid injection issues.

## License

Add a license of your choice (e.g. MIT) here.
