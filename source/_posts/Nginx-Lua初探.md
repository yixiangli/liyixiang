---
title: Nginx+Lua初探
toc: true
date: 2016-12-28 16:57:36
category: 
	- 技术贴
	- web中间件
	- Nginx
tags: 
    - Nginx
    - Lua
---


# Nginx
不解释了，还是看我专门介绍Nginx的博客吧。。

make
make upgrade 平滑升级

## Lua
一门小众语言，资料不是很多，用起来也优点费劲

### 安装
<!--more-->
#### Nginx
安装Nginx
#### 安装LuaJIT 2.1

```
wget http://luajit.org/download/LuaJIT-2.1.0-beta2.tar.gz
tar zxf LuaJIT-2.1.0-beta2.tar.gz
cd LuaJIT-2.1.0-beta2
make PREFIX=/usr/local/luajit
make install PREFIX=/usr/local/luajit
```

#### ngx_devel_kit（NDK）模块（无需安装）
```
cd /usr/local/src
wget https://github.com/simpl/ngx_devel_kit/archive/v0.2.19.tar.gz
tar -xzvf v0.2.19.tar.gz
```

#### lua-nginx-module 模块（无需安装）
```
cd /usr/local/src
wget https://github.com/openresty/lua-nginx-module/archive/v0.10.2.tar.gz
tar -xzvf v0.10.2.tar.gz
```

#### 设置环境变量
```
export LUAJIT_LIB=/usr/local/luajit/lib
export LUAJIT_INC=/usr/local/luajit/include/luajit-2.1
```

#### 编译
```
1.进入nginx源码文件 进行编译
./configure --prefix=/usr/local/nginx --with-http_stub_status_module  --with-ld-opt='-ljemalloc' --with-ld-opt="-Wl,-rpath,/usr/local/luajit/lib" --add-module=/usr/local/src/ngx_devel_kit-0.2.19 --add-module=/usr/local/src/lua-nginx-module-0.10.2 

2.安装
make&make install
```

3.查看是否编译成功
```
location /hello_lua { 
      default_type 'text/plain'; 
      content_by_lua 'ngx.say("hello, lua")'; 
}
```
访问 /hello_lua


### lua坑爹之处

1.如何设置状态码
```
location = /hello {
    content_by_lua '
        ngx.say("hello")
        ngx.exit(404)
    ';
}
```

查看nginx.error.log 发现 attempt to set status 404 via ngx.exit after sending out the response status 200

返回码还是200 原因是当调用ngx.say/ngx.print时自动发送响应状态码 因此必须
```
location = /hello {
    content_by_lua '
        ngx.status(404)
        ngx.say("hello")
        ngx.exit(404)
    ';
}
```
且执行顺序靠前的代码中不能存在调用ngx.say/ngx.print的逻辑

### nginx配置编写事项

1.空格 空格 空格
先看一段代码
```
location /hello_lua { 
            default_type 'text/html'; 
            content_by_lua_file /usr/local/nginx/conf/test.lua;
            if($status = 302){
               proxy_pass http://lepush_api;
            }
}
```
报错  unknown directive "if($status"


2. if else 不是每个语言都支持
```
location /hello_lua { 
            default_type 'text/html'; 
            content_by_lua_file /usr/local/nginx/conf/test.lua;
            if($status = 302){
               proxy_pass http://lepush_api;
            }else{
               return 200;
            }
}
```
报错  unknown directive "else"

原因：
（1）nginx if 后必须加空格 否则if 和 ( 缺一个空格 ，如果没有空格他把if($status当成一个指令了，没有这个指令 
（2）不存在if else这种嵌套写法，如果需要可以曲线救国
```
location /hello_lua { 
            default_type 'text/html';
            content_by_lua_file /usr/local/nginx/conf/test.lua;
            echo $status;
            if ( $status = 404){
               proxy_pass http://lepush_api;
            }
            if ( $status = 200){
               return 200;
            }
}
```

3. nginx module
```
location /hello_lua { 
            default_type 'text/html';
            content_by_lua_file /usr/local/nginx/conf/test.lua;
            echo $status;
            if ( $status = 404){
               proxy_pass http://lepush_api;
            }
            if ( $status = 200){
               return 200;
            }
        }
```
报错  unknown directive "echo"
原因：
这个模块不包含在 Nginx 源码中，安装方法：

1. 首先下载模块源码：https://github.com/agentzh/echo-nginx-module/tags
2. 解压到某个路径，假设为 /path/to/echo-nginx-module
3. 使用下面命令编译并安装 Nginx


### 一些值得练手的功能
#### 根据post请求的某些参数做相应的服务处理

功能：vlink参数路由

nginx.conf
```
http {
    # 设置共享内存routes
    lua_shared_dict routes 10m;

    # Nginx启动时初始化共享内存
    init_by_lua_block {
       routes = ngx.shared.routes
       routes:set("1",391)
       routes:set("2",392)
       routes:set("0",390)
    }
}

    server {
        location /client/androidReg {
            access_by_lua_file /usr/local/nginx/conf/vlink.lua;
            error_page 391 = @391;
            error_page 392 = @392;
            error_page 390 = @390;
        }
       
        location @391 {
            proxy_pass http://lepush_api;
            proxy_redirect    off;
            proxy_set_header  Host             $host;
            proxy_set_header  X-Real-IP  $http_x_real_ip;
        }
        location @392 {
            proxy_pass http://lepush_api;
            proxy_redirect    off;
            proxy_set_header  Host             $host;
            proxy_set_header  X-Real-IP  $http_x_real_ip;
        }
        location @390 {
            return 200;
        }   
    }

```


vlink.lua
```
local method = ngx.var.request_method
local vlink = nil

--判断请求方式，获取code值
if method == "GET" then
   vlink = ngx.req.get_uri_args()["vlink"] or 0
elseif method == "POST" then
   ngx.req.read_body()
   vlink = ngx.req.get_post_args()["vlink"] or 0
end

-- 通过code寻找route返回码
local route = routes:get(vlink)
ngx.exit(route)
```










