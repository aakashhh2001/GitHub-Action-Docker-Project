Below is a complete `README.md` you can copy-paste directly into your repository. It is detailed, well-structured, and contains everything needed for the project structure you specified.

---

```markdown
# Dockerized Flask App with GitHub Actions CI/CD

A complete, step-by-step guide to containerize a simple Flask application, build the image in CI, and push the image to Docker Hub using GitHub Actions.

---

## Project structure

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

## Contents of this README

1. Purpose & overview  
2. File contents (copy-paste ready)  
   - `app.py`  
   - `requirements.txt`  
   - `Dockerfile`  
   - GitHub Actions workflow (`.github/workflows/docker-ci-cd.yml`)  
3. How to run & test locally (Docker)  
4. Create Docker Hub repository & PAT (permissions)  
5. Push code to GitHub and configure repository secrets  
6. How the GitHub Actions workflow works (explain)  
7. Troubleshooting & common errors  
8. Optional improvements and next steps

---

## 1. Purpose & overview

This repository demonstrates:

- A minimal Flask application (`app.py`) listening on port `3000`.  
- A `Dockerfile` that containerizes the app.  
- A GitHub Actions workflow that builds the Docker image and pushes it to Docker Hub whenever you push to `main`.

---

## 2. File contents (copy-paste)

### `app.py`

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Hello Bubu! Your Flask app is running successfully!"

if __name__ == "__main__":
    # Listen on all interfaces inside the container and port 3000
    app.run(host="0.0.0.0", port=3000)
````

### `requirements.txt`

```
flask==3.0.0
```

> If your file is `requirements.txt.txt`, rename it:
>
> ```bash
> mv requirements.txt.txt requirements.txt
> ```

### `Dockerfile`

```dockerfile
# Use a small Python image
FROM python:3.10-slim

# Create and switch to app directory
WORKDIR /app

# Copy dependency list first to leverage Docker layer caching
COPY requirements.txt .

# Install dependencies without pip cache to keep the image small
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Document the port the app listens on
EXPOSE 3000

# Default command to run the application
CMD ["python", "app.py"]
```

### GitHub Actions workflow: `.github/workflows/docker-ci-cd.yml`

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

> Note: The workflow uses two GitHub repository secrets:
>
> * `DOCKERHUB_USERNAME` — your Docker Hub username
> * `DOCKERHUB_TOKEN` — Docker Hub Personal Access Token (PAT) with **Read & Write** permission

---

## 3. Build & run locally (quick verification)

**Build the local Docker image**

```bash
# From project root
docker build -t local-flask-app:latest .
```

**Run the container and forward port 3000**

```bash
docker run --rm -p 3000:3000 local-flask-app:latest
```

**Test**

Open a browser or use `curl`:

```bash
curl http://localhost:3000
# Expected output:
# Hello Bubu! Your Flask app is running successfully!
```

When done, stop the container with `Ctrl+C` (if run in foreground) or `docker ps` + `docker stop <container-id>`.

---

## 4. Docker Hub repository & PAT (Personal Access Token)

1. Create a repository on Docker Hub:

   * URL: [https://hub.docker.com/](https://hub.docker.com/)
   * Click **Repositories → Create Repository**
   * Name: `flask-app` (or a name of your choice)
   * Visibility: `Public` (recommended for learning) or `Private`

2. Create a PAT (recommended over password):

   * Docker Hub → Account Settings → Security → New Access Token
   * Name the token (e.g., `github-actions-token`)
   * **Permissions:** select **Read & Write** — this allows pushing images.
   * Copy and securely store the token — you cannot view it again after creation.

Your final image reference will be:

```
docker.io/<dockerhub-username>/flask-app:latest
```

---

## 5. Push code to GitHub & add secrets

**Create GitHub repo** (example name: `GitHub-Action-Docker-Project`) and push the project:

```bash
cd ~/Documents/GitHub_Actions/Project-1

git init
git add .
git commit -m "Initial commit: Flask app + Dockerfile + CI workflow"
git branch -M main
git remote add origin https://github.com/<your-github-username>/GitHub-Action-Docker-Project.git
git push -u origin main
```

**Add repository secrets (GitHub UI)**

Go to:
`Repository → Settings → Secrets and variables → Actions → New repository secret`

Create two secrets:

| Name                 | Value                             |
| -------------------- | --------------------------------- |
| `DOCKERHUB_USERNAME` | your Docker Hub username          |
| `DOCKERHUB_TOKEN`    | the Docker Hub PAT (Read & Write) |

The workflow uses these values for `docker/login-action` to authenticate.

---

## 6. How the GitHub Actions workflow runs (step-by-step)

1. **Trigger**: on `push` to `main` or on PR to `main`.
2. **Checkout**: `actions/checkout` fetches your repository.
3. **Login**: `docker/login-action` logs into Docker Hub using secrets.
4. **Build**: `docker build` command builds the image using your `Dockerfile`.
5. **Push**: `docker push` sends the image to Docker Hub at `<username>/flask-app:latest`.

Once the workflow completes successfully, check Docker Hub under your repository tags for `latest`.

---

## 7. Troubleshooting (common problems & fixes)

| Symptom                               | Likely cause                                      | Fix                                                                                      |
| ------------------------------------- | ------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| `401 Unauthorized` on push            | Wrong username or PAT; PAT lacks write permission | Regenerate PAT with **Read & Write** permissions; update GitHub secret `DOCKERHUB_TOKEN` |
| `repository does not exist`           | Docker Hub repo missing or namespace mismatch     | Create repo in Docker Hub or confirm username/repo name                                  |
| Workflow not running                  | Workflow file misplaced or branch mismatch        | Ensure file is at `.github/workflows/docker-ci-cd.yml` and you push to `main`            |
| Build fails                           | Dockerfile syntax error or missing files          | Check Dockerfile lines and confirm `requirements.txt` exists                             |
| Pull rate-limit errors for base image | Docker Hub rate limit                             | Authenticate or use alternative base image registry; retry later                         |

**How to inspect logs**: GitHub → Actions → select the workflow run → view logs for each step.

---

## 8. Optional improvements (recommended next steps)

* Tag images with commit SHA and semantic version tags instead of only `latest`:

  * tag as `latest` and also `:sha-<short-sha>` to allow rollbacks
* Use `docker/build-push-action` to leverage buildx, caching, and multi-platform builds
* Add a test job to run unit tests/linters before building the image
* Deploy automatically from Docker Hub to:

  * Kubernetes (EKS/GKE/AKS)
  * A VM (using SSH or Ansible)
* Use multi-stage builds for larger apps to reduce image size
* Implement image signing (cosign) and vulnerability scanning

---

## 9. .gitignore (recommended)

Create a `.gitignore` file to avoid committing local artifacts:

```
__pycache__/
*.pyc
*.pyo
*.pyd
.env
venv/
.env.*
.DS_Store
docker-compose.override.yml
```

---

## 10. Checklist before you push

* [ ] `app.py` has the correct content and runs locally
* [ ] `requirements.txt` exists and has `flask==3.0.0`
* [ ] `Dockerfile` is in project root and correct
* [ ] `.github/workflows/docker-ci-cd.yml` exists and committed
* [ ] Docker Hub repo created (`flask-app` or your chosen name)
* [ ] Docker Hub PAT created with **Read & Write** permission
* [ ] GitHub secrets `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN` added
* [ ] You pushed to `main` to trigger the workflow

---

## 11. Quick references

**Rebuild & run locally**

```bash
docker build -t local-flask-app:latest .
docker run --rm -p 3000:3000 local-flask-app:latest
```

**Remove dangling images**

```bash
docker system prune -f
```

**View running containers**

```bash
docker ps
```

---

## License

This project is provided for learning and demonstration purposes. Use freely.

---

If you want, I can:

* Create a version of this README with badges (build status / docker hub)
* Add commit-SHA tagging to the workflow and provide the exact YAML changes
* Generate a small architecture diagram SVG you can embed in the README

Tell me which enhancement you want next and I will produce the exact files/changes.
