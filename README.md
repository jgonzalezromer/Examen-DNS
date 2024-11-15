# Examen-DNS

---

Juan Gabriel González Romero
## Preguntas
```
Crea un repositorio para contestar todo o exame.
Este repositorio ten que conter tódo-los ficheiros necesarios para xustifica-la túas respostas.

Contesta as seguintes preguntas, xustificándoas, nun README.md

1. Explica métodos para 'abrir' unha consola/shell a un contenedor en execución.
2. No contenedor anterior (en execución), qué opciones tes que ter usado ó arrincalo para poder interactuar coas súas entradas e salidas
3. Cómo sería un ficheiro docker-compose para que dous contenedores se comuniquen entre si nunha rede só deles?
4. Qué tes que engadir ó ficheiro anterior para que un contenedor teña unha IP fixa?
5. Qué comando de consola podo usar para sabe-las ips dos contenedores anteriores?
6. Cál é a funcionalidade do apartado "ports" en docker compose?
7. Para qué serve o rexistro CNAME? Pon un exemplo
8. Cómo podo facer para que a configuración dun contenedor DNS non se borre se creo outro contenedor?
9. Engade unha zoa tendaelectronica.int no teu docker DNS que teña:
- www á IP 172.16.0.1
- owncloud sexa un CNAME de www
- un rexistro de texto có contido "1234ASDF"
Comproba que todo funciona có comando "dig"
Mostra nos logs que o servicio funciona ben usando a saída da terminal ó levantar o compose ou có comando "docker logs [nomeContenedorOuID]"
(o apartado 9 realízase na máquina virtual)
```

---

### 1. Explica métodos para 'abrir' unha consola/shell a un contenedor en execución.
Está pregunta ten duas respostas viables, a primeira sería co comando `docker exec -it <Nome_do_contenedor> <Terminal_que_utilizaremos>` e o outro método é dunha forma gráfica en Visual Studio Code, xa tendo instalado docker nel, iremos  ao apartado de `Container` e no menú que desplegase ao darlle clic dereito encima do contenedor daremoslle a `Atach shell`

---

### 2. No contenedor anterior (en execución), qué opciones tes que ter usado ó arrincalo para poder interactuar coas súas entradas e salidas

Debemos utilizar no comando `docker run <imaxe_do_contenedor>` a opción `-it` para poder interactuar coas entradas e saidas. O comando quedaría `docker run -it <imaxe_do_contenedor>`.

---

### 3. Cómo sería un ficheiro docker-compose para que dous contenedores se comuniquen entre si nunha rede só deles?
```
services:
  # Definimos os servizos que lanzaremos co docker compose
  Contenedor1: #Nome do primeiro contenedor
    image: bin9 #Imaxe do primeiro contenedor
    networks: #A que networks pertence o contenedor
      Red_Examen: #Nome da rede a que pertence o contenedor
  Contenedor2: #Nome do segundo contenedor
    image: alpine #Imaxe do primeiro contenedor
    tty: true #Que o contenedor siga activo
    stdin_open: true #Que o contenedor siga activo
    networks: #A que networks pertence o contenedor
      Red_Examen: #Nome da rede a que pertence o contenedor
networks:
  #Redes que utilizará o docker-compose
  Red_Examen: #Nome dunha rede
    driver: bridge #Tipo da rede
    ipam:
      config: #Configuración da rede
        - subnet: 172.16.0.0/16
          ip_range: 172.16.0.0/24
          gateway: 172.16.0.254

```

---

### 4. Qué tes que engadir ó ficheiro anterior para que un contenedor teña unha IP fixa?
No apartado de Networks de cada contenedor engadiremos:
```
networks:
      P7_network:
        ipv4_address: <IP_que_queramos_ponerlle>
```
Quedaría:
```
services:
  Contenedor1:
    image: bind9
    networks:
      Red_Examen:
        ipv4_address: 172.16.0.1
  Contenedor2:
    image: alpine
    tty: true
    stdin_open: true
    networks:
      Red_Examen:
        ipv4_address: 172.16.0.2
networks:
  Red_Examen:
    driver: bridge
    ipam:
      config:
        - subnet: 172.16.0.0/16
          ip_range: 172.16.0.0/24
          gateway: 172.16.0.254
```
---

### 5. Qué comando de consola podo usar para sabe-las ips dos contenedores anteriores?
Para saber as IPs dos contenedores usaremos nunha teminal o comando `docker container inspect <Nome_do_container> | grep "IP"`.

---

### 6. Cál é a funcionalidade do apartado "ports" en docker compose?

Decirlle o contenedor a que portos seus corresponde os portos da máquina

O apartado `ports` utilizase para mostrarlle ao contenedor que portos asocianse do contenedor a máquina real(mapeo de portos). Un exemplo sería:
```
ports:
      #Mapeo dos portos host:contenedor
      - 54:53/udp
      - 54:53/tcp
      - 127.0.0.1:953:953/tcp #Neste só pertmite conexións desde localhost
```
O .yml quedaría:
```
services:
  Contenedor1:
    image: bind9
    ports:
      - 54:53/udp
      - 54:53/tcp
      - 127.0.0.1:953:953/tcp
    networks:
      Red_Examen:
        ipv4_address: 172.16.0.1
  Contenedor2:
    image: alpine
    tty: true
    stdin_open: true
    networks:
      Red_Examen:
        ipv4_address: 172.16.0.2
networks:
  Red_Examen:
    driver: bridge
    ipam:
      config:
        - subnet: 172.16.0.0/16
          ip_range: 172.16.0.0/24
          gateway: 172.16.0.254
```
---

### 7. Para qué serve o rexistro CNAME? Pon un exemplo

Este rexistro serve para indicar o nome canónico, e dicir que un dominio sexa un alias para outro. Soen utilizarse para asociar novos subdominios con dominios xa existentes.

---

### 8. Cómo podo facer para que a configuración dun contenedor DNS non se borre se creo outro contenedor?

Para que non se borre a configuración do servidor DNS o que faremos será guardar esta configuración nun volume fora do contenedor este indicaríase nun apartado de volumes no apartado de DNS, así quedaría un exemplo:
```
services:
  Contenedor1:
    image: bind9
    ports:
      - 54:53/udp
      - 54:53/tcp
      - 127.0.0.1:953:953/tcp
    networks:
      Red_Examen:
        ipv4_address: 172.16.0.1
     volumes:
      - ./etc/bind:/etc/bind
      - ./var/cache/bind:/var/cache/bind
      - ./var/lib/bind:/var/lib/bind
  Contenedor2:
    image: alpine
    tty: true
    stdin_open: true
    networks:
      Red_Examen:
        ipv4_address: 172.16.0.2
networks:
  Red_Examen:
    driver: bridge
    ipam:
      config:
        - subnet: 172.16.0.0/16
          ip_range: 172.16.0.0/24
          gateway: 172.16.0.254
```
Agora podremos por as opcions do DNS no directorio `./etc/bind`.
---

### 9. Engade unha zoa tendaelectronica.int no teu docker DNS que teña:
- www á IP 172.16.0.1
- owncloud sexa un CNAME de www
- un rexistro de texto có contido "1234ASDF"
Comproba que todo funciona có comando "dig"
Mostra nos logs que o servicio funciona ben usando a saída da terminal ó levantar o compose ou có comando "docker logs [nomeContenedorOuID]"

Configuraremos a zona no documento `named.conf.local` este estará ubicado en `./etc/bind`(tamén se pode facer todo no documento `named.conf`):
```
zone "tendaelectronica.int" {
    type master;
    file "/var/lib/bind/db.tendaelectronica.int";
    allow-query{
            any;
    };
};
```
Logo faremos un arquivo chamado `db.tendaelectronica.int` no directorio `./var/lib/bind/` coa seguinte información:
```
$TTL 86400 ;
@ IN SOA ns.tendaelectronica.int. some.email.address. (
2024111501 ; Serial
3600 ; Refresh
1800 ; Retry
36000 ; Expire
86400 ; Minimum TTL
)
@ IN NS ns.tendaelectronica.int.
ns IN A 172.16.0.2
www IN A 172.16.0.1 
owncloud IN CNAME www 
texto IN TXT "1234ASDF" 
```
Como o comando dig non me foi non podo mostrar como quedaría pero este sería un exemplo:
```
; <<>> DiG 9.18.31 <<>> @172.18.0.1 practica7.int
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 31289
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 106b97c7fb094b0701000000673644b67358f8270c8d4905 (good)
;; QUESTION SECTION:
;practica7.int.			IN	A

;; AUTHORITY SECTION:
practica7.int.		86400	IN	SOA	ns.practica7.in.practica7.int. some.email.address. 2024111401 3600 1800 36000 86400

;; Query time: 0 msec
;; SERVER: 172.18.0.1#53(172.18.0.1) (UDP)
;; WHEN: Thu Nov 14 18:43:02 UTC 2024
;; MSG SIZE  rcvd: 153
```

---
