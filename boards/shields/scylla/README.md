# Scylla MK2 — ZMK Shield (handwired)

## Estrutura de arquivos

```
zmk-config/
├── build.yaml
└── config/
    ├── west.yml
    ├── boards/
    │   └── shields/
    │       └── scylla/
    │           ├── scylla.dtsi           ← matriz + row-gpios (compartilhado)
    │           ├── scylla_left.overlay   ← col-gpios C0–C5
    │           ├── scylla_right.overlay  ← col-gpios C6–C11 + col-offset
    │           ├── Kconfig.defconfig     ← nome do teclado e roles BLE
    │           ├── Kconfig.shield        ← detecção dos shields
    │           └── scylla.keymap         ← keymap padrão (58 teclas)
    └── scylla.conf                       ← (opcional) sleep, BT power etc.
```

## west.yml

```yaml
manifest:
  defaults:
    revision: v0.3
  remotes:
    - name: zmkfirmware
      url-base: https://github.com/zmkfirmware
  projects:
    - name: zmk
      remote: zmkfirmware
      revision: v0.3
      import: app/west.yml
  self:
    path: config
```

## scylla.conf (opcional)

```conf
CONFIG_ZMK_SLEEP=y
CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=900000
CONFIG_ZMK_EXT_POWER=y
```

## Pinagem (nice!nano via pro_micro)

| Função | pro_micro | nRF P |
|--------|-----------|-------|
| R0     | 2         | P0.17 |
| R1     | 3         | P0.20 |
| R2     | 4         | P0.22 |
| R3     | 5         | P0.24 |
| R4     | 6         | P1.00 |
| R5     | 7         | P0.11 |
| C0/C6  | 8         | P1.04 |
| C1/C7  | 9         | P1.06 |
| C2/C8  | 21        | P0.31 |
| C3/C9  | 20        | P0.29 |
| C4/C10 | 19        | P0.02 |
| C5/C11 | 18        | P1.15 |

## Direção dos diodos

`col2row` — anodo na coluna, catodo na linha.
- Linhas: entradas com `GPIO_PULL_DOWN`
- Colunas: saídas sem pull

## Mapa de posições (matrix_transform)

```
Pos  0– 11 → linha 0 (R0, C0–C11)
Pos 12– 23 → linha 1 (R1, C0–C11)
Pos 24– 35 → linha 2 (R2, C0–C11)
Pos 36– 47 → linha 3 (R3, C0–C11)
Pos 48– 53 → thumb linha 4: RC(4,3) RC(4,4) RC(4,5) | RC(4,6) RC(4,7) RC(4,8)
Pos 54– 57 → thumb linha 5: RC(5,4) RC(5,5) | RC(5,6) RC(5,7)
```

Os bindings do keymap seguem esta ordem — 58 entradas por layer.

## Procedimento de flash

1. Grave `settings_reset` na esquerda → aguarde reiniciar
2. Grave `settings_reset` na direita → aguarde reiniciar
3. Grave `scylla_left` na esquerda
4. Grave `scylla_right` na direita
5. Ligue a esquerda primeiro, aguarde ~5s, ligue a direita
6. Aguarde até 60s para o BLE interno emparelhar

Se alguma tecla sair errada, verifique a posição no mapa acima
e ajuste o binding correspondente no keymap.
