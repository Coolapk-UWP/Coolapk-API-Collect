---
permalink: /Coolapk-API/HTTP首部/HTTP首部.html
---
# HTTP 首部

> **HTTP header**（HTTP 首部，HTTP 头）表示在 HTTP 请求或响应中的用来传递附加信息的字段，修改所传递的消息（或者消息主体）的语义，或者使其更加精确。消息首部不区分大小写，开始于一行的开头，后面紧跟着一个 `':'` 和与之相关的值。字段值在一个换行符（CRLF）前或者整个消息的末尾结束。

---
- [HTTP 首部](#http-首部)
  - [Client](#client)
    - [User-Agent](#user-agent)
  - [Cookie](#cookie)
  - [Miscellaneous](#miscellaneous)
    - [将要用到的函数](#将要用到的函数)
    - [X-App-Token](#x-app-token)
    - [X-App-Device](#x-app-device)
---

## Client

| 参数名 | 内容 | 必要性 | 备注 |
| - | - | - | - |
| User-Agent | UA | 非必要 | 若不设置可能会有错误 |
| X-Requested-With | HTTP请求 | 必要 | `XMLHttpRequest`：返回Json<br>`com.coolapk.market`：返回HTML |

### User-Agent

配置方法：

| UA头 | 系统信息 | 设备信息 | 酷安版本 |
| - | - | - | - |
| Dalvik/2.1.0 | (`系统UA`) | (#Build; `设备厂商`; `设备名称`; `未知`; `系统版本号`) | +CoolMarket/`API版本` |

示例：

```
Dalvik/2.1.0 (Windows NT 10.0; Win64; x64; WebView/3.0) (#Build; HUAWEI; WRT-WX9; CoolapkUWP; 10.0) +CoolMarket/11
```

## Cookie

| 参数名 | 内容 | 必要性 | 备注 |
| - | - | - | - |
| token | Token | 必要 |  |
| username | 用户名 | 必要 |  |
| uid | 用户UID | 必要 |  |
| SESSID | SESSID | 必要 |  |

获取方法(C#)：

```
private static IEnumerable<(string name, string value)> GetCoolapkCookies(Uri uri)
        {
            using (var filter = new Windows.Web.Http.Filters.HttpBaseProtocolFilter())
            {
                var cookieManager = filter.CookieManager;
                foreach (var item in cookieManager.GetCookies(Uri("https://" + uri.Host)))
                {
                    if (item.Name == "uid" ||
                        item.Name == "username" ||
                        item.Name == "token")
                    {
                        yield return (item.Name, item.Value);
                    }
                }
            }
        }
```

## Miscellaneous

| 参数名 | 内容 | 必要性 | 备注 |
| - | - | - | - |
| X-Sdk-Locale | 区域 | 未知 | 如`zh-cn` |
| X-Api-Version | API 版本 | 必要 | `[6-11]` V5没有此项 |
| X-App-Token | Token | 必要 | [CoolapkTokenCrack](https://github.com/ZCKun/CoolapkTokenCrack "CoolapkTokenCrack") |
| X-App-Version | App 版本 | 必要 | 酷安版本号 如`11.2` |
| X-App-Id | App ID | 必要 | `com.coolapk.market` |
| X-App-Code | App 版本代号 | 必要 | 酷安APP 版本代号 如`2105201` |
| X-Sdk-Int | SDK 版本 | 必要 | Android SDK 版本号 如`30` |
| X-App-Device | 设备号 | 非必要 | MD5 但是算法似乎已经被封了 |
| X-Dark-Mode | 是否为夜间模式 | 非必要 | `0`：是</br>`1`：否 |
| X-App-Channel | 未知 | 非必要 | `coolapk` |
| X-App-Mode | 未知 | 非必要 | `universal` |

### 将要用到的函数

GetMD5(C#)：

```
static string GetMD5(string input)
        {
            using (var md5 = MD5.Create())
            {
                var r1 = md5.ComputeHash(Encoding.UTF8.GetBytes(input));
                var r2 = BitConverter.ToString(r1).ToLowerInvariant();
                return r2.Replace("-", "");
            }
        }
```

### X-App-Token

生成方法(C#)：

```
static string GetCoolapkAppToken()
        {
            var t = Utils.ConvertDateTimeToUnixTimeStamp(DateTime.Now);
            var hex_t = "0x" + Convert.ToString((int)t, 16);
            // 时间戳加密
            var md5_t = Utils.GetMD5($"{t}");
            var a = $"token://com.coolapk.market/c67ef5943784d09750dcfbb31020f0ab?{md5_t}${guid}&com.coolapk.market";
            var md5_a = Utils.GetMD5(Convert.ToBase64String(Encoding.UTF8.GetBytes(a)));
            var token = md5_a + guid + hex_t;
            return token;
        }
```

### X-App-Device

生成方法(C#)：

```
static string GetCoolapkAppDevice()
        {
            string guid = Guid.NewGuid().ToString();
            return Utils.GetMD5(guid);
        }
```

***目前通过此算法生成的设备号可能已经被封禁了，所有使用此设备号发送的动态都会被强制进行人工审核***

---

HTTP 首部示例：

```
GET /apk/com.coolapk.market HTTP/1.1
Cookie: token=330eb921UsGtvFqgJ1jxZS6a5Jc_FFJd1qemZLe2qQnhiaO23IcDGjlB1pyTykGyZ_yA7DNpSCnQUcQw49tYgdT4HSfPkgEyR1F0VJyVqIjhBOcMmH722gU_PVoFINpZWCSjuXXLQlwb_t23uFlGi4_NzBS20mnv9Vyju_cQZpIsS5pG_TcqHduC2TY1e16vLf1qnhwraSDEXRZ-rh1HBc8kjDTNXg; username=wherewhere; uid=536381; SESSID=0576dbe9f539e72657286724daa1db2c126c2657
X-Sdk-Locale: zh-CN
X-Api-Version: 11
X-Requested-With: com.coolapk.market
X-App-Token: ac0fb7999bc7253d06d5fabc50802d4b9bf8b3ae-4c5f-4573-93d5-1b86728220290x60b0c5cb
X-App-Version: 11.2
User-Agent: Dalvik/2.1.0 (Windows NT 10.0; Win64; x64; WebView/3.0) (#Build; HUAWEI; WRT-WX9; CoolapkUWP; 10.0) +CoolMarket/11.2-2105201-universal
X-App-Id: com.coolapk.market
X-App-Code: 2105201
X-Sdk-Int: 30
X-App-Device: aaf8dfcce34f988819a9f982c4d87edf
Host: www.coolapk.com
Connection: Keep-Alive
Cache-Control: no-cache
```

[返回主页](https://wherewhere.github.io/Coolapk-API-Collect/ "返回主页")