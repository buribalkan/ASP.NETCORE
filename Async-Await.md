# Async / Await Masterclass – Chapter 1: Temeller & Zihniyet

Bu chapter, async/await konusunu **ezberden değil zihniyetten**
anlatır. Amaç: deadlock, yavaşlık ve yanlış async kullanımını
daha baştan önlemek.

---

## 1️⃣ Async / Await Nedir?

`async/await`, **non-blocking** (thread kilitlemeyen)
asenkron programlama modelidir.

Yanlış algı ❌:
> Async = daha hızlı

Doğru ✅:
> Async = thread’i boşa çıkarır, ölçeklenebilirlik sağlar

---

## 2️⃣ Senkron vs Asenkron

❌ Senkron:
```csharp
var data = GetData(); // thread burada bekler
```

✅ Asenkron:
```csharp
var data = await GetDataAsync(); // thread serbest
```

---

## 3️⃣ async Anahtar Kelimesi Ne Yapar?

`async`:
- Metodun **await** içerebileceğini söyler
- Tek başına async, asenkronluk sağlamaz

```csharp
async Task Foo()
{
    DoSomething(); // hala senkron
}
```

---

## 4️⃣ await Ne Yapar?

`await`:
- Metodu **parçalar**
- I/O beklerken thread’i serbest bırakır
- Devamını **continuation** olarak planlar

```csharp
await Task.Delay(1000);
```

---

## 5️⃣ Task Nedir?

`Task`:
- Devam eden veya bitecek bir işi temsil eder
- Thread değildir ❗

```csharp
Task task = SomeAsync();
```

---

## 6️⃣ Async Metod İmza Kuralları

| İmza | Ne Zaman |
|---|---|
| `Task` | Void yerine |
| `Task<T>` | Değer dönüyorsa |
| `async void` | ❌ Sadece event handler |

---

## 7️⃣ async void Neden KÖTÜ?

```csharp
async void DoWork()
{
    throw new Exception();
}
```

📛 Exception yakalanamaz  
📛 App crash edebilir

---

## 8️⃣ Async Zinciri Kuralı (ÇOK ÖNEMLİ)

> **Async aşağı doğru akar**

```csharp
await ServiceAsync();
await RepoAsync();
```

❌ Zinciri senkrona çevirme:
```csharp
ServiceAsync().Result;
```

---

## 9️⃣ İlk Büyük Tuzak: .Result / .Wait()

```csharp
var result = task.Result; // ❌
```

📛 Deadlock riski  
📛 Thread pool tıkanır

---

# Async / Await Masterclass – Chapter 2: Deadlock, SynchronizationContext & .Result Tuzağı

Bu chapter, async/await dünyasındaki **en tehlikeli ama en öğretici**
konuyu anlatır: **deadlock**.
Bunu anlayan biri async’i gerçekten anlamıştır.

---

## 1️⃣ Deadlock Nedir?

Deadlock:
- Bir thread bir işi bekler
- Beklenen iş de aynı thread’i bekler
➡️ Sonsuz bekleme

Async dünyasında bu genelde **UI / ASP.NET context** kaynaklıdır.

---

## 2️⃣ SynchronizationContext Nedir?

`SynchronizationContext`:
- `await` sonrası **devamın nerede çalışacağını** belirler

Örnek:
- WPF / WinForms → UI thread
- ASP.NET (classic) → request thread
- ASP.NET Core → ❌ yok (önemli fark!)

---

## 3️⃣ Deadlock Nasıl Oluşur?

```csharp
public string GetData()
{
    var result = GetDataAsync().Result; // ❌
    return result;
}

public async Task<string> GetDataAsync()
{
    await Task.Delay(1000);
    return "data";
}
```

📛 Akış:
1. Ana thread `.Result` ile bekler
2. `await` sonrası continuation **aynı thread’i ister**
3. Thread kilitli → deadlock

---

## 4️⃣ ASP.NET Core Neden Daha Güvenli?

ASP.NET Core:
- `SynchronizationContext` kullanmaz
- Deadlock riski **daha düşüktür**

AMA ❗:
- `.Result` hâlâ **thread starvation** yapar

---

## 5️⃣ .Result / .Wait() Neden KÖTÜ?

- Thread bloklar
- ThreadPool dolar
- Throughput düşer
- Scaling bozulur

```csharp
task.Wait();   // ❌
task.Result;  // ❌
```

---

## 6️⃣ Doğru Çözüm: Baştan Sona Async

❌ Yanlış:
```csharp
public string Get()
{
    return GetAsync().Result;
}
```

✅ Doğru:
```csharp
public async Task<string> Get()
{
    return await GetAsync();
}
```

---

## 7️⃣ ConfigureAwait(false) Nedir?

```csharp
await Task.Delay(1000).ConfigureAwait(false);
```

Anlamı:
- Devam eden kod **orijinal context’e dönmesin**

➡️ Library code için ALTIN KURAL

---

## 8️⃣ ConfigureAwait Ne Zaman Kullanılır?

| Yer | Kullan |
|---|---|
| Class library | ✅ EVET |
| UI code | ❌ HAYIR |
| ASP.NET Core | Genelde gerekmez |

---

## 9️⃣ Gerçek Production Bug Örneği

- UI app donuyor
- CPU düşük ama app cevap vermiyor
- Sebep: `.Result` + `await`

---


# Async / Await Masterclass – Chapter 3: Task vs ValueTask (Ne Zaman Hangisi?)

Bu chapter, **Task** ve **ValueTask** farkını doğru yerde doğru aracı
kullanma bakış açısıyla anlatır. Amaç: **gereksiz karmaşadan kaçınmak**.

---

## 1️⃣ Task Nedir?

`Task`:
- Heap üzerinde allocate edilir
- Await edilebilir
- En yaygın ve **varsayılan tercihtir**

```csharp
public Task<int> GetCountAsync()
{
    return Task.FromResult(5);
}
```

---

## 2️⃣ ValueTask Nedir?

`ValueTask`:
- Struct’tır (value type)
- Allocation’ı azaltmak için vardır
- **Özel senaryolar** içindir

```csharp
public ValueTask<int> GetCountAsync()
{
    return new ValueTask<int>(5);
}
```

⚠️ ValueTask **daha hızlı değildir**; sadece allocation’ı azaltır.

---

## 3️⃣ Ne Zaman ValueTask Gerekir?

Aşağıdaki **hepsi** doğruysa düşünebilirsin:

- Metot **çok sık çağrılıyor**
- Çoğu zaman **senkron tamamlanıyor**
- Allocation ölçümlerinde (profiling) Task maliyeti görünüyor
- Public API veya hot-path

Örnek:
```csharp
public ValueTask<bool> IsCachedAsync(string key)
{
    if (_cache.TryGetValue(key, out _))
        return new ValueTask<bool>(true);

    return new ValueTask<bool>(LoadFromDbAsync(key));
}
```

---

## 4️⃣ Ne Zaman Task Kullanmalısın? (Çoğu Zaman)

- IO-bound işler
- EF Core, HTTP, File IO
- Basitlik istiyorsan
- Allocation problemi yoksa

```csharp
public async Task<User?> GetUserAsync(int id)
{
    return await _context.Users.FindAsync(id);
}
```

➡️ **%95 senaryo**

---

## 5️⃣ ValueTask’ın Tehlikeleri ❌

- Birden fazla await edilemez
- Yanlış kullanım bug üretir
- Okunabilirliği düşürür

```csharp
ValueTask<int> vt = GetAsync();
await vt;
await vt; // ❌ HATA
```

---

## 6️⃣ Public API Kuralı

> **Public API’de Task tercih et**

ValueTask:
- Internal / private
- Çok iyi ölçülmüş hot-path

---

## 7️⃣ Performans Gerçeği

- Task allocation ≈ mikro saniyeler
- Yanlış ValueTask kullanımı ≫ daha büyük bug maliyeti

➡️ **Ölçmeden optimize etme**

---

## 8️⃣ Karar Tablosu

| Senaryo | Tercih |
|---|---|
| EF / HTTP | Task |
| Public API | Task |
| Cache hit ağırlıklı | ValueTask |
| Hot-path | ValueTask (ölçerek) |
| Emin değilsen | Task |

---

# Async / Await Masterclass – Chapter 4: Parallelism vs Concurrency

Bu chapter, async dünyasında **en çok karıştırılan iki kavramı**
net şekilde ayırır:
- Concurrency (eşzamanlılık)
- Parallelism (paralellik)

Ve doğru araçların **ne zaman kullanılacağını** gösterir.

---

## 1️⃣ Concurrency Nedir?

Concurrency:
- Birden fazla işin **aynı anda ilerliyor gibi görünmesi**
- Genelde **IO-bound** işler

```csharp
await GetUserAsync();
await GetOrdersAsync();
```

➡️ Thread bloklanmaz  
➡️ Ölçeklenebilirlik artar

---

## 2️⃣ Parallelism Nedir?

Parallelism:
- Birden fazla işin **gerçekten aynı anda çalışması**
- Genelde **CPU-bound** işler

```csharp
Parallel.ForEach(items, item =>
{
    Process(item);
});
```

➡️ CPU çekirdekleri kullanılır

---

## 3️⃣ En Büyük Yanlış ❌

> Async = Parallel

❌ Yanlış:
```csharp
await Task.Run(() => CpuHeavyWork());
```

📛 CPU-bound işi async yapmak çözüm değildir

---

## 4️⃣ Task.WhenAll (Concurrency için ALTIN ARAÇ)

```csharp
await Task.WhenAll(
    GetUserAsync(),
    GetOrdersAsync(),
    GetPaymentsAsync()
);
```

✅ IO-bound işler  
✅ Aynı anda başlar  
✅ Tek await noktası  

---

## 5️⃣ Task.WhenAll Hataları

❌ Yanlış:
```csharp
await Task.WhenAll(tasks).Result;
```

❌ Exception yutmak:
```csharp
try
{
    await Task.WhenAll(tasks);
}
catch { }
```

➡️ AggregateException içerir

---

## 6️⃣ Parallel.For / Parallel.ForEach

```csharp
Parallel.ForEach(numbers, n =>
{
    CpuHeavyCalculation(n);
});
```

Kullan:
- CPU-bound
- Kısa işler
- Stateless logic

KULLANMA:
- Async metotlarla
- IO-bound işlerde
- ASP.NET request içinde (çoğu zaman)

---

## 7️⃣ Async + Parallel = 🚫

❌ Yanlış:
```csharp
Parallel.ForEach(items, async item =>
{
    await SaveAsync(item);
});
```

📛 Fire-and-forget bug  
📛 Kontrolsüz thread

---

## 8️⃣ Doğru Karar Rehberi

```
İş IO-bound mu?
 └─ EVET → async + await + Task.WhenAll

İş CPU-bound mu?
 └─ EVET → Parallel / Task.Run (kontrollü)

ASP.NET request mi?
 └─ EVET → Parallel’den kaçın
```

---

## 9️⃣ Gerçek Production Bug

- Parallel.ForEach içinde HTTP çağrıları
- ThreadPool starvation
- App cevap vermez

---

# Async / Await Masterclass – Chapter 5: ThreadPool, Starvation & Scaling Problemleri

Bu chapter, async/await kullanan uygulamaların **neden yavaşladığını**
ve **yük altında neden çöktüğünü** açıklar.
Sebep çoğu zaman koddur, donanım değil.

---

## 1️⃣ ThreadPool Nedir?

ThreadPool:
- .NET’in yönettiği **paylaşımlı thread havuzu**
- Yeni thread yaratmak yerine tekrar kullanır
- IO ve CPU işleri için ortaktır

Ama:
> ThreadPool **sonsuz değildir**

---

## 2️⃣ ThreadPool Nasıl Çalışır?

- Başlangıçta az thread
- Yük arttıkça **yavaş yavaş** artırır
- Bloklanan thread’ler geri dönmezse → problem

---

## 3️⃣ Thread Starvation Nedir?

Starvation:
- ThreadPool’daki thread’ler **bloklanır**
- Yeni işler thread bulamaz
- Uygulama “donmuş” gibi olur

---

## 4️⃣ Starvation’a Neden Olan En Büyük Günahlar ❌

### ❌ .Result / .Wait()

```csharp
var data = GetAsync().Result;
```

➡️ Thread kilitlenir  
➡️ ThreadPool dolar  

---

### ❌ Uzun Süren CPU İşleri

```csharp
await Task.Run(() => CpuHeavyWork());
```

➡️ ASP.NET’te çok riskli

---

### ❌ Async Olmayan IO

```csharp
File.ReadAllText(path); // bloklar
```

---

## 5️⃣ Scaling Neden Bozulur?

Normalde:
- 100 request → 100 async IO → thread free

Yanlış kullanımda:
- 100 request → 100 thread blok → 101. request bekler

➡️ CPU boş ama app cevap vermez

---

## 6️⃣ Doğru IO Pattern’leri

❌ Yanlış:
```csharp
File.ReadAllText(path);
```

✅ Doğru:
```csharp
await File.ReadAllTextAsync(path);
```

---

## 7️⃣ ASP.NET Core’da En Kritik Kural

> **Request thread’ini ASLA bloklama**

- No `.Result`
- No `.Wait`
- No `Thread.Sleep`

---

## 8️⃣ ThreadPool Ayarı Yapmalı mıyım?

❌ Genelde HAYIR

```csharp
ThreadPool.SetMinThreads(...)
```

➡️ Sorunu maskeler  
➡️ Kök sebebi çözmez

---

## 9️⃣ Gerçek Production Senaryosu

- Trafik artıyor
- CPU %20
- Response time 30sn
- Sebep: `.Result` ile çağrılan HTTP client

---

## 🔟 Scaling İçin Altın Kurallar

- Baştan sona async
- IO-bound işlerde async API
- CPU-bound işi request’ten çıkar
- Background queue kullan

---

## 1️⃣1️⃣ Kısa Checklist

✅ async/await zinciri kırılmadı mı  
✅ IO async mi  
✅ CPU işi request’te mi  
✅ `.Result` var mı  

---

# Async / Await Masterclass – Chapter 6: Gerçek Production Async Bug’ları

Bu chapter, async/await yüzünden **production’da gerçekten yaşanan**
bug’ları inceler.
Amaç: kitap bilgisi değil, **refleks** kazandırmak.

---

## 1️⃣ Bug #1 – Gizli `.Result` (Masum Görünüyor)

```csharp
public User GetUser(int id)
{
    return _repo.GetUserAsync(id).Result; // ❌
}
```

📛 Etki:
- Düşük trafikte çalışır
- Trafik artınca **response time patlar**
- ThreadPool starvation

✅ Çözüm:
```csharp
public async Task<User> GetUser(int id)
{
    return await _repo.GetUserAsync(id);
}
```

---

## 2️⃣ Bug #2 – Fire-and-Forget Async

```csharp
public void Save()
{
    SaveAsync(); // ❌ await yok
}
```

📛 Etki:
- Exception kaybolur
- İş yarıda kalabilir
- Debug edilemez

✅ Çözüm:
```csharp
await SaveAsync();
```

veya **Background queue**

---

## 3️⃣ Bug #3 – Async void Faciası

```csharp
async void Process()
{
    await Task.Delay(1000);
    throw new Exception();
}
```

📛 Etki:
- App crash
- Exception yakalanamaz

✅ Çözüm:
```csharp
async Task Process()
```

---

## 4️⃣ Bug #4 – Parallel + Async Karışımı

```csharp
Parallel.ForEach(items, async item =>
{
    await SaveAsync(item);
});
```

📛 Etki:
- Kontrolsüz thread
- Kaybolan task’lar
- Veri tutarsızlığı

✅ Çözüm:
```csharp
await Task.WhenAll(items.Select(SaveAsync));
```

---

## 5️⃣ Bug #5 – BackgroundService İçinde Blocking

```csharp
while (true)
{
    Thread.Sleep(5000); // ❌
}
```

📛 Etki:
- Thread boşa harcanır
- Scaling bozulur

✅ Çözüm:
```csharp
await Task.Delay(5000, stoppingToken);
```

---

## 6️⃣ Bug #6 – Async + Lock

```csharp
lock (_sync)
{
    await SaveAsync(); // ❌ compile bile olmaz
}
```

Yan çözüm (yanlış):
```csharp
lock (_sync)
{
    SaveAsync().Wait(); // ❌
}
```

✅ Doğru:
```csharp
SemaphoreSlim _sem = new(1,1);

await _sem.WaitAsync();
try
{
    await SaveAsync();
}
finally
{
    _sem.Release();
}
```

---

## 7️⃣ Bug #7 – HTTP Client Yanlış Kullanım

```csharp
using var client = new HttpClient();
await client.GetAsync(url);
```

📛 Etki:
- Socket exhaustion
- Random timeout

✅ Çözüm:
```csharp
IHttpClientFactory
```

---

## 8️⃣ Bug #8 – Async Test Yazılmaması

```csharp
[Test]
public void Test()
{
    service.DoAsync(); // ❌ await yok
}
```

📛 Test yeşil ama iş çalışmadı

✅ Çözüm:
```csharp
[Test]
public async Task Test()
{
    await service.DoAsync();
}
```

---

## 9️⃣ Ortak Kök Sebep

> Async zincirini kırmak  
> Blocking yapmak  
> “Çalışıyor ya” demek

---

## 🔟 Production Async Checklist

✅ `.Result` / `.Wait` yok  
✅ async void yok  
✅ Fire-and-forget yok  
✅ Parallel + async yok  
✅ IO async  
✅ BackgroundService doğru  

---



