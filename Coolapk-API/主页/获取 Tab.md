---
permalink: /Coolapk-API/主页/获取Tab.html
---

# 获取 Tab 内容

*请求方式：POST*

认证方式：Cookie

> https://api.coolapk.com/v6/main/init

## JSON 回复

根对象：

| 字段 | 类型 | 内容 | 备注 |
| - | - | - | - |
| data | array | 信息本体 |  |

`data`数组：

| 项 | 类型 | 内容 | 备注 |
| - | - | - | - |
| 0 | obj | 热门搜索 |  |
| n | obj | Tab内容 |  |
| ... | ... | ... | ... |

---
[返回主页](https://wherewhere.github.io/Coolapk-API-Collect/ "返回主页")