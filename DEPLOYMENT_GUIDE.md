# 🚀 Complete Deployment Guide: GitHub → Docker → Azure

## 📋 Overview
This guide will help you set up automatic deployment of your OAN-Seeker-UI frontend to Azure using GitHub Actions.

**What happens:**
1. You push code to GitHub
2. GitHub Actions automatically builds a Docker image
3. The image is pushed to Docker Hub
4. Azure Container Instances pulls and runs the image

---

## ✅ Prerequisites

Before starting, make sure you have:
- [ ] GitHub account with your repository
- [ ] Docker Hub account (free at https://hub.docker.com)
- [ ] Azure account with active subscription
- [ ] Azure CLI installed (we'll use it for setup)

---

## 🔧 Step 1: Set Up Docker Hub

### 1.1 Create Docker Hub Account
1. Go to https://hub.docker.com
2. Sign up for a free account
3. Remember your username and password

### 1.2 Create a Repository (Optional)
- Docker Hub will automatically create the repository when you first push
- Or manually create one named `oan-seeker-ui`

---

## 🔐 Step 2: Set Up Azure Service Principal

You need to create credentials that GitHub can use to deploy to Azure.

### 2.1 Install Azure CLI
If you don't have it:
- Windows: Download from https://aka.ms/installazurecliwindows
- Or use PowerShell: `winget install -e --id Microsoft.AzureCLI`

### 2.2 Login to Azure
```bash
az login
```

### 2.3 Get Your Subscription ID
```bash
az account show --query id --output tsv
```
**Save this ID - you'll need it!**

### 2.4 Create Service Principal
Replace `YOUR_SUBSCRIPTION_ID` with the ID from above:

```bash
az ad sp create-for-rbac --name "oan-github-actions" --role contributor --scopes /subscriptions/YOUR_SUBSCRIPTION_ID --sdk-auth
```

**IMPORTANT:** This will output JSON credentials. Copy the ENTIRE output - you'll add it to GitHub secrets!

It looks like this:
```json
{
  "clientId": "xxxxx",
  "clientSecret": "xxxxx",
  "subscriptionId": "xxxxx",
  "tenantId": "xxxxx",
  ...
}
```

### 2.5 Create Azure Resource Group
```bash
az group create --name oan-resource-group --location uaenorth
```

---

## 🔑 Step 3: Configure GitHub Secrets

Go to your GitHub repository → Settings → Secrets and variables → Actions → New repository secret

Add these secrets ONE BY ONE:

### 3.1 Docker Hub Credentials
| Secret Name | Value |
|------------|-------|
| `DOCKER_USERNAME` | Your Docker Hub username |
| `DOCKER_PASSWORD` | Your Docker Hub password or access token |

### 3.2 Azure Credentials
| Secret Name | Value |
|------------|-------|
| `AZURE_CREDENTIALS` | The entire JSON output from Step 2.4 |

### 3.3 Backend API URLs
| Secret Name | Value |
|------------|-------|
| `VITE_WEATHER_API_URL` | `http://oan-mock-api.ccbjbrgzg6ewbmec.uaenorth.azurecontainer.io:8000/api/v1/weather/weatherUI` |
| `VITE_SEARCH_API_URL` | `http://oan-mock-api.ccbjbrgzg6ewbmec.uaenorth.azurecontainer.io:8000/api/v1/schemes/search` |
| `VITE_STATES_API_URL` | `http://oan-mock-api.ccbjbrgzg6ewbmec.uaenorth.azurecontainer.io:8000/api/v1/location/states` |
| `VITE_DISTRICTS_API_URL` | `http://oan-mock-api.ccbjbrgzg6ewbmec.uaenorth.azurecontainer.io:8000/api/v1/location/districts` |
| `VITE_AIBOT_API_URL` | `http://oan-ai-api.dhekh2czg2f7exfc.uaenorth.azurecontainer.io:8000/api/v1/chat` |
| `VITE_TRANSCRIBE_API_URL` | `http://oan-ai-api.dhekh2czg2f7exfc.uaenorth.azurecontainer.io:8000/api/v1/transcribe` |
| `VITE_TTS_API_URL` | `http://oan-ai-api.dhekh2czg2f7exfc.uaenorth.azurecontainer.io:8000/api/v1/tts` |

---

## 📤 Step 4: Push to GitHub

Now that everything is configured, push your code:

```bash
git add .
git commit -m "Add Docker and GitHub Actions deployment"
git push origin main
```

**Note:** If your main branch is called `master`, change `main` to `master` in the workflow file.

---

## 👀 Step 5: Monitor Deployment

1. Go to your GitHub repository
2. Click on "Actions" tab
3. You should see a workflow running
4. Click on it to see the progress
5. Wait for all steps to complete (usually 5-10 minutes)

---

## 🎉 Step 6: Access Your Application

After deployment completes:

1. Go to Azure Portal: https://portal.azure.com
2. Search for "Container Instances"
3. Find `oan-seeker-ui`
4. Copy the FQDN (Fully Qualified Domain Name)
5. Open it in your browser!

Or use Azure CLI:
```bash
az container show --resource-group oan-resource-group --name oan-seeker-ui --query ipAddress.fqdn --output tsv
```

---

## 🔍 Troubleshooting

### Build Fails
- Check GitHub Actions logs for specific errors
- Verify all secrets are set correctly
- Make sure Docker Hub credentials are correct

### Deployment Fails
- Verify Azure credentials are correct
- Check if resource group exists
- Ensure you have permissions in Azure subscription

### Application Not Working
- Check if backend APIs are accessible
- Verify environment variables in GitHub secrets
- Check browser console for errors

### Container Won't Start
- Check Azure Container Instances logs:
  ```bash
  az container logs --resource-group oan-resource-group --name oan-seeker-ui
  ```

---

## 🔄 Making Updates

After the initial setup, deploying updates is easy:

1. Make your code changes
2. Commit and push to GitHub:
   ```bash
   git add .
   git commit -m "Your update message"
   git push origin main
   ```
3. GitHub Actions will automatically build and deploy!

---

## 💡 Tips

1. **Use HTTPS for Production**: The backend URLs use HTTP. For production, use HTTPS.
2. **Custom Domain**: You can configure a custom domain in Azure.
3. **Environment-Specific Deployments**: Create separate workflows for dev/staging/prod.
4. **Monitor Costs**: Azure Container Instances charges by the second. Monitor your usage.

---

## 📞 Need Help?

If you encounter issues:
1. Check GitHub Actions logs
2. Check Azure Container logs
3. Verify all secrets are set correctly
4. Make sure backend APIs are running

---

## 🎯 Next Steps

1. ✅ Set up HTTPS/SSL certificates
2. ✅ Configure custom domain
3. ✅ Set up monitoring and alerts
4. ✅ Implement staging environment
5. ✅ Add automated tests to the workflow
