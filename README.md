# Google-Cloud-Sales-Credential
GCP Sales Authentication Demo: This project provides a practical guide and code repository for authenticating to Google Cloud Platform (GCP) for sales-related applications and workflows. The repository demonstrates three common and secure methods for authenticating to GCP, each with a corresponding Python script.
- <b>ACTIONS WORKFLOW:</b>


name: Deploy to Google Cloud

on:


  push:

  
    branches:
      - main

permissions:


  contents: 'read'
  id-token: 'write' # This is required for Workload Identity Federation

jobs:
  auth-and-run:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'

      - name: Install dependencies
        run: pip install -r src/requirements.txt

      - name: Authenticate to Google Cloud
        id: 'auth'
        uses: 'google-github-actions/auth@v2'
        with:
          workload_identity_provider: 'projects/YOUR_PROJECT_NUMBER/locations/global/workloadIdentityPools/YOUR_POOL_NAME/providers/YOUR_PROVIDER_NAME'
          service_account: 'your-service-account@your-project-id.iam.gserviceaccount.com'

      - name: Run the authentication demo
        run: python src/main.py
  - <b>AUTHENTICATION FUNCTIONS:</b>

This contains the core functions for authentication. It demonstrates the three methods. 
import os
from google.auth.transport.requests import Request
from google.oauth2 import service_account
from google.auth import default

def authenticate_with_service_account_key(key_file_path):
    """Authenticates using a service account key file."""
    try:
        credentials = service_account.Credentials.from_service_account_file(key_file_path)
        print("✅ Authenticated successfully with Service Account Key.")
        return credentials
    except Exception as e:
        print(f"❌ Failed to authenticate with Service Account Key: {e}")
        return None

def authenticate_with_adc():
    """Authenticates using Application Default Credentials (ADC)."""
    try:
        credentials, project = default()
        if credentials.valid and credentials.token:
            print(f"✅ Authenticated successfully with ADC. Project: {project}")
            return credentials
        else:
            # Refresh if token has expired
            credentials.refresh(Request())
            print(f"✅ Authenticated successfully with ADC (refreshed).")
            return credentials
    except Exception as e:
        print(f"❌ Failed to authenticate with ADC: {e}")
        return None


# This function demonstrates how to use the credentials once the action provides them.
def authenticate_with_workload_identity():
    """Authenticates using Workload Identity Federation (assumes credentials are set by the environment)."""
    try:
        credentials, project = default()
        print(f"✅ Authenticated successfully with Workload Identity. Project: {project}")
        return credentials
    except Exception as e:
        print(f"❌ Failed to authenticate with Workload Identity: {e}")
        print("Hint: This method requires a specific environment setup (e.g., GitHub Actions).")
        return None

  - <b>MAIN SCRIPT TO RUN DEMO:</b>

       This script uses the authentication functions to perform a simple task, like listing your Google Cloud projects. This provides a tangible demonstration of successful authentication.

    import authenticate
from google.api_core.exceptions import GoogleAPIError
from google.cloud import resourcemanager_v3
import google.api_core.client_options as client_options

def list_gcp_projects(credentials):
    """Lists all Google Cloud projects accessible with the given credentials."""
    if not credentials:
        print("⚠️ No valid credentials provided. Cannot list projects.")
        return

    try:
        # Use the Resource Manager V3 API to list projects
        client = resourcemanager_v3.ProjectsClient(
            credentials=credentials,
            client_options=client_options.ClientOptions(api_endpoint="https://cloudresourcemanager.googleapis.com")
        )

        print("🔎 Listing projects...")
        request = resourcemanager_v3.ListProjectsRequest(parent="organizations/YOUR_ORGANIZATION_ID") # Replace with your organization ID or leave empty to list all projects accessible to the user/service account
        
        page_iterator = client.list_projects(request=request)
        
        for project in page_iterator:
            print(f"   - Project: {project.display_name} ({project.project_id})")
        
        print("✅ Project listing complete.")

    except GoogleAPIError as e:
        print(f"❌ An API error occurred: {e}")
    except Exception as e:
        print(f"❌ An unexpected error occurred: {e}")

if __name__ == "__main__":
    # --- DEMO 1: Service Account Key Authentication ---
    # To run this, you need to set up a service account and download the key file.
    # Put the key file in the src directory and update the path below.
    # 
    print("\n--- Running Demo: Service Account Key ---")
    key_file_path = "path/to/your/key.json"
    sa_credentials = authenticate.authenticate_with_service_account_key(key_file_path)
    if sa_credentials:
        list_gcp_projects(sa_credentials)

    # --- DEMO 2: Application Default Credentials (ADC) Authentication ---
    # Run `gcloud auth application-default login` on your machine first.
    print("\n--- Running Demo: Application Default Credentials (ADC) ---")
    adc_credentials = authenticate.authenticate_with_adc()
    if adc_credentials:
        list_gcp_projects(adc_credentials)

    # --- DEMO 3: Workload Identity Federation (for CI/CD) ---
    # This won't work locally without the proper environment variables,
    # but the code is here to show the concept. It's meant to be used in
    # a GitHub Actions workflow.
    print("\n--- Running Demo: Workload Identity Federation ---")
    wi_credentials = authenticate.authenticate_with_workload_identity()

 
  - <b>PYTHON DEPENDENCIES:</b>

     google-auth
    
     google-api-python-client



 - <b>CERTIFICATE OF COMPLETION:</b>

 
<img src="https://imgur.com/MsJReb3.png" height="65%" width="65%" alt=""/>
