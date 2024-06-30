# nginx introduction

##  Web Server：

如 Nginx、Apache，只能拿來處理靜態資源，負載平衡、代理，所謂動態的資源，是指會把需求轉發到程式語言起的 Application Server，由Application Server 處理完後，再丟 response 回去，由 Web Server 進行回應，最後才回到 Client 端。

## Application Server

可以用程式語言建立出的 Server，且可以靜態跟動態解析。

## Nginx 作用

+ 靜態資源: Nginx 可以高效地提供静态内容，例如 HTML、CSS、JavaScript 和图像文件。

+ 反向代理: Nginx 可以作为反向代理服务器，转发客户端请求到后端服务器，并将后端服务器的响应返回给客户端。它支持负载均衡和缓存功能。

+ 負載平衡: Nginx 支持多种负载均衡算法，如轮询、加权轮询和 IP 哈希，以便将客户端请求均匀地分配到多个后端服务器上

+ HTTPS 和 SSL/TLS 支持：Nginx 支持 HTTPS，能够处理 SSL/TLS 加密，提供安全的通信通道。

+ 请求限速和连接限速：Nginx 可以限制请求速率和连接速率，以防止 DoS 攻击和滥用。