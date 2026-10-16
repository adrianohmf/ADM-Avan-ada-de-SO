[← Anterior: Semana 14 — Pipeline completo e lançamento do projeto](../unidade_2/05_semana_14_pipeline_completo.md) | [Índice](../README.md) | [Próximo: Semana 18 — Apresentação final do projeto →](../unidade_2/07_semana_18_apresentacao_final.md)

# Semanas 15 a 17 — Acompanhamento do projeto

> Estas semanas não têm conteúdo novo de teoria — são dedicadas à orientação e ao desenvolvimento do projeto final em grupo, com um checkpoint de entrega a cada semana.

## Semana 15 — Checkpoint 1: Containerização

**O que deve estar pronto:**
- `Dockerfile` da aplicação, funcional (`docker build` sem erros).
- `docker-compose.yml` subindo a aplicação e suas dependências (banco de dados, proxy, etc.).
- Dados persistindo corretamente via volumes.

**Perguntas para se fazer no checkpoint:**
- A imagem está usando boas práticas (imagem-base enxuta, `.dockerignore`, cache de camadas)?
- Os serviços conseguem se comunicar pela rede interna do Compose?

## Semana 16 — Checkpoint 2: Pipeline de CI/CD

**O que deve estar pronto:**
- Workflow de GitHub Actions rodando a cada `push`.
- Build e testes automatizados.
- Build e push da imagem Docker para um registry.

**Perguntas para se fazer no checkpoint:**
- O workflow falha corretamente quando algo está errado (e não silenciosamente)?
- A imagem publicada no registry é a mesma usada no `docker-compose.yml`?

## Semana 17 — Checkpoint 3: Deploy e documentação

**O que deve estar pronto:**
- Passo de deploy configurado no workflow (automatizado ou simulado).
- `README.md` do repositório documentando como rodar o projeto do zero.
- Histórico de commits organizado, refletindo a evolução do trabalho do grupo.

**Perguntas para se fazer no checkpoint:**
- Alguém de fora do grupo conseguiria rodar o projeto só lendo o `README.md`?
- A documentação cobre também como configurar variáveis de ambiente/segredos necessários?

---

[← Anterior: Semana 14 — Pipeline completo e lançamento do projeto](../unidade_2/05_semana_14_pipeline_completo.md) | [Índice](../README.md) | [Próximo: Semana 18 — Apresentação final do projeto →](../unidade_2/07_semana_18_apresentacao_final.md)
