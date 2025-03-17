---
title: Scale Azure OpenAI for JavaScript with Azure API Management
description: Learn how to add load balancing with Azure API Management to your JavaScript RAG chat app application to extend the chat app beyond the Azure OpenAI token and model quota limits. 
ms.date: 03/17/2025
ms.topic: get-started
ms.subservice: intelligent-apps
ms.custom: devx-track-js, devx-track-js-ai, build-2024-intelligent-apps
ms.collection: ce-skilling-ai-copilot
# CustomerIntent: As a JavaScript developer new to Azure OpenAI, I want to scale my Azure OpenAI capacity to avoid rate limit errors.
---

# Scale Azure OpenAI for JavaScript with Azure API Management

[!INCLUDE [aca-load-balancer-intro](../../ai/includes/scaling-load-balancer-introduction-azure-api-management.md)]

## Prerequisites

* Azure subscription.  [Create one for free](https://azure.microsoft.com/free/ai-services?azure-portal=true)

* [Dev containers](https://containers.dev/) are available for both samples, with all dependencies required to complete this article. You can run the dev containers in GitHub Codespaces (in a browser) or locally using Visual Studio Code.

    #### [Codespaces (recommended)](#tab/github-codespaces)

    * Only a [GitHub account](https://www.github.com/login) is required to use Codespaces

    #### [Visual Studio Code](#tab/visual-studio-code)

    * [Docker Desktop](https://www.docker.com/products/docker-desktop/) - start Docker Desktop if it's not already running
    * [Visual Studio Code](https://code.visualstudio.com/)
    * [Dev Container Extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)

    ---

[!INCLUDE [scaling-load-balancer-aca-procedure.md](../../ai/includes/scaling-load-balancer-procedure-azure-api-management.md)]

[!INCLUDE [py-deployment-procedure](../../ai/includes/redeploy-procedure-chat-azure-api-management.md)]

[!INCLUDE [capacity.md](../../ai/includes/scaling-load-balancer-capacity.md)]

[!INCLUDE [py-apim-cleanup](../../ai/includes/scaling-load-balancer-cleanup-azure-api-management.md)]

## Sample code

Samples used in this article include:

* [JavaScript chat app with RAG](https://github.com/Azure-Samples/azure-search-openai-javascript)
* [Load Balancer with Azure API Management](https://github.com/Azure-Samples/openai-apim-lb)

## Next steps

* [View Azure API Management diagnostic data in Azure Monitor](/azure/api-management/api-management-howto-use-azure-monitor#view-diagnostic-data-in-azure-monitor)
* Use [Azure Load Testing](/azure/load-testing/) to load test your chat app with Azure Load Testing service.
- Learn more about AZD in this template
    - [Tech community blog post](https://techcommunity.microsoft.com/blog/azuredevcommunityblog/kickstart-projects-with-azd-templates/4295235?WT.mc_id=javascript-155021-juliamuiruri)
    - [Reactor video series](https://www.youtube.com/watch?v=sp4_cWnlNQs&list=PLmsFUfdnGr3zzLIPdJsfdSwymIwOsyQ9Z&index=1)
