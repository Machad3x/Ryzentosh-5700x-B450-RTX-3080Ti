# Guia OCLP — Atualizar Sequoia → Tahoe 26

Guia passo a passo pra usar o OpenCore Legacy Patcher (OCLP) pra atualizar do macOS Sequoia 15.8.1 pro macOS Tahoe 26 no seu Ryzen (x86_64).

## Pré-requisitos

- macOS Sequoia 15.8.1 instalado e bootando (via pendrive).
- EFI copiada pro disco interno (ver [POS-INSTALACAO.md](POS-INSTALACAO.md)).
- OCLP 2.5.1 instalado (`OpenCore-Patcher.pkg` na pasta Hackintosh ou baixe em https://github.com/dortania/OpenCore-Legacy-Patcher/releases).
- SIP parcialmente desabilitado (csr-active-config = 0x0A43, já configurado no config.plist).
- SecureBootModel = Disabled (já configurado).

## Passos

### 1. Instalar o OCLP

1. Dê double-click em `OpenCore-Patcher.pkg` (ou baixe o .pkg).
2. Siga o instalador (instala em `/Applications/OpenCore-Patcher.app`).
3. Abra o OCLP.

### 2. Criar instalador do Tahoe

1. No OCLP, clique em **"Create macOS Installer"**.
2. Selecione **macOS Tahoe 26** (ou a versão mais recente disponível).
3. Escolha onde salvar (ex: pendrive ou disco interno).
4. Aguarde o download (~10-15 GB).
5. O OCLP cria o instalador bootável.

### 3. Instalar o Tahoe

1. No OCLP, clique em **"Install macOS"**.
2. Selecione o instalador criado no passo 2.
3. Selecione o disco de destino (mesmo disco do Sequoia).
4. Aguarde a instalação (pode levar 30-60 min).
5. O macOS reinicia automaticamente.

### 4. Aplicar Root Patches

1. Após o Tahoe instalar e bootar, abra o OCLP.
2. Clique em **"Post-Install Root Patch"**.
3. O OCLP detecta o hardware e aplica os patches necessários (AppleHDA rollback, etc.).
4. Clique em **"Start Root Patching"**.
5. Aguarde a conclusão.
6. **Reinicie** quando solicitado.

### 5. Reinstalar o driver NVIDIA (HighDelay)

Após o Tahoe bootar com root patches:

1. Abra o Terminal.
2. Navegue até a pasta do driver:
   ```sh
   cd /path/to/nvidia-pkgroot
   ```
3. Verifique checksums:
   ```sh
   shasum -a 256 --strict -c SHA256SUMS --quiet
   ```
4. Instale:
   ```sh
   sudo ./install.sh
   # ou double-click Install.command
   sudo shutdown -r now
   ```
5. Autorize as extensões em System Settings → Privacy & Security.
6. Reinicie novamente.
7. Verifique:
   ```sh
   kmutil showloaded --list-only | grep nullmoth
   system_profiler SPDisplaysDataType | head -20
   ```

### 6. Reinstalar Wi-Fi/BT (se necessário)

Se o Wi-Fi/BT parar de funcionar após o OCLP root patches:

1. Use o instalador gráfico `AirportMtkWlan Installer.app`.
2. Ou reinstale manualmente os kexts (AirportMtkWlan, SimpleMtkBtFw, SimpleMtkBtPatcher, BlueToolFixup).
3. Verifique `DisableIoMapper = true` no config.plist.

## Troubleshooting OCLP

### OCLP não detecta o macOS

- Verifique se o SIP está parcialmente desabilitado (csr 0x0A43).
- Verifique se `SecureBootModel = Disabled`.
- Verifique se o OCLP está atualizado.

### Root patches falham

- Verifique se o OCLP suporta a versão do Tahoe instalada.
- Tente "Revert Root Patches" e reaplique.
- Verifique os logs do OCLP (Help → View Logs).

### macOS não bootou após root patches

- Boot pelo pendrive (EFI original).
- Monte a partição EFI do disco interno.
- Substitua a pasta `EFI` pela do pendrive.
- Reinicie e tente novamente.

### Driver NVIDIA não carrega após OCLP

- O OCLP root patches podem ter alterado o SIP. Verifique `csr-active-config` = 0x0A43.
- Reinstale o driver (install.sh).
- Autorize as extensões novamente.

## Notas

- O OCLP modifica o sistema (root patches) — isso pode ser revertido.
- Após cada update do macOS, pode ser necessário reaplicar root patches.
- O driver NVIDIA (nullmoth/HighDelay) é compatível com Tahoe x86_64, mas o OS em si precisa de root patches (OCLP).
- Mantenha sempre um backup da EFI funcional (pendrive ou `EFI-Hackintosh.zip`).

## Fontes

- OCLP: https://github.com/dortania/OpenCore-Legacy-Patcher
- OCLP Guide: https://dortania.github.io/OpenCore-Legacy-Patcher/
- nullmoth driver: https://github.com/nullmoth/nvidia-macos-driver
- HighDelay driver: https://github.com/HighDelay/nvidia-macos-driver-nvk
"