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

## Tech Stack Explanation

### React

React is used as the primary UI library to build the application using reusable and composable components.

The component-based architecture makes it easier to organize, reuse, and maintain UI logic across the application.

### Next.js

Next.js is used as the frontend framework to provide features such as server-side rendering and the Backend-for-Frontend (BFF) layer.

The BFF layer provides a controlled API boundary between the browser and the Laravel backend, so the browser communicates with the frontend application rather than directly exposing the backend API endpoint.

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


## Engineering Decisions

### Why SFU?

Peer-to-peer (P2P) mesh communication is simple and works well for one-to-one communication because each participant only needs to maintain a direct media connection with the other participant.

However, as the number of participants increases, the mesh model requires each client to establish and maintain multiple peer connections. This increases the client's CPU usage, network
bandwidth consumption, and the complexity of managing SDP offers, answers, and ICE candidates.

For this reason, I chose an SFU (Selective Forwarding Unit) architecture for multi-user video conferencing.

With an SFU, each participant joins a room and publishes their audio/video tracks once. The SFU receives these media tracks and forwards the required tracks to the other participants.

Each client publishes its own media to the SFU once, while subscribing to the media tracks it needs from the room.

This provides a more suitable architecture for multi-user conferencing by reducing the number of direct peer connections maintained by each client.
