# Hackintosh — Ryzen 7 5700X + RTX 3080 Ti — macOS Tahoe 26

Gerado por OpCore Simplify (target macOS Tahoe 26) + ajustes manuais.
Driver de GPU: **HighDelay/nvidia-macos-driver-nvk** (fork do nullmoth, v3.0.2-nvk).

## Hardware

| Componente | Modelo | Status |
|---|---|---|
| CPU | AMD Ryzen 7 5700X (Vermeer) | OK (AMD Vanilla patches) |
| GPU | NVIDIA GeForce RTX 3080 Ti (Ampere, 10DE-2208) | HighDelay NVK driver (pendente validação física) |
| Placa-mãe | GIGABYTE B450 AORUS ELITE (B450) | OK |
| Áudio | Realtek ALC892 (10EC-0892) | OK (AppleALC + SimpleHDA) |
| Ethernet | Realtek RTL8111 (10EC-8168) | OK (RealtekRTL8111) |
| Wi-Fi | MediaTek MT7961 (14C3-7961) | NÃO suportado no macOS |
| Bluetooth | MediaTek (0489-E0CD) | NÃO suportado no macOS |
| NVMe | ADATA XPG SX8200 Pro | OK |
| SATA | TOSHIBA 1TB + HUSKY 256GB + KINGSTON 240GB | OK |

## SMBIOS

`MacPro7,1` (OpCore escolhe automaticamente pra Tahoe; iMacPro1,1 não suporta Tahoe).

⚠️ O app **1401** do nullmoth usa `iMacPro1,1`. No Tahoe, prefira a **instalação manual** (`install.sh`) pra manter `MacPro7,1`.

## BIOS (GIGABYTE B450 AORUS ELITE)

- Secure Boot: **Disabled**
- CSM: **Disabled** (UEFI puro)
- Above 4G Decoding: **Enabled** (driver exige — card mapeia memória acima de 4GB)
- Resizable BAR / SAM: **Disabled** por padrão (driver usa fallback small-BAR com delay ~100s)
- VT-d: Disabled (se existir)
- Fast Boot: Disabled
- XHCI Hand-off: Enabled

## Config.plist (já aplicado)

- SMBIOS: `MacPro7,1`
- boot-args: `-v debug=0x100 keepsyms=1 nvfb=1 nvaccel=1 nvfbheads=4 -nvkmsnosmooth amfi_get_out_of_my_way=0x1 amfi=0x80 ipc_control_port_options=0`
  - Remova `-v debug=0x100 keepsyms=1` depois de validar (produção).
- csr-active-config: `0x0A43`
- SecureBootModel: Disabled
- Kernel Block: `com.apple.iokit.IONDRVSupport` (Exclude, MinKernel 24.0.0)
- ResizeGpuBars: -1, ResizeAppleGpuBars: -1
- SetupVirtualMap: false
- ProvideCurrentCpuInfo: true (AMD)
- XhciPortLimit: false

## Kexts (EFI/OC/Kexts)

- Lilu.kext
- AMFIPass.kext
- AppleALC.kext
- AppleMCEReporterDisabler.kext
- RealtekRTL8111.kext
- RestrictEvents.kext
- SimpleHDA.kext (Tahoe removeu AppleHDA)
- USBToolBox.kext + UTBDefault.kext (trocar por UTBMap.kext após mapear USB)
- VirtualSMC.kext

## Instalação

### 1. Boot pelo pendrive

Reinicie e boot pelo pendrive (F12 ou BIOS boot order). Selecione o instalador do macOS.

### 2. Instalar macOS Sequoia 15.8.1

O pendrive contém a recovery do Sequoia 15.8.1 (última versão Intel/x86_64 da Apple). Instale no disco interno.

### 3. Copiar EFI pro disco interno

Siga o `POS-INSTALACAO.md`.

### 4. Atualizar pro Tahoe 26 via OCLP

1. Instale o **OpenCore Legacy Patcher 2.5.1** (arquivo `OpenCore-Patcher.pkg` na pasta Hackintosh, ou baixe em https://github.com/dortania/OpenCore-Legacy-Patcher/releases).
2. Abra o OCLP → **Settings** → habilite "Allow native models" se necessário.
3. Vá em **Create macOS Installer** → selecione macOS Tahoe 26 → baixe.
4. Vá em **Install macOS** → instale o Tahoe.
5. Após instalar, volte ao OCLP → **Post-Install Root Patch** → aplique os patches.
6. Reinicie.

### 5. Instalar o driver HighDelay (NVIDIA)

⚠️ OpCore Simplify avisa: "NullMoth NVIDIA driver is experimental."

**Use a instalação manual** (não o 1401, pra não trocar o SMBIOS pra iMacPro1,1):

1. Baixe a release `nvidia-macos-driver-3.0.2-nvk.tar.gz` (já em `C:\Users\machadex\Hackintosh\`, ou em https://github.com/HighDelay/nvidia-macos-driver-nvk/releases).
2. Extraia e verifique checksums:
   ```sh
   tar -xzf nvidia-macos-driver-3.0.2-nvk.tar.gz && cd pkgroot
   shasum -a 256 --strict -c SHA256SUMS --quiet
   ```
3. Instale:
   ```sh
   sudo ./install.sh
   # ou double-click Install.command
   sudo shutdown -r now
   ```
4. Na primeira vez, autorize as extensões em System Settings → Privacy & Security, depois reinicie.
5. Verifique:
   ```sh
   kmutil showloaded --list-only | grep nullmoth
   system_profiler SPDisplaysDataType | head -20
   ```

Opcionais do HighDelay fork (default No, perguntados no install.sh):
- **NMXAudio HD Audio** (Tahoe only)
- **NMIntelFB** (painel interno de laptop via Intel)
- **Notebook TGP** (`nvrmtgp=1`)

### 6. Mapear USB (recomendado)

Use USBToolBox pra mapear portas, gere UTBMap.kext, coloque em EFI/OC/Kexts, remova UTBDefault.kext, rode OC Snapshot (Ctrl+R no ProperTree) e salve.

## Avisos

- **RTX 3080 Ti (Ampere)** está em "Pending" de validação física. Pode funcionar, pode ter bugs. Reporte issues no repo.
- **Resizable BAR**: desabilitado por padrão. Se quiser boot mais rápido/estável, habilite Resizable BAR na BIOS e sete `ResizeAppleGpuBars` para `0`.
- **Gigabyte + Above 4G Decoding**: em placas Gigabyte/Aorus, habilitar Above 4G pode quebrar drivers (ex.: Ethernet). Como o driver EXIGE Above 4G ON, mantenha ligado; se a Ethernet parar, tente `npci=0x3000`.
- **SetupVirtualMap já desabilitado** (correto pra B450 com BIOS recente).
- **MediaTek Wi-Fi/BT não funciona** no macOS. Use Ethernet ou compre adaptador Broadcom/Intel (ex: Fenvi T919).
- **HDMI audio da NVIDIA** não suportado. Use o áudio onboard Realtek (já configurado via AppleALC + SimpleHDA). Se não houver som, troque o layout-id no DeviceProperties (testar 1, 3, 5, 7, 11, 12... pra ALC892). O HighDelay fork oferece NMXAudio como opcional.
- **Sleep/wake**: driver tem suporte experimental. Se o Mac dormir e acordar sem GPU, use boot-arg `-nvrmnopm`.
- **Primeiro boot com o driver** pausa até ~1 minuto em "PCI configuration end" — normal.
- **SSDT-CPUR** não necessário (apenas B550/A520).
- **XhciPortLimit** desabilitado — mapeie USB antes de depender de muitas portas.

## Reverter o driver

O install.sh faz backup do driver anterior. Se o macOS não iniciar, siga os [Recovery removal steps](https://github.com/HighDelay/nvidia-macos-driver-nvk/blob/main/docs/COMBINED-INSTALL.md#recovery-removal) usando `nullmoth-remove=1`, depois reinstale o pacote anterior.

## Arquivos na pasta Hackintosh

- `EFI/` — EFI completa (Tahoe, MacPro7,1, HighDelay config)
- `EFI-Hackintosh.zip` — backup zipado
- `OpenCore-Patcher.pkg` — OCLP 2.5.1 (instalar no macOS)
- `nvidia-macos-driver-3.0.2-nvk.tar.gz` — HighDelay driver v3.0.2
- `SHA256SUMS-HighDelay.txt` — checksums do driver
- `README.md` — este arquivo
- `POS-INSTALACAO.md` — como copiar EFI pro disco interno
- `usb-backup/` — backup do conteúdo anterior do pendrive
"