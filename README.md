# sama-docker-backup

Infraestrutura **Docker Compose** do projeto [SAMA Envios / whastappsender](https://github.com/marcelnagm/whastappsender) (CRM Laravel com disparos via WhatsApp). Este repositório guarda o backup da configuração de produção: containers, Nginx e workers do Supervisor.

## Serviços (`docker-compose.yml`)

| Serviço | Imagem | Função |
|---|---|---|
| `app` | build local (`php:8.3-fpm`) | PHP-FPM com `pdo_mysql`, `gd`, `redis`, `opcache`, `bcmath`, `zip`; roda o Laravel |
| `nginx` | `nginx:alpine` | Servidor web e proxy reverso (SSL via Let's Encrypt montado do host) |
| `mysql` | `mysql:8.0` | Banco do Laravel |
| `redis` | `redis:7-alpine` | Filas e cache (com healthcheck) |
| `postgres` | `postgres:15-alpine` | Banco da Evolution API |
| `evolution-api` | `evoapicloud/evolution-api:v2.3.0` | Gateway WhatsApp (configurado por `.env.docker`) |
| `minio` | `minio/minio` | Armazenamento de arquivos compatível com S3 |
| `phpmyadmin` | `phpmyadmin` | Administração do MySQL (porta 8082) |

Todos compartilham a rede `evolution_network`; os dados persistem em `./data/*`.

## Outros arquivos

- `Dockerfile`: imagem PHP 8.3-FPM com as extensões necessárias, Composer e ajustes de memória/upload (512 MB / 100 MB).
- `nginx/nginx.conf`: virtual hosts para o app Laravel (443), MinIO (9000) e Evolution API (8080/8081). Há variantes `nginx.conf.ssl` e `nginx.bk.conf`.
- `supervisor/`: processos `queue:work` para as filas `disparos` e `default`, `schedule:work` e `php-fpm`.

## Como subir

```bash
git clone https://github.com/marcelnagm/sama-docker-backup
cd sama-docker-backup
mkdir -p app && git clone https://github.com/marcelnagm/whastappsender app
# ajuste .env, .env.docker, o domínio no nginx.conf e as senhas
docker compose up -d --build
```

## ⚠️ Segurança

Os arquivos `.env`, `.env.docker` e o `docker-compose.yml` deste repositório contêm valores de exemplo/senhas de desenvolvimento (MySQL, Redis, Postgres, MinIO, chave da Evolution API). **Troque todas antes de usar em produção** e não versione segredos reais; o ideal é manter apenas `.env.example`.
