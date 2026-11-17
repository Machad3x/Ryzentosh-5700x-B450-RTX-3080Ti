# Análise do 1401 Probe (NullMoth) — pendrive anterior

## O que era o pendrive

O pendrive SanDisk Cruzer Blade (16GB) era um **1401 Probe** da NullMoth Systems — um scanner de hardware offline que roda Linux. Foi usado pra escanear seu hardware e gerar o SysReport que o OpCore Simplify consumiu.

Fonte: https://github.com/nullmoth/1401 (docs/HOW-IT-WORKS.md)

## Como funciona o 1401 Probe

1. **Boot Linux offline** — inicia um kernel Linux 6.18.56 LTS minimalista (sem rede, Wi-Fi/BT desativados por padrão).
2. **GRUB** apresenta menu:
   - "1401 Probe - scan this computer (recommended)" (`p1401.mode=full`)
   - "1401 Probe - safe scan" (`nomodeset p1401.mode=safe`) — se o full travar
   - "Debug shell" (`p1401.mode=shell`)
   - "Turn off"
3. **Hardware Sniffer** (roda via WMI no Linux) lê: CPU, placa-mãe, BIOS/UEFI, GPUs (e firmware), Wi-Fi/BT chips, áudio, USB, storage controllers, ACPI.
4. **acpidump** (pinned por SHA-256) despeja as tabelas ACPI (DSDT, SSDTs, etc.) — usadas pelo OpCore pra patches.
5. **Testa a GPU**: carrega drivers NVIDIA (closed 470/580 + open 610) e Mesa Vulkan 26.1.6 pra ler firmware e capacidades da RTX 3080 Ti. Tela pode piscar/pretejar por até 10 min — normal.
6. **Switch off radios**: Wi-Fi/BT ligam por ~1 min pra registrar firmware, depois desligam. Redes ao redor NÃO são gravadas.
7. **Remove PII**: serial numbers, UUIDs, MAC addresses, chave do Windows são removidos do report.
8. **Escreve o report** em `REPORTS/<computador>.zip` (+ .txt) no pendrive.
9. **Desliga o computador.**

Tempo total: 5 a 20 minutos.

## Arquivos que estavam no pendrive

### Boot (Linux)
| Arquivo | Tamanho | Função |
|---|---|---|
| `boot/vmlinuz` | 14 MB | Kernel Linux 6.18.56 LTS |
| `boot/initramfs.zst` | 175 MB | Initramfs com o scanner + drivers |
| `boot/grub/grub.cfg` | 838 B | Menu GRUB (4 opções) |
| `EFI/BOOT/BOOTX64.EFI` | 4.4 MB | GRUB EFI bootloader |

### Drivers (firmware + módulos Linux)
| Arquivo | Tamanho | Função |
|---|---|---|
| `drivers/nvidia-closed-470.256.02.tar.xz` | 34 MB | Driver NVIDIA fechado 470 (Kepler/Pascal/Volta) — pro scan de GPU antiga |
| `drivers/nvidia-closed-580.178.04.tar.xz` | 97 MB | Driver NVIDIA fechado 580 (Maxwell/Pascal/Volta) |
| `drivers/nvidia-open-610.57.04.tar.xz` | 97 MB | Driver NVIDIA open-source 610 (Turing+) — usado pro seu RTX 3080 Ti |
| `drivers/radios-6.18.56-0-lts.tar.xz` | 61 MB | Firmware Wi-Fi/BT (inclui MediaTek MT7961) |
| `drivers/vulkan-mesa-26.1.6-r0.tar.xz` | 77 MB | Mesa Vulkan 26.1.6 (pro scan de gráficos) |

### Licenças
- `licenses/linux-firmware/` — LICENCE.nvidia, LICENSE.amdgpu, LICENSE.i915, LICENSE.radeon, LICENSE.xe, WHENCE-graphics.txt

### Metadados
- `1401-probe.marker` (109 B) — marcador que identifica o pendrive como probe
- `SHA256SUMS` — checksums de todos os arquivos (integridade)
- `README.txt` — instruções de uso (em inglês)
- `REPORTS/README.txt` — instruções de upload do report
- `GPL-3.0.txt`, `LICENSE.txt`, `THIRD-PARTY-NOTICES.txt`

## O que o 1401 Probe fez no seu PC

1. Escaneou CPU (Ryzen 7 5700X), placa (B450 AORUS ELITE), BIOS (F68a, UEFI, Secure Boot Enabled).
2. Escaneou GPU (RTX 3080 Ti, Ampere, 10DE-2208) — carregou o driver NVIDIA open 610 + Mesa Vulkan pra ler firmware.
3. Escaneou Wi-Fi/BT (MediaTek MT7961) — carregou firmware radio por ~1 min, depois desligou.
4. Escaneou áudio (Realtek ALC892, NVIDIA HD Audio), rede (RTL8111), USB, storage (NVMe ADATA + 3 SATA).
5. Despejou ACPI (DSDT + 7 SSDTs) — usados pelo OpCore.
6. Removeu serials, UUIDs, MACs, chave do Windows.
7. Gerou `REPORTS/<pc>.zip` + `.txt` no pendrive.
8. Desligou o PC.

O report gerado foi usado pelo OpCore Simplify pra criar a EFI do Hackintosh (a que está em `C:\Users\machadex\Hackintosh\EFI`).

## Relação com Wi-Fi/BT

O arquivo `radios-6.18.56-0-lts.tar.xz` (61 MB) contém firmware de rádio do Linux (incluindo MediaTek). Foi usado **apenas pelo scanner** pra identificar seu chip Wi-Fi/BT e ler seu firmware. **Não tem relação com o Hackintosh** — o macOS não usa firmware Linux, e seu MediaTek MT7961 não tem driver no macOS de qualquer forma.

## Segurança

- O probe é **100% offline** — sem rede, sem upload automático.
- Travou drives em read-only antes de escanear (não tocou no Windows/SSD).
- Removeu PII do report (serials, UUIDs, MACs, chave Windows).
- O report é texto puro + firmware dumps — você pode abrir o .zip e ler tudo.

## Status atual

- ✅ Probe já foi executado (SysReport em `C:\Users\machadex\Downloads\OpCore-Simplify-main\SysReport`).
- ✅ Backup do probe salvo em `C:\Users\machadex\Hackintosh\usb-backup\` (incluindo README, marker, drivers, licenses, SHA256SUMS).
- ✅ Pendrive formatado e preparado pro Hackintosh (EFI + recovery Sequoia).
- O pendrive probe não é mais necessário — pode ser descartado ou mantido como backup.

## Fontes online

- 1401 (app): https://github.com/nullmoth/1401
- 1401 Probe (USB scanner): https://github.com/nullmoth/1401 (docs/HOW-IT-WORKS.md)
- 1401 releases: https://github.com/nullmoth/1401/releases
- nullmothsystems.com: https://nullmothsystems.com
- NVIDIA driver (nullmoth): https://github.com/nullmoth/nvidia-macos-driver
- NVIDIA driver (HighDelay fork): https://github.com/HighDelay/nvidia-macos-driver-nvk
- OpCore Simplify: https://github.com/lzhoang2801/OpCore-Simplify
- Hardware Sniffer: https://github.com/lzhoang2801/Hardware-Sniffer
"