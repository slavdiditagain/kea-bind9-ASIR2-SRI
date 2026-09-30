# Clase 30 de Septiembre de 2026 Redes

## 1. Servidor IP fija. Kea instalado. Red Interna

## 2. Subred, rango(pool), exclusiones

## 3. Gateway, servidores DNS, dominio...
/etc/kea/kea-dhcp4.conf
```
 "subnet4": [
      {
        "id": 1,
        "subnet": "172.16.0.0/12",
        "pools": [
          { "pool": "172.20.0.0 - 172.30.0.0" }
        ],
        "option-data": [
          { "name": "routers", "data": "172.16.0.1" }
        ],
        "reservations": [
          {
            "hw-address": "[MAC DE LA MÁQUINA CLIENTE]",
            "ip-address": "172.17.0.0",
            "hostname": "cliente"
          }
        ]
      }
    ],
```

Sudo -u _kea kea-dhcp4 -t /etc/kea-dhcp4.conf

## 4. Tiempos de confesion, T1, T2
```
    "authoritative": true,
    "valid-lifetime": 3600,
    "renew-timer": 1800,
    "rebind-timer": 3150,
```
Se le puede añadir porcentajes de t1 - percent para las renovaciones de IP ya sea con porcentajes, o calculos directos.

## 5. Reserva por dirección fisica
