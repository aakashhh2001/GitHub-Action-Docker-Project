Purpose: Build a Docker image for a small Flask app, push the image to Docker Hub using GitHub Actions, and keep an automated CI/CD pipeline that rebuilds & pushes on every push to main.
Project structure
Project-1/
  ├── app.py
  ├── Dockerfile
  ├── requirements.txt
  └── .github/
        └── workflows/
              └── docker-ci-cd.yml
