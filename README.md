# next-gin-ops

Production deployment configuration for a **Next.js + Gin** stack, orchestrated with Docker Compose and proxied through Nginx.

## Stack

| Service    | Image                                              | Role                        |
|------------|----------------------------------------------------|-----------------------------|
| `frontend` | `ghcr.io/aonoexercist/next-gin-frontend:latest`    | Next.js frontend            |
| `backend`  | `ghcr.io/aonoexercist/gin-api-backend:latest`      | Gin (Go) REST API           |
| `nginx`    | `nginx:alpine`                                     | Reverse proxy (port 80/443) |

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) & [Docker Compose](https://docs.docker.com/compose/install/)
- Access to the GitHub Container Registry images (or build them yourself)

## Getting Started

1. **Clone the repository**

   ```bash
   git clone https://github.com/aonoexercist/next-gin-ops.git
   cd next-gin-ops
   ```

2. **Configure environment variables**

   Edit `docker-compose.yml` and replace the placeholder value:

   ```yaml
   environment:
     - DB_URL=your_db_connection_string
   ```

3. **Add Nginx config**

   Place your Nginx server block files inside `nginx/conf.d/` (e.g., `default.conf`). The directory is mounted read-only into the Nginx container.

4. **Start the services**

   ```bash
   docker compose up -d
   ```

5. **Stop the services**

   ```bash
   docker compose down
   ```

## Project Structure

```
next-gin-ops/
├── docker-compose.yml   # Service definitions
├── nginx/
│   └── conf.d/          # Nginx configuration files (mount point)
└── README.md
```

## Notes

- HTTPS (port 443) is exposed but requires SSL certificates and an updated Nginx config.
- All services are configured with `restart: always` for automatic recovery after reboots.
