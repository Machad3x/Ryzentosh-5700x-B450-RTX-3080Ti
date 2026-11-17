# Wi-Fi e Bluetooth — Fenvi AX1800 PCIe (MediaTek MT7921)

Guia completo pra ativar Wi-Fi e Bluetooth da placa Fenvi FV-AX1800 PCIe (chipset MediaTek MT7921) no macOS Tahoe 26.

## Hardware

- **Placa**: Fenvi FV-AX1800 PCIe
- **Chip Wi-Fi**: MediaTek MT7921 (PCI ID `14c3:7961`)
- **Chip BT**: MediaTek (via USB interno)
- **Wi-Fi**: 802.11ax (Wi-Fi 6), VHT80
- **Bluetooth**: 5.2

## Kexts necessárias

| Kext | Função | Fonte |
|---|---|---|
| `Lilu.kext` | Base (deve ser a primeira) | https://github.com/acidanthera/Lilu |
| `AirportMtkWlan.kext` | Wi-Fi nativo (menu AirPort, Keychain, AWDL) | https://github.com/Arxusion/AirportMtkWlan |
| `SimpleMtkBtFw.kext` | Firmware loader BT | https://github.com/mnnobre/SimpleMtkBtFw (ou laobamac) |
| `SimpleMtkBtPatcher.kext` | Patcher BT | (incluído no SimpleMtkBtFw) |
| `BlueToolFixup.kext` | Fixup BT (do BrcmPatchRAM) | https://github.com/acidanthera/BrcmPatchRAM |

## Passos

### 1. Cabo USB interno (obrigatório pro BT)

1. Localize o cabo USB de 9 pinos que acompanha a Fenvi FV-AX1800.
2. Conecte-o a um **header USB 2.0 livre** na placa-mãe (GIGABYTE B450 AORUS ELITE).
3. Sem esse cabo, o Bluetooth não aparecerá no sistema.

### 2. Mapeamento USB (Internal 255)

1. Mapeie as portas USB com USBToolBox (ver [USB-MAPPING.md](USB-MAPPING.md)).
2. A porta USB onde o cabo da Fenvi está conectado deve ser definida como **Internal (255)**.
3. Caso contrário, o macOS trata o BT como dispositivo externo e falha ao acordar do repouso.

### 3. Instalar kexts

**Opção A — Instalador gráfico (recomendado):**
1. Baixe `AirportMtkWlan-Installer-v2.1.6.zip` em https://github.com/Arxusion/AirportMtkWlan/releases
2. Extraia e abra `AirportMtkWlan Installer.app`
3. Selecione a partição EFI (ou clique Browse)
4. Escolha `macOS 26 Tahoe`
5. Marque "Install companion Bluetooth drivers"
6. Clique **Deploy Unified Suite to EFI** — instala Wi-Fi, BT e AWDL, habilita `DisableIoMapper` e configura o config.plist.

**Opção B — Manual:**
1. Baixe `AirportMtkWlan-Tahoe-v2.1.6-RELEASE.zip` e extraia `AirportMtkWlan.kext` → `EFI/OC/Kexts/`
2. Baixe `SimpleMtkBtFw-0.1.0.zip` e extraia `SimpleMtkBtFw.kext` + `SimpleMtkBtPatcher.kext` → `EFI/OC/Kexts/`
3. Baixe `BrcmPatchRAM-2.7.2-RELEASE.zip` e extraia `BlueToolFixup.kext` → `EFI/OC/Kexts/`
4. Adicione as entradas no `config.plist` → `Kernel -> Add` (ver abaixo).
5. Habilite `DisableIoMapper = true` em `Kernel -> Quirks`.
6. Rode OC Snapshot (Ctrl+R no ProperTree).

### 4. config.plist (Tahoe)

```xml
<!-- AirportMtkWlan.kext -->
<dict>
    <key>Arch</key>
    <string>x86_64</string>
    <key>BundlePath</key>
    <string>AirportMtkWlan.kext</string>
    <key>Comment</key>
    <string>MediaTek MT7921 Native Wi-Fi (Tahoe)</string>
    <key>Enabled</key>
    <true/>
    <key>ExecutablePath</key>
    <string>Contents/MacOS/AirportMtkWlan</string>
    <key>MinKernel</key>
    <string>25.0.0</string>
    <key>MaxKernel</key>
    <string>25.99.99</string>
    <key>PlistPath</key>
    <string>Contents/Info.plist</string>
</dict>
```

```xml
<!-- SimpleMtkBtFw.kext -->
<dict>
    <key>Arch</key>
    <string>x86_64</string>
    <key>BundlePath</key>
    <string>SimpleMtkBtFw.kext</string>
    <key>Comment</key>
    <string>MediaTek BT Firmware Loader</string>
    <key>Enabled</key>
    <true/>
    <key>ExecutablePath</key>
    <string>Contents/MacOS/SimpleMtkBtFw</string>
    <key>MinKernel</key>
    <string></string>
    <key>MaxKernel</key>
    <string></string>
    <key>PlistPath</key>
    <string>Contents/Info.plist</string>
</dict>
```

```xml
<!-- SimpleMtkBtPatcher.kext -->
<dict>
    <key>Arch</key>
    <string>x86_64</string>
    <key>BundlePath</key>
    <string>SimpleMtkBtPatcher.kext</string>
    <key>Comment</key>
    <string>MediaTek BT Patcher</string>
    <key>Enabled</key>
    <true/>
    <key>ExecutablePath</key>
    <string>Contents/MacOS/SimpleMtkBtPatcher</string>
    <key>MinKernel</key>
    <string></string>
    <key>MaxKernel</key>
    <string></string>
    <key>PlistPath</key>
    <string>Contents/Info.plist</string>
</dict>
```

```xml
<!-- BlueToolFixup.kext -->
<dict>
    <key>Arch</key>
    <string>x86_64</string>
    <key>BundlePath</key>
    <string>BlueToolFixup.kext</string>
    <key>Comment</key>
    <string>BlueTool Fixup</string>
    <key>Enabled</key>
    <true/>
    <key>ExecutablePath</key>
    <string>Contents/MacOS/BlueToolFixup</string>
    <key>MinKernel</key>
    <string></string>
    <key>MaxKernel</key>
    <string></string>
    <key>PlistPath</key>
    <string>Contents/Info.plist</string>
</dict>
```

```xml
<!-- Kernel -> Quirks -->
<key>DisableIoMapper</key>
<true/>
```

### Ordem de carga (Kernel -> Add)

1. Lilu.kext
2. AMFIPass.kext
3. AirportMtkWlan.kext
4. SimpleMtkBtFw.kext
5. SimpleMtkBtPatcher.kext
6. BlueToolFixup.kext
7. AppleALC.kext
8. AppleMCEReporterDisabler.kext
9. RealtekRTL8111.kext
10. RestrictEvents.kext
11. SimpleHDA.kext
12. USBToolBox.kext
13. UTBDefault.kext
14. VirtualSMC.kext

## Limitações

- **USB MediaTek (MT7921AU)** não é suportado — apenas PCIe MT7921/MT7922.
- **MT7902/MT7902LEN** não é suportado.
- Para macOS 14 Sonoma ou anterior, use `SimpleMtkWlan` + HeliPort (não AirPort nativo).
- Verifique `MinKernel`/`MaxKernel` corretos pro seu macOS:
  - Sequoia: 24.2.0–24.99.99
  - Tahoe: 25.0.0–25.99.99

## Troubleshooting

### Wi-Fi não aparece
- Verifique se `AirportMtkWlan.kext` está carregado (`kextstat | grep -i mtk`).
- Verifique `DisableIoMapper = true`.
- Verifique `MinKernel`/`MaxKernel`.
- Verifique se o device ID é `14c3:7961`.

### Bluetooth não aparece
- Verifique o cabo USB interno conectado.
- Verifique se a porta USB está mapeada como Internal (255).
- Verifique se `SimpleMtkBtFw.kext`, `SimpleMtkBtPatcher.kext` e `BlueToolFixup.kext` estão carregados.

### Wi-Fi conecta mas sem internet
- Verifique DNS (use 8.8.8.8 ou 1.1.1.1).
- Verifique se o router suporta 802.11ax.

## Fontes

- AirportMtkWlan: https://github.com/Arxusion/AirportMtkWlan
- SimpleMtkBtFw: https://github.com/mnnobre/SimpleMtkBtFw (fork de laobamac)
- BrcmPatchRAM: https://github.com/acidanthera/BrcmPatchRAM
- Dortania OpenCore Post-Install: https://dortania.github.io/OpenCore-Post-Install/
"