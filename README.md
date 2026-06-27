# Advanced Chat

A real-time messaging platform built with ASP.NET Core 8 and SignalR. Users create chat rooms, invite members by email, and exchange messages instantly through a clean, responsive web interface.

## Features

- **Identity-based auth** — Register and sign in via ASP.NET Core Identity with cookie authentication
- **Chat rooms** — Create, browse, and delete rooms; each room tracks ownership and membership
- **Member management** — Invite users by email; membership is required to view or send messages in a room
- **Real-time messaging** — Messages delivered instantly over WebSockets via SignalR, with automatic reconnection
- **Private messages** — Direct messaging between any two registered users
- **Dashboard** — Unified console showing room list, online status, a live message feed, and quick-send controls
- **Responsive UI** — Built with Bootstrap 5, Bootstrap Icons, and custom CSS; adapts to desktop and mobile
- **REST API** — JSON endpoints for auth and room operations, consumable by external clients

## Tech stack

| Layer             | Technology                                    |
|-------------------|-----------------------------------------------|
| Runtime           | .NET 8                                        |
| Web framework     | ASP.NET Core MVC + SignalR                    |
| Database          | SQLite via Entity Framework Core 8            |
| Authentication    | ASP.NET Core Identity (cookie-based)          |
| Frontend          | Bootstrap 5, Bootstrap Icons, Inter font      |
| Client-side       | jQuery, jQuery Validation, SignalR JS client  |

## Architecture

```
AdvancedChat.Web
├── Controllers/
│   ├── ApiAuthController.cs      — JSON auth endpoints
│   ├── ApiRoomsController.cs     — JSON room management API
│   ├── HomeController.cs         — Landing page, privacy, errors
│   └── RoomsController.cs        — MVC room CRUD + member management
├── Hubs/
│   └── ChatHub.cs                — SignalR hub for real-time messaging
├── Services/
│   └── ChatRoomService.cs        — Business logic for rooms, members, messages
├── Models/
│   ├── ChatRoom.cs               — Room entity
│   ├── ChatMessage.cs            — Message entity
│   ├── ChatRoomMember.cs         — Membership join entity
│   ├── RoomViewModels.cs         — View models for all room-related views
│   └── ErrorViewModel.cs         — Error page model
├── Data/
│   ├── ApplicationDbContext.cs   — EF Core context with Fluent API config
│   └── Migrations/               — Schema migrations (Identity + chat tables)
├── Areas/Identity/               — ASP.NET Core Identity scaffolded UI
│   └── Pages/Account/
│       ├── Login.cshtml          — Sign-in page
│       └── Register.cshtml       — Registration page
├── Views/
│   ├── Home/
│   │   ├── Index.cshtml          — Landing page (hero, features, CTA)
│   │   └── Privacy.cshtml        — Privacy policy
│   ├── Rooms/
│   │   ├── Index.cshtml          — Dashboard (rooms, broadcast, DMs, feed)
│   │   ├── Create.cshtml         — Room creation form
│   │   └── Details.cshtml        — Room detail view with live chat
│   └── Shared/
│       ├── _Layout.cshtml        — App shell with nav bar and footer
│       ├── _LoginPartial.cshtml  — Auth status in navbar
│       └── Error.cshtml          — Generic error page
└── wwwroot/
    ├── css/site.css              — Full custom stylesheet (900+ lines)
    ├── js/site.js                — Placeholder for app-level scripts
    └── lib/                      — Vendor libraries (Bootstrap, jQuery, etc.)
```

## Getting started

### Prerequisites

- [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0)

### Run locally

```bash
cd AdvancedChat.Web
dotnet run --launch-profile http
```

Open `http://localhost:5017` in a browser. Register an account, then create or join a room to start chatting.

### Database

The app uses SQLite with EF Core migrations. The database and tables are created automatically on first run. To reset:

```bash
cd AdvancedChat.Web
dotnet ef database drop --force
dotnet ef database update
```

### Configuration

All settings are in `appsettings.json`:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "DataSource=app.db;Cache=Shared"
  }
}
```

Override via environment variables, user secrets, or the `appsettings.Development.json` file.

## API reference

All API endpoints return JSON. Auth endpoints set a cookie on success; subsequent requests use cookie authentication.

### Auth

```
POST /api/auth/register    { "email", "password" }        → 200 { "email" }
POST /api/auth/login       { "email", "password" }        → 200 { "email" }
POST /api/auth/logout      [Auth]                         → 200
```

### Rooms

```
GET   /api/rooms           [Auth]                         → 200 [ { id, name, description, memberCount } ]
POST  /api/rooms           [Auth] { "name", "description"? } → 200 { id, name, description }
POST  /api/rooms/{id}/users [Auth] { "email" }            → 200 | 400 { "message" }
```

## SignalR hub

The real-time hub is available at `/chatHub` and requires authentication.

| Method              | Parameters                | Description                      |
|---------------------|---------------------------|----------------------------------|
| `JoinRoom`          | `roomId`                  | Subscribe to a room's messages   |
| `SendMessage`       | `roomId`, `message`       | Send a message to a room         |
| `SendPrivateMessage`| `targetUserId`, `message` | Send a direct message to a user  |
| `GetRecentMessages` | `roomId`                  | Retrieve last 50 messages        |

### Client events

| Event                     | Payload                                    |
|---------------------------|--------------------------------------------|
| `ReceiveMessage`          | `{ roomId, userName, text, sentAt }`      |
| `ReceivePrivateMessage`   | `{ fromUserId, toUserId, userName, text, sentAt }` |
| `SystemMessage`           | `{ message }`                             |

The JS client is configured with automatic reconnect. The connection state is displayed in the UI as a badge (Connected / Reconnecting / Offline).

## Security

- All hub methods require an authenticated user (`[Authorize]` on the hub class)
- Room membership is enforced server-side before any read or write operation
- Message length is capped at 2000 characters on the server
- Input is validated on both client and server
- ASP.NET Core Data Protection keys are persisted to `App_Data/Keys/` (excluded from version control)
- HSTS is enabled in production

## Development

### Prerequisites for development

```bash
dotnet --version    # 8.0.x required
```

### Build and run

```bash
dotnet build
dotnet run --project AdvancedChat.Web
```

### Add a migration

```bash
dotnet ef migrations add MigrationName --project AdvancedChat.Web
dotnet ef database update --project AdvancedChat.Web
```

### Project conventions

- File-scoped namespaces throughout
- Primary constructor usage where applicable
- View models in `Models/`, business logic in `Services/`
- Async all the way — no `.Result` or `.Wait()` calls
- JSON serialization uses `JsonSerializerDefaults.Web` (camelCase, case-insensitive)

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/my-feature`)
3. Commit your changes (`git commit -am 'Add my feature'`)
4. Push to the branch (`git push origin feature/my-feature`)
5. Open a pull request

## License

MIT
