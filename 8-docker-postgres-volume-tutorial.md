2. Veritabanına Bağlanma ve Örnek Veri Ekleme
Konteyner içerisindeki PostgreSQL CLI (psql) aracına bağlanıyoruz:
docker exec -it film-db psql -U admin

psql içerisine girdikten sonra sırasıyla aşağıdaki SQL komutlarını çalıştırarak bir veritabanı, tablo ve örnek kayıtlar oluşturalım:

-- Yeni bir veritabanı oluşturma
CREATE DATABASE sinema;

-- Veritabanına geçiş yapma
\c sinema

-- Tablo oluşturma
CREATE TABLE filmler (id INT, ad VARCHAR(100));

-- Veri ekleme
INSERT INTO filmler VALUES (1, 'Inception'), (2, 'The Matrix'), (3, 'Interstellar');

-- Eklenen verileri kontrol etme
SELECT * FROM filmler;

-- Çıkış yapma
\q

3. Konteyneri Silme ve Yeniden Başlatma
Mevcut konteyneri durdurup sistemden tamamen silelim:

docker container stop film-db
docker container rm film-db

Şimdi aynı komutla konteyneri sıfırdan tekrar başlatalım:
docker run --name film-db -e POSTGRES_USER=admin -e POSTGRES_PASSWORD=Istanbul34 -d postgres:15


4. Veri Kaybını Kontrol Etme
Tekrar psql ile içeri girip veritabanlarını listeleyelim:
docker exec -it film-db psql -U admin
\l
⚠️ Durum Analizi: Listede sinema adındaki veritabanımızın olmadığını göreceksiniz. Konteyner silindiğinde, izole edilmiş katmandaki tüm veriler de onunla birlikte yok olmuştur.

\q

💾 Bölüm 2: Docker Volume Kullanarak Veriyi Kalıcı Hale Getirme
Veri kaybını önlemek için Docker tarafından yönetilen ve host (ana makine) üzerinde saklanan bir depolama alanı (Volume) oluşturup bunu konteynerimize bağlayacağız.

1. Eski Konteyneri Temizleme
Öncelikle bir önceki adımdan kalan geçici konteyneri kaldıralım:
docker container stop film-db
docker container rm film-db

3. Docker Volume Oluşturma ve İnceleme
Verilerin kalıcı olarak saklanacağı pg_film_data adında bir volume oluşturuyoruz:
docker volume create pg_film_data
Oluşturulan alanın detaylarına ve fiziksel olarak host makinede nereye denk geldiğine (Mountpoint) bakmak için:
docker volume inspect pg_film_data

4. Konteyneri Volume ile Birlikte Başlatma
Bu kez -v parametresini kullanarak oluşturduğumuz volume alanını PostgreSQL'in veri depoladığı dizine (/var/lib/postgresql/data) bağlıyoruz:
docker run \
  --name film-db \
  -e POSTGRES_USER=admin \
  -e POSTGRES_PASSWORD=Istanbul34 \
  -e PGDATA=/var/lib/postgresql/data/pgdata \
  -v pg_film_data:/var/lib/postgresql/data \
  -d postgres:15
   4. Verileri Tekrar Oluşturma
Konteynere bağlanıp az önceki veritabanı ve kayıt işlemlerini tekrarlayalım:
docker exec -it film-db psql -U admin
CREATE DATABASE sinema;
\c sinema
CREATE TABLE filmler (id INT, ad VARCHAR(100));
INSERT INTO filmler VALUES (1, 'Inception'), (2, 'The Matrix'), (3, 'Interstellar');

\q
5. Konteyneri Yok Etme ve Kalıcılık Testi
Şimdi en kritik testi yapıyoruz. Konteyneri tamamen siliyoruz ancak arkasında oluşturduğumuz pg_film_data ismindeki volume yapısını bırakıyoruz.
docker container stop film-db
docker container rm film-db
Aynı Volume'u bağlayarak yeni bir PostgreSQL konteyneri ayağa kaldırıyoruz:
docker run \
  --name film-db \
  -e POSTGRES_USER=admin \
  -e POSTGRES_PASSWORD=Istanbul34 \
  -e PGDATA=/var/lib/postgresql/data/pgdata \
  -v pg_film_data:/var/lib/postgresql/data \
  -d postgres:15

  6. Sonuç ve Doğrulama
Tekrar veritabanına bağlanıp verilerimizin yerinde durup durmadığını kontrol edelim:
docker exec -it film-db psql -U admin
\c sinema
SELECT * FROM filmler;

🎉 Başarılı! Konteyner tamamen silinip yeniden üretilmesine rağmen, Docker Volume host makinede saklandığı için veritabanımız ve filmler tablomuzdaki tüm kayıtlar güvenli bir şekilde geri geldi.
\q


