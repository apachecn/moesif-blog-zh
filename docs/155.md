# 如何使用 Moesif 监控 3Scale API 网关日志

> 原文：<https://www.moesif.com/blog/technical/3scale/How-to-Monitor-3Scale-Api-Logs-With-Moesif/>

毫不奇怪，现代微服务架构中的应用程序通过使用各种 API(RESTful、GraphQL 等等)进行通信。API 网关在公开其 API 的应用程序和 API 的消费者之间提供认证、速率限制和访问控制，是 API 基础设施的基石。简单的体系结构将 API 相关功能捆绑在单个组件中，提供负载平衡、缓存和扩展能力，以确保高可用性。

[Nginx](https://www.nginx.com/) 是一款高性能、高可伸缩、高可用的 web 服务器、反向代理服务器和 web 加速器(结合了 HTTP 负载平衡器、内容缓存等功能)。NGINX 有一个模块化的、事件驱动的、异步的、单线程的体系结构，在通用服务器硬件上和跨多处理器系统上具有极好的伸缩性。

[3Scale](https://www.3scale.net/) 3scale API 管理平台基本上是一组模块化组件，可以部署在从托管在公共云中到内部部署的各种拓扑结构中，公司可以使用该产品来控制他们的 API，并将其置于他们可以使用 3Scale 平台实施的某种治理的控制之下。

![Which RESTful endpoints have high 90th percentile latency?](img/ed1a1e6bf45c8c8ac7814eb344cf5151.png)

## 设置具有 3 个刻度的 Moesif API 分析

Moesif 在 Luarocks 中有一个[插件，可以让你了解 API 的使用情况并监控你的 API 流量。借助 Moesif，您可以了解 API 的使用方式和用户，确定哪些用户遇到了集成问题，并监控需要优化的终端。](https://luarocks.org/modules/moesif/lua-resty-moesif)

建议通过 Luarocks 安装 Moesif:

```py
luarocks install --server=http://luarocks.org/manifests/moesif lua-resty-moesif 
```

## 如何使用

```py
lua_shared_dict moesif_conf 2m;

init_by_lua_block {
   local config = ngx.shared.moesif_conf;
   config:set("application_id", "Your Moesif Application Id")
   config:set("3scale_domain", "YOUR_ACCOUNT-admin.3scale.net")
   config:set("3scale_access_token", "Your 3scale Access Token")
}

lua_package_cpath ";;${prefix}?.so;${prefix}src/?.so;/usr/share/lua/5.1/lua/resty/moesif/?.so;/usr/share/lua/5.1/?.so;/usr/lib64/lua/5.1/?.so;/usr/lib/lua/5.1/?.so;/usr/local/openresty/luajit/share/lua/5.1/lua/resty?.so";
lua_package_path ";;${prefix}?.lua;${prefix}src/?.lua;/usr/share/lua/5.1/lua/resty/moesif/?.lua;/usr/share/lua/5.1/?.lua;/usr/lib64/lua/5.1/?.lua;/usr/lib/lua/5.1/?.lua;/usr/local/openresty/luajit/share/lua/5.1/lua/resty?.lua";

server {
  listen 80;
  resolver 8.8.8.8;

  # Customer identity variables that Moesif will read downstream
  # Set automatically from 3scale management API
  set $moesif_user_id nil;
  set $moesif_company_id nil;
  set $moesif_req_body nil;
  set $moesif_res_body nil;

  access_by_lua_file /usr/share/lua/5.1/lua/resty/moesif/read_req_body.lua;
  body_filter_by_lua_file /usr/share/lua/5.1/lua/resty/moesif/read_res_body.lua;
  log_by_lua_file /usr/share/lua/5.1/lua/resty/moesif/send_event_3Scale.lua;

  # Sample Hello World API
  location /api {
    add_header Content-Type "application/json";
    return 200 '{\r\n  \"message\": \"Hello World\",\r\n  \"completed\": true\r\n}';
  }
} 
```

Moesif 建议对 Nginx 中配置的所有服务和路由使用相同的 Moesif 应用程序 Id。但是，建议对每个隔离环境使用单独的 Moesif 应用程序 id，例如*生产*和*开发*环境。

## 洞察用户行为

API 分析围绕着*用户*(或者*公司*，如果你是 B2B 的话)如何体验你的应用。也被称为*用户行为分析*，每一个事件或行为都可以追溯到单个客户，并且可以通过一起查看多个事件来发现行为趋势。

通过关注以客户为中心的使用，我们可以发现产品问题，例如为什么用户停止使用您的 API，或者他们最常使用的功能或端点。

Moesif 自动从 3scale admin API 获取客户上下文。默认情况下，`id`字段用于从 3Scale 的应用程序 XML 实体中识别用户。我们可以用其他字段覆盖默认字段，包括`user_account_id`、`service_id`等等。Moesif 支持认证模式- API 密钥(user_key)和 App_ID 和 App_Key 对，以便您的客户对您的 API 进行认证，并将该事件链接到单个客户。

要查看 Moesif 的 3Scale 集成，您可以从 [GitHub](https://github.com/Moesif/moesif-3scale-api-gateway-example) 中克隆并运行这个示例应用程序

## 结论

通过这种方式，该插件将捕获 API 请求和响应，并记录到 Moesif，以便通过 3Scale 轻松检查和实时调试您的 API 流量。