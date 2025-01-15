---
author: KarlErickson
ms.author: haiche
ms.date: 04/29/2024
---

In this section, you create an Azure Traffic Manager for distributing traffic to your public facing applications across the global Azure regions. The primary endpoint points to the Azure Application Gateway in the primary WLS cluster, and the secondary endpoint points to the Azure Application Gateway in the secondary WLS cluster.

Create an Azure Traffic Manager profile by following the steps in [Quickstart: Create a Traffic Manager profile using the Azure portal](/azure/traffic-manager/quickstart-create-traffic-manager-profile). Skip the "Prerequisites" section. You just need the following sections: [Create a Traffic Manager profile](/azure/traffic-manager/quickstart-create-traffic-manager-profile#create-a-traffic-manager-profile), [Add Traffic Manager endpoints](/azure/traffic-manager/quickstart-create-traffic-manager-profile#add-traffic-manager-endpoints), and [Test Traffic Manager profile](/azure/traffic-manager/quickstart-create-traffic-manager-profile#test-traffic-manager-profile). Use the following steps as you go through these sections, then return to this article after you create and configure the Azure Traffic Manager:

1. When you reach the section [Create a Traffic Manager profile](/azure/traffic-manager/quickstart-create-traffic-manager-profile#create-a-traffic-manager-profile), in step 2 **Create Traffic Manager profile**, use the following steps:
   1. Save aside the unique Traffic Manager profile name for **Name** - for example, **tmprofile-ejb120623**.
   1. Save aside the new resource group name for **Resource group** - for example, **myResourceGroupTM1**.

1. When you reach the section [Add Traffic Manager endpoints](/azure/traffic-manager/quickstart-create-traffic-manager-profile#add-traffic-manager-endpoints), use the following steps:
   1. After the step **Select the profile from the search results**, use the following steps:
      1. Under **Settings**, select **Configuration**.
      1. For **DNS time to live (TTL)**, enter **10**.
      1. Under **Endpoint monitor settings**, for **Path**, enter **/weblogic/ready**.
      1. Under **Fast endpoint failover settings**, use the following values:
         * For **Probing internal**, enter **10**.
         * For **Tolerated number of failures**, enter **3**.
         * For **Probe timeout**, **5**.
      1. Select **Save**. Wait until it completes.
   1. In step 4 for adding the primary endpoint `myPrimaryEndpoint`, use the following steps:
      1. For **Target resource type**, select **Public IP address**.
      1. Select the **Choose public IP address** dropdown and enter the IP address of Application Gateway deployed in the **East US** WLS cluster that you saved aside previously. You should see one entry matched. Select it for **Public IP address**.
   1. In step 6 for adding a failover / secondary endpoint `myFailoverEndpoint`, use the following steps:
      1. For **Target resource type**, select **Public IP address**.
      1. Select the **Choose public IP address** dropdown and enter the IP address of Application Gateway deployed in the **West US** WLS cluster that you saved aside previously. You should see one entry matched. Select it for **Public IP address**.
   1. Wait for a while. Select **Refresh** until the **Monitor status** reaches the following states:
      * The primary endpoint is **Online**.
      * The failover endpoint is **Degraded**.

1. When you reach the section [Test Traffic Manager profile](/azure/traffic-manager/quickstart-create-traffic-manager-profile#test-traffic-manager-profile), use the following steps:
   1. In subsection [Check the DNS name](/azure/traffic-manager/quickstart-create-traffic-manager-profile#check-the-dns-name), in step 3, save aside the DNS name of your Traffic Manager profile - for example, `http://tmprofile-ejb120623.trafficmanager.net`.
   1. In subsection [View Traffic Manager in action](/azure/traffic-manager/quickstart-create-traffic-manager-profile#view-traffic-manager-in-action), use the following steps:
      1. In step 1 and 3, append **/weblogic/ready** to the DNS name of your Traffic Manager profile in your web browser - for example, `http://tmprofile-ejb120623.trafficmanager.net/weblogic/ready`. You should see an empty page without any error message.
      1. In step 4, you can't access **/weblogic/ready**, which is expected because the secondary cluster is stopped.
      1. Re-enable the primary endpoint.

Now, the primary endpoint has the states **Enabled** and **Online** and the failover endpoint has the states **Enabled** and **Degraded** in the Traffic Manager profile. Keep the page open for monitoring the endpoint status later.
