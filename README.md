**Autores:** 
- Esteban Gaviria Zambrano - A00396019
- Juan Manuel Díaz Moreno - A00394477
- Santiago Valencia García - A00395902

**Institución:** Universidad Icesi - Administración de Plataformas I

---

# Infraestructura de Servidores DNS Primario y Secundario

Este proyecto implementa una infraestructura de Sistema de Nombres de Dominio (DNS) redundante mediante un servidor maestro (primario) y un servidor esclavo (secundario) utilizando BIND9. El despliegue se automatiza con Vagrant para la virtualización y Ansible para la gestión de configuración.

## Contexto y Propósito

La infraestructura DNS desarrollada proporciona resolución de nombres autoritativa para el dominio `plataformas.local` y sus zonas inversas correspondientes. El sistema opera con redundancia mediante la replicación automática de zonas entre el servidor primario y secundario, garantizando disponibilidad del servicio ante fallos del servidor maestro. La solución soporta dual-stack IPv4/IPv6 e incluye mecanismos de transición IPv6 mediante DNS64.

El proyecto se enmarca en el contexto de infraestructuras de red empresariales, donde la resolución de nombres constituye un servicio crítico que requiere alta disponibilidad, seguridad en las transferencias de zonas y capacidad de resolver tanto consultas locales como externas.

## Arquitectura

El sistema se compone de dos servidores virtuales ejecutando Ubuntu 22.04 LTS (Jammy). La sincronización entre servidores se asegura mediante transferencias de zona autenticadas con claves TSIG (Transaction Signature).

| Rol | Hostname | IPv4 | IPv6 |
|-----|----------|------|------|
| DNS Primario | dns-primary | 192.168.20.10 | 2001:db8:20::10 |
| DNS Secundario | dns-secondary | 192.168.20.11 | 2001:db8:20::11 |

### Características Implementadas

* **Zona Directa:** `plataformas.local` con registros A y AAAA para hosts de infraestructura
* **Zonas Inversas IPv4:** `192.168.10.0/24` y `192.168.20.0/24`
* **Zonas Inversas IPv6:** `2001:db8:10::/64` y `2001:db8:20::/64`
* **DNS64:** Síntesis de registros AAAA desde registros A (prefijo `2001:db8:64::/96`) para facilitar conectividad de clientes IPv6-only
* **Seguridad:** Transferencias de zona restringidas mediante autenticación TSIG y ACLs limitadas a redes locales
* **Forwarding:** Consultas externas reenviadas a resolvers públicos (Google DNS: 8.8.8.8, 8.8.4.4, 2001:4860:4860::8888)
* **Replicación Automática:** El servidor secundario recibe actualizaciones de zonas automáticamente vía AXFR

---

## Tecnologías Utilizadas

* **BIND9:** Servidor DNS autoritativo (versión 9.18.39)
* **Vagrant 2.2+:** Gestión de máquinas virtuales
* **VirtualBox 6.0+:** Hypervisor de virtualización
* **Ansible 2.9+:** Automatización de configuración
* **Ubuntu 22.04 LTS:** Sistema operativo base
* **Netplan:** Configuración de red
* **UFW:** Firewall
* **TSIG (hmac-sha256):** Autenticación de transferencias de zona

---

## Prerrequisitos

* **Vagrant:** 2.2 o superior
* **VirtualBox:** 6.0 o superior
* **Ansible:** 2.9 o superior (si se ejecuta manualmente)

---

## Estructura del Proyecto

```
primary-and-secondary-dns-server/
├── Vagrantfile                 # Definición de infraestructura virtual
├── ansible.cfg                 # Configuración de Ansible
├── deploy.yml                  # Playbook principal
├── inventory.ini               # Inventario de hosts
├── templates/
│   └── tsig.key.j2             # Clave TSIG compartida
└── roles/
    ├── dns-primary/            # Rol del servidor maestro
    │   ├── tasks/              # Tareas de instalación y configuración
    │   ├── templates/          # Plantillas de zonas y configuración
    │   │   ├── named.conf.options.j2
    │   │   ├── named.conf.local.j2
    │   │   ├── db.plataformas.local.j2
    │   │   ├── db.192.168.10.j2
    │   │   ├── db.192.168.20.j2
    │   │   ├── db.2001.db8.0010.ip6.arpa.j2
    │   │   ├── db.2001.db8.0020.ip6.arpa.j2
    │   │   ├── db.2001.db8.0064.ip6.arpa.j2
    │   │   └── netplan.yaml.j2
    │   └── handlers/           # Reinicio de servicios
    └── dns-secondary/          # Rol del servidor esclavo
        ├── tasks/              # Tareas de configuración
        ├── templates/          # Plantillas de configuración
        │   ├── named.conf.options.j2
        │   ├── named.conf.local.j2
        │   └── netplan.yaml.j2
        └── handlers/           # Reinicio de servicios
```

---

## Despliegue

### 1. Iniciar las Máquinas Virtuales

```bash
cd primary-and-secondary-dns-server
vagrant up
```

Este comando crea y configura las dos máquinas virtuales definidas en el `Vagrantfile`.

### 2. Ejecutar el Aprovisionamiento con Ansible

```bash
ansible-playbook -i inventory.ini deploy.yml
```

El playbook realiza las siguientes operaciones:

1. Instala BIND9 y utilidades asociadas
2. Configura direcciones IPv6 estáticas mediante Netplan
3. Genera y distribuye la clave TSIG compartida
4. Crea archivos de zona a partir de plantillas Jinja2
5. Configura opciones de BIND (forwarders, DNS64, ACLs)
6. Valida sintaxis de zonas con `named-checkzone`
7. Configura reglas de firewall (UFW)
8. Inicia y habilita el servicio BIND9

---

## Verificación y Pruebas

### 1. Validación de Sintaxis de Zonas (DNS Primario)

Conectarse al servidor primario y verificar la sintaxis de todas las zonas:

```bash
vagrant ssh dns-primary
```

**Zona directa:**
```bash
sudo named-checkzone plataformas.local /etc/bind/zones/db.plataformas.local
# Salida esperada: zone plataformas.local/IN: loaded serial 2025103001
# OK
```

**Zonas inversas IPv4:**
```bash
sudo named-checkzone 10.168.192.in-addr.arpa /etc/bind/zones/db.192.168.10
# Salida esperada: OK

sudo named-checkzone 20.168.192.in-addr.arpa /etc/bind/zones/db.192.168.20
# Salida esperada: OK
```

**Zonas inversas IPv6:**
```bash
sudo named-checkzone 0.0.0.0.0.1.0.0.8.b.d.0.1.0.0.2.ip6.arpa /etc/bind/zones/db.2001.db8.0010.ip6.arpa
# Salida esperada: OK

sudo named-checkzone 0.0.0.0.0.2.0.0.8.b.d.0.1.0.0.2.ip6.arpa /etc/bind/zones/db.2001.db8.0020.ip6.arpa
# Salida esperada: OK

sudo named-checkzone 0.0.0.0.4.6.0.0.8.b.d.0.1.0.0.2.ip6.arpa /etc/bind/zones/db.2001.db8.0064.ip6.arpa
# Salida esperada: OK
```

### 2. Transferencia de Zonas (AXFR)

Verificar que el servidor primario permite transferencias de zona autenticadas con TSIG:

**Zona directa (plataformas.local):**
```bash
sudo dig @127.0.0.1 -k /etc/bind/tsig.key plataformas.local AXFR
```

Salida esperada (extracto):
```
plataformas.local.      604800  IN      SOA     ns1.plataformas.local. admin.plataformas.local. 2025103001 3600 1800 604800 86400
plataformas.local.      604800  IN      NS      ns1.plataformas.local.
plataformas.local.      604800  IN      NS      ns2.plataformas.local.
ns1.plataformas.local.  604800  IN      A       192.168.20.10
ns1.plataformas.local.  604800  IN      AAAA    2001:db8:20::10
ns2.plataformas.local.  604800  IN      A       192.168.20.11
ns2.plataformas.local.  604800  IN      AAAA    2001:db8:20::11
smtp.plataformas.local. 604800  IN      A       192.168.20.20
smtp.plataformas.local. 604800  IN      AAAA    2001:db8:20::20
srv1.plataformas.local. 604800  IN      A       192.168.20.2
srv1.plataformas.local. 604800  IN      AAAA    2001:db8:20::2
srv2.plataformas.local. 604800  IN      A       192.168.20.3
srv2.plataformas.local. 604800  IN      AAAA    2001:db8:20::3
www.plataformas.local.  604800  IN      A       192.168.20.2
www.plataformas.local.  604800  IN      AAAA    2001:db8:20::2
...
;; XFR size: 20 records (messages 1, bytes 656)
```

**Zona inversa IPv4 (20.168.192.in-addr.arpa):**
```bash
sudo dig @127.0.0.1 -k /etc/bind/tsig.key 20.168.192.in-addr.arpa AXFR
```

Salida esperada (extracto):
```
10.20.168.192.in-addr.arpa. 604800 IN   PTR     ns1.plataformas.local.
11.20.168.192.in-addr.arpa. 604800 IN   PTR     ns2.plataformas.local.
2.20.168.192.in-addr.arpa. 604800 IN    PTR     srv1.plataformas.local.
20.20.168.192.in-addr.arpa. 604800 IN   PTR     smtp.plataformas.local.
3.20.168.192.in-addr.arpa. 604800 IN    PTR     srv2.plataformas.local.
...
;; XFR size: 9 records (messages 1, bytes 394)
```

**Zona inversa IPv6 (0.0.0.0.0.2.0.0.8.b.d.0.1.0.0.2.ip6.arpa):**
```bash
sudo dig @127.0.0.1 -k /etc/bind/tsig.key 0.0.0.0.0.2.0.0.8.b.d.0.1.0.0.2.ip6.arpa AXFR
```

Salida esperada (extracto):
```
0.1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.2.0.0.8.b.d.0.1.0.0.2.ip6.arpa. 604800 IN PTR ns1.plataformas.local.
1.1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.2.0.0.8.b.d.0.1.0.0.2.ip6.arpa. 604800 IN PTR ns2.plataformas.local.
2.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.2.0.0.8.b.d.0.1.0.0.2.ip6.arpa. 604800 IN PTR srv1.plataformas.local.
3.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.2.0.0.8.b.d.0.1.0.0.2.ip6.arpa. 604800 IN PTR srv2.plataformas.local.
0.2.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.2.0.0.8.b.d.0.1.0.0.2.ip6.arpa. 604800 IN PTR smtp.plataformas.local.
...
;; XFR size: 9 records (messages 1, bytes 618)
```

### 3. Verificación de Replicación en el Servidor Secundario

Conectarse al servidor secundario y verificar que las zonas fueron transferidas:

```bash
vagrant ssh dns-secondary
ls -l /var/cache/bind/zones/
```

Salida esperada:
```
total 24
-rw-r--r-- 1 bind bind  309 Nov 25 04:51 db.192.168.10
-rw-r--r-- 1 bind bind  600 Nov 25 04:51 db.192.168.20
-rw-r--r-- 1 bind bind  390 Nov 25 04:51 db.2001.db8.0010.ip6.arpa
-rw-r--r-- 1 bind bind  866 Nov 25 04:51 db.2001.db8.0020.ip6.arpa
-rw-r--r-- 1 bind bind  268 Nov 25 04:51 db.2001.db8.0064.ip6.arpa
-rw-r--r-- 1 bind bind 1116 Nov 25 04:51 db.plataformas.local
```

### 4. Resolución de Consultas desde el Servidor Secundario

Verificar que el servidor secundario resuelve consultas locales y externas:

**Consultas locales (IPv4 e IPv6):**
```bash
dig @localhost ns1.plataformas.local A +short
# Salida esperada: 192.168.20.10

dig @localhost ns1.plataformas.local AAAA +short
# Salida esperada: 2001:db8:20::10
```

**Consulta externa con síntesis DNS64:**
```bash
dig @localhost ipv4.google.com AAAA +short
# Salida esperada: ipv4.l.google.com.
#                  2001:db8:64::acd9:1d0e
```

La segunda dirección IPv6 mostrada corresponde a la dirección IPv4 sintetizada mediante el prefijo DNS64 (`2001:db8:64::/96`).

### 5. Monitoreo y Estadísticas del Servidor

**Generar y consultar estadísticas:**
```bash
sudo rndc stats
sudo cat /var/cache/bind/named.stats
```

Las estadísticas incluyen métricas de consultas entrantes y salientes, respuestas por tipo, resoluciones recursivas, uso de caché, transferencias de zona completadas y conexiones activas.

---

## Configuración de Clientes

Los clientes ubicados en las redes autorizadas (`192.168.0.0/16` o `2001:db8::/32`) pueden utilizar la infraestructura DNS configurando sus interfaces de red con los siguientes servidores:

* **DNS Primario:** `192.168.20.10` (IPv4) o `2001:db8:20::10` (IPv6)
* **DNS Secundario:** `192.168.20.11` (IPv4) o `2001:db8:20::11` (IPv6)

Esta configuración permite a los clientes resolver el dominio `plataformas.local` y acceder a Internet utilizando los forwarders configurados.
