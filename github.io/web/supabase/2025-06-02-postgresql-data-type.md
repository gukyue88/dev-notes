---
layout: single
title: "[supabase] PostgreSQL 데이터 타입"
categories: web/supabase
---

#### 주요 데이터 타입

| PostgreSQL Data type | typescript | description                                         |
| -------------------- | ---------- | --------------------------------------------------- |
| bool                 | boolean    | boolean                                             |
| int2                 | number     | signed 2-bytes integer                              |
| int4                 | number     | signed 4-bytes integer                              |
| int8                 | bigint     | signed 8-bytes integer                              |
| float4               | number     | floating-point number (4bytes)                      |
| float8               | number     | floating-point number (8bytes)                      |
| numeric              | Decimal    | precise number (for calculation in bank or finance) |
| text                 | string     | long text (no limit, recommanded text data type)    |
| uuid                 | string     | 32 hex values                                       |
| timestamptz          | Date       | date + time + timezone (recommanded time data type) |
