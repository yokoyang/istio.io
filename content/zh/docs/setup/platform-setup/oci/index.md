---
title: Oracle Cloud 基础架构
description: 使用 Oracle Container 为 Istio 准备集群的说明。
weight: 60
skip_seealso: true
aliases:
    - /zh/docs/setup/kubernetes/prepare/platform-setup/oci/
    - /zh/docs/setup/kubernetes/platform-setup/oci/
keywords: [platform-setup,kubernetes,oke,oci,oracle]
owner: istio/wg-environments-maintainers
test: no
---

本页面最新更新时间为 2021年9月20日。

{{< boilerplate untested-document >}}

根据如下介绍，为 Istio 配置 OKE 集群环境。

## 创建 OKE 集群{$create-an-oke-cluster}

要创建一个OKE集群，您必须属于租户的管理员或被策略授予 `CLUSTER_MANAGE` 权限的组。

[创建一个 OKE 集群][Create]最简单的方法是使用[快速创建工作流程][Quick]可在[Oracle Cloud Infrastructure (OCI)控制台][Console]。其他方法包括[自定义创建工作流][Custom]和[Oracle Cloud Infrastructure (OCI) API][API]。

[OCI CLI][OCICLI]也可以通过下面的例子中的命令行创建集群:

{{< text bash >}}
$ oci ce cluster create \
      --name <oke-cluster-name> \
      --kubernetes-version <kubernetes-version> \
      --compartment-id <compartment-ocid> \
      --vcn-id <vcn-ocid>
{{< /text >}}

| Parameter             | Expected value                                              |
|-----------------------|------------------------------------------------------------ |
| `oke-cluster-name`    | A name to assign to your new OKE cluster                    |
| `kubernetes-version`  | A [supported version of Kubernetes][K8S] to deploy          |
| `compartment-ocid`    | The [OCID][CONCEPTS] of an existing [compartment][CONCEPTS] |
| `vcn-ocid`            | The [OCID][CONCEPTS] of an existing [virtual cloud network][CONCEPTS] (VCN) |

## 建立本地对 OKE 集群的访问{#setting-up-local-access-to-an-OKE-cluster}

从您的本地机器集群[安装 `kubectl`][kubectl]和[OCICLI][OCICLI](`OCI`)接入 OKE 集群。

使用以下 OCI CLI 命令创建或更新 `kubecconfig` 文件包括一个 `oci` 命令，它可以动态地生成和插入一个短期的认证令牌允许' kubectl '访问集群的:

{{< text bash >}}
$ oci ce cluster create-kubeconfig \
      --cluster-id <cluster-ocid> \
      --file $HOME/.kube/config  \
      --token-version 2.0.0 \
      --kube-endpoint [PRIVATE_ENDPOINT|PUBLIC_ENDPOINT]
{{< /text >}}

{{< tip >}}
虽然一个 OKE 集群可能暴露多个端点，但只有一个端点可以被攻击在 `kubecconfig` 文件中。
{{< /tip >}}

`kube-endpoint` 支持的值是 `PUBLIC_ENDPOINT` 或 `PRIVATE_ENDPOINT`。您可能还需要配置 SSH 隧道通过[bastion主机][bastion]访问只有私有端点的集群。

将 `cluster-ocid` 替换为目标OKE集群的[OCID][CONCEPTS]。

## 验证对集群的访问{#verify-access-to-the-cluster}

使用 `kubectl get nodes` 命令验证 `kubectl` 能够连接到集群:

{{< text bash >}}
$ kubectl get nodes
{{< /text >}}

你现在可以使用[`istioctl`](../../install/istioctl/)安装 Istio，
(Helm)(../../install/helm/)，或手动安装。

[CREATE]: https://docs.oracle.com/en-us/iaas/Content/ContEng/Tasks/contengcreatingclusterusingoke.htm
[API]: https://docs.oracle.com/en-us/iaas/Content/ContEng/Tasks/contengcreatingclusterusingoke_topic-Using_the_API.htm
[QUICK]: https://docs.oracle.com/en-us/iaas/Content/ContEng/Tasks/contengcreatingclusterusingoke_topic-Using_the_Console_to_create_a_Quick_Cluster_with_Default_Settings.htm
[CUSTOM]: https://docs.oracle.com/en-us/iaas/Content/ContEng/Tasks/contengcreatingclusterusingoke_topic-Using_the_Console_to_create_a_Custom_Cluster_with_Explicitly_Defined_Settings.htm
[OCICLI]: https://docs.oracle.com/en-us/iaas/Content/API/SDKDocs/cliinstall.htm
[K8S]: https://docs.oracle.com/en-us/iaas/Content/ContEng/Concepts/contengaboutk8sversions.htm
[KUBECTL]: https://kubernetes.io/docs/tasks/tools/
[CONCEPTS]: https://docs.oracle.com/en-us/iaas/Content/GSG/Concepts/concepts.htm
[BASTION]: https://docs.oracle.com/en-us/iaas/Content/ContEng/Tasks/contengdownloadkubeconfigfile.htm#localdownload
[CONSOLE]: https://docs.oracle.com/en-us/iaas/Content/GSG/Concepts/console.htm
