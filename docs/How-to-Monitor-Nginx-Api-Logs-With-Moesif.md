# 如何用 Moesif 监控 NGINX API 日志

> 原文：<https://www.moesif.com/blog/technical/nginx/How-to-Monitor-Nginx-Api-Logs-With-Moesif/>

随着 API 网关越来越受欢迎，API 网关在公开其 API 的应用程序和 API 的消费者之间提供认证、速率限制和访问控制，是 API 基础设施的基石。简单的架构将 API 相关功能捆绑在单个组件中，提供负载平衡、缓存和扩展能力，以确保高可用性。

[Nginx](https://www.nginx.com/) 是一款高性能、高可伸缩、高可用的 web 服务器、反向代理服务器和 web 加速器(结合了 HTTP 负载平衡器、内容缓存等功能)。NGINX 有一个模块化的、事件驱动的、异步的、单线程的体系结构，在通用服务器硬件上和跨多处理器系统上具有极好的伸缩性。

OpenResty 是一个成熟的 web 应用服务器，它捆绑了标准的 NGINX 核心、大量第三方 NGINX 模块以及它们的大部分外部依赖项。它具有一系列功能，如负载平衡、内容缓存等。

![Which RESTful endpoints have high 90th percentile latency?](img/ed1a1e6bf45c8c8ac7814eb344cf5151.png)

## 概观

Moesif Nginx OpenResty 插件是一个代理，它捕获指标并发送给 Moesif 收集网络。这使您能够全面了解 API 的使用情况，甚至跨不同的实例和数据中心区域。该插件在本地捕获指标并对其进行排队，这使得该插件能够将指标数据发送到 Moesif 收集网络的带外，而不会影响您的应用程序。Moesif 建议对 Nginx 中配置的所有服务和路由使用相同的 Moesif 应用程序 Id。但是，建议对每个隔离环境使用单独的 Moesif 应用程序 id，例如*生产*和*开发*环境。

在本文中，我们将讨论 Moesif 给你的见解，以及如何将它集成到你的 Nginx OpenResty 设置中。

## 使用 Nginx OpenResty 设置 Moesif API 分析

Moesif 在 Luarocks 中有一个[插件，可以让你了解 API 的使用情况并监控你的 API 流量。借助 Moesif，您可以了解 API 的使用方式和用户，确定哪些用户遇到了集成问题，并监控需要优化的终端。](https://luarocks.org/modules/moesif/lua-resty-moesif)

建议通过 Luarocks 安装 Moesif:

```py
luarocks install --server=http://luarocks.org/manifests/moesif lua-resty-moesif 
```

## 如何使用

编辑您的`nginx.conf`文件来配置 Moesif OpenResty 插件:如果需要，用正确的插件安装路径替换`/usr/local/openresty/site/lualib`。

```py
lua_shared_dict moesif_conf 2m;

init_by_lua_block {
   local config = ngx.shared.moesif_conf;
   config:set("application_id", "Your Moesif Application Id")
}

lua_package_path "/usr/local/openresty/luajit/share/lua/5.1/lua/resty/moesif/?.lua;;";

server {
  listen 80;
  resolver 8.8.8.8;

  # Default values for Moesif variables
  set $user_id nil;
  set $company_id nil;

  # Optionally, identify the user and the company (account)
  # from a request or response header, query param, NGINX var, etc
  header_filter_by_lua_block  { 
    ngx.var.user_id = ngx.req.get_headers()["User-Id"]
    ngx.var.company_id = ngx.req.get_headers()["Company-Id"]
  }

  access_by_lua_file /usr/local/openresty/luajit/share/lua/5.1/resty/moesif/read_req_body.lua;
  body_filter_by_lua_file /usr/local/openresty/luajit/share/lua/5.1/resty/moesif/read_res_body.lua;
  log_by_lua_file /usr/local/openresty/luajit/share/lua/5.1/lua/resty/moesif/send_event.lua;

  # Sample Hello World API
  location /api {
    add_header Content-Type "application/json";
    return 200 '{\r\n  \"message\": \"Hello World\",\r\n  \"completed\": true\r\n}';
  }
} 
```

## 支持的功能

Moesif 丰富了数据，使其更有条理。例如，不是在`GET /service_a/items/1 OR GET /service_a/items/2 OR GET /service_a/items/3`上过滤，Moesif 将检测剩余的模板，包括在`GET /service_a/items/:id`上启用单个过滤器的任何参数和 id。

## 屏蔽敏感数据

如果您有隐私要求或在高度管制的行业中，Moesif 提供了在发送到 Moesif 之前删除 HTTP 头或正文中的任何敏感数据或跳过记录正文有效负载的能力。

要查看 Nginx OpenResty 与 Moesif 的集成，您可以从 [GitHub](https://github.com/Moesif/lua-resty-moesif-example) 中克隆并运行这个示例应用程序

## 结论

通过这种方式，该插件将捕获 API 请求和响应，并记录到 Moesif，以便通过 Nginx 和 Nginx 处理应用程序周围的所有其他服务，轻松检查和实时调试您的 API 流量。