# DevOps CI/CD Pipeline Implementation

[![Build Status](https://img.shields.io/badge/build-passing-brightgreen.svg)](https://jenkins.io/)
[![Jenkins](https://img.shields.io/badge/Jenkins-2.4+-blue.svg)](https://jenkins.io/)
[![Node.js](https://img.shields.io/badge/Node.js-18+-green.svg)](https://nodejs.org/)
[![MongoDB](https://img.shields.io/badge/MongoDB-Atlas-green.svg)](https://www.mongodb.com/cloud/atlas)

## Overview

A production-ready CI/CD pipeline implementation using Jenkins for Node.js applications with MongoDB integration. Features automated testing, secure credential management, and comprehensive monitoring.

## Architecture

```
GitHub Repository → Webhook → Jenkins Pipeline → Testing → Render Deployment
                                    ↓
                            Slack/Email Notifications
```

## Technology Stack

| Component | Technology | Purpose |
|-----------|------------|---------|
| **CI/CD** | Jenkins | Pipeline orchestration |
| **Tunneling** | ngrok | Webhook tunneling |
| **Database** | MongoDB Atlas | Cloud database |
| **Deployment** | Render | Application hosting |
| **Testing** | Mocha | Test framework |
| **Notifications** | Slack API | Team communication |
| **Containerization** | Docker | Jenkins containerization |

## Prerequisites

- Docker (v20.10+)
- GitHub Account
- MongoDB Atlas Account
- Render Account
- Slack Workspace
- ngrok

## Quick Start

1. **Clone Repository**
   ```bash
   git clone https://github.com/d-0pZ/gallery.git
   cd gallery
   ```

2. **Start Jenkins Container**
   ```bash
   docker run -d \
     --name qays \
     -p 8080:8080 \
     -p 50000:50000 \
     -v jenkins_home:/var/jenkins_home \
     jenkins/jenkins:lts
   ```

3. **Access Jenkins**
   - Open http://localhost:8080
   - Get initial password: `docker logs qays`

## Implementation Guide

### 1. Jenkins Setup

**Required Plugins:**
- GitHub Integration Plugin
- Slack Notification Plugin
- NodeJS Plugin

**Global Tool Configuration:**
- NodeJS: Name = 'NodeJS-Latest', Version = Latest LTS

### 2. Credential Management

**Jenkins Credentials (Manage Jenkins → Credentials):**
```
ID: mongodb-uri
Type: Secret text
Secret: [Base64 encoded MongoDB URI]

ID: slack-webhook
Type: Secret text
Secret: [Slack Webhook URL]
```

### 3. Database Configuration

**MongoDB Atlas Setup:**
- Create cluster and database user
- Configure network access
- Generate connection string

**Environment Variables (.env):**
```env
NODE_ENV=development
PORT=3000
MONGODB_URI=[Base64 encoded URI]
MONGODB_URI_TEST=mongodb://localhost:27017/darkroom-test
```

### 4. Webhook Configuration

**Start ngrok tunnel:**
```bash
nohup ngrok http 8080 &
curl http://localhost:4040/api/tunnels  # Get public URL
```

**GitHub Webhook:**
- URL: `https://your-ngrok-url.ngrok.io/github-webhook/`
- Content-type: application/json
- Events: Push events

### 5. Pipeline Configuration

**Create Jenkins Pipeline:**
- Name: qays-gallery-pipeline
- Type: Pipeline
- Build trigger: GitHub hook trigger for GITScm polling
- Pipeline: Script from SCM

## Pipeline Stages

1. **Checkout** - Clone repository
2. **Install Dependencies** - `npm ci --production=false`
3. **Security Audit** - `npm audit --audit-level=moderate`
4. **Test** - `npm test`
5. **Deploy** - Health check verification
6. **Notify** - Slack/Email notifications

## Security Features

- Credential masking in console output
- Base64 encoded database URIs
- Jenkins credential store integration
- Automated security vulnerability scanning
- Environment-specific database isolation

## Monitoring

**Slack Integration:**
- Build success/failure notifications
- Deployment status updates
- Build information and links

**Email Notifications:**
- Test failure alerts
- Deployment failure notifications
- Pipeline failure reports

## Render Deployment

**Auto-deployment configuration:**
- Connected to GitHub repository
- Environment variables set in Render dashboard
- Automatic builds on git push

## Troubleshooting

**Common Issues:**

1. **Jenkins Connection**
   ```bash
   docker ps | grep qays
   docker logs qays
   docker restart qays
   ```

2. **Webhook Issues**
   - Verify ngrok URL is active
   - Check GitHub webhook delivery status
   - Validate Jenkins configuration

3. **Deployment Failures**
   - Check Render build logs
   - Verify environment variables
   - Validate database connectivity

## File Structure

```
gallery/
├── _config.js          # Database configuration
├── .env               # Environment variables
├── Jenkinsfile        # CI/CD pipeline definition
├── package.json       # Project dependencies
├── server.js          # Application entry point
├── test/              # Test files
│   └── serverTest.js
└── README.md          # This file
```

## Performance Optimizations

- npm cache for faster dependency installation
- `npm ci` for consistent builds
- Silent installation flags
- Build log cleanup
- Credential sanitization

## Support

For issues and questions:
- Email: iqra2.ali@proton.me
- Repository: https://github.com/d-0pZ/gallery