1. 
PS C:\Users\Student\mateuszdziwis\breach-checker> docker inspect --format '{{json .Config.Labels}}' mateusz:v1
{"autor":"Mateusz Dziwis","data":"2026-10-14","maintainer":"NGINX Docker Maintainers <docker-maint@nginx.com>"}
PS C:\Users\Student\mateuszdziwis\breach-checker> docker inspect --format '{{json .Config.Labels}}' mateusz:latest
{"autor":"Mateusz Dziwis","data":"2026-10-14","maintainer":"NGINX Docker Maintainers <docker-maint@nginx.com>"}

PS C:\Users\Student\mateuszdziwis\breach-checker> docker history mateusz:v1
IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
b1b20f94fa5a   2 minutes ago    EXPOSE [80/tcp]                                 0B        buildkit.dockerfile.v0
<missing>      2 minutes ago    RUN /bin/sh -c htmlhint /usr/share/nginx/htm…   4.1kB     buildkit.dockerfile.v0
<missing>      2 minutes ago    COPY . /usr/share/nginx/html # buildkit         32.8kB    buildkit.dockerfile.v0
<missing>      16 minutes ago   RUN /bin/sh -c apk add --no-cache nodejs npm…   102MB     buildkit.dockerfile.v0
<missing>      16 minutes ago   LABEL autor=Mateusz Dziwis data=2026-10-14      0B        buildkit.dockerfile.v0
<missing>      40 hours ago     RUN /bin/sh -c set -x     && apkArch="$(cat …   52.4MB    buildkit.dockerfile.v0
<missing>      40 hours ago     ENV ACME_VERSION=0.4.1                          0B        buildkit.dockerfile.v0
<missing>      40 hours ago     ENV NJS_RELEASE=1                               0B        buildkit.dockerfile.v0
<missing>      40 hours ago     ENV NJS_VERSION=1.0.2                           0B        buildkit.dockerfile.v0
<missing>      41 hours ago     CMD ["nginx" "-g" "daemon off;"]                0B        buildkit.dockerfile.v0
<missing>      41 hours ago     STOPSIGNAL SIGQUIT                              0B        buildkit.dockerfile.v0
<missing>      41 hours ago     EXPOSE map[80/tcp:{}]                           0B        buildkit.dockerfile.v0
<missing>      41 hours ago     ENTRYPOINT ["/docker-entrypoint.sh"]            0B        buildkit.dockerfile.v0
<missing>      41 hours ago     COPY 30-tune-worker-processes.sh /docker-ent…   16.4kB    buildkit.dockerfile.v0
<missing>      41 hours ago     COPY 20-envsubst-on-templates.sh /docker-ent…   12.3kB    buildkit.dockerfile.v0
<missing>      41 hours ago     COPY 15-local-resolvers.envsh /docker-entryp…   12.3kB    buildkit.dockerfile.v0
<missing>      41 hours ago     COPY 10-listen-on-ipv6-by-default.sh /docker…   12.3kB    buildkit.dockerfile.v0
<missing>      41 hours ago     COPY docker-entrypoint.sh / # buildkit          8.19kB    buildkit.dockerfile.v0
<missing>      41 hours ago     RUN /bin/sh -c set -x     && addgroup -g 101…   11.9MB    buildkit.dockerfile.v0
<missing>      41 hours ago     ENV DYNPKG_RELEASE=1                            0B        buildkit.dockerfile.v0
<missing>      41 hours ago     ENV PKG_RELEASE=1                               0B        buildkit.dockerfile.v0
<missing>      41 hours ago     ENV NGINX_VERSION=1.31.7                        0B        buildkit.dockerfile.v0
<missing>      41 hours ago     LABEL maintainer=NGINX Docker Maintainers <d…   0B        buildkit.dockerfile.v0
<missing>      4 weeks ago      CMD ["/bin/sh"]                                 0B        buildkit.dockerfile.v0
<missing>      4 weeks ago      ADD alpine-minirootfs-3.24.2-x86_64.tar.gz /…   9.08MB    buildkit.dockerfile.v0

Warstwy początkowe pochodzą z obrazu bazowego nginx:alpine, a kolejne zostały utworzone przez instrukcje z pliku Dockerfile. Rozmiar 0B mają te warstwy, które modyﬁkują jedynie metadane obrazu i nie dodają żadnych plików do systemu.

2. 
Kontener 1: Fri Oct 16 11:16:01 UTC 2026
Kontener 2: Fri Oct 16 11:16:01 UTC 2026

PS C:\Users\Student\mateuszdziwis\breach-checker> docker ps
CONTAINER ID   IMAGE            COMMAND                  CREATED          STATUS          PORTS                   NAMES
eda167ec9363   mateusz:latest   "/docker-entrypoint.…"   5 seconds ago    Up 4 seconds    0.0.0.0:32769->80/tcp   mateusz1
900ee0b32701   mateusz:latest   "/docker-entrypoint.…"   53 seconds ago   Up 52 seconds   0.0.0.0:32768->80/tcp   mateusz

Instrukcja RUN wykonuje się podczas budowania obrazu, a nie podczas uruchamiania kontenera, więc do pliku build.txt zostanie zapisana data i godzina podczas zbudowania tego konkretnego obrazu, a podczas uruchomienia kontenera plik build.txt zostanie skopiowany.

3. 
        root   /usr/share/nginx/html;

Kopiujemy pliki własne do /usr/share/nginx/html, aby uruchomić własną aplikację. Domyślnie w tym folderze jest domyślna strona nginx, którą zastępujemy własną stroną.

4. 
PS C:\Users\Student\mateuszdziwis\breach-checker> docker exec mateusz ls /usr/share/nginx/html
50x.html
build.txt
index.html

PS C:\Users\Student\mateuszdziwis\breach-checker> docker exec mateusz ls /usr/share/nginx
html
index.html
script.js
style.css

htmlhint sprawdził plik index.html znajdujący się w katalogu źródłowym projektu. Build przeszedł bez błędu, ponieważ z punktu widzenia Dockera instrukcja COPY wykonała się poprawnie. Pliki zostały skopiowane do istniejącego i poprawnego ścieżkowo katalogu docelowego w kontenerze.

5. 
PS C:\Users\Student\mateuszdziwis\breach-checker> docker rmi breach-checker-frontend:latest
Untagged: breach-checker-frontend:latest

PS C:\Users\Student\mateuszdziwis\breach-checker> docker images
                                                                                                                                                                                             i Info →   U  In Use
IMAGE                             ID             DISK USAGE   CONTENT SIZE   EXTRA
alpine:latest                     294b683cb724         13MB         3.94MB    U
breach-checker-frontend:v1        779a4c8924fe        102MB         28.8MB
breach-checker-frontend:v2        8632d1247eca        102MB         28.8MB
breach-checker-frontend:v3        5956387f0457        238MB           62MB
docker/welcome-to-docker:latest   c4d56c24da4f       22.2MB         6.03MB
mateusz:latest                    33061035acdc        238MB           62MB    U
mateusz:v1                        b1b20f94fa5a        238MB           62MB
nginx:alpine                      892bb2152464        103MB         29.7MB    U
ubuntu:latest                     3595d7fc4286        162MB         45.6MB    U

PS C:\Users\Student\mateuszdziwis\breach-checker> docker rmi breach-checker-frontend:v2
Untagged: breach-checker-frontend:v2
Deleted: sha256:8632d1247eca5eaac032f9a45100f4a20ca409b35190a903fb6bbde1d36f20fd

PS C:\Users\Student\mateuszdziwis\breach-checker> docker images
                                                                                                                                                                                             i Info →   U  In Use
IMAGE                             ID             DISK USAGE   CONTENT SIZE   EXTRA
alpine:latest                     294b683cb724         13MB         3.94MB    U
breach-checker-frontend:v1        779a4c8924fe        102MB         28.8MB
breach-checker-frontend:v3        5956387f0457        238MB           62MB
docker/welcome-to-docker:latest   c4d56c24da4f       22.2MB         6.03MB
mateusz:latest                    33061035acdc        238MB           62MB    U
mateusz:v1                        b1b20f94fa5a        238MB           62MB
nginx:alpine                      892bb2152464        103MB         29.7MB    U
ubuntu:latest                     3595d7fc4286        162MB         45.6MB    U

docker rmi faktycznie usuwa obraz z dysku gdy usuwany jest ostatni tag wskazujacy na dany IMAGE ID oraz żadne kontenery nie korzystają z tego obrazu.