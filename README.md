# Agentic Calendar Assistant

<p align="center">
  <img src="https://img.shields.io/badge/Next.js-16.3-black?style=for-the-badge&logo=next.js" alt="Next.js" />
  <img src="https://img.shields.io/badge/React-19.2-20232A?style=for-the-badge&logo=react" alt="React" />
  <img src="https://img.shields.io/badge/Express-5.2-000000?style=for-the-badge&logo=express" alt="Express" />
  <img src="https://img.shields.io/badge/Mastra-Core_1.59-6C47FF?style=for-the-badge&logo=ai" alt="Mastra" />
  <img src="https://img.shields.io/badge/Google_Gemini-3.5_Flash-4285F4?style=for-the-badge&logo=google" alt="Google Gemini" />
  <img src="https://img.shields.io/badge/Auth-Descope-1B254B?style=for-the-badge&logo=descope" alt="Descope" />
  <img src="https://img.shields.io/badge/PostgreSQL-16-336791?style=for-the-badge&logo=postgresql" alt="PostgreSQL" />
  <img src="https://img.shields.io/badge/TypeScript-5.x_|_7.x-3178C6?style=for-the-badge&logo=typescript" alt="TypeScript" />
</p>

<p align="center">
  <strong>An enterprise-grade, agentic meeting scheduling copilot. Powered by Google Gemini 3.5 Flash, Mastra AI Agent Framework, Descope Identity & Outbound Application Management, and Google Calendar API v3.</strong>
</p>

---

## 📑 Table of Contents

1. [Project Overview](#1-project-overview)
2. [Application Preview & Screenshots](#2-application-preview--screenshots)
3. [Key Features](#3-key-features)
4. [Example Conversations](#4-example-conversations)
5. [Technology Stack](#5-technology-stack)
6. [System Architecture](#6-system-architecture)
   - [Architectural Topology](#architectural-topology)
   - [Request Lifecycle & Agent Execution Loop](#request-lifecycle--agent-execution-loop)
7. [Project Folder Structure](#7-project-folder-structure)
8. [Prerequisites](#8-prerequisites)
9. [Local Installation & Setup](#9-local-installation--setup)
   - [Step 1: Clone Repository](#step-1-clone-repository)
   - [Step 2: Database Initialization](#step-2-database-initialization)
   - [Step 3: Backend Setup](#step-3-backend-setup)
   - [Step 4: Frontend Setup](#step-4-frontend-setup)
   - [Step 5: Run Development Servers](#step-5-run-development-servers)
10. [Environment Variables](#10-environment-variables)
11. [API Endpoint Documentation](#11-api-endpoint-documentation)
12. [Database & Memory Architecture](#12-database--memory-architecture)
13. [Authentication & Google Calendar OAuth Flow](#13-authentication--google-calendar-oauth-flow)
14. [Model Context Protocol (MCP) Integration](#14-model-context-protocol-mcp-integration)
15. [Deployment Guide](#15-deployment-guide)
16. [Docker Deployment](#16-docker-deployment)
17. [Production Readiness Checklist](#17-production-readiness-checklist)
18. [Known Limitations](#18-known-limitations)
19. [Troubleshooting Guide](#19-troubleshooting-guide)
20. [License](#20-license)

---

## 1. Project Overview

Coordinating meetings across time zones, checking real-time availability, sending calendar invites with Google Meet video conferencing, and tracking conversational context typically demand manual coordination across disparate apps.

The **Agentic Calendar Assistant** replaces manual workflows with a single conversational interface:
- **Autonomous Reasoning:** Instead of relying on hardcoded conversational branching or rule-based patterns, user queries are processed by **Google Gemini 3.5 Flash** orchestrated via **Mastra**. The model evaluates intent, extracts arguments (such as relative times, durations, and attendees), and executes external calendar operations via Mastra tools.
- **Dynamic Memory Retention:** The agent maintains **LibSQL-backed working memory** across threads. It remembers meeting preferences—such as default meeting lengths, timezone nuances, and regular attendees—so subsequent interactions require minimal input.
- **Enterprise Identity & Token Security:** Authenticated via **Descope**, user identities and outbound third-party Google Calendar OAuth tokens are brokered securely without storing permanent client secrets in application code or client storage.

---

## 2. Application Preview & Screenshots

| Interface | View | Description |
|---|---|---|
| **Chat & Agent Interaction** | *(Add screenshot: `docs/screenshots/chat-dashboard.png`)* | Split-panel dashboard displaying thread history, real-time tool progress badges, and rendered Markdown cards. |
| **Authentication Flow** | *(Add screenshot: `docs/screenshots/sign-in.png`)* | Responsive login screen powered by `@descope/nextjs-sdk` supporting social & magic link sign-ins. |
| **Calendar Connection** | *(Add screenshot: `docs/screenshots/connections-panel.png`)* | Left sidebar widget displaying Google Calendar sync status (`connected`, `disconnected`, `pending`) and reconnect triggers. |

---

## 3. Key Features

- **Natural Language Calendar Scheduling:** Schedule meetings with natural language phrases (*"Book 30 minutes tomorrow at 3 PM with Sarah"*), resolving relative times to ISO-8601 timestamps.
- **Deterministic Tool Calling via Mastra:** Gemini 3.5 Flash autonomously routes queries to five specialized calendar tools:
  - `listUpcomingMeetings`: Lists agenda events, with optional `todayOnly=true` daily filtering.
  - `checkCalendarBusy`: Queries Google Calendar `freebusy` endpoint to verify time conflicts.
  - `createMeeting`: Inserts events, adds Google Meet video conference links, and dispatches email invitations.
  - `rescheduleMeeting`: Updates timestamps for existing events and notifies attendees.
  - `cancelMeeting`: Removes calendar events and sends cancellation notices.
- **Live Server-Sent Events (SSE) Streaming:** Real-time token streaming with live progress state badges indicating tool execution stages (`Running listUpcomingMeetings`, `Running createMeeting`).
- **Mastra Working Memory & Thread Persistence:** Persistent thread memory stored in local LibSQL (`mastra.db`), retaining user preferences (e.g., timezone, default meeting length, usual invitees) and conversation history across sessions.
- **Unified Descope Authentication:** Secure authentication using Descope Next.js SDK (`@descope/nextjs-sdk`) on the frontend, Bearer token session validation (`@descope/node-sdk`) on the backend, and Outbound Application token exchange for Google Calendar access.
- **PostgreSQL User & Connection Management:** Persists registered users, authentication IDs, and Google Calendar connection states (`connected`, `disconnected`, `pending`).
- **MCP Server Endpoint:** Built-in Model Context Protocol (MCP) server powered by `@descope/mcp-express` mounted at `/mcp` for agent tool integration with Cursor or external MCP clients.

---

## 4. Example Conversations

### A. Agenda Review
```text
User: What's on my schedule for today?
Agent: [Running listUpcomingMeetings]
Agent: Here is your agenda for today:
  • 10:00 AM - 10:45 AM: Sprint Planning
    [Join Meet](https://meet.google.com/abc-defg-hij)
  • 02:00 PM - 02:30 PM: 1:1 with Alex
    [Join Meet](https://meet.google.com/xyz-uvwx-rst)
```

### B. Availability Check & Meeting Creation
```text
User: Am I free tomorrow afternoon between 3 PM and 4 PM? If so, schedule a catchup with dev@example.com.
Agent: [Running checkCalendarBusy]
Agent: [Running createMeeting]
Agent: You were free during that time. I've scheduled your meeting:
  • Title: Catchup
  • Time: Tomorrow, 3:00 PM - 3:30 PM
  • Attendees: dev@example.com
  • Video: [Join Meet](https://meet.google.com/uvw-xyza-bcd)
```

### C. Conversational Memory & Preferences
```text
User: My default meeting length is 45 minutes, and I work in EST.
Agent: I have updated your working memory preferences. Future meetings will default to 45 minutes in America/New_York unless you specify otherwise.
```

---

## 5. Technology Stack

### Frontend (`/frontend`)
- **Framework:** Next.js `16.3.0` (Turbopack, App Router)
- **UI Engine:** React `19.2.8`, React DOM `19.2.8`
- **Styling:** Tailwind CSS `^4.x`, `@tailwindcss/postcss ^4`, `tw-animate-css`
- **UI Primitives:** `@base-ui/react ^1.7.0`, `shadcn ^4.16.2`, `class-variance-authority ^0.7.1`
- **Icons:** `lucide-react ^1.31.0`
- **Markdown Rendering:** `react-markdown ^10.1.0`, `remark-gfm ^4.0.1`
- **Authentication:** `@descope/nextjs-sdk ^0.15.65`

### Backend (`/backend`)
- **Server Framework:** Express.js `^5.2.1`
- **Language Runtime:** Node.js (v20+ / v22+), TypeScript `^7.0.2` (ES2022 / NodeNext), `tsx ^4.23.11`
- **AI Agent Framework:** `@mastra/core ^1.59.0`
- **Memory & Storage:** `@mastra/memory ^1.26.2`, `@mastra/libsql ^1.20.0`
- **LLM Integration:** Google Gemini 3.5 Flash (`google/gemini-3.5-flash`) via Mastra Model Router
- **Identity & MCP:** `@descope/node-sdk ^2.14.0`, `@descope/mcp-express ^1.6.0`
- **Google Calendar SDK:** `googleapis ^174.0.1`
- **Relational Database:** `pg ^8.23.0` (PostgreSQL Client Pool)
- **Schema Validation:** `zod ^4.4.3`
- **Cross-Origin Resource Sharing:** `cors ^2.8.6`

---

## 6. System Architecture

### Architectural Topology

```mermaid
flowchart TD
    subgraph ClientLayer["Frontend Application (Next.js 16 - Port 3000)"]
        UI["Dashboard & Chat Interface\n(/dashboard, /sign-in)"]
        DescopeSDK["Descope React SDK\n(Session & Refresh Tokens)"]
        StreamReader["SSE Stream Consumer\n(TextDecoder & AgentStreamEvent)"]
    end

    subgraph IdentityGateway["Descope Cloud Platform"]
        AuthService["Descope Identity Service\n(Session Validation)"]
        OutboundBroker["Descope Outbound Application\n(Google Calendar OAuth Broker)"]
    end

    subgraph BackendLayer["Express 5 Backend (Port 4000)"]
        AuthMiddleware["requireSession Middleware\n(Bearer Token AuthContext)"]
        ConnRouter["Connection Router\n(/api/connections)"]
        AgentRouter["Agent Router\n(/api/agent/chat, /threads)"]
        MCPRouter["MCP Server Router\n(/mcp via @descope/mcp-express)"]

        subgraph MastraEngine["Mastra AI Engine"]
            AgentCore["Meeting Assistant Agent\n(model: google/gemini-3.5-flash)"]
            ToolRegistry["Calendar Tools\n(list, checkBusy, create, reschedule, cancel)"]
            WorkingMem["Mastra Memory\n(Preferences + Recalled Context)"]
        end
    end

    subgraph DataStorage["Data Storage Layer"]
        PostgresDB[("PostgreSQL\n(users, connections tables)")]
        LibSQLDB[("LibSQL SQLite\nmastra.db\n(threads, messages, working memory)")]
    end

    subgraph CloudAPIs["External Cloud APIs"]
        GeminiAPI["Google AI Studio\nGemini 3.5 Flash API"]
        GCalAPI["Google Calendar API v3\n(events, freebusy)"]
    end

    UI -->|1. Authenticate| DescopeSDK
    DescopeSDK <-->|OAuth Tokens| AuthService

    UI -->|2. HTTP /api/connections| AuthMiddleware
    UI -->|3. SSE POST /api/agent/chat| AuthMiddleware

    AuthMiddleware -->|Validate Session| AuthService
    AuthMiddleware -->|Ensure User Record| PostgresDB
    ConnRouter -->|Persist Sync Status| PostgresDB
    ConnRouter -->|Exchange Outbound Token| OutboundBroker

    AgentRouter --> AgentCore
    AgentCore <--> WorkingMem
    WorkingMem <--> LibSQLDB
    AgentCore --> ToolRegistry
    AgentCore <-->|Generate / Stream| GeminiAPI

    ToolRegistry -->|Fetch OAuth Token| OutboundBroker
    ToolRegistry <-->|Calendar Actions| GCalAPI
    AgentRouter -->|data: chunk events| StreamReader
```

### Request Lifecycle & Agent Execution Loop

```mermaid
sequenceDiagram
    autonumber
    actor User as User
    participant Frontend as Frontend (Next.js)
    participant Backend as Backend (Express)
    participant Descope as Descope Auth & Outbound
    participant Mastra as Mastra Agent & Memory
    participant Gemini as Google Gemini 3.5 Flash
    participant GCal as Google Calendar API v3

    User->>Frontend: Enters scheduling request
    Frontend->>Backend: POST /api/agent/chat (Bearer Token, Thread ID, Message)
    Backend->>Descope: Validate Bearer session token
    Descope-->>Backend: Token valid (claims: sub, email)
    Backend->>Mastra: Initialize Agent with thread memory & calendar tools
    Mastra->>Backend: Send SSE event: {"type": "started", "message": "Agent is planning"}
    Backend-->>Frontend: Stream SSE 'started'
    Mastra->>Gemini: Prompt + System Instructions + Tool Definitions + History
    Gemini-->>Mastra: Tool Call Decision (e.g. createMeeting)
    Mastra->>Backend: Send SSE event: {"type": "progress", "message": "Running createMeeting"}
    Backend-->>Frontend: Stream SSE 'progress'
    Mastra->>Descope: Fetch Google Calendar OAuth Access Token
    Descope-->>Mastra: Active Access Token
    Mastra->>GCal: Insert event with conferenceData (Google Meet)
    GCal-->>Mastra: Created event details + Meet URL
    Mastra->>Gemini: Return tool execution payload
    Gemini-->>Mastra: Final formatted response
    Mastra->>Backend: Send SSE tokens: {"type": "token", "token": "..."}
    Backend-->>Frontend: Stream SSE tokens
    Mastra->>Mastra: Write turn to mastra.db & update thread title
    Backend-->>Frontend: Send SSE: {"type": "completed", "message": "done"}
```

---

## 7. Project Folder Structure

```text
agentic-calendar-assistant/
├── docker-compose.yml              # PostgreSQL 16 container definition
├── README.md                       # Complete project documentation
├── backend/
│   ├── .env                        # Backend environment configuration
│   ├── .env.example                # Example backend environment template
│   ├── package.json                # Express, Mastra, Descope & Google API dependencies
│   ├── tsconfig.json               # Backend TypeScript configuration (ES2022 / NodeNext)
│   ├── mastra.db                   # Local LibSQL SQLite memory & conversation store
│   ├── scripts/
│   │   └── migrate.ts              # SQL migration execution script
│   ├── sql/
│   │   ├── 001_users.sql           # Schema definition for 'users' table
│   │   └── 002_connections.sql     # Schema definition for 'connections' table
│   └── src/
│       ├── index.ts                # Application entrypoint & HTTP server bootstrap
│       ├── config/
│       │   ├── agent-instructions.ts # System prompt & behavioral scheduling guidelines
│       │   ├── descope.ts          # Descope client & connection constants
│       │   └── memory.ts           # LibSQLStore & Mastra working memory setup
│       ├── db/
│       │   └── pool.ts             # PostgreSQL connection pool manager
│       ├── mcp/
│       │   ├── calendar-tools.ts   # Descope MCP tool definition for calendar
│       │   └── mount.ts            # MCP endpoint router setup (/mcp)
│       ├── middleware/
│       │   └── requireSession.ts   # Descope token validation & Request auth context
│       ├── repositories/
│       │   ├── connection.repository.ts # PostgreSQL queries for connection state
│       │   └── user.repository.ts       # PostgreSQL queries for user provisioning
│       ├── routes/
│       │   ├── agent.routes.ts     # Express routes: /api/agent/threads and /api/agent/chat
│       │   └── connection.routes.ts# Express routes: /api/connections
│       └── services/
│           ├── agent.service.ts    # Mastra agent initialization and streaming logic
│           ├── agent-tools.service.ts # Mastra tool definitions (@mastra/core/tools)
│           ├── calendar.service.ts # Google Calendar API v3 operations
│           ├── connection.service.ts# Calendar connection lifecycle & token checks
│           └── token.service.ts    # Descope Outbound token fetching
└── frontend/
    ├── .env                        # Frontend environment configuration
    ├── package.json                # Next.js 16, React 19, Tailwind 4 dependencies
    ├── next.config.ts              # Next.js framework configuration
    ├── tsconfig.json               # Frontend TypeScript configuration
    ├── public/                     # Public static SVG assets
    └── src/
        ├── app/
        │   ├── layout.tsx          # Root layout with Descope AuthProvider
        │   ├── page.tsx            # Root redirect to /dashboard
        │   ├── globals.css         # Tailwind 4 utility imports & theme design tokens
        │   ├── dashboard/
        │   │   └── page.tsx        # Dashboard page with auth guard & side panels
        │   └── sign-in/
        │       └── page.tsx        # Descope Sign-In page
        ├── components/
        │   ├── auth/
        │   │   ├── RedirectIfAuthenticated.tsx # Auth redirect guard
        │   │   └── sign-in.tsx     # Descope Flow widget wrapper
        │   ├── dashboard/
        │   │   ├── chat-panel.tsx  # Chat interface, thread management & SSE reader
        │   │   ├── connection-panel.tsx # Google Calendar connection widget
        │   │   └── markdown-message.tsx # Markdown renderer for agent replies
        │   └── ui/                 # Reusable UI primitives (Button, Card, Skeleton, etc.)
        └── lib/
            ├── agent.ts            # Client-side agent streaming & thread fetching
            ├── api.ts              # Typed fetch wrapper with Bearer token injection
            ├── connections.ts      # Client functions to connect & refresh calendar status
            ├── types.ts            # UI and connection type definitions
            └── utils.ts            # Classnames merger (clsx + tailwind-merge)
```

---

## 8. Prerequisites

Before setting up the project locally, verify you have:

1. **Node.js:** `v20.x` or `v22.x` (LTS recommended).
2. **PostgreSQL:** Local PostgreSQL 15+ installation or Docker Desktop running.
3. **Google AI Studio API Key:** Access key with permissions for Gemini models ([Get API Key](https://aistudio.google.com/)).
4. **Descope Project:**
   - Descope Project ID ([Descope Console](https://app.descope.com/)).
   - Descope Management Key.
   - Configured **Outbound Application** for Google Calendar (`DESCOPE_CALENDAR_CONNECTION_ID=google-calendar`).
5. **Google Cloud Console:**
   - Active project with the **Google Calendar API** enabled.
   - OAuth 2.0 Client credentials registered inside the Descope Outbound Application settings.

---

## 9. Local Installation & Setup

### Step 1: Clone Repository
```bash
git clone <repository-url>
cd agentic-calendar-assistant
```

### Step 2: Database Initialization
Start PostgreSQL via the included Docker Compose configuration:
```bash
docker compose up -d postgres
```
*(Or verify your local PostgreSQL server is running and create database `agentic_calendar_app_db`).*

### Step 3: Backend Setup
1. Navigate into `backend` and install dependencies:
   ```bash
   cd backend
   npm install
   ```
2. Create `backend/.env` (see [Environment Variables](#10-environment-variables)).
3. Run the PostgreSQL migrations:
   ```bash
   npm run migrate
   ```
   *Expected Output:*
   ```text
   Migrated: sql/001_users.sql
   Migrated: sql/002_connections.sql
   ```

### Step 4: Frontend Setup
1. Open a new terminal, navigate into `frontend`, and install dependencies:
   ```bash
   cd ../frontend
   npm install
   ```
2. Create `frontend/.env` (see [Environment Variables](#10-environment-variables)).

### Step 5: Run Development Servers

**Start Backend (Terminal 1):**
```bash
cd backend
npm run dev
```
*Expected Output:*
```text
Agentic Calendar App is running on port: 4000
```
Verify backend health:
```bash
curl http://localhost:4000/health
# Response: {"status":"ok","service":"agentic-calendar-app","database":"up"}
```

**Start Frontend (Terminal 2):**
```bash
cd frontend
npm run dev
```
*Expected Output:*
```text
▲ Next.js 16.3.0 (Turbopack)
- Local: http://localhost:3000
```

---

## 10. Environment Variables

### Backend Environment (`backend/.env`)

| Variable Name | Required | Description | Example Placeholder |
|---|---|---|---|
| `PORT` | Optional | Port for Express HTTP server (default: `4000`) | `4000` |
| `APP_URL` | **Required** | Allowed frontend origin for CORS and OAuth redirects | `http://localhost:3000` |
| `DATABASE_URL` | **Required** | PostgreSQL connection string | `postgresql://postgres:postgres@localhost:5432/agentic_calendar_app_db` |
| `DESCOPE_PROJECT_ID` | **Required** | Project ID from Descope dashboard | `P3LuqjaK...` |
| `DESCOPE_MANAGEMENT_KEY` | **Required** | Management Key from Descope dashboard | `K3Luwwkx...` |
| `DESCOPE_CALENDAR_CONNECTION_ID` | **Required** | Outbound App ID in Descope (default: `google-calendar`) | `google-calendar` |
| `GEMINI_API_KEY` | **Required** | Google AI Studio Gemini API Key | `AIzaSy...` |
| `GOOGLE_API_KEY` | Optional | Synced automatically from `GEMINI_API_KEY` by backend | `AIzaSy...` |
| `AI_MODEL` | Optional | Gemini Model Identifier (default: `gemini-3.5-flash`) | `gemini-3.5-flash` |
| `SERVER_URL` | Optional | Backend URL required when exposing the MCP server | `https://your-domain.ngrok-free.dev` |
| `DESCOPE_MCP_SERVER_WELL_KNOWN_URL` | Optional | OpenID configuration URL for MCP clients | `https://api.descope.com/v1/...` |

### Frontend Environment (`frontend/.env`)

| Variable Name | Required | Description | Example Placeholder |
|---|---|---|---|
| `NEXT_PUBLIC_DESCOPE_PROJECT_ID` | **Required** | Descope Project ID for client-side AuthProvider | `P3LuqjaK...` |
| `NEXT_PUBLIC_API_URL` | **Required** | Base URL for the Express backend API | `http://localhost:4000` |

---

## 11. API Endpoint Documentation

All `/api/*` endpoints require header `Authorization: Bearer <token>`.

### System Health
```http
GET /health
```
- **Auth:** None
- **Response (200 OK):**
  ```json
  {
    "status": "ok",
    "service": "agentic-calendar-app",
    "database": "up"
  }
  ```

### Connections Endpoints

```http
GET /api/connections
```
- **Description:** Returns the current Google Calendar connection status for the authenticated user.
- **Response (200 OK):**
  ```json
  {
    "connection": {
      "label": "Google Calendar",
      "status": "connected"
    }
  }
  ```

```http
POST /api/connections/connect
Content-Type: application/json

{
  "refreshToken": "<descope-refresh-token>",
  "redirectUrl": "http://localhost:3000/dashboard"
}
```
- **Description:** Generates a Descope Outbound Application OAuth redirect URL.
- **Response (200 OK):**
  ```json
  {
    "url": "https://api.descope.com/v1/apps/agentic/authorize?..."
  }
  ```

```http
POST /api/connections/refresh-status
```
- **Description:** Verifies that a valid access token exists in Descope for Google Calendar and updates the PostgreSQL `connections` table.
- **Response (200 OK):**
  ```json
  {
    "connection": {
      "label": "Google Calendar",
      "status": "connected"
    }
  }
  ```

### Agent Endpoints

```http
GET /api/agent/threads
```
- **Description:** Returns conversation threads associated with the authenticated user from Mastra LibSQL memory.
- **Response (200 OK):**
  ```json
  {
    "threads": [
      {
        "id": "e674b83b-3d60-49ef-ba38-348625ec1d3f",
        "title": "Discussing Project Milestones",
        "updatedAt": "2026-11-11T12:00:00.000Z"
      }
    ]
  }
  ```

```http
GET /api/agent/threads/:threadId
```
- **Description:** Fetches all conversational messages within a specific thread.
- **Response (200 OK):**
  ```json
  {
    "threadId": "e674b83b-3d60-49ef-ba38-348625ec1d3f",
    "messages": [
      {
        "id": "f291a13b-...",
        "role": "user",
        "content": "What meetings do I have tomorrow?"
      },
      {
        "id": "a984b21c-...",
        "role": "assistant",
        "content": "You have 2 meetings scheduled tomorrow..."
      }
    ]
  }
  ```

```http
POST /api/agent/chat
Content-Type: application/json
Accept: text/event-stream

{
  "threadId": "e674b83b-3d60-49ef-ba38-348625ec1d3f",
  "message": "Schedule a 30 min sync with Bob tomorrow at 2 PM"
}
```
- **Description:** Dispatches user input to the Mastra agent and streams results via Server-Sent Events.
- **Event Chunks (`text/event-stream`):**
  ```text
  data: {"type":"started","message":"Agent is planning"}

  data: {"type":"progress","message":"Running createMeeting"}

  data: {"type":"token","token":"I have"}
  data: {"type":"token","token":" scheduled your meeting with Bob."}

  data: {"type":"completed","message":"done"}
  ```

---

## 12. Database & Memory Architecture

The system segregates relational user state from unstructured conversation memory:

```
┌──────────────────────────────────────────────┐
│           PostgreSQL Database                │
│  - users (id, auth_user_id, email, created)  │
│  - connections (user_id, provider, status)   │
└──────────────────────────────────────────────┘
                       ▲
                       │ Application State & User Map
┌──────────────────────┴───────────────────────┐
│         Backend Server (Express)             │
└──────────────────────┬───────────────────────┘
                       │ Mastra Working Memory & Threads
                       ▼
┌──────────────────────────────────────────────┐
│            LibSQL SQLite (mastra.db)         │
│  - Conversation message history              │
│  - Thread metadata & auto-generated titles   │
│  - Extracted user scheduling preferences     │
└──────────────────────────────────────────────┘
```

1. **PostgreSQL Relational Store:**
   - **`users`:** Tracks internal UUIDs (`id`), Descope authentication user IDs (`auth_user_id`), and user emails.
   - **`connections`:** Manages third-party connection states (`user_id`, `provider='calendar'`, `status='connected'|'disconnected'|'pending'`).
2. **LibSQL Store (`mastra.db`):**
   - Managed directly by `@mastra/libsql` and `@mastra/memory`.
   - Stores raw messages, message parts, and thread metadata.
   - Maintains a structured working memory document (`scope: "resource"`):
     ```markdown
     # Meeting preferences
     - Timezone:
     - Default meeting length (minutes):
     - Preferred meeting hours:
     - Usual invitees:
     - Notes:
     ```

---

## 13. Authentication & Google Calendar OAuth Flow

1. **User Authentication:** 
   The frontend uses `@descope/nextjs-sdk`. The `AuthProvider` handles user tokens in browser cookies (`sameSite: "Lax"`). Requests to the Express API pass the session token as a `Bearer` token.
2. **Backend Authentication Guard:**
   `backend/src/middleware/requireSession.ts` validates the token using `descopeClient.validateSession(token)`. If valid, it ensures a matching user row exists in PostgreSQL and populates `req.auth`.
3. **Google Calendar OAuth via Descope Outbound Applications:**
   Rather than storing sensitive Google OAuth refresh tokens in PostgreSQL, the backend leverages **Descope Outbound Application Management**:
   - `connectionRouter.post("/connect")` initiates an outbound connection flow with Descope for `DESCOPE_CALENDAR_CONNECTION_ID`.
   - When tools execute calendar actions, `token.service.ts` requests a fresh Google OAuth access token on demand:
     ```typescript
     const response = await descopeClient.management.outboundApplication.fetchToken(
       CALENDAR_CONNECTION_ID,
       authUserId
     );
     ```

---

## 14. Model Context Protocol (MCP) Integration

The backend includes a dedicated MCP endpoint mounted at `/mcp` using `@descope/mcp-express`:
- Defined in `backend/src/mcp/mount.ts` and `backend/src/mcp/calendar-tools.ts`.
- Exposes `listUpcomingMeetings` as a standard MCP tool.
- Supports external AI developer environments (such as **Cursor**) or remote MCP orchestrators by authenticating incoming requests via `DESCOPE_MCP_SERVER_WELL_KNOWN_URL`.

---

## 15. Deployment Guide

### Recommended Architecture:

| Component | Target Platform | Justification |
|---|---|---|
| **Frontend** | **Vercel** | Zero-configuration Next.js 16 deployment, fast build caching, global CDN, and automated SSL. |
| **Backend** | **Render / Railway** | Native long-running Node.js process hosting, seamless environment variable configuration, and managed deployment triggers. |
| **PostgreSQL** | **Neon / Supabase / Render Postgres** | Managed serverless PostgreSQL with connection pooling, SSL, and daily automated backups. |
| **Agent Memory** | **Persistent Disk / Turso** | Mount a persistent disk to preserve `mastra.db`, or configure `@mastra/libsql` to connect to a managed Turso instance. |

---

### Step-by-Step Deployment Instructions

#### 1. Database (Neon or Supabase)
1. Create a PostgreSQL project.
2. Retrieve the pooled connection string with SSL:
   ```text
   postgresql://username:password@ep-sample.region.neon.tech/neondb?sslmode=require
   ```
3. Run migrations against your production database:
   ```bash
   cd backend
   DATABASE_URL="<production-connection-string>" npm run migrate
   ```

#### 2. Backend (Render / Railway)
1. Create a new **Web Service** pointing to your repository.
2. Set the **Root Directory** to `backend`.
3. Build & Start commands:
   - **Build Command:** `npm install && npm run build`
   - **Start Command:** `npm run start`
4. Set backend environment variables:
   - `PORT`: `10000`
   - `APP_URL`: `https://your-frontend.vercel.app`
   - `DATABASE_URL`: `<production-postgres-connection-string>`
   - `DESCOPE_PROJECT_ID`: `<your-descope-project-id>`
   - `DESCOPE_MANAGEMENT_KEY`: `<your-descope-management-key>`
   - `DESCOPE_CALENDAR_CONNECTION_ID`: `google-calendar`
   - `GEMINI_API_KEY`: `<your-google-ai-studio-api-key>`
   - `AI_MODEL`: `gemini-3.5-flash`

#### 3. Frontend (Vercel)
1. Import repository on Vercel and select root directory `frontend`.
2. Configure environment variables:
   - `NEXT_PUBLIC_DESCOPE_PROJECT_ID`: `<your-descope-project-id>`
   - `NEXT_PUBLIC_API_URL`: `https://your-backend.onrender.com`
3. Deploy.

#### 4. Update OAuth Redirects
- In **Descope Console**: Update authorized domains with your Vercel URL and set outbound application redirects to `https://your-frontend.vercel.app/dashboard`.
- In **Google Cloud Console**: Add your Vercel URL and Descope callback URL to your OAuth 2.0 authorized origins and redirect URIs.

---

## 16. Docker Deployment

To spin up local infrastructure, use the included `docker-compose.yml`:

```bash
# Launch PostgreSQL 16 container
docker compose up -d postgres

# Verify container status
docker compose ps

# Stop container
docker compose down
```

---

## 17. Production Readiness Checklist

- [ ] **Database Connection:** Verified managed PostgreSQL connection with SSL enabled (`sslmode=require`).
- [ ] **Migrations Executed:** Verified `users` and `connections` tables exist in production database.
- [ ] **Gemini API Key:** Verified `GEMINI_API_KEY` from Google AI Studio has sufficient quota and is set in backend environment.
- [ ] **Descope Configuration:** Verified `DESCOPE_PROJECT_ID` and `DESCOPE_MANAGEMENT_KEY` are active.
- [ ] **Calendar Outbound App:** Confirmed `google-calendar` outbound app is enabled with Google Calendar scopes (`https://www.googleapis.com/auth/calendar`).
- [ ] **CORS Settings:** Verified `APP_URL` on the backend matches the frontend deployment domain.
- [ ] **HTTPS Enforced:** SSL certificates active on both frontend (Vercel) and backend (Render).
- [ ] **Zero Exposed Secrets:** Confirmed `.env` files are in `.gitignore` and omitted from version control.

---

## 18. Known Limitations

- **Local Memory File in Multi-Instance Deployments:** Mastra uses a local SQLite database (`mastra.db`) by default. If running across multiple container replicas without a shared volume or Turso database, memory states will not synchronize between replicas.
- **Single Calendar Provider:** The database schema and tools currently support Google Calendar. Microsoft 365 / Outlook integration is planned for future releases.
- **Calendar Event Window:** Default queries without explicit dates list upcoming events within the immediate planning horizon.

---

## 19. Troubleshooting Guide

<details>
<summary><strong>1. "password authentication failed for user 'postgres'"</strong></summary>

- **Cause:** Local PostgreSQL credentials differ from the connection string in `backend/.env`.
- **Solution:** Verify the password in `DATABASE_URL`. If using Docker, ensure the Docker container is started via `docker compose up -d postgres`.
</details>

<details>
<summary><strong>2. "Could not find API key process.env.GOOGLE_API_KEY..."</strong></summary>

- **Cause:** `GEMINI_API_KEY` or `GOOGLE_API_KEY` is missing from the environment.
- **Solution:** Add `GEMINI_API_KEY="your_key"` to `backend/.env`. The backend automatically copies this value to `GOOGLE_API_KEY` for Mastra's Google provider router.
</details>

<details>
<summary><strong>3. Stuck on "Checking session..." on Frontend</strong></summary>

- **Cause:** No active Descope session or unauthenticated state.
- **Solution:** Verify `NEXT_PUBLIC_DESCOPE_PROJECT_ID` in `frontend/.env`. The frontend automatically redirects unauthenticated users to `/sign-in`.
</details>

<details>
<summary><strong>4. CORS Failure during SSE Streaming</strong></summary>

- **Cause:** `APP_URL` in `backend/.env` does not match the frontend URL.
- **Solution:** Ensure `APP_URL` matches the frontend address (`http://localhost:3000` locally, or your production Vercel domain).
</details>

<details>
<summary><strong>5. "Calendar access token is not present, please check or reconnect"</strong></summary>

- **Cause:** Google Calendar OAuth has not been completed for the logged-in user in Descope.
- **Solution:** On the Dashboard sidebar, click **Connect** next to Google Calendar, finish the Google OAuth consent flow, and click the **Refresh** icon.
</details>

---

## 20. License

This project is licensed under the [MIT License](LICENSE).
