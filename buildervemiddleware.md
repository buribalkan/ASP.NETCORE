
# ASP.NET Core `Program.cs` içinde `builder` ve `app` kullanımı

Evet, `.NET 6+` (veya `.NET Core 6+`) projelerinde `Program.cs` içine `builder` ve `app` nesneleri ile **middleware yazılır ve uygulama yapılandırılır**.

---

## Açıklama

.NET 6 ve sonrası için **minimal hosting modeli** geldi.  
Artık `Startup.cs` sınıfı zorunlu değil, tüm yapılandırma `Program.cs` içinde yapılabiliyor.

---

## Örnek `Program.cs`

```csharp
var builder = WebApplication.CreateBuilder(args);

// Servisleri ekle
builder.Services.AddControllersWithViews();

var app = builder.Build();

// Middleware pipeline
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();

app.UseRouting();

app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

---

## Detaylar

- `builder` üzerinden servisleri (Dependency Injection servisleri, middleware, vb.) ekliyorsun.
- `app` nesnesi üzerinden middleware sırayla pipeline’a ekleniyor (`UseStaticFiles`, `UseRouting`, `UseAuthorization` gibi).
- `app.Map...` ile route’ları tanımlıyorsun.
- `app.Run()` ile uygulamayı başlatıyorsun.

---

## Özet

| Nesne    | Görev                                             |
|----------|---------------------------------------------------|
| `builder` | Servisleri (DI container) yapılandırma            |
| `app`     | Middleware pipeline, route ve uygulama başlatma   |

---

## Sonuç

- `Program.cs` içinde hem `builder` hem `app` kullanarak middleware yazmak ve yapılandırmak **standarttır**.
- `Startup.cs` kullanımı artık **opsiyonel**, istersen ayrı bir `Startup` sınıfı da oluşturabilirsin.

---


