---
desc: 由 genpost (https://github.com/hidevopsio/genpost) 代码生成器生成
title: Istio 1.1 尝鲜记
date: 2019-03-20T15:15:23+08:00
author: gobomb
tags: [""]
---

## 环境

已经安装了 Kubernetes 集群，有1个 master 和4个 node。操作系统都是 CentOS Linux 7


## 下载 Istio 安装文件

`curl -L https://git.io/getLatestIstio | ISTIO_VERSION=1.1.0 sh -`
 
`export PATH="$PATH:/root/istio-1.1.0/bin"`


### 安装 Tiller

这里选择在 Helm 和 Tiller 的环境中使用 `helm install` 命令进行安装的方式。

`kubectl apply -f install/kubernetes/helm/helm-service-account.yaml`

假如已经安装过，结果如下：

![apply-install](/images/blog/istio-1.1/002.png)

`helm init --service-account tiller`

### 安装 istio-init chart

更新 Helm 的本地包缓存：

`helm repo add istio.io "https://gcsweb.istio.io/gcs/istio-prerelease/daily-build/release-1.1-latest-daily/charts/"`

![helm-add-repo](/images/blog/istio-1.1/001.png)

安装 istio-init chart，来启动 Istio CRD 的安装过程：

`helm install istio.io/istio-init --name istio-init --namespace istio-system`

![install-init](/images/blog/istio-1.1/004.png)

确认 Istio 的 CRD 都已经成功的提交给 Kubernetes API Server:

`kubectl get crds | grep 'istio.io\|certmanager.k8s.io' | wc -l`

55



### 安装 istio

`helm install install/kubernetes/helm/istio --name istio --namespace istio-system \
--set tracing.enabled=true \
--set jaeger.enabled=true \
--set grafana.enabled=true \
--set kiali.enabled=true \
 --set "kiali.dashboard.jaegerURL=http://tracing-istio-system.apps.cloud2go.cn" \
 --set "kiali.dashboard.grafanaURL=http://grafana-istio-system.apps.cloud2go.cn”`

输出结果如下：

```
NAME:   istio
LAST DEPLOYED: Wed Mar 20 02:19:04 2019
NAMESPACE: istio-system
STATUS: DEPLOYED

RESOURCES:
==> v1/ClusterRole
NAME                                     AGE
istio-citadel-istio-system               2m17s
istio-galley-istio-system                2m17s
istio-grafana-post-install-istio-system  2m17s
istio-ingressgateway-istio-system        2m17s
istio-mixer-istio-system                 2m17s
istio-pilot-istio-system                 2m17s
istio-reader                             2m17s
istio-sidecar-injector-istio-system      2m17s
kiali                                    2m17s
prometheus-istio-system                  2m17s


...这里省略一部分输出...

NOTES:
Thank you for installing istio.

Your release is named istio.

To get started running application with Istio, execute the following steps:
1. Label namespace that application object will be deployed to by the following command (take default namespace as an example)

$ kubectl label namespace default istio-injection=enabled
$ kubectl get namespace -L istio-injection

2. Deploy your applications

$ kubectl apply -f <your-application>.yaml

For more information on running Istio, visit:
https://istio.io/
```

查看 istio 的 pod：

![get-pod](/images/blog/istio-1.1/005.png)

## 遇到的问题：

1. istio-init 需要的镜像拉不下来

	`kubectl describe po istio-init-crd-10-vmq9p -n istio-system`

	```
Events:
  Type     Reason     Age                   From                  Message
  ----     ------     ----                  ----                  -------
  Normal   Scheduled  4m14s                 default-scheduler     Successfully assigned istio-system/istio-init-crd-10-vmq9p to 10.10.12.81
  Normal   Pulling    108s (x4 over 4m13s)  kubelet, 10.10.12.81  pulling image "gcr.io/istio-release/kubectl:master-latest-daily"
  Warning  Failed     83s (x4 over 3m43s)   kubelet, 10.10.12.81  Failed to pull image "gcr.io/istio-release/kubectl:master-latest-daily": rpc error: code = Unknown desc = Error response from daemon: Get https://gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
  Warning  Failed     83s (x4 over 3m43s)   kubelet, 10.10.12.81  Error: ErrImagePull
  Normal   BackOff    56s (x6 over 3m42s)   kubelet, 10.10.12.81  Back-off pulling image "gcr.io/istio-release/kubectl:master-latest-daily"
  Warning  Failed     43s (x7 over 3m42s)   kubelet, 10.10.12.81  Error: ImagePullBackOff
	```


	解决：到可以拉到的机器拉取到所有需要的镜像，再导入到集群

2. 安装完成后，给 kiali 创建 ingress 成功但是浏览器访问结果是404

	解决：直接访问域名不会自动跳转,需要加 `/kiali/console` 才能进入登陆界面

