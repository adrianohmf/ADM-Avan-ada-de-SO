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

## Prática: observando o `HEALTHCHECK` funcionar

Um container pode estar com o processo rodando (`Up`) e a aplicação travada ou quebrada por dentro. O `HEALTHCHECK` resolve isso: o Docker executa um teste periódico e marca o container como `healthy` ou `unhealthy`. Nesta prática você vai ver os dois estados acontecerem.

**1. Prepare uma pasta com a aplicação**

```bash
mkdir ~/app-saude
cd ~/app-saude

echo "<h1>Estou saudável</h1>" > index.html
nano Dockerfile
```

Conteúdo do `Dockerfile`:

```dockerfile
FROM python:3.12-slim

WORKDIR /app
COPY index.html .

RUN useradd -m appuser
USER appuser

EXPOSE 8000

# O teste pede a página index.html; se ela não existir (erro 404), o teste falha
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD python3 -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/index.html')" || exit 1

CMD ["python3", "-m", "http.server", "8000"]
```

Os parâmetros do `HEALTHCHECK`:

| Parâmetro | Significado |
|---|---|
| `--interval` | De quanto em quanto tempo o teste roda |
| `--timeout` | Tempo máximo para o teste responder, senão conta como falha |
| `--start-period` | Tempo de tolerância na inicialização, em que falhas ainda não contam |
| `--retries` | Quantas falhas seguidas são necessárias para virar `unhealthy` |

O comando do teste precisa terminar com código **0** (saudável) ou **1** (com problema), o mesmo conceito de *exit code* da Semana 5.

**2. Construa e rode com tempos curtos**

Para não esperar 30 segundos a cada teste, os valores podem ser trocados no `docker run`:

```bash
docker build -t app-saude:1.0 .

docker run -d -p 8001:8000 --name saude \
  --health-interval=5s --health-timeout=3s --health-retries=2 \
  app-saude:1.0
```

**3. Observe o estado saudável**

```bash
# A coluna STATUS mostra (health: starting) e depois (healthy)
docker ps

# Só o estado de saúde
docker inspect -f '{{.State.Health.Status}}' saude

# Teste no navegador ou no terminal
curl http://localhost:8001
```

**4. Quebre a aplicação de propósito**

Apague a página dentro do container. O processo do Python continua rodando, mas o teste vai começar a falhar.

```bash
docker exec -u root saude rm /app/index.html

# Aguarde cerca de 15 segundos e confira de novo
docker ps
docker inspect -f '{{.State.Health.Status}}' saude
```

O estado deve mudar para `unhealthy`. Veja o histórico dos testes, com o código de saída e a mensagem de erro de cada um:

```bash
docker inspect -f '{{range .State.Health.Log}}saida={{.ExitCode}} {{.Output}}{{"\n"}}{{end}}' saude
```

Note que o container continua com status `Up`. Só o estado de saúde mudou.

**5. Recupere a aplicação**

```bash
docker cp index.html saude:/app/index.html

# Em alguns segundos volta para healthy
docker inspect -f '{{.State.Health.Status}}' saude
```

**6. Perguntas para responder**

- O Docker reiniciou o container quando ele ficou `unhealthy`? (Dica: confira o `docker ps -a` e o tempo de atividade. Por padrão, o Docker só **sinaliza** o problema e não reinicia sozinho. Quem reage a isso são ferramentas acima dele, como o Compose, na próxima semana, e orquestradores como o Kubernetes.)
- Por que o teste usa `/index.html` e não apenas `/`? O que aconteceria com o `/` quando a página não existe?
- Por que existe o `--start-period`? Que problema ele evita em aplicações lentas para iniciar?

**7. Limpeza**

```bash
docker rm -f saude
docker rmi app-saude:1.0
```

> **Por que isso importa:** na Semana 12, o Docker Compose poderá esperar um serviço ficar `healthy` antes de iniciar outro (por exemplo, a aplicação só sobe depois que o banco de dados estiver pronto), usando `depends_on` com `condition: service_healthy`.

## Exercícios

1. Crie um volume e rode um container de banco de dados (ex.: `postgres` ou `mysql`) usando-o, confirmando que os dados persistem após `docker stop`/`docker start`.
2. Crie uma rede customizada e coloque dois containers nela, confirmando que um consegue acessar o outro pelo nome (`docker exec` em um deles e `ping` ou `curl` no outro).
3. Adicione `USER` e `ENV` ao Dockerfile da Semana 10 e confirme com `docker exec minha-app whoami` e `docker exec minha-app env`.
4. Faça a prática do `HEALTHCHECK` e depois repita, mudando o teste para consultar uma porta errada (por exemplo `localhost:9000`). Observe o container ficar `unhealthy` desde o início.

---

[← Anterior: Semana 10: Introdução a containers e Docker](10-semana10-introducao-docker.md) | [Índice](../../README.md) | [Próximo: Semana 12: Docker Compose →](12-semana12-docker-compose.md)
