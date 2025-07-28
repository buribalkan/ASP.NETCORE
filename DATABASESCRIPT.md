

SQL Server Management Studio (SSMS) kullanarak var olan bir database’in oluşturulma (CREATE DATABASE ve içindeki tüm objeler) script’ini almak için aşağıdaki adımları takip edebilirsin:



---



# Adım Adım Database Script Alma (Generate Scripts)



1. SSMS’i aç ve SQL Server’a bağlan



2. Sol taraftaki Object Explorer’da database’i bul  

   - Script almak istediğin database üzerine sağ tıkla.



3. Tasks > Generate Scripts seçeneğine tıkla



---



4. Script Oluşturma Sihirbazı Başlar  

   - Introduction ekranı varsa “Next” tıkla.



---



5. Object Seçimi  

   - “Select specific database objects” seçeneği ile tüm database objelerini seçebilirsin:  

     - Tables  

     - Views  

     - Stored Procedures  

     - Functions  

     - Triggers  

     - Users  

     - Roles  

   - Veya sadece “Database” seçerek tüm yapıyı alabilirsin (ama genelde tablo ve diğer objeleri seçmek daha yaygın).



---



6. Script Ayarları (Set Scripting Options)  

   - Çıktı dosyası olarak kaydedebilir ya da yeni sorgu penceresinde açabilirsin.  

   - Options (Ayarlar) bölümünde:  

     - Types of data to script:  

       - Schema only (Yapı)  

       - Data only (Veri)  

       - Schema and data (Yapı + Veri)  

     - Script DROP and CREATE:  

       - Script CREATE (sadece oluşturma)  

       - Script DROP and CREATE (sil ve oluştur)  

   - Diğer ayarları ihtiyaçlarına göre yapabilirsin.



---



7. Script Oluştur ve Sonuçları İncele  

   - İleri tuşları ile ilerle, script oluşturulsun.  

   - Script tamamlandığında çıktı dosyasını veya yeni sorgu penceresindeki scripti göreceksin.



---



# Özet



| Adım                    | Açıklama                           |
|-------------------------|----------------------------------|
| Database üzerine sağ tık | Tasks > Generate Scripts          |
| Obje seçimi             | Tüm objeler veya belirli objeler  |
| Script ayarları         | Schema only, Data only, vs.       |
| Script oluştur          | Dosya veya yeni query penceresi  |



---



Bu script’i kullanarak başka bir sunucuda aynı database yapısını oluşturabilirsin.



---
