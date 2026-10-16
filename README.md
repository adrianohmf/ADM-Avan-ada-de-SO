# Administração Avançada de Sistemas Operacionais
### Unidades 1 e 2 — Semanas 2 a 18

> Material de apoio completo da disciplina, com teoria, comandos, imagens e exercícios.
> **Ambiente considerado: WSL2 (Windows Subsystem for Linux) com Ubuntu.**
> A Unidade 1 (Fundamentos — Linux + Servidor Web) cobre as Semanas 2 a 8. A Unidade 2 (Docker + CI/CD) cobre as Semanas 10 a 18, dando continuidade à mesma aplicação configurada manualmente na Unidade 1.

---

## Sumário

### Unidade 1 — Fundamentos (Linux + Servidor Web)

- [Antes de começar: preparando o WSL2](unidade_1/00_preparando_wsl2.md)
- [Semana 2 — Gerenciamento de processos e serviços](unidade_1/01_semana_02_processos_e_servicos.md)
- [Complemento — Acesso SSH, rede básica e teste de conectividade](unidade_1/02_complemento_ssh_rede.md)
  - [Par de chaves: pública e privada](unidade_1/02_complemento_ssh_rede.md#par-de-chaves-pública-e-privada)
  - [Usuários com sudo e por que evitar o root](unidade_1/02_complemento_ssh_rede.md#usuários-com-sudo-e-por-que-evitar-o-root)
  - [Configurando IP fixo no Ubuntu Server](unidade_1/02_complemento_ssh_rede.md#configurando-ip-fixo-no-ubuntu-server)
- [Semana 3 — Permissões e sistema de arquivos](unidade_1/03_semana_03_permissoes.md)
- [Semana 4 — Armazenamento e monitoramento básico](unidade_1/04_semana_04_armazenamento.md)
- [Semana 5 — Shell scripting para automação](unidade_1/05_semana_05_shell_scripting.md)
- [Semana 6 — Introdução a servidores web](unidade_1/06_semana_06_servidores_web.md)
- [Semana 7 — Proxy reverso](unidade_1/07_semana_07_proxy_reverso.md)
- [Semana 8 — TLS/SSL](unidade_1/08_semana_08_tls_ssl.md)
- [Checklist geral (Semanas 2 a 8)](unidade_1/09_checklist_unidade_1.md)

### Unidade 2 — Docker + CI/CD

- [Antes de começar: Docker no WSL2](unidade_2/00_docker_no_wsl2.md)
- [Semana 10 — Introdução a containers e Docker](unidade_2/01_semana_10_containers_docker.md)
- [Semana 11 — Docker avançado](unidade_2/02_semana_11_docker_avancado.md)
- [Semana 12 — Docker Compose](unidade_2/03_semana_12_docker_compose.md)
- [Semana 13 — Introdução a CI/CD e GitHub Actions](unidade_2/04_semana_13_cicd_github_actions.md)
- [Semana 14 — Pipeline completo e lançamento do projeto](unidade_2/05_semana_14_pipeline_completo.md)
- [Semanas 15 a 17 — Acompanhamento do projeto](unidade_2/06_semanas_15_a_17_acompanhamento.md)
- [Semana 18 — Apresentação final do projeto](unidade_2/07_semana_18_apresentacao_final.md)
- [Checklist geral (Semanas 10 a 18)](unidade_2/08_checklist_unidade_2.md)

### Material complementar

- [Material complementar (opcional) — Introdução a Kubernetes](complementar/kubernetes.md)
- [Referências](#referências)

---

## Como navegar

Cada semana está em um arquivo próprio, organizado nas pastas `unidade_1/`, `unidade_2/` e `complementar/`. No início e no fim de cada arquivo há links para o conteúdo anterior, o próximo e este índice. As imagens ficam na pasta `images/`.

---

## Referências

- TITTEL, Ed; SAHA, Sayan; WATT, Steve; ADAM, Michael; BOHAC, Steve. *Container Storage For Dummies*, 2nd Red Hat Special Edition. Hoboken: John Wiley & Sons, 2019. Disponível em: [https://www.redhat.com/rhdc/managed-files/st-container-storage-for-dummies-analyst-material-f31727-202206-en.pdf](https://www.redhat.com/rhdc/managed-files/st-container-storage-for-dummies-analyst-material-f31727-202206-en.pdf).
- STACK OVERFLOW. *2025 Developer Survey: Technology*. 2025. Disponível em: [https://survey.stackoverflow.co/2025/technology](https://survey.stackoverflow.co/2025/technology). Acesso em: out. 2026.
- DOCKER INC. *2025 Docker State of Application Development Report*. 2025. Disponível em: [https://www.docker.com/blog/2025-docker-state-of-app-dev/](https://www.docker.com/blog/2025-docker-state-of-app-dev/). Acesso em: out. 2026.
