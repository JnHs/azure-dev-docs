---
author: KarlErickson
ms.author: karler
ms.reviewer: edburns
ms.date: 09/20/2024
---

### Determine whether Java Message Service (JMS) Queues or Topics are in use

If your application is using JMS Queues or Topics, you'll need to migrate them to an externally-hosted JMS server. Azure Service Bus and the Advanced Message Queuing Protocol can be a great migration strategy for those using JMS. For more information, see [Use Java Message Service 1.1 with Azure Service Bus standard and AMQP 1.0](/azure/service-bus-messaging/service-bus-java-how-to-use-jms-api-amqp).

If JMS persistent stores have been configured, you must capture their configuration and apply it after the migration.

If you're using Oracle Message Broker, you can migrate this software to Azure virtual machines and use it as-is.
