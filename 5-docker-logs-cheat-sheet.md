İşte Docker log komutları için hazırladığım en kısa, pratik ve iki dilli (cheat sheet) formatı:

EN 🇬🇧
View Logs:
docker logs postgresql

Stream/Follow Logs (Real-time):
docker logs -f postgresql

Tail Logs (Last few lines):
docker logs postgresql --tail 10

TR 🇹🇷
Logları Görüntüle:
docker logs postgresql

Logları Canlı Akış Olarak İzle:
docker logs -f postgresql

Logların Son Satırlarını Gör (Tail):
docker logs postgresql --tail 10
