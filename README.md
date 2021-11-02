# boomi-gcp
Google Cloud Builder deployment of Boomi Molecule to Google Kubernetes Engine with an external load balancer and autoscaler.

The configuration is inteneded for prototyping and is not sized or secured for production (i.e. no API gateway).

You must have a Boomi account with unused molecule licenses.

The steps below are for manual execution but the same code can be used with a Google Cloud Build trigger.

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

1. Fork this Github repository and GIT clone the repository to your local PC

1. In Google Cloud SDK Shell cd to the local repository folder and submit the build, replacing the substition values.

   e.g. _ZONE="asia-southeast1-a",_ENV="dev",_BOOMI_USERNAME="BOOMI_TOKEN.me@myorg.com",_BOOMI_PASSWORD="[API Token]".

   Sign-on username and password will not work if Single Sign On or Two Factor Authentication is enabled in your Boomi account).

   _ENV is used for object naming only and does not need to correspond to a GCP environment

   ```sh
     gcloud builds submit --substitutions=_PROJECT="[project-id]",_ZONE="australia-southeast1-a",_ENV=dev,_BOOMI_USERNAME="BOOMI_TOKEN.james_m_hutton@dell.com",_BOOMI_PASSWORD="b9b9d894-7ea9-4516-9bcf-79d1630b95ac",_BOOMI_ACCOUNTID="boomi_jameshutton-N7LJSM",_BOOMI_ENVIRONMENTID="40590140-dc5b-4c49-89f9-47f99641a0d4"
     ```

1. If the build is successful, the new runtime should be visible in the Boomi UI within ten minutes.

1. To test web services, deploy a Shared Web Services listener process to your environment and obtain the external IP from kubectl or the GCP console (Google Kubernetes Engine>Services & Ingress).

   The URL will following this format: http://144.155.122.133/ws/simple/myTestGet.
   
   HTTP only is configured and there is no authentication.

1. Scale down or delete the GKE cluster when not in use to avoid costs. If deleting the cluster, manually delete the molecule from Boomi Atom Management to avoid exceeding Molecule licenses