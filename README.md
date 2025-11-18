
# Dockerized Flask App with GitHub Actions CI/CD

A complete, step-by-step guide to containerize a simple Flask application, build its Docker image, and push the image to Docker Hub automatically using GitHub Actions.

---

## Project Structure

```

Project-1/
├── app.py
├── Dockerfile
├── requirements.txt
└── .github/
└── workflows/
└── docker-ci-cd.yml

````

---

## 1. Overview

This repository contains:

- A Flask application (`app.py`)
- A Dockerfile for containerizing the app
- A GitHub Actions workflow for automating image builds and pushing to Docker Hub

Whenever you push to the `main` branch, GitHub Actions builds the Docker image and pushes it to your Docker Hub repository.

---

## 2. File Contents

### app.py

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Hello Bubu! Your Flask app is running successfully!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=3000)
````

### requirements.txt

```
flask==3.0.0
```

### Dockerfile

```dockerfile
FROM python:3.10-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 3000

CMD ["python", "app.py"]
```

### GitHub Actions Workflow

`.github/workflows/docker-ci-cd.yml`

```yaml
name: Docker CI/CD Pipeline

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Source Code
        uses: actions/checkout@v3

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build Docker Image
        run: |
          docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/flask-app:latest .

      - name: Push Docker Image to Docker Hub
        run: |
          docker push ${{ secrets.DOCKERHUB_USERNAME }}/flask-app:latest
```

---

## 3. Run the Project Locally (Docker)

### Build image

```bash
docker build -t local-flask-app:latest .
```

### Run container

```bash
docker run --rm -p 3000:3000 local-flask-app:latest
```

### Test in browser

```
http://localhost:3000
```

---

## 4. Create Docker Hub Repository

1. Go to: [https://hub.docker.com/](https://hub.docker.com/)
2. Click **Create Repository**
3. Name it: `flask-app`
4. Visibility: **Public** (recommended)

Your image path:

```
docker.io/<your-username>/flask-app:latest
```

---

## 5. Generate Docker Hub PAT (Access Token)

1. Docker Hub → Account Settings → Security
2. Click **New Access Token**
3. Name it
4. Permission: **Read & Write**
5. Copy the token and save it

---

## 6. Push Local Code to GitHub

```bash
git init
git add .
git commit -m "Initial CI/CD setup"
git branch -M main
git remote add origin https://github.com/<your-username>/GitHub-Action-Docker-Project.git
git push -u origin main
```

---

## 7. Add GitHub Secrets

Go to:

```
GitHub Repo → Settings → Secrets → Actions
```

Add:

| Name               | Value                    |
| ------------------ | ------------------------ |
| DOCKERHUB_USERNAME | Your Docker Hub username |
| DOCKERHUB_TOKEN    | Docker Hub PAT           |

---

## 8. How the GitHub Actions Workflow Works

1. Detects push to `main`
2. Checks out your code
3. Logs into Docker Hub
4. Builds Docker image
5. Pushes image to:

```
docker.io/<username>/flask-app:latest
```

---

## 9. Troubleshooting

| Error                | Cause                           | Fix                              |
| -------------------- | ------------------------------- | -------------------------------- |
| 401 Unauthorized     | Wrong PAT or permission         | Regenerate PAT with Read & Write |
| Repo not found       | DockerHub repo missing          | Create repo with correct name    |
| Workflow not running | Wrong folder path               | Must be `.github/workflows/...`  |
| Build failed         | Missing files or bad Dockerfile | Verify file names & paths        |

---

## 10. Recommended Improvements

* Add version tags (`sha-<commit>`)
* Multi-stage Dockerfile
* Kubernetes deployment
* Deploy to AWS EC2/ECR
* Add test steps before build

---

## License

This project is for learning and demonstration purposes.

```

---

If you want this README with **images**, **badges**, or **architecture diagrams**, just say the word — I’ll generate it exactly like the screenshot you showed.
```
