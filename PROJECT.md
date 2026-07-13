# Navigater

**Human-like navigation, powered by AI and Google Maps.**

AI Navigation reinterprets raw Google Maps routing data into natural, conversational, human-style directions — the way a friend riding shotgun would guide you, rather than a robotic turn-by-turn list.

---

## 1. Overview

| | |
|---|---|
| **App Name** | NaviGater |
| **Purpose** | Human-like, conversational turn-by-turn navigation built on top of Google Maps |
| **Architecture** | Clean Architecture (layered, dependency-inverted) |
| **Client** | Flutter (iOS / Android) |
| **Backend** | FastAPI (Python) |
| **Database** | PostgreSQL |
| **Auth / Realtime / Cloud Services** | Firebase |
| **Mapping / Routing** | Google Maps API |

---

## 2. Core Concept

Standard GPS apps give mechanical instructions ("In 200 meters, turn right onto Main St"). AI Navigation layers a natural-language generation step on top of Google Maps' routing/directions data to produce guidance that sounds human:

- Landmark-based cues ("Turn right just after the coffee shop on the corner")
- Context-aware phrasing (traffic, time of day, driving vs. walking)
- Conversational tone, adjustable to the user's preferred style
- Proactive, friendly heads-ups instead of rigid distance countdowns

---

## 3. Tech Stack

### Frontend — Flutter
- Cross-platform client for iOS and Android
- Google Maps Flutter plugin for map rendering
- Text-to-speech for spoken human-like directions
- State management (Bloc/Riverpod/Provider — team's choice)

### Backend — FastAPI
- Serves as the orchestration layer between the client, Google Maps API, and the AI language layer
- Exposes REST (and optionally WebSocket) endpoints for route requests, live guidance updates, and user data
- Handles business logic, request validation, and AI prompt orchestration

### Database — PostgreSQL
- Stores user profiles, saved places, trip history, and preferences
- Relational integrity for structured navigation and account data

### Firebase
- Authentication (sign-in / sign-up)
- Push notifications (traffic alerts, arrival notices)
- Optional: Firestore/Realtime Database for lightweight live sync, Crashlytics, Analytics

### Google Maps API
- Directions API — base route calculation
- Roads API / Places API — landmark and road-context data
- Maps SDK (Flutter plugin) — map rendering on-device

---

## 4. Architecture — Clean Architecture

The project follows Clean Architecture principles on **both** the Flutter client and the FastAPI backend, ensuring separation of concerns, testability, and independence from frameworks/tools.

### 4.1 Layers

```
┌─────────────────────────────────────────┐
│              Presentation                │  UI, Widgets, ViewModels/Controllers
├─────────────────────────────────────────┤
│                Domain                    │  Entities, Use Cases, Repository Interfaces
├─────────────────────────────────────────┤
│                  Data                    │  Repository Implementations, Data Sources, DTOs/Models
├─────────────────────────────────────────┤
│              Core / Infra                │  Networking, DB, Firebase, Google Maps SDK, Error handling
└─────────────────────────────────────────┘
```

**Dependency rule:** outer layers depend on inner layers, never the reverse. The Domain layer has zero knowledge of Flutter, FastAPI, PostgreSQL, or Firebase.

### 4.2 Flutter Client Structure

```
lib/
├── core/
│   ├── error/              # Failures, exceptions
│   ├── network/            # Dio/HTTP client, interceptors
│   ├── utils/              # Constants, helpers, extensions
│   └── di/                 # Dependency injection setup (get_it / injectable)
│
├── features/
│   └── navigation/
│       ├── presentation/
│       │   ├── pages/
│       │   ├── widgets/
│       │   └── controllers/    # Bloc/Cubit/Provider
│       ├── domain/
│       │   ├── entities/
│       │   ├── repositories/       # abstract interfaces
│       │   └── usecases/
│       └── data/
│           ├── models/
│           ├── datasources/         # remote (API), local (cache)
│           └── repositories/        # implementations
│
└── main.dart
```

### 4.3 FastAPI Backend Structure

```
app/
├── core/
│   ├── config.py           # Settings, env vars
│   ├── security.py         # Auth/Firebase token verification
│   └── exceptions.py
│
├── domain/
│   ├── entities/            # Pure business objects
│   ├── repositories/        # Abstract repository interfaces (ports)
│   └── use_cases/           # Business logic / application rules
│
├── infrastructure/
│   ├── database/
│   │   ├── models.py         # SQLAlchemy ORM models
│   │   └── postgres_repository.py   # Repository implementations
│   ├── maps/
│   │   └── google_maps_client.py
│   ├── ai/
│   │   └── navigation_language_service.py   # Human-like instruction generation
│   └── firebase/
│       └── firebase_client.py
│
├── presentation/
│   ├── api/
│   │   └── v1/
│   │       ├── routes/          # FastAPI routers
│   │       └── schemas/         # Pydantic request/response models
│   └── dependencies.py       # DI wiring (FastAPI Depends)
│
└── main.py
```

### 4.4 Benefits for This Project
- **Testability** — Use cases (e.g., "Generate human-like route") can be unit tested without Google Maps, PostgreSQL, or Firebase.
- **Swappable infrastructure** — Google Maps could be supplemented/replaced with another provider without touching domain logic.
- **Clear boundaries** — AI language-generation logic stays isolated from routing/data-fetching logic.

---

## 5. High-Level Data Flow

1. User requests a route in the **Flutter app** (origin → destination).
2. Request hits **FastAPI** via a use case (`GetHumanNavigationRoute`).
3. Backend calls **Google Maps Directions API** for raw route/steps.
4. Backend's AI language layer transforms raw steps into human-like guidance (landmarks, natural phrasing).
5. Response returned to Flutter client; **PostgreSQL** logs the trip; **Firebase** handles auth/notifications.
6. Flutter renders the route on the map and optionally speaks directions via TTS.

---

## 6. Key Features (Planned)

- [ ] Human-like, landmark-based turn-by-turn guidance
- [ ] Voice narration with natural phrasing
- [ ] Saved places & trip history
- [ ] Real-time traffic-aware rerouting
- [ ] Firebase authentication (Google/Apple/Email sign-in)
- [ ] Push notifications for traffic/arrival alerts
- [ ] Offline caching of recent routes

---

## 7. Getting Started (Placeholder)

```bash
# Flutter client
cd mobile
flutter pub get
flutter run

# FastAPI backend
cd backend
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --reload
```

Environment variables required:
```
GOOGLE_MAPS_API_KEY=
FIREBASE_PROJECT_ID=
FIREBASE_CREDENTIALS_JSON=
DATABASE_URL=postgresql://user:password@localhost:5432/ai_navigation
```

---

## 8. Roadmap

| Phase | Goal |
|---|---|
| Phase 1 | Core routing + human-like instruction generation (MVP) |
| Phase 2 | Voice narration, trip history, saved places |
| Phase 3 | Real-time traffic adaptation, offline support |
| Phase 4 | Personalization (learning user's preferred phrasing/style) |

---

## 9. License

_To be determined._
