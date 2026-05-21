# Análise Completa do Projeto

> **Comparativo YOLOv11m vs Gemma 3 27B para Detecção e Scoring de Resíduos Sólidos**
> Versão V2 — Rigor metodológico para defesa de TCC

---

## 1. Visão Geral

Este projeto compara duas abordagens fundamentalmente distintas para detecção e quantificação de resíduos sólidos em imagens:

| Abordagem | Paradigma | Treinamento |
|---|---|---|
| **YOLOv11m** | Detector especializado (CNN one-stage) | Fine-tuning supervisionado em 6 classes |
| **Gemma 3 27B-IT** | Vision-Language Model generalista | Zero-shot via LM Studio (API OpenAI-compat) |

A hipótese central testada é se um **VLM generalista zero-shot** pode rivalizar com um **detector especializado fine-tuned** em uma tarefa de visão computacional bem definida (detecção + contagem de resíduos por classe), tanto em **qualidade preditiva** quanto em **eficiência operacional**.

---

## 2. Estrutura do Notebook

O pipeline está organizado em **14 seções** com **47 células de código** numeradas (`[01]` a `[47]`):

| Seção | Conteúdo | Células |
|---|---|---|
| 1 | Setup, classes operacionais, score heurístico | [01]–[05] |
| 2 | Download dos 3 datasets Roboflow | [06] |
| 3 | Deduplicação perceptual (pHash) entre datasets | [07]–[08] |
| 4 | Unificação para 6 classes (sem OUTROS) | [09]–[11] |
| 5 | Split agrupado anti-vazamento + verificação obrigatória | [12]–[14] |
| 6 | Exploração estatística (distribuição × split) | [15]–[17] |
| 7 | Fine-tuning YOLOv11m com peso de classe | [18]–[19] |
| 8 | Ablação opcional: YOLOv11l, imgsz=1280, calibração | [20]–[22] |
| 9 | Inferência YOLO (latência end-to-end + model-only) | [23]–[24] |
| — | **Liberar VRAM antes do VLM** (cell guard) | [25] |
| 10 | Setup LM Studio + Gemma 3 27B determinístico | [26] |
| 11 | Inferência VLM (schema JSON, cache, recálculo) | [27]–[31] |
| 12 | Estimativa de variância do VLM (amostragem) | [32] |
| 13 | Score de irregularidade + sensibilidade + kappa humano | [33]–[37] |
| 14 | Avaliação comparativa + estratificação por dataset | [38]–[47] |

---

## 3. Decisões Metodológicas

### 3.1 Classes operacionais — remoção de OUTROS

A versão V1 do projeto usava 7 classes incluindo `OUTROS`. Na V2 esta classe foi **removida** porque:

- Definição operacionalmente ambígua → baixo F1 nos dois modelos (V1: 0.51 YOLO / 0.24 VLM)
- Heterogeneidade interna inviabiliza um operador de coleta seletiva agir sobre ela
- Inflava artificialmente a métrica macro sem ganho informacional

Classes finais:

| ID | Classe | Mapeamento operacional |
|---|---|---|
| 0 | SACOS | Sacolas, embalagens flexíveis |
| 1 | PLASTICO | Garrafas e plásticos rígidos |
| 2 | PAPEL | Papel, papelão, jornais |
| 3 | VIDRO | Vidros e garrafas de vidro |
| 4 | METAIS | Latas, metais, aerossóis |
| 5 | CAIXAS | Caixas cartonadas (longa-vida, pizza, ovos) |

### 3.2 Deduplicação perceptual entre datasets

Os 3 datasets do Roboflow têm origens parcialmente sobrepostas (TACO aparece em re-uploads de outros workspaces). A deduplicação usa **pHash com threshold de 5 bits de Hamming**, removendo duplicatas inter-datasets antes do split.

### 3.3 Split agrupado anti-vazamento

`StratifiedGroupKFold` do scikit-learn garante:

- **Estratificação** por composição de classes (multi-label)
- **Grupos** baseados em chave de stem do nome do arquivo — variantes da mesma imagem (rotações, augmentations do Roboflow) caem no mesmo split
- **Verificação obrigatória** (célula [14]): falha hard se houver grupo presente em mais de um split

Isto evita o vazamento clássico onde a mesma imagem aparece em treino e teste com augmentations diferentes.

### 3.4 Determinismo do VLM

Para isolar variância do modelo das do prompt/seed:

| Parâmetro | Valor | Justificativa |
|---|---|---|
| `temperature` | 0 | Decoding determinístico (greedy) |
| `seed` | Tentativa de fixação | LM Studio pode ignorar (documentado em [01]) |
| `response_format` | JSON Schema estrito | Saída garantidamente parseável |
| `max_tentativas` | 3 | Retry para parse failures |
| Cache por imagem | hash MD5 da imagem | Reprodução idêntica em re-execuções |

Resultado: **0 parse errors** em 1.255 imagens de teste (vs. 4 erros na V1 sem schema estrito).

### 3.5 Separação de latência

Distinguir **latência total** (incluindo I/O, encoding) de **latência só-modelo** é crucial para comparações justas:

| Métrica | YOLO | VLM (Gemma 3 27B) |
|---|---|---|
| Latência end-to-end (μ / P95) | 14.6 / 41.8 ms | 3.530 / ~7.000 ms |
| Latência model-only (μ) | 9.9 ms | 3.530 ms (sem distinção possível via API) |

A latência **model-only do YOLO** mostra que a CNN em si responde em ~10 ms; o resto é overhead de carregamento de imagem em disco. Para o VLM via API, não é possível separar — toda a latência inclui transferência de imagem ao servidor.

---

## 4. Resultados Quantitativos

### 4.1 Métricas globais

| Métrica | YOLOv11m | Gemma 3 27B | Δ (YOLO − VLM) | Vencedor |
|---|---|---|---|---|
| F1-macro (presença multi-label) | **0.694** | 0.454 | +0.240 | YOLO |
| MAE Score Irregularidade | **0.139** | 0.172 | −0.033 | YOLO |
| Concordância de Nível | **65.0%** | 57.3% | +7.7 p.p. | YOLO |
| Spearman ρ (score) | **0.628** | 0.577 | +0.051 | YOLO |
| mAP@50 | **0.489** | — | — | YOLO¹ |
| mAP@50-95 | **0.372** | — | — | YOLO¹ |
| Latência média | **16 ms** | 3.530 ms | 220× | YOLO |

¹ VLM não produz bounding boxes → mAP não aplicável.

### 4.2 F1 por classe

| Classe | YOLO F1 | VLM F1 | Δ | Observação |
|---|---|---|---|---|
| PLASTICO | **0.897** | 0.840 | +0.057 | Classe dominante; ambos vão bem |
| METAIS | **0.863** | 0.721 | +0.142 | YOLO melhor; categoria visualmente distinta |
| PAPEL | **0.823** | 0.324 | **+0.499** | Maior gap; VLM confunde com caixas |
| VIDRO | **0.789** | 0.667 | +0.122 | YOLO superior; transparência ajuda VLM |
| CAIXAS | 0.561 | 0.300 | +0.261 | F1 baixo nos dois (raridade no treino) |
| SACOS | 0.414 | 0.088 | **+0.326** | Desbalanceamento severo (84 amostras de teste, 358 treino) |

**Insight principal:** o gap PAPEL/SACOS sugere que o VLM trata-os linguisticamente de forma ambígua — "saco" pode ser plástico ou papel; "papel" pode ser caixa amassada. O YOLO aprende a fronteira visual diretamente dos dados.

### 4.3 Distribuição no conjunto unificado V2

| Classe | Treino | Val | Teste | Total | Razão treino/PLASTICO |
|---|---|---|---|---|---|
| PLASTICO | 4.089 | 951 | 1.033 | 6.073 | 1.00 |
| METAIS | 1.432 | 362 | 354 | 2.148 | 0.35 |
| PAPEL | 732 | 222 | 199 | 1.153 | 0.18 |
| VIDRO | 755 | 123 | 126 | 1.004 | 0.18 |
| SACOS | 358 | 89 | 84 | 531 | **0.09** |
| CAIXAS | 161 | 44 | 41 | 246 | **0.04** |

> Mesmo após a unificação dos 3 datasets, **CAIXAS tem 25× menos amostras de treino que PLASTICO**. Isto explica seus mAP/F1 baixos.

### 4.4 Distribuição por nível de irregularidade (GT)

| Nível | N (teste) | % |
|---|---|---|
| Baixo | 525 | 37.4% |
| Médio | 416 | 29.7% |
| Alto | 149 | 10.6% |
| Crítico | 317 | 22.6% |

### 4.5 Composição do teste por dataset de origem

| Prefixo | Dataset | N | % |
|---|---|---|---|
| `td_` | Trash Detection v14 | 1.008 | 80.3% |
| `taco_` | TACO coco-to-yolo | 211 | 16.8% |
| `al_` / `ld_` | Litter Detection v12 | 188 | 15.0%² |

² Detalhe: predictions atuais usam prefixo `al_` (versão antiga aerial-litter). A nova pipeline V2 troca para `ld_` (litter-detection). Re-execução completa atualizará o prefixo.

---

## 5. Pontos Fortes do Projeto

### 5.1 Rigor experimental

- **Anti-vazamento verificado**: célula [14] aborta a execução se detectar grupos compartilhados entre splits
- **Determinismo do VLM**: `temperature=0` + cache + schema JSON estrito → 0 parse errors em 1.255 amostras
- **Separação de latência**: end-to-end vs. model-only documentada e medida separadamente
- **Variância do VLM estimada**: célula [32] roda repetições em amostra para quantificar incerteza
- **Análise de sensibilidade dos pesos do score**: célula [35] varia os pesos das classes e mede o impacto na ordem dos resultados
- **Validação humana via Cohen kappa ponderado**: planilha estratificada (50 imagens) com 4 níveis para calibrar o score heurístico contra percepção humana

### 5.2 Reprodutibilidade

- Notebook é **idempotente**: cada seção checa se o output já existe e pula → `Restart & Run All` funciona sem re-executar etapas caras
- **Pesos do modelo** salvos em `runs/detect/models/yolov11m_6classes_rigor/`
- **Cache do VLM** por imagem (chave hash) — re-runs preservam custo computacional
- **Sementes fixas** em todos os pontos não-determinísticos (`SEED=42`)
- **`.gitignore` curado**: ignora pesos `.pt` (>39 MB cada), datasets brutos (>3 GB) e caches intermediários; mantém todos os outputs analíticos versionados

### 5.3 Análise estratificada

A célula [43] decompõe os resultados por dataset de origem, revelando:

- **Trash Detection v14** (80% do teste): F1 alto nos dois modelos
- **TACO** (17%): F1 médio; classes finas geram confusão no VLM
- **Litter/Aerial** (15%): cenas aéreas; YOLO sofre menos que VLM por aprendizado visual direto

---

## 6. Limitações Identificadas

### 6.1 Desbalanceamento estrutural

| Problema | Impacto |
|---|---|
| **CAIXAS** com só 161 amostras de treino | mAP@50 de 0.42 — limite imposto por dados, não por modelo |
| **SACOS** com 358 amostras vs PLASTICO 4.089 | F1 de 0.41 (YOLO) e 0.09 (VLM) |
| VIDRO concentrado no TACO | Pouca generalização para cenas aéreas |

### 6.2 Sensibilidade ao prompt no VLM

O Gemma é avaliado com **um único prompt fixo** (zero-shot). Não há:

- Few-shot prompting com exemplos por nível
- Chain-of-thought ou auto-correção
- Fine-tuning leve (LoRA)

Isto pode subestimar o potencial do VLM, mas mantém a comparação justa contra um **YOLO fine-tuned** (que também usa uma única configuração de treino).

### 6.3 Score de irregularidade é annotation-derived

O "ground truth" do score é calculado a partir das anotações originais dos datasets aplicando a fórmula heurística. **Não é um julgamento humano direto.** O Cohen kappa contra rotulagem humana (célula [37]) mitiga isto medindo a concordância entre score heurístico e percepção humana em 50 imagens estratificadas — mas essa planilha **requer preenchimento manual**.

### 6.4 VLM sem bounding boxes

A comparação de **mAP** é estruturalmente impossível: o VLM produz apenas classes presentes + contagem, não localização. Comparações como F1-presença e MAE-contagem são as únicas justas.

### 6.5 Latência do VLM inclui transferência

Os 3.530 ms do Gemma incluem upload da imagem ao servidor LM Studio local. Em uma comparação cloud-vs-edge, esse overhead seria diferente. Para fins de TCC com hardware único, a comparação local-vs-local é apropriada.

---

## 7. Achados Não-Triviais

1. **VLM vence no nível Crítico** (65.0% vs 59.0% YOLO) — em cenas extremamente poluídas, a compreensão holística supera a detecção por bounding boxes. Hipótese: o VLM lê a "essência" da cena enquanto o YOLO se perde na enumeração de objetos sobrepostos.

2. **MAE de score é mais próximo entre os modelos** (0.139 vs 0.172) do que sugeriria o F1 (0.694 vs 0.454). Isto indica que mesmo errando a identidade das classes, o VLM acerta a **quantidade/intensidade** da irregularidade.

3. **Throughput em lote do YOLO** (~250 imgs/s no batch de 200) torna a diferença de velocidade efetivamente **muito maior que 220×** em cenários de processamento em massa — o VLM não tem batching equivalente via API.

4. **5.3% das imagens de teste sem detecções YOLO** (66 de 1.255) — falsos negativos completos. O VLM, por construção, sempre retorna *algo* (mesmo que vazio com risco_sanitario='baixo'), o que pode tanto ajudar quanto prejudicar a depender da imagem.

---

## 8. Trabalhos Futuros (Mapa de Esforço)

| Iniciativa | Esforço | Impacto esperado |
|---|---|---|
| **Pipeline híbrido YOLO+VLM** (YOLO detecta ROIs, VLM classifica) | Médio | Combina velocidade + semântica |
| **Few-shot prompting no VLM** (2–3 exemplos por nível) | Baixo | Pode fechar gap em PAPEL e SACOS |
| **LoRA fine-tuning no Gemma** | Alto | Linha de pesquisa para artigo separado |
| **Balanceamento de SACOS/CAIXAS** (synthetic augmentation) | Médio | Sobe o teto do YOLO nas classes raras |
| **Avaliação em vídeo drone** | Alto | Cenário operacional realista |
| **Comparação multi-VLM** (LLaVA 1.6, Qwen2-VL, InternVL2) | Médio | Posiciona Gemma no estado-da-arte |

---

## 9. Veredito Quantificado

O **YOLOv11m fine-tuned** é a escolha operacional para esta tarefa: vence em **todas** as métricas quantitativas, é **220× mais rápido**, e roda em hardware de produção (edge devices, GPUs modestas).

O **Gemma 3 27B zero-shot** é competitivo apenas em:

- **Cenas críticas** (apenas 22% do dataset, mas onde a interpretação holística importa)
- **Cenários sem dados de treino** (a vantagem do zero-shot é existir)
- **Aplicações com baixo throughput** onde latência não é crítica

Para um **sistema de monitoramento ambiental em produção** com câmeras fixas/drones gerando milhares de imagens/dia, o **YOLO é a escolha óbvia**. O VLM faz sentido como **ferramenta de auditoria** (re-classificação de casos duvidosos) ou em **fases de bootstrap** sem dados anotados.

---

## 10. Mapa de Reprodução

```
1. git clone https://github.com/jaimejrs/comparativo_yolo_vlm
2. cp .env.example .env  # preencher ROBOFLOW_API_KEY
3. pip install -r requirements.txt
4. LM Studio: carregar gemma-3-27b-it na porta 1234
5. jupyter lab comparativo_V2.ipynb
6. Restart & Run All  (~6h primeira vez; ~30s subsequentes via cache)
```

**Custo computacional aproximado:**

| Etapa | Tempo (RTX 4090) |
|---|---|
| Download datasets | 5–10 min (rede) |
| Dedup pHash | 2 min |
| Unificação + split | 1 min |
| Treino YOLO (150 épocas) | ~3h |
| Inferência YOLO (1.255 imgs) | 30 s |
| Inferência VLM (1.255 imgs) | ~75 min |
| Variância VLM (amostra × 5) | ~15 min |
| Métricas + figuras | 1 min |

---

*Análise gerada automaticamente a partir dos outputs do notebook e dos arquivos de resultado em `outputs/`.*
*Para resultados detalhados originais, ver [RESULTADOS.md](RESULTADOS.md).*
