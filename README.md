# Deployment Control Plane

> **Status: DRAFT.** This document describes the initial concept and is subject to review and change.

## Overview

Deployment Control Plane is a platform that lets a user manage deployments across multiple servers from a single place. After a single login, a user can register and set up any number of servers. Each server runs a lightweight control daemon that communicates with the platform and manages the full deployment lifecycle for the applications hosted on that server.

Deployed applications can be exposed to the outside world in one of two ways:

- **Nginx reverse proxy**: traffic is routed through Nginx on the server (for example, by domain or path).
- **Direct IP address**: the application is reached directly via the server IP and port.

## Components

The system is made up of three applications.

### 1. Frontend

The web interface used by the end user.

- Single login for the user
- Add, configure, and remove servers
- View the status of each server and its control daemon
- Create, deploy, update, roll back, and remove applications
- Choose how each application is exposed (Nginx reverse proxy or direct IP)
- View deployment history, status, and logs

### 2. Backend

The central API and source of truth for the platform.

- Authentication and user/session management
- Stores servers, applications, deployments, and their configuration
- Exposes the API consumed by the frontend
- Coordinates with the control daemons on each server, dispatching deployment actions and receiving status updates
- Tracks deployment state and history

### 3. Control Daemon

An agent installed on every managed server.

- Registers the server with the control plane
- Receives deployment instructions and executes them locally
- Manages the lifecycle of each deployed application on the server (deploy, start, stop, restart, update, roll back, remove)
- Configures Nginx reverse proxy entries when an application is exposed through Nginx
- Reports health, status, and logs back to the control plane

## High-Level Flow

```
            +------------+
   User --> |  Frontend  |
            +-----+------+
                  |
                  v
            +------------+
            |  Backend   |
            +-----+------+
                  |
     +------------+------------+
     |            |            |
     v            v            v
+---------+  +---------+  +---------+
| Daemon  |  | Daemon  |  | Daemon  |
| Server1 |  | Server2 |  | ServerN |
+----+----+  +----+----+  +----+----+
     |            |            |
  Apps via     Apps via     Apps via
  Nginx / IP   Nginx / IP   Nginx / IP
```

1. The user logs in to the frontend.
2. The user adds a server and installs the control daemon on it.
3. The daemon registers with the control plane and establishes communication.
4. The user creates an application deployment and selects how it should be exposed.
5. The control plane sends the deployment instructions to the daemon on the target server.
6. The daemon performs the deployment, configures Nginx if required, and reports status back.
7. The user monitors and manages the application lifecycle from the frontend.

## Repository Layout

To be defined. Planned top-level areas:

- `frontend/`: web user interface
- `backend/`: control plane API
- `daemon/`: control daemon installed on managed servers
- `internal_docs/`: internal documentation (excluded from version control)

## Open Questions

The following items are not yet decided:

- Technology stack for each component
- Communication protocol between backend and daemon (for example, WebSocket, gRPC, or polling over HTTPS)
- Daemon authentication and server registration mechanism
- Deployment packaging (for example, containers, binaries, or source builds)
- TLS certificate management for Nginx-exposed applications
