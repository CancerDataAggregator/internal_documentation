# Obtaining CloudOne Database Credentials

## Purpose
The purpose of this SOP is to provide guidance on how to obtain the credentials necessary for accessing the CDA databases hosted within CloudOne.

## Prequirements
- NIH-NCI account
- NIH laptop with admin rights & NIH-VPN access
- Access to https://iam.cancer.gov/
- Access to the CloudOne AWS role: **NIH.NCI.CBIIT.CRDC-CDA.NONPROD**
    - > **_NOTE:_** If you do not have access to this role, request through https://service.cancer.gov/

## Procedures
1. Connect to the NIH VPN via the Cisco Secure Client application
1. Log in to https://iam.cancer.gov/
1. Click on the AWS app 
    - > **_NOTE:_** If you do not see it put in a request through https://service.cancer.gov/
1. You should now be on the AWS access portal
1. Click the dropdown arrow to the right of the **NIH.NCI.CBIIT.CRDC-CDA.NONPROD** account and click **NCIAWSPowerUserAccess** link
    - > **_NOTE:_** If you do not see it put in a request through https://service.cancer.gov/
1. You should now be on the AWS console homepage
1. Ensure you are in the us-east-1 region by clicking on the region dropdown in the top right
    - > **_NOTE:_** Not all services will be accessible in other regions
1. In the search bar at the top, look for Secrets Manager and select it
1. Click on the credentials for the environment you want to deploy to
    - > **_NOTE:_** They should look like: **crdccda{ENV}-db-credentials** where “{ENV}” is the environment you want
1. Click the Retrieve secret value button in the Secret Value section
1. The host, username, dbname, and password values here will be used in later sections
