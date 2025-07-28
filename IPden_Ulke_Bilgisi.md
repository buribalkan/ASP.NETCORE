
# ASP.NET Core'da IP'den Ülke Bilgisi Bulma

IP adresinin hangi ülkeye (lokasyona) ait olduğunu belirlemek için genellikle IP Geolocation (IP konum belirleme) servisleri veya kütüphaneleri kullanılır. Aşağıda ASP.NET Core projelerinde IP’den ülke bilgisi bulmak için yaygın yöntemleri ve araçları özetledim:

---

## 1. Ücretsiz ve Ücretli IP Geolocation Servisleri

**Popüler servisler:**

| Servis Adı        | Açıklama               | Ücretsiz Kota          | Kolay Entegrasyon     |
|-------------------|------------------------|-------------------------|------------------------|
| **IPStack**       | JSON API, kolay kullanılır | Ücretsiz plan var (limitli) | REST API               |
| **IPinfo**        | Zengin veri            | Ücretsiz plan var       | REST API               |
| **MaxMind GeoIP2**| En çok tercih edilen DB | Ücretsiz Lite DB mevcut | Lokal DB veya API      |
| **IP2Location**   | API ve offline DB      | Ücretsiz sınırlı        | API veya SDK           |

---

## 2. ASP.NET Core’da Kullanım Yöntemleri

### A) REST API Çağrısı ile

**Örnek: IPStack API ile**

```csharp
public async Task<string> GetCountryFromIp(string ip)
{
    var httpClient = new HttpClient();
    string apiKey = "YOUR_API_KEY";
    string url = $"http://api.ipstack.com/{ip}?access_key={apiKey}";

    var response = await httpClient.GetStringAsync(url);
    var json = JsonDocument.Parse(response);
    var country = json.RootElement.GetProperty("country_name").GetString();

    return country ?? "Unknown";
}
```

---

### B) MaxMind GeoIP2 Lokal Kullanımı

MaxMind GeoIP2 ücretsiz veritabanını indirip projene dahil edebilirsin.  
**NuGet paketi:**

```bash
dotnet add package MaxMind.GeoIP2
```

**Kullanımı:**

```csharp
using MaxMind.GeoIP2;

public string GetCountryByIp(string ip)
{
    using var reader = new DatabaseReader("GeoLite2-Country.mmdb");
    var country = reader.Country(ip);
    return country.Country.Name;
}
```

---

## 3. IP Adresini Alma (ASP.NET Core)

```csharp
string ipAddress = HttpContext.Connection.RemoteIpAddress?.ToString();
```

---

## 4. Dikkat Edilmesi Gerekenler

- Kullanıcı VPN, proxy veya NAT arkasındaysa gerçek IP alınamayabilir.  
- IP veritabanları ve servisler düzenli güncellenmeli.  
- Ücretsiz planlar sınırlamalar getirebilir.

---

## 5. Özet

| Yöntem             | Avantaj               | Dezavantaj                     |
|--------------------|-----------------------|---------------------------------|
| **REST API kullanımı** | Kolay, güncel veri     | Ücretli/limitli, ağ gecikmesi  |
| **Lokal DB (MaxMind)** | İnternet bağımsız, hızlı | Veritabanı güncelleme gerektir |
