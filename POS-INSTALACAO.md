# Pós-Instalação — Copiar EFI pro disco interno

Guia pra bootar sem o pendrive após instalar o macOS.

## Quando usar

Depois de instalar o macOS no disco interno (NVMe/SATA) e validar que tudo funciona bootando pelo pendrive.

## Opção 1 — Pelo macOS (mais fácil)

1. Boot pelo pendrive, entre no macOS instalado.
2. Abra o **Terminal**.
3. Monte a partição EFI do disco interno:
   ```sh
   sudo diskutil list
   # identifique a EFI do disco interno (ex: disk0s1, ~200MB, FAT32)
   sudo diskutil mount disk0s1
   ```
4. Monte a EFI do pendrive (se não estiver montada):
   ```sh
   sudo diskutil mount disk2s1   # ajuste o número do pendrive
   ```
5. Copie a pasta EFI:
   ```sh
   # backup da EFI existente (se houver)
   cp -R /Volumes/EFI/EFI /Volumes/EFI/EFI.backup 2>/dev/null
   # copia a nossa EFI
   cp -R /Volumes/USB/EFI /Volumes/EFI/
   ```
   Ou arraste a pasta `EFI` do pendrive pra dentro da partição EFI do disco interno no Finder.

6. Reinicie **sem** o pendrive. Deve bootar pelo OpenCore direto.

## Opção 2 — Pelo Windows (antes de instalar, ou via dual boot)

1. Use o **diskpart** pra montar a partição EFI do disco interno:
   ```cmd
   diskpart
   list disk
   select disk 0   (disco interno)
   list partition
   select partition 1   (EFI, ~200MB, FAT32)
   assign letter=E
   exit
   ```
2. Copie a pasta `EFI` de `C:\Users\machadex\Hackintosh\EFI` para `E:\`.
3. Remova a letra:
   ```cmd
   diskpart
   select partition 1
   remove letter=E
   exit
   ```

## Opção 3 — Usando o OpenCore (mais seguro)

O OpenCore tem um tool `Bootstrap` que configura a NVRAM pra apontar pro OpenCore no disco interno. Mas o mais simples e confiável é copiar a pasta EFI manualmente (Opção 1 ou 2).

## Dicas importantes

- **Não formate** a partição EFI do disco interno — só copie/sobrescreva a pasta `EFI`. Formatar pode destruir o boot do Windows (se dual boot).
- Se tiver **dual boot com Windows**, a partição EFI já existe e é compartilhada — só adicione/substitua a pasta `EFI/OC` e `EFI/BOOT`.
- Depois de copiar, **remova o pendrive** e reinicie. Se não bootar, verifique na BIOS se o boot order aponta pro disco interno (ou use F12 pra selecionar).
- Se o macOS não aparecer no picker do OpenCore, verifique se a partição EFI foi copiada corretamente e se o `config.plist` está em `EFI/OC/`.

## Se algo der errado

- Boot pelo pendrive de novo e repita o processo.
- Use o 1401 (se estiver usando nullmoth) pra reverter/reparar a EFI.
- Mantenha sempre um backup da EFI funcional no pendrive.

## Nota sobre Tahoe 26 no x86_64

A recovery baixada é do macOS Sequoia 15.8.1 (Intel/x86_64) — a última versão que a Apple fornece pra Intel. Pra chegar ao **Tahoe 26** no Ryzen (x86_64), você precisará usar **OCLP (OpenCore Legacy Patcher)** depois de instalar o Sequoia, pois a Apple não fornece recovery do Tahoe pra Intel. O nullmoth driver suporta Tahoe x86_64, mas o OS em si precisa de root patches.

### Caminho sugerido

1. Instale o Sequoia 15.8.1 (via recovery do pendrive).
2. Copie a EFI pro disco interno (este guia).
3. Use o OCLP pra aplicar root patches e atualizar pro Tahoe 26.
4. Instale o driver nullmoth (install.sh) — já com SIP parcialmente desabilitado (csr 0x0A43).
5. Valide com `kmutil showloaded | grep nullmoth` e `system_profiler SPDisplaysDataType`.
"