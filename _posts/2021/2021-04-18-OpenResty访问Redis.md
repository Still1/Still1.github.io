---
title: OpenResty访问Redis
tags: 
  - 实践案例
---

## OpenResty内置redis2-nginx-module访问Redis

<!--more-->

`nginx.conf`

```
location /setRedis {
    default_type text/html;
	redis2_pass 127.0.0.1:6379;
	set_unescape_uri $key $arg_key;
	set_unescape_uri $val $arg_val;
	redis2_query set $key $val;
}

location /getRedis {
    default_type text/html;
    redis2_pass 127.0.0.1:6379;
    set_unescape_uri $key $arg_key;
    redis2_query get $key;
}
```

## OpenResty内置lua-resty-redis访问Redis

`connectRedis.lua`

```lua
local redis = require 'resty.redis'
local redisObj = redis:new()

local ok, err = redisObj:connect('127.0.0.1', 6379)
if not ok then
  ngx.say('Connection error:', err)
  return
end

ok, err = redisObj:set('country', 'Canada')
if not ok then
  ngx.say('Setting error:', err)
  return
end

local response, errorResponse = redisObj:get('country')
if not response then
  ngx.say('Getting error', errorResponse)
  return
end

ngx.say('Country:', response);
```

