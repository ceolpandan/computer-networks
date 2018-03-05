## Laborator 1

### In prealabil
Rulati comenzi de docker din https://github.com/senisioi/computer-networks.

Stergeti toate containerele create si resetati modificarile efectuate in branch-ul local de git.
```bash
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)
docker network prune

git fetch --all
git reset --hard origin/master
```

### Comenzi de baza
```bash
# executati un shell in containerul rt1
docker-compose exec rt1 bash

# listati configuratiile de retea
ifconfig

### eth0 - Ethernet device to communicate with the outside  ###
# eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
#        inet 172.27.0.3  netmask 255.255.0.0  broadcast 0.0.0.0
#        ether 02:42:ac:1b:00:03  txqueuelen 0  (Ethernet)
# lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
#        inet 127.0.0.1  netmask 255.0.0.0
#        loop  txqueuelen 1000  (Local Loopback)
```

Comanda *ifconfig* ne indica doua device-uri care ruleaza pe containerul *rt1*:

- *eht0* - placa de retea Ethernet virtuala care indica configuratia pentru stabilirea unei conexiuni de retea a containerului.
- *lo* - [local Loopback device](https://askubuntu.com/questions/247625/what-is-the-loopback-device-and-how-do-i-use-it) care defineste categoria de adrese care se mapeaza pe localhost.
- Ce este ether?
- Ce este inet?
- Ce este [netmask](https://www.computerhope.com/jargon/n/netmask.htm)?
- Netmask si Subnet cu [prefix notation](https://www.ripe.net/about-us/press-centre/IPv4CIDRChart_2015.pdf)?
- Maximum Transmission Unit [MTU](https://en.wikipedia.org/wiki/Maximum_transmission_unit) dimensiunea in bytes a pachetului maxim

##### Exercitiu
Modificati docker-compose.yml pentru a adauga inca o retea si inca 3 containere atasate la reteaua respectiva. Modificati definitia container-ului rt1 pentru a face parte din ambele retele. 
Exemplu de retele:
```bash
networks:
    dmz:
        ipam:
            driver: default
            config:
                - subnet: 172.111.111.0/16 
                  gateway: 172.111.111.1
    net:
        ipam:
            driver: default
            config:
                - subnet: 198.13.13.0/16
                  gateway: 198.13.13.1
```
Ce se intampla daca constrangeti subnet-ul definit pentru a nu putea permite mai mult de 4 ip-uri intr-o retea.

### Ping
Este un tool de networking care se foloseste de [ICMP](https://en.wikipedia.org/wiki/Internet_Control_Message_Protocol) pentru a verifica daca un host este conectat la o retea prin IP.

```bash
# ping localhost and loopback
ping localhost

ping 127.0.0.1

ping 127.0.2.12

# ping neighbour from network dmz
ping 172.111.0.3

# ping neighbour from network net
ping 198.13.13.1
```

1. Cand rulati ping, utilizati optiunea -R pentru a vedea si calea pe care o efectueaza pachetul.

2. Ce reprezinta adresa 127.0.2.12? De ce functioneaza ping catre aceasta?

3. Intr-un terminal nou, rulati comanda `docker network inspect computernetworks_dmz` pentru a vedea ce adrese au celelalte containere. Incercati sa trimiteti un ping catre adresele IP ale celorlalte containere.

4. Folositi `docker stop` pentru a opri un container, cum arata rezultatul comenzii `ping` catre adresa IP a containerului care tocmai a fost oprit?

4. Retelele dmz si net au in comun containerul rt1. Un container din reteaua dmz primeste raspunsuri la ping de la containere din reteaua net?

5. Folositi optiunea `-c 10` pentru a trimite un numar fix de pachete.

6. Folositi optiunea `-s 1000` pentru a schimba dimensiunea pachetului ICMP

7. Reporniti toate containerele. Cum arata rezultatele pentru `ping -M do -s 30000 172.111.0.4` si pentru `ping -M do -s 30000 172.111.0.4`. Care este diferenta dintre cele doua? Care este rezultatul daca selectati dimensiunea 1500?

8. Optiunea `-f` este folosita pentru a face un flood de ping-uri.  Rulati un shell cu user root, apoi `ping -f 172.111.0.4`. Separat, intr-un alt terminal rulati `docker stats`. Ce observati?

9. De multe ori raspunsurile la ping [sunt dezactivate](https://superuser.com/questions/318870/why-do-companies-block-ping) pe servere. Pentru a dezactiva raspunsul la ping rulati intr-un container cu userul root: `echo "1" > /proc/sys/net/ipv4/icmp_echo_ignore_all`


### tcpdump
Este un tool care va permite monitorizarea traficului de pe containerul/masina pe care va aflati. Vom folosi *tcpdump* pentru a monitoriza traficul generat de comanda ping. Pentru a rula tcpdump, trebuie sa ne atasam unui container cu user **root** apoi putem rula:

```bash
# pentru a capta pachete circula verbose default
# -i denota interfata pe care o folosim pentru a capta pachete, in cazul acesta eth0
# -n indica afisarea valorilor numerice ale ip-urilo in loc de valorile date de nameserver
# -tttt afiseaza pachetele cu timestamp
tcpdump -vv -n -tttt -i eth0
```
Daca tcpdump nu exista ca aplicatie, va trebui sa modificati fisierul *Dockerfile* pentru a adauga comanda de instalare a acestei aplicatii. Reconstruiti imaginea folosind comanda docker build, distrugeti si reconstruiti containerele folosind `docker-compose down` si `docker-compose up -d`.
Daca in urma rularii acestei comenzi nu apare nimic, inseamna ca in momentul acesta interfata data pe containerul respectiv nu executa operatii pe retea. Pentru a vedea ce interfete (device-uri) putem folosi pentru a capta pachete, putem rula:
```bash
tcpdump -D
```


##### Exercitii
In paralel cu terminalul in care ati rulat tcpdump, deschideti un alt terminal pe care sa-l folositi pentru a genera diferite tipuri de trafic:
```bash
ping google.com
ping VECIN_DE_PE_RETEA
ping localhost

wget https://github.com/senisioi/computer-networks/
```

In paralel rulati tcpdump cu diferite optiuni:
```bash
# -c pentru a capta un numar fix de pachete
tcpdump -c 20

# -w pentru a salva pachetele intr-un fisier si -r pentru a citi fisierul
tcpdump -w pachete.pcap
tcpdump -r pachete.pcap

# pentru a afisa doar pachetele care vin sau pleaca cu adresa google.com
tcpdump host google.com

# folositi -XX pentru a afisa si continutul in HEX si ASCII
tcpdump -XX
```

 - Intrebare: este posibil sa captati pachetele care circula intre google.com si rt2 folosind masina rt1?
 - Pentru mai multe detalii puteti urmari acest [tutorial](https://danielmiessler.com/study/tcpdump/)
