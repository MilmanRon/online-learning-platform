# Online Learning Platform

A full-stack web application where instructors can create courses with video lessons, and students can enroll, watch lessons, and track their progress.

## Tech Stack

**Backend**
- ASP.NET Core 9 Web API
- Entity Framework Core 9 with SQL Server
- JWT Bearer authentication
- FluentValidation
- AutoMapper
- Serilog

**Frontend**
- Angular 19 (standalone components, lazy loading)
- Bootstrap 5
- Signal-based state management

## Architecture

**API** follows a clean architecture pattern:
```
API/
├── Controllers/         # HTTP endpoints
├── Application/         # Services, DTOs, validators, AutoMapper profiles
├── Domain/              # Entities, repository interfaces
└── Infrastructure/      # EF Core DbContext, repository implementations, JWT token service
```

**Client** mirrors clean architecture on the frontend:
```
Client/src/app/
├── core/                # Domain models, repository interfaces
├── data/                # API services, DTOs, mappers, repositories, signal stores
├── features/            # Feature modules (auth, courses, lessons, users, dashboard)
├── infrastructure/      # Guards, interceptors, resolvers
└── layouts/             # Auth layout, main layout
```

## Domain Model

| Entity | Description |
|---|---|
| `User` | Has a `Role` of either `Instructor` or `Student` |
| `Course` | Created by instructors; has a title, description, and a list of lessons |
| `Lesson` | Belongs to a course; contains a title and a YouTube video URL |
| `Enrollment` | Links a student to a course |
| `Progress` | Records when a student has watched a specific lesson |

## Features

- **Authentication** — register and login with role selection (Instructor / Student); JWT token persisted in localStorage
- **Courses** — students see all available courses and their enrollment status; instructors can create and edit courses
- **Lessons** — instructors can add and edit lessons (YouTube video URLs); students watch lessons from the course detail view
- **Progress tracking** — watched lessons are recorded per user; progress is visible on the course detail page
- **User profile** — users can edit their own profile information
- **Role-based access** — route guards restrict instructor-only actions (create/edit courses and lessons) from students

## Prerequisites

- [.NET 9 SDK](https://dotnet.microsoft.com/download)
- [Node.js 20+](https://nodejs.org/)
- [Angular CLI 19](https://angular.dev/tools/cli)
- SQL Server (or Docker — see below)

## Getting Started

### 1. Database

Run SQL Server locally or via Docker:

```bash
docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=YourPassword123!" \
  -p 1433:1433 --name sqlserver -d mcr.microsoft.com/mssql/server:2022-latest
```

### 2. API

Update `API/appsettings.json` with your connection string and a JWT secret key:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost,1433;Database=LearningPlatformDB;User Id=sa;Password=YourPassword123!;TrustServerCertificate=True;"
  },
  "TokenKey": "your-super-secret-key-at-least-64-characters-long"
}
```

Apply migrations and start the API:

```bash
cd API
dotnet ef database update
dotnet run
```

The API runs on `http://localhost:5000`.

### 3. Client

```bash
cd Client
npm install
ng serve
```

The client runs on `http://localhost:4200`.

## API Endpoints

| Method | Route | Auth | Description |
|---|---|---|---|
| POST | `/api/auth/register` | Public | Register a new user |
| POST | `/api/auth/login` | Public | Login and receive a JWT |
| GET | `/api/courses` | Required | List all courses |
| POST | `/api/courses` | Instructor | Create a course |
| PUT | `/api/courses/{id}` | Instructor | Update a course |
| DELETE | `/api/courses/{id}` | Instructor | Delete a course |
| GET | `/api/lessons/{courseId}` | Required | Get lessons for a course |
| POST | `/api/lessons` | Instructor | Add a lesson |
| PUT | `/api/lessons/{id}` | Instructor | Update a lesson |
| DELETE | `/api/lessons/{id}` | Instructor | Delete a lesson |
| GET | `/api/enrollments` | Required | Get current user's enrollments |
| POST | `/api/enrollments` | Student | Enroll in a course |
| DELETE | `/api/enrollments/{id}` | Student | Unenroll from a course |
| GET | `/api/progress` | Required | Get current user's progress |
| POST | `/api/progress` | Student | Mark a lesson as watched |
| GET | `/api/users/{id}` | Required | Get a user profile |
| PUT | `/api/users/{id}` | Required | Update a user profile |
