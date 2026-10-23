[← Anterior: Semana 10: Introdução a containers e Docker](10-semana10-introducao-docker.md) | [Índice](../../README.md) | [Próximo: Semana 12: Docker Compose →](12-semana12-docker-compose.md)

# Semana 11 — Docker avançado

## Teoria

Containers são, por natureza, **efêmeros**: ao serem removidos, tudo que foi escrito dentro deles se perde. Para dados que precisam persistir (como os de um banco de dados), o Docker oferece **volumes**:

![Volumes do Docker](../../images/docker_volumes.png)

Além disso, containers se comunicam entre si através de **redes Docker**.

## Comandos essenciais — volumes

```bash
# Criar um volume nomeado
docker volume create dados_app

# Rodar um container usando esse volume
docker run -d -v dados_app:/var/lib/dados --name app-com-volume minha-app:1.0

# Listar volumes
docker volume ls

# Inspecionar onde o volume está no disco do host
docker volume inspect dados_app

# Remover um volume (o container que o usa precisa estar parado/removido)
docker volume rm dados_app
```

## Comandos essenciais — redes

```bash
# Criar uma rede customizada
docker network create minha-rede

# Rodar dois containers na mesma rede — eles conseguem se comunicar pelo nome
docker run -d --network minha-rede --name banco postgres:16
docker run -d --network minha-rede --name app minha-app:1.0

# Listar redes
docker network ls

# Inspecionar uma rede (mostra quais containers estão conectados)
docker network inspect minha-rede
```

> Dentro da rede `minha-rede`, o container `app` pode acessar o banco simplesmente usando o hostname `banco` — o Docker resolve isso internamente, sem precisar saber o IP.

## Mais instruções do Dockerfile

```dockerfile
FROM python:3.12-slim

# ARG existe só durante o build; ENV fica disponível quando o container roda
ARG VERSAO_APP=1.0
ENV APP_ENV=producao \
    PORTA=8000

WORKDIR /app
COPY . .

# Rodar como usuário comum, sem privilégios de root
RUN useradd -m appuser
USER appuser

EXPOSE 8000

# Docker verifica periodicamente se a aplicação está saudável
HEALTHCHECK --interval=30s --timeout=3s CMD python3 -c "import urllib.request; urllib.request.urlopen('http://localhost:8000')" || exit 1

# ENTRYPOINT é o executável fixo; CMD traz os argumentos padrão, que podem ser trocados no docker run
ENTRYPOINT ["python3", "-m", "http.server"]
CMD ["8000"]
```

- **`ARG` e `ENV`:** `ARG` vale só na construção (`docker build --build-arg VERSAO_APP=2.0 .`); `ENV` vale também em execução e pode ser trocada com `docker run -e`.
- **`USER`:** roda a aplicação sem ser root, reduzindo o estrago em caso de invasão (lembra do alerta sobre o kernel compartilhado na Semana 10).
- **`HEALTHCHECK`:** o `docker ps` passa a mostrar `healthy` ou `unhealthy`, e orquestradores usam isso para reiniciar o serviço.
- **`ENTRYPOINT` e `CMD`:** com a forma acima, `docker run minha-app:2.0 9000` troca só a porta, mantendo o executável.

> As boas práticas gerais de Dockerfile (imagens `slim`, `.dockerignore`, ordem das camadas) estão na Semana 10 e continuam valendo aqui.

## Exercícios

1. Crie um volume e rode um container de banco de dados (ex.: `postgres` ou `mysql`) usando-o, confirmando que os dados persistem após `docker stop`/`docker start`.
2. Crie uma rede customizada e coloque dois containers nela, confirmando que um consegue acessar o outro pelo nome (`docker exec` em um deles e `ping` ou `curl` no outro).
3. Adicione `USER`, `ENV` e `HEALTHCHECK` ao Dockerfile da Semana 10 e observe o estado `healthy` no `docker ps`.

---

[← Anterior: Semana 10: Introdução a containers e Docker](10-semana10-introducao-docker.md) | [Índice](../../README.md) | [Próximo: Semana 12: Docker Compose →](12-semana12-docker-compose.md)
