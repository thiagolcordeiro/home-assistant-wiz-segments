# Hardware validation / Validação de hardware

## Validated platform / Plataforma validada

- WiZ RGBIC 5 m, product / produto **605568**.
- Module / módulo: `ESP25_MHORGB_01`.
- Firmware: `1.38.0`.
- Addressable block / bloco endereçável: **6 LEDs**.
- Validated custom mode / modo personalizado validado: **258**.

Independent RGB regions and both white controls have been visually validated.
Warm white and cool white are correctly mapped in Home Assistant's RGBWW order.
Version 0.2.1 was validated in Home Assistant; the new 0.3.0 animations still
require visual validation on hardware.

As regiões RGB independentes e os dois controles de branco foram validados
visualmente. Branco quente e branco frio estão corretamente mapeados na ordem
RGBWW do Home Assistant. A versão 0.2.1 foi validada no Home Assistant; as novas
animações da 0.3.0 ainda precisam de validação visual no hardware.

## Confirmed channel mapping / Mapeamento confirmado

| Output / Saída | Home Assistant `rgbww_color` | Wire field / Campo no protocolo |
|---|---|---|
| Red / Vermelho | `[255, 0, 0, 0, 0]` | Step index / índice 1 |
| Green / Verde | `[0, 255, 0, 0, 0]` | Step index / índice 2 |
| Blue / Azul | `[0, 0, 255, 0, 0]` | Step index / índice 3 |
| Warm white / Branco quente | `[0, 0, 0, 0, 255]` | Step index / índice 4 |
| Cool white / Branco frio | `[0, 0, 0, 255, 0]` | Step index / índice 5 |
| RGB white / Branco RGB | `[255, 255, 255, 0, 0]` | RGB combined / RGB combinados |

White controls were tested separately with RGB zeroed. An RGB-white reference
produced a visibly different white from the cool-white control. Home Assistant
orders the channels as R,G,B,cold,warm; the wire uses R,G,B,warm,cold. The encoder
converts that order. These results confirm functional white controls; emitter
composition, Kelvin calibration and mixed-white brightness curves are unmeasured.

Os controles de branco foram testados separadamente, com RGB zerado. A referência
de branco RGB apresentou diferença visual em relação ao controle de branco frio.
O Home Assistant usa R,G,B,frio,quente; o protocolo usa R,G,B,quente,frio.
O codificador converte essa ordem. Os resultados confirmam controles funcionais;
a composição dos emissores, a calibração em Kelvin e as curvas de brilho das
misturas de branco não foram medidas.

## Protocol behavior / Comportamento do protocolo

The integration enforces a 12-region command limit, including dark gaps and the
unused tail. The device does not return segment colors, so entity state is assumed.
A saved WiZ custom mode may override supplied colors even when a command is
acknowledged. Keep the configured slot free of saved modes. The all-off command
uses only `setPilot` with `state: false`; combined restoration fields containing
sceneId 0 can be rejected by this firmware.

A integração limita os comandos a 12 regiões, incluindo lacunas apagadas e o
final sem uso. O dispositivo não devolve as cores dos segmentos; o estado das
entidades é presumido. Um modo personalizado salvo no WiZ pode substituir as cores
enviadas mesmo com confirmação do comando. Mantenha o modo configurado livre.
O comando para desligar tudo usa apenas `setPilot` com `state: false`; campos
combinados de restauração contendo sceneId 0 podem ser recusados pelo firmware.

## Diagnostic utility / Utilitário de diagnóstico

Read-only identification / Identificação sem alterar a iluminação:

```text
python tools/probe.py STRIP_IP
```

Optional RGB test, starting with the strip off: three regions, each three blocks
wide, for 20 seconds. The utility restores and checks the off state afterward.
This utility tests RGB; the separate white-control validation is recorded above.

Teste RGB opcional, com a fita inicialmente apagada: três regiões de três blocos
por 20 segundos. O utilitário restaura e verifica o estado desligado ao final.
Esse utilitário testa RGB; a validação separada dos brancos está registrada acima.

```text
python tools/probe.py STRIP_IP --test --slot 258 --width 3 --seconds 20
```

Replace `STRIP_IP` with the device address. Width is in blocks, not individual LEDs.
Substitua `STRIP_IP` pelo endereço do dispositivo. A largura é em blocos, não LEDs individuais.

## References / Referências

- [Independent protocol investigation / Investigação independente do protocolo](https://github.com/TechAntohere/WizScreenSyncController/)
- [Official product sheet / Ficha oficial do produto](https://www.assets.signify.com/is/content/Signify/US.en_US.046677605568)
