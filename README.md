# Java CI/CD Pipeline: Maven, SonarQube, and AWS ECS

This repository contains a declarative Jenkins pipeline designed to automate the build, testing, quality analysis, and deployment of a Java application.

## 🚀 Pipeline Lifecycle

The pipeline consists of the following automated stages:

* **Source:** Fetches the latest code from GitHub.
* **Build:** Compiles the application using **Maven 3.9.9** and **JDK 17**.
* **Test:** Executes unit tests and performs Checkstyle analysis.
* **Quality Gate:** Integrates with **SonarQube** to ensure code meets quality standards.
* **Containerize:** Builds a Docker image using a multi-stage Dockerfile.
* **Registry:** Pushes the versioned image to **Amazon ECR**.
* **Deploy:** Triggers a rolling update to an **Amazon ECS** service.

---

## 🛠 Prerequisites

Ensure the following tools are configured in your Jenkins **Global Tool Configuration**:

| Tool | Name in Jenkins |
| :--- | :--- |
| **Maven** | `Maven-3.9.9` |
| **JDK** | `JDK17` |
| **SonarQube Scanner** | `sonar6.2` |

### Required Credentials
You must add the following credentials to your Jenkins store:
1.  **AWS Credentials:** (e.g., `awscreds`) for pushing to ECR and updating ECS.
2.  **SonarQube Server:** A configured server instance named `sonarserver`.

---

## ⚙️ Configuration Guide

Before running the pipeline, modify the `environment` block in the `Jenkinsfile` with your specific AWS details:

```groovy
environment {
    registryCredential = 'your-aws-credentials-id'
    imageName          = '[123456789012.dkr.ecr.us-east-1.amazonaws.com/your-app-repo](https://123456789012.dkr.ecr.us-east-1.amazonaws.com/your-app-repo)'
    Name_of_the_ECR    = '[https://123456789012.dkr.ecr.us-east-1.amazonaws.com](https://123456789012.dkr.ecr.us-east-1.amazonaws.com)'
    cluster            = 'your-ecs-cluster-name'
    service            = 'your-ecs-service-name'
}