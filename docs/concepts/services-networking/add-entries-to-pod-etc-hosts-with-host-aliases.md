---
assignees:
- rickypai
- thockin
title: 使用 HostAliases 向 Pod /etc/hosts 文件添加条目
redirect_from:
- "/docs/user-guide/add-entries-to-pod-etc-hosts-with-host-aliases/"
- "/docs/user-guide/add-entries-to-pod-etc-hosts-with-host-aliases.md"
---

* TOC
{:toc}

<!--
Adding entries to a Pod's /etc/hosts file provides Pod-level override of hostname resolution when DNS and other options are not applicable. In 1.7, users can add these custom entries with the HostAliases field in PodSpec.

Modification not using HostAliases is not suggested because the file is managed by Kubelet and can be overwritten on during Pod creation/restart.
-->

当 DNS 配置以及其它选项不合理的时候，通过向 Pod 的 /etc/hosts 文件中添加条目，可以在 Pod 级别覆盖对主机名的解析。在 1.7 版本，用户可以通过 PodSpec 的 HostAliases 字段来添加这些自定义的条目。

建议通过使用 HostAliases 来进行修改，因为该文件由 Kubelet 管理，并且可以在 Pod 创建/重启过程中进行重写。 

<!--
## Default Hosts File Content

Lets start an Nginx Pod which is assigned an Pod IP:
-->

## 默认 hosts 文件内容

让我们从一个 Nginx Pod 开始，给该 Pod 分配一个 IP：

```
$ kubectl get pods --output=wide
NAME     READY     STATUS    RESTARTS   AGE    IP           NODE
nginx    1/1       Running   0          13s    10.200.0.4   worker0
```

<!--
The hosts file content would look like this:
->>

该 hosts 文件的内容看起来像下面这样：

```
$ kubectl exec nginx -- cat /etc/hosts
# Kubernetes-managed hosts file.
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
fe00::0	ip6-mcastprefix
fe00::1	ip6-allnodes
fe00::2	ip6-allrouters
10.200.0.4	nginx
```

<!--
by default, the hosts file only includes ipv4 and ipv6 boilerplates like `localhost` and its own hostname.

## Adding Additional Entries with HostAliases
-->

默认，hosts 文件只包含 ipv4 和 ipv6 的样板内容，像 `localhost` 和主机名称。

## 通过 HostAliases 增加额外的条目

<!--
In addition the the default boilerplate, we can add additional entries to the hosts file to resolve `foo.local`, `bar.local` to `127.0.0.1` and `foo.remote`, `bar.remote` to `10.1.2.3`, we can by adding HostAliases to the Pod under `.spec.hostAliases`:

{% include code.html language="yaml" file="hostaliases-pod.yaml" ghlink="/docs/concepts/services-networking/hostaliases-pod.yaml" %}

The hosts file content would look like this:
-->

除了默认的样板内容，我们可以向 hosts 文件添加额外的条目，将 `foo.local`、 `bar.local` 解析为`127.0.0.1`，将 `foo.remote`、 `bar.remote` 解析为 `10.1.2.3`，我们可以在 `.spec.hostAliases` 下为 Pod 添加 HostAliases。

{% include code.html language="yaml" file="hostaliases-pod.yaml" ghlink="/docs/concepts/services-networking/hostaliases-pod.yaml" %}

hosts 文件的内容看起来类似如下这样：

```
$ kubectl logs hostaliases-pod
# Kubernetes-managed hosts file.
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
fe00::0	ip6-mcastprefix
fe00::1	ip6-allnodes
fe00::2	ip6-allrouters
10.200.0.4	hostaliases-pod
127.0.0.1	foo.local
127.0.0.1	bar.local
10.1.2.3	foo.remote
10.1.2.3	bar.remote
```

<!--
With the additional entries specified at the bottom.

## Limitations

As of 1.7, Pods with hostNetwork enabled will not be able to use this feature. This is because kubelet only manages the hosts file for non-hostNetwork Pods. There are ongoing discussions to change this.
-->

在最下面额外添加了一些条目。

## 限制

在 1.7 版本，如果 Pod 启用 hostNetwork，那么将不能使用这个特性，因为 kubelet 只管理非 hostNetwork 类型 Pod 的 hosts 文件。目前正在讨论要改变这个情况。

<!--
## Why Does Kubelet Manage the Hosts File?

kubelet [manages](https://github.com/kubernetes/kubernetes/issues/14633) the hosts file for each container of the Pod to prevent Docker from [modifying](https://github.com/moby/moby/issues/17190) the file after the containers have already been started.

Because of the managed-nature of the file, any user-written content will be overwritten whenever the hosts file is remounted by Kubelet in the event of a container restart or a Pod reschedule. Thus, it is not suggested to modify the contents of the file.
-->

## 为什么 Kubelet 管理 hosts文件？

kubelet [管理](https://github.com/kubernetes/kubernetes/issues/14633) Pod 中每个容器的 hosts 文件，避免 Docker 在容器已经启动之后去 [修改](https://github.com/moby/moby/issues/17190) 该文件。

因为该文件是托管性质的文件，无论容器重启或 Pod 重新调度，用户修改该 hosts 文件的任何内容，都会在 Kubelet 重新安装后被覆盖。因此，不建议修改该文件的内容。
