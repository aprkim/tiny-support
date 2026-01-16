# Bridge.js API Guide (version 0.3 January 15, 2026 7:50am PT)

## Overview

Bridge.js is a JavaScript wrapper class that provides a unified API for building real-time video/audio collaboration applications. It abstracts three major subsystems:

1. **Database operations** - User, channel, and membership management
2. **Real-time updates** - WebSocket-based pub/sub for live data synchronization
3. **Media streaming** - WebRTC video/audio streaming via SFU (Selective Forwarding Unit)

### Supported Media Types

- **streamTypes**: `"camera"`, `"screencast"`
- **trackTypes**: `"audio"`, `"video"`

---

## Common Use Cases

Bridge.js supports several distinct collaboration patterns. Choose the approach that matches your application's needs:

### Use Case 1: Quick Meet Rooms (Anonymous, Code-Based)
**Best for**: Simple video meetings, temporary collaboration, sharable room links

**Key Features**:
- No login required - users join as anonymous guests
- Room codes for easy sharing (just send the channel.pid)
- Short-lived sessions without persistence
- Minimal UI complexity

**Implementation Pattern**:
1. Auto-login as guest using `signupAnonInContext()` 
2. Either create a new channel with `createNewChannel()` or join existing with `addChannelMember()`
3. Display channel.pid as shareable room code
4. Setup media server and start camera/audio
5. Handle incoming streams with `setOnNewDownstream()`

**Example**: See `meetme.html` for complete implementation

**Sections to Study**: 1 (signupAnonInContext), 2 (createNewChannel, addChannelMember), 3 (setupServer), 4 (startMedia), 5 (startUpstream)

---

### Use Case 2: Direct User Connections (1-to-1 Quick Chat)
**Best for**: Instant messaging with video, user directory browsing, spontaneous calls

**Key Features**:
- User login with persistent accounts
- Browse and search user directories
- Create ephemeral 1-to-1 channels on demand
- Automatic invitation and notification system

**Implementation Pattern**:
1. Login with `login()` using email/password
2. Display user list via `getManyUsers()` with search
3. Select target user and call `createQuickChatChannel(invited: userPid)`
4. Listen for channel events with `subscribeToChannelTagEvents()`
5. Handle invitation acceptance on remote side
6. Setup media and exchange streams

**Example**: See `qc_demo.html` for complete implementation

**Sections to Study**: 1 (login), 2 (getManyUsers, createQuickChatChannel), 3 (subscribeToChannelTagEvents), 4 (startMedia), 5 (startUpstream)

---

### Use Case 3: Persistent Channel Rooms
**Best for**: Team workspaces, recurring meetings, organized collaboration spaces

**Key Features**:
- User login with persistent accounts
- Browse existing channels/rooms
- Persistent membership management
- Channel discovery and joining

**Implementation Pattern**:
1. Login with `login()` using email/password
2. Display channel list via `getManyChannels()` with filtering
3. Join channel with `addChannelMember(channelPid, userPid)`
4. Subscribe to channel events with `subscribeToChannelTagEvents()`
5. Setup media and exchange streams with all members

**Sections to Study**: 1 (login), 2 (getManyChannels, addChannelMember), 3 (subscribeToChannelTagEvents), 4 (startMedia), 5 (startUpstream)

---

### Use Case 4: Invitation-Driven Connections
**Best for**: Reactive collaboration, notification-based joins, controlled access

**Key Features**:
- User login with persistent accounts
- Real-time invitation notifications
- Accept/reject flow for incoming requests
- Persistent channel history

**Implementation Pattern**:
1. Login with `login()` and subscribe to invite notifications
2. Setup tag event listener for 'quick_chat_invite_accept' or similar tags
3. When invite received, display notification UI to user
4. On accept, join channel with `addChannelMember()`
5. On reject, optionally remove invite tag or ignore
6. Setup media after accepting invitation

**Example**: Invitation handling in `qc_demo.html`

**Sections to Study**: 1 (login), 2 (addChannelMember), 3 (subscribeToTagTreeNodeEvents for invites), 4 (startMedia), 5 (startUpstream)

---

## Architecture

Bridge acts as a facade over:
- **Fetch** - RESTful API calls for database operations
- **PubSub** - WebSocket dispatcher for real-time events
- **ChannelComms** - WebRTC peer connection management

---

## Setup Requirements

### Import Map Configuration

Bridge.js requires an **import map** to resolve module dependencies. You must include this in your HTML file before importing Bridge.

**Standard Configuration (Recommended):**

Use the hosted Bridge.js server - most developers will use this:

```html
<script type="importmap">
{
  "imports": {
    "bridge": "https://proto2.makedo.com:8883/v01/scripts/bridge.js",
    "config": "https://proto2.makedo.com:8883/v01/scripts/configServer.js"
  }
}
</script>
```

**Important Notes:**

1. **Both modules required**: Bridge requires both the `bridge` and `config` imports to function properly.

2. **Use the server URLs**: Unless you are hosting your own SFU (Selective Forwarding Unit) media server and Bridge.js instance, use the URLs shown above. Most developers do not need to host their own infrastructure.

3. **Local development alternative** (only if you're hosting Bridge.js yourself):
   ```html
   <script type="importmap">
   {
     "imports": {
       "bridge": "./scripts/bridge.js",
       "config": "./scripts/configLocal.js"
     }
   }
   </script>
   ```

4. **Place before script imports**: The import map must be defined before any `<script type="module">` tags that import Bridge.

### Basic HTML Template

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Bridge App</title>
    
    <!-- Import Map - Must come first -->
    <script type="importmap">
    {
      "imports": {
        "bridge": "https://proto2.makedo.com:8883/v01/scripts/bridge.js",
        "config": "https://proto2.makedo.com:8883/v01/scripts/configServer.js"
      }
    }
    </script>
</head>
<body>
    <!-- Your UI here -->
    
    <!-- Your application script -->
    <script type="module">
        import Bridge from 'bridge';
        
        const bridge = new Bridge();
        // ... your code
    </script>
</body>
</html>
```

---

## Quick Reference: Critical Method Signatures

This section provides the exact method signatures for the most commonly used Bridge.js functions. Refer to detailed sections below for complete documentation.

### Server Connection (Section 3)

```javascript
// Setup server - 3 required parameters
bridge.setupServer(
    channelId: string,
    memberId: string,
    accessCode: string
)

// Join server
await bridge.joinServer()

// Leave server
bridge.leaveServer(killStreams: boolean)
```

### Media Management (Section 4)

```javascript
// Start local media
await bridge.startMedia(
    streamType: string,      // 'camera' or 'screencast'
    audioOn: boolean,
    videoOn: boolean,
    deviceId?: string
) : Promise<MediaStream | null>

// Stop local media
await bridge.stopMedia(
    streamType: string,      // 'camera' or 'screencast'
    trackType?: string       // 'video', 'audio', or omit for all
)

// Get current media
bridge.getMedia(streamType: string) : MediaStream | null
```

### Broadcasting (Section 5)

```javascript
// Start broadcasting
await bridge.startUpstream(
    streamType: string,      // 'camera' or 'screencast'
    trackType?: string       // 'video', 'audio', or omit for all
)

// Stop broadcasting
await bridge.stopUpstream(
    streamType: string,
    trackType?: string
)

// Mute/unmute
await bridge.setMutedUpstream(
    streamType: string,
    trackType: string,       // 'video', 'audio', or 'all'
    muted: boolean
)
```

### Event Callbacks (Section 3)

```javascript
// Incoming stream from another participant
bridge.setOnNewDownstream((
    streamId: string,
    memberId: string,
    type: string,            // 'camera' or 'screencast'
    stream: MediaStream | null
) => { /* ... */ })

// Stream ended from another participant
bridge.setOnEndDownstream((
    streamId: string,
    memberId: string,
    type: string
) => { /* ... */ })

// Your stream started broadcasting
bridge.setOnNewUpstream((
    streamId: string,
    memberId: string,
    type: string,
    stream: MediaStream
) => { /* ... */ })

// Server connection events
bridge.setOnJoinedServer(() => { /* ... */ })
bridge.setOnExitedServer(() => { /* ... */ })
```

### Channel & Member Management (Section 2)

```javascript
// Get or create membership
await bridge.getOrCreateSelfMember({
    channel_id: string
}) : Promise<Member>

// Get channel data
await bridge.getOneChannel({
    channel_id: string,
    depth?: number
}) : Promise<Channel>

// Create new channel
await bridge.createNewChannel({
    title: string,
    description?: string,
    allows_guests?: boolean,
    max_size?: number
}) : Promise<Channel>

// Send invite to user
await bridge.sendInvite({
    channel_id: string,
    user_id: string
}) : Promise<any>
```

### Invite Handling (Section 2)

```javascript
// Listen for incoming invites (WebSocket)
bridge.setOnInviteUpdate((inviteData) => {
    // inviteData contains:
    // - type: 'quickchat' or 'channel_invite'
    // - channel_id: string
    // - username: string
    // - requestor_id: string
    // - message: string (optional)
    
    console.log(`${inviteData.username} invited you`);
})

// Send invite to another user
await bridge.sendInvite({
    channel_id: 'channel_xyz',
    user_id: 'user_abc123'
})
```

---

## Section 0: Basics

### `hasComms()`

Check if the media server connection has been established.

**Returns**: `boolean`

```javascript
const bridge = new Bridge();

if (bridge.hasComms()) {
    console.log('Media server is ready');
} else {
    console.log('No media server connection - call setupServer() first');
}
```

**When to use**: Before any media operations (Section 4 & 5) to ensure the communication channel exists.

---

## Section 1: Login/Logout/Self Profile

### Authentication Flow

```javascript
const bridge = new Bridge();

// Login
const loginResult = await bridge.login({
    email: 'user@example.com',
    password: 'securePassword'
});

// Response structure:
// {
//     status: 'loggedIn',
//     email: 'user@example.com',
//     pid: 'user_abc123',
//     message: 'Login successful'
// }

if (loginResult.status === 'loggedIn') {
    console.log('Logged in as:', loginResult.email);
    console.log('User ID:', loginResult.pid);
    
    // WebSocket dispatcher is automatically initialized on successful login
}
```

### Get Self Profile

```javascript
// Retrieve logged-in user's profile data
const selfData = await bridge.getSelf({ depth: 2 });

// Response includes:
// {
//     pid: 'user_abc123',
//     email: 'user@example.com',
//     username: 'JohnDoe',
//     status: 'active',
//     current_context_id: 'ctx_xyz',
//     current_context_title: 'My Workspace',
//     config: { ... },
//     tag_tree: { ... }
// }

console.log('Current context:', selfData.current_context_title);
console.log('User status:', selfData.status);
```

### Signup Variants

#### Obtaining Context Credentials

**REQUIRED FOR DEVELOPMENT**: Before you can use Bridge.js, you must obtain context credentials from MakeDo.

**To get your credentials:**

1. Visit **[makedo.com/developers](https://makedo.com/developers)**
2. Request developer access through the contact form or developer portal
3. You will receive:
   - `contextId` - Your unique workspace identifier
   - `contextAuthToken` - Your authentication token for that context

**Important Notes:**
- Context credentials are required for all signup operations
- Keep your `contextAuthToken` secure - treat it like a password
- Development and production contexts may be separate
- Contact support at makedo.com if you have questions about your credentials

---

#### Context Requirement

**IMPORTANT DESIGN NOTE**: Bridge.js **requires a valid context** (workspace/organization) for all signups. The basic `signup()` function without context has been removed. All users must be created within a context using either `signupInContext()` or `signupAnonInContext()`.

#### Auto-Login Behavior

**CRITICAL**: Both signup functions automatically log the user in. You do **NOT** need to call `login()` after signup. The response includes `status: 'loggedIn'` and the user's data.

```javascript
// Signup with email (requires valid email address)
const result = await bridge.signupInContext({
    contextId: 'YOUR_CONTEXT_ID',              // Required: Obtain from makedo.com/developers
    contextAuthToken: 'YOUR_CONTEXT_AUTH_TOKEN',       // Required: Obtain from makedo.com/developers
    email: 'newuser@example.com',
    password: 'securePassword'
});

// Response structure:
// {
//     status: 'loggedIn',              // â† Already logged in!
//     email: 'newuser@example.com',
//     pid: 'user_abc456',
//     ...
// }

if (result.status === 'loggedIn') {
    console.log('Signed up and auto-logged in as:', result.email);
    // Ready to use Bridge - no need to call login()
}

// Anonymous signup with username only (guest users)
const anonResult = await bridge.signupAnonInContext({
    contextId: 'YOUR_CONTEXT_ID',              // Required: Obtain from makedo.com/developers
    contextAuthToken: 'YOUR_CONTEXT_AUTH_TOKEN',       // Required: Obtain from makedo.com/developers
    username: 'GuestUser42',
    password: 'tempPassword'
});

// Response structure:
// {
//     status: 'loggedIn',              // â† Already logged in!
//     pre_email_id: 'anon_xyz123',     // â† Use this instead of email for future logins
//     pid: 'user_abc456',
//     ...
// }

if (anonResult.status === 'loggedIn') {
    console.log('Anonymous user created and auto-logged in:', anonResult.pre_email_id);
    // Store pre_email_id if user needs to login again later
    localStorage.setItem('guestId', anonResult.pre_email_id);
}
```

#### Logging In Existing Users

**For returning users** (not new signups), use `login()`:

```javascript
// Login with email
const loginResult = await bridge.login({
    email: 'user@example.com',
    password: 'securePassword'
});

// Login with pre_email_id (anonymous users)
const guestId = localStorage.getItem('guestId');
const guestLogin = await bridge.login({
    email: guestId,           // Use pre_email_id as "email"
    password: 'tempPassword',
    contextId: 'YOUR_CONTEXT_ID'  // Must provide contextId for guest users
});
```

**Note on Anonymous Users**: Anonymous signup creates a guest account without requiring an email address. The returned `pre_email_id` serves as a unique identifier that you must use in place of an email address when logging in. Store this value if you need the guest user to log in again later. When logging in as an anonymous user, you **must provide the contextId** they were created in.

#### Simple Use Case: Offline Channel Sharing

A common pattern is to let an anonymous user create a channel and share the channel ID manually (via text, email, etc.) with friends:

```javascript
// User A: Create anonymous account and channel
const bridge = new Bridge();

// 1. Signup as guest (auto-logged in)
const user = await bridge.signupAnonInContext({
    contextId: 'YOUR_CONTEXT_ID',
    contextAuthToken: 'YOUR_CONTEXT_AUTH_TOKEN',
    username: 'Alice',
    password: 'temp123'
});
// user.status === 'loggedIn' - ready to go!

// 2. Create a new channel
const channel = await bridge.createNewChannel({
    title: 'Video Chat Room',
    allows_guests: true
});

// 3. Display channel ID to share
console.log('Share this code with friends:', channel.pid);
// Example: "abc123xyz"

// User B receives the code and joins:
const bridgeB = new Bridge();

// 1. Signup or login
const userB = await bridge.signupAnonInContext({
    contextId: 'YOUR_CONTEXT_ID',  // Same context as User A
    contextAuthToken: 'YOUR_CONTEXT_AUTH_TOKEN',
    username: 'Bob',
    password: 'temp456'
});
// userB.status === 'loggedIn'

// 2. Join the channel using the shared code
const channelId = 'abc123xyz'; // Received from User A
const memberB = await bridgeB.getOrCreateSelfMember({
    channel_id: channelId
});

// Both users are now in the same channel and can start media
```

### Update Self Profile

```javascript
// Update your own profile settings
await bridge.updateSelf({
    config: {
        display_name: 'John Smith',
        theme: 'dark',
        notifications_enabled: true
    }
});
```

### Logout

```javascript
// Logout and close WebSocket connection
await bridge.logout({});

// WebSocket dispatcher is automatically disconnected
```

---

## Section 2: Context Data / Event Callbacks

This section manages database objects (users, channels, members, invites) and subscribes to real-time updates.

### User Management

#### Get Users

```javascript
// Search for users by query string
const users = await bridge.getUsers({
    query: 'john',      // Search term (searches username, email, etc.)
    count: 20,          // Max results to return
    depth: 1            // Data depth (1=basic, 2=includes tags, etc.)
});

// Response: Array of user objects
// [
//     {
//         pid: 'user_abc123',
//         username: 'JohnDoe',
//         email: 'john@example.com',
//         status: 'online',
//         tag_tree: { pinned: true, ... }
//     },
//     ...
// ]

users.forEach(user => {
    console.log(`${user.username} (${user.email})`);
});
```

#### Get Single User

```javascript
const user = await bridge.getOneUser({
    user_id: 'user_abc123',
    depth: 2
});

console.log('User:', user.username);
console.log('Tags:', user.tag_tree);
```

#### Listen for User Updates (WebSocket)

```javascript
// Set callback for real-time user updates
bridge.setOnUserUpdate((message) => {
    const userId = message.id_map?.user;
    const event = message.event;  // 'update', 'status_change', etc.
    const delta = message.delta;  // Changed fields
    
    console.log(`User ${userId} ${event}:`, delta);
    
    // Example: Update UI when user goes online/offline
    if (delta.status) {
        updateUserStatusInUI(userId, delta.status);
    }
});
```

### Channel Management

#### Get Channels

```javascript
// Get channels you're a member of
const channels = await bridge.getChannels({
    count: 10,
    depth: 2,                    // depth=2 includes member list
    sortField: 'uts',            // Sort by update timestamp
    sortOrder: 'desc'            // Newest first
});

// Response: Array of channel objects
// [
//     {
//         pid: 'channel_xyz',
//         title: 'Team Meeting',
//         description: 'Daily standup',
//         max_size: 10,
//         members: [ { pid: 'member_123', ... }, ... ],
//         tag_tree: { pinned: true }
//     },
//     ...
// ]

channels.forEach(channel => {
    console.log(`${channel.title}: ${channel.members.length} members`);
});
```

#### Get Single Channel

```javascript
const channel = await bridge.getOneChannel({
    channel_id: 'channel_xyz',
    depth: 2  // Include member list
});

console.log('Channel:', channel.title);
console.log('Members:', channel.members.length);
console.log('Access code:', channel.access_code);
```

#### Create New Channel

```javascript
// Create a structured channel
const newChannel = await bridge.createNewChannel({
    title: 'Project Planning',
    description: 'Q1 2026 Planning',
    tone: 'professional',
    max_size: 15,
    allows_guests: false
});

console.log('Created channel:', newChannel.pid);
```

#### Create Quick Chat

```javascript
// Create instant 1-on-1 or small group chat
const quickChat = await bridge.createQuickChatChannel({
    invited: 'user_abc123',
    message: 'Hey, want to video chat?'  // Optional invite message
});

// Response:
// {
//     pid: 'channel_xyz',
//     title: 'Quick Chat',
//     access_code: '413239',
//     ...
// }

console.log('Quick chat created:', quickChat.pid);
```

#### Listen for Channel Updates (WebSocket)

```javascript
bridge.setOnChannelUpdate((message) => {
    const channelId = message.id_map?.channel;
    const event = message.event;
    const delta = message.delta;
    
    console.log(`Channel ${channelId} ${event}:`, delta);
    
    // Example: Update UI when channel settings change
    if (delta.title) {
        updateChannelTitleInUI(channelId, delta.title);
    }
});
```

### Member Management

Members represent a user's participation in a specific channel.

#### Get or Create Self Member

**CRITICAL**: You **must** call `getOrCreateSelfMember()` **before** `setupServer()` because `setupServer()` requires the `access_code` from your member object.

```javascript
// Get your membership in a channel (creates if doesn't exist)
const memberData = await bridge.getOrCreateSelfMember({
    channel_id: 'channel_xyz'
});

// Response:
// {
//     pid: 'member_123',
//     user_id: 'user_abc123',
//     channel_id: 'channel_xyz',
//     access_code: '413239',      // â† Required for setupServer()
//     status: 'invited',          // 'invited' | 'in_lobby' | 'live' | 'paused'
//     local_name: 'John',
//     cam_audio_state: false,
//     cam_video_state: false,
//     screen_audio_state: false,
//     screen_video_state: false
// }

console.log('Member ID:', memberData.pid);
console.log('Access code:', memberData.access_code);  // Save this for setupServer()
console.log('Status:', memberData.status);

// âš ï¸ REQUIRED FLOW:
// 1. getOrCreateSelfMember() â† Do this first
// 2. setupServer(channelId, userId, accessCode) â† Use accessCode from step 1
// 3. startMedia() â† Only after setupServer()
```

#### Get All Channel Members

```javascript
const members = await bridge.getMembers({
    channel_id: 'channel_xyz',
    depth: 1
});

members.forEach(member => {
    console.log(`${member.local_name}: ${member.status}`);
});
```

#### Listen for Member Updates (WebSocket)

```javascript
bridge.setOnMemberUpdate((message) => {
    const memberId = message.id_map?.member;
    const channelId = message.id_map?.channel;
    const event = message.event;
    const delta = message.delta;
    
    console.log(`Member ${memberId} ${event}:`, delta);
    
    // Common events:
    // 'member_joined' - Member connected to media server
    // 'member_left' - Member disconnected
    // 'member_in_lobby' - Member waiting to join
    // 'member_camera_change' - Camera media state changed
    // 'member_screencast_change' - Screencast media state changed
    // 'update' - General profile update
    
    if (event === 'member_joined') {
        console.log(`${delta.local_name} joined the channel`);
    }
});
```

#### Get Member Media States

```javascript
// Get detailed media state for a specific member
const mediaStates = bridge.getMemberMediaStates({
    member_id: 'member_123'
});

// Response:
// {
//     cam_audio_state: true,           // Has audio track
//     cam_video_state: true,           // Has video track
//     cam_audio_detail: 'ON',          // 'OFF' | 'ON' | 'MUTED'
//     cam_video_detail: 'ON',          // 'OFF' | 'ON' | 'HIDDEN'
//     screen_audio_state: false,
//     screen_video_state: false,
//     screen_audio_detail: 'OFF',
//     screen_video_detail: 'OFF'
// }

if (mediaStates.cam_video_state) {
    console.log('Member has camera:', mediaStates.cam_video_detail);
}
```

#### Listen for Member Media State Changes

```javascript
// Fires when any member's media state changes
bridge.setOnMemberMediaStateChange((memberId) => {
    const states = bridge.getMemberMediaStates({ member_id: memberId });
    
    console.log(`Member ${memberId} media changed:`, states);
    
    // Update UI indicators
    updateMemberVideoIndicator(memberId, states.cam_video_detail);
    updateMemberAudioIndicator(memberId, states.cam_audio_detail);
});
```

### Invite Management

Invites allow users to be notified of and join channels. There are two types of invites:

1. **Quick Chat Invites** (`type: 'quickchat'`) - Creates a new 1-to-1 or small group channel
2. **Channel Invites** (`type: 'channel_invite'`) - Invites to an existing channel

#### Send Invite

Use `sendInvite()` to invite a user to a channel you're currently in:

```javascript
// Send invite to specific user for current channel
const result = await bridge.sendInvite({
    channel_id: 'channel_xyz',    // The channel's pid
    user_id: 'user_abc123'        // The user's pid to invite
});

console.log('Invite sent:', result);
```

**When to Use:**
- You're already in or have access to a channel
- Want to add another user to the channel
- User will receive real-time notification via WebSocket

**Example Implementation:**

```javascript
// In a video chat app with "Invite User" button
async function showInviteModal(currentChannel) {
    // Load list of users
    const users = await bridge.getUsers({ query: '', count: 50 });
    
    // Filter out users already in channel
    const channelMembers = await bridge.getMembers({ 
        channel_id: currentChannel.pid 
    });
    const memberUserIds = channelMembers.map(m => m.user_id);
    const invitableUsers = users.filter(u => !memberUserIds.includes(u.pid));
    
    // Display users and handle invite click
    invitableUsers.forEach(user => {
        renderUserWithInviteButton(user, async () => {
            await bridge.sendInvite({
                channel_id: currentChannel.pid,
                user_id: user.pid
            });
            showNotification(`Invited ${user.username}`);
        });
    });
}
```

#### Get Pending Invites

```javascript
// Get pending invites
const invites = await bridge.getInvites({
    count: 10,
    depth: 1,
    sortField: 'uts',
    sortOrder: 'desc'
});

// Response: Array of invite objects
// [
//     {
//         pid: 'invite_xyz',
//         channel_id: 'channel_abc',
//         type: 'quickchat',
//         data: {
//             username: 'JohnDoe',
//             requestor_id: 'user_abc123',
//             message: 'Let\'s chat!'
//         }
//     },
//     ...
// ]

invites.forEach(invite => {
    console.log(`Invite from ${invite.data.username}`);
});
```

#### Listen for Invite Updates (WebSocket)

**CRITICAL**: Bridge.js sends **only the `data` field** to your callback, not the full WebSocket message. Both invite types have `channel_id` at the root level of the data object.

```javascript
bridge.setOnInviteUpdate((inviteData) => {
    // inviteData is the 'data' field from the WebSocket message
    // Both quickchat and channel_invite have same structure
    
    const inviteType = inviteData.type;           // 'quickchat' or 'channel_invite'
    const channelId = inviteData.channel_id;      // Channel to join
    const username = inviteData.username;         // Who sent the invite
    const requestorId = inviteData.requestor_id;  // Sender's user ID
    const message = inviteData.message;           // Optional custom message
    
    if (inviteType === 'quickchat') {
        // Quick chat invite - new 1-to-1 channel created
        console.log(`${username} wants to quick chat`);
        
        // Show notification with accept/decline
        showInviteNotification({
            title: `${username} invited you`,
            message: 'Quick chat invitation',
            onAccept: async () => {
                // Join the channel
                const channelData = await bridge.getOneChannel({ 
                    channel_id: channelId 
                });
                const memberData = await bridge.getOrCreateSelfMember({ 
                    channel_id: channelId 
                });
                
                // Setup and join
                bridge.setupServer(channelId, memberData.pid, memberData.access_code);
                await bridge.joinServer();
            }
        });
        
    } else if (inviteType === 'channel_invite') {
        // Channel invite - invited to existing channel
        console.log(`${username} invited you to channel: ${message}`);
        
        // Show notification with channel info
        showInviteNotification({
            title: `${username} invited you`,
            message: message || 'Join my channel',
            onAccept: async () => {
                // Join the existing channel
                const channelData = await bridge.getOneChannel({ 
                    channel_id: channelId 
                });
                const memberData = await bridge.getOrCreateSelfMember({ 
                    channel_id: channelId 
                });
                
                // Setup and join
                bridge.setupServer(channelId, memberData.pid, memberData.access_code);
                await bridge.joinServer();
            }
        });
    }
});
```

**Complete Example: Handling Both Invite Types**

```javascript
class InviteHandler {
    constructor(bridge) {
        this.bridge = bridge;
        this.pendingInvite = null;
        
        // Setup listener
        bridge.setOnInviteUpdate((inviteData) => {
            this.handleInvite(inviteData);
        });
    }
    
    handleInvite(inviteData) {
        const { type, channel_id, username, requestor_id, message } = inviteData;
        
        // Store pending invite
        this.pendingInvite = {
            type,
            channelId: channel_id,
            username,
            requestorId: requestor_id,
            message
        };
        
        // Highlight inviting user in UI (if showing user list)
        this.highlightUser(requestor_id);
        
        // Show appropriate notification based on type
        if (type === 'quickchat') {
            this.showNotification(
                `${username} invited you to quick chat`,
                'Accept to start 1-to-1 video call'
            );
        } else if (type === 'channel_invite') {
            this.showNotification(
                `${username} invited you`,
                message || 'Join channel'
            );
        }
        
        // Enable "Join" button if not currently in a channel
        if (!this.isInChannel) {
            document.getElementById('joinBtn').disabled = false;
        }
    }
    
    async acceptInvite() {
        if (!this.pendingInvite) return;
        
        const { channelId } = this.pendingInvite;
        
        try {
            // Get channel and member data
            const channelData = await this.bridge.getOneChannel({ 
                channel_id: channelId,
                depth: 2
            });
            
            const memberData = await this.bridge.getOrCreateSelfMember({ 
                channel_id: channelId 
            });
            
            // Setup server connection
            this.bridge.setupServer(
                channelData.pid, 
                memberData.pid, 
                memberData.access_code
            );
            
            // Enable camera/audio preview before joining
            await this.bridge.startMedia('camera', true, true);
            
            // Join the server to start broadcasting
            await this.bridge.joinServer();
            
            this.pendingInvite = null;
            console.log('Successfully joined invited channel');
            
        } catch (error) {
            console.error('Error accepting invite:', error);
        }
    }
    
    declineInvite() {
        this.pendingInvite = null;
        this.clearNotification();
    }
}

// Usage
const inviteHandler = new InviteHandler(bridge);
```

**Key Differences Between Invite Types:**

| Feature | Quick Chat | Channel Invite |
|---------|------------|----------------|
| Creates new channel | âœ… Yes | âŒ No (uses existing) |
| Custom message | Optional | âœ… Included |
| Typical use case | 1-to-1 spontaneous calls | Group invites, persistent rooms |
| Channel lifetime | Ephemeral | Persistent |

---

## Section 3: Server Connection / Event Callbacks

This section manages the WebRTC media server connection and stream events.

### Setup Server Connection

**CRITICAL DESIGN**: You **must** call `setupServer()` before using any local media functions (Section 4). This is required even if you only want camera/screencast preview without joining the channel.

**What setupServer() Does**: This function tells Bridge the data structure for a channel media communication session. It initializes the internal ChannelComms object that manages WebRTC connections. Without this, `startMedia()` and other media functions will fail.

**Required Flow**:
```
1. getOrCreateSelfMember() â†’ Get access_code
2. setupServer()           â†’ Initialize media communication structure
3. startMedia()            â†’ Access camera/microphone
4. joinServer()            â†’ Connect to WebRTC server
5. startUpstream()         â†’ Broadcast to other members
```

Before joining a channel, you must set up the media server connection.

**Method Signature:**
```javascript
bridge.setupServer(
    channelId: string,      // The channel's pid
    memberId: string,       // Your member's pid
    accessCode: string      // Your member's access_code
)
```

**Example Usage:**
```javascript
// After getting/creating your member object:
const memberData = await bridge.getOrCreateSelfMember({
    channel_id: 'channel_xyz'
});

const channelData = await bridge.getOneChannel({
    channel_id: 'channel_xyz'
});

// Initialize media server connection with 3 required parameters
bridge.setupServer(
    channelData.pid,              // channelId (string)
    memberData.pid,               // memberId (string)
    memberData.access_code        // accessCode (string)
);

// Now bridge.hasComms() will return true
console.log('Server setup complete, ready for media operations');
```

**âš ï¸ Important: When to Call setupServer()**

You must call `setupServer()` in these scenarios:

1. **Before local media preview** - Even if not joining the channel yet
2. **When selecting/preparing a channel** - Not when actually joining
3. **Before any `startMedia()` calls** - Camera/screencast require it

**Common Pattern:**
```javascript
// âœ“ CORRECT: Setup server when selecting channel
async function selectChannel(channelId) {
    const memberData = await bridge.getOrCreateSelfMember({ channel_id: channelId });
    const channelData = await bridge.getOneChannel({ channel_id: channelId });
    
    // Setup comms immediately
    bridge.setupServer(channelData.pid, memberData.pid, memberData.access_code);
    
    // Now camera preview works
    await bridge.startMedia('camera', true, true);
    
    // Later, join to broadcast
    await bridge.joinServer();
}

// âœ— WRONG: Setup server only when joining
async function selectChannel(channelId) {
    // Try to start camera...
    await bridge.startMedia('camera', true, true);  // ERROR! No comms object
}

async function joinChannel() {
    bridge.setupServer(...);  // Too late for preview mode
    await bridge.joinServer();
}
```

### Join and Leave Server

```javascript
// Connect to media server and start receiving streams
await bridge.joinServer();

// Member status automatically updates to 'live' in database
console.log('Connected to media server');

// Later, disconnect from server
bridge.leaveServer(true);  // true = kill local media streams

// For "pause" functionality (keep local streams alive):
bridge.leaveServer(false);  // false = preserve local streams for preview
```

### Event Callbacks

Set up callbacks **before** calling `joinServer()` to handle media events.

#### onJoinedServer

**NOTE**: This callback fires when the server state changes, including when **other members join the server**. If you're auto-starting camera or performing initialization actions, add state checks to avoid re-triggering when others join.

```javascript
let hasInitialized = false;

bridge.setOnJoinedServer(() => {
    console.log('Server join event (may be self or others joining)');
    
    // Only initialize once on YOUR first join
    if (!hasInitialized) {
        hasInitialized = true;
        console.log('First join - initializing');
        
        // Now you can start broadcasting your streams
        // Example: If you had enabled camera before joining, now broadcast it
        if (cameraIsEnabled) {
            bridge.startUpstream('camera', 'video');
        }
    }
});
```

#### onNewUpstream

**Callback Signature:**
```javascript
bridge.setOnNewUpstream((streamId, memberId, type, stream) => {
    // streamId: string      - Unique identifier for your outgoing stream
    // memberId: string      - Your member ID
    // type: string          - 'camera' or 'screencast'
    // stream: MediaStream   - The local MediaStream being broadcast
});
```

**Example Usage:**
```javascript
bridge.setOnNewUpstream((streamId, memberId, type, stream) => {
    console.log(`Your ${type} stream is now broadcasting`);
    console.log('Stream ID:', streamId);
    console.log('Member ID:', memberId);
});
```

#### onNewDownstream

**Callback Signature:**
```javascript
bridge.setOnNewDownstream((streamId, memberId, type, stream) => {
    // streamId: string   - Unique identifier for this stream
    // memberId: string   - The member who is broadcasting
    // type: string       - 'camera' or 'screencast'
    // stream: MediaStream | null - The incoming media stream
});
```

**âš ï¸ Important**: The `stream` parameter can sometimes be `null` due to timing issues or connection states. Always check for null before accessing stream methods.

**âš ï¸ Member ID Checking**: The `memberId` parameter identifies **which member** is broadcasting. In multi-member channels or when handling edge cases, you should check the member ID to take appropriate action.

**Example Usage:**
```javascript
// Get your own member ID first (recommended)
const selfMember = await bridge.getOrCreateSelfMember({ channel_id: channelId });
const myMemberId = selfMember.pid;

bridge.setOnNewDownstream((streamId, memberId, type, stream) => {
    console.log(`Received ${type} stream from member ${memberId}`);
    
    // âœ… ALWAYS check if stream is null first
    if (!stream) {
        console.warn('Received null stream, ignoring');
        return;
    }
    
    // âœ… OPTIONAL but RECOMMENDED: Check if it's your own stream
    // (Sometimes you receive your own downstream as an echo)
    if (memberId === myMemberId) {
        console.log('Ignoring my own stream');
        return;
    }
    
    // type: 'camera' or 'screencast'
    // stream: MediaStream object with audio/video tracks
    
    // Attach stream to video element
    if (type === 'camera') {
        // For 1-to-1 apps: Single remote video element
        const videoElement = document.getElementById('remote-camera');
        videoElement.srcObject = stream;
        videoElement.play().catch(e => console.log('Autoplay prevented:', e));
        
        // For multi-member apps: Create/update member-specific element
        // let videoElement = document.getElementById(`video-${memberId}`);
        // if (!videoElement) {
        //     videoElement = createVideoElementForMember(memberId);
        // }
        // videoElement.srcObject = stream;
        // videoElement.play();
    } else if (type === 'screencast') {
        const videoElement = document.getElementById('remote-screen');
        videoElement.srcObject = stream;
        videoElement.play().catch(e => console.log('Autoplay prevented:', e));
    }
    
    // Check what tracks are in the stream
    const hasAudio = stream.getAudioTracks().length > 0;
    const hasVideo = stream.getVideoTracks().length > 0;
    console.log(`Stream has audio: ${hasAudio}, video: ${hasVideo}`);
});
```

#### onEndDownstream

**Callback Signature:**
```javascript
bridge.setOnEndDownstream((streamId, memberId, type) => {
    // streamId: string  - Unique identifier for the stream that ended
    // memberId: string  - The member who stopped broadcasting
    // type: string      - 'camera' or 'screencast'
});
```

**âš ï¸ CRITICAL - Member ID Checking**: When you stop your own camera, this callback fires with **your member ID**. You must check the member ID to avoid clearing the remote video when your own stream ends.

**Example Usage:**
```javascript
// Get your own member ID first (required for proper handling)
const selfMember = await bridge.getOrCreateSelfMember({ channel_id: channelId });
const myMemberId = selfMember.pid;

bridge.setOnEndDownstream((streamId, memberId, type) => {
    console.log(`${type} stream from member ${memberId} ended`);
    
    // âœ… CRITICAL: Only clear remote video if it's NOT your own stream
    if (memberId === myMemberId) {
        console.log('My own stream ended, not clearing remote video');
        return;
    }
    
    // Remove stream from video element
    if (type === 'camera') {
        // For 1-to-1 apps:
        const videoElement = document.getElementById('remote-camera');
        videoElement.srcObject = null;
        
        // For multi-member apps:
        // const videoElement = document.getElementById(`video-${memberId}`);
        // if (videoElement) {
        //     videoElement.srcObject = null;
        //     videoElement.remove(); // Optionally remove the element
        // }
    } else if (type === 'screencast') {
        const videoElement = document.getElementById('remote-screen');
        videoElement.srcObject = null;
    }
});
```

#### onExitedServer

```javascript
bridge.setOnExitedServer(() => {
    console.log('Disconnected from media server');
    
    // Clean up UI, clear remote video elements
    document.querySelectorAll('video').forEach(vid => {
        vid.srcObject = null;
    });
});
```

### Complete Connection Example

This example shows the complete workflow from joining a channel to streaming video.

**âš ï¸ CRITICAL WORKFLOW:** You must call `startMedia()` BEFORE `joinServer()`, and you must wait for the `onJoinedServer` callback to fire before calling `startUpstream()`. Do NOT call `startUpstream()` immediately after `joinServer()` - the connection needs time to establish.

```javascript
async function connectToChannel(channelId) {
    // 1. Get member data (creates membership if doesn't exist)
    const memberData = await bridge.getOrCreateSelfMember({
        channel_id: channelId
    });
    
    // 2. Get channel data
    const channelData = await bridge.getOneChannel({
        channel_id: channelId,
        depth: 2  // Include member list
    });
    
    // 3. Setup server connection (3 required parameters)
    bridge.setupServer(
        channelData.pid,           // channelId
        memberData.pid,            // memberId
        memberData.access_code     // accessCode
    );
    
    console.log('âœ“ Server setup complete');
    
    // 4. Setup event callbacks BEFORE joining
    const myMemberId = memberData.pid;
    
    bridge.setOnJoinedServer(() => {
        console.log('âœ“ Joined server, ready to broadcast');
    });
    
    bridge.setOnNewDownstream((streamId, memberId, type, stream) => {
        // Check for null stream
        if (!stream) {
            console.warn('Received null stream, ignoring');
            return;
        }
        
        // Ignore own stream echo
        if (memberId === myMemberId) {
            console.log('Ignoring my own stream');
            return;
        }
        
        // Attach to video element
        const videoEl = document.getElementById('remote-video');
        videoEl.srcObject = stream;
        videoEl.play().catch(e => console.log('Autoplay prevented:', e));
        
        console.log(`âœ“ Receiving ${type} from member ${memberId}`);
    });
    
    bridge.setOnEndDownstream((streamId, memberId, type) => {
        // Only clear video if it's not our own stream ending
        if (memberId === myMemberId) {
            return;
        }
        
        const videoEl = document.getElementById('remote-video');
        videoEl.srcObject = null;
        
        console.log(`âœ— ${type} from member ${memberId} ended`);
    });
    
    bridge.setOnExitedServer(() => {
        console.log('âœ— Disconnected from server');
    });
    
    // 5. Join server to connect to other participants
    await bridge.joinServer();
    
    console.log('âœ“ Connected to channel');
}

// Complete example: Join and start camera
// This shows the CORRECT order of operations
async function joinAndStartCamera(channelId) {
    // Setup connection
    await connectToChannel(channelId);
    
    // âœ… STEP 1: Start local camera FIRST (await it!)
    const stream = await bridge.startMedia('camera', true, true);
    
    if (stream) {
        // Show local preview
        const myVideo = document.getElementById('my-video');
        myVideo.srcObject = stream;
        myVideo.muted = true;  // Prevent audio feedback
        myVideo.play();
        
        // âœ… STEP 2: Setup onJoinedServer callback to start upstream when ready
        bridge.setOnJoinedServer(async () => {
            console.log('âœ“ Server joined, starting broadcast...');
            
            // Start broadcasting (this happens AFTER server connection is established)
            await bridge.startUpstream('camera', 'video');
            await bridge.startUpstream('camera', 'audio');
            
            console.log('âœ“ Camera broadcasting to channel');
        });
        
        // âœ… STEP 3: Join server (callback above will fire when ready)
        await bridge.joinServer();
        
        console.log('âœ“ Joining server...');
    }
}

// âŒ WRONG - Do NOT do this:
async function joinAndStartCameraWRONG(channelId) {
    await connectToChannel(channelId);
    const stream = await bridge.startMedia('camera', true, true);
    
    await bridge.joinServer();
    
    // âŒ ERROR: Calling startUpstream immediately after joinServer
    // The connection isn't ready yet - this will fail!
    await bridge.startUpstream('camera', 'video');  // WILL FAIL
    await bridge.startUpstream('camera', 'audio');  // WILL FAIL
}
```

**Key Takeaways:**

1. **Always `await startMedia()` first** - Get the local stream before joining
2. **Setup `onJoinedServer` callback** - Define what to do when join completes
3. **Call `startUpstream()` inside the callback** - Wait for the signal that connection is ready
4. **Then `await joinServer()`** - Start the connection process
        await bridge.startUpstream('camera', 'video');
        await bridge.startUpstream('camera', 'audio');
        
        console.log('âœ“ Broadcasting camera to channel');
    }
}
```

---

## Section 4: Media Local Stream Management

Manage local camera and screencast streams. These operations create/control local hardware access.

**âš ï¸ PREREQUISITE: Must call `setupServer()` first!**

Before using any functions in this section, you must have called `bridge.setupServer()` (Section 3). Even for local preview without joining a channel, the comms object must exist.

```javascript
// Always check before starting media
if (!bridge.hasComms()) {
    console.error('Must call setupServer() before starting media');
    return;
}
```

### Start Media

**Method Signature:**
```javascript
await bridge.startMedia(
    streamType: string,     // 'camera' or 'screencast'
    audioOn: boolean,       // true to enable microphone
    videoOn: boolean,       // true to enable camera/screen
    deviceId?: string       // optional: specific device ID
) : Promise<MediaStream | null>
```

**Example Usage:**
```javascript
// Start camera with audio and video
const cameraStream = await bridge.startMedia(
    'camera',      // streamType
    true,          // audioOn: enable microphone
    true,          // videoOn: enable camera
    null           // deviceId: specific device (optional)
);

// Response: MediaStream object or null on failure
if (cameraStream) {
    console.log('Camera started');
    console.log('Audio tracks:', cameraStream.getAudioTracks().length);
    console.log('Video tracks:', cameraStream.getVideoTracks().length);
    
    // Attach to local video element for preview
    const myVideo = document.getElementById('my-camera');
    myVideo.srcObject = cameraStream;
    myVideo.muted = true;  // Mute to prevent feedback
    myVideo.play();
}
```

### Start Screencast

```javascript
// Start screen sharing (video only, typically)
const screenStream = await bridge.startMedia(
    'screencast',  // streamType
    false,         // audioOn: usually false for screencasts
    true,          // videoOn: true to capture screen
    null
);

if (screenStream) {
    console.log('Screencast started');
    
    // Attach to local video element for preview
    const myScreen = document.getElementById('my-screen');
    myScreen.srcObject = screenStream;
    myScreen.muted = true;
    myScreen.play();
    
    // Listen for user stopping screenshare via browser UI
    screenStream.getVideoTracks()[0].onended = () => {
        console.log('User stopped screensharing via browser');
        myScreen.srcObject = null;
        // Update your UI state
    };
}
```

### Get Existing Media

```javascript
// Retrieve currently active local stream
const currentCamera = bridge.getMedia('camera');
const currentScreen = bridge.getMedia('screencast');

if (currentCamera) {
    console.log('Camera is active');
} else {
    console.log('Camera is not started');
}
```

### Stop Media

**Method Signature:**
```javascript
await bridge.stopMedia(
    streamType: string,     // 'camera' or 'screencast'
    trackType?: string      // 'video', 'audio', or omit for all tracks
)
```

**âš ï¸ CRITICAL PATTERN: Managing Stream Lifecycle**

MediaStreams **cannot exist with zero tracks** - an empty stream is inactive and will be garbage collected by the browser and media server. This has important implications for toggling video/audio:

**The Rule:**
- If stopping the **last track** on a stream â†’ Call `stopMedia(streamType)` WITHOUT trackType to kill entire stream
- If other tracks remain active â†’ Call `stopMedia(streamType, trackType)` to stop only that track

**Why This Matters:**
When you restart media after stopping, WebRTC needs to renegotiate the connection. If you only stop a track but leave the stream alive (with other tracks), the old stream is reused. If the stream had no tracks, reusing it fails because it's inactive.

**Pattern: Video Toggle When Audio Might Be Active**
```javascript
async function stopVideo() {
    const audioIsOn = /* your audio state tracking */;
    
    if (audioIsOn) {
        // Audio still flowing - just stop video track
        bridge.stopMedia('camera', 'video');
    } else {
        // No audio - kill entire stream so restart gets fresh one
        bridge.stopMedia('camera');
    }
    
    // Stop upstream
    if (inChannel) {
        await bridge.stopUpstream('camera', 'video');
    }
}

async function startVideo() {
    const audioIsOn = /* your audio state tracking */;
    
    // CRITICAL: Pass current audio state to ensure proper stream creation
    const stream = await bridge.startMedia('camera', audioIsOn, true);
    
    // Start upstream
    if (inChannel) {
        await bridge.startUpstream('camera', 'video');
    }
}
```

**Example Usage:**
```javascript
// Stop entire camera stream (both audio and video)
await bridge.stopMedia('camera');

// Stop only camera video track (audio continues) - ONLY if audio is actually on
await bridge.stopMedia('camera', 'video');

// Stop only camera audio track (video continues) - ONLY if video is actually on
await bridge.stopMedia('camera', 'audio');

// Stop screencast completely
await bridge.stopMedia('screencast');

// After stopping, remove from video element
const myVideo = document.getElementById('my-camera');
myVideo.srcObject = null;
```

**âš ï¸ KNOWN LIMITATION - onEndDownstream Behavior:**

**Current Behavior:** When you call `stopMedia(streamType, trackType)` to stop a single track (e.g., video while audio remains active), the underlying implementation removes the track from the MediaStream. This causes remote users to receive an `onEndDownstream` event, even though the stream technically still exists with other tracks.

**Important Implications:**
1. **Track Information Lost**: The `onEndDownstream` callback receives an optional 4th parameter `trackKind` ("audio" or "video") indicating which specific track ended, but this parameter is not documented and most applications ignore it.

2. **Remote User Experience**: Remote users may interpret this as the entire stream ending and clear the video element, causing unexpected behavior when only one track was stopped.

3. **Workaround for Remote Users**: In your `onEndDownstream` handler, check the member's media state using `getMemberMediaStates()` before clearing the video element:

```javascript
bridge.setOnEndDownstream(async (streamId, memberId, type, trackKind) => {
    // Check if member still has other tracks active
    const mediaStates = bridge.getMemberMediaStates(memberId);
    
    if (type === 'camera') {
        const hasAudio = mediaStates?.camera?.audio?.enabled;
        const hasVideo = mediaStates?.camera?.video?.enabled;
        
        // Only clear video if BOTH tracks are gone
        if (!hasAudio && !hasVideo) {
            const videoElement = document.getElementById(`video-${memberId}`);
            if (videoElement) {
                videoElement.srcObject = null;
            }
        }
    }
});
```

4. **Best Practice - Use stopUpstream Instead**: When stopping a single track while other tracks remain active, prefer using `stopUpstream(streamType, trackType)` which disables the track without removing it, avoiding the spurious `onEndDownstream` event:

```javascript
// âŒ BAD: Causes onEndDownstream for remote users even though audio still active
if (audioIsOn) {
    await bridge.stopMedia('camera', 'video');
    await bridge.stopUpstream('camera', 'video');
}

// âœ… GOOD: Only disable track, no onEndDownstream triggered
if (audioIsOn) {
    await bridge.stopUpstream('camera', 'video');  // Just disable - don't remove
}

// âœ… GOOD: Kill entire stream when it's the last track
if (!audioIsOn) {
    bridge.stopMedia('camera');  // Kill entire stream
    await bridge.stopUpstream('camera', 'video');
}
```

**Summary**: When other tracks are still active, call `stopUpstream()` ONLY without calling `stopMedia()` to avoid triggering false stream-end events for remote users.

### Local Media Example Workflow

```javascript
let cameraEnabled = false;
let audioEnabled = false;

async function toggleCamera() {
    if (!cameraEnabled) {
        // Turn on camera - pass current audio state
        const stream = await bridge.startMedia('camera', audioEnabled, true);
        
        if (stream) {
            // Show local preview
            const videoEl = document.getElementById('my-camera');
            videoEl.srcObject = stream;
            videoEl.muted = true;
            videoEl.play();
            
            cameraEnabled = true;
            
            // If already in channel, start broadcasting
            if (bridge.hasComms() && isInChannel) {
                await bridge.startUpstream('camera', 'video');
            }
        }
    } else {
        // Turn off camera
        // CRITICAL: Check if audio is on to decide stream handling
        if (!audioEnabled) {
            // No audio - kill entire stream
            bridge.stopMedia('camera');
        } else {
            // Audio still on - just stop video track
            bridge.stopMedia('camera', 'video');
        }
        
        const videoEl = document.getElementById('my-camera');
        videoEl.srcObject = null;
        
        cameraEnabled = false;
        
        // Stop broadcasting if in channel
        if (bridge.hasComms() && isInChannel) {
            await bridge.stopUpstream('camera', 'video');
        }
    }
}

async function toggleAudio() {
    if (!audioEnabled) {
        // Turn on audio - pass current video state
        const stream = await bridge.startMedia('camera', true, cameraEnabled);
        
        if (stream) {
            audioEnabled = true;
            
            // If already in channel, start broadcasting
            if (bridge.hasComms() && isInChannel) {
                await bridge.startUpstream('camera', 'audio');
            }
        }
    } else {
        // Turn off audio
        // CRITICAL: Check if video is on to decide stream handling
        if (!cameraEnabled) {
            // No video - kill entire stream
            bridge.stopMedia('camera');
        } else {
            // Video still on - just stop audio track
            bridge.stopMedia('camera', 'audio');
        }
        
        audioEnabled = false;
        
        // Stop broadcasting if in channel
        if (bridge.hasComms() && isInChannel) {
            await bridge.stopUpstream('camera', 'audio');
        }
    }
}
```

---

## Section 5: Upstream Broadcasting Management

Control which local media streams are sent to the media server. You can have local streams running (preview) without broadcasting them.

**Key Distinction**:
- **Section 4** = Local hardware (camera/mic/screen)
- **Section 5** = Broadcasting to other participants

**âš ï¸ CRITICAL PREREQUISITE:** You must call `joinServer()` and wait for the `onJoinedServer` callback to fire BEFORE calling `startUpstream()`. If you call `startUpstream()` immediately after `joinServer()` without waiting for the callback, it will fail because the WebRTC connection is not yet established.

**Correct Flow:**
```javascript
// 1. Start media first
const stream = await bridge.startMedia('camera', true, true);

// 2. Setup callback to start upstream when ready
bridge.setOnJoinedServer(async () => {
    // 3. This fires when connection is ready - NOW start upstream
    await bridge.startUpstream('camera', 'video');
    await bridge.startUpstream('camera', 'audio');
});

// 4. Join server (triggers callback above when ready)
await bridge.joinServer();
```

### Start Upstream

**Method Signature:**
```javascript
await bridge.startUpstream(
    streamType: string,     // 'camera' or 'screencast'
    trackType?: string      // 'video', 'audio', or omit for all tracks
)
```

**Example Usage:**
```javascript
// Start broadcasting your camera video to other participants
await bridge.startUpstream('camera', 'video');

// Start broadcasting your camera audio
await bridge.startUpstream('camera', 'audio');

// Start broadcasting entire camera stream (audio + video)
await bridge.startUpstream('camera');  // No trackType = broadcast all tracks

// Start broadcasting screencast video
await bridge.startUpstream('screencast', 'video');
```

**âš ï¸ Important:** You must call `joinServer()` before calling `startUpstream()`, otherwise the broadcast will not reach other participants. Best practice is to call `startUpstream()` inside the `onJoinedServer` callback.

### Stop Upstream

**Method Signature:**
```javascript
await bridge.stopUpstream(
    streamType: string,     // 'camera' or 'screencast'
    trackType?: string      // 'video', 'audio', or omit for all tracks
)
```

**Example Usage:**
```javascript
// Stop broadcasting camera video (others won't see you)
await bridge.stopUpstream('camera', 'video');

// Stop broadcasting camera audio (others won't hear you)
await bridge.stopUpstream('camera', 'audio');

// Stop broadcasting entire camera stream
await bridge.stopUpstream('camera');

// Stop broadcasting screencast
await bridge.stopUpstream('screencast', 'video');
```

### Mute/Hide Tracks

Temporarily mute audio or hide video without stopping the broadcast connection.

```javascript
// Mute microphone (track continues, but silent)
await bridge.setMutedUpstream('camera', 'audio', true);

// Unmute microphone
await bridge.setMutedUpstream('camera', 'audio', false);

// Hide video (track continues, but black/frozen)
await bridge.setMutedUpstream('camera', 'video', true);

// Show video again
await bridge.setMutedUpstream('camera', 'video', false);

// Mute both audio and video
await bridge.setMutedUpstream('camera', 'all', true);
```

### Broadcasting Workflow Example

```javascript
// State tracking
let localMediaState = {
    camera: { video: false, audio: false },
    screencast: { video: false }
};
let inChannel = false;

// 1. Enable local camera (preview, not broadcasting)
async function enableCamera() {
    const stream = await bridge.startMedia('camera', true, true);
    if (stream) {
        // Show local preview
        document.getElementById('my-video').srcObject = stream;
        document.getElementById('my-video').play();
        localMediaState.camera.video = true;
        localMediaState.camera.audio = true;
    }
}

// 2. Join channel
async function joinChannel(channelId) {
    await connectToChannel(channelId);  // From Section 3 example
    
    await bridge.joinServer();
    inChannel = true;
    
    // 3. Start broadcasting (now that we're in channel)
    if (localMediaState.camera.video) {
        await bridge.startUpstream('camera', 'video');
    }
    if (localMediaState.camera.audio) {
        await bridge.startUpstream('camera', 'audio');
    }
}

// 4. Pause (stop broadcasting but keep local preview)
async function pauseChannel() {
    bridge.leaveServer(false);  // false = keep local streams
    inChannel = false;
    
    // Local preview still visible, but not broadcasting
    console.log('Paused - preview mode');
}

// 5. Rejoin (resume broadcasting)
async function rejoinChannel() {
    await bridge.joinServer();
    inChannel = true;
    
    // Resume broadcasting streams that are enabled
    if (localMediaState.camera.video) {
        await bridge.startUpstream('camera', 'video');
    }
    if (localMediaState.camera.audio) {
        await bridge.startUpstream('camera', 'audio');
    }
}
```

---

## Section 6: Tagging

Tag users, channels, and yourself with custom metadata. Tags are stored in a tree structure.

### Tag Yourself

```javascript
// Add a tag to your own profile
await bridge.tagSelf({
    tagtreeAction: 'add',          // 'add' | 'set' | 'clear'
    tag_info: 'developer'
});

// Add multiple tags
await bridge.tagSelf({
    tagtreeAction: 'add',
    tag_info: 'team:engineering,role:senior'
});

// Set tags (replaces existing)
await bridge.tagSelf({
    tagtreeAction: 'set',
    tag_info: 'status:available'
});

// Clear specific tag
await bridge.tagSelf({
    tagtreeAction: 'clear',
    tag_info: 'status:available'
});
```

### Tag Another User

```javascript
// Add tag to another user
await bridge.tagUser({
    user_id: 'user_abc123',
    tagtreeAction: 'add',
    tag_info: 'colleague'
});

// Set pinned status
await bridge.tagUser({
    user_id: 'user_abc123',
    tagtreeAction: 'set',
    tag_info: 'pinned:true'
});
```

### Pin/Unpin Users

```javascript
// Pin a user (for favorites/quick access)
await bridge.pinUser({
    user_id: 'user_abc123'
});

// Unpin a user
await bridge.unpinUser({
    user_id: 'user_abc123'
});

// Check if user is pinned (in user object)
const user = await bridge.getOneUser({ user_id: 'user_abc123', depth: 2 });
if (user.tag_tree?.pinned === true) {
    console.log('User is pinned');
}
```

### Tag Channels

```javascript
// Tag a channel (future functionality - partial implementation)
await bridge.tagChannel_TODO({
    channel_id: 'channel_xyz',
    tag: 'important'
});

// Pin a channel
await bridge.tagChannel_TODO({
    channel_id: 'channel_xyz',
    tag: 'pinned:true'
});
```

### Tagging Example: Filter Pinned Users

```javascript
// Get all users
const allUsers = await bridge.getUsers({ count: 100, depth: 2 });

// Filter to only pinned users
const pinnedUsers = allUsers.filter(user => user.tag_tree?.pinned === true);

console.log('Pinned users:', pinnedUsers.length);
pinnedUsers.forEach(user => {
    console.log(`- ${user.username}`);
});
```

---

## Complete Application Example

Here's a minimal example bringing it all together:

```javascript
import Bridge from './scripts/bridge.js';

class SimpleVideoApp {
    constructor() {
        this.bridge = new Bridge();
        this.currentChannel = null;
        this.inChannel = false;
    }
    
    async login(email, password) {
        const result = await this.bridge.login({ email, password });
        
        if (result.status === 'loggedIn') {
            console.log('âœ“ Logged in');
            this.setupListeners();
            return true;
        }
        return false;
    }
    
    setupListeners() {
        // Listen for invites
        this.bridge.setOnInviteUpdate((message) => {
            if (message?.type === 'quickchat') {
                console.log(`ðŸ“¨ Invite from ${message.username}`);
                // Auto-accept and join
                this.joinChannel(message.channel_id);
            }
        });
        
        // Listen for member updates
        this.bridge.setOnMemberUpdate((message) => {
            const event = message.event;
            console.log(`ðŸ‘¤ Member ${event}`);
        });
    }
    
    async startCamera() {
        const stream = await this.bridge.startMedia('camera', true, true);
        
        if (stream) {
            const videoEl = document.getElementById('my-video');
            videoEl.srcObject = stream;
            videoEl.muted = true;
            videoEl.play();
            console.log('ðŸ“¹ Camera started');
            
            // If in channel, start broadcasting
            if (this.inChannel) {
                await this.bridge.startUpstream('camera', 'video');
                await this.bridge.startUpstream('camera', 'audio');
            }
        }
    }
    
    async joinChannel(channelId) {
        // Get member data
        const memberData = await this.bridge.getOrCreateSelfMember({
            channel_id: channelId
        });
        
        const channelData = await this.bridge.getOneChannel({
            channel_id: channelId,
            depth: 2
        });
        
        // Setup server
        this.bridge.setupServer(
            channelData.pid,
            memberData.pid,
            memberData.access_code
        );
        
        // Setup callbacks
        this.bridge.setOnJoinedServer(() => {
            console.log('âœ“ Joined media server');
            this.inChannel = true;
        });
        
        this.bridge.setOnNewDownstream((streamId, memberId, type, stream) => {
            console.log(`ðŸ“º Received ${type} from ${memberId}`);
            
            const videoEl = document.getElementById('remote-video');
            videoEl.srcObject = stream;
            videoEl.play();
        });
        
        this.bridge.setOnEndDownstream((streamId, memberId, type) => {
            console.log(`âŒ Stream ended: ${type} from ${memberId}`);
            document.getElementById('remote-video').srcObject = null;
        });
        
        // Join
        await this.bridge.joinServer();
        this.currentChannel = channelData;
    }
    
    async leaveChannel() {
        this.bridge.leaveServer(true);  // true = kill streams
        this.inChannel = false;
        
        // Clean up
        document.getElementById('my-video').srcObject = null;
        document.getElementById('remote-video').srcObject = null;
        console.log('ðŸ‘‹ Left channel');
    }
}

// Usage
const app = new SimpleVideoApp();

await app.login('user@example.com', 'password');
await app.startCamera();
await app.joinChannel('channel_xyz');
```

---

## Best Practices

### 1. Check Bridge Readiness

Always verify comms exist before media operations:

```javascript
if (bridge.hasComms()) {
    // Safe to use media operations (Section 4 & 5)
    await bridge.startMedia('camera', true, true);
} else {
    console.error('Must call setupServer() first');
    // Call setupServer() before any media operations
}
```

**Remember**: `setupServer()` is required even for local preview (before joining).

### 2. Setup Callbacks Before Joining

Always set event callbacks **before** calling `joinServer()`:

```javascript
// âœ“ CORRECT
bridge.setOnNewDownstream(callback);
bridge.setOnJoinedServer(callback);
await bridge.joinServer();

// âœ— WRONG - might miss early events
await bridge.joinServer();
bridge.setOnNewDownstream(callback);  // Too late!
```

### 3. Separate Local and Broadcast State

Track whether media is:
- **Enabled** = Hardware active (local preview)
- **Broadcasting** = Sent to other participants

```javascript
const state = {
    enabled: false,      // Local hardware
    broadcasting: false  // Sending to SFU
};
```

### 4. Handle Async Properly

Most operations are async - always await:

```javascript
// âœ“ CORRECT
await bridge.startMedia('camera', true, true);
await bridge.startUpstream('camera', 'video');

// âœ— WRONG - race conditions
bridge.startMedia('camera', true, true);
bridge.startUpstream('camera', 'video');  // May execute before media starts
```

### 5. Clean Up on Exit

```javascript
async function cleanup() {
    // Stop broadcasting
    await bridge.stopUpstream('camera');
    
    // Stop local media
    await bridge.stopMedia('camera');
    
    // Leave server
    bridge.leaveServer(true);
    
    // Clear video elements
    document.querySelectorAll('video').forEach(v => v.srcObject = null);
}
```

---

## Error Handling

```javascript
try {
    const stream = await bridge.startMedia('camera', true, true);
    if (!stream) {
        console.error('Failed to get camera access');
        // Notify user, check permissions
    }
} catch (error) {
    console.error('Camera error:', error);
    // Handle specific errors (permission denied, hardware not found, etc.)
}
```

---

## Common Patterns

### Pattern: Join Channel Flow

```javascript
async function completeChannelJoinFlow(channelId) {
    // 1. Get data
    const [member, channel] = await Promise.all([
        bridge.getOrCreateSelfMember({ channel_id: channelId }),
        bridge.getOneChannel({ channel_id: channelId, depth: 2 })
    ]);
    
    // 2. Setup
    bridge.setupServer(channel.pid, member.pid, member.access_code);
    
    // 3. Callbacks
    bridge.setOnJoinedServer(() => console.log('Ready'));
    bridge.setOnNewDownstream(handleRemoteStream);
    bridge.setOnEndDownstream(handleStreamEnd);
    
    // 4. Join
    await bridge.joinServer();
}
```

### Pattern: Toggle Media with State

```javascript
class MediaToggle {
    constructor(streamType, trackType) {
        this.streamType = streamType;
        this.trackType = trackType;
        this.enabled = false;
    }
    
    async toggle() {
        if (this.enabled) {
            await bridge.stopMedia(this.streamType, this.trackType);
            this.enabled = false;
        } else {
            const audioOn = this.trackType === 'audio';
            const videoOn = this.trackType === 'video';
            await bridge.startMedia(this.streamType, audioOn, videoOn);
            this.enabled = true;
            
            if (inChannel) {
                await bridge.startUpstream(this.streamType, this.trackType);
            }
        }
    }
}

const cameraVideo = new MediaToggle('camera', 'video');
await cameraVideo.toggle();  // Turn on/off
```

---

## Appendix: Data Structures

### User Object
```javascript
{
    pid: 'user_abc123',
    email: 'user@example.com',
    username: 'JohnDoe',
    status: 'active',
    current_context_id: 'ctx_xyz',
    current_context_title: 'Workspace Name',
    config: { /* custom settings */ },
    tag_tree: { pinned: true, /* other tags */ }
}
```

### Channel Object
```javascript
{
    pid: 'channel_xyz',
    title: 'Meeting Room',
    description: 'Weekly sync',
    max_size: 10,
    access_code: '413239',
    allows_guests: false,
    members: [ /* member objects if depth >= 2 */ ],
    tag_tree: { /* tags */ }
}
```

### Member Object
```javascript
{
    pid: 'member_123',
    user_id: 'user_abc123',
    channel_id: 'channel_xyz',
    access_code: '413239',
    status: 'live',  // 'invited' | 'in_lobby' | 'live' | 'paused'
    local_name: 'John',
    cam_audio_state: true,
    cam_video_state: true,
    cam_audio_detail: 'ON',  // 'OFF' | 'ON' | 'MUTED'
    cam_video_detail: 'ON',  // 'OFF' | 'ON' | 'HIDDEN'
    screen_audio_state: false,
    screen_video_state: false,
    screen_audio_detail: 'OFF',
    screen_video_detail: 'OFF'
}
```

### WebSocket Message Structure
```javascript
{
    event: 'member_joined',  // Event type
    id_map: {
        user: 'user_abc123',
        member: 'member_123',
        channel: 'channel_xyz'
    },
    delta: { /* changed fields */ }
}
```

---

## Summary

Bridge.js provides six functional areas:

0. **Basics** - Check connection readiness
1. **Auth** - Login, logout, profile management
2. **Context Data** - Users, channels, members, invites + real-time listeners
3. **Server** - Connect to media server + stream event callbacks
4. **Media** - Control local camera/mic/screen hardware
5. **Broadcasting** - Control what gets sent to other participants
6. **Tagging** - Metadata and organization

**Typical Flow:**
1. Login â†’ WebSocket auto-connects
2. Setup listeners for real-time updates
3. Create/join channel
4. Setup server connection
5. Start local media (preview)
6. Join server (connect to SFU)
7. Start broadcasting (others see/hear you)
8. Receive remote streams (see/hear others)

Start building your video collaboration app today! ðŸŽ¥