# Slack Clone - Dioxus & Rust Implementation Plan

## Tech Stack

| Layer | Technology |
|---|---|
| **Frontend** | Dioxus 0.7 (web target) with RSX macro |
| **Styling** | Tailwind CSS v4 |
| **Backend** | Dioxus fullstack with server functions + Axum |
| **Real-time** | WebSockets (tokio-tungstenite) |
| **Database** | SQLite via SQLx (easy dev setup; swap to Postgres later) |
| **Auth** | Session-based auth with argon2 password hashing |

## Project Structure

```
slack-clone/
├── Cargo.toml
├── Dioxus.toml
├── input.css                  # Tailwind entry
├── assets/
│   └── tailwind.css           # Generated
├── src/
│   ├── main.rs                # Launch config
│   ├── app.rs                 # Root App component + router
│   ├── models/
│   │   ├── mod.rs
│   │   ├── user.rs            # User struct
│   │   ├── channel.rs         # Channel struct
│   │   └── message.rs         # Message struct
│   ├── components/
│   │   ├── mod.rs
│   │   ├── sidebar.rs         # Workspace/channel list
│   │   ├── channel_header.rs  # Channel name, topic, members
│   │   ├── message_list.rs    # Scrollable message feed
│   │   ├── message_item.rs    # Single message bubble
│   │   ├── message_input.rs   # Compose bar
│   │   └── user_avatar.rs     # Avatar component
│   ├── pages/
│   │   ├── mod.rs
│   │   ├── login.rs           # Login page
│   │   ├── register.rs        # Register page
│   │   └── workspace.rs       # Main workspace layout
│   ├── server/
│   │   ├── mod.rs
│   │   ├── auth.rs            # Server functions: login, register, session
│   │   ├── channels.rs        # Server functions: CRUD channels
│   │   ├── messages.rs        # Server functions: send/fetch messages
│   │   └── ws.rs              # WebSocket handler for real-time
│   └── state.rs               # Global app state (signals/context)
├── migrations/
│   ├── 001_create_users.sql
│   ├── 002_create_channels.sql
│   └── 003_create_messages.sql
```

## Implementation Phases

### Phase 1: Project Scaffolding & Auth
1. **Initialize project** — `dx new slack-clone` with web+fullstack template
2. **Set up Tailwind CSS v4** — input.css, tailwind watcher, asset linking
3. **Database setup** — SQLx + SQLite, migration files for users/channels/messages tables
4. **Auth server functions** — `register`, `login`, `logout`, `get_current_user` using Dioxus server functions
5. **Login/Register pages** — Forms with validation, route guards for protected pages
6. **Session state** — Store current user in a signal/context provider at the app root

### Phase 2: Channels & Static Messaging
7. **Sidebar component** — List channels, highlight active channel, "create channel" button
8. **Create channel** — Modal/form to name a new channel, server function to persist
9. **Workspace layout** — Three-column layout: sidebar | message list | (future: thread panel)
10. **Message list** — Fetch and display messages for the active channel via server function
11. **Message input** — Compose bar with send button, calls server function to persist message
12. **Routing** — `/#/channel/:id` routes using Dioxus Router, `Outlet` for nested layout

### Phase 3: Real-time with WebSockets
13. **WebSocket server** — Axum WebSocket endpoint, broadcast messages to channel subscribers
14. **WebSocket client hook** — `use_websocket` custom hook that connects on mount, reconnects on drop
15. **Live message updates** — New messages from WS pushed into the message list signal
16. **Typing indicators** — Broadcast "user is typing" events over WS
17. **Online presence** — Track connected users, show online/offline dots on avatars

### Phase 4: Polish & Extra Features
18. **Message timestamps & grouping** — Group consecutive messages from same user, relative timestamps
19. **Markdown/rich text** — Render basic markdown in messages
20. **Emoji reactions** — Click to react, show reaction counts
21. **Thread replies** — Side panel for threaded conversations
22. **File uploads** — Upload images/files, store in local filesystem or S3
23. **Search** — Full-text search across messages using SQLite FTS5
24. **Unread indicators** — Track last-read message per user per channel

## Key Database Schema

```sql
-- users
CREATE TABLE users (
    id TEXT PRIMARY KEY,           -- UUID
    username TEXT UNIQUE NOT NULL,
    display_name TEXT NOT NULL,
    password_hash TEXT NOT NULL,
    avatar_url TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- channels
CREATE TABLE channels (
    id TEXT PRIMARY KEY,
    name TEXT UNIQUE NOT NULL,
    topic TEXT DEFAULT '',
    created_by TEXT REFERENCES users(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- messages
CREATE TABLE messages (
    id TEXT PRIMARY KEY,
    channel_id TEXT REFERENCES channels(id) NOT NULL,
    user_id TEXT REFERENCES users(id) NOT NULL,
    content TEXT NOT NULL,
    parent_id TEXT REFERENCES messages(id),  -- for threads
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- channel_members
CREATE TABLE channel_members (
    channel_id TEXT REFERENCES channels(id),
    user_id TEXT REFERENCES users(id),
    last_read_at TIMESTAMP,
    PRIMARY KEY (channel_id, user_id)
);
```

## Key Dioxus Patterns Used

- **Server functions** (`#[server]`) for all backend calls — no separate API layer needed
- **Signals** (`use_signal`) for local component state (input fields, toggles)
- **Context** (`use_context_provider` / `use_context`) for global state (current user, active channel)
- **Router** with `#[derive(Routable)]` enum for page navigation
- **`rsx!` macro** for declarative UI with Tailwind classes
- **Custom hooks** (`use_websocket`) for encapsulating WebSocket logic
- **`use_server_future`** for loading data on component mount

## Commands to Run

```bash
# Terminal 1: Dioxus dev server
dx serve

# Terminal 2: Tailwind watcher
npx @tailwindcss/cli -i ./input.css -o ./assets/tailwind.css --watch

# Run migrations
sqlx migrate run
```
