介绍：

利用 nginx 支持 SNI 分流特性，对 vless+tcp+tls 或 vless+tcp+xtls、trojan-go 或 trojan、HTTP/2 server 进行 SNI 分流（四层转发），实现除 v2ray 或 Xray 的 KCP 应用外共用 443 端口。其中 vless+tcp+tls 或 vless+tcp+xtls 为 WebSocket（WS） 提供分流转发；nginx 同时为 vless+tcp+tls 或 vless+tcp+xtls 与 trojan-go 或 trojan 提供回落服务，为 v2ray 或 Xray 的 gRPC 提供反向代理。包括应用如下：

1、E=vless+tcp+tls/xtls（回落/分流配置，TLS/XTLS由自己提供及处理。）

2、B=vless+ws+tls（TLS由vless+tcp+tls/xtls提供及处理，不需配置。另可改、可增其它WS类应用，参考对应的服务端单一应用配置示例。）

3、C=shadowsocks+xray-plugin/v2ray-plugin+tls（TLS由vless+tcp+tls/xtls提供及处理，不需配置。）

4、G=vless+grpc+tls（TLS由nginx提供及处理，不需配置。另可改、可增其它gRPC类应用，参考对应的服务端单一应用配置示例。）

5、A=vless+kcp+seed（可改成vmess+kcp+seed，或添加它。）

6、trojan-go或trojan（TLS由自己提供及处理。）

注意：

1、nginx 支持 SNI 分流，需要 nginx 包含 stream_core_module 和 stream_ssl_preread_module 模块。

2、采用 SNI 方式实现共用 443 端口，支持各自特色应用，但需多个域名来标记分流。

3、Xray 版本不小于 v1.4.0 或 v2ray 版本不小于v4.36.2，才支持 gRPC 传输方式。

4、因 trojan-go 或 trojan 不支持 Unix Domain Socket，故 nginx SNI 分流 trojan-go 或 trojan 仅启用端口转发；故与 trojan-go 或 trojan 共用 WEB 服务的 vless+tcp+xtls 或 vless+tcp+tls 回落也仅端口回落，即全部端口回落。

5、因 trojan-go 或 trojan 不支持 PROXY protocol，故 nginx SNI 分流启用 PROXY protocol 须特殊处理（具体见配置示例）；故与 trojan-go 或 trojan 共用 WEB 服务的 vless+tcp+xtls 或 vless+tcp+tls 回落不启用此项应用，即全部回落不启用此项应用。

6、trojan-go 完全兼容 trojan，服务端还有自己的特色：支持 trojan 应用与自己的 WebSocket 应用共存；支持 CDN 流量中转(基于 WebSocket over TLS)；支持使用 AEAD 对 trojan 协议流量进行二次加密(基于 Shadowsocks AEAD)。

7、nginx 支持 HTTP/2 server、gRPC proxy 及 H2C server，需要 nginx 包含 http_v2_module 和 http_ssl_module 模块。

8、nginx 支持 TLSv1.3，需要包含版本不小于 1.1.1 的 OpenSSL 软件库包和 http_ssl_module 模块。

9、nginx 支持 HTTP 功能块接收 PROXY protocol，需要 nginx 包含 http_realip_module 模块。

10、nginx 支持 H2C server，但不支持 HTTP/1.1 server 与 H2C server 共用一个端口或一个进程；故回落分成 http/1.1 回落与 h2 回落分别对应 nginx 的 HTTP/1.1 server 与 H2C server。

11、因 trojan-go 目前不支持 http/1.1 回落与 h2 回落分开，故 trojan-go 开启 Websocket 支持后只选 http/1.1 连接及 http/1.1 回落。

12、不要使用 ACME 客户端在当前服务器上以 HTTP-01 或 TLS-ALPN-01 验证方式申请与更新证书及密钥，因 HTTP-01 或 TLS-ALPN-01 验证方式申请与更新证书及密钥需监听 80 或 443 端口，从而与当前应用端口冲突。

13、配置1：采用端口分流、端口回落/分流、端口转发。配置2：采用进程分流（对应trojan-go/trojan除外）、端口回落/分流（对应vless+ws除外）、进程转发。配置3：采用进程分流（对应trojan-go/trojan除外）、端口回落/分流（对应vless+ws除外）、进程转发，且启用了 PROXY protocol（全部回落除外）。
