在讲配置之前，我们先来对http的header参数做一下区分，后面会用到

#####End-to-end 端到端头部 

此类头部字段会转发给 请求/响应 的最终接收目标。 
必须保存在由缓存生成的响应头部。 
必须被转发。

#####Hop-by-hop 逐跳首部 

此类头部字段只对单次转发有效。会因为转发给缓存/代理服务器而失效。 

HTTP 1.1 版本之后，如果要使用Hop-by-hop头部字段则需要提供Connection字段。
 
除了以下8个字段为逐跳字段，其余均为端到端字段。
 
Connection

Keep-Alive

Proxy-Authenticate

Proxy-Authenrization

Trailer

TE

Tranfer-Encoding

Upgrade


####1、在nginx的配置文件增加以下配置

```shell
server{
		.... //省略以上配置
		location /logs/ {
			 autoindex       off;
			 deny all;
		}
		location /autodeploy/ {
			proxy_set_header Host $host;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			proxy_pass http://tomcat_done.duolabao;
			expires 0;
			##重点以下3行配置
			proxy_http_version 1.1;
			proxy_set_header Upgrade $http_upgrade;
			proxy_set_header Connection "upgrade";
		}
		.... //省略以下配置
			        
}

```
重点就是这3个配置：
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";

第一行是说采用http1.1协议进行请求，之所以要用http1.1，是因为HTTP 1.1支持长连接（PersistentConnection）和请求的流水线（Pipelining）处理，在一个TCP连接上可以传送多个HTTP请求和响应，减少了建立和关闭连接的消耗和延迟，在HTTP1.1中默认开启Connection： keep-alive，一定程度上弥补了HTTP1.0每次请求都要创建连接的缺点。

另外需要注意的就是网络架构，如果从客户端到应用服务器如果存在多层代理，每级代理都需要该配置，原因从上面Hop-by-hop首部的说明可以很好理解



