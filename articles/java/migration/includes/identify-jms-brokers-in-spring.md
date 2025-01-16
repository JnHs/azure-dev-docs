---
author: KarlErickson
ms.author: karler
ms.date: 2/12/2020
---

#### JMS message brokers

Identify the broker or brokers in use by looking in the build manifest (typically, a **pom.xml** or **build.gradle** file) for the relevant dependencies.

For example, a Spring Boot application using ActiveMQ would typically contain this dependency in its **pom.xml** file:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-activemq</artifactId>
</dependency>
```

Spring Boot applications using commercial brokers typically contain dependencies directly on the brokers' JMS driver libraries. Here's an example from a **build.gradle** file:

```json
    dependencies {
      ...
      compile("com.ibm.mq:com.ibm.mq.allclient:9.4.0.5")
      ...
    }
```
