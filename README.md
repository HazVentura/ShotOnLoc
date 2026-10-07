# ShotOnLoc

A worldwide map of movie and TV filming locations. Find the exact spot where a scene was shot, see what it looks like today, and add your own photos.

> Early development — nothing to run yet.

## Planned stack

Java 25 · Spring Boot 4 · Thymeleaf + htmx · Bootstrap 5 · Leaflet / OpenStreetMap · MySQL 8 (spatial) · Flyway · RustFS / S3 · Testcontainers · GitHub Actions

## Run locally

Requirements: Docker with Compose.

```bash
cp .env.example .env     # then adjust the passwords
docker compose up -d     # start the backing services
docker compose ps        # wait until they are "healthy"
docker compose down      # stop them (data is kept in volumes)
docker compose down -v   # stop them and delete all data
```

| Service   | Address                                | Credentials                           |
| --------- | -------------------------------------- | ------------------------------------- |
| MySQL 8.4 | `localhost:3307`, database `shotonloc` | `DB_USER` / `DB_PASSWORD` from `.env` |
| RustFS (S3 API) | `http://localhost:9000` | `S3_ACCESS_KEY` / `S3_SECRET_KEY` from `.env` |
| RustFS console | <http://localhost:9001/rustfs/console/> | same as S3 API |

## Docs

- [Vision & decisions](docs/VISION.md)
- [Glossary](GLOSSARY.md)

## Attribution

This product uses the TMDB API but is not endorsed or certified by TMDB.

## License

[MIT](LICENSE)
