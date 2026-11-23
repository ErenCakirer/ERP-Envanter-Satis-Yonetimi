# Kurumsal ERP Envanter ve Satış Yönetim Sistemi (SQL Server & Power BI)

Bu proje, kurumsal kaynak planlama (ERP) süreçlerindeki **stok takibi, otomatik envanter düşümü ve satış/ciro raporlamasını** uçtan uca simüle eden bir veritabanı ve iş zekası (BI) çözümüdür.

## Proje Mimarisi ve Kullanılan Teknolojiler

* **Veritabanı Yönetimi:** Microsoft SQL Server (T-SQL)
* **İş Zekası ve Raporlama:** Microsoft Power BI Desktop (Import Mode & DAX)
* **Veri Modelleme:** İlişkisel Veritabanı Tasarımı

---

## Veritabanı Katmanı (SQL Server)

Sistem; `Kategoriler`, `Urunler`, `Musteriler`, `Siparisler` ve `SiparisDetaylari` tabloları üzerine ilişkisel (`Primary Key` / `Foreign Key`) olarak inşa edilmiştir:

* **Otomatik Stok Tetikleyicisi (`AFTER INSERT TRIGGER`):** `SiparisDetaylari` tablosuna yeni bir satış satırı eklendiğinde, ilgili ürünün `Urunler` tablosundaki stok miktarını otomatik olarak eksilten tetikleyici mekanizması geliştirildi.
* **İş Mantığı ve Veri Bütünlüğü (`Stored Procedure` & `TRANSACTION`):** Hatalı işlemlerde veri tutarsızlığını önlemek için `BEGIN TRAN / COMMIT / ROLLBACK` yapıları ve parametreli saklı yordamlar oluşturuldu.
* **Analitik Sorgular & Görünümler (`VIEW`, `CTE`, `Window Functions`):** Kategori bazlı stok değerleri ve dönemler arası değişim analizleri için `vw_KategoriOzetRaporu`, `WITH (CTE)`, `ROW_NUMBER()` ve `LAG()` fonksiyonları kullanıldı.

---

## Raporlama ve İş Zekası Katmanı (Power BI)

SQL Server veritabanına doğrudan bağlanılarak iki ana yönetim ekranı tasarlandı:

### 1. Envanter Özeti Paneli
* **KPI Kartları:** Toplam Envanter Değeri (`SUMX`) ve Kritik Stok Seviyesindeki Ürün Sayısı (`CALCULATE`).
* **Görseller:** Kategori bazlı stok adedi ve maliyet dağılımı (Bar & Donut Chart), kategori dilimleyici (`Slicer`) ve kritik stok eşiği altındaki ürünleri kırmızı renkle uyaran koşullu biçimlendirmeli yönetici detay tablosu.



### 2. Satış Analizi Paneli
* **KPI Kartları:** Toplam Ciro (`SUMX`), Toplam Sipariş Sayısı (`DISTINCTCOUNT`) ve Satılan Toplam Ürün Adedi (`SUM`).
* **Görseller:** Tarihsel ciro trendi (Line Chart), müşteri bazlı ciro sıralaması (Clustered Bar Chart) ve ödeme yöntemlerine (Vadeli, Havale, Kredi Kartı) göre ciro dağılımı (Donut Chart).



## Temel DAX Ölçüleri

```dax
Toplam Envanter Degeri = SUMX(Urunler, Urunler[Fiyat] * Urunler[Stok])

Kritik Stok Sayisi = CALCULATE(COUNTROWS(Urunler), Urunler[Stok] < 10)

Toplam Ciro = SUMX(SiparisDetaylari, SiparisDetaylari[Adet] * SiparisDetaylari[BirimSatisFiyati])

Toplam Siparis = DISTINCTCOUNT(SiparisDetaylari[SiparisID])
