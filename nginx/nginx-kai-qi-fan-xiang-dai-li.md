# Nginx 开启反向代理

### 开启SSL实例

```
server { 
  listen 443 ssl; 
  server_name localhost; 
  ssl_certificate server.cert; 
  ssl_certificate_key server.key; 
  ssl_session_cache shared:SSL:1m; 
  ssl_session_timeout 5m; 
  ssl_ciphers HIGH:!aNULL:!MD5; 
  ssl_prefer_server_ciphers on; 
  location / { 
    root html; 
    index index.html index.htm; 
  }
}
```

## 主nginx配置2

```
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    include conf.d/*.conf;
    proxy_cache off;
}
```

## 使用vue-cli开发时

* [https://blog.csdn.net/YanzYan/article/details/125260166](https://blog.csdn.net/YanzYan/article/details/125260166)

```
  devServer: {
    host: "localhost",//配置本项目运行主机
    port: 8080,//配置本项目运行端口
    //配置代理服务器来解决跨域问题
    proxy: {
      // ‘/api’ 的作用就是 替换 baseURL 的，假如这里我用的 localhost：8080 ,前端请求时直接用 /api 就行了
      //  ‘/api’ 只在前端请求时添加，而后端不需要添加 /api 这个路径
      "/api": {
        target: "http://xxx.com", //配置要替换的后台接口地址
        changOrigin: true, //配置允许改变Origin
        ws: true, // proxy websockets
        pathRewrite: {
          "^/api": "/",
          //pathRewrite: {'^/api': '/'} 重写之后url为 http://localhost:8080/xxxx
          //pathRewrite: {'^/api': '/api'} 重写之后url为http://localhost:8080/api/xxxx
        },
      },
    },
  },
```
