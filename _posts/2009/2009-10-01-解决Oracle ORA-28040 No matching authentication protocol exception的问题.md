---
title: 解决Oracle ORA-28040 No matching authentication protocol exception的问题
tags: [Oracle, 问题解决]
---

## 问题原因

旧的jdbc驱动不支持oracle 12c

## 解决方法

Set `SQLNET.ALLOWED_LOGON_VERSION=8` in the `oracle/network/admin/sqlnet.ora` file.

http://stackoverflow.com/questions/24100117/ora-28040-no-matching-authentication-protocol-exception