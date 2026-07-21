# RememberAll

A collaborative todo list application with real-time sharing, built with a .NET 8 backend and a SvelteKit frontend. Share lists with others, manage tasks together, and stay organized — all wrapped in a mobile-first, dark-themed PWA.

## Features

- **User Authentication** — Secure registration and login with HttpOnly cookie-based sessions
- **Todo List Management** — Create, edit, and organize multiple lists
- **Collaborative Sharing** — Share lists with other users via an invite system with granular access control
- **Todo Items** — Add, complete, and track items with completion counters
- **PWA Support** — Installable as a standalone app with offline-capable service worker
- **Multi-language** — English and Danish localization built-in
- **Mobile-First Design** — Dark theme, minimal UI, optimized for touch interaction
- **Dockerized Deployment** — Run the entire stack with a single command

## Architecture

```
Browser ──► SvelteKit (proxy) ──► .NET 8 API ──► SQLite
```

The frontend uses a **secure proxy pattern**: all API requests go through the SvelteKit server, which forwards them to the .NET backend. This keeps the backend URL hidden from the browser, simplifies CORS handling, and enables HttpOnly cookies to work securely.

> **Note:** The backend is never directly exposed to the browser. All communication flows through the frontend proxy at `/api`.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| **Backend** | .NET 8, ASP.NET Core, Entity Framework Core, SQLite |
| **Backend Auth** | Cookie-based authentication, ASP.NET Core Identity password hashing |
| **Backend Docs** | Swagger / OpenAPI (Swashbuckle) |
| **Frontend** | SvelteKit 2, Svelte 5, TypeScript |
| **Styling** | Tailwind CSS 4 |
| **PWA** | Service Worker, Web App Manifest |
| **Infrastructure** | Docker, Docker Compose |

## Getting Started

### Prerequisites

- [Docker](https://docs.docker.com/get-docker/) and [Docker Compose](https://docs.docker.com/compose/install/)

### Quick Start

```bash
# Clone the repository
git clone <repository-url>
cd rememberall

# Start all services
docker compose up -d --build
```

The application will be available at:

- **Frontend:** http://localhost:3000
- **Backend API:** http://localhost:5000 (internal, not exposed to browser)
- **Swagger UI:** http://localhost:5000 (development only)

### Running Without Docker

**Backend:**

```bash
cd RememberAllBackend
dotnet run
```

The API starts on `http://localhost:5000` with Swagger UI at the root.

**Frontend:**

```bash
cd rememberall-frontend
pnpm install
pnpm dev
```

The frontend dev server starts on `http://localhost:5173` and proxies API requests to `http://localhost:5000/api`.

## Project Structure

```
rememberall/
├── RememberAllBackend/          # .NET 8 Web API
│   ├── src/
│   │   ├── Controllers/         # API endpoints
│   │   ├── Services/            # Business logic
│   │   ├── Repositories/        # Data access layer
│   │   ├── Entities/            # Domain models
│   │   ├── DTOs/                # Data transfer objects
│   │   ├── Middleware/          # Global exception handling
│   │   ├── Extensions/          # Validation & mapping
│   │   ├── Utilities/           # Email & password validation
│   │   └── Data/                # EF Core DbContext & migrations
│   ├── Program.cs
│   └── Dockerfile
├── RememberAllBackend.Tests/    # xUnit test suite
│   ├── Unit/                    # Unit tests (services, validators)
│   └── Integration/             # Integration tests (API endpoints)
├── rememberall-frontend/        # SvelteKit PWA
│   ├── src/
│   │   ├── lib/
│   │   │   ├── api/             # API client, types, services
│   │   │   ├── stores/          # Svelte 5 rune-based state
│   │   │   ├── components/      # Reusable UI components
│   │   │   ├── i18n/            # Internationalization
│   │   │   └── utils/           # Error handling, helpers
│   │   └── routes/              # SvelteKit pages
│   ├── static/                  # PWA icons, manifest, service worker
│   └── Dockerfile
├── scripts/
│   ├── deploy.sh                # Deploy to remote server
│   ├── download-db.sh           # Download remote database
│   └── coverage.sh              # Run tests with coverage
├── docker-compose.yml           # Production stack
└── data/                        # SQLite database (bind mount)
```

## API Endpoints

All endpoints are accessed through the frontend proxy at `/api/*`.

### Authentication

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/auth/register` | Create a new account |
| `POST` | `/api/auth/login` | Log in and receive session cookie |
| `POST` | `/api/auth/logout` | Log out (requires auth) |
| `GET` | `/api/auth/me` | Get current user (requires auth) |
| `GET` | `/api/auth/password-requirements` | Get password validation rules |
| `DELETE` | `/api/auth/delete-account` | Delete user account (requires auth) |

### Todo Lists

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/lists` | Create a new list |
| `GET` | `/api/lists` | Get all user's lists |
| `GET` | `/api/lists/{id}` | Get a single list by ID |
| `PATCH` | `/api/lists` | Update a list |
| `DELETE` | `/api/lists/{id}` | Delete a list |

### Todo Items

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/todoitems` | Create a new item |
| `GET` | `/api/todoitems/bylist/{listId}` | Get all items for a list |
| `PATCH` | `/api/todoitems` | Update an item |
| `PATCH` | `/api/todoitems/{id}/complete` | Mark item as complete |
| `PATCH` | `/api/todoitems/{id}/incomplete` | Mark item as incomplete |
| `DELETE` | `/api/todoitems/{id}` | Delete an item |

### Invitations

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/invites` | Send an invite to share a list |
| `GET` | `/api/invites/sent` | Get sent invites |
| `GET` | `/api/invites/received` | Get received invites |
| `PATCH` | `/api/invites/{id}/accept` | Accept an invite |
| `DELETE` | `/api/invites/{id}` | Decline or delete an invite |

### List Access

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/listaccess` | Get access permissions (optional `?listId=` filter) |
| `DELETE` | `/api/listaccess/{id}` | Remove a user's access to a list |

## Scripts

| Script | Description |
|--------|-------------|
| `scripts/deploy.sh` | Deploy the application to a remote server via SSH |
| `scripts/download-db.sh` | Download the production SQLite database from the remote server |
| `scripts/coverage.sh` | Run tests with code coverage and generate an HTML report |
| `scripts/test-class.sh` | Run tests for a specific test class |

## Testing

The project includes a comprehensive test suite using **xUnit**:

```bash
# Run all tests
dotnet test

# Run tests with code coverage report
./scripts/coverage.sh
```

The coverage report is generated with [ReportGenerator](https://github.com/danielpalme/ReportGenerator) and written to the `CoverageReport/` directory.

- **Unit tests** — Services (AuthService, TodoListService, TodoItemService, InviteService, ListAccessService), validators, and entity mapping
- **Integration tests** — Full API endpoint testing with a test database fixture

## Development

### Backend Conventions

- Repository pattern with dependency injection for data access
- Global exception middleware for standardized error responses
- Custom exception types (`NotFoundException`, `ForbiddenException`, `BusinessLogicException`, etc.)
- PATCH for partial updates, validation via DTO extension methods

### Frontend Conventions

- Svelte 5 runes (`$state`, `$derived`, `$effect`, `$props`) for reactivity
- Store files use `.svelte.ts` extension for rune support
- Service layer uses plain async functions with native `fetch`
- API client goes through the SvelteKit proxy at `/api`
- Tailwind CSS with CSS custom properties for colors

### Adding a New Language

1. Create a new JSON file in `rememberall-frontend/src/lib/messages/` (e.g., `de.json`)
2. Mirror the structure from existing language files
3. Update the i18n configuration in `src/lib/i18n/index.ts`

### Environment Variables

**Frontend:**

| Variable | Default | Description |
|----------|---------|-------------|
| `BACKEND_URL` | `http://localhost:5000/api` | Backend API URL (internal) |

**Backend:**

| Variable | Default | Description |
|----------|---------|-------------|
| `ASPNETCORE_URLS` | `http://+:5000` | Server binding URL |
| `ASPNETCORE_ENVIRONMENT` | `Production` | Environment mode |
| `DATABASE_PATH` | `/app/data` | SQLite database directory |
| `DataProtection__KeysPath` | `/app/keys` | Data Protection key directory |
