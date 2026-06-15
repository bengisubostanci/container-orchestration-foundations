Ağ Detaylarını Keşfetme:
Linux terminalinizde varsayılan köprü ağını incelemek ve ağ arayüzlerini karşılaştırmak için şu komutları çalıştırabilirsiniz:

docker network inspect bridge
ifconfig

💡 Teknik İpucu: inspect çıktısındaki Gateway (Geçit) adresi ile ifconfig çıktısındaki docker0 sanal ağ arayüzünün IP adresi birbiriyle tamamen aynıdır. Docker, host makinede bir köprü (bridge) oluşturarak trafiği buradan akıtır.

🛠️ Bölüm 3: Kullanıcı Tanımlı Köprü Ağı (User-Defined Bridge) Oluşturma
Neden Varsayılan Ağ Yerine Kendi Ağımızı Oluşturmalıyız?
Ağ İzolasyonu ve Güvenlik: Örneğin, veritabanı konteynerini sadece web uygulamamızın erişebileceği özel bir ağa alıp dış dünyadan tamamen soyutlayabiliriz.

Otomatik DNS Çözümleme (Konteyner İsmiyle İletişim): Varsayılan bridge ağındaki konteynerler birbirlerine isimleriyle (container_name) ulaşamazlar, sadece IP adresleriyle ulaşabilirler (IP'ler dinamik değiştiği için bu verimsizdir). Ancak Kullanıcı Tanımlı (User-Defined) bir ağda Docker, dahili bir DNS sunucusu çalıştırarak konteynerlerin birbirlerine isimleriyle ping atabilmesini sağlar.

🚀 4. Uygulama: Konteynerler Arası İsim ile Haberleşme (DNS Testi)
Senaryo A: Varsayılan Ağda Başarısız Bağlantı Denemesi
Herhangi bir ağ parametresi belirtmeden iki konteyner açıp isimle ping atmayı deneyelim:

# 1. Terminalde Nginx başlatalım
docker run --rm --name nginx -p 8081:80 -d nginx

# 2. Terminalde interaktif bir Alpine Linux başlatalım
docker container run --rm -it alpine /bin/sh

Alpine konteynerinin içinde Nginx'e ismiyle ulaşmaya çalışalım:
/ # ping nginx
# Çıktı: ping: bad address 'nginx' (Bağlantı başarısız, çünkü varsayılan ağda dahili DNS kapalıdır.)
/ # exit

Senaryo B: Kullanıcı Tanımlı Özel Ağda Başarılı Bağlantı
Şimdi kendi ağımızı oluşturup konteynerleri bu ağa bağlayalım:
# Özel köprü ağını oluşturalım
docker network create my-net
Ağın başarıyla eklendiğini doğrulayalım:
docker network ls
# Çıktıda 'my-net' ağının 'bridge' sürücüsüyle oluşturulduğunu göreceksiniz.

Konteynerlerimizi --net my-net parametresiyle bu ağa dahil ederek yeniden başlatalım:
# Nginx'i özel ağda başlatalım
docker run --rm --name nginx -p 8081:80 -d --net my-net nginx

# Alpine'ı aynı özel ağda başlatalım
docker container run --rm --net my-net -it alpine /bin/sh

Şimdi Alpine içerisinden Nginx'e ismiyle ping atalım ve HTTP isteği gönderelim:
/ # ping ngin
Çıktı:
PING nginx (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.136 ms
64 bytes from 172.18.0.2: seq=1 ttl=64 time=0.178 ms

(Ping işlemini durdurmak için Ctrl + C yapabilirsiniz)

Dahili web servisine erişimi doğrulamak için curl aracını yükleyip Nginx ana sayfasını çekelim:

/ # apk update && apk add curl
/ # curl nginx

🎉 Başarılı! Nginx'in hoş geldiniz HTML içeriğinin terminale aktığını göreceksiniz. Konteynerler IP adresine ihtiyaç duymadan birbirlerini isimlerinden tanıdılar.
/ # exit

🖥️ Bölüm 5: Konteyner İçerisinden Host (Ana Makine) Servislerine Erişim
Bazen konteyner içinde çalışan bir uygulamanın, doğrudan host makinede (Docker dışında) çalışan bir servise (örneğin Gitea, yerel PostgreSQL vb.) erişmesi gerekir.

Bunun için Host makinenin Docker köprü arayüzündeki (docker0) IP adresini bulmamız gerekir.

1. Host IP Adresini Öğrenme
Linux terminalinizde şu komutla docker0 ağ geçidinin IP'sini süzün:
ip a | grep -A2 docker0:
Örnek Çıktı:
6: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 ...
    link/ether 02:42:b9:a7:fd:23 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0

   Buradan ana makinemizin konteynerlere bakan IP adresinin 172.17.0.1 olduğunu öğreniyoruz.

2. Host Üzerindeki Servisi Başlatma ve Erişim Testi
Ana makinede (Docker dışında) çalışan gitea servisini başlatalım (Gitea varsayılan olarak 3000 portunda çalışır):
sudo systemctl start gitea
Şimdi herhangi bir Docker konteynerinin içine girip, öğrendiğimiz Host IP'sine istek atalım:
docker container run --rm -it alpine /bin/sh
/ # apk add curl
/ # curl 172.17.0.1:3000

🚀 Sonuç: Konteyner içinden başarıyla çıkıp, host makinedeki lokal Gitea servisinin HTML içeriğine ulaştınız!
(Alternatif olarak modern Docker sürümlerinde IP adresi yerine doğrudan host.docker.internal DNS adresi de kullanılabilir).
/ # exit
