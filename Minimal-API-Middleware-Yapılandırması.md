# ASP.NET Core Minimal API Middleware Yapılandırması (.NET CLI ile)

.NET CLI (Command Line Interface) kullanarak `Program.cs` içindeki `builder` ve `app` ile middleware yapılandıran bir ASP.NET Core projesi oluşturma adımları:

---

## 1. Yeni ASP.NET Core Web Projesi Oluşturma

Terminalde (komut istemcisi):

```bash
dotnet new web -o MyMiddlewareApp
```

- `web` minimal API template’idir, `Program.cs` içinde `builder` ve `app` ile middleware yazarız.
- `-o MyMiddlewareApp` klasör adıdır.

---

## 2. Proje Klasörüne Git

```bash
cd MyMiddlewareApp
```

---

## 3. Projeyi Çalıştır

```bash
dotnet run
```

---

## 4. `Program.cs` Örneği (Oluşturulan Proje İçinde)

```csharp
var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

app.Use(async (context, next) =>
{
    Console.WriteLine("Middleware before next");
    await next.Invoke();
    Console.WriteLine("Middleware after next");
});

app.MapGet("/", () => "Hello World!");

app.Run();
```

---

## 5. Yeni Middleware Eklemek İçin

```csharp
app.Use(...); // Inline middleware
app.UseMiddleware<YourMiddlewareClass>(); // Custom middleware class
```

---

## Özet Komutlar

| Komut                                | Açıklama                         |
|--------------------------------------|----------------------------------|
| `dotnet new web -o MyMiddlewareApp`  | Minimal API projesi oluşturur    |
| `cd MyMiddlewareApp`                 | Proje klasörüne geçer            |
| `dotnet run`                         | Projeyi çalıştırır               |

---
