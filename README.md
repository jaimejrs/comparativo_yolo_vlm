# Comparativo YOLO vs VLM para Detecção de Resíduos

**TCC — Engenharia / Ciência da Computação**

Comparação metodológica entre **YOLOv11m fine-tuned** e **Gemma 3 27B-IT (zero-shot via LM Studio)** para detecção e scoring de irregularidades de resíduos sólidos em imagens.

---

## Resultados Principais

| Métrica | YOLOv11m | Gemma 3 27B | Vencedor |
|---|---|---|---|
| F1-macro | **0.694** | 0.454 | YOLO |
| MAE Score Irregularidade | **0.139** | 0.172 | YOLO |
| Concordância de Nível | **65.0%** | 57.3% | YOLO |
| Spearman (score) | **0.628** | 0.577 | YOLO |
| mAP@50 | **0.489** | — | YOLO |
| Latência média | **16 ms** | 3.530 ms | YOLO (~220× mais rápido) |

> O YOLO supera o VLM em todas as métricas quantitativas. A VLM apresenta vantagem apenas em cenas de nível **Crítico**, onde a compreensão semântica holística supera a detecção por bounding boxes.

---

## Estrutura do Projeto

```
comparativo_yolo_vlm/
├── comparativo_V2.ipynb        # Notebook principal (pipeline completo)
├── TCC_YOLO_vs_VLM.ipynb       # Notebook versão 1 (referência)
├── requirements.txt
├── .env.example                # Template de variáveis de ambiente
├── RESULTADOS.md               # Resultados detalhados com análise qualitativa
├── data/
│   ├── class_mapping_v2.json   # Mapeamento das 6 classes operacionais
│   └── class_mapping.json      # Mapeamento versão 1
└── outputs/
    ├── figures_v2/             # Figuras geradas (curvas, scatter, boxplots)
    ├── predictions_yolo_v2.json
    ├── predictions_vlm.json
    ├── scores_irregularidade.json
    ├── distribuicao_v2.csv
    ├── results_summary.csv
    └── tabela_capitulo3.tex
```

---

## Classes Operacionais

| ID | Classe | Descrição |
|---|---|---|
| 0 | SACOS | Sacolas e embalagens flexíveis |
| 1 | PLASTICO | Garrafas e plásticos rígidos |
| 2 | PAPEL | Papel, papelão, caixas |
| 3 | VIDRO | Vidros e garrafas de vidro |
| 4 | METAIS | Latas, metais e aerossóis |
| 5 | CAIXAS | Caixas e embalagens cartonadas |

---

## Datasets

Três datasets do Roboflow, unificados nas 6 classes acima:

| Dataset | Classes originais | Imagens (aprox.) |
|---|---|---|
| [Trash Detection v14](https://universe.roboflow.com/trash-dataset-for-oriented-bounded-box/trash-detection-1fjjc) | 5 | ~3.000 |
| [Litter Detection v12](https://universe.roboflow.com/chens-workspace-elib6/litter-detection-bcnrf) | 5 | ~4.000 |
| [TACO coco-to-yolo](https://universe.roboflow.com/new-workspace-am0dh/coco-to-yolo-2m8ut) | 59 | ~5.000 |

> Os datasets brutos não estão versionados (>3 GB). O notebook os baixa automaticamente via Roboflow API.

---

## Reprodução

### 1. Requisitos

- Python 3.11
- CUDA 12.1+ (GPU com ≥24 GB VRAM recomendado para rodar YOLO + Gemma)
- [LM Studio](https://lmstudio.ai/) com **Gemma 3 27B-IT** carregado

### 2. Instalação

```bash
git clone https://github.com/jaimejrs/comparativo_yolo_vlm
cd comparativo_yolo_vlm

python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### 3. Configurar credenciais

```bash
cp .env.example .env
# edite .env com sua chave do Roboflow
```

### 4. LM Studio

1. Abra o LM Studio e carregue o modelo **gemma-3-27b-it**
2. Inicie o servidor local em `http://localhost:1234`

### 5. Executar

Abra `comparativo_V2.ipynb` e execute todas as células em ordem (`Restart & Run All`).

O notebook gerencia automaticamente:
- Download dos datasets via Roboflow
- Deduplicação perceptual (pHash)
- Unificação e split agrupado anti-vazamento
- Fine-tuning do YOLOv11m (pula se pesos já existirem)
- Inferência YOLO + liberação de VRAM
- Inferência VLM via LM Studio (com cache por imagem)
- Métricas comparativas e figuras

---

## Hardware Utilizado

| Componente | Especificação |
|---|---|
| GPU | NVIDIA RTX 4090 (24 GB VRAM) |
| YOLO fine-tuning | ~150 épocas, batch 12, imgsz 640 |
| VLM inferência | Gemma 3 27B-IT, temperature 0, via LM Studio |

---

## Principais Achados

- **PAPEL** e **SACOS** são as classes com maior diferença entre YOLO e VLM (ΔF1 de +0.50 e +0.33 respectivamente)
- **SACOS** apresenta desempenho baixo em ambos os modelos — reflexo do desbalanceamento severo (59 amostras de treino vs. 6.578 de PLÁSTICO)
- Em cenas de nível **Crítico**, o VLM supera o YOLO em concordância de nível (65% vs. 59%), sugerindo que a compreensão semântica holística é vantajosa em cenários extremos
- A latência do VLM (3.530 ms) inviabiliza uso em tempo real; YOLO (16 ms) é viável para aplicações embarcadas

---

## Citação

```bibtex
@misc{ribeiro2026yolo_vlm,
  author  = {Jaime Ribeiro},
  title   = {Comparativo YOLOv11m vs Gemma 3 27B para Detecção de Resíduos},
  year    = {2026},
  url     = {https://github.com/jaimejrs/comparativo_yolo_vlm}
}
```
