PS C:\Users\Student\Documents\Mateusz Dziwis\breach-checker> curl.exe http://localhost:5000/health
{"status":"ok"}
PS C:\Users\Student\Documents\Mateusz Dziwis\breach-checker> docker logs backend
Wczytano 89 hashy z /app/data/leaked-sha1.txt
 * Serving Flask app 'app'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:5000
 * Running on http://172.18.0.2:5000
Press CTRL+C to quit
172.18.0.1 - - [23/Oct/2026 11:28:20] "GET /health HTTP/1.1" 200 -
172.18.0.1 - - [23/Oct/2026 11:28:29] "GET /health HTTP/1.1" 200 -
PS C:\Users\Student\Documents\Mateusz Dziwis\breach-checker> 

PS C:\Users\Student\Documents\Mateusz Dziwis\breach-checker> docker run --rm --network breach-net healthcheck-probe
backend: OK
PS C:\Users\Student\Documents\Mateusz Dziwis\breach-checker> docker run --rm healthcheck-probe
backend: UNREACHABLE
PS C:\Users\Student\Documents\Mateusz Dziwis\breach-checker> docker run --rm --network breach-net healthcheck-probe
backend: UNREACHABLE

domyślna sieć nie udostępnia wewnętrznego DNS Dockera, który tłumaczy nazwę kontenera na adres IP

PS C:\Users\Student\Documents\Mateusz Dziwis\breach-checker> docker exec backend sh -c "echo notatka > /tmp/notatka.txt"
PS C:\Users\Student\Documents\Mateusz Dziwis\breach-checker> docker restart backend
backend
PS C:\Users\Student\Documents\Mateusz Dziwis\breach-checker> docker exec backend cat /tmp/notatka.txt
notatka
PS C:\Users\Student\Documents\Mateusz Dziwis\breach-checker> docker rm -f backend
backend
PS C:\Users\Student\Documents\Mateusz Dziwis\breach-checker> docker run -d --name backend --network breach-net -p 5000:5000 -v ${PWD}/data:/app/data breach-backend
afc705a1d8995a74084d17a1e54ecd23fcd3e37ea1c9a0a786133354b1240215
PS C:\Users\Student\Documents\Mateusz Dziwis\breach-checker> docker exec backend cat /tmp/notatka.txt
cat: /tmp/notatka.txt: No such file or directory

PS C:\Users\Student\Documents\Mateusz Dziwis\breach-checker> docker run --rm -v breach-dane:/dane alpine sh -c "date >> /dane/historia.txt"
PS C:\Users\Student\Documents\Mateusz Dziwis\breach-checker> docker run --rm -v breach-dane:/dane alpine sh -c "date >> /dane/historia.txt; cat /dane/historia.txt"
Fri Oct 23 11:30:48 UTC 2026
Fri Oct 23 11:30:53 UTC 2026

Plik w /tmp/notatka.txt przetrwał restart, ponieważ operacja restartu zachowuje warstwę zapisywalną kontenera. Nie przetrwał jednak polecenia rm, które całkowicie niszczy kontener i jego tymczasową warstwę zapisu.

Plik historia.txt przetrwał usunięcie obu kontenerów, ponieważ został zapisany w osobnym wolumenie dockera, którego cykl życia jest niezależny od pojedynczych kontenerów.

Lista wycieków również przetrwałaby docker rm, ponieważ była montowana jako wolumen lub bind mount z dysku hosta, a dane fizycznie znajdują się poza kontenerem.