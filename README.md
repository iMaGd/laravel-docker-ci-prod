# Laravel Docker CI Template – Ready-to-Copy Dockerized Laravel + GitLab CI Setup

This repository provides a production-ready **Docker and CI/CD setup for Laravel 11+**, focused on simplicity and portability. It's perfect for developers who want to **dockerize their Laravel application** and **enable GitLab CI/CD** with minimal configuration.

> ✅ **Copy & paste friendly!** Easily integrate these files into your existing Laravel project and go live with Docker and GitLab CI/CD in minutes.

---

## 🧱 What You Get

- ✅ A multi-stage **Dockerfile** optimized for performance and clean builds
- ✅ A robust `.gitlab-ci.yml` with static checks, Docker image build + push, and runtime health checks
- ✅ A clear `docker compose` setup tailored for **local dev** and **CI testing**
- ✅ Preconfigured PHP-FPM 8.3 & Nginx with Laravel defaults

---

## 📁 Project Structure

```
ci/
├── docker/
│   ├── php/
│   │   ├── Dockerfile         # Laravel + PHP-FPM optimized Dockerfile
│   │   ├── entrypoint.sh      # Entrypoint script for container boot
│   │   ├── fpm.conf           # PHP-FPM tuning
│   │   └── php.ini            # PHP settings
│   └── webserver/
│       ├── nginx-dev.conf     # Local dev nginx config
│       └── nginx-prod.conf    # Production nginx config
├── testing/
│   └── compose.yml            # Used by GitLab for runtime HTTP checks
├── README.Docker.md           # Additional Docker notes
compose.yml                    # Base compose file
.gitlab-ci.yml                 # GitLab CI/CD pipeline
```

---

## ⚙️ Getting Started

### 1. Copy These Files into Your Laravel Project

Add the `ci/`, `.gitlab-ci.yml`, `compose.yml`, and `.dockerignore` files to your Laravel repo.


### 2. Install Static Testing Modules

```bash
composer require --dev enlightn/security-checker
composer require --dev phpstan/phpstan
composer require --dev laravel/pint
```

### 3. Build and Run the Image Locally

```bash
docker compose up -d --build
```

Make sure to configure your `.env` and database connections accordingly.


### 4. 🛠️ GitLab CI/CD Setup

Ensure your GitLab runner supports **Docker-in-Docker (dind)**. Then, commit your `.gitlab-ci.yml` and push to trigger CI.

#### Key GitLab CI Features:

- **Static Analysis**: PHPStan & Security Checker
- **Multi-platform Docker Build & Push** (via `docker buildx`)
- **Runtime HTTP Health Check** to validate deployment success
- Configurable with minimal env vars

---

## 🐘 Dockerfile Highlights

- **Multi-stage**: Builds frontend assets (Vite, Tailwind) separately using Node.js
- **Production Optimized**: No dev dependencies, `php artisan optimize`, storage permissions handled
- **Base Image**: [depicter/php:8.3-fpm-alpine](https://hub.docker.com/r/depicter/php)

---

## 🧪 CI Test Strategy

The `.gitlab-ci.yml` is structured into three stages:

1. **Standards**
    - Run PHPStan
    - Run Security Checks

2. **Build**
    - Use `buildx` to build & push multi-platform image

3. **Test**
    - Pull the built image
    - Boot via Compose
    - Run a `/up` HTTP check to confirm runtime health

---

## 📦 Composer Requirements

Includes Laravel 11+ stack with Jetstream and modern dev tools like:

- PHPStan
- Enlightn Security Checker
- PestPHP
- Laravel Pint (optional)

To install:
```bash
composer require --dev enlightn/security-checker
composer require --dev phpstan/phpstan
composer require --dev laravel/pint
```

---

## 📥 Environment Variables

Customize `.env.testing` and use it with CI jobs. You can adjust `APP_COMPOSE_SERVICE` to point to the right container for health checks.

---

## 🐳 Using Docker-in-Docker (DinD) in GitLab CI

If you're using a Docker-based GitLab Runner and need to run Docker commands (e.g., to build or run containers within your CI pipeline), you'll need to enable **Docker-in-Docker** (DinD).

### 🔧 Step 1: Enable Privileged Mode

Edit the GitLab Runner configuration:

```bash
sudo nano /etc/gitlab-runner/config.toml
```

In the relevant runner section, set:

```toml
privileged = true
```

> ⚠️ Note: Privileged mode is required to run the Docker daemon inside the runner container.

### 📄 Step 2: Define Variables and CI Job

Update your `.gitlab-ci.yml`:

```yaml
variables:
  DOCKER_TLS_CERTDIR: "" # Required to disable TLS and allow DinD

test:
  image: docker:28.1
  stage: test
  services:
    - name: docker:28.1-dind
  script:
    - docker version
```
---

## ✅ Final Tips

- Make sure your GitLab runner is **privileged** (required for `dind`)
- Customize `ci/docker/php/php.ini` and `fpm.conf` for production tuning
- Use `.env.testing` in `ci/testing` for your CI test environment

---

## 💬 Feedback & Contributions

Feel free to open issues or submit PRs to enhance this template. This project is meant to save Laravel devs time and effort when getting started with CI/CD.
