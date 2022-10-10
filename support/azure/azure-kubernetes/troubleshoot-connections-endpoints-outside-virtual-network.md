---
title: Troubleshoot connections to endpoints outside the virtual network
description: Troubleshoot connections to endpoints outside the virtual network (through the public internet) from an Azure Kubernetes Service (AKS) cluster.
ms.date: 9/30/2022
author: DennisLee-DennisLee
ms.author: v-dele
ms.reviewer: chiragpa
editor: v-jsitser
ms.service: container-service
#Customer intent: As an Azure Kubernetes user, I want to troubleshoot connections to endpoints outside the virtual network so that I don't experience outbound connection issues from an Azure Kubernetes Service (AKS) cluster.
---
# Troubleshoot connections to endpoints outside the virtual network

This article discusses how to troubleshoot connections to endpoints outside the virtual network (in other words, through the public internet) from a Microsoft Azure Kubernetes Service (AKS) cluster.

## Prerequisites

- [Azure CLI](/cli/azure/install-azure-cli).

- The Kubernetes [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/) tool, or a similar tool to connect to the cluster. To install kubectl by using Azure CLI, run the [az aks install-cli](/cli/azure/aks#az-aks-install-cli) command.

## Troubleshooting checklist

### Step 1: Do basic troubleshooting

The first step is to make sure that you can connect to public endpoints on the internet. For instructions, see [Basic troubleshooting of outbound AKS cluster connections](basic-troubleshooting-outbound-connections.md).

### Step 2: Determine the outbound type for the AKS cluster

To see what the outbound type of the AKS cluster is, run the [az aks show](/cli/azure/aks#az-aks-show) command:

```azurecli
az aks show --resource-group <resource_group> --name <cluster_name> --query "networkProfile.outboundType"
```

If the outbound type is `loadBalancer`, make sure that the route table that's associated with the AKS nodes has the default route to the internet. Details are shown in the following table.

| Source  | Address prefixes | Next hop type |
|:--------|:-----------------|:--------------|
| Default | 0.0.0.0/0        | Internet      |

If the outbound type is `userDefinedRouting`, make sure that the following conditions are met:

- The egress device (firewall or proxy) is reachable.

- The egress device allows the [required outbound traffic](/azure/aks/limit-egress-traffic#required-outbound-network-rules-and-fqdns-for-aks-clusters) from the cluster.

  To get the list of FQDNs that are allowed for your AKS cluster, run the [az aks egress-endpoints list](/cli/azure/aks/egress-endpoints#az-aks-egress-endpoints-list) command:

  ```azurecli
  az aks egress-endpoints list --resource-group <resource_group> --name <cluster_name>
  ```

If the outbound type is `managedNATGateway`, check whether the AKS subnet is associated to the NAT gateway by running the [az network nat gateway show](/cli/azure/network/nat/gateway#az-network-nat-gateway-show) command:

```azurecli
az network nat gateway show --resource-group <resource_group> --name <nat_gateway_name> --query "subnets[].id"
```

For more information about using a NAT gateway with AKS, see [Managed NAT gateway](/azure/aks/nat-gateway).

### Step 3: Examine the cURL output when you connect to the cluster

The cURL response codes can help you identify the issue type. After the response code becomes available, try to better understand the behavior of the issue. For more information about the HTTP status codes and the underlying behavior, see the following table.

| Information source | Link |
|:-|:-|
| Internet Assigned Numbers Authority (IANA) | [Hypertext Transfer Protocol (HTTP) status code registry](https://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml) |
| Mozilla | [HTTP response status codes](https://developer.mozilla.org/docs/Web/HTTP/Status) |
| Wikipedia | [List of HTTP status codes](https://wikipedia.org/wiki/List_of_HTTP_status_codes) |

The following HTTP status codes might indicate the listed issues.

| HTTP status code | Issue | Example |
|--|--|--|
| `4xx` | <ol> <li>An issue affects the client request.</li> <li>A network blocker exists between the client and the server.</li> </ol> | <ol> <li>The requested page doesn't exist, or the client doesn't have permission to access the page.</li> <li>Traffic is being blocked by a network security group or a firewall.</li> </ol> |
| `5xx` | An issue affects the server. | The application is down, or a gateway isn't working. |

### Step 4: Determine what happens if the outbound traffic is normally through a virtual appliance, but you bypass it instead

For quick testing, to determine whether the egress device (virtual appliance) causes the issue, you can temporarily allow all traffic to go through the internet. To configure this setup, you can change the default IP address and port route of `0.0.0.0`/`0` through the virtual appliance to go through the internet instead.

#### Is the issue intermittent?

You may experience intermittent outbound issues because of many reasons. For troubleshooting intermittent outbound connection issues, try the following checks:

#### Is the pod or node exhausted on resources?

Run the following code to check the utilization of the resources:

```bash
kubectl top pods
kubectl top nodes
```

**#### Is the operating system disk heavily used?**

You can navigate to the VMSS Portal, and select the Virtual Machine Scale Set for your AKS Cluster. Under the Metrics section, you can explore the [OS Disk and Data Disk metrics](https://learn.microsoft.com/en-us/azure/virtual-machines/disks-metrics).

You can get the Disk Metrics from the `Metrics` section in VMSS as:

- Scope: VMSS Name
- Metrics Namespace: Virtual Machine Host
- Metrics: OS and Data Disk Metric

AKS also provides Recommendations based on the Disk Utilization. You can navigate to the AKS Portal, and jump to `Advisor Recommendations` to see recommendations about the Disk usage.

If the OS disk is used heavily, consider using the following remedies:



\- Increase the OS disk size.

\- Switch to [Ephemeral OS disks](/azure/aks/cluster-configuration#ephemeral-os).



If these remedies don't solve the issue, analyze the process that does heavy read/write operations on the disk. Then, check whether you can move the actions to a data disk instead of the OS disk.

#### Is the source network address translation port exhausted?

If applications are making many outbound connections, they may exhaust the number of available ports on the outbound device's IP address. Follow [Standard load balancer diagnostics with metrics, alerts, and resource health](/azure/load-balancer/load-balancer-standard-diagnostics) to monitor the usage and allocation of your existing load balancer’s [source network address translation (SNAT) port](/azure/load-balancer/load-balancer-outbound-connections#what-are-snat-ports). Monitor to verify or determine the risk of [SNAT port exhaustion](/azure/load-balancer/load-balancer-outbound-connections#port-exhaustion).

Are you reaching or exceeding the maximum number of allocated SNAT ports? In that case, you can check your application to determine whether it's reusing existing connections. For more information, see [Design your applications to use connections efficiently](/azure/load-balancer/troubleshoot-outbound-connection#design-your-applications-to-use-connections-efficiently).

If you feel that the application is configured correctly, and you do need more SNAT ports than the default number of allocated ports, follow these steps:

1. Increase the number of public IPs on the egress device. If the egress device is the load balancer, you can [Increase the number of public IP addresses on the load balancer](/azure/aks/load-balancer-standard#scale-the-number-of-managed-outbound-public-ips).

1. [Increase the ports per node for your AKS worker nodes](/azure/aks/load-balancer-standard#configure-the-allocated-outbound-ports).

[!INCLUDE [Azure Help Support](../../includes/azure-help-support.md)]
