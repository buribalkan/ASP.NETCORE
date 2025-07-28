# ASP.NET Core CLI ile Model Oluşturma Yöntemleri

ASP.NET Core’da model sınıfını komut satırından (CLI) otomatik olarak oluşturmak için doğrudan bir “model oluştur” komutu yoktur. Ancak aşağıdaki yollarla model ve ilişkili bileşenler oluşturulabilir:

---

## 1. Entity Framework Core ile Var Olan Veritabanından Model (Reverse Engineering)

Eğer veritabanın varsa, EF Core CLI komutları ile model ve `DbContext` otomatik oluşturulabilir:

```bash
dotnet ef dbcontext scaffold "Connection_String" Microsoft.EntityFrameworkCore.SqlServer -o Models
```

- `"Connection_String"` yerine kendi bağlantı dizini girilmeli
- `-o Models`: Oluşturulan dosyalar `Models/` klasörüne gelir

🔧 Bu komut, veritabanındaki tüm tabloları `DbContext` ve C# model sınıfı olarak üretir.

---

## 2. Koddan Model Yazmak İçin IDE veya Editör Kullan

Model sınıfları genelde manuel yazılır. Örneğin:

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
}
```

👨‍💻 Visual Studio kullanıyorsan: `Add > Class > Product.cs` diyerek oluşturabilirsin.

---

## 3. `dotnet-aspnet-codegenerator` ile Otomatik CRUD Sayfaları Oluşturma

Eğer bir modelin hazırsa ve ona ait Razor Pages CRUD arayüzü oluşturmak istersen:

```bash
dotnet aspnet-codegenerator razorpage -m Product -dc AppDbContext -outDir Pages/Products --referenceScriptLibraries
```

- `-m`: Model adı
- `-dc`: `DbContext` sınıf adı
- `-outDir`: Oluşturulan sayfaların dizini
- `--referenceScriptLibraries`: JavaScript doğrulama script’lerini ekler

⚡ Bu komut otomatik olarak `Create`, `Edit`, `Delete`, `Details`, `Index` Razor Pages oluşturur.

---

## 4. Model Şablonu ile Manuel Sınıf Oluşturma (CLI)

Model dosyasını elle oluşturmak istersen:

```bash
dotnet new class -n Product -o Models
```

Bu komut şunu yapar:

- `Models/Product.cs` adında boş bir C# sınıfı oluşturur
- Daha sonra içine özellikleri manuel olarak yazarsın

---

## 🧠 Özet Tablo

| Senaryo                      | Komut / Yöntem                                       |
|------------------------------|------------------------------------------------------|
| Veritabanından Model Oluştur | `dotnet ef dbcontext scaffold ...`                  |
| CRUD Scaffold                | `dotnet aspnet-codegenerator razorpage ...`         |
| Sadece Boş Sınıf Oluşturma   | `dotnet new class -n ModelName -o Folder`           |

---


 
