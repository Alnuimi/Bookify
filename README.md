<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:0d1117,50:0f3460,100:1a1f2e&height=220&section=header&text=Bookify&fontSize=80&fontColor=00d4aa&fontAlignY=38&desc=Apartment%20Booking%20System%20%7C%20Clean%20Architecture%20%7C%20.NET&descAlignY=60&descSize=18&descColor=8b949e&animation=fadeIn" width="100%"/>

<br/>

[![.NET](https://img.shields.io/badge/.NET-8.0-512BD4?style=for-the-badge&logo=dotnet&logoColor=white)](https://dotnet.microsoft.com/)
[![C#](https://img.shields.io/badge/C%23-12.0-239120?style=for-the-badge&logo=csharp&logoColor=white)](https://learn.microsoft.com/en-us/dotnet/csharp/)
[![EF Core](https://img.shields.io/badge/EF_Core-ORM-512BD4?style=for-the-badge&logo=dotnet&logoColor=white)](https://docs.microsoft.com/ef/)
[![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-Welcome-brightgreen?style=for-the-badge)](CONTRIBUTING.md)

<br/>

> **A production-ready apartment booking platform built on Clean Architecture, DDD, and CQRS — designed for maintainability, testability, and scalability.**

</div>

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Layer Breakdown](#-layer-breakdown)
- [Tech Stack](#-tech-stack)
- [Domain Model](#-domain-model)
- [Getting Started](#-getting-started)
- [Project Structure](#-project-structure)
- [Testing](#-testing)
- [Design Patterns](#-design-patterns)

---

## 🏠 Overview

**Bookify** is a full-featured apartment booking system built with **C#** and **.NET**, following **Clean Architecture** principles to ensure a modular, maintainable, and testable codebase. 

The system allows users to search for apartments, make reservations, process payments, and leave reviews — all powered by a robust domain model with rich business rules.

```
Users ──► Search Apartments ──► Reserve ──► Confirm Booking ──► Review
```

**Core Goals:**
- Clear separation of concerns across all architectural layers
- Domain-centric design with rich business logic
- Testable by design — every layer is independently verifiable
- Production-ready patterns: Outbox, CQRS, Circuit Breaker, Optimistic Concurrency

---

## 🏛️ Architecture

Bookify follows a **Domain-Centric Clean Architecture**, where dependencies always flow inward — outer layers depend on inner layers, never the reverse.

```
┌─────────────────────────────────────────────┐
│              Presentation Layer              │  ← REST API, Controllers, Middleware
├─────────────────────────────────────────────┤
│              Application Layer              │  ← Use Cases, CQRS, MediatR, Validation
├─────────────────────────────────────────────┤
│               Domain Layer                  │  ← Entities, Value Objects, Domain Events
├─────────────────────────────────────────────┤
│            Infrastructure Layer             │  ← EF Core, RabbitMQ, Identity, Email
└─────────────────────────────────────────────┘
```

**Key Architectural Principles:**

| Principle | Description |
|---|---|
| **Separation of Concerns** | Each layer has a single, well-defined responsibility |
| **Dependency Inversion** | Inner layers define interfaces; outer layers implement them |
| **Persistence Ignorance** | Domain has zero knowledge of how data is stored |
| **Bounded Contexts** | Clear boundaries prevent domain logic leakage |
| **Encapsulation** | State changes only happen through well-defined domain methods |
| **Single Responsibility** | Every class has one reason to change |

---

## 🔍 Layer Breakdown

### 🟣 Domain Layer — The Heart of the System

The core of the application. Contains **pure business logic** with no external dependencies.

```csharp
// Example: Rich Domain Entity
public sealed class Booking : Entity
{
    public ApartmentId ApartmentId { get; private set; }
    public UserId UserId { get; private set; }
    public DateRange Duration { get; private set; }
    public Money TotalPrice { get; private set; }
    public BookingStatus Status { get; private set; }

    public static Result<Booking> Reserve(
        Apartment apartment,
        UserId userId,
        DateRange duration,
        PricingService pricingService)
    {
        // Rich business rules enforced here
    }
}
```

**Contains:** Entities · Value Objects · Domain Events · Domain Services · Repository Interfaces · Exceptions · Enums

---

### 🔴 Application Layer — Orchestrating the Domain

Coordinates domain objects to fulfill use cases. Implements **CQRS** via **MediatR**.

```csharp
// Command: Reserve an Apartment
public record ReserveApartmentCommand(
    Guid ApartmentId,
    Guid UserId,
    DateOnly StartDate,
    DateOnly EndDate) : ICommand<Guid>;

// Handler
internal sealed class ReserveApartmentCommandHandler : ICommandHandler<ReserveApartmentCommand, Guid>
{
    public async Task<Result<Guid>> Handle(
        ReserveApartmentCommand request,
        CancellationToken cancellationToken)
    {
        // Orchestrate: fetch → validate → execute → persist → publish
    }
}
```

**Contains:** Commands & Queries · Handlers · Validators (FluentValidation) · Behaviors (Pipeline) · DTOs · Abstractions

---

### 🔵 Infrastructure Layer — External World Integration

Implements the interfaces defined by inner layers. Handles all I/O concerns.

```csharp
// EF Core Repository Implementation
internal sealed class ApartmentRepository : IApartmentRepository
{
    private readonly ApplicationDbContext _context;

    public async Task<Apartment?> GetByIdAsync(
        ApartmentId id,
        CancellationToken cancellationToken = default) =>
        await _context.Apartments
            .FirstOrDefaultAsync(a => a.Id == id, cancellationToken);
}
```

**Contains:** EF Core DbContext · Entity Configurations · Repositories · Keycloak Integration · RabbitMQ Publisher · Email Provider · Outbox Pattern · Background Jobs

---

### 🟢 Presentation Layer — System Entry Point

Exposes the application via a REST API. Thin layer — delegates everything to Application.

```csharp
[ApiController]
[Route("api/v{version:apiVersion}/bookings")]
public class BookingsController : ControllerBase
{
    [HttpPost]
    public async Task<IActionResult> ReserveApartment(
        ReserveApartmentRequest request,
        CancellationToken cancellationToken)
    {
        var command = new ReserveApartmentCommand(/* map request */);
        var result = await _sender.Send(command, cancellationToken);
        return result.IsSuccess ? Ok(result.Value) : BadRequest(result.Error);
    }
}
```

**Contains:** Controllers / Minimal API Endpoints · Middleware · DI Composition Root · Docker Compose Setup · API Versioning

---

## ⚒️ Tech Stack

<div align="center">

### Backend Core
![.NET](https://img.shields.io/badge/.NET_8-512BD4?style=for-the-badge&logo=dotnet&logoColor=white)
![C#](https://img.shields.io/badge/C%23-239120?style=for-the-badge&logo=csharp&logoColor=white)
![ASP.NET Core](https://img.shields.io/badge/ASP.NET_Core-512BD4?style=for-the-badge&logo=dotnet&logoColor=white)
![MediatR](https://img.shields.io/badge/MediatR-CQRS-blue?style=for-the-badge)

### Data & Messaging
![SQL Server](https://img.shields.io/badge/SQL_Server-CC2927?style=for-the-badge&logo=microsoftsqlserver&logoColor=white)
![EF Core](https://img.shields.io/badge/EF_Core-512BD4?style=for-the-badge&logo=dotnet&logoColor=white)
![RabbitMQ](https://img.shields.io/badge/RabbitMQ-FF6600?style=for-the-badge&logo=rabbitmq&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white)

### Security & Identity
![Keycloak](https://img.shields.io/badge/Keycloak-4D4D4D?style=for-the-badge&logo=keycloak&logoColor=white)
![JWT](https://img.shields.io/badge/JWT-000000?style=for-the-badge&logo=jsonwebtokens&logoColor=white)

### DevOps & Observability
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Serilog](https://img.shields.io/badge/Serilog-Logging-brightgreen?style=for-the-badge)
![xUnit](https://img.shields.io/badge/xUnit-Testing-512BD4?style=for-the-badge)

</div>

---

## 🗂️ Domain Model

The system is centered around four core aggregates:

```
┌──────────────┐         ┌─────────────────────────────────────┐
│     User     │         │              Booking                 │
│──────────────│    ┌───►│──────────────────────────────────── │
│ Id: Guid     │    │    │ Id · ApartmentId · UserId           │
│ FirstName    │────┘    │ Start · End · PriceForPeriod        │
│ LastName     │         │ CleaningFee · AmenitiesUpCharge     │
│ Email        │         │ TotalPrice · Status                 │
└──────────────┘         │ CreatedOnUtc · ConfirmedOnUtc       │
                         └───────────────────┬─────────────────┘
┌──────────────────────┐                     │
│      Apartment       │                     │
│──────────────────────│                     ▼
│ Id · Name            │         ┌───────────────────┐
│ Description          │         │      Review        │
│ Address (VO)         │         │───────────────────│
│ Price (Money VO)     │         │ Id · ApartmentId  │
│ CleaningFee          │         │ BookingId · UserId│
│ Amenities[]          │         │ Rating · Comment  │
│ LastBookedOnUtc      │         └───────────────────┘
└──────────────────────┘
```

**Value Objects:** `Money` · `Address` · `DateRange` · `ApartmentId` · `UserId` · `BookingId`

---

## 🚀 Getting Started

### Prerequisites

- [.NET 8 SDK](https://dotnet.microsoft.com/download)
- [Docker & Docker Compose](https://www.docker.com/)
- [SQL Server](https://www.microsoft.com/sql-server) (or use Docker)

### Run with Docker Compose

```bash
# Clone the repository
git clone https://github.com/Alnuimi/Bookify.git
cd Bookify

# Start all services (SQL Server, RabbitMQ, Redis, Keycloak)
docker-compose up -d

# Apply database migrations
dotnet ef database update --project src/Bookify.Infrastructure

# Run the API
dotnet run --project src/Bookify.Api
```

### Run Locally

```bash
# Restore dependencies
dotnet restore

# Build the solution
dotnet build

# Run tests
dotnet test

# Start the API
dotnet run --project src/Bookify.Api
```

The API will be available at `https://localhost:5001` with Swagger at `https://localhost:5001/swagger`.

---

## 📁 Project Structure

```
Bookify/
│
├── src/
│   ├── Bookify.Domain/              # 🟣 Core business logic
│   │   ├── Apartments/              
│   │   │   ├── Apartment.cs         # Aggregate root
│   │   │   ├── Address.cs           # Value object
│   │   │   └── Money.cs             # Value object
│   │   ├── Bookings/
│   │   │   ├── Booking.cs           # Aggregate root
│   │   │   ├── BookingStatus.cs     # Enum
│   │   │   └── Events/              # Domain events
│   │   └── Users/
│   │       └── User.cs
│   │
│   ├── Bookify.Application/         # 🔴 Use cases & orchestration
│   │   ├── Abstractions/
│   │   │   ├── ICommand.cs
│   │   │   └── IQuery.cs
│   │   ├── Apartments/
│   │   │   └── SearchApartments/    # Query + Handler
│   │   ├── Bookings/
│   │   │   ├── ReserveApartment/    # Command + Handler
│   │   │   └── ConfirmBooking/      # Command + Handler
│   │   └── Behaviors/               # Pipeline behaviors
│   │       ├── LoggingBehavior.cs
│   │       └── ValidationBehavior.cs
│   │
│   ├── Bookify.Infrastructure/      # 🔵 External integrations
│   │   ├── Data/
│   │   │   ├── ApplicationDbContext.cs
│   │   │   └── Configurations/      # EF entity configs
│   │   ├── Repositories/
│   │   ├── Authentication/          # Keycloak integration
│   │   ├── Messaging/               # RabbitMQ
│   │   ├── Outbox/                  # Outbox pattern
│   │   └── BackgroundJobs/
│   │
│   └── Bookify.Api/                 # 🟢 REST API entry point
│       ├── Controllers/
│       ├── Middleware/
│       └── Program.cs
│
└── tests/
    ├── Bookify.Domain.UnitTests/
    ├── Bookify.Application.UnitTests/
    ├── Bookify.Application.IntegrationTests/
    └── Bookify.Architecture.Tests/
```

---

## 🧪 Testing

The solution includes a comprehensive testing strategy across all layers:

```bash
# Run all tests
dotnet test

# Run with coverage
dotnet test --collect:"XPlat Code Coverage"
```

| Test Type | Target | Tool |
|---|---|---|
| **Unit Tests** | Domain logic, Application handlers | xUnit + NSubstitute |
| **Integration Tests** | Infrastructure, API endpoints | WebApplicationFactory |
| **Architecture Tests** | Layer dependency rules | NetArchTest |

**Architecture tests enforce that:**
- Domain has **zero** external dependencies
- Application only references Domain
- Infrastructure does not reference Presentation
- Controllers are thin and delegate to MediatR

---

## 🧩 Design Patterns

| Pattern | Location | Purpose |
|---|---|---|
| **CQRS** | Application | Separate read/write models |
| **Repository** | Domain/Infrastructure | Persistence abstraction |
| **Outbox** | Infrastructure | Reliable domain event publishing |
| **Unit of Work** | Infrastructure | Transactional consistency |
| **Pipeline Behavior** | Application | Cross-cutting concerns (logging, validation) |
| **Result Pattern** | Domain/Application | Explicit error handling without exceptions |
| **Optimistic Concurrency** | Infrastructure | Conflict detection in concurrent updates |
| **Domain Events** | Domain | Decouple side effects from core logic |

---

<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:0f3460,50:1a1f2e,100:0d1117&height=120&section=footer&text=Built+with+Clean+Architecture+%E2%9D%A4&fontSize=20&fontColor=00d4aa&fontAlignY=65&animation=fadeIn" width="100%"/>

**⭐ If this project helped you, consider giving it a star!**

[![GitHub stars](https://img.shields.io/github/stars/Alnuimi/Bookify?style=social)](https://github.com/Alnuimi/Bookify)

</div>
