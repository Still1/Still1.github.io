---
title: OpenResty运行Lua脚本
tags: 
  - 实践案例
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

