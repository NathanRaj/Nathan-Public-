# Identity Access Management Service

The Identity Access Management (IDAM) Service is responsible for resolving person identities from multiple data sources, generating resolved person grants for each identity and synchornising with third party identity providers such as OKTA.

Documentation for IDAM can be found here: <https://dominos.atlassian.net/wiki/spaces/SellAndTrack/pages/1277887543/IDAM+-+Identity+Access+Management>

## Description

IDAM solution conceptually has three main layers. These are the staging layer, operational layer and resolved layer. IDAM Receives data from data sources such as pulse and nexus into a staging area (Staging Layer). These staging records are then analysed and matched on the basis of email, first and last name. From the matched staguing record state IDAM builds and upserts a person identity including their business role grants. The person identity information is then created into document and forwarded to the idam autorisation service (Resolved Layer). In addition IDAM makes the decision to synchronise that person with it's identity provider (OKTA).

The key thing to note in the code is we have separted concerns with how we received and store staged records and the person identity handling. The staging area works on the notion of data sources, from the the person identity services works with a PersonStagingRecord service which brings the various data sources togethor into a common structure called a PersonStaginRecord. The person identity services (ie: matchers, suitability resolvers and builders) work with that common format. This theoretically should allow the addition of new staging sources into idam relatively easily.

In addition to this how we synchronse with the identity provider (OKTA) is also separated, we essentially send identity details for the IdentityProviderUpdateService and the synchornisation process is managed by the IdentityProviderServices.

## Getting Started

In order to successfully contribute to the IDAM Service you should have access to the following

- OKTA Spike
  - Obtaining admin access to this environment will be helpful if you have a requirement to update okta user sychronisation
- Send Grid
- Lanuch Darkley

In Order to start the application on your local environment you will need to have the following settings changed temporarily in your appsettings & appsettings.development:

- Redis
- SQL Server running locally (optional, reccomended)
  - If you run SQL Server locally you will need to execute the dacpac project against your local sql server.
    - repo: <https://dominos-au.visualstudio.com/OneDigital/_git/Database.IdentityAccessManagement>
- OKTA Spike  (<https://dpe-spike-admin.oktapreview.com/>)
  - Further information can be found here with regard to keys: <https://dominos.atlassian.net/wiki/spaces/SellAndTrack/pages/1312851230/OKTA+access>
  - You will be required to add the dev spike okta token in the Appsettings.Development.json->OktaClientConfiguration.Token property
- Send Grid Configuration
  - If you do not intend on developing for the support emailing side which implements send grid at a bare minum you need to enter a made up key value in the AppSettings.Json->SendGridEmailClientSettings.ApiKey property
  - Note Idam uses it's own send grid account
- Service Bus Settings
  - You need to keep the structure for this intact when deploying into the cloud so do not ever commit the following temporary change. The connections are layed out so that cloud formation can configured enivronment vars appropriately in the docker container.
  - In order to start idam you will need to temporarily remove all but the first service bus connections.
- S3 Bucket locally configured
  - Create a sandbox S# bucket
  - Configure in AppSettings.Development
    - AWS settings to match your local credentials and sandbox region
    - IdentityAccessManagementConfigurationS3Settings->BucketName to match your sandbox bucket name
    - Temporarily configure IdentityAccessManagementConfigurationS3Settings->InitialiseUsingAWSOptions to true so that it uses the AWS settings you configured before
      - Do not commit this as true otherwise you may find IDAM does not work in the cloud.
