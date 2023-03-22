# testing

import os
import json
from google.cloud import artifactregistry_v1beta2, secretmanager_v1
from google.auth.transport.requests import Request
from google.oauth2 import service_account

def test_artifact_registry_connectivity(request):
    project_id = "serviceproject-379219"
    location = "europe-west2"
    repository = "quickstart-python-repo"
    SCOPES = [
        "https://www.googleapis.com/auth/cloud-platform",
        "https://www.googleapis.com/auth/devstorage.read_write",
    ]

    # Load the service account key from Secret Manager
    secret_project_id = "hostproject-379219"
    secret_name = "serviceprojectartifact"
    secret_version = "latest"

    secret_manager_client = secretmanager_v1.SecretManagerServiceClient()
    secret_resource_name = f"projects/{secret_project_id}/secrets/{secret_name}/versions/{secret_version}"
    secret_response = secret_manager_client.access_secret_version(request={"name": secret_resource_name})
    secret_payload = secret_response.payload.data.decode("UTF-8")
    service_account_info = json.loads(secret_payload)

    # Create credentials from the service account key
    credentials = service_account.Credentials.from_service_account_info(service_account_info, scopes=SCOPES)

    # Create an Artifact Registry client
    client = artifactregistry_v1beta2.ArtifactRegistryClient(credentials=credentials)

    # List packages in the repository
    repository_path = f"projects/{project_id}/locations/{location}/repositories/{repository}"
    package_list = client.list_packages(parent=repository_path)
