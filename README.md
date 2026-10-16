# Administração Avançada de Sistemas Operacionais

> Material de apoio da disciplina optativa **Administração Avançada de Sistemas Operacionais** (Sistemas para Internet, IFPE), com teoria, comandos, imagens e exercícios.

**Ambiente considerado: WSL2 (Windows Subsystem for Linux) com Ubuntu.**

## Como o material está organizado

- **Unidade 1 (Semanas 1 a 9):** Fundamentos, Linux + servidor web. A Semana 9 é o seminário avaliativo.
- **Unidade 2 (Semanas 10 a 18):** Docker + CI/CD com GitHub Actions, projeto final, acompanhamento nas Semanas 15 a 17 e apresentação na Semana 18.
- Cada assunto está em um arquivo próprio dentro da pasta `docs/`, e as imagens ficam em `images/`.
- A Unidade 2 dá continuidade à mesma aplicação configurada manualmente na Unidade 1.

## Antes de começar

1. Tenha o **WSL2 com Ubuntu** instalado e atualizado (passo a passo em [Preparando o WSL2](docs/unidade1/00-preparando-wsl2.md)).
2. Para a Unidade 2, instale também o Docker ([Preparando o Docker no WSL2](docs/unidade2/00-preparando-docker.md)).
3. Siga os arquivos na ordem do índice. Cada um termina com exercícios e tem links para o anterior e o próximo.

## Índice

### Unidade 1: Fundamentos (Linux + Servidor Web)

- [Preparando o WSL2](docs/unidade1/00-preparando-wsl2.md)
- [Semana 2: Gerenciamento de processos e serviços](docs/unidade1/01-semana02-processos-e-servicos.md)
- [Complemento: Acesso SSH, rede básica e teste de conectividade](docs/unidade1/02-complemento-ssh-e-rede.md)
  - [Par de chaves: pública e privada](docs/unidade1/02-complemento-ssh-e-rede.md#par-de-chaves-pública-e-privada)
  - [Usuários com sudo e por que evitar o root](docs/unidade1/02-complemento-ssh-e-rede.md#usuários-com-sudo-e-por-que-evitar-o-root)
  - [Configurando IP fixo no Ubuntu Server](docs/unidade1/02-complemento-ssh-e-rede.md#configurando-ip-fixo-no-ubuntu-server)
- [Semana 3: Permissões e sistema de arquivos](docs/unidade1/03-semana03-permissoes.md)
- [Semana 4: Armazenamento e monitoramento básico](docs/unidade1/04-semana04-armazenamento.md)
- [Semana 5: Shell scripting para automação](docs/unidade1/05-semana05-shell-scripting.md)
- [Semana 6: Introdução a servidores web](docs/unidade1/06-semana06-servidores-web.md)
- [Semana 7: Proxy reverso](docs/unidade1/07-semana07-proxy-reverso.md)
- [Semana 8: TLS/SSL](docs/unidade1/08-semana08-tls-ssl.md)
- [Checklist geral (Semanas 2 a 8)](docs/unidade1/09-checklist-unidade1.md)

### Unidade 2: Docker + CI/CD

- [Preparando o Docker no WSL2](docs/unidade2/00-preparando-docker.md)
- [Semana 10: Introdução a containers e Docker](docs/unidade2/10-semana10-introducao-docker.md)
- [Semana 11: Docker avançado](docs/unidade2/11-semana11-docker-avancado.md)
- [Semana 12: Docker Compose](docs/unidade2/12-semana12-docker-compose.md)
- [Semana 13: Introdução a CI/CD e GitHub Actions](docs/unidade2/13-semana13-cicd-github-actions.md)
- [Semana 14: Pipeline completo e lançamento do projeto](docs/unidade2/14-semana14-pipeline-completo.md)
- [Semanas 15 a 17: Acompanhamento do projeto](docs/unidade2/15-semanas15-17-acompanhamento.md)
- [Semana 18: Apresentação final do projeto](docs/unidade2/18-semana18-apresentacao-final.md)
- [Checklist geral (Semanas 10 a 18)](docs/unidade2/19-checklist-unidade2.md)

### Material complementar

- [Introdução a Kubernetes (opcional)](docs/complementar/kubernetes.md)
- [Referências](docs/referencias.md)

---

*Professor: Adriano França (IFPE).*
