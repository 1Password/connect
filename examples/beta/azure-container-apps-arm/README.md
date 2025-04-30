# Deploy 1Password Connect Server on Azure Container Apps using the Azure Portal

_Learn how to deploy 1Password Connect Server on the [Azure Container Apps](https://azure.microsoft.com/en-us/products/container-apps/#overview) service._

This deployment consists of two [containers](https://learn.microsoft.com/en-us/azure/container-apps/containers): One for the Connect API and another for Connect Sync. There's also an [ingress](https://learn.microsoft.com/en-us/azure/container-apps/ingress-overview) for the API container. There are a few benefits to deploying 1Password Connect Server on Azure Container Apps:

- **Low cost:** For standard deployments, the service will host your Connect Server for ~$16 USD/month (as of January 2024). Container Apps pricing is variable based on activity, and you can learn more on [Microsoft's pricing page](https://azure.microsoft.com/en-us/pricing/details/container-apps/).
- **Automatic DNS record management:** You don't need to manage a DNS record. Azure Container Apps automatically provides a unique one for your Connect Server domain.
- **Automatic TLS certificate management:** Azure Container Apps automatically handles TLS certificate management on your behalf.

## Before you begin

Before you begin, ensure you have an Azure account with permission to create a Container App. You'll also need the `1password-credentials.json` file for your 1Password Connect Server, which contains the credentials for authenticating the server.

> [!NOTE]
> If you don't have an Azure account, you can sign up for a free trial with starting credit: https://azure.microsoft.com/free/

## Step 1: Download the template file

1. Download the [ARM(Azure Resource Manager) template](./aca-op-connect-server-template.json) file using the download icon at the top right.

## Step 2: Create the Container App

1. Sign in to the Azure Portal and go to the [Deploy a custom template](https://portal.azure.com/#create/Microsoft.Template) page.
2. Click **Build your own template in the editor**.
3. Click **Load file** and upload the template file you downloaded earlier. Then click **Save**.
4. Fill out the following fields:
   - **Subscription** : Choose the subscription you prefer.
   - **Resource group**: Choose an existing Resource Group or create a new one using **Create new** button.
   - **Region**: Choose the region you prefer.
   - **Container App Name**: Enter a name you'd like to use, default will be `op-connect-con-app`.
   - **Container App Env Name**: Enter a name you'd like to use, default will be `op-connect-con-env`.
   - **Container App Log Analytics Name**: Enter a name you'd like to use, default will be `op-connect-con-app-log-analytics`.
   - **Credentials** : Paste the entire contents of the `1password-credentials.json` file.
5. Click **Review + create**.
6. Once the validation succeeds, click **Create**. It is expected to take a couple of minutes to complete the deployment.

## Step 3: Test your Connect Server

Once your deployment is complete, click **Go to resource group** and click on the container app you created.

To test if your Connect Server is online, choose **Overview** in your application's sidebar, then copy your **Application Url** link. This is your **Connect Server URL**. Use your 1Password Connect access token to interact with the API (e.g., via a `curl` request or a client application) to verify that your Connect Server is connected to your 1Password account.

```sh
curl --silent --show-error --request GET --header "Accept: application/json" \
  --header "Authorization: Bearer mF_9.B5f-4.1JqM" \
  https://op-connect.example.com/v1/vaults
```

<details>
<summary>Example JSON response:</summary>

```json
[
  {
    "attributeVersion": 1,
    "contentVersion": 9,
    "createdAt": "2025-03-21T13:06:35Z",
    "id": "hgm3sn37vkj3lsdkeouq46wyca",
    "items": 5,
    "name": "REDACTED",
    "type": "USER_CREATED",
    "updatedAt": "2025-04-18T16:38:40Z"
  },
  {
    "attributeVersion": 1,
    "contentVersion": 2,
    "createdAt": "2025-03-21T02:15:33Z",
    "id": "mn4w65hq6ar7nc2fc7qgoru3ki",
    "items": 1,
    "name": "REDACTED",
    "type": "USER_CREATED",
    "updatedAt": "2025-03-21T02:16:32Z"
  },
  {
    "attributeVersion": 1,
    "contentVersion": 94,
    "createdAt": "2025-03-21T15:20:41Z",
    "id": "z4s56cbejab6q6urod3l5icqky",
    "items": 3,
    "name": "REDACTED",
    "type": "USER_CREATED",
    "updatedAt": "2025-04-29T02:49:13Z"
  }
]
```

</details>
<br />

## Step 4: Integrate with your application

To use the Connect Server, configure your application to communicate with the Connect Server URL using the access token provided in your 1Password account. Follow the [1Password Connect integration guide](https://developer.1password.com/docs/connect) to set up your application.

<hr>

## Update your Connect Server in the Azure Portal

> [!TIP]
> Check for 1Password Connect Server updates on the [1Password releases page](https://releases.1password.com/).

1. Within your deployed 1Password Connect Server Container App in the [Azure Container Apps Portal](https://portal.azure.com/#view/HubsExtension/BrowseResource/resourceType/Microsoft.App%2FcontainerApps), select **Containers** from the sidebar.
2. Click **Edit and deploy**.
3. Select the checkbox next to your **connect-api** container, then choose **Edit**.
4. Change the version number in the **Image and Tag** field, `1password/connect-api:latest`, to match the latest version from our [1Password releases page](https://releases.1password.com/).
5. Repeat for the **connect-sync** container, updating `1password/connect-sync:latest` to the same version.
6. Select **Save**.
7. Select **Create** to deploy a new revision using the updated images.
8. Test your Connect Server URL with your access token to verify the update.

## Get help

### How to update the **credentials** secret

To use a new `1password-credentials.json` credentials file for your Connect Server, replace the secret in your Container App:

<details>
<summary>Replace your <code>credentials</code> secret using the Azure Portal</summary>

1. Open the Azure Portal and go to the [Container Apps](https://portal.azure.com/#view/HubsExtension/BrowseResource/resourceType/Microsoft.App%2FcontainerApps) page.
2. Choose **Secrets** from the Settings section in the sidebar.
3. Edit the **credentials** secret and paste the entire contents of your new `1password-credentials.json` file.
4. Select the checkbox and click **Save**.
5. Choose the **Revisions** from the Application section in the sidebar.
6. Click your current active revision and choose **Restart** in the details pane.
7. Test your Connect Server URL with your new access token to [test your Connect Server](#step-3-test-your-connect-server).
</details>