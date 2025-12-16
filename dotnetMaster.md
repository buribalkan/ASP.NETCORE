# Chapter 1 – .NET CLI’ye Giriş

## 1.1 .NET CLI Nedir?

.NET CLI (Command Line Interface), .NET uygulamalarını komut satırı üzerinden
oluşturmak, derlemek, çalıştırmak ve yönetmek için kullanılan resmi araçtır.

IDE bağımsızdır ve özellikle:
- Backend geliştirme
- CI/CD süreçleri
- Linux & Docker ortamları
için kritik öneme sahiptir.

---

## 1.2 .NET SDK ve Runtime Farkı

| Bileşen | Açıklama |
|-------|---------|
| .NET Runtime | Uygulamaları çalıştırır |
| .NET SDK | Runtime + CLI + derleyici + template’ler |

> dotnet CLI kullanımı için SDK gereklidir.


---

## 1.3 dotnet CLI Komut Yapısı

```bash
dotnet <komut> [opsiyonlar]
```

Örnekler:
```bash
dotnet new
dotnet build
dotnet run
```

---

## 1.4 Yardım Komutları

```bash
dotnet --help
dotnet new --help
dotnet build --help
```

---

## 1.5 dotnet new – Proje Oluşturma

Mevcut template’ler:
```bash
dotnet new list
```

Console uygulaması:
```bash
dotnet new console -n HelloCli
cd HelloCli
```

Oluşan yapı:
```text
HelloCli/
 ├─ Program.cs
 ├─ HelloCli.csproj
```

---

## 1.6 Uygulamayı Çalıştırma

```bash
dotnet run
```

Çıktı:
```text
Hello, World!
```



---

# Chapter 2 – Build, Restore, Clean & Solution Yönetimi

## 2.1 Build, Restore ve Clean Nedir?

.NET CLI projeleri derlerken üç temel işlem vardır:

- **restore**: NuGet paketlerini indirir
- **build**: Kodu derler
- **clean**: Derleme çıktısını temizler

Bu işlemler CI/CD ve günlük geliştirme sürecinin temelidir.

---

## 2.2 dotnet restore

Projede kullanılan NuGet paketlerini indirir.

```bash
dotnet restore
```

Belirli bir proje için:
```bash
dotnet restore MyApp.csproj
```

> `dotnet build` otomatik olarak restore yapar, ancak CI ortamlarında ayrı çalıştırmak tercih edilir.

---

## 2.3 dotnet build

Projeyi derler ancak çalıştırmaz.

```bash
dotnet build
```

Release modunda build:
```bash
dotnet build -c Release
```

Çıktı klasörü:
```text
bin/
obj/
```

---

## 2.4 dotnet clean

Önceki build çıktısını temizler.

```bash
dotnet clean
```

Release için:
```bash
dotnet clean -c Release
```

Clean sonrası:
- `bin/` silinir
- `obj/` silinir

---

## 2.5 Build Akışı (Önerilen)

```bash
dotnet clean
dotnet restore
dotnet build
```

CI/CD için idealdir.

---

## 2.6 Solution (sln) Nedir?

Solution, birden fazla projeyi tek çatı altında toplar.

Örnek:
```text
MySolution.sln
 ├─ ApiProject/
 ├─ CoreProject/
 └─ TestProject/
```

---

## 2.7 Solution Oluşturma

```bash
dotnet new sln -n MySolution
```

Projeyi solution’a ekleme:
```bash
dotnet sln MySolution.sln add ApiProject/ApiProject.csproj
```

---

## 2.8 Solution’dan Proje Silme

```bash
dotnet sln MySolution.sln remove ApiProject/ApiProject.csproj
```

---

## 2.9 Solution Üzerinden Build

```bash
dotnet build MySolution.sln
```

Tüm projeler tek seferde derlenir.


---
# Chapter 3 – dotnet run, publish & Environment Yönetimi

## 3.1 dotnet run Nedir?

`dotnet run`, projeyi **derleyip çalıştıran** birleşik bir komuttur.
Geliştirme sürecinde en sık kullanılan komutlardan biridir.

```bash
dotnet run
```

Varsayılan olarak:
- Debug modunda çalışır
- `Program.cs` içindeki giriş noktasını kullanır

---

## 3.2 dotnet run Parametreleri

### Release modunda çalıştırma
```bash
dotnet run -c Release
```

### Uygulamaya argüman gönderme
```bash
dotnet run -- --port 5000 --env dev
```

> `--` sonrası parametreler uygulamaya iletilir.

---

## 3.3 Ortam (Environment) Kavramı

.NET uygulamalarında environment, uygulamanın davranışını belirler.

Yaygın environment isimleri:
- Development
- Staging
- Production

.NET bunu `ASPNETCORE_ENVIRONMENT` değişkeni ile yönetir.

---

## 3.4 Environment Ayarlama

### Windows (PowerShell)
```powershell
$env:ASPNETCORE_ENVIRONMENT="Development"
dotnet run
```

### Linux / macOS
```bash
export ASPNETCORE_ENVIRONMENT=Development
dotnet run
```

---

## 3.5 dotnet publish Nedir?

`dotnet publish`, uygulamayı **deploy edilebilir** hale getirir.

```bash
dotnet publish
```

Publish çıktısı:
```text
bin/
 └─ Release/
     └─ net8.0/
         └─ publish/
```

---

## 3.6 Release Modunda Publish

```bash
dotnet publish -c Release
```

Bu çıktı:
- Production ortamı
- Docker
- Sunucu deploy
için kullanılır.

---

## 3.7 Runtime Spesifik Publish

### Linux için
```bash
dotnet publish -c Release -r linux-x64
```

### Windows için
```bash
dotnet publish -c Release -r win-x64
```

---

## 3.8 Self-Contained vs Framework-Dependent

### Framework-Dependent
- .NET Runtime gerektirir
- Daha küçük çıktı

```bash
dotnet publish
```

### Self-Contained
- Runtime içerir
- Daha büyük ama bağımsız

```bash
dotnet publish -c Release -r linux-x64 --self-contained true
```

---

## 3.9 Chapter 3 Özet

- dotnet run kullanımı
- Environment yönetimi
- dotnet publish
- Runtime bazlı dağıtım
- Self-contained vs framework-dependent

---


# Chapter 4 – NuGet Paket Yönetimi

## 4.1 NuGet Nedir?

NuGet, .NET ekosisteminin resmi paket yöneticisidir.
Uygulamalara:
- Kütüphane
- Araç
- SDK
eklemek için kullanılır.

.NET CLI, NuGet işlemlerini komut satırından yönetmeyi sağlar.

---

## 4.2 dotnet add package

Projeye NuGet paketi ekler.

```bash
dotnet add package Newtonsoft.Json
```

Belirli versiyon ekleme:
```bash
dotnet add package Newtonsoft.Json --version 13.0.3
```

Komut sonucu:
- `.csproj` dosyası güncellenir
- Paket otomatik restore edilir

---

## 4.3 dotnet remove package

Projeden paket kaldırır.

```bash
dotnet remove package Newtonsoft.Json
```

---

## 4.4 dotnet list package

Projede yüklü paketleri listeler.

```bash
dotnet list package
```

Tüm bağımlılıkları görmek için:
```bash
dotnet list package --include-transitive
```

---

## 4.5 NuGet Kaynakları (Sources)

Varsayılan NuGet kaynağı:
- https://api.nuget.org/v3/index.json

Kaynakları listeleme:
```bash
dotnet nuget list source
```

Yeni kaynak ekleme:
```bash
dotnet nuget add source https://my-nuget-feed --name MyFeed
```

---

## 4.6 Paket Restore Süreci

Paketler şu klasöre indirilir:
```text
~/.nuget/packages
```

Manuel restore:
```bash
dotnet restore
```

Cache temizleme:
```bash
dotnet nuget locals all --clear
```

---

## 4.7 Central Package Management (CPM)

Tek merkezden paket versiyonu yönetimi sağlar.

`Directory.Packages.props` dosyası:
```xml
<Project>
  <ItemGroup>
    <PackageVersion Include="Newtonsoft.Json" Version="13.0.3" />
  </ItemGroup>
</Project>
```

Avantajları:
- Versiyon tutarlılığı
- Büyük solution’larda kontrol

---

## 4.8 Offline & CI/CD Senaryoları

Offline restore:
```bash
dotnet restore --ignore-failed-sources
```

CI/CD için öneriler:
- `dotnet restore` ayrı çalıştır
- Cache kullan
- Sabit versiyon kullan

---

## 4.9 Chapter 4 Özet

- dotnet add/remove/list package
- NuGet kaynak yönetimi
- Cache & restore
- Central Package Management
- CI/CD best practices

---

# Chapter 5 – Global & Local Tools (dotnet tool)

## 5.1 dotnet tool Nedir?

`dotnet tool`, .NET CLI üzerinden çalışan yardımcı araçları (tools)
yönetmek için kullanılır.

Bu araçlar:
- Kod formatlama
- Migration
- CLI yardımcıları
- Analiz ve otomasyon
amaçlıdır.

---

## 5.2 Tool Türleri

| Tür | Açıklama |
|---|---|
| Global Tool | Sisteme global kurulur |
| Local Tool | Proje bazlı kurulur |

---

## 5.3 Global Tool Kurulumu

Global tool, tüm sistemde kullanılabilir.

```bash
dotnet tool install --global dotnet-ef
```

Kurulu global tool’ları listeleme:
```bash
dotnet tool list --global
```

Güncelleme:
```bash
dotnet tool update --global dotnet-ef
```

Kaldırma:
```bash
dotnet tool uninstall --global dotnet-ef
```

---

## 5.4 Local Tool (Tool Manifest)

Local tool’lar proje bazlıdır ve repo ile paylaşılabilir.

### Tool manifest oluşturma
```bash
dotnet new tool-manifest
```

Bu işlem:
```text
.config/dotnet-tools.json
```
dosyasını oluşturur.

---

## 5.5 Local Tool Kurulumu

```bash
dotnet tool install dotnet-ef
```

Local tool’ları listeleme:
```bash
dotnet tool list
```

Restore etme (CI/CD için önemli):
```bash
dotnet tool restore
```

---

## 5.6 Tool Çalıştırma

Local tool çalıştırma:
```bash
dotnet ef migrations add InitialCreate
```

> Global veya local olması fark etmez, kullanım aynıdır.

---

## 5.7 Yaygın Kullanılan Tools

| Tool | Amaç |
|---|---|
| dotnet-ef | Entity Framework CLI |
| dotnet-format | Kod formatlama |
| coverlet | Test coverage |
| reportgenerator | Coverage raporu |

---

## 5.8 CI/CD Best Practices

- Local tool tercih et
- `dotnet tool restore` ekle
- Tool versiyonlarını sabitle
- Repo ile birlikte commit et

---

## 5.9 Chapter 5 Özet

- dotnet tool kavramı
- Global vs Local tool farkı
- Tool manifest
- CI/CD uyumlu kullanım

---

# Chapter 6 – Test, Format & Analyze

## 6.1 dotnet test Nedir?

`dotnet test`, test projelerini derler ve testleri çalıştırır.
Genellikle xUnit, NUnit veya MSTest ile kullanılır.

```bash
dotnet test
```

Varsayılan olarak:
- Restore
- Build
- Test
işlemlerini yapar.

---

## 6.2 Test Projesi Oluşturma

xUnit test projesi:
```bash
dotnet new xunit -n MyApp.Tests
```

Solution’a ekleme:
```bash
dotnet sln add MyApp.Tests/MyApp.Tests.csproj
```

---

## 6.3 dotnet test Parametreleri

Release modunda test:
```bash
dotnet test -c Release
```

Belirli bir test çalıştırma:
```bash
dotnet test --filter FullyQualifiedName~MyTestMethod
```

---

## 6.4 Code Coverage (Coverlet)

Coverlet ile coverage alma:
```bash
dotnet test /p:CollectCoverage=true
```

LCOV formatı:
```bash
dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=lcov
```

---

## 6.5 dotnet format Nedir?

`dotnet format`, kod stilini otomatik düzeltir.

Local tool olarak kurulması önerilir:
```bash
dotnet tool install dotnet-format
```

Çalıştırma:
```bash
dotnet format
```

Sadece kontrol (CI için):
```bash
dotnet format --verify-no-changes
```

---

## 6.6 Kod Analizi (Analyzers)

.NET SDK, yerleşik analyzer’lar içerir.

.csproj içinde aktif etme:
```xml
<PropertyGroup>
  <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
</PropertyGroup>
```

---

## 6.7 dotnet build ile Analyze

Analyzer’lar build sırasında çalışır:
```bash
dotnet build
```

Uyarılar:
- Performans
- Güvenlik
- Kod kalitesi

---

## 6.8 CI/CD Önerilen Akış

```bash
dotnet restore
dotnet build
dotnet test
dotnet format --verify-no-changes
```

---

## 6.9 Chapter 6 Özet

- dotnet test
- Test filtreleme
- Code coverage
- dotnet format
- Analyzer ve kalite kontrol

---

# Chapter 7 – classlib ve Proje Türleri

## 7.1 classlib Nedir?

`classlib`, .NET’te **yeniden kullanılabilir kütüphaneler** oluşturmak için kullanılan
proje türüdür. Çalıştırılabilir değildir; başka projeler tarafından referans alınır.

Kullanım alanları:
- Business logic
- Domain katmanı
- Ortak yardımcı kütüphaneler
- SDK / paket geliştirme

---

## 7.2 classlib Oluşturma

```bash
dotnet new classlib -n MyLibrary
```

Oluşan yapı:
```text
MyLibrary/
 ├─ Class1.cs
 └─ MyLibrary.csproj
```

.NET hedef framework belirtme:
```bash
dotnet new classlib -n MyLibrary -f net8.0
```

---

## 7.3 classlib Referanslama

### Başka projeye referans ekleme

```bash
dotnet add reference ../MyLibrary/MyLibrary.csproj
```

Referans kontrolü:
```bash
dotnet list reference
```

---

## 7.4 Solution İçinde Katmanlı Mimari

Örnek solution:
```text
MySolution.sln
 ├─ MyApp.Api        (Web API)
 ├─ MyApp.Application (classlib)
 ├─ MyApp.Domain      (classlib)
 └─ MyApp.Infrastructure (classlib)
```

Solution oluşturma:
```bash
dotnet new sln -n MySolution
```

Projeleri ekleme:
```bash
dotnet sln add MyApp.Domain/MyApp.Domain.csproj
```

---

## 7.5 Yaygın Proje Türleri

| Proje Türü | Komut |
|---|---|
| Console | dotnet new console |
| Class Library | dotnet new classlib |
| Web API | dotnet new webapi |
| MVC | dotnet new mvc |
| Worker Service | dotnet new worker |
| Test (xUnit) | dotnet new xunit |

Tüm template’ler:
```bash
dotnet new list
```

---

## 7.6 classlib ve NuGet Paketleme

Class library’yi NuGet paketi yapmak:

```bash
dotnet pack -c Release
```

Çıktı:
```text
bin/Release/MyLibrary.1.0.0.nupkg
```

---

## 7.7 Versioning & Metadata

.csproj örneği:
```xml
<PropertyGroup>
  <Version>1.0.0</Version>
  <Authors>YourName</Authors>
  <Description>Reusable class library</Description>
</PropertyGroup>
```

---

## 7.8 CI/CD Senaryosu (Library)

```bash
dotnet restore
dotnet build
dotnet test
dotnet pack -c Release
```

---

## 7.9 Chapter 7 Özet

- classlib nedir, ne zaman kullanılır
- Projeler arası referans
- Katmanlı mimari
- NuGet paketleme
- Library CI akışı

---

# Chapter 8 – DbContext & Entity Framework Core (CLI Odaklı)

## 8.1 DbContext Nedir?

`DbContext`, Entity Framework Core’un veritabanı ile uygulama arasındaki
**ana köprüsüdür**.

Görevleri:
- Veritabanı bağlantısını yönetir
- Entity’leri takip eder (Change Tracking)
- CRUD işlemlerini yürütür
- Migration’ları uygular

---

## 8.2 EF Core Paketleri

Bir classlib (Infrastructure katmanı) içinde önerilir.

Gerekli paketler:
```bash
dotnet add package Microsoft.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
```

---

## 8.3 DbContext Oluşturma

Örnek DbContext:
```csharp
using Microsoft.EntityFrameworkCore;

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options)
        : base(options)
    {
    }

    public DbSet<User> Users => Set<User>();
}
```

---

## 8.4 Entity Tanımı

```csharp
public class User
{
    public int Id { get; set; }
    public string Name { get; set; } = default!;
}
```

---

## 8.5 DbContext Konfigürasyonu

### ASP.NET Core (Program.cs)

```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));
```

### appsettings.json
```json
{
  "ConnectionStrings": {
    "Default": "Server=.;Database=MyDb;Trusted_Connection=True;"
  }
}
```

---

## 8.6 dotnet ef Tool Kurulumu

Local tool olarak önerilir.

```bash
dotnet new tool-manifest
dotnet tool install dotnet-ef
dotnet tool restore
```

---

## 8.7 Migration Oluşturma

```bash
dotnet ef migrations add InitialCreate
```

Infrastructure katmanı için:
```bash
dotnet ef migrations add InitialCreate \
  --project MyApp.Infrastructure \
  --startup-project MyApp.Api
```

---

## 8.8 Database Güncelleme

```bash
dotnet ef database update
```

---

## 8.9 DbContext Lifetime

Varsayılan:
- **Scoped**

Neden?
- Web request başına tek DbContext
- Thread-safe değildir

---

## 8.10 CLI Odaklı EF Akışı

```bash
dotnet restore
dotnet build
dotnet ef migrations add AddUsers
dotnet ef database update
```

---

## 8.11 Chapter 8 Özet

- DbContext kavramı
- EF Core paketleri
- Migration & database update
- dotnet ef CLI kullanımı
- Katmanlı mimari uyumu

---

# Chapter 9 – Program.cs, Dependency Injection & Service Lifetimes

## 9.1 Program.cs Nedir?

`Program.cs`, .NET uygulamasının **giriş noktasıdır**.
.NET 6+ ile birlikte:
- Startup.cs kaldırıldı
- Tüm yapılandırma Program.cs içinde toplandı

Burada:
- Service registration (DI)
- Middleware pipeline
- App configuration
yapılır.

---

## 9.2 Dependency Injection (DI) Nedir?

DI, bağımlılıkların **manuel new ile değil**, container üzerinden yönetilmesidir.

Avantajları:
- Test edilebilirlik
- Gevşek bağlılık (loose coupling)
- Yaşam süresi kontrolü

.NET, **built-in DI container** ile gelir.

---

## 9.3 Service Lifetime Kavramı

.NET’te 3 temel lifetime vardır:

| Lifetime | Açıklama |
|---|---|
| Singleton | Uygulama boyunca tek instance |
| Scoped | Request başına tek instance |
| Transient | Her kullanımda yeni instance |

---

## 9.4 Singleton

Uygulama ayağa kalktığında **bir kez** oluşturulur.

```csharp
builder.Services.AddSingleton<IMailService, MailService>();
```

Özellikler:
- En performanslı
- State taşır
- Thread-safe olmak zorunda

⚠️ **DbContext singleton OLAMAZ**

---

## 9.5 Scoped

Her HTTP request için **bir instance**.

```csharp
builder.Services.AddScoped<IUserService, UserService>();
```

Kullanım alanları:
- DbContext
- Business services
- Unit of Work

```csharp
builder.Services.AddDbContext<AppDbContext>();
```

---

## 9.6 Transient

Her resolve işleminde **yeni instance**.

```csharp
builder.Services.AddTransient<ILogService, LogService>();
```

Kullanım alanları:
- Stateless helper servisler
- Hafif servisler

---

## 9.7 Lifetime Karşılaştırma

```text
Request 1:
  Singleton -> A
  Scoped    -> B
  Transient -> C1, C2

Request 2:
  Singleton -> A
  Scoped    -> D
  Transient -> E1
```

---

## 9.8 Program.cs Tam Örnek

```csharp
var builder = WebApplication.CreateBuilder(args);

// Singleton
builder.Services.AddSingleton<ICacheService, MemoryCacheService>();

// Scoped
builder.Services.AddScoped<IUserService, UserService>();
builder.Services.AddDbContext<AppDbContext>();

// Transient
builder.Services.AddTransient<IEmailSender, SmtpEmailSender>();

builder.Services.AddControllers();

var app = builder.Build();

app.MapControllers();
app.Run();
```

---

## 9.9 Lifetime Hataları (Çok Önemli)

❌ Scoped servisi Singleton içine enjekte etmek:

```csharp
// HATALI
AddSingleton<MyService>(); // MyService içinde DbContext varsa
```

✅ Çözüm:
- Scoped yap
- Veya Factory pattern kullan

---

## 9.10 Factory Kullanımı

```csharp
builder.Services.AddDbContextFactory<AppDbContext>();
```

```csharp
public class MyService
{
    private readonly IDbContextFactory<AppDbContext> _factory;

    public MyService(IDbContextFactory<AppDbContext> factory)
    {
        _factory = factory;
    }
}
```

---

## 9.11 Best Practices

- DbContext → Scoped
- Stateless servis → Transient
- Cache / Config → Singleton
- Local state tutma → dikkat!

---

## 9.12 Chapter 9 Özet

- Program.cs yapısı
- DI ve service registration
- Singleton / Scoped / Transient
- DbContext lifetime kuralları
- Gerçek dünya best practices

---


# Chapter 10 – BackgroundService & HostedService

## 10.1 BackgroundService Nedir?

`BackgroundService`, ASP.NET Core uygulaması çalışırken
arka planda sürekli veya periyodik iş yapmak için kullanılır.

Örnek kullanım alanları:
- Queue consumer
- Scheduled job
- Email sender
- Cache refresh
- Event processing

---

## 10.2 HostedService Kavramı

`IHostedService`, uygulama lifecycle’ına bağlanan servislerdir.

Lifecycle:
- Uygulama başlarken → `StartAsync`
- Uygulama kapanırken → `StopAsync`

`BackgroundService`, `IHostedService`’in hazır bir implementasyonudur.

---

## 10.3 BackgroundService vs IHostedService

| Özellik | BackgroundService | IHostedService |
|---|---|---|
| Sürekli loop | ✅ Var | ❌ Manuel |
| Basit kullanım | ✅ | ❌ |
| Kontrol | Orta | Yüksek |

---

## 10.4 BackgroundService Oluşturma

```csharp
public class Worker : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            Console.WriteLine("Worker çalışıyor...");
            await Task.Delay(5000, stoppingToken);
        }
    }
}
```

---

## 10.5 Program.cs Kaydı

```csharp
builder.Services.AddHostedService<Worker>();
```

> Hosted service’ler **singleton** olarak çalışır.

---

## 10.6 Scoped Service Kullanımı (ÇOK ÖNEMLİ)

BackgroundService singleton olduğu için
**doğrudan DbContext enjekte edilemez** ❌

Yanlış:
```csharp
public Worker(AppDbContext context) { }
```

Doğru:
```csharp
public class Worker : BackgroundService
{
    private readonly IServiceScopeFactory _scopeFactory;

    public Worker(IServiceScopeFactory scopeFactory)
    {
        _scopeFactory = scopeFactory;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            using var scope = _scopeFactory.CreateScope();
            var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();

            // db işlemleri
            await Task.Delay(5000, stoppingToken);
        }
    }
}
```

---

## 10.7 IHostedService ile Manuel Kontrol

```csharp
public class CustomHostedService : IHostedService
{
    public Task StartAsync(CancellationToken cancellationToken)
    {
        Console.WriteLine("App başladı");
        return Task.CompletedTask;
    }

    public Task StopAsync(CancellationToken cancellationToken)
    {
        Console.WriteLine("App duruyor");
        return Task.CompletedTask;
    }
}
```

Kayıt:
```csharp
builder.Services.AddHostedService<CustomHostedService>();
```

---

## 10.8 Zamanlanmış İşler (Timer)

```csharp
protected override async Task ExecuteAsync(CancellationToken stoppingToken)
{
    using var timer = new PeriodicTimer(TimeSpan.FromSeconds(10));

    while (await timer.WaitForNextTickAsync(stoppingToken))
    {
        Console.WriteLine("10 saniyede bir çalışır");
    }
}
```

---

## 10.9 Graceful Shutdown

Uygulama kapanırken:
- `stoppingToken` tetiklenir
- Sonsuz loop’lar düzgün şekilde durdurulmalıdır

```csharp
if (stoppingToken.IsCancellationRequested)
    break;
```

---

## 10.10 Best Practices

- Uzun işlerde cancellation token kullan
- Scoped servisleri scope içinde al
- Blocking işlem yapma
- Logging ekle
- Exception’ları yutma

---

## 10.11 Chapter 10 Özet

- BackgroundService nedir
- IHostedService farkı
- Program.cs kayıtları
- Scoped servis kullanımı
- Gerçek dünya senaryoları

---

# Chapter 11 – Clean Architecture (API + classlib + EF + DI)

## 11.1 Clean Architecture Nedir?

Clean Architecture, uygulamayı **katmanlara ayırarak**
bağımlılıkları içe doğru yöneten bir mimaridir.

Temel hedefler:
- Bağımlılıkların ters çevrilmesi
- Test edilebilirlik
- Framework bağımsız core
- Uzun vadeli sürdürülebilirlik

---

## 11.2 Katmanlar

```text
Presentation (API)
Infrastructure (EF, DB, External)
Application (Use cases)
Domain (Entities)
```

Bağımlılık yönü:
```text
API → Application → Domain
Infrastructure → Application → Domain
```

---

## 11.3 Solution Yapısı

```text
MySolution.sln
 ├─ src/
 │   ├─ MyApp.Api
 │   ├─ MyApp.Application
 │   ├─ MyApp.Domain
 │   └─ MyApp.Infrastructure
 └─ tests/
     └─ MyApp.Tests
```

---

## 11.4 Projeleri Oluşturma (CLI)

```bash
dotnet new sln -n MySolution

dotnet new webapi -n MyApp.Api
dotnet new classlib -n MyApp.Application
dotnet new classlib -n MyApp.Domain
dotnet new classlib -n MyApp.Infrastructure
```

Solution’a ekleme:
```bash
dotnet sln add src/MyApp.Api
dotnet sln add src/MyApp.Application
dotnet sln add src/MyApp.Domain
dotnet sln add src/MyApp.Infrastructure
```

---

## 11.5 Proje Referansları

```text
Api → Application
Application → Domain
Infrastructure → Application
```

CLI ile:
```bash
dotnet add src/MyApp.Api reference src/MyApp.Application
dotnet add src/MyApp.Application reference src/MyApp.Domain
dotnet add src/MyApp.Infrastructure reference src/MyApp.Application
```

---

## 11.6 Domain Katmanı

Sadece **iş kuralları** içerir.

```csharp
public class User
{
    public int Id { get; set; }
    public string Email { get; set; } = default!;
}
```

❌ EF, DI, Logger yok  
✅ Saf C#

---

## 11.7 Application Katmanı

Use case + interface’ler burada olur.

```csharp
public interface IUserRepository
{
    Task<User?> GetByIdAsync(int id);
}
```

```csharp
public class GetUserQuery
{
    private readonly IUserRepository _repo;

    public GetUserQuery(IUserRepository repo)
    {
        _repo = repo;
    }
}
```

---

## 11.8 Infrastructure Katmanı (EF Core)

EF Core implementasyonu burada.

```csharp
public class UserRepository : IUserRepository
{
    private readonly AppDbContext _context;

    public UserRepository(AppDbContext context)
    {
        _context = context;
    }
}
```

DI kaydı:
```csharp
services.AddScoped<IUserRepository, UserRepository>();
```

---

## 11.9 API Katmanı

Sadece:
- HTTP
- DTO
- Request / Response

```csharp
[ApiController]
[Route("api/users")]
public class UsersController : ControllerBase
{
    private readonly IUserRepository _repo;

    public UsersController(IUserRepository repo)
    {
        _repo = repo;
    }
}
```

---

## 11.10 Program.cs (Composition Root)

```csharp
builder.Services.AddApplication();
builder.Services.AddInfrastructure(builder.Configuration);
```

Katmanlar kendi extension’larını sağlar:
```csharp
public static class DependencyInjection
{
    public static IServiceCollection AddInfrastructure(
        this IServiceCollection services, IConfiguration config)
    {
        services.AddDbContext<AppDbContext>();
        services.AddScoped<IUserRepository, UserRepository>();
        return services;
    }
}
```

---

## 11.11 EF Migration Akışı

```bash
dotnet ef migrations add InitialCreate  --project src/MyApp.Infrastructure  --startup-project src/MyApp.Api
```

---

## 11.12 Test Yaklaşımı

- Domain → unit test
- Application → mock repository
- Infrastructure → integration test

---

## 11.13 Chapter 11 Özet

- Clean Architecture prensipleri
- Katmanlı solution yapısı
- CLI ile proje yönetimi
- EF + DI entegrasyonu
- Production-ready mimari

---
# Chapter 12 – Production Ayarları (Logging, Configuration & Secrets)

## 12.1 Production Nedir?

Production ortamı:
- Canlı kullanıcıların eriştiği
- Güvenlik, performans ve stabilitenin kritik olduğu
ortamdır.

Environment değişkeni:
```bash
ASPNETCORE_ENVIRONMENT=Production
```

---

## 12.2 Configuration Sistemi

.NET configuration sırası:

1. appsettings.json
2. appsettings.{Environment}.json
3. Environment variables
4. Command-line args

Örnek:
```json
{
  "ConnectionStrings": {
    "Default": "Server=.;Database=ProdDb;"
  }
}
```

---

## 12.3 appsettings Environment Bazlı Kullanım

```text
appsettings.json
appsettings.Development.json
appsettings.Production.json
```

```csharp
builder.Configuration.GetConnectionString("Default");
```

---

## 12.4 Logging Temelleri

.NET built-in logging sağlar.

```csharp
builder.Logging.ClearProviders();
builder.Logging.AddConsole();
```

Log seviyeleri:
- Trace
- Debug
- Information
- Warning
- Error
- Critical

---

## 12.5 Logging Seviyesi Ayarı

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning"
    }
  }
}
```

Production için:
```json
"Default": "Warning"
```

---

## 12.6 Serilog (Production Önerilir)

Paketler:
```bash
dotnet add package Serilog.AspNetCore
```

Program.cs:
```csharp
builder.Host.UseSerilog((ctx, cfg) =>
{
    cfg.ReadFrom.Configuration(ctx.Configuration)
       .WriteTo.Console();
});
```

---

## 12.7 Secrets Yönetimi

❌ appsettings.json içine secret koyma

### Environment Variable
```bash
export ConnectionStrings__Default="..."
```

### User Secrets (Local)
```bash
dotnet user-secrets init
dotnet user-secrets set "ConnectionStrings:Default" "..."
```

---

## 12.8 Azure / Docker Secrets Mantığı

- Azure Key Vault
- Docker secrets
- Kubernetes secrets

Hepsi config pipeline’a entegre olur.

---

## 12.9 Health Checks

```bash
dotnet add package Microsoft.Extensions.Diagnostics.HealthChecks
```

Program.cs:
```csharp
builder.Services.AddHealthChecks();
app.MapHealthChecks("/health");
```

---

## 12.10 Production Best Practices

- Logging seviyesini düşür
- Secret’ları environment’a taşı
- Health check ekle
- Exception detaylarını gizle
- Migration’ları kontrollü çalıştır

---


# Mini Örnek – classlib Nedir? Nasıl Kullanılır?

Bu mini örnek, **class library (classlib)** kavramını hızlıca ve net şekilde
anlaman için hazırlanmıştır.

---

## 1️⃣ Senaryo

Bir uygulamada **KDV hesaplama** logic’ini:
- API’den
- Console’dan
- Test projelerinden

tekrar tekrar kullanmak istiyoruz.

➡️ Bu logic **classlib** içinde olmalı.

---

## 2️⃣ Class Library Oluşturma

```bash
dotnet new classlib -n TaxCalculator
```

Oluşan yapı:
```text
TaxCalculator/
 ├─ Class1.cs
 └─ TaxCalculator.csproj
```

`Class1.cs` dosyasını silebilirsin.

---

## 3️⃣ Classlib İçeriği

📄 `TaxService.cs`

```csharp
namespace TaxCalculator;

public class TaxService
{
    public decimal CalculateVat(decimal price)
    {
        return price * 0.20m;
    }
}
```

🔹 Bu proje:
- Çalıştırılamaz
- Sadece **referans alınır**

---

## 4️⃣ Console App Oluşturma

```bash
dotnet new console -n ShopApp
```

---

## 5️⃣ Classlib Referans Ekleme

```bash
dotnet add ShopApp reference TaxCalculator
```

Kontrol:
```bash
dotnet list ShopApp reference
```

---

## 6️⃣ Classlib Kullanımı

📄 `Program.cs`

```csharp
using TaxCalculator;

var taxService = new TaxService();

var price = 100m;
var vat = taxService.CalculateVat(price);

Console.WriteLine($"KDV: {vat}");
```

Çalıştır:
```bash
dotnet run --project ShopApp
```

Çıktı:
```text
KDV: 20
```

---

## 7️⃣ Classlib Ne Zaman Kullanılır?

✅ Kullan:
- Business logic
- Domain kuralları
- Ortak helper’lar
- Repository / Service katmanı

❌ Kullanma:
- UI
- HTTP / Controller
- Console giriş noktası

---

## 8️⃣ Gerçek Proje İpucu

Clean Architecture’da:
```text
Domain        → classlib
Application   → classlib
Infrastructure→ classlib
API           → webapi
```

---


# Mini Örnek – classlib + Dependency Injection (AddScoped)

Bu mini örnek, bir **class library (classlib)** içindeki servisin
**Dependency Injection (DI)** ile nasıl kullanıldığını net şekilde gösterir.

---

## 1️⃣ Senaryo

Bir uygulamada **fiyat + KDV hesaplama** servisi olsun.
Bu servis:
- API’de
- Console app’te
- Test’lerde

aynı şekilde kullanılabilsin.

➡️ Logic **classlib**, yaşam süresi **Scoped** olacak.

---

## 2️⃣ Class Library Oluşturma

```bash
dotnet new classlib -n Pricing
```

---

## 3️⃣ Servis ve Interface Tanımı (classlib)

📄 `IPricingService.cs`

```csharp
namespace Pricing;

public interface IPricingService
{
    decimal CalculateTotal(decimal price);
}
```

📄 `PricingService.cs`

```csharp
namespace Pricing;

public class PricingService : IPricingService
{
    public decimal CalculateTotal(decimal price)
    {
        var vat = price * 0.20m;
        return price + vat;
    }
}
```

➡️ classlib içinde **DI yok**, sadece kod var.

---

## 4️⃣ API Projesi Oluşturma

```bash
dotnet new webapi -n Shop.Api
```

---

## 5️⃣ Classlib Referans Ekleme

```bash
dotnet add Shop.Api reference Pricing
```

---

## 6️⃣ Program.cs – AddScoped ile Kayıt

📄 `Program.cs`

```csharp
using Pricing;

var builder = WebApplication.CreateBuilder(args);

// DI registration
builder.Services.AddScoped<IPricingService, PricingService>();

builder.Services.AddControllers();
var app = builder.Build();

app.MapControllers();
app.Run();
```

🔹 `AddScoped` → her HTTP request için **tek instance**

---

## 7️⃣ Controller İçinde Kullanım

📄 `PricingController.cs`

```csharp
using Microsoft.AspNetCore.Mvc;
using Pricing;

[ApiController]
[Route("api/pricing")]
public class PricingController : ControllerBase
{
    private readonly IPricingService _pricingService;

    public PricingController(IPricingService pricingService)
    {
        _pricingService = pricingService;
    }

    [HttpGet("{price}")]
    public decimal Get(decimal price)
    {
        return _pricingService.CalculateTotal(price);
    }
}
```

---

## 8️⃣ Test Etme

```bash
dotnet run --project Shop.Api
```

Tarayıcı:
```text
https://localhost:5001/api/pricing/100
```

Çıktı:
```text
120
```

---

## 9️⃣ Neden AddScoped?

| Lifetime | Durum |
|---|---|
| Singleton | ❌ State riski |
| Scoped | ✅ Request bazlı |
| Transient | ⚠️ Gereksiz instance |

➡️ **Business servisler için Scoped en doğru tercihtir**

---

## 🔟 Gerçek Proje İpucu

Clean Architecture’da:
```text
Application → Interface
Infrastructure → Implementation
API → AddScoped
```

---
# Mini Örnek – classlib + DbContext (AddDbContext)

Bu mini örnek, **Entity Framework Core DbContext**’in
bir **class library (classlib)** içinde tanımlanıp
**API tarafında AddDbContext ile bağlanmasını** gösterir.

---

## 1️⃣ Senaryo

- Veritabanı erişimi **classlib** içinde olacak
- API sadece DI ve HTTP ile ilgilenecek
- DbContext **Scoped** olarak yönetilecek

➡️ Gerçek dünya Clean Architecture yaklaşımı

---

## 2️⃣ Infrastructure Class Library Oluşturma

```bash
dotnet new classlib -n DataAccess
```

---

## 3️⃣ EF Core Paketlerini Ekleme (classlib)

```bash
dotnet add DataAccess package Microsoft.EntityFrameworkCore
dotnet add DataAccess package Microsoft.EntityFrameworkCore.SqlServer
dotnet add DataAccess package Microsoft.EntityFrameworkCore.Design
```

---

## 4️⃣ Entity Tanımı (classlib)

📄 `Entities/User.cs`

```csharp
namespace DataAccess.Entities;

public class User
{
    public int Id { get; set; }
    public string Name { get; set; } = default!;
}
```

---

## 5️⃣ DbContext Tanımı (classlib)

📄 `AppDbContext.cs`

```csharp
using Microsoft.EntityFrameworkCore;
using DataAccess.Entities;

namespace DataAccess;

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options)
        : base(options)
    {
    }

    public DbSet<User> Users => Set<User>();
}
```

➡️ DbContext **classlib içinde**, DI yok.

---

## 6️⃣ API Projesi Oluşturma

```bash
dotnet new webapi -n Users.Api
```

---

## 7️⃣ Classlib Referans Ekleme

```bash
dotnet add Users.Api reference DataAccess
```

---

## 8️⃣ Program.cs – AddDbContext

📄 `Program.cs`

```csharp
using DataAccess;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("Default")
    ));

builder.Services.AddControllers();

var app = builder.Build();
app.MapControllers();
app.Run();
```

🔹 `AddDbContext` → **Scoped**
🔹 Request başına tek DbContext

---

## 9️⃣ appsettings.json

```json
{
  "ConnectionStrings": {
    "Default": "Server=.;Database=UsersDb;Trusted_Connection=True;"
  }
}
```

---

## 🔟 DbContext Kullanımı (Controller)

📄 `UsersController.cs`

```csharp
using Microsoft.AspNetCore.Mvc;
using DataAccess;

[ApiController]
[Route("api/users")]
public class UsersController : ControllerBase
{
    private readonly AppDbContext _context;

    public UsersController(AppDbContext context)
    {
        _context = context;
    }

    [HttpGet]
    public IEnumerable<object> Get()
    {
        return _context.Users
            .Select(u => new { u.Id, u.Name })
            .ToList();
    }
}
```

---

## 1️⃣1️⃣ Migration Oluşturma (CLI)

```bash
dotnet tool install dotnet-ef
dotnet ef migrations add InitialCreate \
  --project DataAccess \
  --startup-project Users.Api
```

Database update:
```bash
dotnet ef database update
```

---

## 1️⃣2️⃣ Neden AddDbContext Scoped?

❌ Singleton → Thread-safe değil  
❌ Transient → Connection overhead  
✅ Scoped → Request bazlı, güvenli

---

## 1️⃣3️⃣ Gerçek Proje İpucu

Clean Architecture:
```text
Infrastructure → DbContext
Application    → Interface
API            → AddDbContext
```

---

# Mini Örnek – classlib + Repository Pattern

Bu mini örnek, **Repository Pattern**’in
bir **class library (classlib)** içinde nasıl uygulanacağını
ve API tarafından nasıl kullanılacağını gösterir.

---

## 1️⃣ Senaryo

- Veritabanı erişimi EF Core ile yapılacak
- DbContext doğrudan API’de kullanılmayacak
- API → Repository arayüzü ile konuşacak

➡️ Amaç: **gevşek bağlılık + test edilebilirlik**

---

## 2️⃣ Proje Yapısı

```text
MySolution
 ├─ MyApp.Api
 ├─ MyApp.Domain        (Entities)
 ├─ MyApp.Application   (Interfaces)
 └─ MyApp.Infrastructure (EF + Repository)
```

---

## 3️⃣ Domain – Entity (classlib)

📄 `User.cs`

```csharp
namespace MyApp.Domain;

public class User
{
    public int Id { get; set; }
    public string Name { get; set; } = default!;
}
```

---

## 4️⃣ Application – Repository Interface (classlib)

📄 `IUserRepository.cs`

```csharp
using MyApp.Domain;

namespace MyApp.Application;

public interface IUserRepository
{
    Task<IEnumerable<User>> GetAllAsync();
    Task<User?> GetByIdAsync(int id);
    Task AddAsync(User user);
}
```

➡️ **EF, DbContext yok**

---

## 5️⃣ Infrastructure – DbContext (classlib)

📄 `AppDbContext.cs`

```csharp
using Microsoft.EntityFrameworkCore;
using MyApp.Domain;

namespace MyApp.Infrastructure;

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options)
        : base(options) { }

    public DbSet<User> Users => Set<User>();
}
```

---

## 6️⃣ Infrastructure – Repository Implementasyonu

📄 `UserRepository.cs`

```csharp
using Microsoft.EntityFrameworkCore;
using MyApp.Application;
using MyApp.Domain;

namespace MyApp.Infrastructure;

public class UserRepository : IUserRepository
{
    private readonly AppDbContext _context;

    public UserRepository(AppDbContext context)
    {
        _context = context;
    }

    public async Task<IEnumerable<User>> GetAllAsync()
    {
        return await _context.Users.AsNoTracking().ToListAsync();
    }

    public async Task<User?> GetByIdAsync(int id)
    {
        return await _context.Users.FindAsync(id);
    }

    public async Task AddAsync(User user)
    {
        _context.Users.Add(user);
        await _context.SaveChangesAsync();
    }
}
```

---

## 7️⃣ API – DI Kayıtları (Program.cs)

```csharp
using MyApp.Application;
using MyApp.Infrastructure;
using Microsoft.EntityFrameworkCore;

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));

builder.Services.AddScoped<IUserRepository, UserRepository>();
```

---

## 8️⃣ API – Controller Kullanımı

📄 `UsersController.cs`

```csharp
using Microsoft.AspNetCore.Mvc;
using MyApp.Application;
using MyApp.Domain;

[ApiController]
[Route("api/users")]
public class UsersController : ControllerBase
{
    private readonly IUserRepository _repo;

    public UsersController(IUserRepository repo)
    {
        _repo = repo;
    }

    [HttpGet]
    public async Task<IEnumerable<User>> Get()
    {
        return await _repo.GetAllAsync();
    }

    [HttpPost]
    public async Task<IActionResult> Post(User user)
    {
        await _repo.AddAsync(user);
        return Ok();
    }
}
```

---

## 9️⃣ Repository Pattern Neden?

❌ DbContext’i her yerde kullanmak  
❌ EF’e sıkı bağımlılık  

✅ Interface ile soyutlama  
✅ Testlerde mock repository  
✅ EF değişse bile Application bozulmaz  

---

## 🔟 Repository + DI Lifetime

- DbContext → Scoped
- Repository → Scoped

```csharp
builder.Services.AddScoped<IUserRepository, UserRepository>();
```

---

## 1️⃣1️⃣ Gerçek Proje İpucu

- Application → sadece interface
- Infrastructure → EF + implementasyon
- API → sadece DI + HTTP



---

# Scoped mi Singleton mı? (Mini Cheat Sheet)

Bu doküman, **Dependency Injection lifetime** kararını hızlı ve net vermen için hazırlanmıştır.

---

## Altın Kural

> **Request / DbContext / kullanıcıya bağlıysa → Scoped**  
> **Stateless / cache / config ise → Singleton**

---

## Lifetime Türleri

| Lifetime | Ömür | Ne Zaman |
|---|---|---|
| Singleton | App boyunca | Stateless, cache, config |
| Scoped | Request boyunca | DbContext, repository |
| Transient | Her resolve | Hafif, stateless |

---

## Ne Zaman Scoped?

Aşağıdakilerden biri varsa **Scoped**:

- DbContext kullanıyorsa
- Repository ise
- Business logic içeriyorsa
- Transaction yönetiyorsa
- Request bazlı state varsa

```csharp
builder.Services.AddScoped<AppDbContext>();
builder.Services.AddScoped<IUserRepository, UserRepository>();
builder.Services.AddScoped<IOrderService, OrderService>();
```

---

## Ne Zaman Singleton?

Aşağıdakilerden hepsi sağlanıyorsa **Singleton**:

- Stateless
- Thread-safe
- DbContext KULLANMIYOR
- Request bağımsız

```csharp
builder.Services.AddSingleton<ICacheService, MemoryCacheService>();
builder.Services.AddSingleton<IClock, SystemClock>();
```

---

## En Büyük Hata ❌

Singleton içine Scoped enjekte etmek:

```csharp
// HATALI
builder.Services.AddSingleton<MyService>();

public MyService(AppDbContext context) { }
```

---

## Doğru Çözüm

### Çözüm 1 – Scoped Yap

```csharp
builder.Services.AddScoped<MyService>();
```

### Çözüm 2 – Factory Kullan

```csharp
builder.Services.AddDbContextFactory<AppDbContext>();
```

---

## Karar Ağacı

```
DbContext var mı?
 └─ EVET → Scoped

Request’e bağlı mı?
 └─ EVET → Scoped

Stateless mi?
 └─ EVET → Singleton

Thread-safe mi?
 └─ HAYIR → Singleton OLMAZ
```

---

## Gerçek Hayat Tablosu

| Servis | Lifetime |
|---|---|
| DbContext | Scoped |
| Repository | Scoped |
| Business Service | Scoped |
| Cache | Singleton |
| Logger | Singleton |
| HttpClient | Singleton |
| BackgroundService | Singleton |

---

## Not (BackgroundService)

BackgroundService **her zaman Singleton**’dır.  
DbContext **doğrudan enjekte edilmez**.

---

# DI Lifetime Hataları (Gerçek Bug Örnekleri)

## Hata 1 – Singleton + DbContext ❌
Runtime exception + data corruption.

## Hata 2 – Transient DbContext ❌
Connection pool şişer.

## Hata 3 – Scoped servis singleton içinde ❌
Memory leak + race condition.

## Altın Kural
> DbContext ve Repository **her zaman Scoped**


# Mini Örnek – Seed Data (DbContext + DI + EF Core)

Bu doküman, **Entity Framework Core** kullanarak
uygulama ilk ayağa kalktığında **seed (başlangıç verisi)** eklemenin
en temiz ve güvenli yollarını gösterir.

---

## 1️⃣ Seed Nedir?

Seed:
- Uygulama ilk çalıştığında
- Veritabanı boşsa
- Varsayılan kayıtların eklenmesidir

Örnek:
- Admin user
- Varsayılan roller
- Sabit lookup tabloları

---

## 2️⃣ Seed Ne Zaman Kullanılır?

✅ Kullan:
- Lookup data (Role, Status, Category)
- Default admin
- Test / demo data

❌ Kullanma:
- Büyük production dataları
- Kullanıcıya özel data

---

## 3️⃣ Yöntem 1 – DbContext OnModelCreating (Statik Seed)

📄 `AppDbContext.cs`

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Role>().HasData(
        new Role { Id = 1, Name = "Admin" },
        new Role { Id = 2, Name = "User" }
    );
}
```

📌 Özellikler:
- Migration’a yazılır
- Değişiklik migration gerektirir
- Lookup data için idealdir

---

## 4️⃣ Yöntem 2 – Program.cs Seed (En Yaygın)

📄 `Program.cs`

```csharp
using (var scope = app.Services.CreateScope())
{
    var context = scope.ServiceProvider.GetRequiredService<AppDbContext>();
    context.Database.Migrate();

    if (!context.Users.Any())
    {
        context.Users.Add(new User
        {
            Name = "Admin"
        });

        context.SaveChanges();
    }
}
```

📌 Özellikler:
- Runtime çalışır
- Esnek
- Production’da kontrollü kullanılmalı

---

## 5️⃣ Yöntem 3 – Seed Service (EN TEMİZ YÖNTEM)

### Interface (Application)

```csharp
public interface IDataSeeder
{
    Task SeedAsync();
}
```

### Implementasyon (Infrastructure)

```csharp
public class DataSeeder : IDataSeeder
{
    private readonly AppDbContext _context;

    public DataSeeder(AppDbContext context)
    {
        _context = context;
    }

    public async Task SeedAsync()
    {
        if (!_context.Users.Any())
        {
            _context.Users.Add(new User { Name = "Admin" });
            await _context.SaveChangesAsync();
        }
    }
}
```

### DI Kaydı

```csharp
builder.Services.AddScoped<IDataSeeder, DataSeeder>();
```

### Program.cs Çağırma

```csharp
using (var scope = app.Services.CreateScope())
{
    var seeder = scope.ServiceProvider.GetRequiredService<IDataSeeder>();
    await seeder.SeedAsync();
}
```

📌 **Clean Architecture uyumlu**

---

## 6️⃣ Production İçin Güvenlik

```csharp
if (app.Environment.IsDevelopment())
{
    // seed çalıştır
}
```

veya:

```csharp
if (!context.Users.Any())
```

---

## 7️⃣ Seed + Migration Akışı

```bash
dotnet ef migrations add InitialCreate
dotnet ef database update
dotnet run
```

---

## 8️⃣ En Sık Yapılan Hatalar ❌

- Her startup’ta duplicate data
- Production’da test data
- Seed logic’i DbContext içine gömmek
- Singleton seeder kullanmak

---

# SeedService Nedir? (Clean Architecture Uyumlu Seed)

Bu doküman, **SeedService** yaklaşımını
neden ve nasıl kullanman gerektiğini **net ve pratik** şekilde açıklar.

---

## 1️⃣ SeedService Nedir?

**SeedService**, veritabanına başlangıç verilerini eklemek için yazılan
**ayrı bir servistir**.

Amaç:
- Seed logic’i `Program.cs`’ten ayırmak
- Test edilebilirlik
- Clean Architecture uyumu
- Kontrol edilebilir seed süreci

---

## 2️⃣ Neden SeedService Kullanılır?

❌ Yanlış yaklaşımlar:
- Seed kodunu `DbContext` içine yazmak
- `Program.cs`’i şişirmek
- Her startup’ta kontrolsüz veri eklemek

✅ SeedService ile:
- Seed mantığı **tek yerde**
- Kolay test edilir
- Ortam bazlı (Dev / Prod) kontrol edilir

---

## 3️⃣ Katmanlı Yapı (Önerilen)

```text
Application      → Interface
Infrastructure   → Implementation
API              → Çalıştırır
```

---

## 4️⃣ Application – Interface

📄 `IDataSeeder.cs`

```csharp
namespace MyApp.Application;

public interface IDataSeeder
{
    Task SeedAsync();
}
```

➡️ **Hiç EF, DbContext yok**

---

## 5️⃣ Infrastructure – Implementasyon

📄 `DataSeeder.cs`

```csharp
using MyApp.Application;
using MyApp.Domain;

namespace MyApp.Infrastructure;

public class DataSeeder : IDataSeeder
{
    private readonly AppDbContext _context;

    public DataSeeder(AppDbContext context)
    {
        _context = context;
    }

    public async Task SeedAsync()
    {
        if (!_context.Roles.Any())
        {
            _context.Roles.AddRange(
                new Role { Name = "Admin" },
                new Role { Name = "User" }
            );

            await _context.SaveChangesAsync();
        }
    }
}
```

📌 Kurallar:
- **Idempotent** olmalı (`Any()` kontrolü)
- DbContext → Scoped

---

## 6️⃣ DI Kaydı

📄 `Infrastructure.DependencyInjection.cs`

```csharp
services.AddScoped<IDataSeeder, DataSeeder>();
```

---

## 7️⃣ Program.cs – Seed Çalıştırma

📄 `Program.cs`

```csharp
using (var scope = app.Services.CreateScope())
{
    var seeder = scope.ServiceProvider.GetRequiredService<IDataSeeder>();
    await seeder.SeedAsync();
}
```

➡️ Program.cs **sade kalır**

---

## 8️⃣ Environment Bazlı Seed

```csharp
if (app.Environment.IsDevelopment())
{
    await seeder.SeedAsync();
}
```

veya:

```csharp
if (!context.Roles.Any())
```

---

## 9️⃣ SeedService Lifetime

| Servis | Lifetime |
|---|---|
| DbContext | Scoped |
| SeedService | Scoped |

❌ Singleton OLMAZ

---

## 🔟 En Sık Yapılan Hatalar

- SeedService’i Singleton yapmak
- Her startup’ta duplicate data
- Production’a test seed’i sokmak
- Migration yerine seed ile schema yönetmek

---

## 1️⃣1️⃣ Ne Zaman SeedService?

✅ Kullan:
- Admin / Role seed
- Demo data
- Environment bazlı seed

❌ Kullanma:
- Büyük bulk data
- Gerçek user dataları

---












