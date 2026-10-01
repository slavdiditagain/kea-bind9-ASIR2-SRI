# EXÁMEN DE REDES 07/10/2026

Guia explicativa que cubrira la mayoria de puntos sobre el examen de redes sobre Kea.

Recomiendo consultar el [README](README.md) que se adjunta con este repositorio sobretodo porque sirve como guia de instalación de manera real es bastante útil y así te dará mayor idea para poder comprender todo lo que explica dentro de aquí.

## 1. Servidor IP fija. Kea instalado. Red Interna

**Toda esta configuración es tocada con mayor detalle en [README](README.md)**

> [!] Como ya he avisado, recomiendo que esto sea material de apoyo más que otra cosa.

## 2. Subred, rango(pool), exclusiones

### Previos
---
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
### Subredes
---
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

### Rango (pool)
---
El pool de manera resumida son los rangos de ips, es decir las ips que se les daran a los distintos equipos que estén en la misma red, por ejemplo si tenemos está configuración:

**Server = 172.16.0.1**
**Cliente = 172.17.0.0**

Ahora si conectamos distintos ordenadores iran agarrando ips

Esta se configura expresamente en /etc/kea/kea-dhcp4.conf
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


## 3. Gateway, servidores DNS, dominio...

## 4. Tiempos de confesion, T1, T2

## 5. Reserva por dirección fisica

## 6. /etc/bind/named.conf.options (Distintas configuraciones)
 
## 7. Zona directa, registro DNS ,SOA, NS, A, MX, CNAMF

## 8. Zona inversa. PTR