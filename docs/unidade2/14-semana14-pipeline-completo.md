[← Anterior: Semana 13: Introdução a CI/CD e GitHub Actions](13-semana13-cicd-github-actions.md) | [Índice](../../README.md) | [Próximo: Projeto Final →](15-projeto-final.md)

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

A partir desta semana, começa o **projeto final da disciplina**, feito em **trios**: construir o ambiente de uma aplicação web em camadas (front, back, banco, Nginx), com Compose para desenvolvimento e produção, pipeline de CI/CD com testes, deploy preferencialmente na nuvem com **HTTPS**, e monitoramento com Prometheus e/ou Grafana.

O projeto é entregue **pelo GitHub** e vale **10 pontos**. Todos os requisitos, o que deve ter no `README.md` e a avaliação estão em [Projeto Final](15-projeto-final.md).

**Datas:** acompanhamento em **04/12** e apresentação em **13/12**.

## Exercícios

1. Forme o trio e defina qual aplicação será usada no projeto final (um CRUD simples já basta).
2. Crie o repositório no GitHub com a estrutura de camadas (front, back, banco, Nginx) e os arquivos do Compose.
3. Configure o workflow de build + push da imagem para um registry.
4. Desafio: pesquise uma forma de automatizar o passo de deploy de verdade (ex.: SSH para um servidor próprio, ou um serviço gratuito como Railway/Render), e, se possível, implemente.

---

[← Anterior: Semana 13: Introdução a CI/CD e GitHub Actions](13-semana13-cicd-github-actions.md) | [Índice](../../README.md) | [Próximo: Projeto Final →](15-projeto-final.md)
