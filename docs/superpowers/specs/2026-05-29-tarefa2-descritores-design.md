---
name: tarefa2-descritores-design
description: Design do Tarefa2.ipynb — Descritores Float vs. Binários, comparação SIFT/ORB/AKAZE com Ratio Test
metadata:
  type: project
---

# Design — Tarefa 2: Descritores Float vs. Binários

## Contexto

Parte da Atividade 3 (TECA2 20261). Continuação do Tarefa1.ipynb. Entrega via Google Classroom até 30/05/2026. Arquivo de entrega: `Tarefa2.ipynb` (apresentação fica por conta do aluno).

## Par de imagens

- `images/ursinho1.jpeg` e `images/ursinho2.jpeg` — par real fotografado com celular.

## Estrutura do notebook (Opção A — espelha Tarefa1)

### Cabeçalho e base teórica (Markdown)
- Título, alunos, descrição da tarefa.
- Conceito-chave: float (SIFT, 128 floats) → NORM_L2 (distância euclidiana); binário (ORB, AKAZE) → NORM_HAMMING (contagem de bits diferentes).
- Ratio Test de Lowe: para cada keypoint, os 2 melhores candidatos; aceita match se `d1 < ratio * d2`. Valor canônico 0.75 elimina ~90% dos falsos sem descartar os verdadeiros.

### Setup (código)
- Imports: `cv2`, `numpy`, `matplotlib.pyplot`.
- Verificação de versões e disponibilidade de SIFT/AKAZE.
- Constantes: `PATH_IMG1 = 'images/ursinho1.jpeg'`, `PATH_IMG2 = 'images/ursinho2.jpeg'`.
- Função `load(path)` → `(img_bgr, gray)`.

### Item 1 — Par de imagens (código + markdown)
- Carrega e exibe as duas fotos lado a lado em RGB.
- Mostra dimensões de cada imagem.
- Sem processamento — confirmação visual do par de testes.

### Item 2 — Pipeline de matching (código + markdown)
- Função `pipeline_match(img1, img2, det_name='SIFT', ratio=0.75)`:
  - Instancia detector e norma correta por `det_name` (`SIFT`→NORM_L2, `ORB`→NORM_HAMMING, `AKAZE`→NORM_HAMMING).
  - `detectAndCompute` em ambas as imagens.
  - `BFMatcher.knnMatch(d1, d2, k=2)`.
  - Ratio Test: aceita par `(m, n)` se `len(pair)==2` e `m.distance < ratio * n.distance`.
  - Imprime: `{det_name}: {len(kp1)} kp | {n_ok}/{n_tot} bons`.
  - Retorna `kp1, kp2, good`.
- Roda para SIFT, ORB e AKAZE com `ratio=0.75`.
- Exibe `cv2.drawMatches` para cada descritor (3 figuras separadas).

### Item 3 — Efeito do Ratio Test (código + markdown)
- Loop sobre ratios `[0.50, 0.65, 0.75, 0.90]` usando SIFT.
- Para cada limiar: chama `pipeline_match`, conta bons matches, exibe matches com `drawMatches`.
- Objetivo: demonstrar trade-off quantidade × qualidade visualmente.
- Análise em Markdown: o que acontece com matches ruins ao aumentar o limiar?

### Item 4 — Tabela comparativa (markdown)
- Preenche a tabela do enunciado com valores reais obtidos no Item 2:

| Método | Distância   | KPs  | Match (%) |
|--------|-------------|------|-----------|
| SIFT   | L2 (float)  | —    | —         |
| ORB    | Hamming (bin.) | — | —         |
| AKAZE  | Hamming (bin.) | — | —         |

- Análise crítica: diferença prática entre float e binário; quando preferir cada um.

### Conclusão (markdown)
- Qual método teve mais matches bons absolutos.
- Qual teve maior proporção (Match %).
- Recomendação contextualizada: sistema embarcado (baixo custo computacional) → ORB/AKAZE; precisão → SIFT.

## Critérios de avaliação cobertos

1. **Fundamentos** — explicação de float vs. binário e métricas de distância na base teórica.
2. **Implementação** — `pipeline_match` correto para os 3 detectores com BFMatcher adequado.
3. **Análise Crítica** — tabela preenchida + efeito do Ratio Test discutido com evidências.
4. **Apresentação** — figuras com matches visualizados; notebook organizado com legendas e conclusões.

## Fora de escopo

- `Tarefa2.pptx` — o aluno fará ao final.
- Tarefa 3 (Image Stitching) e Tarefa 4 (DISK/Kornia) — notebooks separados.
