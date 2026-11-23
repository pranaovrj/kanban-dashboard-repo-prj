 Project Overview
This project demonstrates the implementation of an automated Continuous Integration and Continuous Deployment (CI/CD) pipeline for a Kanban Dashboard web application using GitHub, Jenkins, Docker, Docker Hub, and AWS EC2.
The primary objective is to automate the application deployment process, starting from source code management in GitHub to containerized deployment on an AWS EC2 instance.
The project uses two separate EC2 instances to isolate the CI/CD environment from the application hosting environment.
Key Objectives
Automate application build and deployment using Jenkins.
Integrate GitHub with Jenkins through webhooks.
Containerize the application using Docker.
Implement Docker image versioning and registry management.
Deploy the application on AWS EC2.
Apply Docker container security best practices.
Implement health checks, logging, monitoring, and resource limits.
Key Features
Automated CI/CD pipeline.
Multi-stage Docker build.
Non-root container execution.
Docker Hub image publishing.
Automated deployment workflow.
Container health monitoring.
Resource utilization limits.









Components
Component
Responsibility
GitHub
Source code management
GitHub Webhook
Triggers Jenkins automatically
Jenkins EC2
CI/CD pipeline execution
Docker
Application containerization
Docker Hub
Docker image registry
Application EC2
Runs the deployed container
Nginx
Serves the production application
Docker HEALTHCHECK
Container health validation

Deployment Flow
Developer pushes code to GitHub.
GitHub Webhook triggers Jenkins.
Jenkins retrieves the latest source code.
Jenkins builds the Docker image.
The image is tagged with a version.
Jenkins authenticates with Docker Hub and pushes the image.
The application EC2 instance pulls the image.
The previous application container is replaced with the new deployment.
Docker health checks and application validation confirm deployment status.
Users access the application through the EC2 public IP and application port.


Technologies Used
Technology
Purpose
AWS EC2
Cloud infrastructure
Ubuntu Linux
Operating system
Git
Version control
GitHub
Source code repository
GitHub Webhooks
Automated pipeline triggering
Jenkins
CI/CD automation
Docker
Containerization
Docker Hub
Container image registry
Nginx
Production web server
React
Frontend framework
TypeScript
Application development
Vite
Frontend build tool
Linux Shell
Server administration



Architecture
<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/10757cdf-23c8-4c47-a63f-b9c5929183f0" />
