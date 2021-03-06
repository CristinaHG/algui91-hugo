---
author: alex
categories:
- servidores
mainclass: servidores
date: '2016-01-01'
lastmod: 2017-09-24T19:10:45+01:00
description: "Cuando se administra un servidor, te das cuenta de la cantidad de máquinas  automatizadas que existen realizando ataques de fuerza bruta hacia tu servidor.  Para poner fin a algunos de estos ataques existe una herramienta llamada Fail2Ban,  que monitoriza los logs del sistema para detectar estos ataques y mitigarlos. En  este artículos veremos cómo configurar Fail2Ban para bloquear el acceso a  nuestra máquina a robots atacando por fuerza bruta a WordPress y al servidor  web Nginx"
image: 2013/11/Bloquear-ataques-de-fuerza-bruta-en-Nginx-y-Wordpress-con-Fail2Ban2.png
url: /bloquear-ataques-de-fuerza-bruta-en-nginx-y-wordpress-con-fail2ban/
aliases: /administracion-de-servidores/bloquear-ataques-de-fuerza-bruta-en-nginx-y-wordpress-con-fail2ban/
tags:
- fail2ban
- nginx
title: Bloquear ataques de fuerza bruta en Nginx y WordPress con Fail2Ban
---

<figure>
    <img sizes="(min-width: 600px) 600px, 100vw" on="tap:lightbox1" role="button" tabindex="0" layout="responsive" src="/img/2013/11/Bloquear-ataques-de-fuerza-bruta-en-Nginx-y-Wordpress-con-Fail2Ban2.png" title="Bloquear ataques de fuerza bruta en Nginx y WordPress con Fail2Ban" alt="Bloquear ataques de fuerza bruta en Nginx y WordPress con Fail2Ban" width="600px" height="600px"></img>
</figure>

Cuando se administra un servidor, te das cuenta de la cantidad de máquinas automatizadas que existen realizando ataques de fuerza bruta hacia tu servidor. Para poner fin a algunos de estos ataques existe una herramienta llamada ***Fail2Ban***, que monitoriza los logs del sistema para detectar estos ataques y mitigarlos. En este artículos veremos cómo configurar **Fail2Ban** para bloquear el acceso a nuestra máquina a robots atacando por [fuerza bruta][1] a *WordPress* y al servidor web *[Nginx][2]*

<!--more--><!--ad-->

# Instalar Fail2Ban

Si no se encuentra instalado en nuestro sistema:

```bash
$ sudo apt-get install fail2ban

```

# Antes de empezar

Antes de modificar el archivo */etc/fail2ban/jail.conf*, es recomendable -y de hecho lo menciona el propio archivo en su cabecera &#8212; realizar una copia y trabajar sobre ella:

```bash
cd /etc/fail2ban && cp jail.conf jail.local

```

# Bloquear ataques de fuerza bruta a WordPress

La siguiente configuración bloqueará durante 20 minutos cualquier intento de loggearse en WordPress de forma incorrecta más de 3 veces. Hay que escribirla en el *jail.local*:

```bash
[nginx-wp-login]

enabled = true
port = http,https
filter = nginx-wp-login
logpath = /ruta/log/wordpress/access.log
maxretry = 3
findtime = 120
bantime = 1200

```

Ahora queda añadir el filtro para esta regla, en el archivo *filter.d/nginx-wp-login.conf*:

```bash
[Definition]

failregex = <host>.*] "POST /wp-login.php
ignoreregex =

```

# Inconvenientes a tener en cuenta

Con esta configuración, hay que considerar que:

  * Los usuarios que no recuerden su contraseña se autobloquearán.
  * No te protegerá eficientemente de un ataque distribuido.

## Configurar Nginx con Fail2Ban

Las siguientes configuraciones serán a nivel del servidor web nginx, no de WordPress, los objetivos son:

  * Bloquear a todo aquel intentando ejecutar scripts (.pl, .cgi, .exe etc).
  * Bloquear a todo aquel intentando usar el servidor como un proxy.
  * Bloquear a todo aquel que falle al usar la autentificación básica de nginx.
  * Bloquear a todo aquel que falle al autentificarse en nuestra aplicación.
  * Bloquear **bad bots**.

Al igual que antes, en el fichero *jail.local* añadimos:

```bash
[nginx-auth]
enabled = true
filter = nginx-auth
action = iptables-multiport[name=NoAuthFailures, port="http,https"]
logpath = /var/log/nginx*/*error*.log
bantime = 600 # 10 minutes
maxretry = 6

[nginx-login]
enabled = true
filter = nginx-login
action = iptables-multiport[name=NoLoginFailures, port="http,https"]
logpath = /var/log/nginx*/*access*.log
bantime = 600 # 10 minutes
maxretry = 6

[nginx-badbots]
enabled  = true
filter = apache-badbots
action = iptables-multiport[name=BadBots, port="http,https"]
logpath = /var/log/nginx*/*access*.log
bantime = 86400 # 1 day
maxretry = 1

[nginx-noscript]
enabled = true
action = iptables-multiport[name=NoScript, port="http,https"]
filter = nginx-noscript
logpath = /var/log/nginx*/*access*.log
maxretry = 6
bantime  = 86400 # 1 day

[nginx-proxy]
enabled = true
action = iptables-multiport[name=NoProxy, port="http,https"]
filter = nginx-proxy
logpath = /var/log/nginx*/*access*.log
maxretry = 0
bantime  = 86400 # 1 day

```

Y sus correspondientes filtros en */etc/fail2ban/filter.d/* (Cada uno en un fichero separado, con el mismo nombre que aparece en la primera línea):

```bash
# Proxy filter /etc/fail2ban/filter.d/proxy.conf:
#
# Block IPs trying to use server as proxy.
#
# Matches e.g.
# 192.168.1.1 - - "GET http://www.something.com/
#
[Definition]
failregex = ^<host> -.*GET http.*
ignoreregex =

# Noscript filter /etc/fail2ban/filter.d/nginx-noscript.conf:
#
# Block IPs trying to execute scripts such as .php, .pl, .exe and other funny scripts.
#
# Matches e.g.
# 192.168.1.1 - - "GET /something.php
#
[Definition]
failregex = ^<host> -.*GET.*(\.php|\.asp|\.exe|\.pl|\.cgi|\scgi)
ignoreregex =

#
# Auth filter /etc/fail2ban/filter.d/nginx-auth.conf:
#
# Blocks IPs that fail to authenticate using basic authentication
#
[Definition]

failregex = no user/password was provided for basic authentication.*client: <host>
            user .* was not found in.*client: <host>
            user .* password mismatch.*client: <host>

ignoreregex =

#
# Login filter /etc/fail2ban/filter.d/nginx-login.conf:
#
# Blocks IPs that fail to authenticate using web application's log in page
#
# Scan access log for HTTP 200 + POST /sessions => failed log in
[Definition]
failregex = ^<host> -.*POST /sessions HTTP/1\.." 200
ignoreregex =

```

# Enviar por correo los bloqueos

Para terminar, si queremos recibir un correo por cada bloqueo que se produzca, basta con añadir estas dos líneas al fichero *jail.local*:

```bash
destemail = direccion@correo
mta = sendmail

```

¿Administras un servidor? ¿Qué técnicas usas?, deja un comentario.

# Referencias

- *Fail2Ban para wordpress* »» <a href="http://codepoets.co.uk/2013/fail2ban-filter-for-wordpress/" target="_blank">codepoets.co.uk</a>
- *Fail2Ban para Nginx* »» <a href="http://snippets.aktagon.com/snippets/554-how-to-secure-an-nginx-server-with-fail2ban" target="_blank">snippets.aktagon.com</a>

 [1]: https://elbauldelprogramador.com/bloquear-una-ip-atacanto-el-servidor-mediante-iptables/ "Bloquear una IP atacando el servidor mediante iptables"
 [2]: https://elbauldelprogramador.com/como-instalar-nginx-con-php5-fpm/ "Cómo instalar y configurar Nginx con php5-fpm"
