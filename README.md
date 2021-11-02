# boomi-gcp
Google Cloud Builder deployment of Boomi Molecule to Google Kubernetes Engine with an external load balancer and autoscaler.

The configuration is inteneded for prototyping and is not sized or secured for production (i.e. no API gateway).

You must have a Boomi account with unused molecule licenses.

## Steps

1. Create a Google Cloud Platform account if you don't have one. This deployment should work on the free tier.

1. Create a Google Cloud Platform project

1. Install Google Cloud SDK on your local PC

1. In Google Cloud SDK Shell, configure the gcloud command-line tool to use your project:

   ```sh
     gcloud config set project [project-id]
   ```

1. Enable the required APIs:

   ```sh
   gcloud services enable compute.googleapis.com deploymentmanager.googleapis.com cloudbuild.googleapis.com container.googleapis.com gkeconnect.googleapis.com gkehub.googleapis.com
   ```

1. Copy the service account number (as in nnn@cloudbuild.gserviceaccount.com) from the GCP console (IAM & Admin>IAM)

1. Add IAM roles to your project's Cloud Build service account:

   ```sh
	 gcloud projects add-iam-policy-binding [project-id] --member=serviceAccount:[service-account-no]@cloudbuild.gserviceaccount.com --role=roles/compute.instanceAdmin.v1
	 gcloud projects add-iam-policy-binding [project-id] --member=serviceAccount:[service-account-no]@cloudbuild.gserviceaccount.com --role=roles/container.admin
     gcloud projects add-iam-policy-binding [project-id] --member=serviceAccount:[service-account-no]@cloudbuild.gserviceaccount.com --role=roles/iam.serviceAccountUser
     ```

1. Fork this Github repository 

1. Submit the build using Google Cloud SDK Shell (GIT clone the repository to your local PC, cd to the local repository folder and execute the command below, replacing the substition values) or a Google Cloud trigger.

   For zone use an ID like "asia-southeast1-a".

   Sign-on username and password will not work if Single Sign On or Two Factor Authentication is enabled in your Boomi account so instead use _BOOMI_USERNAME="BOOMI_TOKEN.me@myorg.com",_BOOMI_PASSWORD="[API Token]".
   
   The Boomi Platform UI shows account ID (Settings>Account Information and Setup) and environment ID (Management>Atom Management).

   ```sh
     gcloud builds submit --substitutions=_PROJECT="[project-id]",_ZONE="[gcp-zone-id]",_ENV=dev,_BOOMI_USERNAME="[username]",_BOOMI_PASSWORD="[password]",_BOOMI_ACCOUNTID="[account-id]",_BOOMI_ENVIRONMENTID="[environment-guid]"
     ```

1. If the build is successful, the new runtime should be visible in the Boomi UI within ten minutes from the start time.

1. To test web services, deploy a Shared Web Services listener process to your environment and obtain the external IP from kubectl or the GCP console (Google Kubernetes Engine>Services & Ingress).

   The URL will following this format: http://144.155.122.133/ws/simple/myTestGet.
   
   HTTP only is configured and there is no authentication.

1. Scale down or delete the GKE cluster when not in use to avoid costs. If deleting the cluster, manually delete the molecule from Boomi Atom Management to avoid exceeding Molecule licenses