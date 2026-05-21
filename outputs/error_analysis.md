# Análise Qualitativa — YOLO vs Gemma 3 27B

## Top-10 Casos com Maior Discordância de Score

| Imagem | Score GT | Score YOLO | Score VLM | ΔScore |
|--------|----------|------------|-----------|--------|
| taco_000021_jpg.rf.4c0c64517c9718ce41b14ea367f32c9c.jpg | 1.000 | 0.000 | 1.000 | 1.000 |
| taco_000028_jpg.rf.7f506106c8b04f85d83c4a043ca81d76.jpg | 0.120 | 0.000 | 1.000 | 1.000 |
| taco_000030_jpg.rf.b18fd79ef6fde285940bd6bd5ce46ebe.jpg | 0.150 | 0.000 | 1.000 | 1.000 |
| taco_000048_jpg.rf.7a29791da64064c88111e17dd7772ea2.jpg | 1.000 | 1.000 | 0.000 | 1.000 |
| td_000021_JPG.rf.6e12cd40798b7d23827c4f4c4251c1e4.jpg | 0.120 | 0.000 | 1.000 | 1.000 |
| td_000027_JPG.rf.289949d2c4a4a2e071fd59e62005ec66.jpg | 0.310 | 0.000 | 1.000 | 1.000 |
| td_000049_jpg.rf.77c9d6643925c95d9da6d8b1936168d6.jpg | 0.200 | 1.000 | 0.000 | 1.000 |
| td_000071_JPG.rf.c9303f76740cb3fad6ec5048ce02eb60.jpg | 0.180 | 0.000 | 1.000 | 1.000 |
| td_000077_JPG.rf.59b1c51d94bc23a176cb9e0eb4be797d.jpg | 0.450 | 0.000 | 1.000 | 1.000 |
| td_000090_JPG.rf.79d2a658a28292251897b458f2f29ac0.jpg | 0.120 | 0.000 | 1.000 | 1.000 |

## Casos por Nível de Irregularidade


### Nível: BAIXO

- **al_IMG_4476_MOV-0007_jpg.rf.4f77e0e1fe42ef57fc83bf0ec43e0b76.jpg** — GT:0.100 YOLO:0.100 VLM:1.000
- **al_IMG_4476_MOV-0042_jpg.rf.6d13ebcb78d8caae830572469550988c.jpg** — GT:0.120 YOLO:0.120 VLM:0.120
- **al_IMG_4476_MOV-0053_jpg.rf.68ce0af445e5ff647720f75d926a6e6c.jpg** — GT:0.100 YOLO:0.100 VLM:0.630

### Nível: MEDIO

- **al_IMG_4405_MOV-0007_jpg.rf.fe90a49fd6f3510d06588977b6f607f6.jpg** — GT:0.150 YOLO:0.150 VLM:0.405
- **al_IMG_4405_MOV-0012_jpg.rf.251986543f796a895ea94607d525b911.jpg** — GT:0.375 YOLO:0.375 VLM:0.740
- **al_IMG_4405_MOV-0012_jpg.rf.b55c55f2090849527e6fdc30ae3601b5.jpg** — GT:0.375 YOLO:0.525 VLM:0.740

### Nível: ALTO

- **al_IMG_4405_MOV-0002_jpg.rf.63dbff5035bf828190cd628304672fec.jpg** — GT:0.630 YOLO:0.405 VLM:1.000
- **al_IMG_4405_MOV-0014_jpg.rf.7c649a583aa95408586221b8a3e7ecce.jpg** — GT:0.613 YOLO:0.838 VLM:1.000
- **al_IMG_4476_MOV-0002_jpg.rf.1a10fa9683584a3843b18865e90d3ba6.jpg** — GT:0.600 YOLO:0.750 VLM:1.000

### Nível: CRITICO

- **al_IMG_4405_MOV-0001_jpg.rf.391519deea8283401df5077d829a9152.jpg** — GT:1.000 YOLO:1.000 VLM:1.000
- **al_IMG_4405_MOV-0002_jpg.rf.af12b6dd3ec6f0e84254492a518c4c10.jpg** — GT:0.762 YOLO:1.000 VLM:1.000
- **al_IMG_4405_MOV-0004_jpg.rf.a41b380b5fac508b6c8bd7e0f864dce9.jpg** — GT:0.762 YOLO:0.762 VLM:0.942
