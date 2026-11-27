[← Anterior: Semana 14: Pipeline completo e lançamento do projeto](14-semana14-pipeline-completo.md) | [Índice](../../README.md) | [Próximo: Semana 18: Apresentação final do projeto →](18-semana18-apresentacao-final.md)

# Projeto Final: ambiente em camadas, do código à nuvem

**Equipe:** trios.

**Objetivo:** construir o ambiente de uma aplicação web separada em camadas, que roda igual no desenvolvimento e na produção, entregue por um pipeline com testes e publicada na nuvem com HTTPS e monitoramento.

**Valor:** o projeto todo vale **10 pontos**.

## Datas

| Data | O que acontece |
|---|---|
| **04/12** | Acompanhamento do projeto |
| **13/12** | Apresentação final |

## Entrega pelo GitHub

O projeto é entregue **pelo GitHub**. O repositório (público, ou privado com o professor convidado) deve conter:

- o código da aplicação;
- os `Dockerfile` de cada serviço;
- os arquivos do Compose (base, desenvolvimento e produção);
- os workflows do GitHub Actions;
- o `README.md` descrito no requisito R7;
- uma **tag de release `v1.0`** marcando a versão entregue.

O link do repositório é enviado ao professor **antes da apresentação**.

## A aplicação

A aplicação deve ser um **CRUD de uma aplicação web**, em **qualquer linguagem** à escolha do trio (por exemplo, tarefas, alunos ou livros). **O foco da avaliação é a infraestrutura**, e não a funcionalidade.

## Arquitetura mínima

```
Navegador (HTTPS)
   |
 Nginx (única porta pública: 80 redireciona para 443)
   |-- /          -> front
   |-- /api       -> back (API REST)
   |                  |
   |               banco (PostgreSQL, com volume)
   |-- /grafana   -> Grafana (com senha)   (desejável)
                      |
                   Prometheus -> coleta métricas da stack
```

São no mínimo 4 containers: **nginx, front, back e db**. Com o monitoramento, entram também o Prometheus e/ou o Grafana.

**Divisão sugerida no trio:** cada pessoa é dona de uma camada, mas todas precisam explicar o projeto inteiro.

- **Pessoa 1:** front e seu Dockerfile.
- **Pessoa 2:** back, banco e testes da API.
- **Pessoa 3:** Compose, Nginx, pipeline, deploy e monitoramento.

## Requisitos

### R1. Aplicação em camadas

- Front, back e banco em containers separados, conversando pela rede do Compose.
- Nginx como proxy reverso e única porta pública.
- Endpoint `/api/health` no back, também usado pelo `HEALTHCHECK`.

### R2. Dockerfiles com boas práticas

- Imagem base enxuta, com versão fixada (nada de `latest`).
- Aplicação rodando como usuário comum (`USER`).
- `.dockerignore` presente e coerente.
- `HEALTHCHECK` definido.
- O front pode ser estático ou ter o build gerado no pipeline.

### R3. Compose com dois ambientes (conteúdo da Semana 12)

- **Desenvolvimento:** código montado em volume (a alteração aparece sem reconstruir a imagem), portas de depuração abertas só no `localhost` e banco com dados de exemplo.
- **Produção:** serviços usam as imagens publicadas no GHCR (sem `build`), só o Nginx expõe porta e `restart: unless-stopped`.
- Senhas e configurações em `.env`, **fora do Git**, com um `.env.example` sem valores reais.
- `depends_on` com `condition: service_healthy` onde fizer sentido.

### R4. Pipeline de CI/CD (GitHub Actions)

- Dispara em `push` na `main` e em pull requests.
- Etapas: testes do back, lint ou build do front, build das imagens e push para o GHCR (com as tags `sha` e `latest`).
- **Teste de integração:** sobe a stack com Compose no runner e roda um teste simples (`/api/health` e uma gravação e leitura no banco pela API).
- **Deploy** somente a partir da `main`, seguido de **teste pós deploy** na URL pública.
- O pipeline **falha de verdade** quando algo está errado.
- Segredos somente em GitHub Secrets.

### R5. Deploy e acesso por HTTPS

- O acesso à aplicação deve ser por **HTTPS**, com redirecionamento automático de HTTP para HTTPS (como na Semana 8).
- O ideal é um certificado válido (Let's Encrypt), que exige um domínio ou subdomínio (existem opções gratuitas). Serviços PaaS já entregam HTTPS pronto. Sem domínio, um certificado autoassinado é aceito, com nota menor.
- Onde implantar:

| Opção | Descrição | Nota máxima do critério |
|---|---|---|
| **Nuvem com VM** (AWS, Azure ou GCP) | Servidor Linux com Docker. O workflow entra por SSH e executa `docker compose pull` e `up -d` | 100% |
| **Nuvem com PaaS** (Render, Railway ou similar) | Deploy automatizado da imagem ou do repositório | 100% |
| **Simulado** | Job que sobe a stack no runner e valida com `curl` | 70% |

- Na VM, o grupo aplica o que viu na Unidade 1: usuário com sudo, login por chave SSH, `PermitRootLogin no` e firewall liberando só as portas 22, 80 e 443.
- **Cuidados com a nuvem:**
  - confira as condições gratuitas e os créditos vigentes de cada provedor antes de começar;
  - configure alerta de gasto;
  - **destrua os recursos depois da avaliação**;
  - nunca versione chaves, tokens ou o `.env`.

### R6. Monitoramento com Prometheus e/ou Grafana (desejável)

- Prometheus coletando métricas da stack (por exemplo, do back, do Nginx ou dos containers) e/ou Grafana com pelo menos um painel útil.
- O Grafana **não pode ficar aberto sem senha**: a senha é definida por variável de ambiente (fora do Git), e o acesso é feito pelo Nginx com HTTPS ou por túnel SSH.
- É desejável, mas vale pontos: quem não o fizer fica com no máximo **9,0**.

### R7. `README.md` do projeto

O repositório deve ter, na raiz, um `README.md` que explique o projeto por completo. Ele é a porta de entrada para quem avalia e para qualquer pessoa que precise rodar o projeto. Deve conter, **nesta ordem**:

1. **Título e descrição:** o que a aplicação faz e qual problema resolve, em poucas linhas.
2. **Equipe:** nome dos três integrantes e quem foi dono de cada camada.
3. **Endereço da aplicação em HTTPS** e, se houver, do painel de monitoramento.
4. **Arquitetura:** diagrama (imagem ou Mermaid) com Nginx, front, back, banco e monitoramento, e uma tabela com a função de cada serviço.
5. **Tecnologias usadas** e a versão de cada uma (linguagens, banco, imagens base).
6. **Pré-requisitos:** o que instalar (WSL2, Docker, Compose, Git).
7. **Como rodar em desenvolvimento:** passo a passo do zero, com os comandos, o que esperar ver e onde acessar.
8. **Variáveis de ambiente:** tabela com nome, função e exemplo (sem valores reais) e como criar o `.env` a partir do `.env.example`.
9. **Como rodar em produção:** diferenças em relação ao desenvolvimento e o comando usado.
10. **Pipeline de CI/CD:** o que cada etapa faz (testes, build, push, deploy e teste pós deploy) e quando dispara.
11. **Testes:** como executá-los localmente e o que cobrem.
12. **Deploy:** onde está hospedado, como foi configurado (servidor ou serviço, firewall, HTTPS e certificado) e quais **segredos** o GitHub precisa ter (apenas os nomes).
13. **Monitoramento:** o que é coletado e como acessar os painéis.
14. **Decisões técnicas:** por que escolheram essas tecnologias e imagens, e o que fariam diferente.
15. **Problemas encontrados:** pelo menos 3 erros reais que tiveram e como os resolveram.
16. **Custos e limpeza:** o que consome recursos na nuvem e como desligar.
17. **Contribuição de cada integrante.**

**Critérios de qualidade do README:**

- Alguém de fora do trio deve conseguir rodar o ambiente de desenvolvimento **apenas lendo o README**.
- Nenhuma senha, token, chave ou IP privado pode aparecer no texto.
- Os comandos precisam ser copiáveis e funcionar.
- O README deve estar atualizado com a versão marcada na tag `v1.0`.

### R8. Processo

- Os 3 integrantes com commits no repositório.
- Pull requests com **revisão cruzada**: cada PR é aprovado por outra pessoa do trio. No mínimo 5 PRs mergeados.

## Apresentação (13/12)

Demonstração ao vivo e perguntas. O trio deve:

1. mostrar o ambiente de desenvolvimento com uma alteração instantânea;
2. fazer um commit e acompanhar o pipeline (testes, imagens, deploy e teste pós deploy);
3. abrir a URL pública em HTTPS já atualizada;
4. mostrar o painel de monitoramento, se houver.

O professor pede a cada integrante que explique uma parte do projeto.

## Avaliação (10 pontos)

| Critério | Pontos |
|---|---|
| Aplicação em camadas e Dockerfiles (R1 e R2) | 1,5 |
| Compose em dois ambientes e segurança da configuração (R3) | 1,5 |
| Pipeline de CI/CD com testes (R4) | 2,0 |
| Deploy na nuvem, HTTPS e teste pós deploy (R5) | 1,5 |
| Monitoramento com Prometheus e/ou Grafana (R6, desejável) | 1,0 |
| `README.md` e documentação (R7) | 1,0 |
| Processo, revisão cruzada e entrega pelo GitHub (R8) | 0,5 |
| Apresentação e domínio técnico | 1,0 |
| **Total** | **10,0** |

**Nota individual:** a nota do trio é ajustada pelos commits, pelas revisões feitas e pelas respostas de cada pessoa na apresentação.

## Para consultar

- Docker Compose: https://docs.docker.com/compose/
- GitHub Actions: https://docs.github.com/actions
- Prometheus: https://prometheus.io/docs/
- Grafana: https://grafana.com/docs/
- Let's Encrypt: https://letsencrypt.org/docs/

---

[← Anterior: Semana 14: Pipeline completo e lançamento do projeto](14-semana14-pipeline-completo.md) | [Índice](../../README.md) | [Próximo: Semana 18: Apresentação final do projeto →](18-semana18-apresentacao-final.md)
