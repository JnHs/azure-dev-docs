---
ms.custom: devx-track-js, devx-track-ts, 
ms.topic: include
ms.date: 11/12/2024
# Used as part of /developer/ai/get-started-securing-your-ai-app
---

Use the following steps to create a new GitHub Codespace on the `main` branch of the [`Azure-Samples/openai-chat-app-quickstart-javascript`](https://github.com/Azure-Samples/openai-chat-app-quickstart-javascript) GitHub repository.

1. Right-click on the following button, and select _Open link in new window_. This action allows you to have the development environment and the documentation available for review.

 [![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://codespaces.new/Azure-Samples/openai-chat-app-quickstart-javascript)

1. On the **Create codespace** page, review and then select **Create new codespace**

    :::image type="content" source="github-create-codespace.png" lightbox="github-create-codespace.png" alt-text="Screenshot of the confirmation screen before creating a new codespace.":::

1. Wait for the codespace to start. This startup process can take a few minutes.

1. Sign in to Azure with the Azure Developer CLI in the terminal at the bottom of the screen.

    ```azdeveloper
    azd auth login
    ```

1. Copy the code from the terminal and then paste it into a browser. Follow the instructions to authenticate with your Azure account.

The remaining tasks in this article take place in the context of this development container.
