[← Anterior: Semana 13: Introdução a CI/CD e GitHub Actions](13-semana13-cicd-github-actions.md) | [Índice](../../README.md) | [Próximo: Semanas 15 a 17: Acompanhamento do projeto →](15-semanas15-17-acompanhamento.md)

# Semana 14 — Pipeline completo e lançamento do projeto

## Teoria

Juntando tudo que foi visto até aqui, uma pipeline completa de CI/CD para uma aplicação containerizada costuma seguir estes passos: build da aplicação, testes, build da imagem Docker, push para um registry, e deploy automatizado.

## Comandos e configuração — pipeline completa

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

## Lançamento do projeto final

A partir desta semana, começa o **projeto final da disciplina**: aplicar, em uma aplicação de verdade (de preferência a mesma evoluída desde a Unidade 1), tudo que foi visto na Unidade 2.

**Entrega esperada:**
- Repositório no GitHub com `Dockerfile` e `docker-compose.yml` funcionais.
- Workflow de GitHub Actions rodando build, testes e push da imagem para um registry (Docker Hub ou GHCR).
- Deploy automatizado (pode ser simplificado/simulado, dependendo do ambiente disponível).
- `README.md` documentando como rodar o projeto.

## Exercícios

1. Defina, em grupo, qual aplicação será usada no projeto final (pode reaproveitar a da Unidade 1).
2. Monte o repositório no GitHub com `Dockerfile` e `docker-compose.yml`.
3. Configure o workflow de build + push da imagem para um registry.
4. Desafio: pesquise uma forma de automatizar o passo de deploy de verdade (ex.: SSH para um servidor próprio, ou um serviço gratuito como Railway/Render), e, se possível, implemente.

---

[← Anterior: Semana 13: Introdução a CI/CD e GitHub Actions](13-semana13-cicd-github-actions.md) | [Índice](../../README.md) | [Próximo: Semanas 15 a 17: Acompanhamento do projeto →](15-semanas15-17-acompanhamento.md)
