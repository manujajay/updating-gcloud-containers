# Automating Build and Deployment with Google Cloud Run and GitHub

This README provides detailed instructions on setting up a CI/CD pipeline for automatically building and deploying containers to Google Cloud Run when changes are pushed to a GitHub repository.

## 1. Prerequisites

- Google Cloud account with billing enabled.
- GitHub account.
- Google Cloud SDK installed and initialized.

## 2. Connect GitHub with Google Cloud Build

### Set Up the GitHub Repository

1. Push your code to a GitHub repository.
2. Ensure your repository contains a `Dockerfile` and your application code (e.g., `app.py`).

### Enable Google Cloud Build

1. Go to the Google Cloud Console.
2. Navigate to Cloud Build and enable the API.
3. Set up a trigger by connecting to your GitHub repository.

## 3. Creating the Dockerfile

Your `Dockerfile` should be set up to install dependencies and run your application. Here's an example for a Python Flask app:

```Dockerfile
# Use an official Python runtime as a parent image
FROM python:3.8-slim

# Set the working directory in the container
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Make port 8080 available to the world outside this container
EXPOSE 8080

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

## 4. Setting up the cloudbuild.yaml

Create a `cloudbuild.yaml` file in your repository to define the build steps and specify the deployment to Google Cloud Run:

```yaml
steps:
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'gcr.io/$PROJECT_ID/my-app:$COMMIT_SHA', '.']
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'gcr.io/$PROJECT_ID/my-app:$COMMIT_SHA']
  - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
    entrypoint: 'gcloud'
    args:
      - 'run'
      - 'deploy'
      - 'my-app'
      - '--image'
      - 'gcr.io/$PROJECT_ID/my-app:$COMMIT_SHA'
      - '--region'
      - 'us-central1'
      - '--platform'
      - 'managed'
      - '--allow-unauthenticated'
```

## 5. Configuring Deployment on Google Cloud Run

When you push changes to your GitHub repository, the Cloud Build trigger you set up will automatically start a new build, push the built image to Google Cloud Container Registry, and deploy it to Google Cloud Run.

## Conclusion

This setup allows you to fully automate the process of building and deploying your Python applications to Google Cloud Run through GitHub commits, simplifying your development and deployment workflow.
