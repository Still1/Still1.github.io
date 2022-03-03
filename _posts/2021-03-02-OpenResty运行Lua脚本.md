---
title: OpenResty运行Lua脚本
tags: 
  - OpenResty
  - Lua
modify_date: 2021-03-02
---

## 代码例子

<!--more-->

`nginx.conf`

```
location /lua {
	default_type text/html;
	content_by_lua_file script/helloWorld.lua;
}
```

`helloWorld.lua`

```lua
ngx.say("<h1>Hello, world!</h1>");
```

