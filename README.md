# 🐳 Docker Lab 1: The Friend-to-Friend Image Exchange

**Course**: Artificial Intelligence (Machine Learning; Deep Learning; Communication) — Batch IV  
**Lab Assignment**: MLOps Weekly Labs — Lab03_Containerization (Docker)  
**Student Name**: Seerat Fatima  
**Student ID**: 3840392051012  
**Docker Hub Repository**: [hub.docker.com/r/seeratfatima02/secret-msg](https://hub.docker.com/r/seeratfatima02/secret-msg)  
**Lab Partner**: Syeda (`jnsyeda`)

---

## 📌 Executive Summary & Objectives
This lab demonstrates the foundational principles of containerization and proves how Docker eliminates the *"it works on my machine"* problem. In this project:
- A Python micro-application was built, frozen into a portable container image, and published to Docker Hub.
- Cross-platform compatibility was solved between Apple Silicon (Mac `arm64`) and Windows/Intel (`amd64`) using multi-architecture builds.
- A live, two-way image exchange was completed with a partner (`jnsyeda`).
- Multi-container orchestration, persistent volumes, custom networking, and remote SSH container execution were implemented and verified.

---

## 🛠️ Code Implementation

### 1. Application Code (`app.py`)
Implements the **12-Factor App pattern** where configuration lives outside the code:
```python
import os

message = os.getenv('SECRET_MSG', 'No secret message found!')
print('\n' + '='*30)
print(f'MESSAGE FROM A FRIEND: {message}')
print('='*30 + '\n')
```

### 2. Dockerfile (`Dockerfile`)
Uses a minimal Debian base to minimize image attack surface and transfer size:
```dockerfile
# Base image: Python 3.9 slim (~120MB)
FROM python:3.9-slim

# Copy application file into container filesystem
COPY app.py /app.py

# Execution command when container starts
CMD ["python", "/app.py"]
```

### 3. Docker Compose Configuration (`docker-compose.yml`)
Declarative orchestration for building and running with runtime environment variables:
```yaml
version: '3.9'

services:
  app:
    build: .
    environment:
      - SECRET_MSG=Hello from Compose!
```

---

## 🚀 Lab Modules & Execution Summary

### Module 1: Environment Setup
- Installed and configured Docker Desktop on macOS.
- Verified Docker daemon connectivity and architecture:
```bash
docker --version
docker run hello-world
docker images
```

### Module 2 & 3: Building & Cross-Platform Architecture
- Addressed the **Apple Silicon Architecture Trap**: Because M-series Macs run `arm64`, building with default settings can fail on Windows/Linux x86 machines with `exec format error`.
- Built and pushed a **Multi-Architecture Image** supporting both `linux/amd64` and `linux/arm64`:
```bash
docker buildx build --platform linux/amd64,linux/arm64 -t seeratfatima02/secret-msg:v1 --push .
```

### Module 4: Cloud Registry Publishing
- Authenticated with Docker Hub using `docker login`.
- Verified layer deduplication, manifest indexing, and verified retrieval via clean pull:
```bash
docker pull seeratfatima02/secret-msg:v1
```

### Module 5: The Friend-to-Friend Exchange
- Partnered with `jnsyeda` (running on Windows).
- Pulled and executed partner images with custom injected messages:
```bash
docker run --platform linux/amd64 --rm -e SECRET_MSG='Hey Syeda, pizza is on me!' jnsyeda/secret-msg:v1
```
- Tested default fallback without `-e` flag (`MESSAGE FROM A FRIEND: No secret message found!`).
- Inspected container internal filesystem using interactive shell override (`/bin/sh`).

### Module 6: Container & Image Lifecycle Management
- Inspected metadata JSON and layer history:
```bash
docker inspect seeratfatima02/secret-msg:v1
docker history seeratfatima02/secret-msg:v1
```
- Practiced container and image deletion (`docker rm`, force-removal `docker rm -f`, and mass cleanup with `docker container prune`).

### Module 7: Container Debugging Techniques
- Evaluated stdout/stderr logs in detached mode with timestamps:
```bash
docker logs -t myapp
docker logs --tail 10 myapp
```
- Explored running containers using process execution (`docker exec -it debug_box /bin/sh`).

### Module 8: Docker Networking & Port Mapping
- Created user-defined bridge network `mynet` enabling DNS container-name resolution:
```bash
docker network create mynet
```
- Tested host-to-container port binding (`-p 8080:80`) with an Nginx web server and verified via `curl http://localhost:8080`.

### Module 9: Docker Compose & Data Persistence
- Executed stack using `docker compose up` and tore down resources with `docker compose down`.
- **Named Volumes**: Demonstrated data survival across container deletion:
```bash
docker run --rm -v mydata:/data python:3.9-slim sh -c "echo 'Persistence works!' > /data/message.txt"
docker run --rm -v mydata:/data python:3.9-slim cat /data/message.txt
```
- **Bind Mounts**: Linked local macOS directory directly into a Linux container (`-v $(pwd):/workspace`).

### Module 10: Storage Hygiene & Optimization
- Analyzed disk usage breakdown across images, containers, and volumes with `docker system df`.
- Cleaned intermediate layer build cache using `docker builder prune`.

### Module 11: Remote SSH Container Execution
- Established an SSH session from macOS to a partner's Windows machine over the university Wi-Fi subnet:
```bash
ssh cp@192.168.0.178
```
- Remotely triggered `docker pull` and executed the containerized message application directly on the partner's hardware.

---

## 💡 Discussion Checkpoints (Module 5.2)

1. **Did you install Python on your machine to run your partner's code?**  
   *Answer*: No. The Python 3.9 runtime was packaged directly into the container image layers. Docker executes the code inside an isolated namespace without touching or depending on the host machine's Python installation.

2. **Did your partner's container modify any file on your laptop?**  
   *Answer*: No. The container runs inside an isolated filesystem namespace. Any writes occurred strictly inside the container's disposable writable layer, which was destroyed upon exit via the `--rm` flag.

3. **What if you had a database password — how would you pass it to a real app?**  
   *Answer*: Passwords and sensitive credentials must never be hardcoded into a Dockerfile. They should be injected at container startup using environment variables (`-e DB_PASSWORD=...`) managed securely through a secrets management service (such as AWS Secrets Manager, Azure Key Vault, or Docker Secrets).
