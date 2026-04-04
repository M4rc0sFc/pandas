# Sync Bitácora → Git

Ejecutá los pasos en orden. Los pasos 1-3 usan `pwsh -NoProfile` (solo operaciones de archivo). El paso 4 usa comandos Bash nativos para git — NO dentro de PowerShell, ya que el hook `block-no-verify` no puede verificar git cuando corre dentro de pwsh y bloquea el comando.

## Paso 1 — Validar fuente

```bash
pwsh -NoProfile -Command "
\$src = 'C:\Users\Marco\Documents\Obsidian\Tesis Backup\Bitácora.md'
if (-not (Test-Path \$src)) { Write-Error 'ERROR: No se encontró el archivo fuente'; exit 1 }
if ((Get-Item \$src).Length -eq 0) { Write-Error 'ERROR: El archivo fuente está vacío'; exit 1 }
Write-Host 'Fuente validada OK'
"
```

## Paso 2 — Detectar cambios

```bash
pwsh -NoProfile -Command "
\$src = 'C:\Users\Marco\Documents\Obsidian\Tesis Backup\Bitácora.md'
\$dst = 'C:\Users\Marco\OneDrive\Documents\Pandas con Claude\Bitacora\Bitacora.md'
if (-not (Test-Path \$dst)) { Write-Host 'SYNC_NEEDED'; exit 0 }
\$h1 = (Get-FileHash \$src -Algorithm SHA256).Hash
\$h2 = (Get-FileHash \$dst -Algorithm SHA256).Hash
if (\$h1 -ne \$h2) { Write-Host 'SYNC_NEEDED' } else { Write-Host 'NO_CHANGES' }
"
```

Si el resultado es `NO_CHANGES`, detené la ejecución. No hay nada que sincronizar.

## Paso 3 — Copiar

```bash
pwsh -NoProfile -Command "
\$src = 'C:\Users\Marco\Documents\Obsidian\Tesis Backup\Bitácora.md'
\$dst = 'C:\Users\Marco\OneDrive\Documents\Pandas con Claude\Bitacora\Bitacora.md'
\$dstDir = 'C:\Users\Marco\OneDrive\Documents\Pandas con Claude\Bitacora'
if (-not (Test-Path \$dstDir)) { New-Item -ItemType Directory -Path \$dstDir | Out-Null }
Copy-Item -Path \$src -Destination \$dst -Force
Write-Host 'Copiado OK'
"
```

## Paso 4 — Git commit y push

Ejecutar como comandos Bash nativos (no dentro de PowerShell):

```bash
cd "C:/Users/Marco/OneDrive/Documents/Pandas con Claude" && git add "Bitacora/Bitacora.md" && git commit -m "sync(bitacora): $(date -u +%Y-%m-%dT%H:%M:%S+00:00)"
```

```bash
cd "C:/Users/Marco/OneDrive/Documents/Pandas con Claude" && git push
```

---

Si algún paso falla (excepto el push), detené la ejecución e informá el error sin modificar el archivo fuente.
