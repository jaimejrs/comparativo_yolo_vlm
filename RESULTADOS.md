# Resultados — TCC: YOLO vs VLM para Detecção de Resíduos

> Gerado em: 2026-05-20  
> Modelos: **YOLOv11m** (fine-tuned, 7 classes) vs **Gemma 3 27B-IT** (zero-shot via LM Studio)  
> Dataset de teste: **1.407 imagens** (TACO + Aerial Litter Detection + Trash Detection)

---

## 1. Configuração Experimental

| Item | Detalhe |
|---|---|
| Modelo YOLO | YOLOv11m, fine-tuned, 7 classes |
| Modelo VLM | Gemma 3 27B-IT (zero-shot, sem fine-tuning) |
| Classes | SACOS, VIDRO, PLÁSTICO, PAPEL, CAIXAS, METAIS, OUTROS |
| Imagens de teste | 1.407 |
| Hardware inferência YOLO | GPU local |
| Hardware inferência VLM | RTX 4090 (LM Studio) |

---

## 2. Métricas Globais

| Métrica | YOLO | VLM | Vencedor |
|---|---|---|---|
| **F1-macro** | **0.694** | 0.454 | YOLO |
| **MAE Score Irregularidade** | **0.139** | 0.172 | YOLO |
| **Concordância de Nível** | **65.0%** | 57.3% | YOLO |
| **Spearman (score)** | **0.628** | 0.577 | YOLO |
| **mAP@50** | 0.489 | — | YOLO |
| **mAP@50-95** | 0.372 | — | YOLO |
| **Latência média** | **16 ms** | 3.530 ms | YOLO (~220× mais rápido) |
| Parse errors VLM | — | 4 (0,3%) | — |

> O YOLO supera a VLM em todas as métricas quantitativas, com diferença especialmente expressiva em velocidade (220× mais rápido).

---

## 3. F1 por Classe

| Classe | YOLO F1 | VLM F1 | Δ |
|---|---|---|---|
| PLÁSTICO | **0.897** | 0.840 | +0.057 |
| METAIS | **0.863** | 0.721 | +0.142 |
| PAPEL | **0.823** | 0.324 | **+0.499** |
| VIDRO | **0.789** | 0.667 | +0.122 |
| CAIXAS | **0.561** | 0.300 | +0.261 |
| OUTROS | **0.508** | 0.239 | +0.269 |
| SACOS | **0.414** | 0.088 | **+0.326** |

**Observações:**
- PAPEL e SACOS são as classes onde a VLM mais perde para o YOLO.
- PLÁSTICO é a classe melhor predita por ambos os modelos.
- SACOS tem F1 baixo em ambos — possivelmente por sub-representação no dataset (apenas 95 amostras de treino no TACO, zero nos outros datasets).

---

## 4. mAP por Classe (YOLO)

| Classe (idx) | mAP@50 |
|---|---|
| PLASTICO | 0.591 |
| METAIS | 0.591 |
| PAPEL | 0.517 |
| CAIXAS | 0.424 |
| VIDRO | 0.331 |
| OUTROS | 0.088 |
| SACOS | 0.061 |

> SACOS e OUTROS têm mAP@50 muito abaixo da média (0.489 global). SACOS em particular sugere problema estrutural de dados.

---

## 5. Score de Irregularidade por Nível

| Nível | Total | YOLO Acurácia | VLM Acurácia |
|---|---|---|---|
| **Baixo** | 525 | **79,1%** | 73,0% |
| **Médio** | 416 | **63,9%** | 44,5% |
| **Alto** | 149 | **31,5%** | 21,5% |
| **Crítico** | 317 | 59,0% | **65,0%** |

**Destaque:** A VLM supera o YOLO apenas no nível **Crítico** — sugerindo que, em cenas extremamente poluídas, a linguagem natural do VLM captura melhor a severidade. Nos demais níveis, YOLO vence.

---

## 6. Análise Qualitativa — Casos de Maior Discordância

Os casos com maior discordância (ΔScore = 1.0) revelam dois padrões opostos:

**Padrão A — VLM superestima, YOLO subestima:**
- Imagens com baixa contagem de objetos mas cena visualmente suja → VLM atribui score alto, YOLO não detecta nada.
- Exemplo: `taco_000028` (GT=0.12, YOLO=0.00, VLM=1.00)

**Padrão B — YOLO acerta, VLM falha:**
- Imagens com objetos bem definidos e isolados → YOLO detecta corretamente, VLM ignora.
- Exemplo: `taco_000048` (GT=1.00, YOLO=1.00, VLM=0.00)

---

## 7. Distribuição do Dataset

| Classe | Treino (total) | Teste (total) |
|---|---|---|
| PLÁSTICO | 6.578 | 1.408 |
| OUTROS | 3.449 | 802 |
| METAIS | 3.386 | 772 |
| PAPEL | 2.356 | 565 |
| CAIXAS | 166 | 38 |
| VIDRO | 741 | 257 |
| SACOS | 59 | 18 |

> **Desbalanceamento severo**: SACOS tem ~59 amostras de treino vs ~6.578 de PLÁSTICO (razão 1:111). Isso explica diretamente o baixo desempenho em SACOS.

---

## 8. Pontos de Melhoria

### 8.1 Dataset

- [ ] **Balancear SACOS e CAIXAS**: coletar ou sintetizar (data augmentation) amostras para atingir pelo menos 500 instâncias de treino cada.
- [ ] **Revisar a classe OUTROS**: definição ambígua → baixo F1 em ambos os modelos. Considerar subdividir ou eliminar.
- [ ] **Aumentar diversidade de VIDRO**: dataset atual concentrado no TACO, sem representação nos datasets aéreos.

### 8.2 Modelo YOLO

- [ ] **Treinar com pesos de classe** (`cls_pw`) para compensar o desbalanceamento.
- [ ] **Testar YOLOv11l/x**: o modelo `m` tem capacidade limitada para classes raras — o modelo maior pode melhorar mAP em SACOS e OUTROS.
- [ ] **Aumentar resolução de entrada** (ex.: 1280px): objetos pequenos em imagens aéreas podem se beneficiar.
- [ ] **Focal Loss com gamma maior** para classes raras.

### 8.3 VLM (Gemma 3 27B)

- [ ] **Melhorar o prompt de contagem**: os 4 parse errors e a tendência de superestimar o score em nível Crítico indicam que o prompt precisa de instruções mais estritas de formato.
- [ ] **Few-shot prompting**: incluir 2–3 exemplos no prompt por nível de irregularidade para calibrar o VLM.
- [ ] **Testar fine-tuning (LoRA)**: mesmo um fine-tuning leve pode aproximar o VLM do YOLO sem perder a flexibilidade linguística.
- [ ] **Explorar modelos menores** (Gemma 3 4B, Phi-4): latência de 3,5 s por imagem inviabiliza uso em tempo real; um modelo menor com prompt otimizado pode ser competitivo.

### 8.4 Avaliação

- [ ] **Métrica de irregularidade weighted**: o score atual trata todas as classes com igual peso — considerar pesos por tipo de resíduo (ex.: VIDRO > peso ambiental > PAPEL).
- [ ] **Análise de erro por dataset de origem**: separar resultados TACO vs Aerial vs Trash Detection para entender quais domínios penalizam cada modelo.
- [ ] **Estudo de calibração de confiança**: o limiar padrão (conf=0.25) do YOLO pode estar subestimando detecções em cenas densas.

### 8.5 Trabalhos Futuros

- [ ] **Pipeline híbrido**: usar YOLO para detecção rápida de bounding boxes + VLM para classificação semântica fina apenas nas ROIs — combinaria velocidade do YOLO com capacidade descritiva do VLM.
- [ ] **Avaliação em vídeo**: o dataset atual é estático; testar em sequências de vídeo drone com tracking.
- [ ] **Comparação com outros VLMs**: LLaVA 1.6, InternVL2, Qwen2-VL como baselines adicionais.

---

## 9. Conclusão

O **YOLOv11m fine-tuned** supera o **Gemma 3 27B zero-shot** em todas as métricas quantitativas no contexto de detecção e scoring de irregularidades de resíduos. A diferença mais expressiva é na **velocidade** (220× mais rápido) e no **F1-macro** (+0.24). A VLM demonstra vantagem apenas em cenas de nível Crítico, onde sua compreensão semântica holística supera o YOLO na classificação do nível de irregularidade.

O principal gargalo do YOLO é o **desbalanceamento de classes** (SACOS, CAIXAS, OUTROS), enquanto o principal gargalo da VLM é a **latência e a sensibilidade ao prompt**.

---

*Figuras disponíveis em `outputs/figures/`. Análise qualitativa detalhada em `outputs/error_analysis.md`.*
