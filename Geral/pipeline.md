---
tags: [cheatsheet, powershell, cmd, pipeline]
aliases: [Pipeline]
---

# 🔀 Pipeline & Junção — CMD + PowerShell

> [!abstract] Ideia em 1 linha
> **PS passa objetos, CMD passa texto.** Por isso o filtro no PS é por propriedade (`$_.Status`), no CMD é por string (`findstr`).

## 1️⃣ Pipe `|` e os 4 filtros do PS

| Cmdlet | Alias | Papel | Exemplo |
|---|---|---|---|
| `Select-Object` | `select` | Escolhe colunas | `… \| select Name,CPU` |
| `Where-Object` | `where`, `?` | Filtra linhas | `… \| where {$_.Status -eq 'Running'}` |
| `Sort-Object` | `sort` | Ordena | `… \| sort CPU -Desc` |
| `ForEach-Object` | `%` | Roda p/ cada item | `… \| % { Remove-Item $_.FullName }` |

> [!tip] `$_` = item atual
> `$_` ou `$PSItem` é o objeto passando no cano agora.

### Exemplos comentados

```powershell
# 1. Só serviços rodando
Get-Service | Where-Object { $_.Status -eq 'Running' }

# 2. Top 10 CPU
Get-Process | Sort-Object CPU -Descending | Select-Object Name,CPU -First 10

# 3. Apaga todos .log
Get-ChildItem *.log | ForEach-Object { Remove-Item $_.FullName }
```

---

## 2️⃣ Redirecionamento & encadeamento

| Op | Onde | Efeito |
|---|---|---|
| `>` | CMD + PS | Sobrescreve arquivo |
| `>>` | CMD + PS | Anexa no final |
| `2>` | CMD + PS | Erro → arquivo |
| `*>` | PS só | Todos fluxos → arquivo |

### 🔗 CMD

| Sintaxe | Roda o 2º quando… |
|---|---|
| `A && B` | A **deu certo** |
| `A \|\| B` | A **deu erro** |
| `A & B` | **Sempre** |

### 🔗 PowerShell

| Sintaxe | Efeito |
|---|---|
| `A ; B` | Um depois do outro |
| `A && B` / `A \|\| B` | Igual CMD (só PS 7+) |

---

## 🎯 3. Casos HTB / CTF

> [!example] Receitas prontas

**Quantos ocultos existem?**
```powershell
(Get-ChildItem -Hidden -Recurse -File).Count
```

**Achar `flag{` em milhares de arquivos**
```cmd
findstr /s /i "flag{" C:\Users\*\Documents\*
```
```powershell
Get-ChildItem C:\ -Recurse -File -EA SilentlyContinue | Select-String "flag{"
```

**Processos `vm*` suspeitos**
```cmd
tasklist | findstr /i "^vm"
```
```powershell
Get-Process | Where-Object { $_.Name -like "vm*" }
```
