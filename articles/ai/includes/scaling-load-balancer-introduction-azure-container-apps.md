---
ms.custom: overview
ms.topic: include
ms.date: 01/31/2024
ms.author: diberry
author: diberry
ms.service: azure
---

Learn how to add load balancing to your application to extend the chat app beyond the Azure OpenAI Service token and model quota limits. This approach uses Azure Container Apps to create three Azure OpenAI endpoints and a primary container to direct incoming traffic to one of the three endpoints.

This article requires you to deploy two separate samples:

* Chat app
    * If you haven't deployed the chat app yet, wait until after the load balancer sample is deployed.
    * If you already deployed the chat app once, change the environment variable to support a custom endpoint for the load balancer and redeploy it again.
    * The chat app is available in these languages:

        * [.NET](/dotnet/ai/get-started-app-chat-template)
        * [JavaScript](/azure/developer/javascript/get-started-app-chat-template)
        * [Python](/azure/developer/python/get-started-app-chat-template)

* Load balancer app

> [!NOTE]
> This article uses one or more [AI app templates](/azure/developer/ai/intelligent-app-templates) as the basis for the examples and guidance in the article. AI app templates provide you with well-maintained reference implementations that are easy to deploy. They help to ensure a high-quality starting point for your AI apps.

## Architecture for load balancing Azure OpenAI with Azure Container Apps

Because the Azure OpenAI resource has specific token and model quota limits, a chat app that uses a single Azure OpenAI resource is prone to have conversation failures because of those limits.

:::image type="content" source="../media/get-started-scaling-load-balancer-azure-container-apps/chat-app-original-architecuture.png" alt-text="Diagram that shows chat app architecture with the Azure OpenAI resource highlighted.":::

To use the chat app without hitting those limits, use a load-balanced solution with Container Apps. This solution seamlessly exposes a single endpoint from Container Apps to your chat app server.

:::image type="content" source="../media/get-started-scaling-load-balancer-azure-container-apps/chat-app-architecuture.png" alt-text="Diagram that shows chat app architecture with Azure Container Apps in front of three Azure OpenAI resources.":::

The container app sits in front of a set of Azure OpenAI resources. The container app solves two scenarios: normal and throttled. During a *normal scenario* where token and model quota is available, the Azure OpenAI resource returns a 200 back through the container app and app server.

:::image type="content" source="../media/get-started-scaling-load-balancer-azure-container-apps/intro-load-balance-normal-usage.png" alt-text="Diagram that shows a normal scenario. The normal scenario shows three Azure OpenAI endpoint groups with the first group of two endpoints getting successful traffic. ":::

When a resource is in a *throttled scenario* because of quota limits, the container app can retry a different Azure OpenAI resource immediately to fulfill the original chat app request.

:::image type="content" source="../media/get-started-scaling-load-balancer-azure-container-apps/intro-load-balance-throttled-usage.png" alt-text="Diagram that shows a throttling scenario with a 429 failing response code and a response header of how many seconds the client has to wait to retry.":::
