# Real-Time Communication Platform

A real-time multi-user communication platform supporting group chat
and video conferencing.

The application combines WebSocket-based real-time messaging with
an SFU-based video conferencing architecture to provide real-time
communication between multiple users.

## features

- User authentication
- Active chat reordering based on latest message activity
- Friends contact
- Both p2p and group chat
- Real-time user online/offline statement
- Real-time messaging
- Cursor-based pagination
- Real-time video conferencing
- Room-base video conferencing
- Local https /SSL

## Tech Stack

### Frontend
- React
- Next.js
- TypeScript
- TanStack Query
- Zustand
- Axios
- LiveKit Client SDK

### Backend
- Laravel
- LiveKit Server SDK
- Beacon

### Real-Time Communication
- Laravel Reverb
- Laravel Echo

### Video Conferencing
- LiveKit
- SFU (Selective Forwarding Unit)

### Database
- Mysql
- Redis

### DevOps & Infrastructure
- Docker
- Caddy
- Local HTTPS / SSL

## Setup

1. Set your local LAN IP address as `DOMAIN_NAME` in `.env`.

2. Build and start the containers:

```bash
docker compose up --build
```

## Architecture

### API Request Flow

```
                         Browser
                            │
                         HTTPS
                            ▼
                    Caddy (app.domain)
                            │
                            ▼
                       Next.js BFF
                            │
                          Axios
                            ▼
                    Caddy (api.domain)
                            │
                            ▼
                       Laravel API
                       /    |    \
                      /     |     \
                     ▼      ▼      ▼
                Eloquent  Redis   Queue
                   │                │
                   ▼                ▼
                 MySQL            Reverb
                                      │
                                      ▼
                                   Browser
```

### Chat Communication

                   SENDER
                       │
                 Send Message
                       │
                       ▼
              TanStack Mutation
                       │
                       ▼
                  Next.js BFF
                       │
                     Axios
                       │
                       ▼
                 Laravel API
                       │
                       ▼
                Message Table
                       │
                       ▼
               Broadcast Event
                       │
                       ▼
                 Laravel Reverb
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
         RECEIVER A          RECEIVER B
             │                   │
        Laravel Echo        Laravel Echo
             │                   │
      Subscribe Channel     Subscribe Channel
             │                   │
       Listen Event          Listen Event
             │                   │
             ▼                   ▼
       Message Received    Message Received
             │                   │
             ▼                   ▼
       TanStack Query      TanStack Query
             │                   │
             ▼                   ▼
             UI                  UI

### Video Communication

***call initialization***

```
                         CALL SIGNALING
                              │
Caller                        │
  │                           │
  │ Call Button               │
  ▼                           │
TanStack Mutation             │
  │                           │
  ▼                           │
Next.js BFF                   │
  │                           │
  ▼                           │
Laravel API ──────→ Call Event
                         │
                         ▼
                   Laravel Reverb
                         │
                         ▼
                    Callee Browser
                         │
                    Accept Call
                         │
                         ▼
                    Laravel API
                         │
                         ▼
                 Validate Call State
                         │
                         ▼
                       Redis
                         │
                         ▼
                LiveKit Server SDK
                         │
                    Create Room
                    Create Token
                         │
                         ▼
                   LiveKit Server
                         │
                         ▼
                    Broadcast Event
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
          Caller                  Callee
          Browser                 Browser
              │                     │
              └──────────┬──────────┘
                         ▼
                    Zustand Store
                         │
                         ▼
                      CallView

```

***call Invitation***

    ```text
        Caller
           │
           ▼
        CallView
           │
           │ Check user availability
           ▼
        Invite User
           │
           ▼
        TanStack Query Mutation
           │
           ▼
        Next.js BFF
           │
           ▼
        Laravel API
           │
           ▼
        Laravel Reverb
           │
           ▼
        Call Invitation Event
           │
           ▼
        Callee Browser
           │
           │ Accept Invitation
           ▼
        Retrieve Call Room Data
           │
           ▼
        Zustand Store
           │
           ▼
        Redirect to CallView

***call view component***

          CallView
         │
         ▼
      Get Room Name & Token
      from Zustand
         │
         ▼
      Create LiveKit Room
         │
         ▼
      Initialize LiveKit Connection
         │
         ▼
      Connect to LiveKit Server
         │
         ▼
      Publish Local Media
         ├── Audio Track
         └── Video Track
         │
         ▼
      Subscribe to Remote Participants
         │
         ▼
      Receive Remote Media Tracks
         │
         ▼
      Render Tracks in UI

## Tech Stack Explanation

### React

React is used as the primary UI library to build the application using reusable and composable components.

The component-based architecture makes it easier to organize, reuse, and maintain UI logic across the application.

### Next.js

Next.js is used as the frontend framework to provide features such as server-side rendering and the Backend-for-Frontend (BFF) layer.

### Axios

Axios is used as the HTTP client for communication between the frontend/BFF layer and the backend API.

It provides a centralized API client and avoids repeatedly implementing request logic inside React components.

### TanStack Query

TanStack Query is used for server-state management.

It handles data fetching, caching, mutations, query invalidation, and synchronization of server data with the frontend.

This keeps server-state logic separate from UI components and reduces the need to manually manage loading, error, and cached
data states.

### Zustand

Zustand is used for client-side global state management.

It is useful for application state that needs to be accessed by multiple components without passing values through the component
tree.

### Laravel Echo

Laravel Echo is used as the client-side WebSocket library for Laravel Reverb.

It provides an abstraction for subscribing to channels and listening for real-time events broadcast by the Laravel backend.

### Beacon

Beacon is used for request and response monitoring, HTTP status logging, and database query tracing.

It helps track both raw SQL queries and query-builder operations, allowing developers to identify how many database queries were executed, which queries were executed, and which application
operations triggered them during a request lifecycle.

### Shadcn / Tweakcn / DiceBear

**Shadcn UI** is used as the UI component foundation to avoid repeatedly building common interface components from scratch with Tailwind CSS.

**Tweakcn** is used to customize and manage the application's visual theme and design system.

**DiceBear** is used to generate consistent user avatars for users who do not have a custom profile image.

### MySQL

The application requires persistent storage for data such as users,
chats, messages, and other application records.

I chose MySQL as the primary relational database because it integrates
well with Laravel and I am familiar with its relational data model
and SQL-based data management.

MySQL is used to persist application data and maintain relationships
between the different entities in the system.

### Redis

Some API endpoints require frequently accessed, real-time state that
does not need to be persisted in the database.

For example, video call room management requires tracking which users
are currently participating in a room.

I chose Redis as an in-memory data store for this type of temporary
state. Redis Sets are used to store unique user identifiers, which
prevents duplicate entries.

By keeping this temporary state in memory, the application can avoid
unnecessary database queries and reduce database load while providing
fast access to frequently changing data.



## Engineering Decisions

### Why SFU?

Peer-to-peer (P2P) mesh communication is simple and works well for one-to-one communication because each participant only needs to maintain a direct media connection with the other participant.

However, as the number of participants increases, the mesh model requires each client to establish and maintain multiple peer connections. This increases the client's CPU usage, network
bandwidth consumption, and the complexity of managing SDP offers, answers, and ICE candidates.

For this reason, I chose an SFU (Selective Forwarding Unit) architecture for multi-user video conferencing.

With an SFU, each participant joins a room and publishes their audio/video tracks once. The SFU receives these media tracks and forwards the required tracks to the other participants.

Each client publishes its own media to the SFU once, while subscribing to the media tracks it needs from the room.

This provides a more suitable architecture for multi-user conferencing by reducing the number of direct peer connections maintained by each client.

### Why BFF?

The BFF layer provides a controlled API boundary between the browser
and the Laravel backend.

Instead of communicating directly with the Laravel API, the browser
sends requests to the Next.js BFF layer, which then communicates with
the Laravel backend.

```
Browser
   ↓
Next.js BFF Endpoint
   ↓
Laravel API Endpoint
```

### Why Reverb?

The application requires a WebSocket server for real-time
communication within the Laravel and Docker ecosystem.

I chose Laravel Reverb because it can be self-hosted and integrates
directly with Laravel, making it suitable for a containerized
environment.

Reverb also works with Laravel Echo on the client side, providing a
straightforward way to subscribe to channels and listen for
real-time events from the Laravel backend.

Compared with using a hosted WebSocket service such as Pusher,
self-hosting Reverb gives me more control over the WebSocket
infrastructure and allows the service to run within my own Docker
environment.

### Why Caddy?

The application requires HTTPS in the local development environment
because browser-based media access requires a secure context.

I chose Caddy as the reverse proxy instead of Nginx because it
provides a simpler configuration and makes it easier to set up
HTTPS for local development.

Caddy handles reverse proxying and TLS configuration, allowing the
application services to be accessed through HTTPS during local
development.

### Why Docker?

The application uses a self-hosted LiveKit SFU for video
conferencing, so I needed a consistent environment for running
LiveKit and the other application services.

I chose Docker to containerize the application services and simplify
the setup of the development environment.

Docker also helps reduce environment-related issues by providing
isolated and reproducible environments across different machines.

Additionally, this project gave me an opportunity to gain practical
experience with Docker and multi-container application setup.


## Challenges & Solutions
### 1. Scalable Chat Participant Relationships

**Challenge:**  
The initial chat schema used a one-to-many relationship for chat participants, which was restrictive for supporting group conversations.

**Solution:**  
Refactored the relationship into a many-to-many structure using a dedicated chat participants relationship.

**Result:**  
The same chat architecture can support both one-to-one and multi-user group conversations while keeping the database structure flexible.

---

### 2. Efficient Message Pagination & Scroll Management

**Challenge:**  
Traditional skip/offset pagination can become inefficient as the message history grows. In addition, loading older messages at the top of a chat can cause the user's scroll position to jump.

**Solution:**  
Implemented cursor-based pagination with TanStack Query. The message container uses `useRef` and scroll events to detect when the user reaches the top of the conversation and automatically fetch older messages.

To preserve the user's position, the previous `scrollHeight` is stored before fetching additional messages. After new messages are inserted, the height difference is calculated and applied to maintain the previous visual position.

**Result:**  
Users can load older messages seamlessly without noticeable scroll-position jumps.

---

### 3. Local HTTPS for WebSocket & Media Features

**Challenge:**  
Browser media APIs such as camera and microphone access require a secure context. Running the application directly over HTTP made local testing of real-time media features difficult.

**Solution:**  
Configured Caddy as a reverse proxy and used `.nip.io` subdomains to provide local HTTPS endpoints for the application.

**Result:**  
The application can be tested locally with HTTPS while using browser camera, microphone, WebSocket, and LiveKit features.

---

### 4. Multi-Container Development Environment

**Challenge:**  
The application depends on multiple services, including Next.js, Laravel, Reverb, Redis, MySQL, and a queue worker. Managing these services independently would make local development more complex.

**Solution:**  
Containerized the services using Docker and Docker Compose. Each service runs in its own container and communicates through the Docker network, while Caddy handles reverse proxying and local HTTPS.

**Result:**  
The complete application can be started and managed as a consistent multi-container development environment.

---

### 5. WebSocket Authentication & Lifecycle Management

**Challenge:**  
Integrating Laravel Reverb with private channels introduced authentication and event-handling issues. Additional care was required to prevent stale channel subscriptions when React components were unmounted.

**Solution:**  
Debugged the Reverb authentication flow and event configuration. On the frontend, Laravel Echo channel subscriptions are explicitly cleaned up with `echo.leave()` when components unmount.

**Result:**  
Private WebSocket channels can be used reliably for chat, presence, and call-related events while preventing unnecessary subscriptions and stale connections.

---

### 6. P2P Mesh Limitations for Group Video Calls

**Challenge:**  
The initial video implementation used P2P WebRTC. While suitable for one-to-one communication, a mesh architecture requires each participant to maintain direct connections with other participants. As the number of participants increases, bandwidth and connection-management complexity also increase.

**Solution:**  
Replaced the P2P mesh architecture with LiveKit's SFU architecture.

Instead of establishing direct connections between every participant, each participant publishes their media tracks to the SFU, which forwards the required tracks to other participants.

**Result:**  
The video architecture became more suitable for multi-user conferencing while reducing the complexity of managing multiple direct peer connections.

---

### 7. Self-Hosted LiveKit Integration

**Challenge:**  
Self-hosting LiveKit introduced infrastructure and authentication issues during local development, including JWT secret requirements and SSL certificate trust problems when communicating with the LiveKit server.

**Solution:**  
Configured the self-hosted LiveKit server and integrated the LiveKit Server SDK with Laravel for room creation and access-token generation. Local SSL configuration was also adjusted so the backend could communicate with the LiveKit server.

**Result:**  
The backend can dynamically create rooms and generate authenticated LiveKit access tokens for call participants.

---

### 8. Real-Time Call Signaling

**Challenge:**  
LiveKit is responsible for the media plane, but the application still required an application-level signaling mechanism for call invitations and call state changes.

**Solution:**  
Used Laravel Reverb as the signaling layer for events such as call invitation, acceptance, and rejection.

The architecture separates signaling from media communication:

```text
Laravel Reverb
    │
    └── Signaling
        ├── Call Invitation
        ├── Accept
        └── Reject


LiveKit
    │
    └── Media Plane
        ├── Audio
        ├── Video
        └── Remote Media Tracks
