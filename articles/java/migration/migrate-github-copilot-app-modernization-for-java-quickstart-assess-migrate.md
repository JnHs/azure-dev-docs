---
title: "Quickstart: Assess and Migrate a Java Project Using GitHub Copilot App Modernization for Java (Preview)"
titleSuffix: Azure
description: Shows you how to use GitHub Copilot App Modernization for Java (preview) to assess and migrate a Java project.
author: KarlErickson
ms.author: karler
ms.reviewer: xiading
ms.topic: quickstart
ms.date: 05/19/2025
ms.custom: devx-track-java
ms.service: azure-java
---

# Quickstart: assess and migrate a Java project using GitHub Copilot App Modernization for Java (preview)

This quickstart shows you how to use GitHub Copilot App Modernization for Java (preview) to assess and migrate a Java project. In this quickstart, you install and configure the extension, then assess and migrate a sample project. For example, you use a predefined formula to update an Azure SQL database connection to use Azure Managed Identity instead of a username and password.

## Prerequisites

- A GitHub account with [GitHub Copilot](https://github.com/features/copilot) enabled. All plans are supported, including the Free plan.
- The latest version of [Visual Studio Code](https://code.visualstudio.com/).
- The latest version of the [GitHub Copilot extension in Visual Studio Code](https://code.visualstudio.com/docs/copilot/overview).
- [GitHub Copilot App Modernization](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-app-mod-pack) extension pack, which bundles the following two extensions
  > - GitHub Copilot App Modernization for Java (preview)
  > - [GitHub Copilot App Modernization - upgrade for Java (preview)](/java/upgrade/overview)
- [AppCAT](https://aka.ms/appcat-install). This tool is required for the app assessment feature.
- App Modernization for Java does not require Java in your local environment. However to build your project successfully, please ensure you have the correct version of Java and Maven installed. We recommend [Microsoft Build of OpenJDK](https://learn.microsoft.com/en-us/java/openjdk/) and [Maven](https://maven.apache.org/download.cgi).
  
## Sign in to use Copilot and then install App Modernization

To use GitHub Copilot, sign in to your GitHub account in Visual Studio Code. Select the Copilot icon at the top of Visual Studio Code to access the GitHub Copilot pane. For more information about setting up GitHub Copilot, see [Set up GitHub Copilot in VS Code](https://code.visualstudio.com/docs/copilot/setup).

Then, use the following steps to install GitHub Copilot App Modernization for Java (preview):

1. In Visual Studio Code, open the **Extensions** view from the Activity Bar.
1. Search for **GitHub Copilot App Modernization** in the marketplace.
1. Select the **GitHub Copilot App Modernization for Java (preview)** extension or the **GitHub Copilot App Modernization** extension pack.
1. On the extension page, select **Install**.
1. Restart Visual Studio Code.

After installation completes, you should see a notification in the corner of Visual Studio Code confirming success.

For more information, see [Install a VS Code extension](https://code.visualstudio.com/docs/getstarted/extensions#_install-a-vs-code-extension).

## Configure

Use the following steps to configure Visual Studio Code:

1. In Visual Studio Code, press <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>P</kbd> and then select **Preferences: Configure Runtime Arguments**.

   :::image type="content" source="./media/migrate-github-copilot-app-modernization-for-java/configure-runtime-arguments.png" lightbox="./media/migrate-github-copilot-app-modernization-for-java/configure-runtime-arguments.png" alt-text="Screenshot of the Visual Studio Code dialog box that shows Preferences: Configure Runtime Arguments.":::

1. Add the following JSON snippet into the editor and then save:

   ```json
   "enable-proposed-api": ["Microsoft.migrate-java-to-azure"],
   ```

   :::image type="content" source="./media/migrate-github-copilot-app-modernization-for-java/config-api-for-extension.png" lightbox="./media/migrate-github-copilot-app-modernization-for-java/config-api-for-extension.png" alt-text="Screenshot of the Visual Studio Code editor that shows the JSON configuration file with the enable-proposed-api line highlighted.":::

1. Restart Visual Studio Code.

## Assess cloud readiness

Use the following steps to start your migration process with solution assessment. This assessment helps you understand what your cloud readiness challenges are and how impactful they are, provides recommended solutions. A solution recommendation includes references to set up Azure resources, add configurations, and make code changes.

1. Clone the [Java migration copilot samples](https://github.com/Azure-Samples/java-migration-copilot-samples) repository.

1. In Visual Studio Code, open the **mi-sql-public-demo** project folder in the samples repository.

1. On the sidebar, select the **App Modernization for Java** pane, and then, in the **Assessment** section, select **Assess**.

   :::image type="content" source="./media/migrate-github-copilot-app-modernization-for-java/assess-button-of-assessment.png" lightbox="./media/migrate-github-copilot-app-modernization-for-java/assess-button-of-assessment.png" alt-text="Screenshot of Visual Studio Code that shows the Assessment section with the Assess button highlighted.":::

   The GitHub Copilot chat window with agent mode is opened to call the modernization assessor to execute the App Modernization assessment.

1. Select **Continue** to confirm.

The modernization assessor now opens **assessment.md**. This file is the configuration for running AppCAT, which performs the app assessment. AppCAT asks for your confirmation to continue. You can examine its content and make changes there, if necessary.

The modernization assessor verifies your local environment first. If the AppCAT and its dependencies aren't installed, then you need to install them first. For more information, see [Azure Migrate application and code assessment for Java version 7 (Preview)](/azure/migrate/appcat/java-preview). After installation, it calls AppCAT to assess the current project. This step could take several minutes to complete.

Upon completion of the analysis, the modernization assessor produces a categorized view of cloud readiness issues in an opened summary report.

:::image type="content" source="./media/migrate-github-copilot-app-modernization-for-java/assessment-summary-report.png" lightbox="./media/migrate-github-copilot-app-modernization-for-java/assessment-summary-report.png" alt-text="Screenshot of the Visual Studio Code pane that shows the summary report of the assessment.":::

When reviewing the summary report, you can select **Propose Solution** and move to the next step, where you choose your desired solution per category/sub category.

Confirm the selection of the **Migrate to Azure SQL Database (SDK on Public Cloud)** solution by selecting **Confirm solution** to proceed to the migration step. Then, select **Migrate** for the **Migrate to Azure SQL Database (SDK on Public Cloud)** solution to move to the code remediation stage.

:::image type="content" source="./media/migrate-github-copilot-app-modernization-for-java/confirm-sql-solution.png" lightbox="./media/migrate-github-copilot-app-modernization-for-java/confirm-sql-solution.png" alt-text="Screenshot of the Visual Studio Code Solution pane with the Migrate to Azure SQL Database option selected and the Confirm Solution button highlighted.":::

## Apply a predefined formula

The migration Copilot provides predefined formulas for common migration scenarios that you might face when migrating to Azure. For example, with the **mi-sql-public-demo** sample, the Managed Identity formulas change the Azure SQL database connection to use Azure Managed Identity instead of a username and password.

1. In the **Solution Report**, select **Migrate**. The Copilot chat window opens with Agent Mode.

1. Select **Continue** repeatedly to confirm each tool action in the Copilot Chat window. The Copilot agent uses various tools for App Modernization and each tool requires confirmation to proceed.

1. After each step, manually input **continue** to confirm and proceed.

1. Wait for the code changes to be generated.

## Apply the Build-Fix tool

Use the following steps to apply the Build-Fix tool:

1. When the Java Application Build-Fix tool is suggested, select **Continue** to build the project and fix errors. This tool attempts to resolve any build errors in up to 10 iterations.

1. After the Build-Fix tool begins, select **Continue** to proceed and show progress.

1. After the process finishes, review the code changes and confirm them by selecting **Keep**.

## Next step

[Quickstart: create and apply your own formulas](migrate-github-copilot-app-modernization-for-java-quickstart-create-and-apply-your-own-formula.md)
