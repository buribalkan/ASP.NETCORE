
# JavaScript `addEventListener` Hatası ve Çözümü

JavaScript’te `document.getElementById("someId").addEventListener(...)` satırında hata genellikle şu durumda ortaya çıkar:

- O `id`'ye sahip HTML elementi **sayfada bulunmazsa**.

---

## Sorunlu Kod Örneği

```js
const closeBtn = document.getElementById('closePanel');
closeBtn.addEventListener('click', () => {
    panel.classList.add('hidden');
});
```

Burada `closePanel` id’sine sahip bir element arıyorsun, ancak HTML içinde böyle bir element yoksa:

- `getElementById('closePanel')` **null** döner.
- `null` üzerinde `addEventListener` çağrısı yapılmaya çalışılınca hata oluşur.

---

## Çözüm

1. **HTML'de `closePanel` id’sine sahip bir element olduğundan emin ol**  
   Örneğin:
   ```html
   <button id="closePanel">Kapat</button>
   ```

2. Eğer böyle bir element yoksa,  
   - Ya id’sini ekle,  
   - Ya da JavaScript kodunu şu şekilde güncelle:

   ```js
   const closeBtn = document.getElementById('closePanel');
   if (closeBtn) {
       closeBtn.addEventListener('click', () => {
           panel.classList.add('hidden');
       });
   }
   ```

---

## Özet

- Hata, **null olan bir elemente** `addEventListener` eklenmeye çalışıldığında ortaya çıkar.
- Bu yüzden, event listener eklemeden önce elementin varlığını kontrol etmek iyi bir pratiktir.
- Ya HTML içinde gerekli element ve id olmalı, ya da JavaScript kodu buna göre güvenli hale getirilmelidir.

---

Bu şekilde hem hata önlenir hem de kod daha sağlam olur.
