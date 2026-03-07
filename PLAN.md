# Slack Clone - Freya + Rust Implementation Plan

## Overview
Build a desktop Slack clone using **Freya** (cross-platform Rust GUI powered by Dioxus + Skia). The app will feature real-time messaging with channels, direct messages, and a familiar Slack-like layout.

---

## Architecture

```
slack-clone/
├── Cargo.toml
├── src/
│   ├── main.rs                  # App entry, launch config, router setup
│   ├── state/
│   │   ├── mod.rs
│   │   ├── app_state.rs         # Global app state (current user, active workspace)
│   │   ├── channels.rs          # Channel list, active channel, unread counts
│   │   ├── messages.rs          # Message store, message operations
│   │   └── users.rs             # User profiles, online status
│   ├── components/
│   │   ├── mod.rs
│   │   ├── sidebar/
│   │   │   ├── mod.rs
│   │   │   ├── workspace_header.rs   # Workspace name + dropdown
│   │   │   ├── channel_list.rs       # #channel list with unread badges
│   │   │   ├── dm_list.rs            # Direct message user list
│   │   │   └── channel_item.rs       # Single channel row
│   │   ├── chat/
│   │   │   ├── mod.rs
│   │   │   ├── chat_header.rs        # Channel name, topic, member count
│   │   │   ├── message_list.rs       # Virtualized scrolling message list
│   │   │   ├── message_item.rs       # Single message (avatar, name, time, text)
│   │   │   ├── message_input.rs      # Text input + send button
│   │   │   └── message_group.rs      # Grouped messages by same author
│   │   ├── common/
│   │   │   ├── mod.rs
│   │   │   ├── avatar.rs             # User avatar circle
│   │   │   ├── badge.rs              # Unread count badge
│   │   │   ├── divider.rs            # Horizontal separator
│   │   │   └── tooltip.rs            # Hover tooltip
│   │   └── thread/
│   │       ├── mod.rs
│   │       └── thread_panel.rs       # Thread side panel
│   ├── views/
│   │   ├── mod.rs
│   │   ├── main_view.rs         # Primary layout: sidebar + chat area
│   │   └── login_view.rs        # Simple login/user select screen
│   ├── models/
│   │   ├── mod.rs
│   │   ├── message.rs           # Message struct (id, author, content, timestamp, reactions)
│   │   ├── channel.rs           # Channel struct (id, name, topic, members, is_dm)
│   │   └── user.rs              # User struct (id, name, avatar_url, status)
│   └── theme.rs                 # Colors, fonts, spacing constants (Slack-like theme)
```

---

## Implementation Phases

### Phase 1: Project Setup & Theme
**Goal:** Scaffold the project, define data models, and establish the Slack color theme.

1. Initialize Cargo project with Freya dependencies:
   ```toml
   [dependencies]
   freya = { version = "0.3", features = ["router"] }
   dioxus = { version = "0.6", features = ["hooks"] }
   uuid = { version = "1", features = ["v4"] }
   chrono = "0.4"
   ```
2. Define the Slack-inspired color theme in `theme.rs`:
   - Sidebar dark purple: `#3F0E40` / `#4A154B`
   - Chat background: `#FFFFFF`
   - Message hover: `#F8F8F8`
   - Active channel: `#1164A3`
   - Text colors, spacing, font sizes
3. Create data models (`Message`, `Channel`, `User`) with builder patterns

### Phase 2: Layout Shell
**Goal:** Build the two-panel Slack layout (sidebar + main chat area).

1. Create `main_view.rs` — horizontal split layout:
   - Left sidebar: fixed ~260px width, dark background
   - Right chat area: fills remaining space
2. Implement `workspace_header.rs` — workspace name at top of sidebar
3. Stub out `channel_list.rs` with hardcoded channels
4. Stub out `chat_header.rs` with channel name display

### Phase 3: State Management
**Goal:** Wire up reactive state for channels, messages, and users.

1. Build `app_state.rs` using Dioxus signals/context:
   - `use_context_provider` for global state at app root
   - `use_context` to consume state in child components
2. Implement `channels.rs` state:
   - `Vec<Channel>` with active channel tracking
   - Channel switching updates active channel signal
   - Unread count per channel
3. Implement `messages.rs` state:
   - `HashMap<ChannelId, Vec<Message>>` message store
   - Add message, load messages per channel
4. Implement `users.rs` state:
   - Current user identity
   - User lookup by ID
5. Seed with demo data (3-4 channels, ~10 messages, 3-4 users)

### Phase 4: Channel Sidebar
**Goal:** Fully functional sidebar with channel navigation.

1. Implement `channel_list.rs`:
   - Render channels with `#` prefix icon
   - Highlight active channel
   - Show unread badge via `badge.rs`
   - `on_press` switches active channel
2. Implement `dm_list.rs`:
   - Show user avatars + names
   - Online/offline status indicator (green dot)
3. Add section headers ("Channels", "Direct Messages") with collapse toggle

### Phase 5: Message Display
**Goal:** Show messages in the chat area with proper formatting.

1. Implement `message_list.rs`:
   - Use Freya's `VirtualScrollView` for performant scrolling
   - Auto-scroll to bottom on new messages
   - Filter messages by active channel
2. Implement `message_item.rs`:
   - Avatar (colored circle with initials) on the left
   - Author name (bold) + timestamp (gray, smaller) on first line
   - Message text below
3. Implement `message_group.rs`:
   - Group consecutive messages by same author
   - Only show avatar/name on first message in group
4. Add date dividers between message groups from different days

### Phase 6: Message Input & Sending
**Goal:** Compose and send messages.

1. Implement `message_input.rs`:
   - Multi-line text input area at bottom of chat
   - Placeholder text: "Message #channel-name"
   - Send on Enter (Shift+Enter for newline)
   - Send button on the right
2. Wire input to state:
   - On send, create `Message` with current user + timestamp
   - Push to message store for active channel
   - Clear input field
   - Scroll message list to bottom

### Phase 7: Thread Panel (Stretch)
**Goal:** Side panel for threaded replies.

1. Implement `thread_panel.rs`:
   - Slides in from right side when a message is clicked
   - Shows parent message + replies below
   - Own message input for thread replies
2. Add reply count indicator on messages that have threads

### Phase 8: Polish & Extras
**Goal:** Make it feel like Slack.

1. **Hover states**: Message background changes on hover, action buttons appear
2. **Reactions**: Click to add emoji reactions to messages (simplified set)
3. **User presence**: Green/gray dots for online/offline
4. **Channel creation**: Simple modal dialog to add a new channel
5. **Search bar**: Filter messages by keyword (local only)
6. **Keyboard shortcuts**: Ctrl+K for quick channel switcher

---

## Key Technical Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| State management | Dioxus signals + context | Built-in, reactive, no extra deps |
| Message scrolling | `VirtualScrollView` | Handles large message lists efficiently |
| Data persistence | In-memory only (Phase 1) | Keep scope manageable, add SQLite later |
| Networking | None (local mock data) | Focus on UI first, add WebSocket later |
| ID generation | `uuid::v4` | Simple, unique message/channel IDs |
| Timestamps | `chrono` | Formatting "2:34 PM", "Yesterday", etc. |

---

## Phases Summary

| Phase | Deliverable | Estimated Complexity |
|-------|------------|---------------------|
| 1 | Project scaffold + theme + models | Low |
| 2 | Two-panel layout shell | Low |
| 3 | Reactive state with demo data | Medium |
| 4 | Interactive channel sidebar | Medium |
| 5 | Virtualized message display | Medium |
| 6 | Message input + sending | Medium |
| 7 | Thread panel | High |
| 8 | Polish, reactions, search | High |

Phases 1-6 deliver a functional Slack clone MVP. Phases 7-8 add depth.
