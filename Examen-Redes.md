# EXÁMEN DE REDES 07/10/2026

# ESTA GUIA NO ES PARA COPIAR Y PEGAR (en su mayoria) sirve para aprender que hacen las cosas, si quieren pasarlo a la IA para super resumirlo me parece perfecto.

Guia explicativa que cubrira la mayoria de puntos sobre el examen de redes sobre Kea.

Recomiendo consultar el [README](README.md) que se adjunta con este repositorio sobretodo porque sirve como guia de instalación de manera real es bastante útil y así te dará mayor idea para poder comprender todo lo que explica dentro de aquí.


# 1. Servidor IP fija. Kea instalado. Red Interna

**Toda esta configuración es tocada con mayor detalle en [README](README.md)**

> [!] Como ya he avisado, recomiendo que esto sea material de apoyo más que otra cosa.

# 2. Subred, rango(pool), exclusiones

## Previos

La mayoria de consiguraciones es hacen el siguiente archivos: /etc/kea/kea-dhcp4.conf y /etc/netplan/00-installer-config.yaml

**Dentro /etc/netplan/00-installer-config.yaml configuramos la IP fija, como se ha establecido anteriormente pero aparte tiene una configuración importante para que funcione con el demás.**

```
network:
  ethernets:
    enp0s3: 
      dhcp4: true
      dhcp6: false
    enp0s8:
      addresses: [172.16.0.1/12]
      accept-ra: true
      nameservers:
        search: [fp.internal]
        addresses: [172.16.0.1]
  version: 2
```

## Subredes
Aquí estamos configurando las redes, lo importante es sobretodo en el caso de la red interna saber cual es la red que vamos a tomar, por ejemplo si queremos establecer una red 172.16.0.0/12 nos tendremos que atener a las limitaciones y reglas de la red elegida.

Si tomamos como ejemplo las subredes que podría tener 172.16.0.0/12 al ser una ip de Clase B, abarca de 172.16.0.0 hasta 172.31.255.255 con una máscara de red 255.240.0.0

**Subredes que puede tener**
| Subred (CIDR) | Máscara de Red | ID de Red | Primera IP Útil | Última IP Útil | Dirección Broadcast |
|---|---|---|---|---|---|
| 172.16.0.0/16 | 255.255.0.0 | 172.16.0.0 | 172.16.0.1 | 172.16.255.254 | 172.16.255.255 |
| 172.17.0.0/16 | 255.255.0.0 | 172.17.0.0 | 172.17.0.1 | 172.17.255.254 | 172.17.255.255 |
| 172.18.0.0/16 | 255.255.0.0 | 172.18.0.0 | 172.18.0.1 | 172.18.255.254 | 172.18.255.255 |
| ... (omitidas) | ... | ... | ... | ... | ... |
| 172.31.0.0/16 | 255.255.0.0 | 172.31.0.0 | 172.31.0.1 | 172.31.255.254 | 172.31.255.255 |

> [!] Está red obviamente puede cambiar y puedes elegir la que quieras, pero siempre tienes que tener idea de como hacer cada una de las subredes necesarias.

## Rango (pool)
El pool de manera resumida son los rangos de ips, es decir las ips que se les daran a los distintos equipos que estén en la misma red, por ejemplo si tenemos está configuración:

**Server = 172.16.0.1**
**Cliente = 172.17.0.0**

Ahora si conectamos distintos ordenadores iran agarrando ips

### Archivo: /etc/kea/kea-dhcp4.conf:
```
...
"subnet4": [
    {
        "subnet": "172.16.0.0/12",
        "pools": [
            {
                // Este es el rango pool dinámico
                "pool": "172.20.0.0 - 172.30.0.0" 
            }
        ]
    }
}
```

## Exclusiones
En el caso de las exclusiones como tal no hay una pólitica directa de exclusiones, pero se pueden hacer de distintas formas (solo mostraré una):
### Fragmentación de rangos mediante (pools)
Como se ha dicho en lugar de excluir direcciones, se usa un rango de ips dinámicas dejando huecos fuera de la pools.

```
{
  "Dhcp4": {
    "subnet4": [
      {
        "subnet": "172.16.0.0/12",
        "pools": [
          // Se asignan de la .10 a la .49
          { "pool": "172.16.0.10 - 172.16.0.49" },
          // Se EXCLUYEN las IPs de la .50 a la .99 (hueco)
          // Se reanuda el pool dinámico en la .100
          { "pool": "172.16.0.100 - 172.16.0.200" }
        ]
      }
    ]
  }
}
```
# 3. Gateway, servidores DNS, dominio

En Kea todo esto son opciones DHCP (option-data) que se envián al cliente junto con la IP, es decir se pueden poner a nivel global o dentro de cada subred y **la de la subred pisa a la global.**
### Archivo: /etc/kea/kea-dhcp4.conf:
```
"option-data": [
  { "name": "routers", "data": "172.16.0.1" },
  { "name": "domain-name-servers", "data": "172.16.0.2, 172.16.0.3" },
  { "name": "domain-name", "data": "fp.internall" },
  { "name": "domain-search", "data": "fp.internal" }
]
```
- **routers:** Sirve para determinar **la puerta de enlace**
- **domain-name-servers:** Son los distintos DNS que usará el cliente, pueden ser varios y deben ser separados por coma.
- **domain-name:** Dominio que se asigna al cliente
- **domain-search:** Lista de sufijos de búsqueda, **para que ping servidor resuelva a servidor.fp.internal**
# 4. Tiempos de confesion, T1, T2
**valid-lifetime:** Segundos que dura la concesión (la lease).
**T1 (renew-timer):** Se usa cuando el cliente intenta renovar con el mismo servidor que le dio la ip.
**T2 (rebind-timer):** Si no se pudo renovar pregunta aquí a cualquier servidor DHCP.

### Archivo: /etc/kea/kea-dhcp4.conf:

```
"valid-lifetime": 3600,
"renew-timer": 1800,
"rebind-timer": 3150
```
Si no se define, Kea por defecto calcula: **T1 = 50% y T2 = 87,5 de la lease**. También existen min-valid-lifetime y max-valid-lifetime que acotan lo que el cliente puede pedir.
# 5. Reserva por dirección fisica
Explicado en [README](./README.md#4-comprobar-que-todo-está-bien-configurado) a la hora de configurar el Kea.
# 6. /etc/bind/named.conf.options (Distintas configuraciones)
**/etc/bind/named.conf.options** tiene distintas configuraciones en el bloque **options{...};** son estás las opciones que son clave:

- **directory:** carpeta de trabajo (/var/cache/bind).
- **listen-on / listen-on-v6:** en qué interfaces escucha.
- **recursion yes|no:** si resuelve consultas de terceros o solo responde de sus zonas.
- **allow-query:** quién puede preguntar.
- **allow-recursion:** quién puede usar la recursividad.
- **forwarders:** DNS a los que reenvía lo que no sabe.
- **allow-transfer:** quién puede descargar zonas (esclavos).
- **dnssec-validation:** auto, yes o no.

### Ejemplo con todas las funciones:

```
options {
    directory "/var/cache/bind";
    listen-on { 127.0.0.1; 172.16.0.1; };
    allow-query { localhost; 172.16.0.0/12; };
    recursion yes;
    allow-recursion { localhost; 172.16.0.0/12; };
    forwarders { 8.8.8.8; 1.1.1.1; };
    dnssec-validation auto;
};
```

### Servidor con esclavos sin recursividad:
```
options {
    directory "/var/cache/bind";
    recursion no;
    allow-query { any; };
    allow-transfer { 172.16.0.3; };   # IP del esclavo
};
```
# 7. Zona directa, registro DNS, SOA, NS, A, MX, CNAMF
### Archivo /etc/bind/named.conf.options:
```
zone "fp.internal" {
    type master;
    file "/etc/bind/db.fp.internal";
};
```
En "file" establecemos el lugar donde se va a guardar la db de la zona.

### Fichero de la zona /etc/bind/db.fp.internal:
```
$TTL 86400
@   IN  SOA server.fp.internal. admin.fp.internal. (
            2026100201  ; Serial
            3600        ; Refresh
            900         ; Retry
            604800      ; Expire
            86400 )     ; Negative cache TTL

@       IN  NS    server.fp.internal.
@       IN  MX 10 correo.fp.internal.

server     IN  A     172.16.0.1
correo  IN  A     172.16.0.5
www     IN  A     172.16.0.10
web     IN  CNAME www.fp.internal.
```

**SOA** datos de la zona:
- server: es el servidor privado.
- admin: es el correo del responsable.
- Serial: Se incrementa por cada cambio.
- Refresh: cada cuánto consulta el esclado si hay cambios.
- Retry: cada cuánto reintenta si falló.
- Expire: cuándo el esclavo deja de responder si no contacta con el maestro.
- Negative TTL: cuánto se cachea un "NOT EXIST".

### Definición de terminos:
- **DNS**: Se refiere a donde se resuelve el servidor.
- **NS**: Se refiere a servidores de nombres de la zona. Debe apuntar a un **nombre** y no a una IP
- **A**: nombre en IPv4
- **MX**: Servidor de correo con prioridad (menor número = más prioridad)
- **CNAME**: Alias de otro nombre no puede existir con otros registros del mismo nombre.
# 8. Zona inversa. PTR
**Resuelve IP → nombre.** El nombre de la zona se forma con los octetos de la red al revés más

Usaremos la IP que usamos para la practica de [README](README.md).

## Opción A (la más simple):

### Archivo /etc/bind/named.conf.options:
```
zone "172.in-addr.arpa" {
    type master;
    file "/etc/bind/db.172";
};
```

### PTR

Con la zona 172.in-addr.arpa, en cada PTR se escriben los tres octetos restantes al revés. Ejemplo: 172.16.0.2 → 2.0.16.
```
$TTL 86400
@   IN  SOA server.fp.internal. admin.fp.internal. (
            2026100201  ; Serial
            3600        ; Refresh
            900         ; Retry
            604800      ; Expire
            86400 )     ; Negative cache TTL

@       IN  NS   server.fp.internal.

2.0.16      IN  PTR  server.fp.internal.      ; 172.16.0.2
5.0.16      IN  PTR  correo.fp.internal.   ; 172.16.0.5
10.0.16     IN  PTR  www.fp.internal.      ; 172.16.0.10
20.1.16     IN  PTR  pc1.fp.internal.      ; 172.16.1.20
5.0.17      IN  PTR  srv.fp.internal.      ; 172.17.0.5
```

## Opción B: una zona por cada /16
16.172.in-addr.arpa, 17.172.in-addr.arpa... hasta 31.172.in-addr.arp. Son 16 zonas, pero es más ordenado en redes grandes o si quieres delegar partes a otros servidores.
### Archivo /etc/bind/named.conf.options:
```
zone "16.172.in-addr.arpa" {
    type master;
    file "/etc/bind/db.172.16";
};
```
### Fichero de zona: /etc/bind/db.172.16
```
$TTL 86400
@   IN  SOA server.fp.internal. admin.fp.internal. (
            2026100201 3600 900 604800 86400 )

@       IN  NS   server.fp.internal.

2.0     IN  PTR  server.fp.internal.      ; 172.16.0.2
5.0     IN  PTR  correo.fp.internal.   ; 172.16.0.5
10.0    IN  PTR  www.fp.internal.      ; 172.16.0.10
20.1    IN  PTR  pc1.fp.internal.      ; 172.16.1.20
```
> [!] Cada IP / Zona solo debe tener un PTR, si no todo falla.

## Enlace con Kea 
Si quieres que los PTR se creen solos con las leases, en Kea se indica la zona inversa en kea-dhcp-ddns:

### /etc/kea/kea-dhcp4.conf

```
"reverse-ddns": {
  "ddns-domains": [
    {
      "name": "172.in-addr.arpa.",
      "dns-servers": [ { "ip-address": "172.16.0.2" } ]
    }
  ]
}
```