---
tags: [cheatsheet, cmd, windows]
aliases: [CMD Base]
---

# ⬛ CMD — Guia de Bolso

> [!abstract] Tudo com `/?`
> Qualquer comando + `/?` mostra ajuda só no CMD. `help dir` = mesma coisa.

## 🗂️ 1. Navegação e arquivos

| Comando | O que faz | Essencial |
|---|---|---|
| `dir` | Lista | `/a` ocultos · `/s` subpastas |
| `cd` | Navega | `cd \` raiz · `cd ..` volta · `cd /d D:` troca drive |
| `tree` | Árvore | `/f` inclui arquivos |
| `mkdir` / `md` | Cria pasta | `"pasta com espaço"` com aspas |
| `rmdir` / `rd` | Remove pasta | `/s` cheia · `/q` quieto |
| `type` | Lê texto | `type arquivo.txt` |
| `copy` | Copia simples | `copy a.txt b\` |
| `xcopy` | Cópia avançada | `/s` subpastas · `/e` +vazias · `/h` ocultos |
| `robocopy` | Cópia pro | `/E` +vazias · `/Z` retomável · `/MT:8` paralelo · `/MIR` espelha |
| `ren` | Renomeia | `ren velho.txt novo.txt` |
| `move` | Move | `move C:\a\b.txt C:\c\` |
| `del` | Apaga | `del *.tmp /s /q` |
| `attrib` | Atributos | `+h/-h` oculta · `+r/-r` leitura · `+s/-s` sistema |
| `cls` | Limpa tela | — |

## 🌐 2. Rede e troubleshooting

| Comando | O que faz | Essencial |
|---|---|---|
| `ipconfig` | Rede | `/all` MAC/DNS · `/renew` novo IP · `/flushdns` (PS: `Clear-DnsClientCache`) |
| `ping` | Vivo? | `-t` contínuo · `-n 4` qtd · `-l 1500` tamanho |
| `tracert` | Rota | `-d` sem DNS (rápido) · `-h 30` max hops |
| `nslookup` | DNS | `nslookup google.com` |
| `netstat` | Conexões | `netstat -ano` = tudo + número + PID |
| `arp -a` | IP → MAC | Rede local |
| `route print` | Rotas | Gateway + métricas |
| `systeminfo` | SO completo | Versão + hotfixes + boot |
| `hostname` | Nome máquina | — |
| `tasklist` | Processos | `/v` dono · `/fi "PID eq 1234"` · `/svc` serviços |
| `taskkill` | Mata | `/F` força · `/PID 1234` · `/IM notepad.exe` |
| `find` | Acha string | `/i` ignora case — `find "erro" log.txt` |
| `findstr` | Grep do CMD | `/i` · `/s` subpastas · `/m` só nomes — `findstr /i /s "senha" *.txt` |
| `sfc /scannow` | Repara SO | Substitui corrompidos |

## 🔑 3. Admin, registro e WMI

| Comando | O que faz | Essencial |
|---|---|---|
| `net user` | Users | `net user` lista · `net user ana /add` cria · `net localgroup Admins ana /add` |
| `icacls` | Perms | `/grant User:(F)` full · `(M)` modify · `(R)` read |
| `schtasks` | Agendadas | `/query /fo LIST /v` detalhe · `/create /tn "n" /tr "cmd" /sc daily` |
| `reg` | Registro | `query HKLM\..\Run` · `add` · `delete` |
| `gpupdate /force` | GPO agora | AD imediato |
| `vssadmin` | Shadow copy | `list shadows` · `delete shadows` |
| `wmic` | WMI legado | `process get name,executablepath` · `product get name,version` |

> [!warning] `wmic` depreciado
> Prefira `Get-CimInstance` no PowerShell. `wmic` ainda cai em server antigo.
