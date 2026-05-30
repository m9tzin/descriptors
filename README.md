# Atividade 3 — Pontos de Interesse e Descritores

> Visão Computacional (TECA2 20261) · UFG · 2026/1
>
> **Alunos:** Henryque Oliveira · Matheus Marinho · Rodrigo Oliveira

O enunciado completo está em [`docs/Atividade3_TECA2_20261_rev.pdf`](docs/Atividade3_TECA2_20261_rev.pdf).

---

## Tarefas

### Tarefa 1 — Espaço de Escala, Visualização e Análise de Descritores
[`Tarefa1.ipynb`](Tarefa1.ipynb)

Construção manual da Pirâmide Gaussiana e DoG; visualização rica de keypoints SIFT e AKAZE em 3 imagens; análise do descritor de 128 dimensões; efeito da escala na detecção SIFT.

| Imagem | SIFT (kp) | AKAZE (kp) |
|--------|-----------|------------|
| Prédio | 9.711 | 6.368 |
| Mona Lisa | 815 | 277 |
| Fig0438 | 686 | 264 |

Invariância a escala (Mona Lisa): 815 kp → 240 kp → 138 kp (100% → 50% → 25%)

---

### Tarefa 2 — Descritores Float vs. Binários: Comparação Prática
[`Tarefa2.ipynb`](Tarefa2.ipynb)

Pipeline de matching completo para SIFT, ORB e AKAZE no par ursinho. Análise do Ratio Test de Lowe com 4 limiares diferentes.

| Método | Tipo | KPs | Matches (ratio=0.75) | Match (%) |
|--------|------|-----|---------------------|-----------|
| SIFT | Float / L2 | 5.535 | 97 | 1.8% |
| ORB | Binário / Hamming | 2.000 | 17 | 0.8% |
| AKAZE | Binário / Hamming | 1.636 | 36 | 2.2% |

Efeito do ratio test (SIFT): 5 matches (r=0.50) → 35 (r=0.65) → 97 (r=0.75) → 993 (r=0.90)

---

### Tarefa 3 — Montagem de Panorama (Image Stitching)
[`Tarefa3.ipynb`](Tarefa3.ipynb)

Pipeline completo de stitching: detecção → Ratio Test → homografia com USAC_MAGSAC → warp perspectivo → composição simples. Comparação SIFT vs ORB no par mesa.

| Detector | KP | Matches | Inliers | Taxa | Qualidade |
|----------|----|---------|---------|------|-----------|
| SIFT | 2.377 | 336 | 110 | 32.7% | ❌ |
| ORB | 2.000 | 383 | 235 | 61.4% | ✅ |

ORB superou SIFT neste par (critério: taxa de inliers > 60%).

---

### Tarefa 4 — Descritores Neurais com Kornia (DISK)
[`Tarefa4.ipynb`](Tarefa4.ipynb)

Exploração do DISK (DIscrete Keypoints) via Kornia: modelo neural treinado por reforço que aprende detector e descritor conjuntamente. Comparação com os métodos clássicos no par ursinho.

| Método | Tipo | KP | Matches | Ratio |
|--------|------|----|---------|-------|
| SIFT | Clássico / float | 5.535 | 97 | 0.75 |
| ORB | Clássico / binário | 2.000 | 17 | 0.75 |
| AKAZE | Clássico / binário | 1.636 | 36 | 0.75 |
| **DISK** | **Neural / float** | **2.048** | **58** | **0.90** |

Execução local: CPU · PyTorch 2.12.0+cpu · Kornia 0.8.3

---

## Estrutura do repositório

```
.
├── Tarefa1.ipynb           # T1 — Espaço de escala, pirâmide, DoG, SIFT/AKAZE
├── Tarefa2.ipynb           # T2 — Float vs. binário, Ratio Test, matching
├── Tarefa3.ipynb           # T3 — Stitching, homografia, RANSAC
├── Tarefa4.ipynb           # T4 — DISK (Kornia), descritores neurais
├── images/                 # Imagens de entrada
│   ├── ursinho1.jpeg       # Par T2 e T4
│   ├── ursinho2.jpeg
│   ├── mesa_esq.jpeg       # Par T3
│   ├── mesa_dir.jpeg
│   ├── building2-2.png     # T1
│   ├── monalisa.png        # T1
│   └── Fig0438_a__bld_...  # T1
├── results/                # Imagens de resultado extraídas dos notebooks (22 figuras)
├── docs/
│   └── Atividade3_TECA2_20261_rev.pdf  # Enunciado completo
├── pyproject.toml
└── uv.lock
```

---

## Ambiente

**Requisitos:** Python 3.12+ · [uv](https://docs.astral.sh/uv/)

**Dependências principais:** OpenCV 4.13 · PyTorch 2.12 (CPU) · Kornia 0.8.3 · NumPy · Matplotlib

```bash
# Instalar dependências
uv sync

# Abrir Jupyter
uv run jupyter notebook
```

**Google Colab:** os notebooks incluem célula de instalação automática de Kornia. Para a Tarefa 4, faça upload de `ursinho1.jpeg` e `ursinho2.jpeg` e ajuste os paths conforme indicado no notebook.

---

## Resultados

[`results/`](results/) contém 22 figuras extraídas dos outputs dos notebooks (T1–T4).
