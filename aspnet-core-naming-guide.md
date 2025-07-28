# ASP.NET Core / C# Project Naming Convention Guide

ASP.NET Core projelerinde **model**, **controller**, **view** ve diğer bileşenlerin isimlendirilmesi için genel kabul görmüş standartlar ve pratikler vardır. Bunlara uyarsan kodun daha okunabilir, tutarlı ve sürdürülebilir olur.

---

## 1. Sınıf ve Model İsimlendirmesi

- **PascalCase** kullanılır.
- Her kelimenin ilk harfi büyük olmalıdır.
- Örnekler:
  - `Product`
  - `CustomerOrder`
  - `UserProfile`

---

## 2. Controller İsimlendirmesi

- Controller sınıf isimleri **PascalCase** olmalı ve `Controller` eki ile bitmelidir.
- Örnekler:
  - `ProductsController`
  - `AccountController`
  - `HomeController`

---

## 3. Property / Özellik İsimlendirmesi

- Model veya sınıf içindeki özellikler **PascalCase** kullanır.
- Örnek:

```csharp
public int ProductId { get; set; }
public string FirstName { get; set; }
```

---

## 4. Local Variables / Parametreler

- **camelCase** kullanılır.
- İlk harf küçük, sonraki kelimeler büyük harfle başlar.
- Örnek:

```csharp
int productCount;
string userName;
```

---

## 5. Namespace ve Klasör İsimleri

- **PascalCase** kullanılır.
- Örnek:
  - `MyApp.Models`
  - `MyApp.Controllers`
  - `MyApp.Services`

---

## 6. View ve Razor Page Dosya İsimleri

- Controller veya model isimleriyle uyumlu olmalıdır.
- PascalCase veya küçük harfli olabilir.
- Örnek: `ProductsController` için view klasörü `Views/Products` olur.

---

## Örnek Hiyerarşi ve İsimlendirme

```
/Models
   Product.cs
   UserProfile.cs

/Controllers
   ProductsController.cs
   AccountController.cs

/Views
   /Products
       Index.cshtml
       Create.cshtml
```

---

## Özet Tablo

| Tür                  | İsimlendirme Stili      | Örnek                      |
|----------------------|--------------------------|-----------------------------|
| Sınıf / Model        | PascalCase               | `Product`, `OrderItem`     |
| Controller           | PascalCase + Controller  | `HomeController`           |
| Özellik / Property   | PascalCase               | `FirstName`                |
| Değişken / Parametre | camelCase                | `userId`, `orderDate`      |
| Namespace / Klasör   | PascalCase               | `MyApp.Services`           |

---

## Neden?

- C# topluluğu ve Microsoft standartları bu şekildedir.
- Kod okunabilirliği ve işbirliği kolaylaşır.
- Framework ve tooling ile uyumlu olur.

---
