
# SQL Server'da Silinen Veriyi Loglama (Trigger ile)

Bir kayıt silindiğinde trigger kullanarak silinen verinin bilgilerini (örneğin, silinen kişi, silinme zamanı, IP adresi vs.) başka bir tabloya loglama işlemi için SQL Server’da şöyle bir örnek hazırlayabiliriz.

---

## 1. Log Tablosu Oluşturma

```sql
CREATE TABLE DeletedRecordsLog (
    Id INT IDENTITY PRIMARY KEY,
    DeletedTable NVARCHAR(100),
    DeletedRecordId INT,
    DeletedBy NVARCHAR(100),
    DeletedAt DATETIME DEFAULT GETDATE(),
    DeletedFromIP NVARCHAR(50)
);
```

---

## 2. Trigger Yazımı (INSTEAD OF DELETE veya AFTER DELETE)

Aşağıda **Users** tablosu için `AFTER DELETE` trigger örneği var.

```sql
CREATE TRIGGER trg_Users_Delete_Log
ON Users
AFTER DELETE
AS
BEGIN
    SET NOCOUNT ON;

    INSERT INTO DeletedRecordsLog (DeletedTable, DeletedRecordId, DeletedBy, DeletedAt, DeletedFromIP)
    SELECT
        'Users',                 -- Hangi tablodan silindiği
        d.Id,                    -- Silinen kaydın Id'si
        SUSER_SNAME(),           -- SQL Server login kullanıcı adı (sileni yapan)
        GETDATE(),               -- Silinme zamanı
        HOST_NAME()              -- Silme işlemin yapıldığı makine adı (IP için alternatif)
    FROM deleted d;
END
```

---

## 3. IP Adresi Almak

- SQL Server’da doğrudan IP alma fonksiyonu yoktur.  
- `HOST_NAME()` genellikle istemci bilgisayar adını verir.  
- IP adresini almak için uygulama tarafından trigger’a parametre ile göndermek gerekebilir.  
- Alternatif olarak, uygulamada silme işlemi yapılırken, IP ve kullanıcı bilgisi log tablosuna `INSERT` yapılabilir.

---

## 4. ASP.NET Core’dan IP ve Kullanıcı Bilgisini Trigger’a Aktarma

Trigger’a doğrudan IP gönderemezsin ama uygulamada silme işlemi sonrası log tablosuna ekleme yapabilirsin.

**Örnek:**

```csharp
string userIp = HttpContext.Connection.RemoteIpAddress?.ToString();
string username = User.Identity.Name;

// Silme işlemi sonrası log ekleme
var sql = "INSERT INTO DeletedRecordsLog (DeletedTable, DeletedRecordId, DeletedBy, DeletedAt, DeletedFromIP) VALUES (@table, @recordId, @user, GETDATE(), @ip)";

await dbContext.Database.ExecuteSqlRawAsync(sql,
    new SqlParameter("@table", "Users"),
    new SqlParameter("@recordId", deletedUserId),
    new SqlParameter("@user", username),
    new SqlParameter("@ip", userIp));
```

---

## Özet

| Adım                | Açıklama                                             |
|---------------------|------------------------------------------------------|
| Log tablosu oluştur | Silinen kayıt bilgileri için tablo                   |
| AFTER DELETE trigger| Silme sonrası loglama                                |
| Kullanıcı bilgisi   | `SUSER_SNAME()` ile alınabilir                      |
| IP adresi           | `HOST_NAME()` sınırlı, uygulamadan alınması daha doğru |
| ASP.NET Core        | IP ve kullanıcı bilgisi uygulamadan log tablosuna yazılabilir |
