# jitsi-docker

Requisitos mínimos (VPS)

Ubuntu 24.04 LTS

CPU: 2 vCPU mínimo (4 vCPU recomendado)

RAM: 4 GB mínimo (8 GB recomendado)

Disco: 25–40 GB libres (según grabaciones/crecimiento de logs)

Docker + Docker Compose plugin

Dominio apuntando al NPM (Nginx Proxy Manager) y el NPM debe poder llegar al VPS por HTTP

IP pública del VPS (para JVB_ADVERTISE_IPS)

Importante: la media (audio/video) viaja directo al VPS por UDP 10000 (no pasa por NPM).

Puertos a abrir (UFW)

En el VPS donde corre Jitsi:

sudo ufw allow OpenSSH
sudo ufw allow 10000/udp
sudo ufw allow 4443/tcp
sudo ufw allow from 34.118.175.190 to any port 8080 proto tcp
sudo ufw enable
sudo ufw status


8080/tcp solo desde el VPS del NPM (34.118.175.190)

10000/udp (media WebRTC)

4443/tcp fallback (útil si el UDP se bloquea en alguna red)

Instalación / Deploy

En el VPS:

sudo mkdir -p /opt/jitsi
cd /opt/jitsi

# copiar docker-compose.yml y .env desde tu repo a /opt/jitsi
# (o git clone + copy)

docker compose up -d
docker compose ps

Variables importantes (.env)

DOMAIN=tu-dominio.com (el que apunta a NPM)

SCHEME=https (si NPM entrega HTTPS)

JVB_ADVERTISE_IPS=IP_PUBLICA_DEL_VPS_JITSI

Passwords XMPP obligatorias:

JVB_AUTH_PASSWORD

JICOFO_AUTH_PASSWORD

JIBRI_XMPP_PASSWORD

JIGASI_XMPP_PASSWORD

JIBRI_RECORDER_PASSWORD

Configuración en NPM (Reverse Proxy)

En el Proxy Host del dominio:

Forward Hostname/IP: IP del VPS de Jitsi

Forward Port: 8080

Websockets: ON

SSL: el que uses (Let’s Encrypt recomendado)

En “Custom Nginx Configuration” agregá:

location /xmpp-websocket {
    proxy_pass http://IP_DEL_VPS_JITSI:8080/xmpp-websocket;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}

location /colibri-ws {
    proxy_pass http://IP_DEL_VPS_JITSI:8080/colibri-ws;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}

location / {
    proxy_pass http://IP_DEL_VPS_JITSI:8080/;
    proxy_http_version 1.1;
}

Uso

Abrí en el navegador:

https://TU_DOMINIO/NOMBRE_DE_SALA

Ejemplo:

https://stacka.gpowstack.com/prueba

Comandos útiles
cd /opt/jitsi

docker compose logs -f web
docker compose logs -f prosody
docker compose logs -f jvb

docker compose restart
docker compose down
docker compose up -d

