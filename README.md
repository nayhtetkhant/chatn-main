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

```text
Browser
   │
   │ HTTPS
   ▼
Caddy (app.domain)
   │
   ▼
Next.js BFF
   │
   │ Axios
   ▼
Caddy (api.domain)
   │
   ▼
Laravel API
   │
   ├──→ Eloquent ORM ──→ MySQL
   │
   └──→ Redis
```

### Chat Communication

### Video Communication



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

