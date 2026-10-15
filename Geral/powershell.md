---
tags: [cheatsheet, powershell]
aliases: [PowerShell Base]
---

# 💙 PowerShell — Base & Descoberta

> [!abstract] Alias → Comando → Uso
> `==destaque==` = parâmetro que mais cai. Tabela 1 = precisa decorar. Tabelas 2-3 = consulta.

## 🧭 1. Base e descoberta

| Alias | Comando | O que faz | Parâmetros ouro |
|---|---|---|---|
| `gcm` | `Get-Command` | Lista cmdlets/funções/alias | `Name` · `Module` · `Syntax` |
| `help` | `Get-Help` | Ajuda | `Examples` · `Detailed` · `Online` · `Full` |
| `gm` | `Get-Member` | **Propriedades/métodos do objeto** | `MemberType` · `Static` |
| `ls, dir` | `Get-ChildItem` | Lista arquivos | `Hidden` · `Recurse` · `File` · `Directory` · `Depth` |
| `cd` | `Set-Location` | Muda pasta | `Path` · `PassThru` |
| `gc, cat` | `Get-Content` | Lê arquivo | `Tail` · `Raw` · `TotalCount` |
| `ni` | `New-Item` | Cria pasta/arquivo | `ItemType File/Directory` · `Force` |
| `rm, del` | `Remove-Item` | Apaga | `Recurse` · `Force` · `Confirm:$false` |
| — | `Set-ExecutionPolicy` | `Bypass` libera script | `RemoteSigned` (padrão) · `Bypass` · `Scope CurrentUser` |

## 🖥️ 2. SO, rede e dados

| Alias | Comando | O que faz | Parâmetros ouro |
|---|---|---|---|
| `ps, gps` | `Get-Process` | Processos | `Name` · `Id` · `IncludeUserName` |
| `kill` | `Stop-Process` | Mata | `Name`/`Id` · `Force` |
| `gsv` | `Get-Service` | Serviços | `Name` · `DependentServices` |
| — | `Restart-Service` | Reinicia travado | `Name` · `Force` |
| — | `Test-Connection` | Ping avançado | `Count` · `Quiet` (bool) |
| — | `Resolve-DnsName` | DNS | `Type A/MX` · `Server` |
| — | `Get-NetIPAddress` | IP detalhado | `AddressFamily IPv4` · `InterfaceAlias` |
| — | `Get-LocalUser` | Usuários | `Name` · `Enabled` |
| `epcsv` | `Export-Csv` | Salva CSV | `NoTypeInformation` · `Delimiter` |
| `ipcsv` | `Import-Csv` | Lê CSV | `Delimiter` · `Header` |
| — | `ConvertTo-Json` | → JSON | `Depth` · `Compress` |
| `iwr` | `Invoke-WebRequest` | Baixa/acessa URL | `Uri` · `OutFile` · `Method` · `Headers` · `Body` |

## 📡 3. Logs, WMI, AD e remoto

| Alias | Comando | O que faz | Parâmetros ouro |
|---|---|---|---|
| — | `Get-WinEvent` | Lê Event Log | `LogName Security/System` · `MaxEvents` · `FilterXPath` |
| `gcim` | `Get-CimInstance` | WMI moderno | `ClassName Win32_Process` · `Filter "Name='x'"` |
| — | `Get-ADUser` | User AD | `Filter {Enabled -eq $true}` · `Properties *` · `Identity` |
| `icm` | `Invoke-Command` | Comando remoto | `ComputerName` · `ScriptBlock {}` · `Credential` |
| `etsn` | `Enter-PSSession` | Shell remoto | `ComputerName` · `Credential` |
| — | `Foreach -Parallel` | Loop paralelo (PS7) | `Parallel` |

> [!tip] Descoberta no alvo
> `comando | Get-Member` mostra o que dá para filtrar. `Get-Help comando -Examples` é mais rápido que Google.
