
# CST8919 Assignment 1: Securing and Monitoring an Authenticated Flask App

## Overview
This project implements a secure Flask web application with **Auth0 authentication** and **Azure monitoring**.  
It combines user login/logout functionality, structured logging, threat detection via KQL, and cloud deployment on Azure.

---

## Features
- **Auth0 SSO Integration:** Secure user authentication using Auth0  
- **Structured Logging:** JSON logs for Azure Monitor / Log Analytics  
- **Activity Monitoring:** Logs user logins, profile access, and suspicious activity  
- **Threat Detection:** KQL queries for brute force detection  
- **Azure Alerts:** Email alerts on detection of suspicious behavior  
- **Cloud Deployment:** Deployed to Azure App Service  

---

## Project Structure
```
secure-flask-auth0/
├── app.py               # Flask app with Auth0 integration
├── requirements.txt      # Python dependencies
├── templates/            # HTML templates
│   ├── base.html
│   ├── index.html
│   └── profile.html
├── test-app.http         # REST Client test file (optional)
├── env.example           # Template for environment variables
└── README.md             # This file
```

---

## Setup Instructions

### 1️⃣ Local Development

Clone the repository:
```bash
git clone https://github.com/mspanwar21/secure-flask-auth0
cd secure-flask-auth0
```

Create a virtual environment:
```bash
python3 -m venv venv
source venv/bin/activate   # On Windows: venv\Scripts\activate
```

Install dependencies:
```bash
pip install -r requirements.txt
```

Configure environment variables:
```bash
cp env.example .env
# Edit .env with your Auth0 credentials and secret key
```

Run the application:
```bash
flask --app app run
```
App runs at: [http://localhost:5000](http://localhost:5000)

---

### 2️⃣ Auth0 Configuration

- Create an Auth0 Regular Web Application.
- Configure:
  - **Allowed Callback URLs:**  
    ```
    http://localhost:5000/callback
    ```
  - **Allowed Logout URLs:**  
    ```
    http://localhost:5000/
    ```
  - **Allowed Web Origins:**  
    ```
    http://localhost:5000
    ```
- Update your `.env` file:
  ```
  AUTH0_DOMAIN=your-tenant.auth0.com
  AUTH0_CLIENT_ID=your-client-id
  AUTH0_CLIENT_SECRET=your-client-secret
  APP_SECRET_KEY=your-secret-key
  ```

---

### 3️⃣ Azure Deployment

Create resources:
```bash
az group create --name flask-auth0-rg --location canadacentral
az appservice plan create --name flask-auth0-plan --resource-group flask-auth0-rg --sku B1
az webapp create --name flask-auth0-app --resource-group flask-auth0-rg --plan flask-auth0-plan --runtime "PYTHON:3.9"
```

Package app:
```bash
zip -r app.zip . -x "venv/*" "*.pyc" "__pycache__/*" ".env" ".git/*"
```

Deploy:
```bash
az webapp deployment source config-zip --resource-group flask-auth0-rg --name flask-auth0-app --src app.zip
```

Configure environment variables in Azure → App Service → Configuration → Application settings.

Update Auth0:
- **Allowed Callback URLs:** `https://flask-auth0-app.azurewebsites.net/callback`
- **Allowed Logout URLs:** `https://flask-auth0-app.azurewebsites.net/`

---

## Security Monitoring & Alerting

### 🔍 **KQL Query for suspicious activity**
```kql
AppServiceConsoleLogs
| where TimeGenerated > ago(15m)
| where ResultDescription contains "USER_ACTIVITY"
| where ResultDescription contains "profile_access"
| extend LogData = parse_json(substring(ResultDescription, indexof(ResultDescription, "{")))
| where isnotempty(LogData.user_id)
| summarize AccessCount = count() by 
    user_id = tostring(LogData.user_id),
    email = tostring(LogData.email),
    bin(TimeGenerated, 15m)
| where AccessCount > 10
| project TimeGenerated, user_id, email, AccessCount
| order by AccessCount desc
```

### 📢 **Azure Alert**
- Signal type: Log
- Query: Use KQL above
- Threshold: 0 (any result triggers)
- Frequency: 5 min
- Window: 15 min
- Severity: 3
- Action: Email via action group

---

## Logging Format
```json
{
  "timestamp": "2025-07-04T14:00:00Z",
  "activity_type": "profile_access",
  "user_id": "auth0|abc123",
  "email": "user@example.com",
  "client_ip": "192.168.1.10",
  "user_agent": "Mozilla/5.0...",
  "details": {
    "route": "/profile"
  }
}
```

---

## Links
**YouTube Demo:** [Watch on YouTube]()
