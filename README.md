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
```


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


