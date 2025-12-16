# ASP.NET IP Adresi Alma ve Kaydetme

Bu doküman ASP.NET projelerinde kullanıcının **IP adresini alma** ve **veritabanına kaydetme** işlemini anlatır.

---

## 1️⃣ ASP.NET Core – IP Adresi Alma

### Controller İçinde
```csharp
string ipAddress = HttpContext.Connection.RemoteIpAddress?.ToString();
```

### Proxy / Load Balancer Varsa (Önemli)
```csharp
string ipAddress = HttpContext.Request.Headers["X-Forwarded-For"].FirstOrDefault();

if (string.IsNullOrEmpty(ipAddress))
{
    ipAddress = HttpContext.Connection.RemoteIpAddress?.ToString();
}
```

---

## 2️⃣ ASP.NET MVC / WebForms (.NET Framework)

```csharp
string ipAddress = Request.UserHostAddress;
```

Proxy varsa:
```csharp
string ipAddress = Request.ServerVariables["HTTP_X_FORWARDED_FOR"];

if (string.IsNullOrEmpty(ipAddress))
{
    ipAddress = Request.ServerVariables["REMOTE_ADDR"];
}
```

---

## 3️⃣ IP Adresini Veritabanına Kaydetme (Entity Framework)

### Model
```csharp
public class Log
{
    public int Id { get; set; }
    public string IpAddress { get; set; }
    public DateTime CreatedDate { get; set; }
}
```

### Kaydetme
```csharp
var log = new Log
{
    IpAddress = ipAddress,
    CreatedDate = DateTime.Now
};

_context.Logs.Add(log);
_context.SaveChanges();
```

---

## 4️⃣ Login / İşlem Bazlı IP Loglama

```csharp
public IActionResult Login()
{
    string ip = HttpContext.Connection.RemoteIpAddress?.ToString();

    // Login işlemleri...

    LogIp(ip);
    return View();
}

void LogIp(string ip)
{
    _context.Logs.Add(new Log
    {
        IpAddress = ip,
        CreatedDate = DateTime.Now
    });
    _context.SaveChanges();
}
```

---

## 5️⃣ Dikkat Edilmesi Gerekenler ⚠️

- IPv6 adresler gelebilir → `VARCHAR(45)` kullan
- Proxy / Cloudflare / Nginx varsa `X-Forwarded-For` zorunlu
- IP adresi **kişisel veri** sayılır (KVKK / GDPR)

---

## 6️⃣ SQL Alan Önerisi

```sql
IpAddress VARCHAR(45) NOT NULL
```
# ASP.NET IP Kısıtlama (Allow / Deny)

Bu doküman ASP.NET ve ASP.NET Core projelerinde **IP adresine göre erişim kısıtlama** yöntemlerini anlatır.

---

## 1️⃣ ASP.NET Core – Basit IP Kontrolü (Controller Seviyesi)

```csharp
public IActionResult Index()
{
    var ip = HttpContext.Connection.RemoteIpAddress?.ToString();

    if (ip != "192.168.1.10")
        return Unauthorized("Bu IP adresi yetkili değil.");

    return View();
}
```

---

## 2️⃣ Allow List (IP Listesi)

```csharp
List<string> allowedIps = new()
{
    "192.168.1.10",
    "10.0.0.5"
};

string ip = HttpContext.Connection.RemoteIpAddress?.ToString();

if (!allowedIps.Contains(ip))
{
    return StatusCode(403);
}
```

---

## 3️⃣ Middleware ile Global IP Kısıtlama

### Middleware
```csharp
public class IpRestrictionMiddleware
{
    private readonly RequestDelegate _next;
    private readonly List<string> _allowedIps = new()
    {
        "127.0.0.1",
        "::1",
        "192.168.1.10"
    };

    public IpRestrictionMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task Invoke(HttpContext context)
    {
        var ip = context.Connection.RemoteIpAddress?.ToString();

        if (!_allowedIps.Contains(ip))
        {
            context.Response.StatusCode = StatusCodes.Status403Forbidden;
            await context.Response.WriteAsync("IP erişime kapalı");
            return;
        }

        await _next(context);
    }
}
```

### Program.cs
```csharp
app.UseMiddleware<IpRestrictionMiddleware>();
```

---

## 4️⃣ Proxy / Cloudflare Gerçek IP Alma

```csharp
string ip = context.Request.Headers["X-Forwarded-For"]
            .FirstOrDefault()?.Split(',').FirstOrDefault();

if (string.IsNullOrEmpty(ip))
{
    ip = context.Connection.RemoteIpAddress?.ToString();
}
```

---

## 5️⃣ IP Range (Basit)

```csharp
bool IsInRange(string ip)
{
    return ip.StartsWith("192.168.");
}
```

---

## 6️⃣ ASP.NET MVC / WebForms

```csharp
string ip = Request.UserHostAddress;

if (ip != "192.168.1.10")
{
    Response.StatusCode = 403;
    Response.End();
}
```

---

## 7️⃣ Güvenlik Notları

- IP tek başına güvenlik değildir
- VPN / Proxy ile aşılabilir
- Admin paneli ve API için uygundur
- IP adresi kişisel veridir (KVKK / GDPR)


---


# 🔒 Sadece Admin İçin IP Kısıtlama (ASP.NET Core)

Bu doküman, **sadece admin kullanıcıların** belirli IP adreslerinden erişebilmesini sağlar.
Normal kullanıcılar IP kısıtlamasından etkilenmez.

---

## 1️⃣ Senaryo

- Kullanıcı **Admin rolündeyse**
- IP adresi **Allow List** içinde değilse
- ➜ **403 Forbidden**

---

## 2️⃣ Admin IP Kısıtlama Middleware

```csharp
public class AdminIpRestrictionMiddleware
{
    private readonly RequestDelegate _next;
    private readonly List<string> _adminAllowedIps = new()
    {
        "127.0.0.1",
        "::1",
        "192.168.1.10"
    };

    public AdminIpRestrictionMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task Invoke(HttpContext context)
    {
        // Kullanıcı giriş yapmamışsa geç
        if (!context.User.Identity.IsAuthenticated)
        {
            await _next(context);
            return;
        }

        // Admin değilse geç
        if (!context.User.IsInRole("Admin"))
        {
            await _next(context);
            return;
        }

        // IP al
        string ip = context.Request.Headers["X-Forwarded-For"]
                        .FirstOrDefault()?.Split(',').FirstOrDefault();

        if (string.IsNullOrEmpty(ip))
        {
            ip = context.Connection.RemoteIpAddress?.ToString();
        }

        // IP kontrol
        if (!_adminAllowedIps.Contains(ip))
        {
            context.Response.StatusCode = StatusCodes.Status403Forbidden;
            await context.Response.WriteAsync("Admin erişimi bu IP için kapalı");
            return;
        }

        await _next(context);
    }
}
```

---

## 3️⃣ Program.cs

```csharp
app.UseAuthentication();
app.UseAuthorization();

app.UseMiddleware<AdminIpRestrictionMiddleware>();
```

> Middleware **Authentication ve Authorization'dan sonra** eklenmelidir.

---

## 4️⃣ Sadece Admin Controller Koruma (Alternatif)

```csharp
[Authorize(Roles = "Admin")]
public class AdminController : Controller
{
    public IActionResult Index()
    {
        string ip = HttpContext.Connection.RemoteIpAddress?.ToString();

        if (ip != "192.168.1.10")
            return Forbid();

        return View();
    }
}
```

---

## 5️⃣ appsettings.json’dan IP Okuma (Önerilen)

### appsettings.json
```json
"AdminAllowedIps": [
  "127.0.0.1",
  "::1",
  "192.168.1.10"
]
```

### Program.cs
```csharp
builder.Services.Configure<List<string>>(
    builder.Configuration.GetSection("AdminAllowedIps"));
```

---

## 6️⃣ Güvenlik Notları ⚠️

- IP + Role birlikte kullanmak en güvenlisidir
- Admin panel için idealdir
- VPN / Proxy riskine dikkat
- IP kişisel veridir (KVKK / GDPR)

---

## 7️⃣ Özet

| Kontrol | Var |
|------|-----|
| Admin rol kontrolü | ✅ |
| IP allow list | ✅ |
| Middleware | ✅ |
| Proxy desteği | ✅ |

---

# 🔑 Admin IP + 2FA Güvenliği (ASP.NET Core)

Bu doküman **Admin kullanıcılar için IP kısıtlama + İki Faktörlü Kimlik Doğrulama (2FA)** birlikte kullanımını anlatır.

Amaç:
- Admin **doğru IP'den** gelmeli
- Admin **2FA doğrulamasını geçmiş** olmalı

---

## 1️⃣ Güvenlik Akışı

1. Kullanıcı giriş yapar
2. Rolü **Admin** mi?
3. IP adresi **Allow List** içinde mi?
4. **2FA tamamlanmış mı?**
5. ➜ Admin paneline erişim

---

## 2️⃣ Admin IP + 2FA Middleware

```csharp
public class AdminSecurityMiddleware
{
    private readonly RequestDelegate _next;
    private readonly List<string> _allowedAdminIps = new()
    {
        "127.0.0.1",
        "::1",
        "192.168.1.10"
    };

    public AdminSecurityMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task Invoke(HttpContext context)
    {
        // Giriş kontrolü
        if (!context.User.Identity.IsAuthenticated)
        {
            await _next(context);
            return;
        }

        // Admin değilse geç
        if (!context.User.IsInRole("Admin"))
        {
            await _next(context);
            return;
        }

        // 2FA kontrolü
        var is2FaVerified = context.User.Claims
            .Any(c => c.Type == "amr" && c.Value == "mfa");

        if (!is2FaVerified)
        {
            context.Response.StatusCode = StatusCodes.Status403Forbidden;
            await context.Response.WriteAsync("2FA doğrulaması gerekli");
            return;
        }

        // Gerçek IP alma
        string ip = context.Request.Headers["X-Forwarded-For"]
            .FirstOrDefault()?.Split(',').FirstOrDefault();

        if (string.IsNullOrEmpty(ip))
        {
            ip = context.Connection.RemoteIpAddress?.ToString();
        }

        // IP kontrol
        if (!_allowedAdminIps.Contains(ip))
        {
            context.Response.StatusCode = StatusCodes.Status403Forbidden;
            await context.Response.WriteAsync("Admin IP erişimi reddedildi");
            return;
        }

        await _next(context);
    }
}
```

---

## 3️⃣ Program.cs

```csharp
app.UseAuthentication();
app.UseAuthorization();

app.UseMiddleware<AdminSecurityMiddleware>();
```

> Middleware mutlaka **Authentication + Authorization** sonrasında eklenmelidir.

---

## 4️⃣ ASP.NET Core Identity ile 2FA

### 2FA Aktif mi?
```csharp
bool twoFactorEnabled = await _userManager.GetTwoFactorEnabledAsync(user);
```

### 2FA Login Sonrası Claim Ekleme
```csharp
var claims = new List<Claim>
{
    new Claim("amr", "mfa")
};

await _signInManager.SignInWithClaimsAsync(user, true, claims);
```

---

## 5️⃣ Sadece Admin Controller (Ekstra Koruma)

```csharp
[Authorize(Roles = "Admin")]
public class AdminController : Controller
{
    public IActionResult Index()
    {
        return View();
    }
}
```

---

## 6️⃣ appsettings.json’dan IP Yönetimi (Önerilen)

```json
"AdminSecurity": {
  "AllowedIps": [
    "127.0.0.1",
    "::1",
    "192.168.1.10"
  ]
}
```

---

## 7️⃣ Güvenlik Notları ⚠️

- IP + Role + 2FA birlikte kullanımı **yüksek güvenlik**
- Admin panel için idealdir
- VPN / Proxy riskine karşı loglama önerilir
- IP adresi kişisel veridir (KVKK / GDPR)

---

## 8️⃣ Özet

| Güvenlik Katmanı | Durum |
|-----------------|-------|
| Admin rol | ✅ |
| IP allow list | ✅ |
| 2FA (MFA) | ✅ |
| Middleware | ✅ |


---

# ⏱ Rate Limit (ASP.NET Core)

Bu doküman ASP.NET Core uygulamalarında **IP bazlı Rate Limiting** (istek sınırlandırma) kullanımını anlatır.
Amaç: **Brute force, abuse ve DDoS benzeri istekleri engellemek**.

---

## 1️⃣ Senaryo

- Aynı IP
- Belirli süre içinde
- Belirlenen istek sayısını aşarsa
- ➜ **429 Too Many Requests**

---

## 2️⃣ .NET 7+ Built‑in Rate Limiting (ÖNERİLEN ✅)

### Program.cs
```csharp
using System.Threading.RateLimiting;

builder.Services.AddRateLimiter(options =>
{
    options.AddPolicy("IpRateLimit", context =>
    {
        string ip = context.Connection.RemoteIpAddress?.ToString() ?? "unknown";

        return RateLimitPartition.GetFixedWindowLimiter(
            partitionKey: ip,
            factory: _ => new FixedWindowRateLimiterOptions
            {
                PermitLimit = 100,          // 100 istek
                Window = TimeSpan.FromMinutes(1),
                QueueLimit = 0
            });
    });
});
```

### Middleware
```csharp
app.UseRateLimiter();
```

### Controller / Endpoint
```csharp
[EnableRateLimiting("IpRateLimit")]
public IActionResult Index()
{
    return Ok("İstek kabul edildi");
}
```

---

## 3️⃣ Admin Panel İçin Özel Rate Limit

```csharp
options.AddPolicy("AdminRateLimit", context =>
{
    if (!context.User.IsInRole("Admin"))
        return RateLimitPartition.GetNoLimiter("no-limit");

    string ip = context.Connection.RemoteIpAddress?.ToString();

    return RateLimitPartition.GetFixedWindowLimiter(
        ip,
        _ => new FixedWindowRateLimiterOptions
        {
            PermitLimit = 20,
            Window = TimeSpan.FromMinutes(1)
        });
});
```

---

## 4️⃣ Login / Auth Endpoint Rate Limit

```csharp
options.AddPolicy("LoginRateLimit", context =>
{
    string ip = context.Connection.RemoteIpAddress?.ToString();

    return RateLimitPartition.GetSlidingWindowLimiter(
        ip,
        _ => new SlidingWindowRateLimiterOptions
        {
            PermitLimit = 5,
            Window = TimeSpan.FromMinutes(1),
            SegmentsPerWindow = 5
        });
});
```

```csharp
[EnableRateLimiting("LoginRateLimit")]
public IActionResult Login()
{
    return Ok();
}
```

---

## 5️⃣ 429 Hatası Özelleştirme

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.OnRejected = async (context, token) =>
    {
        context.HttpContext.Response.StatusCode = 429;
        await context.HttpContext.Response.WriteAsync(
            "Çok fazla istek gönderdiniz, lütfen bekleyin.");
    };
});
```

---

## 6️⃣ Proxy / Cloudflare Gerçek IP

```csharp
app.UseForwardedHeaders(new ForwardedHeadersOptions
{
    ForwardedHeaders = ForwardedHeaders.XForwardedFor
});
```

---

## 7️⃣ Güvenlik Notları ⚠️

- Login endpoint’lerinde mutlaka rate limit kullan
- Admin panel için daha sıkı limit uygula
- Proxy arkasında gerçek IP’yi kullan
- Rate limit tek başına güvenlik değildir

---

## 8️⃣ Özet

| Alan | Öneri |
|----|----|
| Genel API | 100/dk |
| Login | 5/dk |
| Admin | 20/dk |
| Middleware | Built‑in RateLimiter |

---

# 🔐 Rate Limit + Otomatik IP Ban (ASP.NET Core)

Bu doküman **Rate Limit aşıldığında IP adresini otomatik olarak banlama** (geçici engelleme) sistemini anlatır.

Amaç:
- Brute-force saldırıları durdurmak
- Login / API abuse engellemek
- Şüpheli IP’leri otomatik karantinaya almak

---

## 1️⃣ Güvenlik Akışı

1. IP kısa sürede çok fazla istek atar
2. Rate limit aşılır
3. IP **Ban Listesine eklenir**
4. Ban süresi dolana kadar erişim engellenir
5. Süre bitince otomatik açılır

---

## 2️⃣ Ban Modeli

```csharp
public class BannedIp
{
    public int Id { get; set; }
    public string IpAddress { get; set; }
    public DateTime BannedUntil { get; set; }
}
```

---

## 3️⃣ IP Ban Service

```csharp
public class IpBanService
{
    private static readonly Dictionary<string, DateTime> _bannedIps = new();

    public bool IsBanned(string ip)
    {
        if (_bannedIps.TryGetValue(ip, out var bannedUntil))
        {
            if (DateTime.UtcNow < bannedUntil)
                return true;

            _bannedIps.Remove(ip);
        }
        return false;
    }

    public void Ban(string ip, TimeSpan duration)
    {
        _bannedIps[ip] = DateTime.UtcNow.Add(duration);
    }
}
```

---

## 4️⃣ Ban Middleware

```csharp
public class IpBanMiddleware
{
    private readonly RequestDelegate _next;
    private readonly IpBanService _banService;

    public IpBanMiddleware(RequestDelegate next, IpBanService banService)
    {
        _next = next;
        _banService = banService;
    }

    public async Task Invoke(HttpContext context)
    {
        string ip = context.Connection.RemoteIpAddress?.ToString();

        if (_banService.IsBanned(ip))
        {
            context.Response.StatusCode = 403;
            await context.Response.WriteAsync("IP geçici olarak engellendi");
            return;
        }

        await _next(context);
    }
}
```

---

## 5️⃣ Rate Limit + Ban Entegrasyonu

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.AddPolicy("LoginRateLimit", context =>
    {
        string ip = context.Connection.RemoteIpAddress?.ToString();

        return RateLimitPartition.GetFixedWindowLimiter(
            ip,
            _ => new FixedWindowRateLimiterOptions
            {
                PermitLimit = 5,
                Window = TimeSpan.FromMinutes(1)
            });
    });

    options.OnRejected = async (context, token) =>
    {
        var ip = context.HttpContext.Connection.RemoteIpAddress?.ToString();
        var banService = context.HttpContext.RequestServices.GetRequiredService<IpBanService>();

        banService.Ban(ip, TimeSpan.FromMinutes(15));

        context.HttpContext.Response.StatusCode = 429;
        await context.HttpContext.Response.WriteAsync(
            "Çok fazla istek. IP 15 dakika banlandı.");
    };
});
```

---

## 6️⃣ Program.cs

```csharp
builder.Services.AddSingleton<IpBanService>();

app.UseMiddleware<IpBanMiddleware>();
app.UseRateLimiter();
```

> ⚠️ **Ban middleware**, RateLimiter’dan önce çalışmalıdır.

---

## 7️⃣ Login Endpoint Örneği

```csharp
[EnableRateLimiting("LoginRateLimit")]
public IActionResult Login()
{
    return Ok();
}
```

---

## 8️⃣ Geliştirme Önerileri

- Ban listesini **Redis / DB**’de tut
- Ban süresini kademeli artır (15dk → 1s → 24s)
- Admin IP’leri whitelist yap
- Ban loglarını sakla

---

## 9️⃣ Güvenlik Notları ⚠️

- IP ban tek başına yeterli değildir
- VPN / botnet saldırılarına karşı sınırlıdır
- 2FA + Rate Limit birlikte önerilir
- KVKK/GDPR kapsamında IP kişisel veridir

---

## 10️⃣ Özet

| Özellik | Var |
|------|-----|
| Rate Limit | ✅ |
| Otomatik IP Ban | ✅ |
| Süreli Ban | ✅ |
| Middleware | ✅ |

---

# 🚨 Login Brute-Force Koruması (ASP.NET Core)

Bu doküman **login endpoint’lerini brute-force saldırılarına karşı korumak** için
Rate Limit + Otomatik IP Ban + Hesap kilitleme yaklaşımlarını içerir.

---

## 1️⃣ Tehdit Modeli

- Aynı IP’den çok sayıda login denemesi
- Yanlış parola denemeleri
- Bot / script saldırıları

Amaç:
- IP’yi geçici olarak engellemek
- Hesabı kilitlemek
- Saldırıyı erken durdurmak

---

## 2️⃣ Login Rate Limit (IP Bazlı)

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.AddPolicy("LoginRateLimit", context =>
    {
        string ip = context.Connection.RemoteIpAddress?.ToString();

        return RateLimitPartition.GetSlidingWindowLimiter(
            ip,
            _ => new SlidingWindowRateLimiterOptions
            {
                PermitLimit = 5,
                Window = TimeSpan.FromMinutes(1),
                SegmentsPerWindow = 5
            });
    });
});
```

```csharp
[EnableRateLimiting("LoginRateLimit")]
public IActionResult Login(LoginModel model)
{
    return Ok();
}
```

---

## 3️⃣ Otomatik IP Ban (Rate Limit Sonrası)

```csharp
public class IpBanService
{
    private static readonly Dictionary<string, (int Count, DateTime Until)> _ips = new();

    public void RegisterFailure(string ip)
    {
        if (!_ips.ContainsKey(ip))
            _ips[ip] = (0, DateTime.UtcNow);

        var data = _ips[ip];
        data.Count++;

        if (data.Count >= 10)
            data.Until = DateTime.UtcNow.AddMinutes(15);

        _ips[ip] = data;
    }

    public bool IsBanned(string ip)
        => _ips.ContainsKey(ip) && _ips[ip].Until > DateTime.UtcNow;
}
```

---

## 4️⃣ Ban Middleware

```csharp
public class LoginBanMiddleware
{
    private readonly RequestDelegate _next;
    private readonly IpBanService _banService;

    public LoginBanMiddleware(RequestDelegate next, IpBanService banService)
    {
        _next = next;
        _banService = banService;
    }

    public async Task Invoke(HttpContext context)
    {
        string ip = context.Connection.RemoteIpAddress?.ToString();

        if (_banService.IsBanned(ip))
        {
            context.Response.StatusCode = 403;
            await context.Response.WriteAsync("Çok fazla hatalı giriş. IP geçici olarak engellendi.");
            return;
        }

        await _next(context);
    }
}
```

---

## 5️⃣ Login Başarısızlığında IP Sayacı Artırma

```csharp
if (!loginSuccess)
{
    string ip = HttpContext.Connection.RemoteIpAddress?.ToString();
    _ipBanService.RegisterFailure(ip);
}
```

---

## 6️⃣ ASP.NET Core Identity – Hesap Kilitleme (ÖNERİLEN ✅)

```csharp
var result = await _signInManager.PasswordSignInAsync(
    model.Username,
    model.Password,
    false,
    lockoutOnFailure: true);
```

### Identity Ayarları
```csharp
builder.Services.Configure<IdentityOptions>(options =>
{
    options.Lockout.MaxFailedAccessAttempts = 5;
    options.Lockout.DefaultLockoutTimeSpan = TimeSpan.FromMinutes(15);
});
```

---

## 7️⃣ Program.cs Sıralama

```csharp
builder.Services.AddSingleton<IpBanService>();

app.UseMiddleware<LoginBanMiddleware>();
app.UseRateLimiter();
```

---

## 8️⃣ Güçlü Koruma İçin Öneriler ⚠️

- Login’de **rate limit zorunlu**
- Yanlış girişleri **logla**
- Admin login için **daha sıkı limit**
- Captcha + 2FA ekle
- Ban süresini kademeli artır

---

## 9️⃣ Özet

| Koruma Katmanı | Var |
|--------------|-----|
| Rate Limit | ✅ |
| IP Ban | ✅ |
| Hesap Kilitleme | ✅ |
| Middleware | ✅ |
| Identity Desteği | ✅ |

---

# 🤖 CAPTCHA Entegrasyonu (Login Brute-Force Koruması) — ASP.NET Core

Bu doküman, login formuna CAPTCHA ekleyip **sunucu tarafında doğrulayarak** brute-force saldırılarını zorlaştırır.

Aşağıda 2 popüler seçenek var:
- **Cloudflare Turnstile** (genelde daha az sürtünme)
- **Google reCAPTCHA** (yaygın)

> Not: **Client widget tek başına yeterli değildir**. Mutlaka **server-side verify** yapılmalıdır. (Turnstile Siteverify / reCAPTCHA siteverify)

---

## Seçenek A — Cloudflare Turnstile (Önerilen)

### 1) appsettings.json
```json
"Turnstile": {
  "SiteKey": "YOUR_SITE_KEY",
  "SecretKey": "YOUR_SECRET_KEY"
}
```

### 2) Login View (Razor) — Widget
```html
<script src="https://challenges.cloudflare.com/turnstile/v0/api.js" async defer></script>

<form method="post" asp-action="Login">
  <!-- username/password alanların -->
  <div class="cf-turnstile" data-sitekey="@Configuration["Turnstile:SiteKey"]"></div>

  <button type="submit">Giriş</button>
</form>
```

Turnstile token form ile birlikte **`cf-turnstile-response`** alanında gelir.

### 3) Verify DTO
```csharp
public sealed class TurnstileVerifyResponse
{
    public bool success { get; set; }
    public string[] error_codes { get; set; }
    public string hostname { get; set; }
    public DateTime? challenge_ts { get; set; }
}
```

### 4) Verify Service (HttpClientFactory ile)
```csharp
using System.Net.Http.Headers;

public interface ITurnstileVerifier
{
    Task<bool> VerifyAsync(string token, string remoteIp);
}

public class TurnstileVerifier : ITurnstileVerifier
{
    private readonly HttpClient _http;
    private readonly IConfiguration _cfg;

    public TurnstileVerifier(HttpClient http, IConfiguration cfg)
    {
        _http = http;
        _cfg = cfg;
    }

    public async Task<bool> VerifyAsync(string token, string remoteIp)
    {
        if (string.IsNullOrWhiteSpace(token)) return false;

        var secret = _cfg["Turnstile:SecretKey"];
        var form = new FormUrlEncodedContent(new Dictionary<string, string?>
        {
            ["secret"] = secret,
            ["response"] = token,
            ["remoteip"] = remoteIp
        });

        var resp = await _http.PostAsync(
            "https://challenges.cloudflare.com/turnstile/v0/siteverify",
            form);

        if (!resp.IsSuccessStatusCode) return false;

        var body = await resp.Content.ReadFromJsonAsync<TurnstileVerifyResponse>();
        return body?.success == true;
    }
}
```

### 5) Program.cs DI
```csharp
builder.Services.AddHttpClient<ITurnstileVerifier, TurnstileVerifier>();
```

### 6) Login Action — CAPTCHA kontrolü
```csharp
[HttpPost]
public async Task<IActionResult> Login(LoginModel model, [FromForm(Name="cf-turnstile-response")] string turnstileToken)
{
    var ip = HttpContext.Connection.RemoteIpAddress?.ToString();

    // CAPTCHA doğrula
    var ok = await _turnstile.VerifyAsync(turnstileToken, ip);
    if (!ok)
    {
        ModelState.AddModelError("", "CAPTCHA doğrulaması başarısız.");
        return View(model);
    }

    // sonra normal login (Identity vb.)
    // ...
    return RedirectToAction("Index", "Home");
}
```

---

## Seçenek B — Google reCAPTCHA (v2 / Invisible / v3 mantığı benzer)

### 1) appsettings.json
```json
"Recaptcha": {
  "SiteKey": "YOUR_SITE_KEY",
  "SecretKey": "YOUR_SECRET_KEY"
}
```

### 2) Login View (v2 Checkbox örneği)
```html
<script src="https://www.google.com/recaptcha/api.js" async defer></script>

<form method="post" asp-action="Login">
  <div class="g-recaptcha" data-sitekey="@Configuration["Recaptcha:SiteKey"]"></div>
  <button type="submit">Giriş</button>
</form>
```

Token form ile birlikte **`g-recaptcha-response`** alanında gelir.

### 3) Verify Service
```csharp
public sealed class RecaptchaVerifyResponse
{
    public bool success { get; set; }
    public string hostname { get; set; }
    public DateTime? challenge_ts { get; set; }
    public string[] error-codes { get; set; } // bazı örneklerde tireli dönebilir
}

public interface IRecaptchaVerifier
{
    Task<bool> VerifyAsync(string token, string remoteIp);
}

public class RecaptchaVerifier : IRecaptchaVerifier
{
    private readonly HttpClient _http;
    private readonly IConfiguration _cfg;

    public RecaptchaVerifier(HttpClient http, IConfiguration cfg)
    {
        _http = http;
        _cfg = cfg;
    }

    public async Task<bool> VerifyAsync(string token, string remoteIp)
    {
        if (string.IsNullOrWhiteSpace(token)) return false;

        var secret = _cfg["Recaptcha:SecretKey"];
        var form = new FormUrlEncodedContent(new Dictionary<string, string?>
        {
            ["secret"] = secret,
            ["response"] = token,
            ["remoteip"] = remoteIp
        });

        var resp = await _http.PostAsync("https://www.google.com/recaptcha/api/siteverify", form);
        if (!resp.IsSuccessStatusCode) return false;

        // Not: JSON alan isimleri varyasyon gösterebilir; production’da JsonPropertyName ile map etmek iyi olur.
        var json = await resp.Content.ReadAsStringAsync();
        return json.Contains(""success": true", StringComparison.OrdinalIgnoreCase);
    }
}
```

### 4) Login Action
```csharp
[HttpPost]
public async Task<IActionResult> Login(LoginModel model, [FromForm(Name="g-recaptcha-response")] string recaptchaToken)
{
    var ip = HttpContext.Connection.RemoteIpAddress?.ToString();

    if (!await _recaptcha.VerifyAsync(recaptchaToken, ip))
    {
        ModelState.AddModelError("", "CAPTCHA doğrulaması başarısız.");
        return View(model);
    }

    // normal login...
    return RedirectToAction("Index", "Home");
}
```

---

## 3️⃣ Rate Limit ile Birlikte “Dinamik CAPTCHA” (Pratik Yaklaşım)

İki yaygın strateji:
- **Her login denemesinde CAPTCHA** (en güvenli, en fazla sürtünme)
- **Sadece şüpheli durumda CAPTCHA** (daha iyi UX)

Örn. “aynı IP 3 kez hata yaptıysa CAPTCHA zorunlu”:
- IP başarısız deneme sayısını tut (MemoryCache / Redis)
- Eşik aşılınca login view’de CAPTCHA render et
- Sunucuda token yoksa login’i reddet

---

## 4️⃣ Önemli Notlar

- Token’lar **tek kullanımlık** ve kısa süreli olur; her submit’te yeniden doğrulanır.
- Proxy arkasındaysan gerçek IP için `ForwardedHeaders` ayarla (aksi halde hep proxy IP’si gelir).
- CAPTCHA tek başına yeterli değil: **Rate limit + lockout + 2FA** birlikte en iyi sonuç.
---

# 📊 Saldırı & Deneme Loglama Dashboard (ASP.NET Core)

Bu doküman, **login denemeleri / brute-force / rate-limit / IP ban** olaylarını loglayıp
**admin panelde dashboard** olarak göstermek için pratik bir örnek mimari sunar.

Hedef:
- Kim, ne zaman, hangi IP’den denedi?
- Kaç başarısız deneme var?
- Hangi IP’ler banlandı?
- Son 24 saatte trend nasıl?

---

## 1) Mimari Öneri

**Uygulama → Olay Logları (DB) → Dashboard (Admin UI)**

- Olayları uygulama içinde yakala (login, 2FA, rate-limit reject, ban)
- Tek tablo (kolay) veya normalleştirilmiş şema (ileri seviye)
- Admin panel:
  - Son denemeler
  - Top IP’ler
  - Zaman serisi grafik (son 24 saat)
  - Ban listesi / ban süresi

> Küçük projelerde DB yeterli. Trafik büyürse: **Serilog + Seq/ELK**, veya **OpenTelemetry + Prometheus/Grafana**.

---

## 2) Veri Modeli (EF Core)

### 2.1 AttackEvent Tablosu (Önerilen tek tablo)
```csharp
public enum SecurityEventType
{
    LoginSuccess,
    LoginFailed,
    TwoFactorRequired,
    TwoFactorFailed,
    RateLimitRejected,
    IpBanned,
    IpUnbanned
}

public class SecurityEvent
{
    public long Id { get; set; }
    public DateTime CreatedUtc { get; set; } = DateTime.UtcNow;

    public SecurityEventType Type { get; set; }

    public string IpAddress { get; set; } = "";
    public string? Username { get; set; }
    public string? UserId { get; set; }

    public string? Path { get; set; }
    public string? UserAgent { get; set; }

    public int? StatusCode { get; set; }
    public string? Message { get; set; }

    // İstersen correlation id / request id
    public string? TraceId { get; set; }
}
```

### 2.2 Ban Tablosu (opsiyonel)
```csharp
public class BannedIp
{
    public long Id { get; set; }
    public string IpAddress { get; set; } = "";
    public DateTime BannedUntilUtc { get; set; }
    public string? Reason { get; set; }
}
```

---

## 3) Event Logger Service

```csharp
public interface ISecurityEventLogger
{
    Task LogAsync(HttpContext ctx, SecurityEventType type, string ip, string? username = null,
                  int? statusCode = null, string? message = null);
}

public class SecurityEventLogger : ISecurityEventLogger
{
    private readonly AppDbContext _db;

    public SecurityEventLogger(AppDbContext db) => _db = db;

    public async Task LogAsync(HttpContext ctx, SecurityEventType type, string ip, string? username = null,
                               int? statusCode = null, string? message = null)
    {
        var ev = new SecurityEvent
        {
            Type = type,
            IpAddress = ip,
            Username = username,
            Path = ctx.Request.Path,
            UserAgent = ctx.Request.Headers.UserAgent.ToString(),
            StatusCode = statusCode,
            Message = message,
            TraceId = ctx.TraceIdentifier
        };

        _db.SecurityEvents.Add(ev);
        await _db.SaveChangesAsync();
    }
}
```

### Program.cs DI
```csharp
builder.Services.AddScoped<ISecurityEventLogger, SecurityEventLogger>();
```

---

## 4) Nerelerde Loglanır?

### 4.1 Login Action (Başarılı / Başarısız)
```csharp
[HttpPost]
public async Task<IActionResult> Login(LoginModel model)
{
    var ip = HttpContext.Connection.RemoteIpAddress?.ToString() ?? "unknown";

    var result = await _signInManager.PasswordSignInAsync(
        model.Username, model.Password,
        isPersistent: false,
        lockoutOnFailure: true);

    if (result.Succeeded)
    {
        await _secLog.LogAsync(HttpContext, SecurityEventType.LoginSuccess, ip, model.Username, 200);
        return RedirectToAction("Index", "Home");
    }

    await _secLog.LogAsync(HttpContext, SecurityEventType.LoginFailed, ip, model.Username, 401, "Bad credentials");
    ModelState.AddModelError("", "Hatalı giriş");
    return View(model);
}
```

### 4.2 Rate Limit Rejected (OnRejected)
```csharp
builder.Services.AddRateLimiter(options =>
{
    options.OnRejected = async (context, token) =>
    {
        var ip = context.HttpContext.Connection.RemoteIpAddress?.ToString() ?? "unknown";
        var logger = context.HttpContext.RequestServices.GetRequiredService<ISecurityEventLogger>();

        await logger.LogAsync(context.HttpContext, SecurityEventType.RateLimitRejected, ip,
                              username: context.HttpContext.User.Identity?.Name,
                              statusCode: 429,
                              message: "Rate limit exceeded");

        context.HttpContext.Response.StatusCode = 429;
        await context.HttpContext.Response.WriteAsync("Too many requests");
    };
});
```

### 4.3 IP Ban Olayı
Ban ederken:
```csharp
await _secLog.LogAsync(HttpContext, SecurityEventType.IpBanned, ip, model.Username, 403, "Banned 15m");
```

---

## 5) Dashboard için Query’ler (EF Core)

### 5.1 Son denemeler (son 200)
```csharp
var recent = await _db.SecurityEvents
    .OrderByDescending(x => x.CreatedUtc)
    .Take(200)
    .ToListAsync();
```

### 5.2 Son 24 saatte başarısız login sayısı
```csharp
var since = DateTime.UtcNow.AddHours(-24);

var failedCount = await _db.SecurityEvents.CountAsync(x =>
    x.CreatedUtc >= since && x.Type == SecurityEventType.LoginFailed);
```

### 5.3 Top saldırgan IP’ler (son 24 saat)
```csharp
var topIps = await _db.SecurityEvents
    .Where(x => x.CreatedUtc >= since && x.Type == SecurityEventType.LoginFailed)
    .GroupBy(x => x.IpAddress)
    .Select(g => new { Ip = g.Key, Count = g.Count() })
    .OrderByDescending(x => x.Count)
    .Take(20)
    .ToListAsync();
```

### 5.4 Zaman serisi (saatlik)
```csharp
var hourly = await _db.SecurityEvents
    .Where(x => x.CreatedUtc >= since && x.Type == SecurityEventType.LoginFailed)
    .GroupBy(x => new DateTime(x.CreatedUtc.Year, x.CreatedUtc.Month, x.CreatedUtc.Day, x.CreatedUtc.Hour, 0, 0))
    .Select(g => new { Hour = g.Key, Count = g.Count() })
    .OrderBy(x => x.Hour)
    .ToListAsync();
```

---

## 6) Admin Dashboard UI (Razor Pages veya MVC)

### 6.1 Route + Yetki
```csharp
[Authorize(Roles = "Admin")]
public class SecurityDashboardController : Controller
{
    private readonly AppDbContext _db;
    public SecurityDashboardController(AppDbContext db) => _db = db;

    public async Task<IActionResult> Index()
    {
        // burada query’leri çalıştırıp ViewModel döndür
        return View();
    }
}
```

### 6.2 Basit Chart.js entegrasyonu (View)
```html
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<canvas id="failChart"></canvas>

<script>
  const labels = @Html.Raw(Json.Serialize(Model.HourLabels));
  const data = @Html.Raw(Json.Serialize(Model.HourCounts));

  new Chart(document.getElementById('failChart'), {
    type: 'line',
    data: { labels, datasets: [{ label: 'Login Failed (24h)', data }] }
  });
</script>
```

> Not: CDN istenmiyorsa Chart.js projeye eklenebilir.

---

## 7) Üretim Notları (Çok Önemli)

- **PII / KVKK**: IP kişisel veridir. Saklama süresi ve amaç belirle.
- **Performans**: Yük artarsa log insert’i kuyrukla (Channel/BackgroundService) veya Redis/Queue kullan.
- **İndeks**: `CreatedUtc`, `Type`, `IpAddress` alanlarına indeks ekle.
- **Gizlilik**: Dashboard sadece Admin/2FA/IP allow list ile erişsin.
- **Alerting**: “X dakikada Y başarısız login” eşiğinde uyarı üret (email/slack vs.)

### SQL Index Önerisi
```sql
CREATE INDEX IX_SecurityEvents_CreatedUtc ON SecurityEvents(CreatedUtc);
CREATE INDEX IX_SecurityEvents_Type_CreatedUtc ON SecurityEvents(Type, CreatedUtc);
CREATE INDEX IX_SecurityEvents_Ip_Type_CreatedUtc ON SecurityEvents(IpAddress, Type, CreatedUtc);
```

---

# 🧠 Redis Tabanlı IP Ban + Log Kuyruğu (ASP.NET Core)

Bu doküman iki parçadan oluşur:

1) **Redis ile IP Ban (TTL’li, dağıtık çalışır)**  
2) **Log kuyruğu** (istek thread’ini yormadan güvenlik olaylarını yazma)

> Hedef: Uygulama birden fazla instance çalışsa bile (K8s, IIS farm, Azure App Service scale-out) ban ve log tutarlılığı.

---

## 0) Paketler

- Redis istemcisi: `StackExchange.Redis`
- EF Core (DB’ye yazacaksan): `Microsoft.EntityFrameworkCore.*`

```bash
dotnet add package StackExchange.Redis
```

---

## 1) Redis IP Ban Tasarımı

### 1.1 Key Şeması (öneri)

- Ban anahtarı:  
  `ban:ip:{ip}`  ➜ value: sebep/metin, TTL: ban süresi

- Fail counter:  
  `fail:login:ip:{ip}` ➜ value: sayı, TTL: pencere süresi (örn 10 dk)

- (Opsiyonel) Whitelist:  
  `wl:ip:{ip}` ➜ value: 1, TTL: yok

Bu sayede:
- Ban süre sonunda **otomatik kalkar** (TTL)
- Fail counter penceresi dolunca sıfırlanır (TTL)

---

## 2) Redis Ban Servisi (StackExchange.Redis)

### 2.1 Ayarlar (appsettings.json)

```json
"Redis": {
  "ConnectionString": "localhost:6379"
},
"Security": {
  "LoginFailWindowMinutes": 10,
  "LoginFailThreshold": 10,
  "BanMinutes": 15
}
```

### 2.2 Program.cs – Redis bağlantısı

```csharp
using StackExchange.Redis;

builder.Services.AddSingleton<IConnectionMultiplexer>(sp =>
    ConnectionMultiplexer.Connect(builder.Configuration["Redis:ConnectionString"]));

builder.Services.AddSingleton<RedisIpBanService>();
```

### 2.3 RedisIpBanService

```csharp
using StackExchange.Redis;

public sealed class RedisIpBanService
{
    private readonly IDatabase _db;
    private readonly IConfiguration _cfg;

    public RedisIpBanService(IConnectionMultiplexer mux, IConfiguration cfg)
    {
        _db = mux.GetDatabase();
        _cfg = cfg;
    }

    private static string BanKey(string ip) => $"ban:ip:{ip}";
    private static string FailKey(string ip) => $"fail:login:ip:{ip}";
    private static string WlKey(string ip) => $"wl:ip:{ip}";

    public async Task<bool> IsWhitelistedAsync(string ip)
        => await _db.StringGetAsync(WlKey(ip)) == "1";

    public async Task<bool> IsBannedAsync(string ip)
        => await _db.KeyExistsAsync(BanKey(ip));

    public async Task BanAsync(string ip, TimeSpan duration, string? reason = null)
        => await _db.StringSetAsync(BanKey(ip), reason ?? "banned", duration);

    public async Task UnbanAsync(string ip)
        => await _db.KeyDeleteAsync(BanKey(ip));

    /// <summary>
    /// Login başarısızlığını kaydeder. Eşik aşılırsa IP'yi banlar.
    /// </summary>
    public async Task RegisterLoginFailureAsync(string ip)
    {
        if (await IsWhitelistedAsync(ip))
            return;

        int windowMin = _cfg.GetValue<int>("Security:LoginFailWindowMinutes", 10);
        int threshold = _cfg.GetValue<int>("Security:LoginFailThreshold", 10);
        int banMin = _cfg.GetValue<int>("Security:BanMinutes", 15);

        // INCR ile sayacı artır
        var newValue = await _db.StringIncrementAsync(FailKey(ip));

        // İlk kez oluştuysa pencere TTL ver
        if (newValue == 1)
            await _db.KeyExpireAsync(FailKey(ip), TimeSpan.FromMinutes(windowMin));

        if (newValue >= threshold)
        {
            await BanAsync(ip, TimeSpan.FromMinutes(banMin), $"login-fail>={threshold}");
        }
    }
}
```

> Not: Bu yaklaşım pratikte çoğu senaryoda yeterli. Daha “atomik” kontrol istersen Lua script ile TTL + ban işlemini tek işlemde yapabilirsin.

---

## 3) Ban Middleware (Tüm istekler veya sadece /account/login)

### 3.1 Sadece login endpoint’i için (öneri)

```csharp
public class LoginBanMiddleware
{
    private readonly RequestDelegate _next;

    public LoginBanMiddleware(RequestDelegate next) => _next = next;

    public async Task Invoke(HttpContext ctx, RedisIpBanService ban)
    {
        // sadece login path’i
        if (!ctx.Request.Path.StartsWithSegments("/account/login"))
        {
            await _next(ctx);
            return;
        }

        var ip = ctx.Connection.RemoteIpAddress?.ToString() ?? "unknown";

        if (await ban.IsBannedAsync(ip))
        {
            ctx.Response.StatusCode = StatusCodes.Status403Forbidden;
            await ctx.Response.WriteAsync("IP geçici olarak engellendi (ban).");
            return;
        }

        await _next(ctx);
    }
}
```

### 3.2 Program.cs sıralama

```csharp
app.UseMiddleware<LoginBanMiddleware>();

app.UseAuthentication();
app.UseAuthorization();

// rate limiter kullanıyorsan:
app.UseRateLimiter();
```

---

## 4) Login Action – Başarısızlıkta Redis sayacı artırma

```csharp
[HttpPost]
public async Task<IActionResult> Login(LoginModel model)
{
    var ip = HttpContext.Connection.RemoteIpAddress?.ToString() ?? "unknown";

    var result = await _signInManager.PasswordSignInAsync(
        model.Username, model.Password,
        isPersistent: false,
        lockoutOnFailure: true);

    if (result.Succeeded)
    {
        // başarılı login: istersen fail counter sıfırla (opsiyonel)
        return RedirectToAction("Index", "Home");
    }

    await _ban.RegisterLoginFailureAsync(ip);

    ModelState.AddModelError("", "Hatalı giriş");
    return View(model);
}
```

---

# 5) Log Kuyruğu (Request thread’ini yormadan log yazma)

İki temel yaklaşım var:

- **A) In-memory kuyruk (Channel) + arka plan worker** ➜ DB’ye batch insert  
- **B) Redis Stream (XADD) + consumer** ➜ birden fazla instance ile ölçeklenebilir

Küçük-orta ölçek için A, dağıtık/çok instance + yüksek trafik için B daha iyi.

---

## 5A) Channel + BackgroundService (Basit ve hızlı)

### 5A.1 Log DTO

```csharp
public record SecurityEventDto(
    DateTime CreatedUtc,
    string Type,
    string Ip,
    string? Username,
    string? Path,
    int? StatusCode,
    string? Message);
```

### 5A.2 Kuyruk (Channel)

```csharp
using System.Threading.Channels;

public interface ISecurityEventQueue
{
    bool TryEnqueue(SecurityEventDto ev);
    ValueTask<SecurityEventDto> DequeueAsync(CancellationToken ct);
}

public class SecurityEventQueue : ISecurityEventQueue
{
    private readonly Channel<SecurityEventDto> _ch = Channel.CreateBounded<SecurityEventDto>(
        new BoundedChannelOptions(50_000)
        {
            SingleReader = true,
            SingleWriter = false,
            FullMode = BoundedChannelFullMode.DropOldest // ya da Wait
        });

    public bool TryEnqueue(SecurityEventDto ev) => _ch.Writer.TryWrite(ev);

    public ValueTask<SecurityEventDto> DequeueAsync(CancellationToken ct)
        => _ch.Reader.ReadAsync(ct);
}
```

### 5A.3 Worker (Batch insert)

```csharp
public class SecurityEventWorker : BackgroundService
{
    private readonly ISecurityEventQueue _q;
    private readonly IServiceProvider _sp;

    public SecurityEventWorker(ISecurityEventQueue q, IServiceProvider sp)
    {
        _q = q;
        _sp = sp;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        var batch = new List<SecurityEventDto>(500);

        while (!stoppingToken.IsCancellationRequested)
        {
            try
            {
                // en az 1 tane bekle
                var first = await _q.DequeueAsync(stoppingToken);
                batch.Add(first);

                // hızlıca batch’i doldur
                var deadline = DateTime.UtcNow.AddMilliseconds(250);
                while (batch.Count < 500 && DateTime.UtcNow < deadline && _q is SecurityEventQueue)
                {
                    // TryRead erişimi yoksa basitçe timeout yaklaşımıyla ilerle.
                    break;
                }

                // DB’ye yaz
                using var scope = _sp.CreateScope();
                var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();

                foreach (var ev in batch)
                {
                    db.SecurityEvents.Add(new SecurityEvent
                    {
                        CreatedUtc = ev.CreatedUtc,
                        Type = Enum.Parse<SecurityEventType>(ev.Type),
                        IpAddress = ev.Ip,
                        Username = ev.Username,
                        Path = ev.Path,
                        StatusCode = ev.StatusCode,
                        Message = ev.Message
                    });
                }

                await db.SaveChangesAsync(stoppingToken);
                batch.Clear();
            }
            catch (OperationCanceledException) { }
            catch
            {
                // burada bir ILogger ile hata logla
                batch.Clear();
                await Task.Delay(500, stoppingToken);
            }
        }
    }
}
```

> Not: Yukarıdaki worker “minimum örnek”. Production’da batch doldurmayı daha iyi yaparsın (TryRead, timeouts, ölçümleme).

### 5A.4 DI

```csharp
builder.Services.AddSingleton<ISecurityEventQueue, SecurityEventQueue>();
builder.Services.AddHostedService<SecurityEventWorker>();
```

### 5A.5 Kullanım (enqueue)

```csharp
_q.TryEnqueue(new SecurityEventDto(
    CreatedUtc: DateTime.UtcNow,
    Type: SecurityEventType.LoginFailed.ToString(),
    Ip: ip,
    Username: model.Username,
    Path: HttpContext.Request.Path,
    StatusCode: 401,
    Message: "Bad credentials"));
```

---

## 5B) Redis Stream ile Log Kuyruğu (Dağıtık, ölçeklenebilir)

### 5B.1 Stream adı
- `stream:security-events`

### 5B.2 Producer: XADD (log ekleme)

```csharp
public class RedisSecurityEventProducer
{
    private readonly IDatabase _db;

    public RedisSecurityEventProducer(IConnectionMultiplexer mux)
        => _db = mux.GetDatabase();

    public Task AddAsync(SecurityEventDto ev)
    {
        var key = "stream:security-events";

        return _db.StreamAddAsync(key, new NameValueEntry[]
        {
            new("createdUtc", ev.CreatedUtc.ToString("O")),
            new("type", ev.Type),
            new("ip", ev.Ip),
            new("username", ev.Username ?? ""),
            new("path", ev.Path ?? ""),
            new("statusCode", ev.StatusCode?.ToString() ?? ""),
            new("message", ev.Message ?? "")
        });
    }
}
```

### 5B.3 Consumer Group ile Worker
- Consumer: stream’den okur
- DB’ye yazar
- ACK verir

> Bu kısım biraz uzun olduğu için istersen “tam çalışan worker” örneğini ayrıca ekleyebilirim.

---

## 6) Üretim Önerileri

- **Whitelist**: Ofis IP’leri / admin IP’leri ban’dan muaf tut
- **Kademeli ban**: 15dk → 1s → 24s (Redis’te ayrı sayaçla yönet)
- **Gerçek IP**: Proxy arkasında `ForwardedHeaders` ayarla
- **Observability**: ban sayısı / fail sayısı / 429 sayısı metrikle
- **KVKK/GDPR**: IP saklama süresi ve amaç politikası oluştur

---

## 7) Hızlı Özet

| Özellik | Çözüm |
|---|---|
| Dağıtık IP Ban | Redis key + TTL |
| Fail sayacı | Redis INCR + TTL |
| Log’u request’ten ayırma | Channel + BackgroundService |
| Dağıtık log kuyruğu | Redis Streams |


---

# 🖥 Blazor Admin Dashboard (Tablo + Grafik + Filtre)

Bu doküman **Blazor (Server veya WASM)** ile yapılmış bir **Admin Security Dashboard** örneğidir.

Dashboard şunları gösterir:
- Login denemeleri
- Saldırgan IP’ler
- Rate-limit / ban olayları
- Zaman bazlı grafikler
- Filtreleme (tarih, IP, event tipi)

---

## 1️⃣ Mimari

```
Blazor Admin UI
   ↓
Security API / DbContext
   ↓
SecurityEvents (DB)
```

> Dashboard **sadece Admin + IP + 2FA** ile erişilebilir olmalıdır.

---

## 2️⃣ Veri Modeli (Özet)

```csharp
public enum SecurityEventType
{
    LoginSuccess,
    LoginFailed,
    RateLimitRejected,
    IpBanned
}

public class SecurityEvent
{
    public long Id { get; set; }
    public DateTime CreatedUtc { get; set; }
    public SecurityEventType Type { get; set; }
    public string IpAddress { get; set; }
    public string? Username { get; set; }
}
```

---

## 3️⃣ Dashboard Route (Yetkili)

```razor
@page "/admin/security"
@attribute [Authorize(Roles = "Admin")]
```

---

## 4️⃣ Filtre Paneli

```razor
<div class="filters">
    <input type="date" @bind="fromDate" />
    <input type="date" @bind="toDate" />

    <select @bind="selectedType">
        <option value="">Tümü</option>
        <option value="LoginFailed">Login Failed</option>
        <option value="LoginSuccess">Login Success</option>
        <option value="RateLimitRejected">Rate Limit</option>
        <option value="IpBanned">IP Ban</option>
    </select>

    <input placeholder="IP adresi" @bind="ipFilter" />
    <button @onclick="Load">Filtrele</button>
</div>
```

---

## 5️⃣ Tablo (Data Grid)

```razor
<table class="table">
    <thead>
        <tr>
            <th>Tarih</th>
            <th>Tip</th>
            <th>IP</th>
            <th>Kullanıcı</th>
        </tr>
    </thead>
    <tbody>
        @foreach (var e in events)
        {
            <tr>
                <td>@e.CreatedUtc</td>
                <td>@e.Type</td>
                <td>@e.IpAddress</td>
                <td>@e.Username</td>
            </tr>
        }
    </tbody>
</table>
```

---

## 6️⃣ Grafik (Chart.js – Login Failed / Saatlik)

```razor
@inject IJSRuntime JS

<canvas id="failChart"></canvas>

@code {
    protected override async Task OnAfterRenderAsync(bool first)
    {
        if (first)
        {
            await JS.InvokeVoidAsync("renderFailChart", labels, counts);
        }
    }
}
```

### wwwroot/js/chart.js
```javascript
window.renderFailChart = (labels, data) => {
    new Chart(document.getElementById('failChart'), {
        type: 'line',
        data: {
            labels: labels,
            datasets: [{
                label: 'Login Failed',
                data: data
            }]
        }
    });
};
```

---

## 7️⃣ Code Behind (Veri Yükleme)

```csharp
@code {
    List<SecurityEvent> events = new();
    DateTime fromDate = DateTime.UtcNow.AddDays(-1);
    DateTime toDate = DateTime.UtcNow;
    string? selectedType;
    string? ipFilter;

    async Task Load()
    {
        events = await Db.SecurityEvents
            .Where(x => x.CreatedUtc >= fromDate && x.CreatedUtc <= toDate)
            .Where(x => string.IsNullOrEmpty(selectedType) || x.Type.ToString() == selectedType)
            .Where(x => string.IsNullOrEmpty(ipFilter) || x.IpAddress.Contains(ipFilter))
            .OrderByDescending(x => x.CreatedUtc)
            .Take(500)
            .ToListAsync();
    }
}
```

---

## 8️⃣ Top Saldırgan IP’ler (Mini Widget)

```razor
<ul>
@foreach (var ip in topIps)
{
    <li>@ip.Ip - @ip.Count deneme</li>
}
</ul>
```

```csharp
var topIps = await Db.SecurityEvents
    .Where(x => x.Type == SecurityEventType.LoginFailed)
    .GroupBy(x => x.IpAddress)
    .Select(g => new { Ip = g.Key, Count = g.Count() })
    .OrderByDescending(x => x.Count)
    .Take(10)
    .ToListAsync();
```

---

## 9️⃣ Güvenlik Notları ⚠️

- Dashboard **Admin + IP allow list + 2FA**
- Pagination / limit zorunlu
- Loglar için retention policy
- IP maskleme (opsiyonel)

---

## 10️⃣ Genişletme Fikirleri

- 🔴 Canlı dashboard (SignalR)
- 📁 CSV / Excel export
- 🚨 Alarm & bildirim
- 🧠 Davranış analizi

---

## 11️⃣ Özet

| Özellik | Durum |
|------|------|
| Tablo | ✅ |
| Grafik | ✅ |
| Filtreleme | ✅ |
| Admin koruma | ✅ |
| Genişletilebilir | ✅ |

