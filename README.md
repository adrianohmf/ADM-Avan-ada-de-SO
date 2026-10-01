# Administração Avançada de Sistemas Operacionais
### Unidade 2 — Docker + CI/CD (Semanas 10 a 18)

> Material de apoio da disciplina, com teoria, comandos, imagens e exercícios.
> **Ambiente considerado: WSL2 (Windows Subsystem for Linux) com Ubuntu.**
> Este material dá continuidade ao da Unidade 1 (Semanas 2 a 8) — a mesma aplicação que vocês configuraram manualmente em um servidor Linux com Nginx vai, agora, ser containerizada e ganhar uma pipeline de entrega automatizada.

---

## Sumário

- [Antes de começar: Docker no WSL2](#antes-de-começar-docker-no-wsl2)
- [Semana 10 — Introdução a containers e Docker](#semana-10--introdução-a-containers-e-docker)
- [Semana 11 — Docker avançado](#semana-11--docker-avançado)
- [Semana 12 — Docker Compose](#semana-12--docker-compose)
- [Semana 13 — Introdução a CI/CD e GitHub Actions](#semana-13--introdução-a-cicd-e-github-actions)
- [Semana 14 — Pipeline completo e lançamento do projeto](#semana-14--pipeline-completo-e-lançamento-do-projeto)
- [Semanas 15 a 17 — Acompanhamento do projeto](#semanas-15-a-17--acompanhamento-do-projeto)
- [Semana 18 — Apresentação final do projeto](#semana-18--apresentação-final-do-projeto)
- [Checklist geral (Semanas 10 a 18)](#checklist-geral-semanas-10-a-18)

---

## Antes de começar: Docker no WSL2

1. **Instale o Docker Engine diretamente no WSL2** (não é obrigatório usar o Docker Desktop do Windows):
   ```bash
   curl -fsSL https://get.docker.com -o get-docker.sh
   sudo sh get-docker.sh
   ```
2. **Adicione seu usuário ao grupo `docker`**, para não precisar de `sudo` em todo comando:
   ```bash
   sudo usermod -aG docker $USER
   ```
   Feche e abra o terminal do WSL2 de novo para o grupo ser aplicado.
3. **Inicie o serviço do Docker** (já que vocês habilitaram o systemd na Semana 2):
   ```bash
   sudo systemctl enable --now docker
   sudo systemctl status docker
   ```
4. **Confirme a instalação:**
   ```bash
   docker --version
   docker run hello-world
   ```
   Se aparecer uma mensagem de boas-vindas do Docker, está tudo certo.
5. **Rede**: assim como o Nginx da Unidade 1, containers no WSL2 expõem portas para `localhost` automaticamente — ao publicar a porta 3000 de um container, ele já fica acessível em `http://localhost:3000` no navegador do Windows.

---

## Semana 10 — Introdução a containers e Docker

### Teoria

Um **container** empacota uma aplicação junto com tudo que ela precisa para rodar (código, dependências, bibliotecas), de forma isolada, leve e portátil. Diferente de uma máquina virtual, que virtualiza um computador inteiro (incluindo um sistema operacional completo), o container compartilha o kernel do sistema operacional host — o que o torna muito mais leve e rápido de iniciar.

![Máquinas virtuais comparadas a containers](images/vm_vs_container.png)

O **Docker** é a ferramenta mais usada para criar, rodar e gerenciar containers. Sua arquitetura básica:

![Arquitetura do Docker](images/docker_architecture.png)

- **Docker CLI** — a ferramenta de linha de comando (`docker ...`) que você usa para dar comandos.
- **Docker Daemon (`dockerd`)** — o processo que realmente gerencia containers, imagens, redes e volumes em segundo plano.
- **Imagem** — um "molde" somente leitura, com tudo que a aplicação precisa (sistema de arquivos, dependências, comando de inicialização).
- **Container** — uma instância em execução de uma imagem.
- **Registry** — um repositório de imagens (ex.: Docker Hub, GitHub Container Registry), de onde imagens são baixadas (`pull`) ou enviadas (`push`).

### Comandos essenciais

```bash
# Baixar uma imagem do Docker Hub
docker pull nginx

# Listar imagens baixadas localmente
docker images

# Rodar um container a partir de uma imagem
docker run nginx

# Rodar em segundo plano (-d) e mapear a porta 8080 do host para a 80 do container
docker run -d -p 8080:80 --name meu-nginx nginx

# Listar containers em execução
docker ps

# Listar todos os containers (inclusive parados)
docker ps -a

# Ver logs de um container
docker logs meu-nginx

# Parar e remover um container
docker stop meu-nginx
docker rm meu-nginx

# Entrar no terminal de um container em execução
docker exec -it meu-nginx bash
```

Acesse `http://localhost:8080` no navegador do Windows — deve aparecer a página padrão do Nginx, agora rodando dentro de um container.

### Criando sua primeira imagem (Dockerfile)

Um `Dockerfile` descreve, passo a passo, como construir uma imagem. Vamos containerizar a mesma aplicação simples usada na Unidade 1:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY . .

EXPOSE 8000

CMD ["python3", "-m", "http.server", "8000"]
```

- **`FROM`** — a imagem-base sobre a qual a sua é construída (aqui, Python já instalado).
- **`WORKDIR`** — define o diretório de trabalho dentro do container.
- **`COPY`** — copia arquivos do seu computador para dentro da imagem.
- **`EXPOSE`** — documenta qual porta a aplicação usa (não publica a porta sozinho — isso é feito no `docker run -p`).
- **`CMD`** — o comando executado quando o container inicia.

Construa e rode:

```bash
# Construir a imagem a partir do Dockerfile na pasta atual
docker build -t minha-app:1.0 .

# Rodar a imagem construída
docker run -d -p 8000:8000 --name minha-app minha-app:1.0

# Testar
curl http://localhost:8000
```

### Exercícios

1. Baixe e rode a imagem oficial do `nginx`, mapeando para a porta 8080, e acesse pelo navegador.
2. Containerize a aplicação Python simples (ou a página estática da Semana 6) usando o `Dockerfile` acima.
3. Use `docker exec -it` para entrar no container rodando e explorar seu sistema de arquivos com `ls`.
4. Desafio: pesquise a diferença entre `docker stop` e `docker kill`, e explique quando usar cada um.

---

## Semana 11 — Docker avançado

### Teoria

Containers são, por natureza, **efêmeros**: ao serem removidos, tudo que foi escrito dentro deles se perde. Para dados que precisam persistir (como os de um banco de dados), o Docker oferece **volumes**:

![Volumes do Docker](images/docker_volumes.png)

Além disso, containers se comunicam entre si através de **redes Docker**, e imagens mais eficientes são construídas com a técnica de **multi-stage build**, que separa a etapa de compilação da etapa final de execução, reduzindo bastante o tamanho da imagem.

### Comandos essenciais — volumes

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

### Comandos essenciais — redes

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

### Multi-stage build

Uma imagem construída direto com as ferramentas de compilação (compiladores, gerenciadores de pacote completos) costuma ficar desnecessariamente grande. O **multi-stage build** usa uma etapa só para compilar, e copia apenas o resultado final para a imagem que efetivamente vai rodar:

```dockerfile
# Etapa 1: build (imagem maior, com ferramentas de compilação)
FROM node:20 AS build
WORKDIR /app
COPY . .
RUN npm install && npm run build

# Etapa 2: imagem final (muito mais enxuta)
FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
```

A imagem final não carrega o Node.js nem as dependências de build — só o resultado (os arquivos estáticos), servidos por um Nginx enxuto (`alpine`).

### Boas práticas de Dockerfile

- Prefira imagens-base **`slim`** ou **`alpine`** (bem menores que as imagens completas).
- Use um arquivo **`.dockerignore`** para não copiar arquivos desnecessários (`.git`, `node_modules`, etc.) para dentro da imagem.
- Combine comandos `RUN` relacionados em uma única linha (com `&&`) para reduzir o número de camadas da imagem.
- Copie primeiro os arquivos de dependências (ex.: `package.json`) e só depois o restante do código — isso aproveita o cache do Docker quando só o código muda, sem precisar reinstalar dependências.

### Exercícios

1. Crie um volume e rode um container de banco de dados (ex.: `postgres` ou `mysql`) usando-o, confirmando que os dados persistem após `docker stop`/`docker start`.
2. Crie uma rede customizada e coloque dois containers nela, confirmando que um consegue acessar o outro pelo nome (`docker exec` em um deles e `ping` ou `curl` no outro).
3. Escreva um `.dockerignore` para o projeto da Semana 10.
4. Desafio: reescreva o `Dockerfile` da Semana 10 usando multi-stage build (mesmo que de forma simulada) e compare o tamanho final da imagem com `docker images`.

---

## Semana 12 — Docker Compose

### Teoria

Gerenciar vários containers relacionados (aplicação, banco de dados, proxy reverso) manualmente via `docker run` fica rapidamente inviável — são várias redes, volumes e parâmetros para lembrar. O **Docker Compose** resolve isso descrevendo toda a stack em um único arquivo `docker-compose.yml`:

![Stack orquestrada pelo Docker Compose](images/docker_compose_stack.png)

### Comandos e configuração

Exemplo de `docker-compose.yml`, unindo a aplicação, um banco de dados e o Nginx como proxy reverso (reaproveitando o que foi configurado na Unidade 1):

```yaml
version: "3.9"

services:
  app:
    build: .
    expose:
      - "8000"
    networks:
      - rede-interna
    depends_on:
      - db

  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: senha123
      POSTGRES_DB: meubanco
    volumes:
      - dados_db:/var/lib/postgresql/data
    networks:
      - rede-interna

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - app
    networks:
      - rede-interna

networks:
  rede-interna:

volumes:
  dados_db:
```

- **`services`** — cada bloco é um container gerenciado pelo Compose.
- **`build: .`** — constrói a imagem a partir do `Dockerfile` na pasta atual, em vez de usar uma imagem pronta.
- **`depends_on`** — garante a ordem de inicialização (não espera a aplicação "estar pronta", só que o container já tenha subido).
- **`networks`** — todos os serviços na mesma rede interna conseguem se comunicar pelo nome do serviço (`db`, `app`).
- **`volumes`** — `dados_db` persiste os dados do banco entre reinícios.

Comandos principais:

```bash
# Subir toda a stack em segundo plano
docker compose up -d

# Ver o status dos serviços
docker compose ps

# Ver logs de todos os serviços (ou de um específico)
docker compose logs -f
docker compose logs -f app

# Parar e remover os containers (sem apagar os volumes)
docker compose down

# Parar e remover TAMBÉM os volumes (cuidado: apaga os dados)
docker compose down -v

# Reconstruir as imagens após uma mudança no código
docker compose up -d --build
```

### Exercícios

1. Monte um `docker-compose.yml` com a aplicação da Semana 10 e um banco de dados (`postgres` ou `mysql`), na mesma rede interna.
2. Adicione o Nginx como proxy reverso na frente da aplicação, reaproveitando os conceitos da Semana 7.
3. Confirme que os dados do banco persistem após `docker compose down` (sem `-v`) seguido de `docker compose up -d`.
4. Desafio: adicione um quarto serviço (ex.: um `redis` para cache) à stack, e explique, em texto, para que ele poderia servir numa aplicação real.

---

## Semana 13 — Introdução a CI/CD e GitHub Actions

### Teoria

**CI/CD** significa **Integração Contínua** (Continuous Integration) e **Entrega/Implantação Contínua** (Continuous Delivery/Deployment):

- **Integração Contínua**: toda vez que um código é enviado ao repositório, ele é automaticamente construído e testado — detectando problemas cedo, antes de chegarem a produção.
- **Entrega/Implantação Contínua**: o processo de levar essa mudança, já validada, até o ambiente de produção — de forma automatizada, em vez de manual.

![Pipeline de CI/CD](images/cicd_pipeline.png)

O **GitHub Actions** é a ferramenta de CI/CD integrada ao GitHub. Um workflow é definido em um arquivo YAML dentro de `.github/workflows/`, e é disparado por eventos do repositório (um `push`, um `pull request`, etc.).

### Estrutura de um workflow

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-e-testes:
    runs-on: ubuntu-latest

    steps:
      - name: Baixar o código do repositório
        uses: actions/checkout@v4

      - name: Configurar o Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Instalar dependências
        run: pip install -r requirements.txt

      - name: Rodar os testes
        run: pytest
```

- **`on`** — define os eventos que disparam o workflow (aqui, `push` ou `pull request` na branch `main`).
- **`jobs`** — um workflow pode ter um ou mais jobs, cada um rodando em uma máquina virtual própria (`runs-on`).
- **`steps`** — a sequência de passos de um job, executados em ordem.
- **`uses`** — reutiliza uma *action* pronta, feita pela comunidade ou pelo próprio GitHub (ex.: `actions/checkout` baixa o código do repositório para a máquina do workflow).
- **`run`** — executa um comando shell diretamente.

### Comandos e fluxo de trabalho

Não há um "comando" para rodar o GitHub Actions localmente — ele roda automaticamente nos servidores do GitHub sempre que o evento configurado acontece. O fluxo de trabalho típico é:

```bash
# 1. Crie a pasta e o arquivo de workflow no seu projeto
mkdir -p .github/workflows
nano .github/workflows/ci.yml

# 2. Adicione, faça commit e envie para o GitHub
git add .github/workflows/ci.yml
git commit -m "Adiciona pipeline de CI"
git push
```

Depois do `push`, acesse a aba **Actions** do repositório no GitHub para acompanhar a execução do workflow em tempo real.

### Exercícios

1. Crie um repositório no GitHub com o projeto da Semana 10 (ou 12) e adicione um workflow simples que só imprime uma mensagem (`run: echo "Rodando CI"`).
2. Adicione um step que instale as dependências do seu projeto e rode algum tipo de verificação (pode ser um teste simples, ou até um `python3 -m py_compile` validando a sintaxe do código).
3. Faça o workflow falhar de propósito (ex.: um comando inexistente) e observe, na aba Actions do GitHub, como o erro é reportado.
4. Desafio: configure o workflow para rodar também quando um `pull request` for aberto, e explique, em texto, por que isso é uma boa prática em equipes.

---

## Semana 14 — Pipeline completo e lançamento do projeto

### Teoria

Juntando tudo que foi visto até aqui, uma pipeline completa de CI/CD para uma aplicação containerizada costuma seguir estes passos: build da aplicação, testes, build da imagem Docker, push para um registry, e deploy automatizado.

### Comandos e configuração — pipeline completa

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD

on:
  push:
    branches: [main]

jobs:
  build-test-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Baixar o código
        uses: actions/checkout@v4

      - name: Login no GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build da imagem Docker
        run: docker build -t ghcr.io/${{ github.repository }}:latest .

      - name: Push da imagem para o registry
        run: docker push ghcr.io/${{ github.repository }}:latest

      - name: Deploy (exemplo simplificado)
        run: echo "Aqui entraria o passo de deploy, ex.: via SSH ou docker compose no servidor"
```

- **`secrets.GITHUB_TOKEN`** — um token gerado automaticamente pelo GitHub para cada execução, usado aqui para autenticar no GitHub Container Registry (GHCR) sem precisar expor uma senha.
- **`ghcr.io/${{ github.repository }}`** — monta o caminho da imagem a partir do próprio nome do repositório.
- O passo de **deploy** varia bastante conforme o ambiente real (pode ser um `ssh` para o servidor seguido de `docker compose pull && docker compose up -d`, um `kubectl apply`, ou a chamada a uma API de um provedor de cloud).

### Lançamento do projeto final

A partir desta semana, começa o **projeto final da disciplina**: aplicar, em uma aplicação de verdade (de preferência a mesma evoluída desde a Unidade 1), tudo que foi visto na Unidade 2.

**Entrega esperada:**
- Repositório no GitHub com `Dockerfile` e `docker-compose.yml` funcionais.
- Workflow de GitHub Actions rodando build, testes e push da imagem para um registry (Docker Hub ou GHCR).
- Deploy automatizado (pode ser simplificado/simulado, dependendo do ambiente disponível).
- `README.md` documentando como rodar o projeto.

### Exercícios

1. Defina, em grupo, qual aplicação será usada no projeto final (pode reaproveitar a da Unidade 1).
2. Monte o repositório no GitHub com `Dockerfile` e `docker-compose.yml`.
3. Configure o workflow de build + push da imagem para um registry.
4. Desafio: pesquise uma forma de automatizar o passo de deploy de verdade (ex.: SSH para um servidor próprio, ou um serviço gratuito como Railway/Render), e, se possível, implemente.

---

## Semanas 15 a 17 — Acompanhamento do projeto

> Estas semanas não têm conteúdo novo de teoria — são dedicadas à orientação e ao desenvolvimento do projeto final em grupo, com um checkpoint de entrega a cada semana.

### Semana 15 — Checkpoint 1: Containerização

**O que deve estar pronto:**
- `Dockerfile` da aplicação, funcional (`docker build` sem erros).
- `docker-compose.yml` subindo a aplicação e suas dependências (banco de dados, proxy, etc.).
- Dados persistindo corretamente via volumes.

**Perguntas para se fazer no checkpoint:**
- A imagem está usando boas práticas (imagem-base enxuta, `.dockerignore`, cache de camadas)?
- Os serviços conseguem se comunicar pela rede interna do Compose?

### Semana 16 — Checkpoint 2: Pipeline de CI/CD

**O que deve estar pronto:**
- Workflow de GitHub Actions rodando a cada `push`.
- Build e testes automatizados.
- Build e push da imagem Docker para um registry.

**Perguntas para se fazer no checkpoint:**
- O workflow falha corretamente quando algo está errado (e não silenciosamente)?
- A imagem publicada no registry é a mesma usada no `docker-compose.yml`?

### Semana 17 — Checkpoint 3: Deploy e documentação

**O que deve estar pronto:**
- Passo de deploy configurado no workflow (automatizado ou simulado).
- `README.md` do repositório documentando como rodar o projeto do zero.
- Histórico de commits organizado, refletindo a evolução do trabalho do grupo.

**Perguntas para se fazer no checkpoint:**
- Alguém de fora do grupo conseguiria rodar o projeto só lendo o `README.md`?
- A documentação cobre também como configurar variáveis de ambiente/segredos necessários?

---

## Semana 18 — Apresentação final do projeto

Cada grupo apresenta o projeto desenvolvido ao longo da Unidade 2, demonstrando:

1. A aplicação containerizada rodando (`docker compose up`).
2. O pipeline de CI/CD funcionando — idealmente, uma alteração real sendo enviada ao repositório durante a apresentação, mostrando o workflow disparando automaticamente.
3. O deploy automatizado (ou simulado) resultando na aplicação atualizada.
4. A documentação do projeto.

### Critérios de avaliação

- Containerização correta da aplicação (Dockerfile e Docker Compose funcionais)
- Pipeline de CI/CD funcional (build, testes, push para registry)
- Deploy automatizado funcionando de ponta a ponta
- Qualidade da documentação (README do repositório)
- Histórico de commits/PRs como evidência do processo de desenvolvimento
- Apresentação final: clareza da demonstração e domínio técnico do grupo

---

## Checklist geral (Semanas 10 a 18)

- [ ] Docker instalado e rodando no WSL2 (com systemd)
- [ ] Primeira imagem construída a partir de um Dockerfile próprio
- [ ] Volumes testados, confirmando persistência de dados
- [ ] Rede customizada criada, com containers se comunicando pelo nome
- [ ] Multi-stage build praticado (ou ao menos compreendido)
- [ ] Stack completa (app + banco + proxy) orquestrada via Docker Compose
- [ ] Workflow de GitHub Actions criado e disparando a cada push
- [ ] Pipeline completa: build, testes, push da imagem para um registry
- [ ] Passo de deploy configurado (automatizado ou simulado)
- [ ] Projeto final documentado em um README.md completo
