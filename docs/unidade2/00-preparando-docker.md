[← Anterior: Checklist geral (Semanas 2 a 8)](../unidade1/09-checklist-unidade1.md) | [Índice](../../README.md) | [Próximo: Semana 10: Introdução a containers e Docker →](10-semana10-introducao-docker.md)

# Antes de começar: Docker no WSL2

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

[← Anterior: Checklist geral (Semanas 2 a 8)](../unidade1/09-checklist-unidade1.md) | [Índice](../../README.md) | [Próximo: Semana 10: Introdução a containers e Docker →](10-semana10-introducao-docker.md)
