# Configurações da BIOS — GIGABYTE B450 AORUS ELITE

Guia completo de settings da BIOS pra rodar macOS (Sequoia/Tahoe) com OpenCore + driver NVIDIA (nullmoth/HighDelay).

Referenciado pelo [README.md](README.md).

## ✅ OBRIGATÓRIO (ativar)

| Setting | Valor | Por quê |
|---|---|---|
| **Above 4G Decoding** | Enabled | O driver NVIDIA (nullmoth/HighDelay) exige — a RTX 3080 Ti mapeia memória acima de 4GB. Sem isso, o driver não carrega. |
| **CSM (Compatibility Support Module)** | Disabled | UEFI puro. CSM ativo causa problemas de boot e `gIO` stalls. |
| **Secure Boot** | Disabled | O OpenCore e as kexts (não assinadas pela Apple) são bloqueadas pelo Secure Boot. |
| **UEFI Mode** | UEFI (não Legacy/CSM) | Necessário pro OpenCore bootar. |
| **XHCI Hand-off** | Enabled | Permite que o macOS controle os controladores USB 3.0. |
| **SATA Mode** | AHCI | macOS não suporta RAID/IDE. |

## ✅ RECOMENDADO (ativar)

| Setting | Valor | Por quê |
|---|---|---|
| **Resizable BAR / SAM** | Disabled (padrão) | O driver nullmoth/HighDelay funciona com fallback small-BAR (delay ~100s no boot). Se quiser boot mais rápido, habilite e sete `ResizeAppleGpuBars` para `0`. |
| **Fast Boot** | Disabled | Evita que o BIOS pule inicialização de dispositivos necessários pro macOS. |
| **VT-d** | Disabled (se existir) | AMD não tem VT-d nativo; se a opção existir, desative pra evitar conflitos. |
| **IOMMU** | Disabled | AMD não usa IOMMU pra macOS; o AirportMtkWlan exige `DisableIoMapper = true` no OpenCore (já configurado). |
| **EHCI Hand-off** | Enabled (se existir) | Complementa o XHCI Hand-off. |

## ⚠️ CUIDADO (Gigabyte/Aorus)

| Setting | Observação |
|---|---|
| **Above 4G Decoding** | Em placas Gigabyte/Aorus, habilitar Above 4G pode **quebrar a Ethernet** ou causar falha de boot em outros OSes. Se a Ethernet (Realtek RTL8111) parar de funcionar, tente `npci=0x3000` no boot-args (sem Above 4G) — mas isso pode conflitar com o driver NVIDIA. Teste com cuidado. |
| **ResizeAppleGpuBars** | Se habilitar Resizable BAR, sete `ResizeAppleGpuBars = 0` (Dortania). O driver nullmoth diz "keep existing value" e configura o BAR sozinho após o macOS iniciar. |

## ❌ DESATIVAR

| Setting | Por quê |
|---|---|
| **CSM** | Já listado acima — deve estar desativado. |
| **Secure Boot** | Bloqueia kexts não assinadas. |
| **Fast Boot** | Já listado acima. |
| **Intel VT-d / AMD IOMMU** | Já listado acima. |

## 🔧 CONFIGURAÇÕES AVANÇADAS (se disponível)

| Setting | Valor | Por quê |
|---|---|---|
| **XHCI Hand-off** | Enabled | (já listado) |
| **EHCI Hand-off** | Enabled | (já listado) |
| **SATA Mode** | AHCI | (já listado) |
| **Boot Option Priorities** | USB first (durante instalação), depois disco interno | Pra bootar pelo pendrive na instalação |
| **Hard Drive BBS Priorities** | Disco interno (após instalar) | Pra bootar pelo disco interno |
| **Windows 8.1/10 WHQL Support** | Enabled ou Other OS | Algumas placas Gigabyte precisam de "Other OS" pra bootar macOS |

## 📋 Checklist antes de instalar

1. [ ] Secure Boot → Disabled
2. [ ] CSM → Disabled
3. [ ] Above 4G Decoding → Enabled
4. [ ] Resizable BAR → Disabled (padrão)
5. [ ] Fast Boot → Disabled
6. [ ] XHCI Hand-off → Enabled
7. [ ] SATA Mode → AHCI
8. [ ] Boot order → USB first (pendrive com EFI)
9. [ ] (Opcional) IOMMU → Disabled

## ⚠️ Nota importante

Se após habilitar Above 4G a **Ethernet (Realtek RTL8111) parar de funcionar**, você tem duas opções:
1. Desabilitar Above 4G e usar `npci=0x3000` no boot-args (mas isso pode conflitar com o driver NVIDIA).
2. Manter Above 4G e trocar a kext de rede (ex: `LucyRTL8125Ethernet.kext` em vez de `RealtekRTL8111.kext`).

O driver NVIDIA (nullmoth/HighDelay) **exige** Above 4G ON, então a opção 1 é arriscada. Prefira manter Above 4G e resolver a Ethernet de outra forma se necessário.

## Fontes

- Dortania OpenCore Install Guide (AMD Zen): https://dortania.github.io/OpenCore-Install-Guide/AMD/zen.html
- nullmoth/nvidia-macos-driver (OpenCore wiring): https://github.com/nullmoth/nvidia-macos-driver
- HighDelay/nvidia-macos-driver-nvk (OpenCore setup): https://github.com/HighDelay/nvidia-macos-driver-nvk/blob/main/docs/OPENCORE-SETUP.md
"