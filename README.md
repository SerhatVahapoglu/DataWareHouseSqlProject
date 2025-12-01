📊 End-to-End SQL Data Warehouse (Bronze → Silver → Gold)

Bu proje, SQL Server + Docker kullanarak modern bir Data Warehouse (DW) mimarisini uçtan uca uygulamak için hazırlanmıştır.
Proje Bronze, Silver ve Gold katmanlarından oluşan kurumsal veri işleme yaklaşımını takip eder.

ÖNEMLİ NOT
Bu proje, YouTube’daki Data With Baraa kanalındaki “Build End-to-End Data Warehouse Using SQL Server” eğitim serisi temel alınarak geliştirilmiştir.
Orijinal kaynak:
https://www.youtube.com/watch?v=SSKVgrwhzus

Kodların büyük bölümü eğitim sahibine aittir. Bu repoda kendi çalışmalarım, eklemelerim ve özelleştirmelerim bulunmaktadır.

📁 Proje Yapısı
├── datasets/
│   ├── source_crm/
│   │   ├── cust_info.csv
│   │   ├── prd_info.csv
│   │   └── sales_details.csv
│   ├── source_erp/
│   │   ├── CUST_AZ12.csv
│   │   ├── LOC_A101.csv
│   │   └── PX_CAT_G1V2.csv
│
├── scripts/
│   ├── bronz/
│   │   └── bronzLayer.sql
│   ├── silver/
│   │   ├── silverLayer.sql
│   │   └── silverProcess.sql
│   ├── gold/
│   │   └── goldLayer.sql
│   ├── tests/
│   │   ├── quality_check_gold.sql
│   │   └── quality_checks_silver.sql
│   └── create_db.sql

🧱 1. MİMARİ: BRONZE → SILVER → GOLD
🥉 BRONZE LAYER (Raw Data – Cleaning Yok)

CSV dosyalarındaki ham veri yüklenir.

Docker container içinden BULK INSERT ile tabloya alınır.

Null, boşluk, yanlış tarih gibi hatalar burada düzeltilmez.

Tümü raw biçimde saklanır.

Script: scripts/bronz/bronzLayer.sql

Bronze’da yapılanlar

CRM müşteri bilgileri

CRM ürün bilgileri

CRM satış detayları

ERP müşteri

ERP country/location

ERP category/master data

🥈 2. SILVER LAYER (Temizleme + Dönüşüm)

En kritik katman.
Burada tüm kurumsal temizlik ve standardizasyon kuralları uygulanır.

Script: scripts/silver/silverLayer.sql

✨ Silver Katmanında Yapılan Bazı Transformations
CRM Customer Information

Aynı müşterinin birden fazla kaydı varsa ROW_NUMBER() ile son kayıt seçilir.

Gender → F → Female, M → Male, diğerleri n/a

Marital status normalize edilir.

String alanlar TRIM edilir.

CRM Product Information

prd_key parçalanır → Category ID + product key

Product line normalize edilir → M → Mountain, R → Road, E → Electric

Cost bozuksa 0 yapılır.

CRM Sales Details

Tarihler integer formatındaysa DATE’e dönüştürülür.

bozuk tarih → NULL

sales = quantity * price kontrolü yapılır (hatalılar düzeltilir)

ERP Location (A101)

cid içindeki - karakterleri temizlenir.

Ülke kodu normalize edilir:

DE → Germany

US/USA → United States

NULL veya boş → n/a

ERP Customer (AZ12)

NASxxx pattern’leri → prefix kaldırılır.

Tarihler gelecekteyse NULL yapılır.

Gender normalize edilir.

ERP Category Table

Temiz kopya olduğu için direkt aktarılır.

🥇 3. GOLD LAYER (Star Schema – Boyut & Olgu)

Script: scripts/gold/goldLayer.sql

Star-schema yapısı:

⭐ Dimensions
gold.dim_customers

CRM + ERP verileri birleştirilir

Cinsiyet birincil olarak CRM’den alınır, yoksa ERP fallback

Surrogate key → ROW_NUMBER()

gold.dim_products

Ürün bilgileri + ERP kategori bilgileri birleşir

📦 Fact Table
gold.fact_sales

Satış verileri fact formatına dönüştürülür

Dim tablolar ile JOIN edilir

Tarih, fiyat, miktar tümü temiz formatta

🧪 4. DATA QUALITY TESTLERİ

Scriptler:

quality_checks_silver.sql

quality_check_gold.sql

Bu testler şunları kontrol eder:

Null veya duplicate primary key

Tarih validasyonu

Ülke, kategori gibi alanların standardizasyonu

fact-sales’in dim tablolarıyla ilişki bütünlüğü

sales = quantity × price doğruluğu

Kurumsal ortamda bu testler CI/CD veri kalitesi pipeline’larında kullanılır.

🐳 5. DOCKER + BULK INSERT MİMARİSİ

CSV’ler container içindeki:

/var/opt/mssql/data/datasets/


path’ine kopyalanır.
Bulk insert tamamen Docker içinden okunacak şekilde yapılandırılmıştır.
