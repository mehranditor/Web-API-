# Web API

ASP.NET Core 8 Web API featuring JWT auth with refresh tokens, ASP.NET Identity (roles), SQLite, Redis cache, Swagger, SignalR, CQRS/MediatR, and custom middleware.

## Note : This is a project i made during my internship at yildiz technical university

## Features
- JWT auth + refresh tokens: `/api/auth/*`
- Identity roles seeded (`Admin`, `User`) on startup
- User endpoints via CQRS/MediatR: `/api/user/*`
- Admin token by shared secret: `/api/admin/auth`
- RAG endpoints (index/search/summarize): `/api/rag/*`
- Redis-backed caching and simple per-IP rate limiting for `/api/user*`
- Global exception handling and auth header enforcement
- Swagger UI at `/swagger`
- SignalR hub: `/hubs/notification`

## Tech
- .NET 8 (`TargetFramework: net8.0`)
- EF Core (SQLite) + ASP.NET Identity
- MediatR (CQRS)
- Serilog (console + rolling file)
- StackExchange.Redis cache
- Swashbuckle + example filters
- SignalR

## Getting Started
### Prerequisites
- .NET SDK 8.x
- (Optional) Redis at `localhost:6379` for cache/rate limit

### Configure
Edit `appsettings.json`:
- `Jwt:Key`, `Jwt:Issuer`, `Jwt:Audience`
- `ConnectionStrings:DefaultConnection` (default `authdb.sqlite`)
- `Redis:ConnectionString` (e.g., `localhost:6379`)
- `AdminAuth:SecretKey`

Use environment variables or User Secrets for sensitive values in real deployments.

### Run
```bash
# from project folder
 dotnet restore
 dotnet build
 dotnet run
```
- Binds to `http://0.0.0.0:<PORT>` where `PORT` env var defaults to `5025`.
- Roles `Admin` and `User` are ensured on startup.
- Swagger: `http://localhost:5025/swagger`

### Database
- SQLite file `authdb.sqlite`. Created/ensured at startup.

## Auth Flow
- Register: `POST /api/auth/register` → `{ userName, password, email }`
- Login: `POST /api/auth/login` → `{ accessToken, refreshToken }`
- Refresh: `POST /api/auth/refresh` → new tokens with `{ refreshToken }`
- Use header: `Authorization: Bearer <accessToken>`

## Endpoints (high level)
- `POST /api/admin/auth` — exchange `AdminAuth:SecretKey` for Admin JWT
- `/api/user`
  - `GET /` — list users
  - `GET /{id}` — get user
  - `POST /` — create user (anonymous allowed)
  - `PUT /{id}` — update (Admin)
  - `DELETE /{id}` — delete (Admin)
  - `GET /paged` — pagination/filter/sort
  - `GET /test-redis` — Redis connectivity test (anonymous)
- `/api/rag`
  - `POST /index` — index `{ content, metadata? }`
  - `POST /search` — semantic search
  - `DELETE /document/{documentId}` — delete document
  - `DELETE /clear` — clear index (Admin)
  - `GET /health` — health (anonymous)
  - `POST /summarize-logs` — summarize log content
- SignalR hub: `/hubs/notification`

## Middleware
- `ExceptionMiddleware` — global error handling (JSON 500)
- `TokenValidationMiddleware` — requires `Authorization` on protected routes
- `RateLimitingMiddleware` — per-IP limit for `/api/user*` using Redis

## Logging
- Serilog: console + `logs/app-<PORT>.log` (daily rolling)

## Environment variables
- `PORT` — port binding (default `5025`)
- Any `appsettings.json` key can be overridden (e.g., `Jwt__Key`)

## Git hygiene
Recommended `.gitignore` entries: `bin/`, `obj/`, `.vs/`, `logs/`, `nohup.out`, `*.sqlite`, `*.user`.
Avoid committing real secrets in `appsettings*.json`.

## License
MIT (or your choice)
