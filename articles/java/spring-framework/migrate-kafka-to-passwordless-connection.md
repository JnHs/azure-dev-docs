---
title: Migrate an application to use passwordless connections with Azure Event Hubs for Kafka
description: Learn how to migrate existing applications using Azure Event Hubs for Kafka away from authentication patterns such as connection strings to more secure approaches like Managed Identity.
author: KarlErickson
ms.author: karler
ms.reviewer: seal
ms.topic: how-to
ms.date: 11/16/2022
ms.custom:
  - passwordless-java
  - passwordless-js
  - passwordless-python
  - passwordless-dotnet
  - spring-cloud-azure
  - devx-track-java
  - devx-track-azurecli
  - devx-track-extended-java
  - sfi-image-nochange
---

# Migrate an application to use passwordless connections with Azure Event Hubs for Kafka

This article explains how to migrate from traditional authentication methods to more secure, passwordless connections with Azure Event Hubs for Kafka.

Application requests to Azure Event Hubs for Kafka must be authenticated. Azure Event Hubs for Kafka provides different ways for apps to connect securely. One of the ways is to use a connection string. However, you should prioritize passwordless connections in your applications when possible.

Passwordless connections are supported since Spring Cloud Azure 4.3.0. This article is a migration guide for removing credentials from Spring Cloud Stream Kafka applications.

## Compare authentication options

When the application authenticates with Azure Event Hubs for Kafka, it provides an authorized entity to connect the Event Hubs namespace. Apache Kafka protocols provide multiple Simple Authentication and Security Layer (SASL) mechanisms for authentication. According to the SASL mechanisms, there are two authentication options that you can use to authorize access to your secure resources: Microsoft Entra authentication and Shared Access Signature (SAS) authentication.

<a name='azure-ad-authentication'></a>

### Microsoft Entra authentication

Microsoft Entra authentication is a mechanism for connecting to Azure Event Hubs for Kafka using identities defined in Microsoft Entra ID. With Microsoft Entra authentication, you can manage service principal identities and other Microsoft services in a central location, which simplifies permission management.

Using Microsoft Entra ID for authentication provides the following benefits:

- Authentication of users across Azure services in a uniform way.
- Management of password policies and password rotation in a single place.
- Multiple forms of authentication supported by Microsoft Entra ID, which can eliminate the need to store passwords.
- Customers can manage Event Hubs permissions using external (Microsoft Entra ID) groups.
- Support for token-based authentication for applications connecting to Azure Event Hubs for Kafka.

### SAS authentication

Event Hubs also provides Shared Access Signatures (SAS) for delegated access to Event Hubs for Kafka resources.

Although it's possible to connect to Azure Event Hubs for Kafka with SAS, it should be used with caution. You must be diligent to never expose the connection strings in an unsecure location. Anyone who gains access to the connection strings is able to authenticate. For example, there's a risk that a malicious user can access the application if a connection string is accidentally checked into source control, sent through an unsecure email, pasted into the wrong chat, or viewed by someone who shouldn't have permission. Instead, authorizing access using the OAuth 2.0 token-based mechanism provides superior security and ease of use over SAS. Consider updating your application to use passwordless connections.

[!INCLUDE [introducing-passwordless-connections](includes/introducing-passwordless-connections.md)]

## Migrate an existing application to use passwordless connections

The following steps explain how to migrate an existing application to use passwordless connections instead of a SAS solution.

### 0) Prepare the working environment for local development authentication

First, use the following command to set up some environment variables.

```bash
export AZ_RESOURCE_GROUP=<YOUR_RESOURCE_GROUP>
export AZ_EVENTHUBS_NAMESPACE_NAME=<YOUR_EVENTHUBS_NAMESPACE_NAME>
export AZ_EVENTHUB_NAME=<YOUR_EVENTHUB_NAME>
```

Replace the placeholders with the following values, which are used throughout this article:

- `<YOUR_RESOURCE_GROUP>`: The name of the resource group you'll use.
- `<YOUR_EVENTHUBS_NAMESPACE_NAME>`: The name of the Azure Event Hubs namespace you'll use.
- `<YOUR_EVENTHUB_NAME>`: The name of the event hub you'll use.

### 1) Grant permission for Azure Event Hubs

If you want to run this sample locally with Microsoft Entra authentication, be sure your user account has authenticated via Azure Toolkit for IntelliJ, Visual Studio Code Azure Account plugin, or Azure CLI. Also, be sure the account has been granted sufficient permissions.

#### [Azure portal](#tab/azure-portal)

1. In the Azure portal, locate your Event Hubs namespace using the main search bar or left navigation.

1. On the Event Hubs overview page, select **Access control (IAM)** from the left-hand menu.

1. On the **Access control (IAM)** page, select the **Role assignments** tab.

1. Select **Add** from the top menu and then **Add role assignment** from the resulting drop-down menu.

   :::image type="content" source="media/migrate-kafka-to-passwordless-connection/migration-role-eventhubs.png" alt-text="Screenshot of Azure portal Access Control (IAM) page of Event Hubs Namespace resource with Add role assignment highlighted." lightbox="media/migrate-kafka-to-passwordless-connection/migration-role-eventhubs.png":::

1. Use the search box to filter the results to the desired role. For this example, search for **Azure Event Hubs Data Sender** and **Azure Event Hubs Data Receiver** and select the matching result and then choose **Next**.

1. Under **Assign access to**, select **User, group, or service principal**, and then choose **Select members**.

1. In the dialog, search for your Microsoft Entra username (usually your **user@domain** email address) and then choose **Select** at the bottom of the dialog.

1. Select **Review + assign** to go to the final page, and then **Review + assign** again to complete the process.

#### [Azure CLI](#tab/azure-cli)

To authenticate using the Azure CLI, use the following steps:

1. Use the following command to sign to your Azure account:

   ```bash
   az login
   ```

1. Use the following command to set your current subscription context. Replace `ssssssss-ssss-ssss-ssss-ssssssssssss` with the GUID for the subscription you want to use with Azure:

   ```azurecli
   az account set --subscription ssssssss-ssss-ssss-ssss-ssssssssssss
   ```

1. Use the following command to get the resource ID for your Azure Event Hubs namespace:

   ```azurecli
   export AZURE_EVENTHUBS_RESOURCE_ID=$(az resource show \
       --resource-group $AZ_RESOURCE_GROUP \
       --name $AZ_EVENTHUBS_NAMESPACE_NAME \
       --resource-type Microsoft.EventHub/Namespaces \
       --query "id" \
       --output tsv)
   ```

1. Use the following command to get your user object ID of your Azure CLI user account:

   ```azurecli
   export AZURE_AD_ACCOUNT_ID=$(az ad signed-in-user show \
       --query "id" --output tsv)
   ```

1. Use the following commands to assign `Azure Event Hubs Data Sender` and `Azure Event Hubs Data Receiver` roles to your account.

   ```azurecli
   az role assignment create \
       --assignee $AZURE_AD_ACCOUNT_ID \
       --role "Azure Event Hubs Data Receiver" \
       --scope $AZURE_EVENTHUBS_RESOURCE_ID
   az role assignment create \
       --assignee $AZURE_AD_ACCOUNT_ID \
       --role "Azure Event Hubs Data Sender" \
       --scope $AZURE_EVENTHUBS_RESOURCE_ID
   ```

---

For more information about granting access roles, see [Authorize access to Event Hubs resources using Microsoft Entra ID](/azure/event-hubs/authorize-access-azure-active-directory).

### 2) Sign in and migrate the app code to use passwordless connections

For local development, make sure you're authenticated with the same Microsoft Entra account you assigned the role to on your Event Hubs. You can authenticate via the Azure CLI, Visual Studio, Azure PowerShell, or other tools such as IntelliJ.

[!INCLUDE [sign-in](includes/passwordless-sign-in.md)]

Next, use the following steps to update your Spring Kafka application to use passwordless connections. Although conceptually similar, each framework uses different implementation details.

#### [Java](#tab/java-kafka)

1. Inside your project, open the **pom.xml** file and add the following reference:

   ```xml
   <dependency>
      <groupId>com.azure</groupId>
      <artifactId>azure-identity</artifactId>
      <version>1.6.0</version>
   </dependency>
   ```

1. After migration, implement [AuthenticateCallbackHandler](https://kafka.apache.org/30/javadoc/org/apache/kafka/common/security/auth/AuthenticateCallbackHandler.html) and [OAuthBearerToken](https://kafka.apache.org/30/javadoc/org/apache/kafka/common/security/oauthbearer/OAuthBearerToken.html) in your project for OAuth2 authentication, as shown in the following example.

   ```Java
   public class KafkaOAuth2AuthenticateCallbackHandler implements AuthenticateCallbackHandler {

      private static final Duration ACCESS_TOKEN_REQUEST_BLOCK_TIME = Duration.ofSeconds(30);
      private static final String TOKEN_AUDIENCE_FORMAT = "%s://%s/.default";

      private Function<TokenCredential, Mono<OAuthBearerTokenImp>> resolveToken;
      private final TokenCredential credential = new DefaultAzureCredentialBuilder().build();

      @Override
      public void configure(Map<String, ?> configs, String mechanism, List<AppConfigurationEntry> jaasConfigEntries) {
         TokenRequestContext request = buildTokenRequestContext(configs);
         this.resolveToken = tokenCredential -> tokenCredential.getToken(request).map(OAuthBearerTokenImp::new);
      }

      private TokenRequestContext buildTokenRequestContext(Map<String, ?> configs) {
         URI uri = buildEventHubsServerUri(configs);
         String tokenAudience = buildTokenAudience(uri);

         TokenRequestContext request = new TokenRequestContext();
         request.addScopes(tokenAudience);
         return request;
      }

      @SuppressWarnings("unchecked")
      private URI buildEventHubsServerUri(Map<String, ?> configs) {
         String bootstrapServer = Arrays.asList(configs.get(BOOTSTRAP_SERVERS_CONFIG)).get(0).toString();
         bootstrapServer = bootstrapServer.replaceAll("\\[|\\]", "");
         URI uri = URI.create("https://" + bootstrapServer);
         return uri;
      }

      private String buildTokenAudience(URI uri) {
         return String.format(TOKEN_AUDIENCE_FORMAT, uri.getScheme(), uri.getHost());
      }

      @Override
      public void handle(Callback[] callbacks) throws UnsupportedCallbackException {
         for (Callback callback : callbacks) {
            if (callback instanceof OAuthBearerTokenCallback) {
               OAuthBearerTokenCallback oauthCallback = (OAuthBearerTokenCallback) callback;
               this.resolveToken
                       .apply(credential)
                       .doOnNext(oauthCallback::token)
                       .doOnError(throwable -> oauthCallback.error("invalid_grant", throwable.getMessage(), null))
                       .block(ACCESS_TOKEN_REQUEST_BLOCK_TIME);
            } else {
               throw new UnsupportedCallbackException(callback);
            }
         }
      }

      @Override
      public void close() {
         // NOOP
      }
   }
   ```

   ```java
   public class OAuthBearerTokenImp implements OAuthBearerToken {
       private final AccessToken accessToken;
       private final JWTClaimsSet claims;

       public OAuthBearerTokenImp(AccessToken accessToken) {
           this.accessToken = accessToken;
           try {
               claims = JWTParser.parse(accessToken.getToken()).getJWTClaimsSet();
           } catch (ParseException exception) {
               throw new SaslAuthenticationException("Unable to parse the access token", exception);
           }
       }

       @Override
       public String value() {
           return accessToken.getToken();
       }

       @Override
       public Long startTimeMs() {
           return claims.getIssueTime().getTime();
       }

       @Override
       public long lifetimeMs() {
           return claims.getExpirationTime().getTime();
       }

       @Override
       public Set<String> scope() {
           // Referring to https://docs.microsoft.com/azure/active-directory/develop/access-tokens#payload-claims, the scp
           // claim is a String, which is presented as a space separated list.
           return Optional.ofNullable(claims.getClaim("scp"))
                   .map(s -> Arrays.stream(((String) s)
                   .split(" "))
                   .collect(Collectors.toSet()))
                   .orElse(null);
       }

       @Override
       public String principalName() {
           return (String) claims.getClaim("upn");
       }

       public boolean isExpired() {
           return accessToken.isExpired();
       }
   }
   ```

1. When you create your Kafka producer or consumer, add the configuration needed to support the [SASL/OAUTHBEARER](https://kafka.apache.org/documentation/#security_sasl_oauthbearer) mechanism. The following examples show what your code should look like before and after migration. In both examples, replace the `<eventhubs-namespace>` placeholder with the name of your Event Hubs namespace.

   Before migration, your code should look like the following example:

   ```java
   Properties properties = new Properties();
   properties.put(CommonClientConfigs.BOOTSTRAP_SERVERS_CONFIG, "<eventhubs-namespace>.servicebus.windows.net:9093");
   properties.put(CommonClientConfigs.SECURITY_PROTOCOL_CONFIG, "SASL_SSL");
   properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
   properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
   properties.put(SaslConfigs.SASL_MECHANISM, "PLAIN");
   properties.put(SaslConfigs.SASL_JAAS_CONFIG,
           String.format("org.apache.kafka.common.security.plain.PlainLoginModule required username=\"$ConnectionString\" password=\"%s\";", connectionString));
   return new KafkaProducer<>(properties);
   ```

   After migration, your code should look like the following example. In this example, replace the `<path-to-your-KafkaOAuth2AuthenticateCallbackHandler>` placeholder with the full class name for your implemented `KafkaOAuth2AuthenticateCallbackHandler`.

   ```java
   Properties properties = new Properties();
   properties.put(CommonClientConfigs.BOOTSTRAP_SERVERS_CONFIG, "<eventhubs-namespace>.servicebus.windows.net:9093");
   properties.put(CommonClientConfigs.SECURITY_PROTOCOL_CONFIG, "SASL_SSL");
   properties.put(SaslConfigs.SASL_MECHANISM, "OAUTHBEARER");
   properties.put(SaslConfigs.SASL_JAAS_CONFIG, "org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required");
   properties.put(SaslConfigs.SASL_LOGIN_CALLBACK_HANDLER_CLASS, "<path-to-your-KafkaOAuth2AuthenticateCallbackHandler>");
   return new KafkaProducer<>(properties);
   ```

#### [Spring Kafka (Spring Boot application)](#tab/spring-kafka)

1. Inside your project, add the following reference to the `com.azure.spring:spring-cloud-azure-starter` package. This library contains all of the entities necessary to implement passwordless connections.

   ```xml
   <dependency>
       <groupId>com.azure.spring</groupId>
       <artifactId>spring-cloud-azure-starter</artifactId>
   </dependency>
   ```

1. Then, remove all of your existing connection or credential related Kafka properties. Add only the following property to configure the Kafka bootstrap server with your Azure Event Hubs namespace:

   ```properties
   spring.kafka.bootstrap-servers=$AZ_EVENTHUBS_NAMESPACE_NAME.servicebus.windows.net:9093
   ```

#### [Spring Cloud Stream Kafka Binder](#tab/spring-cloud-stream-kafka)

1. Inside your project, add the following reference to the `com.azure.spring:spring-cloud-azure-starter` package. This library contains all of the necessary entities to implement passwordless connections.

   ```xml
   <dependency>
       <groupId>com.azure.spring</groupId>
       <artifactId>spring-cloud-azure-starter</artifactId>
   </dependency>
   ```

1. Then, remove all of your existing connection or credential related Kafka properties. Add the following property to configure the Kafka bootstrap server with your Azure Event Hubs namespace:

   ```properties
   spring.cloud.stream.kafka.binder.brokers=$AZ_EVENTHUBS_NAMESPACE_NAME.servicebus.windows.net:9093
   ```

   > [!NOTE]
   > Starting with version `4.4.0`, this property will be added automatically, so there's no need to add it manually.

---

#### Run the app locally

After making these code changes, run your application locally. The new configuration should pick up your local credentials, assuming you're logged into a compatible IDE or command line tool, such as the Azure CLI, Visual Studio, or IntelliJ. The roles you assigned to your local dev user in Azure will allow your app to connect to the Azure service locally.

### 3) Configure the Azure hosting environment

After your application is configured to use passwordless connections and it runs locally, the same code can authenticate to Azure services after it's deployed to Azure. For example, an application deployed to an Azure Spring Apps instance that has a managed identity assigned can connect to Azure Event Hubs for Kafka.

In this section, you'll execute two steps to enable your application to run in an Azure hosting environment in a passwordless way:

- Assign the managed identity for your Azure hosting environment.
- Assign roles to the managed identity.

> [!NOTE]
> Azure also provides [Service Connector](/azure/service-connector/overview), which can help you connect your hosting service with Event Hubs. With Service Connector to configure your hosting environment, you can omit the step of assigning roles to your managed identity because Service Connector will do it for you. The following section describes how to configure your Azure hosting environment in two ways: one via Service Connector and the other by configuring each hosting environment directly.

> [!IMPORTANT]
> Service Connector's commands require [Azure CLI](/cli/azure/install-azure-cli) 2.41.0 or higher.

#### Assign the managed identity for your Azure hosting environment

The following steps show you how to assign a system-assigned managed identity for various web hosting services. The managed identity can securely connect to other Azure Services using the app configurations you set up previously.

##### [App Service](#tab/app-service)

1. On the main overview page of your Azure App Service instance, select **Identity** from the navigation pane.

1. On the **System assigned** tab, make sure to set the **Status** field to **on**. A system assigned identity is managed by Azure internally and handles administrative tasks for you. The details and IDs of the identity are never exposed in your code.

##### [Service Connector](#tab/service-connector)

When you use Service Connector, it can help to assign the system-assigned managed identity to your Azure hosting environment, and then configure the `Azure Event Hubs Data Sender` and `Azure Event Hubs Data Receiver` roles for the managed identity.

The following compute services are currently supported:

- Azure App Service
- Azure Spring Apps
- Azure Container Apps

For this migration guide, you'll use App Service, but the steps are similar for Azure Spring Apps and Azure Container Apps.

> [!NOTE]
> Azure Spring Apps currently only supports Service Connector using connection strings.

1. In the Azure portal, on the main overview page of your App Service instance, select **Service Connector** from the navigation pane.

1. Select **Create** from the main menu and the **Create connection** panel will open. Enter the following values:

   - **Service type**: Choose **Event Hubs**.
   - **Subscription**: Select the subscription you'd like to use.
   - **Connection Name**: Enter a name for your connection, such as **connector_appservice_eventhub**.
   - **Namespace**: Select the Event Hubs namespace you'd like to use.
   - **Client type**: Leave the default value selected or choose the specific client you'd like to use.

1. Select **Next: Authentication**.

   :::image type="content" source="media/migrate-kafka-to-passwordless-connection/service-connector-app-eventhubs-portal.png" alt-text="Screenshot of Azure portal Service Connector page of App Service resource with Create connection pane showing and Service type field with Event Hubs value highlighted." lightbox="media/migrate-kafka-to-passwordless-connection/service-connector-app-eventhubs-portal.png":::

1. Make sure **System assigned managed identity (Recommended)** is selected, and then select **Next: Networking**.
1. Leave the default values selected, and then select **Next: Review + Create**.
1. After Azure validates your settings, select **Create**.

The Service Connector will automatically assign a system-assigned managed identity for the app service. The connector will also assign the managed identity roles of `Azure Event Hubs Data Sender` and `Azure Event Hubs Data Receiver` for the Event Hubs instance you selected.

##### [Container Apps](#tab/container-apps)

1. On the main overview page of your Azure Container Apps instance, select **Identity** from the navigation pane.

1. On the **System assigned** tab, make sure to set the **Status** field to **on**. A system assigned identity is managed by Azure internally and handles administrative tasks for you. The details and IDs of the identity are never exposed in your code.

   :::image type="content" source="media/passwordless-connections/container-apps-identity.png" alt-text="Screenshot of Azure portal Identity page of Container App resource showing System assigned tab with Status field highlighted." lightbox="media/passwordless-connections/container-apps-identity.png":::

##### [Azure Spring Apps](#tab/azure-spring-apps)

1. On the main overview page of your Azure Spring Apps instance, select **Identity** from the navigation pane.

1. On the **System assigned** tab, make sure to set the **Status** field to **on**. A system assigned identity is managed by Azure internally and handles administrative tasks for you. The details and IDs of the identity are never exposed in your code.

   :::image type="content" source="media/passwordless-connections/spring-apps-identity.png" alt-text="Screenshot of Azure portal Identity page of App resource with System assigned tab showing and Status field highlighted." lightbox="media/passwordless-connections/spring-apps-identity.png":::

##### [Virtual Machines](#tab/virtual-machines)

1. On the main overview page of your virtual machine, select **Identity** from the navigation pane.

1. On the **System assigned** tab, make sure to set the **Status** field to **on**. A system assigned identity is managed by Azure internally and handles administrative tasks for you. The details and IDs of the identity are never exposed in your code.

   :::image type="content" source="media/passwordless-connections/virtual-machine-identity.png" alt-text="Screenshot of Azure portal Identity page of Virtual machine resource with System assigned tab showing and Status field highlighted." lightbox="media/passwordless-connections/virtual-machine-identity.png":::

##### [AKS](#tab/aks)

An Azure Kubernetes Service (AKS) cluster requires an identity to access Azure resources like load balancers and managed disks. This identity can be either a managed identity or a service principal. By default, when you create an AKS cluster, a system-assigned managed identity is automatically created.

---

You can also assign managed identity on an Azure hosting environment by using the Azure CLI.

##### [App Service](#tab/app-service)

You can assign a managed identity to an Azure App Service instance with the [az webapp identity assign](/cli/azure/webapp/identity) command, as shown in the following example.

```azurecli
export AZURE_MANAGED_IDENTITY_ID=$(az webapp identity assign \
    --resource-group $AZ_RESOURCE_GROUP \
    --name <app-service-name> \
    --query principalId \
    --output tsv)
```

##### [Service Connector](#tab/service-connector)

You can create a Service Connection between an Azure compute hosting environment and a target service by using the Azure CLI. The Azure CLI automatically handles creating a managed identity and assigns the proper role, as explained in the [Assign the managed identity for your Azure hosting environment](#assign-the-managed-identity-for-your-azure-hosting-environment) section.

First, install the [Service Connector](/azure/service-connector/overview) passwordless extension for the Azure CLI:

```azurecli
az extension add --name serviceconnector-passwordless --upgrade
```

If you're using an Azure App Service, use the [az webapp connection](/cli/azure/webapp/connection) command, as shown in the following example:

```azurecli
az webapp connection create eventhub \
    --resource-group $AZ_RESOURCE_GROUP \
    --name <app-service-name>
    --target-id $AZURE_EVENTHUBS_RESOURCE_ID \
    --client-type kafka-springBoot \
    --system-identity
```

If you're using Azure Spring Apps, use the `az spring connection` command, as shown in the following example:

```azurecli
az spring connection create eventhub \
    --app <spring-app-name> \
    --service <azure-spring-service-name> \
    --resource-group $AZ_RESOURCE_GROUP \
    --deployment <spring-app-deployment-name> \
    --target-id $AZURE_EVENTHUBS_RESOURCE_ID \
    --client-type kafka-springBoot \
    --system-identity
```

If you're using Azure Container Apps, use the `az containerapp connection` command, as shown in the following example:

```azurecli
az containerapp connection create eventhub \
    --resource-group $AZ_RESOURCE_GROUP \
    --name <container-app-name>
    --target-id $AZURE_EVENTHUBS_RESOURCE_ID \
    --client-type kafka-springBoot \
    --system-identity
```

##### [Container Apps](#tab/container-apps)

You can assign a managed identity to an Azure Container Apps instance with the [az containerapp identity assign](/cli/azure/containerapp/identity) command, as shown in the following example:

```azurecli
export AZURE_MANAGED_IDENTITY_ID=$(az containerapp identity assign \
    --resource-group $AZ_RESOURCE_GROUP \
    --name <container-app-name> \
    --system-assigned \
    --query principalId \
    --output tsv)
```

##### [Azure Spring Apps](#tab/azure-spring-apps)

You can assign a managed identity to an Azure Spring Apps instance with the [az spring app identity assign](/cli/azure/spring/app/identity) command, as shown in the following example:

```azurecli
export AZURE_MANAGED_IDENTITY_ID=$(az spring app identity assign \
    --resource-group $AZ_RESOURCE_GROUP \
    --name <spring-app-name> \
    --service <spring-apps-service-name> \
    --system-assigned \
    --query identity.principalId \
    --output tsv)
```

##### [Virtual Machines](#tab/virtual-machines)

You can assign a managed identity to a virtual machine with the [az vm identity assign](/cli/azure/vm/identity) command, as shown in the following example:

```azurecli
export AZURE_MANAGED_IDENTITY_ID=$(az vm identity assign \
    --resource-group $AZ_RESOURCE_GROUP \
    --name <vm-name> \
    --query principalId \
    --output tsv)
```

##### [AKS](#tab/aks)

You can assign a managed identity to an Azure Kubernetes Service (AKS) instance with the [az aks update](/cli/azure/aks) command, as shown in the following example:

```azurecli
export AZURE_MANAGED_IDENTITY_ID=$(az aks update \
    --resource-group $AZ_RESOURCE_GROUP \
    --name <AKS-cluster-name> \
    --enable-managed-identity \
    --query identityProfile.kubeletidentity.clientId \
    --output tsv)
```

---

#### Assign roles to the managed identity

Next, grant permissions to the managed identity you created to access your Event Hubs namespace. You can grant permissions by assigning a role to the managed identity, just like you did with your local development user.

##### [Service Connector](#tab/assign-role-service-connector)

If you connected your services using the Service Connector, you don't need to complete this step. The following necessary configurations were handled for you:

- If you selected a managed identity when you created the connection, a system-assigned managed identity was created for your app and assigned the `Azure Event Hubs Data Sender` and `Azure Event Hubs Data Receiver` roles on the Event Hubs.

- If you chose to use a connection string, the connection string was added as an app environment variable.

##### [Azure portal](#tab/assign-role-azure-portal)

> [!NOTE]
> If you use Azure Spring Apps, use the following steps to assign roles in Azure Spring Apps.
>
> 1. In the Azure portal, navigate to **Azure Spring Apps**, then choose the app you use.
> 2. Select **Identity** on the navigation menu, and on the **System assigned** tab, select **Azure role assignments**.
> 3. Select **Add role assignments**, search for **Azure Event Hubs Data Sender** and **Azure Event Hubs Data Receiver**, select the matching result, and then select **Save**.

1. In the Azure portal, locate your Event Hubs namespace using the main search bar or the navigation pane.

1. On the Event Hubs overview page, select **Access control (IAM)** from the navigation menu.

1. On the **Access control (IAM)** page, select the **Role assignments** tab.

1. Select **Add** from the main menu and then **Add role assignment** from the drop-down menu.

   :::image type="content" source="media/migrate-kafka-to-passwordless-connection/migration-role-eventhubs.png" alt-text="Screenshot of Azure portal Access control (IAM) page of Event Hubs Namespace resource with Add role assignment menu option highlighted." lightbox="media/migrate-kafka-to-passwordless-connection/migration-role-eventhubs.png":::

1. Use the search box to filter the results to the desired role. For this example, search for **Azure Event Hubs Data Sender** and **Azure Event Hubs Data Receiver**, select the matching result, and then select **Next**.

1. Under **Assign access to**, select **Managed identity**, and then select **Select members**.

1. In the flyout, search for the subscription where hosting service is located. Then, select **All system-assigned managed identities**, and select the managed identity of your hosting service. Select the system assigned identity, and then select **Select** to close the flyout menu.

   > [!NOTE]
   > If you use Azure Kubernetes Service, select **User-assigned managed identities** and then select the managed identity of the Kubernetes cluster, which has a name with the following structure: `<your-kubernetes-cluster-name>-agentpool`.

1. Select **Next** a couple of times until you're able to select **Review + assign** to finish the role assignment.

##### [Azure CLI](#tab/assign-role-azure-cli)

To assign a role at the resource level using the Azure CLI, you can use the following commands:

```azurecli
az role assignment create \
    --assignee $AZURE_MANAGED_IDENTITY_ID \
    --role "Azure Event Hubs Data Receiver" \
    --scope $AZURE_EVENTHUBS_RESOURCE_ID
az role assignment create \
    --assignee $AZURE_MANAGED_IDENTITY_ID \
    --role "Azure Event Hubs Data Sender" \
    --scope $AZURE_EVENTHUBS_RESOURCE_ID
```

---

#### Test the app

After making these code changes, browse to your hosted application in the browser. Your app should be able to connect to the Azure Event Hubs for Kafka successfully. Keep in mind that it may take several minutes for the role assignments to propagate through your Azure environment. Your application is now configured to run both locally and in a production environment without the developers having to manage secrets in the application itself.

## Next steps

In this tutorial, you learned how to migrate an application to passwordless connections.

You can read the following resources to explore the concepts discussed in this article in more depth:

- [Authorize access to blob data with managed identities for Azure resources](/azure/storage/blobs/authorize-managed-identity)
- [Authorize access to blobs using Microsoft Entra ID](/azure/storage/blobs/authorize-access-azure-active-directory)
