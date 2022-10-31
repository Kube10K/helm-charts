# node-local-dns - NodeLocal DNS Cache helm chart

[eks_bp]: https://aws.github.io/aws-eks-best-practices/reliability/docs/networkmanagement/#use-nodelocal-dnscache
[vpc_limits]: https://docs.aws.amazon.com/vpc/latest/userguide/amazon-vpc-limits.html#vpc-limits-dns
[instructions]: https://kubernetes.io/docs/tasks/administer-cluster/nodelocaldns/
[ipvs_mode]: https://www.tigera.io/blog/comparing-kube-proxy-modes-iptables-or-ipvs/#:~:text=Background%3A%20iptables%20proxy%20mode
[iptables_mode]: https://www.tigera.io/blog/comparing-kube-proxy-modes-iptables-or-ipvs/#:~:text=Background%3A%20iptables%20proxy%20mode

The [EKS Best Practices][eks_bp] guide calls out to use the `NodeLocal
DNSCache` to prevent individual `coredns` pods from overwhelming the [VPC
Per-ENI DNS Packet Limit][vpc_limits].

The [Kubernetes-team supplied instructions][instructions] for running the
service are hard to follow and very error prone. Of course, they are also not
baked into any form of a Helm chart or other re-usable asset.

This chart launches the `node-local-dns` DaemonSet in your Kubernetes
environment. It supports operating in either [iptables mode][iptables_mode] or
[IPVS mode][ipvs_mode].

![Version: 0.0.3](https://img.shields.io/badge/Version-0.0.3-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 1.22.8](https://img.shields.io/badge/AppVersion-1.22.8-informational?style=flat-square)

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| coreDnsIP | string | `"172.0.20.10"` | (`string`) The default IP address for the `kube-dns` service in the `kube-system` namespace. This is used by the `node-local-dns` pods to push local cluster queries upstream. |
| healthPort | int | `9252` | (`int`) The port number that will be used to expose health information about the node-local-dns pods. This number must be unique to all hostNetwork:true services on the host, which is why it is configurable here. |
| image.pullPolicy | string | `"IfNotPresent"` |  |
| image.repository | string | `"registry.k8s.io/dns/k8s-dns-node-cache"` | (`string`) This is the default repository. |
| image.tag | `string` | `nil` | The image version we will run. |
| ipvsMode | bool | `true` | (`bool`) Whether or not the cluster is operating in IPVS mode. We default to true because that is how you should be running your clusters now. |
| kubeDnsNamespace | string | `"kube-system"` | The namespace that holds the original `kube-dns` Service object. This hostname is looked up via the Kubernetes API directly (not a DNS lookup), and therefore the `node-local-dns` pods need to be launched in the same namespace as the `kube-dns` service. We call this out here by explicitly defining the namespace for the resources, regardless of the `--namespace` param passed into the helm chart. ref: https://github.com/kubernetes/dns/blob/91ab9712e385a6dfb6c95e45492c693d6c81155c/cmd/node-cache/app/cache_app.go#L272-L296 |
| metricsPort | int | `9253` | (`int`) The port number that is used to expose custom metrics to prometheus for monitoring. This port must also be unique among all of the pods on the host that are running with hostNetwork:true. |
| nodeSelector | object | `{}` | (`map`) Map that controls which nodes this daemonset is applied to. |
| podAnnotations | object | `{}` | (`map`) Annotations to add to the Pods themselves. |
| podSecurityContext | object | `{}` | (`map`)` SecurityContext to be set on the pod itself. |
| priorityClassName | string | `"system-node-critical"` | (`string`) Ensure that these pods have the highest priority possible. |
| resources | object | `{"requests":{"cpu":"30m","memory":"50Mi"}}` | (`map`) The default configuration that caches around ~10k items is supposed to use ~30Mi of memory at max according to the documentation: https://kubernetes.io/docs/tasks/administer-cluster/nodelocaldns/#setting-memory-limits |
| serviceAccount.annotations | object | `{}` | (`map`) Annotations to add to the ServiceAccount. |
| serviceAccount.create | bool | `true` | (`bool`) Whether or not to create the node-local-dns ServiceAccount. |
| serviceAccount.name | string | `""` | (`string`) Optional hard-coded override for the name of the ServiceAccount. |
| serviceIP | string | `"169.254.20.11"` | (`string`) The IP address for the DNS service that will be mapped locally on each host to the node-local-dns pod. If operating in IPVS mode, this is also the value you need to pass into the `--cluster-dns-ip` kubelet parameters on-startup. |
| tolerations | list | `[{"key":"CriticalAddonsOnly","operator":"Exists"},{"effect":"NoExecute","operator":"Exists"},{"effect":"NoSchedule","operator":"Exists"}]` | (`map`) Configures which tolerations this pod will accept. By default we accept ALL because this is a core critical system pod that should run everywhere. |
| updateStrategy | object | `{"rollingUpdate":{"maxUnavailable":"10%"}}` | (`map`) Configure the deployment strategy for the pods, to ensure we don't roll out too many at once. |

----------------------------------------------
Autogenerated from chart metadata using [helm-docs v1.5.0](https://github.com/norwoodj/helm-docs/releases/v1.5.0)
