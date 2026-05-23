EN 🇬🇧
Connect to Container Shell:
docker exec -it <name/id> /bin/bash
(Exit: exit)

Run & Connect to PostgreSQL:

Run: docker run --rm --name postgresql -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=Ankara06 -d postgres:15

Connect via psql: docker exec -it postgresql psql --username postgres (Exit: \q)

Connect via bash: docker exec -it postgresql bash

What is -it?:

-i (interactive): Keeps STDIN open.

-t (tty): Allocates a pseudo-TTY (terminal screen).

TR 🇹🇷
Konteyner Terminaline Bağlanma:
docker exec -it <isim/id> /bin/bash
(Çıkış: exit)

PostgreSQL Çalıştırma & Bağlanma:

Çalıştır: docker run --rm --name postgresql -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=Ankara06 -d postgres:15

psql ile bağlan: docker exec -it postgresql psql --username postgres (Çıkış: \q)

bash ile bağlan: docker exec -it postgresql bash

-it Ne Anlama Gelir?:

-i (interactive): Girdileri (STDIN) açık tutar.

-t (tty): Sanal bir terminal ekranı açar.
