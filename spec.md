# Technical Specification: Task Management & Kanban Board Application

## 1. Project Overview
* **Application Name:** TaskFlow / KanbanPro
* **Objective:** Build a comprehensive task management web application leveraging an Agile/Scannable Kanban model. The platform supports workspace collaboration, project segmentation, custom status columns, and seamless CRUD operations.
* **Target Audience:** Individuals, software development teams, product managers, and marketing teams needing real-time progress tracking.

---

## 2. Core Features

### 2.1. Authentication & Authorization
* **Sign Up:** Email and password registration with complexity validation (minimum 8 characters, upper/lower case, numbers, and special characters). Optional email verification workflow.
* **Sign In:** Authentication via Email/Password or OAuth2 providers (Google, GitHub). Utilizes JSON Web Tokens (JWT) with short-lived Access Tokens (15 minutes) and long-lived Refresh Tokens (7 days).
* **Route Protection:** Client-side (React Router) and server-side (Middleware) route guards to secure protected pages (Dashboard, Project Boards).

### 2.2. Project Management
* **Project List:** Displays projects owned by or shared with the user in Grid or List view.
* **Project CRUD:**
  * **Create:** Input project name, description, and assign team members.
  * **Update:** Edit project metadata and default configurations.
  * **Delete:** Soft/hard deletion with confirmation dialogs.

### 2.3. Column Management
* **Default Structure:** Newly created projects automatically initialize with three standard columns: *To Do*, *In Progress*, and *Done*.
* **Column CRUD:**
  * **Create:** Add custom columns with specific titles and board positions.
  * **Update:** Inline editing for column titles and drag-and-drop column reordering.
  * **Delete:** Options to purge a column alongside its contained tasks or migrate tasks to a default fallback column prior to deletion.

### 2.4. Task Management
* **Task CRUD:**
  * **Create:** Title, description (Markdown supported), priority levels (*Low*, *Medium*, *High*, *Urgent*), due date, labels/tags, and assignee allocation.
  * **Update:** Comprehensive editing modal/drawer interface.
  * **Delete:** Archival or permanent removal.
* **Kanban Board & Drag-and-Drop:**
  * Drag-and-drop tasks between columns (updating `column_id` or status).
  * Reorder task priorities within the same column (`position_index`).
* **Search & Filter:** Real-time filtering by title keywords, priority levels, assignees, or tags.

---

## 3. System Architecture & Tech Stack

### 3.1. Frontend (Client-side)
* **Framework:** React.js / Next.js (App Router) for SSR/SSG and optimized performance.
* **State Management & Data Fetching:** TanStack Query (React Query) paired with Zustand for global UI state management.
* **Drag-and-Drop Engine:** `@hello-pangea/dnd` or `@dnd-kit/core` for responsive multi-column board interactions.
* **Styling & UI Components:** Tailwind CSS with Shadcn UI / Radix UI for accessible components.

### 3.2. Backend (Server-side)
* **Framework:** Node.js with Express.js or NestJS (TypeScript).
* **Database & ORM:** PostgreSQL managed via Prisma ORM or TypeORM for type-safe database access and migrations.
* **Real-time Engine (Optional):** Socket.io for multi-user real-time synchronization on active boards.

---

## 4. Database Schema

```sql
-- Users Table
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  full_name VARCHAR(100) NOT NULL,
  avatar_url TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Projects Table
CREATE TABLE projects (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(150) NOT NULL,
  description TEXT,
  owner_id UUID REFERENCES users(id) ON DELETE CASCADE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Columns Table
CREATE TABLE columns (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id UUID REFERENCES projects(id) ON DELETE CASCADE,
  name VARCHAR(50) NOT NULL,
  position INT NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Tasks Table
CREATE TABLE tasks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id UUID REFERENCES projects(id) ON DELETE CASCADE,
  column_id UUID REFERENCES columns(id) ON DELETE CASCADE,
  title VARCHAR(255) NOT NULL,
  description TEXT,
  priority VARCHAR(20) CHECK (priority IN ('low', 'medium', 'high', 'urgent')),
  due_date TIMESTAMP,
  position INT NOT NULL,
  assignee_id UUID REFERENCES users(id) ON DELETE SET NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 5. API Endpoints (RESTful)

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| **POST** | `/api/auth/register` | Register a new user account |
| **POST** | `/api/auth/login` | Authenticate and return JWT Access/Refresh tokens |
| **GET** | `/api/projects` | Retrieve all projects for the authenticated user |
| **POST** | `/api/projects` | Create a new project |
| **PUT/DELETE** | `/api/projects/:id` | Update or delete a specific project |
| **GET** | `/api/projects/:id/board` | Fetch full column and task tree for a Kanban board |
| **POST** | `/api/columns` | Create a new column in a project |
| **PATCH** | `/api/columns/:id` | Update column title or display order |
| **POST** | `/api/tasks` | Create a new task |
| **PATCH** | `/api/tasks/:id` | Update task details or move position/column |
| **DELETE** | `/api/tasks/:id` | Delete a task |