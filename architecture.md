 # High Level Architecture

## Technical Summary

This architecture outlines a distributed system based on a microservice pattern, all managed within a monorepo. The frontend is a modern server-side rendered application using Next.js. It communicates with backend services through an API Gateway. The backend is split into general-purpose services (like user authentication) built with Express.js and a specialized AI service built with Python and LangGraph. All services are stateless and rely on a central PostgreSQL database for persistence.

## Platform and Infrastructure Choice

The system will be deployed on a traditional hosting environment capable of running multiple services.

**Platform**: Traditional Hosted Server (VPS or Dedicated Server, e.g., Iranserver).

**Key Services**:

  * API Gateway / Reverse Proxy (e.g., Nginx) to route traffic.
  * Node.js Runtime (for Next.js SSR and Express.js services).
  * Python Runtime (for the AI service).
  * PostgreSQL Database Server.

## Repository Structure

**Structure**: Monorepo

**Rationale**: This was decided in the PRD to simplify management of shared code (e.g., data types) between the web application and the backend API.

## High Level Architecture Diagram

```mermaid
graph TD
    A[User's Browser] --> B[Next.js Frontend];
    
    subgraph "Hosted Server Environment"
        B --> C{API Gateway / Nginx};
        C --> D[Auth Service (Express.js)];
        C --> E[Meeting Service (Express.js)];
        C --> F[AI Service (Python/LangGraph)];
        
        D --> G[(PostgreSQL DB)];
        E --> G;
        E --> F;
    end
```

## Architectural Patterns

  * **Microservice Architecture**: The backend logic is separated into smaller, independent services (e.g., an Auth Service for users, a Meeting Service for core logic, and an AI Service for processing). This allows for independent development, deployment, and scaling.
  * **API Gateway**: A single entry point (Nginx) for the frontend. It routes incoming requests to the appropriate microservice, simplifying frontend logic and centralizing concerns like rate limiting and SSL termination.
  * **Component-Based UI**: The Next.js frontend will be built using reusable, self-contained components.
  * **Repository Pattern**: The Express.js services will use the repository pattern to abstract data access logic from the PostgreSQL database.

-----

### `tech-stack.md`

This file is the definitive list of all technologies, libraries, and versions approved for this project. It is a critical document that ensures all developers are building with a consistent and agreed-upon set of tools.

# Tech Stack

| Category              | Technology        | Version | Purpose                    | Rationale                                                   |
| --------------------- | ----------------- | ------- | -------------------------- | ----------------------------------------------------------- |
| Frontend Language     | TypeScript        | 5.4+    | Primary frontend language  | Type safety for large applications, standard for Next.js.   |
| Full-Stack Framework  | Next.js           | 14+     | Web app framework for UI & API | Provides SSR, routing, and a robust full-stack ecosystem.   |
| UI Component Library  | shadcn/ui         | latest  | Component library          | Unstyled, accessible components that are highly customizable. |
| Styling               | Tailwind CSS      | 3.4+    | CSS framework              | Utility-first CSS for rapid and consistent styling.         |
| State Management      | Zustand           | 4.5+    | Global state management    | Simple, lightweight state management solution for React.    |
| AI Language           | Python            | 3.11+   | AI service language        | Industry standard for AI/ML, required for LangGraph.        |
| AI Framework          | LangGraph         | latest  | AI agent orchestration     | State machine library for building complex AI agents.       |
| API Style             | REST              | n/a     | API communication style    | Well-understood, stateless, and simple for microservices.   |
| Database              | PostgreSQL        | 16+     | Relational database        | Robust, open-source SQL database for structured data.       |
| Frontend Testing      | Vitest & RTL      | latest  | Component testing          | Modern, fast testing framework for React components.        |
| Backend Testing       | Jest & Supertest  | latest  | API endpoint testing       | Standard for testing Node.js API Routes.                    |
| E2E Testing           | Playwright        | latest  | End-to-end testing         | Reliable browser automation for testing user flows.         |
| Containerization      | Docker            | latest  | Service containerization   | Standardizes deployment environments for all services.      |
| CI/CD                 | GitHub Actions    | n/a     | Continuous integration/deploy | Tightly integrated with GitHub for automated builds and tests. |

-----

### `data-models.md`

This file contains the TypeScript interfaces for our core data structures. These models ensure that both our frontend and backend applications handle data consistently, which is critical for preventing integration bugs. They are the code-level representation of our approved database schema.

# Data Models (TypeScript Interfaces)

Here are the code-level models based on the database structure:

```typescript
// USERS AND AUTH
export interface User {
  id: number;
  full_name: string;
  email: string;
  voiceprint?: string | null; // Reference to a secure file location
  role?: string | null;
  created_at: Date;
  status: 'active' | 'inactive' | 'pending';
}

export interface Project {
  id: number;
  name: string;
  description?: string | null;
  created_by: number; // User ID
  created_at: Date;
}

// MEETINGS
export interface Meeting {
  id: number;
  title: string;
  owner_id: number; // User ID
  project_id?: number | null; // Project ID
  started_at: Date;
  ended_at?: Date | null;
  audio_file?: string | null; // Reference to cloud storage
  transcript_file?: string | null; // Reference to cloud storage
  status: 'active' | 'archived';
}

export interface MeetingAttendee {
  id: number;
  meeting_id: number;
  user_id: number;
  joined_at: Date;
  voiceprint_matched: boolean;
}

// DETAILED TRANSCRIPT & TOPICS
export interface TranscriptSegment {
  id: number;
  meeting_id: number;
  speaker_id: number; // User ID
  start_time: number;
  end_time: number;
  transcript: string;
  main_subject?: string | null;
  sub_subject?: string | null;
  is_conclusion: boolean;
  flagged_by_owner: boolean;
}

// TASKS, ASSIGNMENTS, APPROVALS
export interface Task {
  id: number;
  meeting_id: number;
  created_by: number; // User ID
  summary: string;
  details?: string | null;
  ai_suggested: boolean;
  status: 'pending' | 'approved' | 'rejected' | 'assigned' | 'done';
  approved_by?: number | null; // User ID
  approved_at?: Date | null;
}

export interface TaskAssignment {
  task_id: number;
  user_id: number;
  assigned_at: Date;
  completed_at?: Date | null;
}
```

-----

### `api-specification.md`

This file defines the strict contract between the frontend and backend services. Following this specification is crucial for the development teams to work in parallel without causing integration issues. It ensures everyone agrees on how the different parts of the system will communicate.

# API Specification

```yaml
openapi: 3.0.1
info:
  title: Meeting Assistant API
  description: API for the Meeting Assistant application, including Auth and Meeting services.
  version: 1.0.0

servers:
  - url: /api/v1
    description: API Gateway

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

  schemas:
    UserRegistration:
      type: object
      properties:
        fullName:
          type: string
        email:
          type: string
          format: email
        password:
          type: string
          format: password
    
    AuthCredentials:
      type: object
      properties:
        email:
          type: string
          format: email
        password:
          type: string
          
    TokenResponse:
      type: object
      properties:
        accessToken:
          type: string

    Error:
      type: object
      properties:
        errorCode:
          type: string
          example: "EMAIL_IN_USE"
        message:
          type: string
          example: "This email address is already registered."

security:
  - bearerAuth: []

paths:
  /users:
    post:
      summary: Register a new user
      tags:
        - Auth Service
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UserRegistration'
      responses:
        '201':
          description: User created successfully
          headers:
            Location:
              description: The URL of the newly created user.
              schema:
                type: string
                format: uri
        '400':
          description: Invalid input
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'

  /auth/tokens:
    post:
      summary: Authenticate a user and create a session token
      tags:
        - Auth Service
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/AuthCredentials'
      responses:
        '200':
          description: Authentication successful
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/TokenResponse'
        '401':
          description: Unauthorized
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'

  /meetings:
    post:
      summary: Create a new meeting and upload audio
      tags:
        - Meeting Service
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          multipart/form-data:
            schema:
              type: object
              properties:
                title:
                  type: string
                audio:
                  type: string
                  format: binary
      responses:
        '202':
          description: Meeting accepted for processing
        '401':
          description: Unauthorized
```

-----

### `components.md`

This file describes the high-level responsibilities of our backend microservices. It defines the boundaries between them, which is essential for the development teams to understand who owns which piece of the application logic.

# Components

Our architecture is composed of three primary backend services. They will be managed as separate applications within our monorepo.

### Auth Service

  * **Responsibility**: Manages all user-related concerns, including registration, login, session management, and profile data (including voiceprints).
  * **Key Interfaces (REST API)**:
      * `POST /register`: Creates a new user.
      * `POST /login`: Authenticates a user and returns a session token.
      * `GET /profile`: Retrieves the profile of the currently logged-in user.
      * `PUT /profile`: Updates the profile of the currently logged-in user.
  * **Dependencies**: Directly interacts with the `users`, `user_projects`, and `projects` tables in the PostgreSQL database.
  * **Technology**: Express.js, TypeScript.

### Meeting Service

  * **Responsibility**: Handles the core application logic. It manages the creation and retrieval of meetings, orchestrates the analysis pipeline by calling the AI Service, and handles task management.
  * **Key Interfaces (REST API)**:
      * `POST /meetings`: Creates a new meeting record and accepts the audio upload.
      * `GET /meetings/{id}`: Retrieves the full analysis results for a completed meeting.
      * `POST /tasks/{id}/approve`: Allows a supervisor to approve a pending task.
  * **Dependencies**: Interacts with most tables in the database. It is the primary orchestrator and is the only service that calls the AI Service.
  * **Technology**: Express.js, TypeScript.

### AI Service

  * **Responsibility**: Performs all heavy computational and AI-driven analysis. This is a "headless" service that accepts data, processes it using Python and LangGraph, and returns a structured result.
  * **Key Interfaces (Internal REST API)**:
      * `POST /analyze`: Accepts preprocessed meeting data and returns the full analysis object (transcript, mindmap, summary, etc.).
  * **Dependencies**: This service is stateless. It does not connect directly to the database. It receives all necessary data from the Meeting Service.
  * **Technology**: Python, LangGraph.

-----

### `core-workflows.md`

This file contains a sequence diagram that visually represents the main process flow of the application—from a user's action to the final result. This is a vital document for developers to understand how the different microservices interact with each other.

# Core Workflows

## New Meeting Processing Workflow

This sequence diagram illustrates how the different components interact when a user records a new meeting and the system generates the analysis.

```mermaid
sequenceDiagram
    participant User
    participant Browser (Next.js)
    participant API Gateway
    participant Meeting Service
    participant AI Service
    participant Database

    User->>Browser (Next.js): Clicks "Stop & Save" Recording
    Browser (Next.js)->>API Gateway: POST /meetings (with audio)
    API Gateway->>Meeting Service: forwards request
    
    Meeting Service->>Database: CREATE Meeting record (status: 'processing')
    Meeting Service->>AI Service: POST /analyze (with meeting data)
    
    activate AI Service
    AI Service-->>Meeting Service: returns Analysis JSON
    deactivate AI Service
    
    Meeting Service->>Database: SAVE Analysis, Tasks, etc.
    Meeting Service->>Database: UPDATE Meeting record (status: 'completed')
    
    Meeting Service-->>API Gateway: responds with success (meetingId)
    API Gateway-->>Browser (Next.js): responds with success
    Browser (Next.js)-->>User: Shows "Meeting is processing"
```

-----

### `database-schema.md`

This file contains the detailed SQL structure for our database. This is the blueprint the development team will use to build the data layer of our application.

# Database Schema

```sql
-- USERS AND AUTH
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    full_name VARCHAR(120) NOT NULL,
    email VARCHAR(180) UNIQUE NOT NULL,
    voiceprint BYTEA,         -- actual print sample, or store reference to secure file
    role VARCHAR(60),         -- e.g. "manager", "developer" etc.
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    status VARCHAR(32) DEFAULT 'active' -- can be "active", "inactive", "pending"
);

CREATE TABLE user_projects (
    user_id INT REFERENCES users(id),
    project_id INT REFERENCES projects(id),
    PRIMARY KEY (user_id, project_id)
);

-- STORE USER'S CURRENT TASK ASSIGNMENTS
CREATE TABLE user_tasks (
    user_id INT REFERENCES users(id),
    task_id INT REFERENCES tasks(id),
    assigned_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    PRIMARY KEY (user_id, task_id)
);

-- PROJECTS OVER ARCHITECTURE
CREATE TABLE projects (
    id SERIAL PRIMARY KEY,
    name VARCHAR(120) NOT NULL,
    description TEXT,
    created_by INT REFERENCES users(id),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- MEETINGS
CREATE TABLE meetings (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    owner_id INT REFERENCES users(id),
    project_id INT REFERENCES projects(id), -- optional, meetings can float outside
    started_at TIMESTAMPTZ NOT NULL,
    ended_at TIMESTAMPTZ,
    audio_file VARCHAR(255), -- reference to cloud storage/file system
    transcript_file VARCHAR(255),
    status VARCHAR(32) DEFAULT 'active' -- "active", "archived"
);

-- MEETING PARTICIPANTS (resolved by voiceprint, linked to user)
CREATE TABLE meeting_attendees (
    id SERIAL PRIMARY KEY,
    meeting_id INT REFERENCES meetings(id),
    user_id INT REFERENCES users(id),
    joined_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    voiceprint_matched BOOLEAN DEFAULT TRUE
);

-- DETAILED TRANSCRIPT & TOPICS
CREATE TABLE transcript_segments (
    id SERIAL PRIMARY KEY,
    meeting_id INT REFERENCES meetings(id),
    speaker_id INT REFERENCES users(id),
    start_time FLOAT NOT NULL,           -- in seconds, aligns to audio
    end_time FLOAT NOT NULL,
    transcript TEXT NOT NULL,
    main_subject VARCHAR(255),
    sub_subject VARCHAR(255),
    is_conclusion BOOLEAN DEFAULT FALSE,
    flagged_by_owner BOOLEAN DEFAULT FALSE
);

-- TASKS, ASSIGNMENTS, APPROVALS
CREATE TABLE tasks (
    id SERIAL PRIMARY KEY,
    meeting_id INT REFERENCES meetings(id), -- linkage to discussion context
    created_by INT REFERENCES users(id),
    summary VARCHAR(255) NOT NULL,
    details TEXT,
    ai_suggested BOOLEAN DEFAULT FALSE,     -- AI-extracted suggestion
    status VARCHAR(32) DEFAULT 'pending',   -- e.g., pending, approved, rejected, assigned, done
    approved_by INT REFERENCES users(id),   -- human supervisor
    approved_at TIMESTAMPTZ
);

-- TASK ASSIGNMENTS: multiple users per task allowed
CREATE TABLE task_assignments (
    task_id INT REFERENCES tasks(id),
    user_id INT REFERENCES users(id),
    assigned_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    completed_at TIMESTAMPTZ,
    PRIMARY KEY (task_id, user_id)
);

-- MEETING FLAGS/CONCLUSIONS
CREATE TABLE conclusions (
    id SERIAL PRIMARY KEY,
    meeting_id INT REFERENCES meetings(id),
    flagged_by INT REFERENCES users(id),
    flagged_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    transcript_segment_id INT REFERENCES transcript_segments(id),
    summary TEXT
);

-- AUDIT TRAIL (for NFRs)
CREATE TABLE audit_logs (
    id SERIAL PRIMARY KEY,
    user_id INT REFERENCES users(id),
    action VARCHAR(255),
    details TEXT,
    log_time TIMESTAMPTZ DEFAULT NOW()
);

-- CONTEXTUAL/KNOWLEDGEBASE FOR CONTEXTUAL AI
CREATE TABLE knowledgebase (
    id SERIAL PRIMARY KEY,
    meeting_id INT REFERENCES meetings(id),
    context TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- SYSTEM SETTINGS & GUIDELINES
CREATE TABLE system_guidelines (
    id SERIAL PRIMARY KEY,
    content TEXT NOT NULL,
    updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

-----

### `frontend-architecture.md`

This document contains the detailed technical blueprint for our Next.js web application. It provides developers with specific patterns, templates, and structures for building the user interface.

# Frontend Architecture

## Component Architecture

To ensure our Next.js application is organized and maintainable, we'll follow a feature-based structure.

### Component Organization

This structure separates reusable UI elements from feature-specific components.

```
/apps/web/src/
|-- /app/                 # Next.js App Router: Layouts and Pages
|-- /components/          # Reusable, global UI components (e.g., Button, Input, Modal)
|   |-- /ui/
|   |-- /icons/
|-- /features/            # Feature-specific components and logic
|   |-- /authentication/
|   |-- /meeting-recorder/
|   |-- /results-dashboard/
|-- /hooks/               # Custom React hooks
|-- /lib/                 # Utility functions, helpers
|-- /services/            # API client and data fetching logic
|-- /stores/              # Global state management (Zustand stores)
```

### Component Template

All new React components should follow this basic structure for consistency.

```typescript
import React from 'react';

// Define the component's props with TypeScript
interface MyComponentProps {
  title: string;
  onAction?: () => void;
}

const MyComponent: React.FC<MyComponentProps> = ({ title, onAction }) => {
  return (
    <div>
      <h2>{title}</h2>
      {onAction && <button onClick={onAction}>Click Me</button>}
    </div>
  );
};

export default MyComponent;
```

## State Management Architecture

We will use Zustand for its simplicity and performance. Our state will be organized into separate "stores" or "slices," each responsible for a specific domain of the application's data.

### Store Structure

All state management files will reside in the `/stores` directory.

```
/apps/web/src/
|-- /stores/
|   |-- useAuthStore.ts       # Manages user session, profile data
|   |-- useMeetingStore.ts    # Manages state for an active meeting
|   |-- index.ts              # Exports all stores
```

### State Management Patterns & Template

  * **Separate Stores**: Each store will manage a distinct piece of state.
  * **Selectors**: Components will use selectors to subscribe to only the specific pieces of state they need, preventing unnecessary re-renders.
  * **Async Actions**: Asynchronous operations like API calls will be handled directly within the store's actions.

Here is a template for a basic authentication store:

```typescript
import { create } from 'zustand';
import { User } from '@/lib/types'; // Assuming types are defined elsewhere

interface AuthState {
  user: User | null;
  token: string | null;
  isLoading: boolean;
  login: (credentials: Credentials) => Promise<void>;
  logout: () => void;
  setUser: (user: User | null) => void;
}

export const useAuthStore = create<AuthState>((set) => ({
  user: null,
  token: null,
  isLoading: false,
  setUser: (user) => set({ user }),
  login: async (credentials) => {
    set({ isLoading: true });
    try {
      // const { user, token } = await authService.login(credentials);
      // set({ user, token, isLoading: false });
    } catch (error) {
      set({ user: null, token: null, isLoading: false });
    }
  },
  logout: () => {
    set({ user: null, token: null });
  },
}));
```

## Routing Architecture

For Next.js, routing is handled by its file-system-based App Router. We'll define the folder structure for our pages and a standard pattern for protecting routes.

### Route Organization

Our routes will be organized within the `/app` directory. We will use route groups `(auth)` and `(app)` to apply different layouts and middleware.

```
/apps/web/src/
|-- /app/
|   |-- /(auth)/              # Group for public, authentication-related pages
|   |   |-- /login/
|   |   |   |-- page.tsx
|   |   |-- /register/
|   |       |-- page.tsx
|   |-- /(app)/               # Group for private, authenticated pages
|   |   |-- /dashboard/
|   |   |   |-- page.tsx
|   |   |-- /profile/
|   |   |   |-- page.tsx
|   |   |-- /meetings/
|   |   |   |-- /[id]/        # Dynamic route for a single meeting
|   |   |       |-- page.tsx
|   |   |-- layout.tsx        # Shared layout for all authenticated pages
|   |-- layout.tsx            # Root layout
|   |-- page.tsx              # Homepage
```

### Protected Route Pattern

We will use a single `middleware.ts` file to protect all routes within the `(app)` group.

```typescript
// /middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const sessionToken = request.cookies.get('session-token')?.value;

  if (!sessionToken && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }
  
  if (sessionToken && request.nextUrl.pathname.startsWith('/login')) {
     return NextResponse.redirect(new URL('/dashboard', request.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/dashboard/:path*', '/login'],
};
```

## Frontend Services Layer

We will use the `axios` library to create a centralized API client. This centralizes all our data-fetching logic, making our components cleaner.

### API Client Setup

```typescript
// /lib/apiClient.ts
import axios from 'axios';

const apiClient = axios.create({
  baseURL: '/api/v1', // The base URL for all our API calls
  headers: {
    'Content-Type': 'application/json',
  },
  withCredentials: true, // Important for sending httpOnly cookies
});

export default apiClient;
```

### Service Example

```typescript
// /services/authService.ts
import apiClient from '@/lib/apiClient';
import { User, Credentials, TokenResponse } from '@/lib/types';

export const authService = {
  async register(data: Omit<User, 'id'>): Promise<User> {
    const response = await apiClient.post<User>('/users', data);
    return response.data;
  },

  async login(credentials: Credentials): Promise<TokenResponse> {
    const response = await apiClient.post<TokenResponse>('/auth/tokens', credentials);
    return response.data;
  },
};
```

-----

### `backend-architecture.md`

This document contains the detailed technical blueprint for our backend services. It provides developers with specific patterns for API routes, database interaction, and security.

# Backend Architecture

## Service Architecture

Our backend consists of two main parts: the core API built with Next.js API Routes and a separate, specialized AI service built with Python.

### Next.js API Route Architecture

All core business logic (authentication, meeting management, database interactions) will be handled within the Next.js application itself.

#### API Route Organization

```
/apps/web/src/
|-- /app/
|   |-- /api/
|   |   |-- /auth/
|   |   |   |-- /tokens/       # POST for login
|   |   |-- /users/          # POST for register, GET/PUT for profile
|   |   |   |-- route.ts
|   |   |-- /meetings/
|   |       |-- /route.ts      # POST to create a new meeting
|   |       |-- /[{id}]/      # GET to retrieve meeting results
|   |           |-- route.ts
```

#### API Route Handler Template

```typescript
// e.g., /app/api/users/route.ts
import { NextResponse } from 'next/server';
import { z } from 'zod'; // For validation

const userRegistrationSchema = z.object({
  fullName: z.string().min(2),
  email: z.string().email(),
  password: z.string().min(8),
});

export async function POST(request: Request) {
  try {
    const json = await request.json();
    const validatedData = userRegistrationSchema.parse(json);

    // Call a service function to handle the business logic
    // const newUser = await userService.register(validatedData);

    return NextResponse.json({ success: true, userId: newUser.id }, { status: 201 });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json({ error: 'Invalid input', details: error.errors }, { status: 400 });
    }
    // Handle other errors
    return NextResponse.json({ error: 'An internal error occurred' }, { status: 500 });
  }
}
```

### AI Service Architecture

The AI service is a standalone Python application responsible only for analysis.

  * **Framework**: A lightweight Python web framework like FastAPI or Flask.
  * **Communication**: It will expose a single, internal REST endpoint (e.g., `/analyze`).
  * **Stateless**: It will not connect to the database. It receives all necessary context from the Meeting Service and returns a JSON object with the analysis results.

## Database Architecture

### Schema Design

The database schema is complete and has been adopted as per your detailed SQL specification.

### Data Access Layer

We will use the Repository Pattern to isolate business logic from direct database queries.

#### Repository Pattern Template

```typescript
// /apps/web/src/services/users/user.repository.ts
import { db } from '@/lib/db'; // Assume a centralized db client
import { User } from '@/lib/types';

export const userRepository = {
  async findById(id: number): Promise<User | null> {
    // Logic to query the 'users' table by ID
    const result = await db.query('SELECT * FROM users WHERE id = $1', [id]);
    return result.rows[0] || null;
  },

  async findByEmail(email: string): Promise<User | null> {
    const result = await db.query('SELECT * FROM users WHERE email = $1', [email]);
    return result.rows[0] || null;
  },

  async create(userData: Omit<User, 'id' | 'created_at'>): Promise<User> {
    // Logic to insert a new user into the 'users' table
    const result = await db.query(
      'INSERT INTO users (full_name, email, ...) VALUES ($1, $2, ...) RETURNING *',
      [userData.fullName, userData.email, ...]
    );
    return result.rows[0];
  },
};
```

## Authentication and Authorization Architecture

We will use a JWT-based authentication strategy, with the token stored in a secure, httpOnly cookie.

### Authentication Flow (Login)

```mermaid
sequenceDiagram
    participant User
    participant Browser (Next.js)
    participant API (Next.js)
    participant Database

    User->>Browser (Next.js): Submits login form (email, password)
    Browser (Next.js)->>API (Next.js): POST /api/v1/auth/tokens with credentials
    
    activate API (Next.js)
    API (Next.js)->>Database: Find user by email
    Database-->>API (Next.js): Returns user record with hashed password
    API (Next.js)->>API (Next.js): Verify submitted password against hash
    API (Next.js)->>API (Next.js): Generate JWT
    
    API (Next.js)-->>Browser (Next.js): Respond 200 OK with `Set-Cookie` header (httpOnly)
    deactivate API (Next.js)
    
    Browser (Next.js)->>User: Redirect to /dashboard
```

### Authorization Pattern (API Route Protection)

We will use a higher-order function ("wrapper") to protect API endpoints.

```typescript
// /lib/auth/withAuth.ts
import { NextResponse } from 'next/server';
import { verifyToken } from './jwt'; // Assume a utility for JWT verification

type ApiHandler = (request: Request, params: { userId: string }) => Promise<NextResponse>;

export function withAuth(handler: ApiHandler) {
  return async (request: Request) => {
    const token = request.cookies.get('session-token')?.value;

    if (!token) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    try {
      const payload = await verifyToken(token);
      return handler(request, { userId: payload.sub });
    } catch (error) {
      return NextResponse.json({ error: 'Invalid token' }, { status: 401 });
    }
  };
}
```

-----

### `project-structure.md`

This file provides the definitive folder structure for our monorepo. It's a crucial guide for developers to ensure that all code is organized consistently, which is especially important when working with multiple applications like our web app and AI service.

# Unified Project Structure

This structure is designed to support our Next.js full-stack application and the separate Python AI service within a single monorepo.

```
/meeting-assistant-demo/
|-- /apps/                      # Contains our deployable applications
|   |-- /web/                   # The Next.js full-stack application
|   |   |-- /src/
|   |   |   |-- /app/           # Next.js App Router (UI Pages & API Routes)
|   |   |   |   |-- /api/       # Backend API routes
|   |   |   |   |-- /(app)/     # Protected UI pages (Dashboard, etc.)
|   |   |   |   |-- /(auth)/    # Public UI pages (Login, etc.)
|   |   |-- /components/      # Reusable React components
|   |   |-- /features/        # Components specific to a feature
|   |   |-- /lib/             # Core utilities (apiClient, db client, etc.)
|   |   |-- /services/        # Business logic for API routes
|   |   |-- /stores/          # Zustand state stores
|   |-- /ai-service/            # The Python AI analysis service
|       |-- /app/               # Main application logic
|       |-- /core/              # Core analysis, LangGraph chains
|       |-- main.py             # FastAPI/Flask application entry point
|       |-- Dockerfile          # Container definition
|
|-- /packages/                  # Shared code between applications
|   |-- /ui/                    # Shared UI components (e.g., from shadcn/ui)
|   |-- /eslint-config-custom/  # Shared ESLint configuration
|   |-- /tsconfig/              # Shared TypeScript configuration
|
|-- /docs/                      # Project documentation (PRD, Architecture)
|-- package.json                # Root package.json for monorepo scripts
|-- turbo.json                  # Turborepo configuration (or similar)
```

-----

### `development-workflow.md`

This file is a practical guide for the development team. It provides the exact commands and configuration needed to get the project running on a local machine, which is key to a smooth start for development.

# Development Workflow

This workflow assumes a developer has the monorepo cloned to their local machine.

## Local Development Setup

### Prerequisites

Before starting, a developer must have the following installed:

  * Node.js (v20.11.0+)
  * pnpm (v9.0+)
  * Python (v3.11+)
  * Docker (latest)

### Initial Setup

To install all dependencies for all applications in the monorepo, run the following command from the project root:

```bash
pnpm install
```

### Development Commands

The monorepo is configured with scripts to run services individually or all at once.

```bash
# Start all services (web frontend and AI service) concurrently
pnpm run dev

# Start only the Next.js web application
pnpm --filter web dev

# Start only the Python AI service
pnpm --filter ai-service dev
```

\<hr\>

## Environment Configuration

The project will require a `.env` file at the root of the monorepo. Developers can copy the `.env.example` file to get started.

```bash
# .env.example

# PostgreSQL Database Connection URL
DATABASE_URL="postgresql://user:password@localhost:5432/meeting_db"

# JWT Secret for signing authentication tokens
JWT_SECRET="your-super-secret-key-for-development"

# URL for the Next.js app to find the AI service
AI_SERVICE_URL="http://localhost:8000"

# --- AI Service Specific --- #

# API Key for the LLM service (e.g., OpenAI, Anthropic)
LLM_API_KEY="your-llm-api-key"
```

\<hr\>

## Development Best Practices

  * **Environment Consistency**: To avoid the "it works on my machine" problem, the project will include a `.nvmrc` file to enforce the Node.js version and a `devcontainer` configuration to ensure a completely consistent development environment for everyone.
  * **Service Dependencies**: The `web` application depends on the `ai-service`. Both services should be running for full functionality. The web app will include a health check to provide a clear warning if it cannot connect to the AI service.
  * **Secret Management**: The `.env` file must be added to `.gitignore` and never committed to the repository. The CI/CD pipeline will validate that our `.env.example` file is kept up-to-date.

# Deployment Architecture

Our strategy is based on containerizing each service with Docker. This ensures that our application runs the same way on the production server as it does locally.

## Deployment Strategy

  * **Containerization**: Each service (`web` and `ai-service`) will be packaged into its own Docker image. A `docker-compose.yml` file will define how these services and our PostgreSQL database run together.
  * **Deployment**: The process will involve building new Docker images, pushing them to a container registry, and then orchestrating a zero-downtime update on the production server.

\<hr\>

## CI/CD Pipeline

We will use GitHub Actions to automate our testing and deployment process.

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [ main ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout Code
      uses: actions/checkout@v4

    - name: Set up Environment
      # ... setup Node.js, Python, pnpm, Docker ...

    - name: Install, Lint, and Test
      run: |
        pnpm install
        pnpm run test

    - name: Build and Push Docker Images
      # ... build and push 'web' and 'ai-service' images ...

    - name: Deploy to Server
      # ... SSH into the server, run database migrations, and perform a blue-green deployment ...
```

\<hr\>

## Environments

| Environment | Frontend URL                  | Backend URL                     | Purpose                     |
| ----------- | ----------------------------- | ------------------------------- | --------------------------- |
| Development | `http://localhost:3000`       | `http://localhost:8000`       | Local developer machines    |
| Staging     | `https://staging.example.com` | `https://api-staging.example.com` | Pre-production testing, final QA |
| Production  | `https://example.com`         | `https://api.example.com`       | Live environment for users  |

\<hr\>

## Deployment Best Practices

  * **Zero Downtime**: Deployments must not cause user-facing downtime. A blue-green deployment strategy will be used where the new version is started and confirmed healthy before traffic is switched over.
  * **Database Migrations**: All database migrations must be backward-compatible. The migration script will be run before the new application code is deployed to prevent crashes.
  * **Resource Monitoring**: The production server's CPU, memory, and disk usage will be actively monitored with alerts to prevent resource exhaustion from impacting application performance.

-----

### `security-and-performance.md`

This file details the non-negotiable requirements for ensuring our application is secure and fast. These are critical quality gates that all new code must pass before release.

# Security and Performance

## Security Requirements

This section outlines the mandatory security practices for the entire application.

### Frontend Security:

  * **Content Security Policy (CSP)**: A strict CSP header will be implemented to prevent Cross-Site Scripting (XSS) and other injection attacks.
  * **Secure Storage**: All session tokens will be stored in secure, `httpOnly` cookies to make them inaccessible to client-side JavaScript.

### Backend Security:

  * **Input Validation**: All incoming data on all API routes must be strictly validated against a schema before processing.
  * **Rate Limiting**: Rate limiting will be enforced at the API Gateway to protect against brute-force attacks.
  * **CORS Policy**: A restrictive Cross-Origin Resource Sharing (CORS) policy will be configured to only allow requests from our designated frontend domain.

### Authentication Security:

  * **Password Hashing**: All user passwords must be securely hashed using a strong algorithm like `bcrypt`.
  * **Token Management**: We will use short-lived JWTs for sessions.

### Operational Security:

  * **Continuous Dependency Scanning**: A security scanner (like GitHub's Dependabot) must be integrated into our CI/CD pipeline to automatically check for vulnerabilities in our open-source dependencies.
  * **Production Secrets Management**: Production secrets (database URLs, API keys) must be injected as environment variables through a secure mechanism, not stored in plaintext files on the server.

\<hr\>

## Performance Optimization

### Frontend Performance:

  * **Bundle Size**: The initial JavaScript bundle for the main page should not exceed 250kB.
  * **Loading Strategy**: We will leverage Next.js features like code-splitting per page and dynamic imports.
  * **Image Optimization**: All images will be served in modern formats (like WebP) and correctly sized using the Next.js `<Image>` component.
  * **Target Score**: All pages must achieve a Google Lighthouse performance score of 90+.

### Backend Performance:

  * **Database Optimization**: We must avoid "N+1 query" problems by using efficient JOINs. All foreign keys and frequently queried columns will be indexed.
  * **Caching Strategy**: The Meeting Service must implement a Redis caching layer for the final, read-only analysis results to reduce database load for frequently accessed meetings.
  * **Observability**: An observability platform (using a standard like OpenTelemetry) will be implemented to provide distributed tracing. This will allow us to trace a single request as it flows from the Next.js frontend to the Python AI service and back.