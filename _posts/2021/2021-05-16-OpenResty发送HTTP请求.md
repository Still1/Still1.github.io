---
title: OpenResty发送HTTP请求
tags: 
  - OpenResty
modify_date: 2021-05-16
---

## OpenResty第三方插件lua-resty-http发送HTTP请求

<!--more-->

插件GitHub地址https://github.com/ledgetech/lua-resty-http

将插件的`http.lua` `http_connect.lua` `http_headers.lua`三个文件加入openResty的`lualib/resty`目录中

`sendHttp.lua`

```lua
local http = require('resty.http')
local httpObj = http.new()
local res, err = httpObj:request_uri('http://127.0.0.1:5000/students', {
  method = 'GET'
})
if not res then
  ngx.log(ngx.ERR, "request failed: ", err)
  return
end
ngx.say(res.body)
```



