# local-proxy

Traefik v2 como reverse proxy local para desenvolvimento. Substitui portas numéricas (`localhost:3000`, `localhost:8080`) por domínios `.localhost` legíveis — sem instalar nada no sistema, sem mexer em `/etc/hosts`.

## Como funciona

O Traefik escuta na porta 80 e descobre os containers automaticamente via Docker socket. Cada projeto se registra com labels no próprio `docker-compose.yml` — o proxy em si não conhece nenhum projeto.

```
browser → http://meu-projeto.localhost → Traefik (porta 80) → container:porta_interna
```

`*.localhost` resolve para `127.0.0.1` nativamente em browsers modernos. Sem DNS server, sem `/etc/hosts`.

## Setup

**1. Criar a rede compartilhada (uma vez)**
```bash
docker network create traefik-proxy
```

**2. Subir o proxy**
```bash
docker compose up -d
```

Dashboard disponível em `http://traefik.localhost:8888`.

## Registrar um projeto

No `docker-compose.yml` do projeto, adicione a rede e os labels:

```yaml
services:
  app:
    # ... resto da config
    networks:
      - default
      - traefik-proxy
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.meu-projeto.rule=Host(`meu-projeto.localhost`)"
      - "traefik.http.services.meu-projeto.loadbalancer.server.port=3000"
      - "traefik.docker.network=traefik-proxy"

networks:
  traefik-proxy:
    external: true
```

Troque `meu-projeto` pelo slug e `3000` pela porta interna do container. O projeto não precisa expor a porta no host.

## Observações

- Testado no WSL2 (Ubuntu) com Docker Desktop
- Requer parar qualquer serviço ocupando a porta 80 (`sudo systemctl stop apache2` ou nginx)
- O `docker-compose.yml` deste repo não contém nada específico dos seus projetos — cada um cuida do próprio registro
- Para HTTPS local, veja a branch `https-local` (mkcert + Traefik TLS)
