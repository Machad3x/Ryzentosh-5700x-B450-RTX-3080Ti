# Troubleshooting — Hackintosh Ryzen + RTX 3080 Ti

Guia de problemas comuns e soluções pro setup Ryzentosh (Ryzen 7 5700X + B450 + RTX 3080 Ti + Tahoe 26).

## Boot

### Black screen / sem imagem no boot

- Verifique se `Above 4G Decoding` está **Enabled** na BIOS.
- Verifique se `CSM` está **Disabled**.
- Verifique se o `config.plist` tem `boot-args` com `nvfb=1 nvaccel=1 nvfbheads=4`.
- Se usar monitor via HDMI, tente DisplayPort (alguns monitores HDMI têm problemas com o framebuffer NVIDIA).
- Verifique se `ResizeAppleGpuBars` está em `-1` (não forçado).

### Kernel panic no boot

- Verifique os logs: OpenCore grava logs em `EFI/OC/` (se `Target` = 67 no Misc → Debug).
- Panics comuns:
  - `IONDRVSupport` → verifique se o Kernel Block está configurado (Exclude).
  - `AppleVTD` / `DMA` → verifique `DisableIoMapper = true` (necessário pro AirportMtkWlan).
  - `AMFIPass` → verifique se está na lista de kexts e habilitado.
- Se o panic é intermitente, tente bootar em **Safe Mode** (pressione espaço no picker → Safe Mode).

### macOS não aparece no picker do OpenCore

- Verifique se a partição EFI foi copiada corretamente (`EFI/OC/config.plist` existe).
- Verifique se `ScanPolicy` é `0` (Misc → Security → ScanPolicy).
- Verifique se o disco interno é detectado (HfsPlus.efi ou ApfsDriverLoader.efi nos Drivers).
- Se dual boot, verifique se `RequestBootVarRouting = YES`.

### Boot lento (~100s de delay)

- Normal se Resizable BAR estiver desabilitado (nullmoth usa fallback small-BAR).
- Pra acelerar: habilite Resizable BAR na BIOS + sete `ResizeAppleGpuBars = 0`.
- Verifique se não há `nv_disable=1` ou `-wegnoegpu` no boot-args.

## Rede

### Ethernet (Realtek RTL8111) não funciona

- **Causa comum**: Gigabyte + Above 4G Decoding pode quebrar a Ethernet.
- **Solução 1**: Troque `RealtekRTL8111.kext` por `LucyRTL8125Ethernet.kext`.
- **Solução 2**: Desabilite Above 4G e use `npci=0x3000` no boot-args (pode conflitar com NVIDIA — teste com cuidado).
- **Solução 3**: Verifique se `built-in` está definido no DeviceProperties pra a Ethernet.

### Wi-Fi (MT7921) não aparece

- Verifique se `AirportMtkWlan.kext` está em `EFI/OC/Kexts/` e habilitado no config.plist.
- Verifique `MinKernel`/`MaxKernel` corretos (Tahoe: 25.0.0–25.99.99).
- Verifique `DisableIoMapper = true` (obrigatório).
- Verifique se o device ID é `14c3:7961` (MT7921). MT7921AU (USB) não é suportado.
- Use o instalador gráfico (`AirportMtkWlan Installer.app`) que configura tudo automaticamente.

### Bluetooth não funciona

- Verifique se o **cabo USB de 9 pinos** da Fenvi está conectado a um header USB 2.0.
- Verifique se a porta USB está mapeada como **Internal (255)** no UTBMap.
- Verifique se `SimpleMtkBtFw.kext`, `SimpleMtkBtPatcher.kext` e `BlueToolFixup.kext` estão carregados.
- Verifique se `Lilu.kext` está antes deles na ordem de carga.

## Áudio

### Sem som (Realtek ALC892)

- Verifique se `AppleALC.kext` está carregado.
- Verifique o `layout-id` no DeviceProperties (atual: 2). Teste outros: 1, 3, 5, 7, 11, 12, 15, 16, 20, 21, 22, 23, 28, 29, 31, 32, 90, 99.
- Verifique se o device path no DeviceProperties corresponde ao seu áudio (PciRoot(0x0)/Pci(0x8,0x1)/Pci(0x0,0x4)).
- Se no Tahoe, considere o NMXAudio do HighDelay fork (opcional).

### HDMI audio da NVIDIA não funciona

- Esperado — o driver nullmoth/HighDelay não suporta HDMI audio.
- Use o áudio onboard (Realtek ALC892) via AppleALC.

## GPU (NVIDIA)

### Driver não carrega (`kmutil showloaded | grep nullmoth` vazio)

- Verifique se o `install.sh` foi executado com sucesso.
- Verifique se as extensões foram autorizadas em System Settings → Privacy & Security.
- Verifique se `csr-active-config` é `0x0A43`.
- Verifique se `SecureBootModel` é `Disabled`.
- Verifique se `Lilu.kext` e `AMFIPass.kext` estão carregados.
- Verifique se não há `nv_disable=1` ou `-wegnoegpu` no boot-args.
- Verifique se `IONDRVSupport` está bloqueado (Kernel → Block, Exclude).

### Tela preta após login

- Tente boot-arg `-nvrmnoflip` (mantém frame no copy path).
- Tente boot-arg `-nvrmnogo` (driver idles até `nvrmctl go`).
- Verifique se o monitor está conectado via DisplayPort (HDMI pode ter problemas).
- Verifique se `nvfbheads=4` está no boot-args (4 display heads).

### Sleep/wake — GPU não acorda

- Use boot-arg `-nvrmnopm` (desliga suporte a sleep).
- O suporte a sleep/wake é experimental no nullmoth/HighDelay.

### Primeiro boot muito lento (~1 min em "PCI configuration end")

- Normal — o driver pausa até ~1 minuto enquanto a GPU sobe (especialmente com Resizable BAR desabilitado).

## Sleep / Power

### Mac não dorme ou acorda sem GPU

- Use boot-arg `-nvrmnopm`.
- Desabilite sleep nas preferências do macOS (Energy Saver) se necessário.

## OCLP (Tahoe)

### Root patches falham

- Verifique se o SIP está parcialmente desabilitado (csr 0x0A43).
- Verifique se o OCLP está atualizado (2.5.1+).
- Se o OCLP não detecta o macOS, verifique se o `config.plist` tem `SecureBootModel = Disabled`.

### macOS não bootou após OCLP root patches

- Boot pelo pendrive (EFI original) e reverta os patches.
- Use o 1401 (se disponível) pra reverter a EFI.

## USB

### Portas USB não funcionam

- Mapeie as portas com USBToolBox (ver [USB-MAPPING.md](USB-MAPPING.md)).
- Substitua `UTBDefault.kext` por `UTBMap.kext`.
- Rode OC Snapshot (Ctrl+R no ProperTree) após trocar.

### Bluetooth da Fenvi não aparece

- Verifique o cabo USB de 9 pinos conectado a header USB 2.0.
- Verifique se a porta USB está mapeada como Internal (255).
- Verifique se `SimpleMtkBtFw.kext`, `SimpleMtkBtPatcher.kext` e `BlueToolFixup.kext` estão carregados.

## Recuperação

### Como reverter a EFI

1. Boot pelo pendrive (EFI original).
2. Monte a partição EFI do disco interno.
3. Substitua a pasta `EFI` pela do pendrive ou pelo backup.
4. Reinicie.

### Como reverter o driver NVIDIA

- O `install.sh` faz backup do driver anterior em `/Library/NullMoth/` ou similar.
- Use o 1401 app → "Remove NVIDIA driver" no boot picker.
- Ou siga os [Recovery removal steps](https://github.com/HighDelay/nvidia-macos-driver-nvk/blob/main/docs/COMBINED-INSTALL.md#recovery-removal) com `nullmoth-remove=1`.

### Backup da EFI

- Sempre mantenha um backup da EFI funcional (no pendrive ou em `C:\Users\machadex\Hackintosh\EFI-Hackintosh.zip`).
- O git repo tem a EFI versionada: https://github.com/Machad3x/Ryzentosh-5700x-B450-RTX-3080Ti

## Fontes

- Dortania Troubleshooting: https://dortania.github.io/OpenCore-Install-Guide/troubleshooting/troubleshooting.html
- nullmoth Troubleshooting: https://github.com/nullmoth/nvidia-macos-driver/blob/main/docs/TROUBLESHOOTING.md
- HighDelay Troubleshooting: https://github.com/HighDelay/nvidia-macos-driver-nvk/blob/main/docs/TROUBLESHOOTING.md
- AirportMtkWlan: https://github.com/Arxusion/AirportMtkWlan
"