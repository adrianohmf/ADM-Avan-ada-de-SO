[Índice](../README.md) | [Próximo: Semana 2 — Gerenciamento de processos e serviços →](../unidade_1/01_semana_02_processos_e_servicos.md)

# Antes de começar: preparando o WSL2

Todos os comandos deste material foram pensados para rodar dentro do **WSL2**. Alguns pontos importantes:

1. **Instalação** (PowerShell como administrador):
   ```powershell
   wsl --install -d Ubuntu
   ```
2. **Atualize o sistema** logo após instalar (e sempre que for retomar os estudos após um tempo parado):
   ```bash
   sudo apt update && sudo apt dist-upgrade
   ```
   O `apt update` atualiza a lista de pacotes disponíveis, e o `apt dist-upgrade` instala as versões mais novas, inclusive lidando com mudanças de dependências entre pacotes (diferente do `apt upgrade`, mais conservador).
3. **Systemd no WSL2**: por padrão pode vir desabilitado. Para habilitar, edite `/etc/wsl.conf` dentro da distribuição:
   ```ini
   [boot]
   systemd=true
   ```
   Depois, no PowerShell:
   ```powershell
   wsl --shutdown
   ```
   E abra o Ubuntu novamente. Sem isso, os comandos `systemctl` da Semana 2 não funcionam.
4. **Rede**: o WSL2 expõe `localhost` automaticamente para o Windows — ao subir um servidor web na porta 80, ele já pode ser acessado em `http://localhost` no navegador do Windows.
5. **Limitações importantes**: o WSL2 roda sobre um disco virtual único (`.vhdx`), então **não há discos físicos separados** para praticar LVM "de verdade". Na Semana 4, vamos usar **arquivos de disco virtuais (loop devices)** para simular volumes — o comportamento dos comandos é idêntico ao de um servidor real.

---

[Índice](../README.md) | [Próximo: Semana 2 — Gerenciamento de processos e serviços →](../unidade_1/01_semana_02_processos_e_servicos.md)
