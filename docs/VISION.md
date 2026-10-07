# ShotOnLoc — Vision & Decisions

This document records every product and technical decision made before development started. Specs for individual milestones are derived from it. When a decision changes, update this file.

## Idea

A worldwide map of filming locations. Users mark **Scenes** from movies and TV series at the exact spot they were shot, upload their own **Photos** of the location today, and link videos.

## Goals and constraints

- Learning and portfolio project, public on GitHub.
- Budget: **0 €**. Every service must have a free tier or be self-hosted.
- Primary audience: **film tourists** ("I'm in Rome, which filming locations are near me?"). Film fans searching by title are a secondary audience.
- Work split: infrastructure tickets (Docker, CI, security config) are implemented by an agent; domain feature tickets (Wikidata import, geo queries, upload pipeline) are implemented by the human developer (`ready-for-human`).

## Product decisions

### Core model

- **One marker = one Scene.** A Scene belongs to exactly one **Title** and has one coordinate. Three movies shot at the same bridge are three Scenes.
- **Titles** are movies **and** TV series, always referenced from TMDB (no free-text titles). For series, a Scene may optionally record season and episode.
- **Required Scene fields:** Title, coordinate, short scene name (e.g. "Chase on the stairs").
- **Optional Scene fields:** description, timestamp in the film (e.g. 01:12:30), Photos, video links, visitor notes ("only accessible during daytime", "private property").
- **Placing the pin:** the app proposes a coordinate (from the uploaded photo's EXIF GPS data or the device location); the user drags the pin and confirms.
- **Scenes belong to the community, Photos belong to their uploader.** When creating a Scene, the app shows existing Scenes of the same Title nearby ("Did you mean one of these?"). Other users add their Photos to an existing Scene instead of creating a duplicate.

### Media

- Uploads: **own photos only** — no film stills or clips (copyright).
- Film imagery is shown only via TMDB images or embeds (e.g. YouTube trailer).
- Videos are **links/embeds only** (YouTube, TikTok, Instagram), no video upload.
- Limits: max 10 photos per upload, max 15 MB per original, JPEG/PNG/WebP/HEIC (HEIC converted to JPEG in the browser before upload), max 50 uploads per user per day.
- Pipeline: read EXIF GPS (pin proposal) → strip all EXIF → resize to max 2000 px WebP → generate thumbnail.

### Seed content

- **Wikidata import** is the main seed source: filming locations (P915) of titles that have a TMDB ID (P4947 movie / TV ID), restricted to locations with their own coordinate (P625) that are **not** a city, region or country.
- Imported Scenes get the status **imported/unconfirmed** and a generic name ("Filming location: Rocky Steps"), rendered with a distinct pin style. A user can **claim** an imported Scene and becomes its creator.
- The import runs as a separate, manually repeatable job, not during normal operation.
- Plus a handful of hand-curated Scenes in the developer's home city.

### Discovery

- MVP: map with marker clustering, "near me" (browser geolocation → radius search), title search → title page listing all its Scenes on a mini map.
- v2: filters (genre, decade). v3: walking tours.

### Social

- MVP: profile page (own Scenes and Photos), **Visit** ("I was here") check-in.
- v2: comments, "pin is wrong" accuracy reports (important for imported Scenes).
- Not planned: following users.

### Language

- UI in **German and English** from the start.
- User content stays in the language it was written in; Title names come from TMDB in the viewer's language.

## Accounts, roles and rights

- Read access without login is switchable via the `app.public-read` flag. First deployment starts with `false` (private).
- Registration requires an **Invite Code**. Only admins create codes; each code is single-use and valid for 14 days.
- Login: email + password **and** Google/GitHub OAuth2. No email verification, no password reset by mail (an admin can reset a password). Login methods are linked manually only, while logged in — never automatically by matching email.
- Roles: `USER`, `ADMIN`. The first admin is created from environment variables at startup.
- Editing a Scene's name, description and pin: creator + admin. Deleting a Photo: uploader + admin.
- Deleting an account deletes the user's Photos and **anonymizes** their Scenes ("deleted user"), because other users may have attached Photos to them.

## Technical decisions

| Area | Decision |
|---|---|
| Language / framework | Java 25, Spring Boot 4, Maven |
| UI | Server-rendered: Thymeleaf + htmx + Bootstrap 5 (WebJars), no Node toolchain |
| Map | Leaflet + OpenStreetMap tiles + clustering plugin |
| Platform | Progressive Web App first; native apps maybe later (would add REST endpoints) |
| Database | MySQL 8 with spatial types/indexes (SRID 4326), Docker container on port 3307 |
| Schema | Flyway migrations (no Hibernate `ddl-auto`) |
| File storage | S3 API: RustFS locally (MinIO no longer publishes free Docker images), Cloudflare R2 in production |
| Title data | TMDB API (attribution notice in footer) |
| Code structure | Package by feature (`title`, `scene`, `media`, `user`, `importer`, …), modular monolith |
| Tests | JUnit 5 unit tests + integration tests against real MySQL via Testcontainers |
| CI | GitHub Actions: build + test on every push |
| Local dev | `docker compose up` starts MySQL + RustFS |
| Hosting | Local only for now; later Oracle Cloud Always Free VM |
| Secrets | Environment variables / `.env` (git-ignored), `.env.example` committed |
| Repo language | English (code, commits, docs) |
| License | MIT |
| Issue tracker | GitHub Issues, one spec issue per milestone |

## Milestones

| # | Milestone | Scope |
|---|---|---|
| M0 | Skeleton | Spring Boot app, Docker Compose (MySQL + RustFS), Flyway, i18n setup (DE/EN), GitHub Actions, README |
| M1 | Read-only map | Title + Scene tables, a few seed Scenes via SQL, Leaflet map with clustering, endpoint "Scenes in viewport" |
| M2 | Wikidata import | SPARQL query, TMDB matching, imported/unconfirmed status |
| M3 | Accounts | Registration with Invite Code, password login, Google/GitHub OAuth2, manual account linking, admin role, `app.public-read` flag |
| M4 | Create Scenes | TMDB title search, place/drag pin, duplicate hint, claim imported Scenes, edit rights |
| M5 | Photos | Upload, browser HEIC conversion, resize/WebP, thumbnail, EXIF → pin proposal, EXIF stripping, S3 storage (RustFS), video links, limits |
| M6 | Discovery | "Near me", title page with mini map |
| M7 | Social | Profile page, Visits, account deletion with anonymization |
| M8 | PWA | Manifest, service worker, installable, mobile polish |
| M9 | Deployment | Oracle VM, domain/HTTPS, imprint & privacy policy, Cloudflare R2, invite-only launch |
