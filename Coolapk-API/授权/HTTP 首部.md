# HTTP 首部

> **HTTP header**（HTTP 首部，HTTP 头）表示在 HTTP 请求或响应中的用来传递附加信息的字段，修改所传递的消息（或者消息主体）的语义，或者使其更加精确。消息首部不区分大小写，开始于一行的开头，后面紧跟着一个 `':'` 和与之相关的值。字段值在一个换行符（CRLF）前或者整个消息的末尾结束。
## Client

| 参数名 | 内容 | 必要性 | 备注 |
| - | - | - | - |
| User-Agent | UA | 非必要 | 若不设置可能会有错误 |
| X-Requested-With | HTTP 请求 | 必要 | `XMLHttpRequest`: 返回 Json<br>`com.coolapk.market`: 返回HTML |

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
| username | 用户名 | 必要 | 需进行 URL 编码 |
| uid | 用户UID | 必要 |  |
| SESSID | SESSID | 非必要 |  |

获取方法(C#)：

```csharp
public static HttpCookieCollection GetCoolapkCookies(Uri uri)
{
    using (HttpBaseProtocolFilter filter = new HttpBaseProtocolFilter())
    {
        HttpCookieManager cookieManager = filter.CookieManager;
        return cookieManager.GetCookies(new Uri($"https://{uri.Host}"));
    }
}
```

## Miscellaneous

| 参数名 | 内容 | 必要性 | 备注 |
| - | - | - | - |
| X-Sdk-Int | SDK 版本 | 必要 | Android SDK 版本号，如 `30` |
| X-Sdk-Locale | 区域 | 未知 | 如 `zh-cn` |
| X-App-Id | App ID | 必要 | `com.coolapk.market` |
| X-App-Token | Token | 必要 | App Token [生成方法](#x-app-token) |
| X-App-Version | App 版本 | 必要 | 酷安版本号，如 `11.2` |
| X-App-Code | App 版本代号 | 必要 | 酷安 APP 版本代号，如 `2105201` |
| X-Api-Version | API 版本 | 必要 | `[6-13]` V5 之前无此项<br>V14 之后需特殊处理 |
| X-App-Device | 设备号 | 非必要 | Device Code [生成方法](#x-app-device) |
| X-Dark-Mode | 是否为夜间模式 | 非必要 | `0`: 否</br>`1`: 是 |
| X-App-Channel | 未知 | 非必要 | `coolapk` |
| X-App-Mode | 未知 | 非必要 | `universal` |
| X-App-Supported | 支持 App 版本代号 | 非必要 | 同酷安 APP 版本代号，如 `2105201` |

### 将要用到的函数

GetMD5 (C#):

```csharp
/// <summary>
/// Get the MD5 hash of the input string.
/// </summary>
/// <param name="input">The input string.</param>
/// <returns>The MD5 hash of the input string.</returns>
public static string GetMD5(this string input)
{
    // Create a new instance of the MD5CryptoServiceProvider object.
    using (MD5 md5Hasher = MD5.Create())
    {
        // Convert the input string to a byte array and compute the hash.
        byte[] data = md5Hasher.ComputeHash(Encoding.UTF8.GetBytes(input));
        string results = BitConverter.ToString(data).ToLowerInvariant();
        return results.Replace("-", string.Empty);
    }
}

/// <summary>
/// Get the Base64 string of the input string.
/// </summary>
/// <param name="input">The input string.</param>
/// <param name="isRaw"><see cref="true"/> to remove the padding characters; otherwise, <see cref="false"/>.</param>
/// <returns>The Base64 string of the input string.</returns>
public static string GetBase64(this string input, bool isRaw = false)
{
    byte[] bytes = Encoding.UTF8.GetBytes(input);
    string result = Convert.ToBase64String(bytes);
    if (!isRaw) { result = result.Replace("=", string.Empty); }
    return result;
}

/// <summary>
/// Inverts the order of the elements in a sequence.
/// </summary>
/// <param name="text">A sequence of values to reverse.</param>
/// <returns>A sequence whose elements correspond to those of the input sequence in reverse order.</returns>
public static string Reverse(this string text)
{
    char[] charArray = text.ToCharArray();
    Array.Reverse(charArray);
    return new string(charArray);
}
```

### X-App-Token

#### 版本 1

> [!WARNING]  
> 版本 1 已部分失效，请使用[版本 2](#版本-2)

生成方法 (C#):

```csharp
/// <summary>
/// Generate a token v1 with your device code.
/// </summary>
/// <param name="deviceCode">The device code.</param>
/// <returns>The generated token.</returns>
string GetCoolapkAppToken(string deviceCode)
{
    long timeStamp = DateTimeOffset.UtcNow.ToUnixTimeSeconds();
    string hex_timeStamp = $"0x{timeStamp:x}";

    // 时间戳加密
    string md5_timeStamp = timeStamp.ToString().GetMD5();
    string md5_deviceCode = deviceCode.GetMD5();

    string token = $"token://com.coolapk.market/c67ef5943784d09750dcfbb31020f0ab?{md5_timeStamp}${md5_deviceCode}&com.coolapk.market";
    string md5_token = token.GetBase64(true).GetMD5();

    string appToken = $"{md5_token}{md5_deviceCode}{hex_timeStamp}";
    return appToken;
}
```

#### 版本 2

生成方法 (C#):

```csharp
#r "nuget:BCrypt.Net-Next"

/// <summary>
/// Generate a token v2 with your device code.
/// </summary>
/// <param name="deviceCode">The device code.</param>
/// <returns>The generated token.</returns>
string GetTokenWithDeviceCode(string deviceCode)
{
    string timeStamp = DateTimeOffset.UtcNow.ToUnixTimeSeconds().ToString();

    string base64TimeStamp = timeStamp.GetBase64();
    string md5TimeStamp = timeStamp.GetMD5();
    string md5DeviceCode = deviceCode.GetMD5();

    string token = $"token://com.coolapk.market/dcf01e569c1e3db93a3d0fcf191a622c?{md5TimeStamp}${md5DeviceCode}&com.coolapk.market";
    string base64Token = token.GetBase64();
    string md5Base64Token = base64Token.GetMD5();
    string md5Token = token.GetMD5();

    string bcryptSalt = $"{$"$2y$10${base64TimeStamp}/{md5Token}".Substring(0, 31)}u";
    string bcryptResult = BCrypt.Net.BCrypt.HashPassword(md5Base64Token, bcryptSalt);

    string appToken = $"v2{bcryptResult.GetBase64()}";
    return appToken;
}
```

### X-App-Device

生成方法 (C#):

```csharp
string CreateDeviceCode(string aid, string mac, string manufactory, string brand, string model, string buildNumber)
{
    const string random = "Random";
    string device = string.Join("; ", aid == random ? RandHexString(16) : aid, string.Empty, string.Empty, mac == random ? RandMacAddress() : mac, manufactory, brand, model, buildNumber, "null");
    return device.GetBase64().Reverse();
}

string RandMacAddress()
{
    Random rand = new Random((int)DateTime.Now.Ticks);
    byte[] bytes = new byte[6];
    rand.NextBytes(bytes);
    return string.Join(":", bytes.Select(x => x.ToString("x2")));
}

string RandHexString(int length)
{
    Random rand = new Random((int)DateTime.Now.Ticks);
    byte[] bytes = new byte[length];
    rand.NextBytes(bytes);
    return BitConverter.ToString(bytes).ToUpperInvariant().Replace("-", string.Empty);
}
```

---

HTTP 首部示例：

```http
GET /apk/com.coolapk.market HTTP/1.1
Cookie: token=Token请勿公开，若泄露请立即修改密码！; username=wherewhere; uid=536381; SESSID=0576dbe9f539e72657286724daa1db2c126c2657
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
