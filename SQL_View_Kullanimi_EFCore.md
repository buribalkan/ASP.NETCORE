
# ASP.NET Core Uygulamasında SQL View Kullanımı (EF Core ile)

ASP.NET Core uygulamasında SQL View kullanmak genellikle Entity Framework Core (EF Core) ile yapılır.  
View, veritabanında tanımlanmış sanal bir tablo gibi davranır; EF Core üzerinden sorgulanabilir ama genellikle güncellenemez.

Aşağıda adım adım SQL View’in EF Core ve ASP.NET Core’da kullanımı anlatılmıştır:

---

## 1. Veritabanında View Oluşturma

Örneğin, aşağıdaki gibi bir view oluşturalım:

```sql
CREATE VIEW ActiveUsers AS
SELECT Id, Username, Email
FROM Users
WHERE IsActive = 1;
```

---

## 2. EF Core Model Sınıfı Oluşturma

View yapısına uygun bir POCO sınıf oluştur:

```csharp
public class ActiveUser
{
    public int Id { get; set; }
    public string Username { get; set; } = "";
    public string Email { get; set; } = "";
}
```

---

## 3. DbContext’te View’i Tanımlama

EF Core 5+ ile view’ları aşağıdaki gibi modelle:

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<ActiveUser>(entity =>
    {
        entity.HasNoKey(); // View’larda genellikle PK yoktur
        entity.ToView("ActiveUsers"); // View ismi
        entity.Property(v => v.Id).HasColumnName("Id");
        entity.Property(v => v.Username).HasColumnName("Username");
        entity.Property(v => v.Email).HasColumnName("Email");
    });
}
```

---

## 4. View’dan Veri Okuma (Örnek Controller)

```csharp
public class UsersController : Controller
{
    private readonly AppDbContext _context;

    public UsersController(AppDbContext context)
    {
        _context = context;
    }

    public IActionResult ActiveUsers()
    {
        var activeUsers = _context.Set<ActiveUser>().ToList();
        return View(activeUsers);
    }
}
```

---

## 5. Razor View (Basit Listeleme)

```razor
@model List<ActiveUser>

<h2>Active Users</h2>

<table>
    <thead>
        <tr><th>Id</th><th>Username</th><th>Email</th></tr>
    </thead>
    <tbody>
    @foreach(var user in Model)
    {
        <tr>
            <td>@user.Id</td>
            <td>@user.Username</td>
            <td>@user.Email</td>
        </tr>
    }
    </tbody>
</table>
```

---

## Özet

| Adım               | Açıklama                               |
|--------------------|----------------------------------------|
| View oluştur       | SQL’de `CREATE VIEW` ile               |
| Model oluştur      | View sütunlarına uygun POCO            |
| DbContext yapılandır | `HasNoKey()`, `ToView("ViewName")`   |
| Veri çekme         | `_context.Set<ViewModel>().ToList()`   |
| UI’de gösterme     | Razor sayfasında listeleme             |
