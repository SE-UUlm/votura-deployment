# Votura-Deployment

In this repository, you will find the necessary information  and resources to deploy an instance of the [Votura application](https://github.com/SE-UUlm/votura) on your own server.

## Prerequisites

You need something that can run Docker containers.
Whether that is a local machine, a virtual machine, or some sort of cloud service does not matter—however, this repository contains examples for a Docker-Compose-based deployment.
So while you are, of course, free to use other deployment methods (Kubernetes, Docker Swarm, or (perhaps proprietary) solutions of any cloud providers), you will have to adapt the instructions accordingly on your own.
Furthermore, you need to have a domain name that points to the server where you want to deploy Votura.
In this repository, we will consistently use the domain name `domain.com` as an example.
Finally, it is highly recommended to have an SMTP server that you can use to send emails from your Votura instance.
This feature is currently only used to send account activation and password reset emails, so if you choose to use Votura only on your own, and you do not need these features, you can leave these settings blank.

## SSL Termination

To ensure that your Votura instance uses a secure connection, this repository doesn't even provide examples for deployments without SSL termination.
Instead, we provide examples that already integrate [Traefik](https://traefik.io/) as a reverse proxy that handles SSL termination with [Let's Encrypt](https://letsencrypt.org/) certificates.
Besides that, as a reverse proxy, it also helps to route requests to the correct service (frontend or backend) based on the specific HTTP request.
To ensure configuration compatibility, these examples use the image `traefik:v3.7`, but if a newer version is available, it is recommended to use that instead and, if necessary, adjust the configuration accordingly.

## Images

The whole Votura application is split into three separate Docker images, which are automatically built from the source code and pushed to GitHub Container Registry (GHCR) whenever a new release is created.

### `votura-database`

The database container runs a PostgreSQL database with all the necessary plugins and extensions.
The image does _not_ contain any data, and schema updates instead rely on migration scripts that are executed by the backend container on startup.
That means that updating the container image does not result in any data loss, as long as the data volume is preserved.
The container must _not_ be exposed to the public, connections to the database are _only_ allowed internally from the Docker-Compose stack.

### `votura-backend`

The backend container runs a vite application server that serves the backend API.
It uses the `votura-database` container to store and retrieve data.
Furthermore, on every startup, it runs the migration scripts to update the database schema if necessary.

### `votura-frontend`

The frontend container runs an nginx web server to serve the static files for the user interface.

## Environment Variables

### `votura-database`

| Variable            | Description       | Example  |
|---------------------|-------------------|----------|
| `POSTGRES_USER`     | Database username | `votura` |
| `POSTGRES_PASSWORD` | Database password | `votura` |
| `POSTGRES_DB`       | Database name     | `votura` |

### `votura-backend`

| Variable                  | Description                                | Example                                                    |
|---------------------------|--------------------------------------------|------------------------------------------------------------|
| `NODE_ENV`                | Node environment                           | `production`                                               |
| `DATABASE_URL`            | Database connection string                 | `postgresql://votura:votura@postgres/votura?schema=public` |
| `PEPPER`                  | Secret key for password hashing            | `supersecretpepper`                                        |
| `BITS_PRIME_P`            | Number of bits for prime number generation | `4096` (2048 bits or more)                                 |
| `KEY_GEN_TIMEOUT_MINUTES` | Timeout for key generation in minutes      | `15`                                                       |
| `SMTP_HOST`               | SMTP server hostname                       | `smtp.domain.com`                                          |
| `SMTP_PORT`               | SMTP server port                           | `465`                                                      |
| `SMTP_SECURE`             | Use secure connection for SMTP             | `true`                                                     |
| `MAIL_SENDER_NAME`        | Name of the sender for emails              | `Votura`                                                   |
| `MAIL_SENDER_EMAIL`       | Email address of the sender for emails     | `noreply@domain.com`                                       |
| `FRONTEND_BASE_URL`       | Base URL of the frontend application       | `https://domain.com`                                       |

### `votura-frontend`

| Variable           | Description                 | Example                  |
|--------------------|-----------------------------|--------------------------|
| `BACKEND_BASE_URL` | Base URL of the backend API | `https://api.domain.com` |

## Examples

- [API on same domain name](./examples/api-on-same-domain-name/)
- [API on separate subdomain](./examples/api-on-separate-subdomain/)
- [Automatic image updates](./examples/automatic-image-updates/)
