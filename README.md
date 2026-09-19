# Filigran Temizleyici

Fotoğraflardan filigran (watermark) bölgelerini tarayıcı içinde, tamamen istemci tarafında (client-side) temizlemeye çalışan basit bir araç. Yapay zeka modeli **kullanmaz** — [OpenCV.js](https://github.com/TechStark/opencv-js) ile klasik görüntü işleme teknikleri (kenar/doku analizi ve inpainting) kullanır.

## Nasıl çalışır

1. **Fotoğraf yükle** — sürükle-bırak veya dosya seçici ile.
2. **Otomatik algıla** — kontrast farkı, kenar yoğunluğu ve bağlı bileşen analiziyle filigran *olabilecek* bölgeleri kırmızı maske olarak işaretler.
3. **Fırça ile düzelt** — otomatik öneri eksik ya da fazla işaretlerse "Ekle"/"Sil" fırçalarıyla maskeyi elle düzenle.
4. **Filigranı Kaldır** — işaretli bölge, OpenCV'nin `inpaint()` fonksiyonu (Telea algoritması) ile çevresindeki piksellerden matematiksel bir tahminle doldurulur.
5. **İndir** — sonucu PNG olarak kaydet.

Tüm işlem tarayıcıda yapılır; hiçbir görsel bir sunucuya yüklenmez.

## Dürüst sınırlamalar

Bu bölümü okumadan kullanmayın — araç ne yapıp ne yapamadığı konusunda net olmalı:

- **Otomatik algılama bir tahmindir, garanti değildir.** Saydamlık ve doku farklarına bakan bir sezgisel (heuristic) yöntemdir. Bazı filigranları kaçırabilir, bazı filigran olmayan alanları yanlışlıkla işaretleyebilir. Bu yüzden manuel fırça düzeltmesi var.
- **İnpainting, orijinal görüntüyü geri getirmez.** Filigranın altındaki gerçek piksel bilgisi kayıptır. Algoritma sadece çevredeki piksellerden "görsel olarak makul" bir yama üretir. Düz/basit arka planlarda (gökyüzü, tek renk zemin) sonuç genelde iyi olur; karmaşık dokulu alanlarda (yüzler, detaylı desenler) bulanıklık veya bozulma görülebilir.
- **Gerçek AI tabanlı inpainting (örn. diffusion modelleri) çok daha iyi sonuç verir** ama bu araç bilinçli olarak yalnızca klasik matematiksel yöntemler kullanıyor.
- Telif hakkıyla korunan görsellerden filigran kaldırmanın kullanım yerine göre hukuki sonuçları olabilir; bu konuda sorumluluk kullanıcıya aittir.

## Teknik detaylar

| Bileşen | Kullanılan yöntem |
|---|---|
| Görüntü işleme kütüphanesi | [OpenCV.js](https://github.com/TechStark/opencv-js) (WebAssembly, tarayıcıda çalışır) |
| Otomatik algılama | Gaussian blur ile yerel kontrast farkı + Canny kenar tespiti + morfolojik temizleme + bağlı bileşen (connected components) filtreleme |
| Kaldırma (inpainting) | `cv.inpaint()` — Telea algoritması (`cv.INPAINT_TELEA`) |
| Maske düzenleme | Canvas tabanlı fırça (ekle/sil), geri al (undo) desteği |

## Dosya yapısı

Tek dosyalık bir uygulama:

```
watermark-remover.html   — HTML + CSS + JS, tamamı tek dosyada
```

Harici bağımlılık yalnızca OpenCV.js'tir; CDN üzerinden (`cdn.jsdelivr.net/npm/@techstark/opencv-js`) yüklenir, kurulum gerekmez.

## Kullanım

Dosyayı bir tarayıcıda açman yeterli. Bazı tarayıcılar `file://` üzerinden açılan sayfalarda CDN script'lerini kısıtlayabilir; sorun yaşarsan basit bir yerel sunucu ile açmak daha güvenilir olur:

```bash
python3 -m http.server 8000
# sonra tarayıcıda: http://localhost:8000/watermark-remover.html
```

Veya doğrudan web sitemiz üzerinden kullanabilirsiniz

## Bilinen sorunlar

- **`ERR_BLOCKED_BY_ORB` / OpenCV.js yüklenmiyor:** Genelde bir tarayıcı eklentisi (reklam/gizlilik engelleyici) ya da `file://` üzerinden açmaktan kaynaklanır. Eklentileri geçici kapatmayı veya yukarıdaki yerel sunucu yöntemini dene.
- Çok yüksek çözünürlüklü görsellerde performans için görüntü otomatik olarak ~1400px'e küçültülür (işlem hızı için); indirilen sonuç bu çözünürlükte olur.
