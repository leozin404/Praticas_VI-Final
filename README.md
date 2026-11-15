# Praticas_VI-Final

**Triagem automatizada de caixas com visão embarcada: inspeção em linha logística com ESP32-S3-CAM e Edge Impulse**

Projeto da disciplina ELI060 Práticas de Inovação VI (Centro Universitário FEI, 2026).
Equipe: Leonardo Aparecido da Silva, Rafaela Rios Ribeiro e Ramon Saraiva Lustosa Santos.

Um ESP32-S3-CAM (N16R8) colocado sobre uma mini esteira classifica cada quadro em quatro classes, **aberta**, **amassada**, **fechada** e **vazio**, com uma CNN int8 treinada no Edge Impulse. A inferência roda no próprio microcontrolador (cerca de 161 ms por imagem). A decisão usa a média das probabilidades em janelas de 2,5 s, com limiar de 70%.

- Projeto público no Edge Impulse (dados, treino e métricas): https://studio.edgeimpulse.com/public/1163248/live
- Validação (int8): 96,0% de acurácia. Teste independente: 91,9% de acurácia, F1 macro 0,92.

## Estrutura

| Pasta | Conteúdo |
|---|---|
| `firmwares/inspecao_caixas_v4` | Firmware de inspeção (inclui `Bocis_inferencing.h`) |
| `firmwares/captura_serial` | Firmware de captura do conjunto de dados pela USB (comando `f`) |
| `biblioteca/` | Biblioteca Arduino exportada do Edge Impulse (EON, int8) |
| `builds/` | Binários completos (`.merged.bin`) prontos para gravar |
| `ferramentas/captura_dataset.html` | Captura das fotos no navegador (Web Serial): salva 96×96 em cinza e o original 320×240 |
| `ferramentas/monitor_caixas_v2.html` | Monitor serial: probabilidades, bipe em caixa com defeito, contagem, CSV e comparação com o modelo no navegador |

## Uso rápido (sem compilar)

1. Abra o https://espressif.github.io/esptool-js/ no Chrome ou no Edge e conecte a placa.
2. Grave `builds/inspecao_caixas_v4.ino.merged.bin` no endereço `0x0`.
3. Abra `ferramentas/monitor_caixas_v2.html`, conecte a placa a 921600 baud e posicione a câmera no suporte, sobre o fundo preto.

## Recompilar

1. Na Arduino IDE, instale a biblioteca: *Sketch > Incluir Biblioteca > Adicionar .ZIP*, escolhendo o arquivo de `biblioteca/`.
2. Salve `firmwares/inspecao_caixas_v4` como `inspecao_caixas_v4/inspecao_caixas_v4.ino` (o `.ino` precisa ficar numa pasta de mesmo nome).
3. Placa **ESP32S3 Dev Module**, *PSRAM: OPI PSRAM*, *Flash Size: 16MB*, *USB CDC On Boot: Enabled*. Núcleo arduino-esp32 3.3.12.

## Capturar mais imagens

Grave `builds/captura_serial.ino.merged.bin` e use `ferramentas/captura_dataset.html` a 115200 baud. O nome de cada arquivo começa com a classe (por exemplo, `aberta.0002.jpg`), e o Edge Impulse usa isso para rotular a imagem.

## Limitações conhecidas

- A classe **vazio** foi treinada com o mousepad preto como fundo. Com a correia exposta, a iluminação ambiente faz o sistema indicar **amassada**. A correção prevista é padronizar a iluminação no suporte da câmera e capturar novas imagens da correia em movimento.
- A saída digital de alerta (`PINO_ALERTA`) está desligada (`-1`). O alerta aparece só na serial e no monitor.
