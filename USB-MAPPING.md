# Mapeamento USB — USBToolBox / UTBMap

Guia pra mapear as portas USB da sua placa-mãe (GIGABYTE B450 AORUS ELITE) e gerar o `UTBMap.kext`, substituindo o `UTBDefault.kext`.

## Por que mapear USB

- O macOS tem limite de 15 portas por controlador USB.
- Sem mapeamento, portas podem não funcionar ou causar kernel panics.
- O `UTBDefault.kext` é um mapeamento genérico — deve ser substituído pelo `UTBMap.kext` personalizado.

## Ferramenta

- **USBToolBox** (Windows): `C:\Users\machadex\Hackintosh\usbtoolbox\dist\Windows.exe`
- Ou baixe em: https://github.com/USBToolBox/tool/releases

## Status: ✅ Mapeamento concluído

O mapeamento USB foi feito com USBToolBox e o `UTBMap.kext` foi gerado e instalado na EFI. O `UTBDefault.kext` foi removido.

- Controladores mapeados: AMD USB 3.10 (1022-43D5) e AMD USB 3.10 (1022-149C)
- Portas mapeadas: 8 + 8 = 16 portas
- Controlador USBip (emulado) ignorado
- Porta da Fenvi AX1800 definida como Internal (255) pro Bluetooth

## Passos (referência)

### 1. Preparar

1. Certifique-se de que o macOS está instalado e bootando (mesmo que sem mapeamento, usando UTBDefault).
2. No Windows (ou no macOS via Wine/Parallels), execute o `Windows.exe` do USBToolBox.
3. O tool detecta seus controladores USB automaticamente.

### 2. Mapear portas

1. Clique em **"Detect Ports"** ou similar — o tool vai listar todas as portas USB.
2. Para cada porta, você precisa **fisicamente** conectar um dispositivo USB (pendrive, mouse, etc.) em cada porta, uma por vez.
3. O tool detecta qual porta foi ativada e permite classificá-la:
   - **USB 2.0** (preto/branco)
   - **USB 3.0** (azul)
   - **USB 3.1/3.2 Gen 2** (vermelho/teal)
   - **Type-C** (com ou sem USB 2.0 companion)
   - **Internal** (portas internas, como headers USB 2.0/3.0)
   - **Internal (Bluetooth)** — para a Fenvi AX1800 (MT7921), defina como **Internal (255)**
4. Marque cada porta conforme o tipo real.

### 3. Gerar UTBMap.kext

1. Após mapear todas as portas, clique em **"Generate UTBMap.kext"** ou similar.
2. O tool gera o `UTBMap.kext` na pasta escolhida.

### 4. Instalar na EFI

1. Copie o `UTBMap.kext` gerado para `EFI/OC/Kexts/` (no disco interno).
2. **Remova** o `UTBDefault.kext` de `EFI/OC/Kexts/`.
3. Abra o `config.plist` no ProperTree.
4. Pressione **Ctrl+R** (OC Snapshot) — isso atualiza a seção `Kernel -> Add` automaticamente.
5. Salve o `config.plist`.

### 5. Testar

1. Reinicie o macOS.
2. Teste todas as portas USB (conecte dispositivos em cada uma).
3. Se alguma porta não funcionar, reabra o USBToolBox e ajuste o mapeamento.

## Configuração específica pra Fenvi AX1800 (MT7921)

- A placa Fenvi FV-AX1800 tem um **cabo USB de 9 pinos** que conecta a um header USB 2.0 da placa-mãe.
- No mapeamento, a porta USB onde esse cabo está conectado deve ser definida como **Internal (255)**.
- Caso contrário, o macOS trata o Bluetooth como dispositivo externo e ele falha ao acordar do repouso.

## Notas importantes

- Se você tem mais de 15 portas em um único controlador, habilite o patch `XhciPortLimit` no config.plist (mas no Sequoia/Tahoe isso pode não funcionar — mapeie antes).
- O `UTBDefault.kext` é um mapeamento genérico — **sempre** substitua pelo `UTBMap.kext` personalizado.
- Se uma porta não funcionar após o mapeamento, verifique se o tipo está correto (USB 2.0 vs 3.0, Internal vs External).
- Portas USB-C (Type-C) podem ter um companion USB 2.0 — mapeie ambos.

## Fontes

- USBToolBox: https://github.com/USBToolBox/tool
- USBToolBox docs: https://github.com/USBToolBox/tool/wiki
- Dortania USB Mapping Guide: https://dortania.github.io/OpenCore-Post-Install/usb/
"