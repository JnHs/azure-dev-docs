---
title: Azure for developers overview
description: An overview of Azure from a developer's perspective.
keywords: azure billing, azure portal
ms.service: azure
ms.topic: overview
ms.date: 07/29/2024
ms.custom: overview
---

# Azure for developers overview

If you're new to developing applications for the cloud, this series of 7 articles is the best place to start.

* Part 1: **Azure for developers overview**
* Part 2: [Key Azure services for developers](azure-developer-key-services.md)
* Part 3: [Hosting applications on Azure](hosting-apps-on-azure.md)
* Part 4: [Connect your app to Azure services](connect-to-azure-services.md)
* Part 5: [How do I create and manage resources in Azure?](azure-developer-create-resources.md)
* Part 6: [Key concepts for building Azure apps](azure-developer-key-concepts.md)
* Part 7: [How am I billed?](azure-developer-billing.md)

Azure is a cloud platform designed to simplify the process of building modern applications. Whether you choose to host your applications entirely in Azure or extend your on-premises applications with Azure services, Azure helps you create applications that are scalable, reliable, and maintainable.

Azure supports the most popular programming languages in use today, including Python, JavaScript, Java, .NET and Go. With a comprehensive SDK library and extensive support in tools you already use like VS Code, Visual Studio, IntelliJ, and Eclipse, Azure is designed to take advantage of skills you already have and make you productive right away.

## Application development scenarios on Azure

You can incorporate Azure into your application in different ways depending on your needs. The following video provides a helpful overview of the most popular development scenarios for Azure developers:


> [!VIDEO e882f09e-efff-465d-a72e-1b430631e6bf]


To review, here are some common software development and deployment scenarios on Azure:

- **Application hosting on Azure -** Azure can host your entire application stack from web applications and APIs to databases to storage services. Azure supports a variety of hosting models from fully managed services to containers to virtual machines. When using fully managed Azure services, your applications can take advantage of the scalability, high-availability, and security built in to Azure.

- **Consuming cloud services from existing on-premises applications -** Existing on-premises apps can incorporate Azure services to extend their capabilities.  For example, an application could use Azure Blob Storage to store files in the cloud, Azure Key Vault to securely store application secrets, or [Azure AI Search](/azure/search/search-what-is-azure-search) to add full-text search capability. These services are fully managed by Azure and can be easily added to your existing apps without changing your current application architecture or deployment model.

- **Container based architectures -** Azure provides a variety of container based services to support your app modernization journey.  Whether you need a private registry for your container images, are containerizing an existing app for ease of deployment, deploying microservices based applications, or managing containers at scale, Azure has solutions that support your needs.

- **AI driven applications -** Build AI-powered applications on your terms, in your preferred software development language, in the cloud, on-premises, or at the edge. Get tools, services, and guidelines to help you apply AI responsibly in your applications, while also preserving data privacy, transparency, and trust Use Azure AI to add speech, vision, language, and decision capabilities to your applications, create chatbots, and uncover insights with AI-powered search.

- **Modern serverless architectures -** Azure Functions simplify building solutions to handle event-driven workflows, whether responding to HTTP requests, handling file uploads in Blob storage, or processing events in a queue.  You write only the code necessary to handle your event without worrying about servers or framework code.  Further, you can take advantage of over 250 connectors to other Azure and third-party services to tackle your toughest integration problems.


How do you implement those scenarios? The next article, "Key Azure services for developers", gives you several Azure service options to implement each scenario.

> [!div class="nextstepaction"]
> [Continue to part 2: Key Azure services for developers](azure-developer-key-services.md)
