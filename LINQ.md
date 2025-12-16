# LINQ Masterclass – Chapter 1: Temeller (Nedir, Neden, Nasıl?)

Bu bölüm LINQ’i **temelden**, anlaşılır örneklerle başlatır.
Hedef: LINQ’i korkutmadan öğretmek.

---

## 1️⃣ LINQ Nedir?

**LINQ (Language Integrated Query)**, C# içinde:
- Koleksiyonları
- Veritabanını
- XML / JSON benzeri verileri

**SQL yazar gibi** sorgulamanı sağlar.

```csharp
var result = numbers.Where(x => x > 10);
```

---

## 2️⃣ LINQ Neden Var?

LINQ olmadan:
```csharp
var result = new List<int>();
foreach (var n in numbers)
{
    if (n > 10)
        result.Add(n);
}
```

LINQ ile:
```csharp
var result = numbers.Where(n => n > 10);
```

✅ Daha okunur  
✅ Daha az hata  
✅ Daha deklaratif  

---

## 3️⃣ LINQ Nerelerde Kullanılır?

- In-memory collections (`List<T>`)
- Entity Framework (DbSet)
- API response shaping
- Filtering / projection
- Reporting

---

## 4️⃣ LINQ Temel Yapı

```csharp
kaynak
    .Where(...)
    .Select(...)
    .OrderBy(...)
    .ToList();
```

Her LINQ sorgusu **zincir** şeklindedir.

---

## 5️⃣ Deferred Execution (ÇOK ÖNEMLİ)

LINQ sorguları **hemen çalışmaz**.

```csharp
var query = users.Where(u => u.IsActive);
// burada DB'ye gitmez

var list = query.ToList();
// burada çalışır
```

❗ `ToList()`, `First()`, `Count()` → **execute eder**

---

## 6️⃣ IEnumerable vs IQueryable

| Tür | Nerede Çalışır |
|---|---|
| IEnumerable | Memory |
| IQueryable | Database (EF) |

```csharp
IQueryable<User> q = context.Users;
IEnumerable<User> e = usersList;
```

❗ EF tarafında **IQueryable** korunmalı

---

## 7️⃣ İlk LINQ Operatörleri

### Where (Filter)

```csharp
var adults = users.Where(u => u.Age >= 18);
```

### Select (Projection)

```csharp
var names = users.Select(u => u.Name);
```

### OrderBy

```csharp
var sorted = users.OrderBy(u => u.Name);
```

---

## 8️⃣ LINQ + classlib Örneği

```csharp
public IEnumerable<User> GetActiveUsers(IEnumerable<User> users)
{
    return users.Where(u => u.IsActive);
}
```

➡️ Test edilebilir  
➡️ Clean Architecture uyumlu

---

## 9️⃣ En Sık Yapılan Hatalar ❌

- Her yerde `ToList()` çağırmak
- IQueryable → IEnumerable erken düşürmek
- LINQ içinde heavy logic yazmak

---
# LINQ Masterclass – Chapter 2: Where, Select, SelectMany

Bu chapter LINQ’in **en çok kullanılan 3 operatörünü**
net farkları ve gerçek örneklerle anlatır.

---

## 1️⃣ Where – Filtreleme

`Where`, koleksiyonu **filtreler**.

```csharp
var adults = users.Where(u => u.Age >= 18);
```

➡️ Kaynak değişmez  
➡️ Yeni bir sorgu oluşur

---

## 2️⃣ Where – Birden Fazla Koşul

```csharp
var result = users.Where(u => u.IsActive && u.Age > 30);
```

❗ Karmaşık logic yazma, gerekirse method’a böl.

---

## 3️⃣ Select – Projection (Şekil Değiştirme)

`Select`, verinin **şeklini değiştirir**.

```csharp
var names = users.Select(u => u.Name);
```

DTO’ya çevirme:
```csharp
var dtos = users.Select(u => new UserDto
{
    Id = u.Id,
    Name = u.Name
});
```

---

## 4️⃣ Where + Select Birlikte

```csharp
var result = users
    .Where(u => u.IsActive)
    .Select(u => u.Name);
```

➡️ En yaygın kullanım

---

## 5️⃣ SelectMany – Flatten (En Zor Anlaşılan)

`SelectMany`, **iç içe koleksiyonları düzleştirir**.

### Veri Yapısı
```csharp
class Order
{
    public List<OrderItem> Items { get; set; }
}
```

### Select (YANLIŞ beklenti)
```csharp
var items = orders.Select(o => o.Items);
// List<List<OrderItem>>
```

### SelectMany (DOĞRU)
```csharp
var items = orders.SelectMany(o => o.Items);
// List<OrderItem>
```

---

## 6️⃣ Select vs SelectMany Karşılaştırma

| Operatör | Çıktı |
|---|---|
| Select | Koleksiyonun koleksiyonu |
| SelectMany | Tek koleksiyon |

---

## 7️⃣ Gerçek Hayat Örneği

```csharp
var productNames =
    orders.SelectMany(o => o.Items)
          .Where(i => i.Price > 100)
          .Select(i => i.ProductName);
```

---

## 8️⃣ EF Core ile Kullanım

```csharp
var users = context.Users
    .Where(u => u.IsActive)
    .Select(u => new { u.Id, u.Name });
```

⚠️ `SelectMany` EF’de JOIN’e dönüşebilir.

---

## 9️⃣ En Sık Yapılan Hatalar ❌

- Select yerine SelectMany kullanmamak
- SelectMany’yi gereksiz kullanmak
- Where sonrası ToList çağırıp zinciri kırmak

---

# LINQ Masterclass – Chapter 3: First, Single, Any, All

Bu chapter, LINQ’te **en sık yanlış kullanılan** operatörleri
net farkları ve gerçek senaryolarla anlatır.

---

## 1️⃣ First – İlk Elemanı Al

`First`, koşula uyan **ilk elemanı** döner.

```csharp
var user = users.First(u => u.IsActive);
```

❗ Hiç eleman yoksa → **Exception fırlatır**

---

## 2️⃣ FirstOrDefault – Güvenli First

```csharp
var user = users.FirstOrDefault(u => u.IsActive);
```

- Eleman yoksa → `null` (veya default)
- En çok kullanılan versiyon

---

## 3️⃣ Single – Tek Eleman Olmalı

`Single`, **tam olarak 1 eleman** bekler.

```csharp
var user = users.Single(u => u.Email == email);
```

❌ 0 eleman → Exception  
❌ 2+ eleman → Exception  

➡️ Gerçekten **unique** olduğundan eminsen kullan

---

## 4️⃣ SingleOrDefault

```csharp
var user = users.SingleOrDefault(u => u.Email == email);
```

- 0 eleman → null
- 2+ eleman → Exception

---

## 5️⃣ Any – Var mı?

`Any`, **en performanslı kontrol** yöntemidir.

```csharp
if (users.Any(u => u.IsActive))
{
}
```

❗ `Count() > 0` YAPMA ❌

---

## 6️⃣ All – Hepsi Sağlıyor mu?

```csharp
var allActive = users.All(u => u.IsActive);
```

- Tüm elemanlar koşulu sağlıyorsa → `true`
- En az biri sağlamıyorsa → `false`

---

## 7️⃣ Karşılaştırma Tablosu

| Operatör | 0 Eleman | 1 Eleman | 2+ Eleman |
|---|---|---|---|
| First | ❌ Exception | ✅ Döner | ✅ İlk |
| FirstOrDefault | null | ✅ Döner | ✅ İlk |
| Single | ❌ Exception | ✅ Döner | ❌ Exception |
| SingleOrDefault | null | ✅ Döner | ❌ Exception |

---

## 8️⃣ EF Core ile Kullanım

```csharp
var exists = context.Users.Any(u => u.Email == email);
```

➡️ SQL’de `EXISTS` olur (çok hızlı)

```csharp
var user = context.Users.Single(u => u.Id == id);
```

➡️ Unique key için ideal

---

## 9️⃣ En Sık Yapılan Hatalar ❌

- `Any()` yerine `Count()`
- `Single()`’ı garanti yokken kullanmak
- `First()` ile null kontrolü yapmamak

---

# LINQ Masterclass – Chapter 4: GroupBy & Join

Bu chapter, LINQ’in **en güçlü ama en zor** konularından ikisini
gerçek hayat örnekleriyle açıklar:
- GroupBy
- Join

---

## 1️⃣ GroupBy Nedir?

`GroupBy`, veriyi **anahtara göre gruplar**.

SQL karşılığı:
```sql
GROUP BY
```

LINQ:
```csharp
var groups = users.GroupBy(u => u.Role);
```

---

## 2️⃣ GroupBy Temel Kullanım

```csharp
var result = users
    .GroupBy(u => u.Role)
    .Select(g => new
    {
        Role = g.Key,
        Count = g.Count()
    });
```

➡️ Rol başına kullanıcı sayısı

---

## 3️⃣ GroupBy + Aggregate

```csharp
var stats = orders
    .GroupBy(o => o.CustomerId)
    .Select(g => new
    {
        CustomerId = g.Key,
        TotalAmount = g.Sum(x => x.Total),
        OrderCount = g.Count()
    });
```

---

## 4️⃣ GroupBy + Where (HAVING)

```csharp
var bigCustomers = orders
    .GroupBy(o => o.CustomerId)
    .Where(g => g.Sum(x => x.Total) > 1000)
    .Select(g => g.Key);
```

➡️ SQL `HAVING` karşılığı

---

## 5️⃣ Join Nedir?

`Join`, iki koleksiyonu **ortak anahtar üzerinden** birleştirir.

```csharp
var result =
    from u in users
    join o in orders on u.Id equals o.UserId
    select new { u.Name, o.Total };
```

---

## 6️⃣ Join – Method Syntax

```csharp
var result = users.Join(
    orders,
    u => u.Id,
    o => o.UserId,
    (u, o) => new
    {
        u.Name,
        o.Total
    });
```

---

## 7️⃣ Inner Join vs Left Join

LINQ’de **Left Join yoktur**  
➡️ `GroupJoin` + `SelectMany` ile yapılır.

---

## 8️⃣ Left Join (GroupJoin)

```csharp
var result =
    from u in users
    join o in orders on u.Id equals o.UserId into gj
    from sub in gj.DefaultIfEmpty()
    select new
    {
        u.Name,
        Total = sub?.Total
    };
```

---

## 9️⃣ EF Core ile GroupBy & Join

```csharp
var result = context.Orders
    .GroupBy(o => o.UserId)
    .Select(g => new
    {
        UserId = g.Key,
        Total = g.Sum(x => x.Total)
    });
```

➡️ SQL’e çevrilir

⚠️ EF’de **GroupBy sonrası client-side düşme** riskine dikkat

---

## 🔟 En Sık Yapılan Hatalar ❌

- GroupBy sonrası ToList() erken çağırmak
- Join yerine navigation property varken Join yazmak
- EF’de desteklenmeyen GroupBy projection

---

# LINQ Masterclass – Chapter 5: LINQ + EF Core (SQL’e Dönüşen LINQ)

Bu chapter, LINQ’in **Entity Framework Core** ile nasıl çalıştığını,
hangi LINQ ifadelerinin **SQL’e çevrildiğini** ve
production’da en sık yapılan hataları anlatır.

---

## 1️⃣ LINQ to Objects vs LINQ to Entities

| Tür | Nerede Çalışır |
|---|---|
| LINQ to Objects | Memory (List, Array) |
| LINQ to Entities | Database (SQL) |

```csharp
context.Users.Where(u => u.IsActive);
```

➡️ SQL’e çevrilir

```csharp
usersList.Where(u => u.IsActive);
```

➡️ Memory’de çalışır

---

## 2️⃣ IQueryable Hayati Öneme Sahip

```csharp
IQueryable<User> query = context.Users;
```

❗ `ToList()` çağrılana kadar SQL çalışmaz.

```csharp
query = query.Where(u => u.Age > 18);
query = query.Where(u => u.IsActive);
```

➡️ Tek SQL olur

---

## 3️⃣ SQL’e Çevrilen LINQ Örnekleri

### Where
```csharp
context.Users.Where(u => u.Email == email);
```

### Select
```csharp
context.Users.Select(u => new { u.Id, u.Name });
```

### Any (EXISTS)
```csharp
context.Users.Any(u => u.Email == email);
```

### Count
```csharp
context.Users.Count();
```

---

## 4️⃣ SQL’e ÇEVRİLMEYEN LINQ ❌

```csharp
context.Users.Where(u => CustomMethod(u));
```

📛 Runtime exception:
> Could not be translated to SQL

---

## 5️⃣ Client-Side Evaluation Tuzağı

❌ Yanlış:
```csharp
context.Users
    .ToList()
    .Where(u => u.IsActive);
```

✅ Doğru:
```csharp
context.Users
    .Where(u => u.IsActive)
    .ToList();
```

---

## 6️⃣ Projection Performans Kazandırır

❌ Yanlış:
```csharp
context.Users.ToList();
```

✅ Doğru:
```csharp
context.Users
    .Select(u => new { u.Id, u.Name })
    .ToList();
```

➡️ Daha az kolon, daha hızlı sorgu

---

## 7️⃣ Navigation Property vs Join

❌ Gereksiz Join:
```csharp
from u in context.Users
join o in context.Orders on u.Id equals o.UserId
select new { u, o };
```

✅ Doğru:
```csharp
context.Users
    .Select(u => new
    {
        u.Name,
        u.Orders.Count
    });
```

---

## 8️⃣ Include Ne Zaman Kullanılır?

```csharp
context.Users.Include(u => u.Orders);
```

⚠️ Include:
- Sadece gerektiğinde
- Aksi halde ağır sorgu

---

## 9️⃣ Generated SQL’i Görmek

```csharp
var sql = context.Users
    .Where(u => u.IsActive)
    .ToQueryString();
```

➡️ Debug için altın değerinde

---

## 🔟 En Sık Production Hataları ❌

- Her yerde ToList()
- Client-side filtering
- Gereksiz Include
- IQueryable erken IEnumerable’a düşürmek

---


# LINQ Masterclass – Chapter 6: Performans Tuzakları & Gerçek Bug’lar

Bu chapter, LINQ kullanan herkesin **en az bir kez production’da yaşadığı**
performans problemlerini ve gerçek bug’ları anlatır.

Amaç:
- “Neden yavaş?” sorusunun cevabı
- LINQ yüzünden çıkan gizli bug’ları tanımak
- Doğru kullanım refleksi kazanmak

---

## 1️⃣ En Büyük Tuzak: Erken ToList()

❌ Yanlış:
```csharp
var users = context.Users.ToList();
var active = users.Where(u => u.IsActive);
```

✅ Doğru:
```csharp
var active = context.Users
    .Where(u => u.IsActive)
    .ToList();
```

📛 Etki:
- Tüm tablo memory’ye çekilir
- Büyük tabloda **felaket**

---

## 2️⃣ Count() > 0 vs Any()

❌ Yanlış:
```csharp
if (context.Users.Count() > 0)
```

✅ Doğru:
```csharp
if (context.Users.Any())
```

➡️ `Any()` SQL’de `EXISTS` olur (çok hızlı)

---

## 3️⃣ Client-Side Evaluation Bug’ı

```csharp
context.Users
    .Where(u => CustomCheck(u)) // SQL’e çevrilemez
    .ToList();
```

📛 Sonuç:
- Runtime exception
- Ya da tüm tablo çekilip memory’de filtre

---

## 4️⃣ N+1 Problemi (En Sinsi Bug)

```csharp
foreach (var user in context.Users)
{
    var orders = user.Orders.Count();
}
```

📛 Sonuç:
- 1 ana sorgu
- N tane ekstra sorgu

✅ Çözüm:
```csharp
context.Users
    .Select(u => new
    {
        u.Name,
        OrderCount = u.Orders.Count
    });
```

---

## 5️⃣ Gereksiz Include Kullanımı

❌ Yanlış:
```csharp
context.Users
    .Include(u => u.Orders)
    .Include(u => u.Roles)
    .ToList();
```

📛 Ağır SQL + gereksiz data

✅ Doğru:
```csharp
context.Users
    .Select(u => new { u.Id, u.Name });
```

---

## 6️⃣ Select Yerine Entity Dönmek

❌ Yanlış:
```csharp
context.Users.ToList();
```

✅ Doğru:
```csharp
context.Users
    .Select(u => new UserDto { u.Id, u.Name })
    .ToList();
```

---

## 7️⃣ Single Yanlış Kullanımı

❌ Yanlış:
```csharp
var user = context.Users.Single(u => u.IsActive);
```

📛 2 kayıt varsa → crash

✅ Doğru:
```csharp
var user = context.Users.FirstOrDefault(u => u.IsActive);
```

---

## 8️⃣ IQueryable Zincirini Kırmak

```csharp
IQueryable<User> q = context.Users;
var list = q.ToList();
q = q.Where(u => u.IsActive); // artık işe yaramaz
```

📛 Filtre DB’de değil memory’de

---

## 9️⃣ Generated SQL’i Kontrol Etmemek

```csharp
var sql = query.ToQueryString();
```

➡️ SQL’i görmeden “LINQ yavaş” deme

---

## 🔟 Gerçek Production Hikayeleri

- 10k kayıt → 10 milyon gibi davranıyor
- Sadece ToList() yüzünden CPU spike
- N+1 yüzünden DB connection pool dolu

---

## 1️⃣1️⃣ Altın Performans Kuralları

- ToList en sonda
- Filtre DB’de
- Projection kullan
- Any > Count
- Include minimum
- SQL’i her zaman düşün

---

## 1️⃣2️⃣ LINQ Performans Checklist

✅ IQueryable mi?  
✅ ToList en sonda mı?  
✅ Select var mı?  
✅ Any kullanıldı mı?  
✅ Include gerçekten gerekli mi?  

---





