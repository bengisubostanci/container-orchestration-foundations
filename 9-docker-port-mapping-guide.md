1.Bağlantı Reddi (Connection Refused) Doğrulaması
Servisler kapalıyken varsayılan porttan bağlanmayı denediğimizde işletim sisteminin bağlantıyı reddettiğinden emin oluyoruz:
[train@localhost prod_level]$ psql -h localhost -d train -U postgres
Çıktı:
psql: could not connect to server: Connection refused
        Is the server running on host "localhost" (127.0.0.1) and accepting
        TCP/IP connections on port 5432?

📦 Bölüm 2: İzole Port Eşleştirmesi ile Docker Kurulumu
Mevcut çakışmayı önlemek için, konteyner içindeki PostgreSQL yine 5432 portunda çalışırken; dış dünyaya (ana makineye) bu portu 5433 olarak izole bir şekilde map'leyeceğiz.

1. Eski Çakışan Konteynerlerin Temizlenmesi
   docker container stop postgresql
docker container rm postgresql

2. Gelişmiş Port Eşleme (-p) ile Konteyneri Başlatma
Buradaki -p 5433:5432 parametresi kritik role sahiptir: [Dış Port (Host)]:[İç Port (Container)]
docker run --name postgresql \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=Ankara06 \
  -e PGDATA=/var/lib/postgresql/data/pgdata \
  -v v_postgresql_10:/var/lib/postgresql/data \
  -p 5433:5432 \
  -d postgres:10

💡 Not: -v parametresi ile bağladığımız volume sayesinde konteyner silinse dahi veritabanı kayıtlarımız v_postgresql_10 biriminde güvende kalır.

3. Yönlendirilen Port Üzerinden Konteyner Bağlantısı
Konteynere dışarıdan erişirken artık açıkça -p 5433 portunu belirtiyoruz:

[train@localhost prod_level]$ psql -h localhost -p 5433 -d train -U postgres
Çıktı Başarılı:
Password for user postgres: Ankara06
psql (9.2.24, server 10.15 (Debian 10.15-1.pgdg90+1))
Type "help" for help.

train=# \q

🔄 Bölüm 3: Yerel ve Konteyner Servislerinin Eşzamanlı Çalıştırılması
Docker konteynerimiz artık 5433 hattını kullandığı için, işletim sisteminin kendi yerel PostgreSQL servisini (5432) çakışma riski olmadan güvenle geri başlatabiliriz.

1. Yerel Servisin Yeniden Başlatılması
sudo systemctl start postgresql-10

2. Yerel Veritabanına Erişim Testi
Varsayılan 5432 portunu işaret ederek yerel sistemdeki traindb veritabanına bağlanıyoruz:
[train@localhost prod_level]$ psql -h localhost -p 5432 -d traindb -U train
Çıktı Başarılı:
Password for user train: Ankara06
psql (9.2.24, server 10.13)
Type "help" for help.

traindb=# \q

📊 Özet Durum Matrisi
Şu anda aynı makine üzerinde birbirini engellemeden çalışan iki farklı PostgreSQL mimarisinin haritası:
Bağlantı Portu,Hedef Altyapı,Veritabanı (DB),Varsayılan Kullanıcı,Sunucu Sürümü,İşletim Sistemi Sistemi
5432,Yerel Servis (Local OS),traindb,train,10.13,Native Linux (Host)
5433,Docker Konteyneri,train,postgres,10.15,Debian Container

🤔 Kritik Analiz: Hangi Veritabanına Bağlıyım?
Geliştirme yaparken bağlantınızın local mi yoksa docker mı olduğunu doğrulamak için her zaman psql bağlantı loglarındaki Server Version (Sunucu Sürümü) ve OS detaylarına odaklanın veya bağlandığınız terminalde şu SQL sorgusunu çalıştırın:
SELECT version();
Çıktıda Debian ibaresini ve 10.15 sürümünü görüyorsanız: Docker Konteynerine bağlısınız.

Çıktıda salt sürüm derlemesi ve 10.13 sürümünü görüyorsanız: Yerel Sunucuya (Local) bağlısınız.


