# Calc App Pipeline 🚀

This project implements a complete **CI/CD Pipeline** for a Calculator application. It showcases modern DevOps practices, including automated testing, containerization, and continuous integration.

## 📋 Table of Contents
* [Overview](#overview)
* [Tech Stack](#tech-stack)
* [Pipeline Architecture](#pipeline-architecture)
* [Getting Started](#getting-started)
* [Project Structure](#project-structure)

---

## 🔍 Overview
The **Calc App Pipeline** is designed to automate the development lifecycle. By using a "Pipeline-as-Code" approach, every commit is automatically built, tested, and prepared for deployment, ensuring high code quality and fast delivery cycles.

## 🛠 Tech Stack
* **Source Control:** Git & GitHub
* **CI/CD Tool:** Jenkins (via Jenkinsfile)
* **Environment:** Docker
* **Application:** Python / Node.js (Adjust based on your specific logic)
* **Container Registry:** Docker Hub

---

## 🏗 Pipeline Architecture
The Jenkins pipeline consists of the following automated stages:

1.  **Code Checkout:** Clones the repository from GitHub.
2.  **Linting/Static Analysis:** Ensures the code follows best practices.
3.  **Unit Testing:** Executes automated tests to verify calculator functions.
4.  **Docker Build:** Packages the application into a lightweight Docker image.
5.  **Security Scan:** (Optional) Scans the image for vulnerabilities.
6.  **Push to Hub:** Uploads the tagged image to Docker Hub.
7.  **Cleanup:** Removes temporary files and images from the build agent.

---

## 🚀 Getting Started

### Prerequisites
* [Docker](https://www.docker.com/) installed locally.
* [Jenkins](https://www.jenkins.io/) server configured with Docker pipeline plugins.

### Installation & Local Run
To test the application on your local machine:

1. **Clone the repo:**
   ```bash
   git clone [https://github.com/alondudi/calc-app-pipeline.git](https://github.com/alondudi/calc-app-pipeline.git)
   cd calc-app-pipeline
