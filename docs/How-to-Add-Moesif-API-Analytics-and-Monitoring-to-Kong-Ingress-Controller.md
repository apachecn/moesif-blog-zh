# 如何将 Moesif API 分析和监控添加到 Kong 入口控制器

> 原文：<https://www.moesif.com/blog/technical/kong/kubernetes/How-to-Add-Moesif-API-Analytics-and-Monitoring-to-Kong-Ingress-Controller/>

Kong 是一个流行的开源 API 网关，可以帮助你管理你的 API。有了 Kong，即使您有多个微服务，您也可以从一个集中的位置处理身份验证、速率限制、数据转换等事务。Kong 的核心是 NGINX，这是最流行的 HTTP 服务器之一。由于是开源的，Kong 很容易在内部部署，通常只需要几分钟，除了 Postgres 或 Cassandra 商店之外，不需要安装许多组件。

[Kong Ingress Controller](https://konghq.com/solutions/kubernetes-ingress/) 在 Kubernetes 集群中实现认证、转换和其他功能，无需停机。Kong Gateway 将 Kubernetes 集群与跨任何环境或平台运行的服务连接起来。通过与 Kubernetes 入口控制器集成，Kong 与 Kubernetes 的生命周期直接相关。随着应用程序的部署和新服务的创建，Kong 将自动对自身进行实时配置，以便为这些服务提供流量。Kong 和 Kong Ingress 控制器拥有完整的管理层和 API、目标和上游的实时配置，以及使用 Postgres 或 Cassandra 的持久、可扩展的状态存储，可确保每个 Kong 实例同步时没有延迟或停机时间。

[Moesif](https://www.moesif.com/?language=kong-api-gateway) 为 Kong 提供了一个插件，只需几分钟就可以开始使用 [API observability](/blog/api-engineering/api-observability/What-is-API-Observability/) ，这样您就可以专注于提供客户喜欢的功能，而不是处理构建自己的数据基础设施的维护成本。借助 Moesif，您可以了解 API 的使用方式和用户，确定哪些用户遇到了集成问题，并监控需要优化的终端。在本文中，我们将讨论 Moesif 给你的见解，以及如何将其与 Kong Ingress 控制器集成。然后，我们将讨论如何使用 Moesif 和 Kong 来最好地利用 API 可观测性。

![Metrics showing users with 400 errors](img/cf0a233ba85ee15d5bfda3767c117b63.png)

## 概观

Moesif Kong plugin 是一个代理，它收集指标并发送给 Moesif 收集网络。这使您能够全面了解 API 的使用情况，即使是跨不同的 Kong 实例和数据中心区域。Moesif 为工程团队提供了深刻的见解，以了解如何使用他们的 API，并快速解决复杂的问题。因为 Moesif 还跟踪谁在调用你的 API 以及它们是如何被访问的，所以产品驱动的团队可以了解整个客户之旅以及在哪里投入更多的资源。借助 API 可观察性，具有前瞻性思维的工程领导者可以通过对激活漏斗、保留报告等进行自助式分析，增强面向客户的团队的能力。Moesif 还分析您的 API 有效负载，以便进行故障排除和业务洞察，这样您就能够了解特定有效负载键的使用情况，等等。

## 设置 Kong 入口控制器

**请注意**只有在没有设置和运行入口控制器的情况下，才需要执行此步骤。如果您已经运行了 ingress 控制器，请参考[如何在现有 kong ingress 控制器运行的情况下加载 moesif 插件的步骤](#load-moesif-plugin-with-existing-kong-ingress-controller-running)。

我们将执行以下步骤来部署入口控制器并加载 Moesif 插件。

### 使用 Moesif 插件代码创建配置映射:

您将需要克隆 [kong-moesif-plugin](https://github.com/Moesif/kong-plugin-moesif) 并导航到`kong/plugins`目录以使用

```py
kubectl create configmap kong-plugin-moesif --from-file=moesif -n kong 
```

请确保在与将要安装 Kong 的名称空间相同的名称空间中创建它。

### 创建名称空间

如果已经创建了想要使用的名称空间，那么可以跳过这一步。但是请确保使用安装了 Kong 的同一个空间。

```py
{  "apiVersion":  "v1",  "kind":  "Namespace",  "metadata":  {  "name":  "kong",  "labels":  {  "name":  "kong"  }  }  } 
```

使用创建命名空间

```py
kubectl apply -f namespace.json 
```

### 添加孔图表

您需要添加 kong 图表，并通过 Helm 更新回购。

```py
# Add kong chart 
helm repo add kong https://charts.konghq.com

# Update helm repo
helm repo update 
```

### 部署 Kubernetes 入口控制器并加载 Moesif 插件:

使用 Helm，您可以通过将以下值添加到您的`values.yaml`文件来加载 Moesif 插件:

```py
# values.yaml
plugins:
  configMaps:
  - name: kong-plugin-moesif
    pluginName: moesif 
```

设置 Kubernetes 入口控制器并使用 Helm chart 加载 Moesif 插件非常简单:

```py
helm install kong kong/kong --namespace kong --set ingressController.installCRDs=false --values values.yaml 
```

### 设置环境变量

您将使用可访问 Kong 的 IP 地址设置一个环境变量。这将用于实际向 Kubernetes 集群发送请求。您将为 kong-proxy 创建一个隧道并获取 url。

如果您在 Amazon Elastic Kubernetes 服务(EKS)上部署 Kong Ingress，您将需要使用

```py
export PROXY_IP=$(kubectl get -o jsonpath="{.status.loadBalancer.ingress[0].hostname}" service -n kong kong-proxy) 
```

*请注意*如果您的`kong-proxy`服务被不同地命名，您必须相应地更改命令。

*请注意*如果您通过不同的方法部署 Kong Ingress，请参考[文档](https://docs.konghq.com/kubernetes-ingress-controller/2.0.x/deployment/overview/)关于如何设置环境变量。

您可以回显环境变量，使用-

```py
echo $PROXY_IP
# http://192.168.99.100:32728 
```

### 测试与孔的连接

这里假设`PROXY_IP`环境变量被设置为包含指向 Kong 的 IP 地址或 URL。如果您还没有这样做，请按照其中一个[部署指南](https://docs.konghq.com/kubernetes-ingress-controller/2.0.x/deployment/overview)来配置这个环境变量。

如果一切都设置正确，那么向 Kong 发出请求应该会返回 HTTP 404 Not Found。

```py
curl -i $PROXY_IP
HTTP/1.1 404 Not Found
Date: Fri, 21 Jun 2019 17:01:07 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Content-Length: 48
Server: kong/1.1.2

{"message":"no Route matched with those values"} 
```

这是意料之中的，因为孔还不知道如何代理请求。*请注意*如果你正在使用 minikube，确保 LoadBalancer 类型的服务通过`minikube tunnel`命令公开。它必须在单独的终端窗口中运行，以保持负载平衡器运行。终端中的 Ctrl-C 可用于终止进程，此时将清理网络路由。

接下来，我们将全局启用 [Moesif 插件](#enable-moesif-plugin-globally)。

### 在现有 Kong 入口控制器运行的情况下加载 Moesif 插件

**请注意**只有在没有执行上一步的情况下，才需要执行这一步。当你已经运行了 Kong Ingress 控制器并且想要启用 Moesif 插件时，你应该执行这个步骤。

本节假设您已经运行了 kong ingress 控制器，并希望加载 Moesif 插件。这也假设你已经有舵孔图可用，如果没有请添加[舵孔图](#add-kong-chart)。

### 使用 Moesif 插件代码创建配置映射:

您将需要克隆 [kong-moesif-plugin](https://github.com/Moesif/kong-plugin-moesif) 并导航到`kong/plugins`目录以使用

```py
kubectl create configmap kong-plugin-moesif --from-file=moesif -n kong 
```

请确保在与将要安装 Kong 的名称空间相同的名称空间中创建它。

### 部署 Kubernetes 入口控制器并加载 Moesif 插件:

使用 Helm，您可以通过将以下值添加到您的`values.yaml`文件来加载 Moesif 插件:

```py
# values.yaml
plugins:
  configMaps:
  - name: kong-plugin-moesif
    pluginName: moesif 
```

### 展开部署

您需要使用 Moesif 插件修补 kong 入口控制器部署，并部署部署。

```py
helm upgrade kong kong/kong --namespace kong --values values.yaml 
```

接下来，我们将全局启用 [Moesif 插件](#enable-moesif-plugin-globally)。

### 全局启用 Moesif 插件

**请注意**无论您执行了上述哪个步骤，您都需要执行该步骤。这是一个关键的部分，它将在全球范围内启用 Moesif 插件，并开始向 Moesif 记录数据。

要设置 KongClusterPlugin 资源，您需要 Moesif 应用程序 Id。您可以通过注册一个免费的 [Moesif 帐户](https://www.moesif.com)来获得一个，然后在入职时选择 Kong。Moesif 建议对所有的 Kong 实例和数据中心区域使用单一的应用程序 Id。这确保了无论物理拓扑如何，您都有一个 API 使用数据的统一视图。使用 Moesif 的高基数、高维度分析引擎，您仍然可以根据任意数量的属性进行分解。

Moesif 仍然建议为每个环境创建单独的应用程序 id，如*生产*、*试运行*和*开发*，以保持数据隔离。

创建一个`global-plugin.yaml`文件

```py
apiVersion: configuration.konghq.com/v1
kind: KongClusterPlugin
metadata:
  name: moesif
  annotations:
    kubernetes.io/ingress.class: kong
  labels:
    global: "true"
config:
  application_id: Your Moesif Application Id
  debug: false
plugin: moesif 
```

然后全局应用插件。

```py
kubectl  apply -f global-plugin.yaml 
```

*请注意*将标签 global 设置为“true”将在 Kong 中全局应用插件，这意味着它将为通过 Kong 代理的每个请求执行。

请确保在与安装 Kong 的名称空间相同的名称空间中创建。如果您的名称空间与`kong`不同，您应该相应地更改这个命令。

### 查看 Moesif 插件的运行情况

此时，如果您已经配置了 Kong 资源(服务),那么通过 Kong 代理的所有 api 调用都将记录在 Moesif 中。您不必执行接下来的步骤，因为这只是一个为想要创建示例服务的人创建服务/资源的示例。

### 建立回声服务

设置一个 echo-service 应用程序，演示如何使用 Kubernetes 入口控制器:

```py
kubectl apply -f https://bit.ly/echo-service -n kong 
```

请注意，`echo-service`文件没有被更改，它正在监听端口`80`。

请确保在与安装 Kong 的名称空间相同的名称空间中创建。如果您的名称空间与`kong`不同，您应该相应地更改这个命令。

### 创建一个使用 Moesif 插件的新入口资源

您可以对入口或服务资源进行注释，以指示 Kong 何时执行插件。您可以在全局范围内、在服务资源上或为特定用户配置插件。这将创建一个入口规则来代理先前创建的 echo 服务器。

```py
echo "
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: demo-example-com
  annotations:
    kubernetes.io/ingress.class: kong
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /bar
        backend:
          serviceName: echo
          servicePort: 80
"  |  kubectl  apply  -f  -  -n  kong 
```

*请注意*因为在我们之前创建的`echo-service`中，端口`80`是暴露的，所以`servicePort`被设置为`80`。如果您在不同的端口公开了服务，请相应地更改`servicePort`。

请确保在与安装 Kong 的名称空间相同的名称空间中创建。如果您的名称空间与`kong`不同，您应该相应地更改这个命令。

### 发送请求

一旦安装了 Moesif 插件，您的 API 流量应该会出现在 Moesif 的实时事件流中。您可以向使用配置的资源发送流量

```py
curl -i -H "Host: example.com" $PROXY_IP/bar/sample 
```

如果一切设置正确，您将看到 200 响应。下面是示例响应的样子-

```py
HTTP/1.1 200 OK
Content-Type: text/plain; charset=UTF-8
Transfer-Encoding: chunked
Connection: keep-alive
Date: Wed, 17 Nov 2021 21:31:04 GMT
Server: echoserver
X-Moesif-Transaction-Id: f8a8660e-1843-4844-89a7-67fbf2ac15ae
X-Kong-Upstream-Latency: 1
X-Kong-Proxy-Latency: 0
Via: kong/2.5.1

Hostname: echo-5fc5b5bc84-5s772

Pod Information:
	node name:	ip-192-148-47-105.us-west-2.compute.internal
	pod name:	echo-5fc5b5bc84-5s772
	pod namespace:	kong
	pod IP:	192.118.36.8

Server values:
	server_version=nginx: 1.12.2 - lua: 10010

Request Information:
	client_address=192.368.25.12
	method=GET
	real path=/bar/sample
	query=
	request_version=1.1
	request_scheme=http
	request_uri=http://example.com:8080/bar/sample

Request Headers:
	accept=*/*
	connection=keep-alive
	host=example.com
	user-agent=curl/7.68.0
	x-forwarded-for=192.178.50.41
	x-forwarded-host=example.com
	x-forwarded-path=/bar/sample
	x-forwarded-port=80
	x-forwarded-proto=http
	x-moesif-transaction-id=f8a8660e-1843-4844-89a7-67fbf2ac15ae
	x-real-ip=192.178.50.41

Request Body:
	-no body in request- 
```

![Real-time API logs](img/3b19552ff953983231af1e85b9d352fa.png)

## 如何使用 API 可观察性

### 工程度量

您可能感兴趣的第一件事是工程度量，比如我的 API 性能如何。这种类型的指标可以通过进入 Moesif 中的*事件*->-*时间序列*视图获得。然后，您可以选择第 90 个百分位数的延迟作为要绘制的指标。然后，您可以按 URI 路由分组，以了解哪些端点的性能最差。在这里，您可以通过 API 属性(如 route、verb)以及 HTTP 头和主体字段来过滤流量。

![90th percentile latency by API endpoint](img/1f5a1d7df40bdee9162936f47b9c18f4.png)

## 处理敏感数据

如果您的应用程序包含敏感数据，如医疗保健或财务数据，您可以通过以下两种方式之一实现数据合规性:

### 1.零知识安全

客户端加密具有低维护 SaaS 的优势，同时仍然让您能够控制您的数据。因为在发送给 Moesif 之前，您在内部对数据进行控制和加密，所以 Moesif 实际上无法访问您的数据。这只需要在您的基础设施中运行一个名为[安全代理](https://www.moesif.com/docs/platform/secure-proxy/)的小型设备，即可动态处理数据的加密/解密。

### 2.数据屏蔽

如果不想客户端加密，也可以直接使用插件屏蔽数据。这通过`config.request_body_masks`和`config.response_body_masks`配置选项来处理。例如，您可以屏蔽有效负载中可能出现的名为*密码*的字段。此外，如果您想一起删除日志请求和响应主体，您可以将`config.disable_capture_request_body`或`config.disable_capture_response_body`配置选项设置为`true`。

## 结束语

通过这种方式，Moesif 插件将捕获 API 请求和响应，并记录到 Moesif，以便通过 Kong 和处理应用程序周围所有其他服务的 Kong，实现深入的 API 可观察性和对 API 流量的实时监控。