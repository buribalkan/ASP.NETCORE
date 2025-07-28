
# ASP.NET Core’da Model Oluşturma

ASP.NET Core’da model oluşturmak aslında C# sınıfı (class) tanımlamak demek. Model, veritabanındaki tabloyu veya uygulamadaki veri yapısını temsil eder. İşte basit ve anlaşılır şekilde model oluşturma anlatımı:

---

## 1. Model Nedir?

- Veritabanındaki tabloyu temsil eden sınıf  
- Uygulama içinde veri taşıyan yapı  
- Özellikleri (properties) kolonları karşılar  

---

## 2. Basit Model Örneği

```csharp
public class Product
{
    public int Id { get; set; }  // Primary key (genellikle)
    public string Name { get; set; } = string.Empty;  // Ürün adı
    public decimal Price { get; set; }  // Ürün fiyatı
}
```

---

## 3. Modelde Veri Doğrulama (Validation)

DataAnnotations kullanarak kurallar ekleyebilirsin:

```csharp
using System.ComponentModel.DataAnnotations;

public class Product
{
    public int Id { get; set; }

    [Required(ErrorMessage = "Name is required")]
    [StringLength(100, MinimumLength = 2)]
    public string Name { get; set; } = string.Empty;

    [Range(0.01, 10000)]
    public decimal Price { get; set; }
}
```

---

## 4. DbContext İçinde Modeli Tanımlama

```csharp
public class AppDbContext : DbContext
{
    public DbSet<Product> Products { get; set; } = null!;

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        => optionsBuilder.UseSqlServer("YourConnectionStringHere");
}
```

---

## 5. Veritabanı için Migration Oluşturma

Terminalde:

```bash
dotnet ef migrations add InitialCreate
dotnet ef database update
```

---

## Özet

| Adım           | Açıklama                                  |
|----------------|-------------------------------------------|
| Model oluştur  | `class` ile özellikleri tanımla           |
| Validation     | `[Required]`, `[Range]` gibi kullan       |
| DbContext      | `DbSet<Model>` ile bağla                  |
| Migration      | `dotnet ef` ile veritabanı oluştur        |
