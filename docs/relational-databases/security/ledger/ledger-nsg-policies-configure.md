---
title: Configure Azure confidential ledger for access to SQL MI
description: "Learn how to configure ledger to work with SQL MI." 
ms.service: sql-managed-instance
ms.subservice: ledger
ms.topic: how-to
---

# Configure ledger to work with Azure SQL Managed Instance

[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

After you've [enabled Azure confidential Ledger](./ledger-how-to-configure-ledger-database.md) as your provider for digest storage on your [Azure SQL Managed Instance](./sql-managed-instance-paas-overview.md), you will need to manually configure the VNET rules of your SQL managed instance to communicate with ledger.

In this article, you learn how to:

> [!div class="checklist"]
>
> - Configure your managed instance network security group (NSG) and routing table rules to allow traffic to Azure confidential ledger

## Permissions

Due to the sensitivity of data in a managed instance, the configuration to enable managed instance public endpoint requires a two-step process. This security measure adheres to separation of duties (SoD):

- The managed instance admin needs to enable the public endpoint on the managed instance. The managed instance admin can be found on the **Overview** page for your managed instance resource.
- A network admin needs to allow traffic to the managed instance using a network security group (NSG). For more information, review [network security group permissions](/azure/virtual-network/manage-network-security-group#permissions).

## Enable outbound NSG rules for ledger

In your newly provisioned ledger `Overview` page, capture the `Ledger Endpoint` hostname. Acquire the IP address of your ledger instance by using `ping` or a similar network tool to :

### Obtain Ledger Endpoint and Identity Service Endpoint IP addresses

```powershell
ping -a <ledgername>.confidential-ledger.azure.com
PING <ledgername>.confidential-ledger.azure.com (1.123.123.123) 56(84) bytes of data.
64 bytes from 1.123.123.123 (1.123.123.123): icmp_seq=1 ttl=105 time=78.7 ms
```

Similarly, perform the procedure for the ledger `Identity Service Endpoint`:

```powershell
ping identity.confidential-ledger.core.azure.com
PING part-0042.t-0009.t-msedge.net (13.107.246.70) 56(84) bytes of data.
64 bytes from 13.107.246.70 (13.107.246.70): icmp_seq=1 ttl=52 time=14.9 ms
```

### Add IP Addresses to the outbound NSG rules

These two IP addresses should be added to the outbound NSG rules of your SQL managed instance:

1. Navigate to the `Network security group` of your SQL managed instance
1. Navigate to the `Outbound security rules` blade
1. Add these two IP addresses as a new outbound rule:
    Select the **Outbound security rules** tab, and **Add** a rule that has higher priority than the **deny_all_inbound** rule with the following settings: </br> </br>

    |Setting  |Suggested value  |Description  |
    |---------|---------|---------|
    |**Source**     |Any IP address or Service tag         |<ul><li>For Azure services like Power BI, select the Azure Cloud Service Tag</li> <li>For your computer or Azure virtual machine, use NAT IP address</li></ul> |
    |**Source port ranges**     |* |Leave this to * (any) as source ports are typically dynamically allocated and as such, unpredictable |
    |**Destination**     |IP addresses         |Set this to IP addresses to add the two ledger addresses |
    |**Destination port ranges**     |1.123.123.123, 13.107.246.70         |Scope destination port to 3342, which is the managed instance public TDS endpoint |
    |**Service**     |HTTPS         |SQL Managed Instance will communicate with ledger over HTTPS |
    |**Action**     |Allow         |Allow outbound traffic from managed instance to ledger |
    |**Priority**     |1500         |Make sure this rule is higher priority than the **deny_all_inbound** rule |

    :::image type="content" source="./media/ledger/ledger-nsg-outbound-rules.png" alt-text="Image of NSG outbound rules to enable SQL to communicate with the ledger":::

### Add IP Addresses to the route table

These two IP addresses should also be added to the Route table:

1. Navigate to the `Route table` of your SQL managed instance
1. Navigate to the `Routes` blade
1. Add these two IP addresses as 2 new internet routes:
    |Setting  |Suggested value  |Description  |
    |---------|---------|---------|
    |**Destination type**     |IP Addresses       | |
    |**Destination IP addresses/CIDR ranges**     | 1.123.123.123/32 | Create a separate rule to add the identity service endpoint, `13.107.246.70/32` in this example |
    |**Destination**     |IP addresses         |Set this to IP addresses to add the two ledger addresses |
    |**Destination port ranges**     |1.123.123.123, 13.107.246.70         |Scope destination port to 3342, which is the managed instance public TDS endpoint |
    |**Next hop type**     |Internet         |SQL Managed Instance will communicate with ledger over HTTPS |

    :::image type="content" source="./media/ledger/ledger-routing-rule.png" alt-text="Screenshot of adding a route for the VNET to the ledger":::

## Confirm that routing is properly configured

You can confirm that the managed instance is now able to communicate with the ledger by running a [database verification](./ledger-database-verification.md). The query should report that `Ledger verification succeeded`.
