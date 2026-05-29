---
name: tarefa3-stitching-design
description: Design do Tarefa3.ipynb — Image Stitching com SIFT + RANSAC, comparação com ORB
metadata:
  type: project
---

# Design — Tarefa 3: Montagem de Panorama (Image Stitching)

## Contexto

Parte da Atividade 3 (TECA2 20261). Entrega via Google Classroom até 30/05/2026. Arquivo de entrega: `Tarefa3.ipynb`. Apresentação fica por conta do aluno.

## Par de imagens

- `images/mesa_esq.jpeg` e `images/mesa_dir.jpeg` — par real fotografado com celular (mesa de trabalho, deslocamento lateral).

## Estrutura do notebook (Opção A — espelha Tarefa1 e Tarefa2)

### Cabeçalho e base teórica (Markdown)
- Título, alunos, descrição.
- **Homografia**: transformação projetiva 3×3 que mapeia pontos de um plano de imagem para outro, preservando linhas retas. Para montar um panorama, usamos H para reprojetar (`warp`) img1 no sistema de coordenadas de img2.
- **RANSAC**: estima H usando apenas os inliers (correspondências consistentes), descartando outliers automaticamente. Itera sorteando subconjuntos mínimos de 4 pares e escolhe o modelo com mais suporte.
- **USAC_MAGSAC**: variante moderna (`cv2.USAC_MAGSAC` dentro de `cv2.findHomography`, OpenCV ≥ 4.5) — mais robusta a outliers que o RANSAC clássico.

### Setup (código)
- Imports: `cv2`, `numpy`, `matplotlib.pyplot`.
- Prints de versão.
- `PATH_ESQ = 'images/mesa_esq.jpeg'`, `PATH_DIR = 'images/mesa_dir.jpeg'`.
- Função `load(path)` → `(img_bgr, gray)`.

### Item 1 — Par de imagens (código + markdown)
- Carrega e exibe as duas fotos lado a lado em RGB.
- Imprime dimensões.

### Item 2 — Pipeline SIFT completo (4 subcélulas de código + markdown)

**2a — Detectar e descrever:**
```python
sift = cv2.SIFT_create()
kp1, d1 = sift.detectAndCompute(gray1, None)
kp2, d2 = sift.detectAndCompute(gray2, None)
print(f'SIFT img1: {len(kp1)} kp | img2: {len(kp2)} kp')
```

**2b — Matching com Ratio Test (ratio=0.75):**
```python
bf = cv2.BFMatcher(cv2.NORM_L2)
pairs = bf.knnMatch(d1, d2, k=2)
good = [m for m, n in pairs if m.distance < 0.75 * n.distance]
print(f'Bons matches: {len(good)}')
```

**2c — Homografia com USAC_MAGSAC:**
```python
src = np.float32([kp1[m.queryIdx].pt for m in good]).reshape(-1, 1, 2)
dst = np.float32([kp2[m.trainIdx].pt for m in good]).reshape(-1, 1, 2)
M, mask = cv2.findHomography(src, dst, cv2.USAC_MAGSAC, 5.0)
inliers = int(mask.ravel().sum())
print(f'Inliers: {inliers} ({100*inliers/len(good):.1f}%)')
```
Guard: if `len(good) < 10` → print mensagem e interromper.

**2d — Warp e composição simples:**
```python
h1, w1 = img1.shape[:2]
h2, w2 = img2.shape[:2]
result = cv2.warpPerspective(img1, M, (w1 + w2, max(h1, h2)))
result[0:h2, 0:w2] = img2
plt.figure(figsize=(16, 6))
plt.imshow(cv2.cvtColor(result, cv2.COLOR_BGR2RGB))
plt.title('Panorama SIFT')
plt.axis('off')
plt.tight_layout()
plt.show()
```

Markdown após cada subcélula: explicação curta do que foi feito (≤2 linhas).

### Item 3 — Pipeline ORB (código + markdown)
- Mesmo pipeline, substituindo SIFT por ORB (NORM_HAMMING, nfeatures=2000, ratio=0.75).
- Exibe panorama ORB.
- Imprime KPs, matches, inliers.

### Item 4 — Tabela comparativa e discussão (código + markdown)
Código imprime valores reais; markdown preenche a tabela e discute:

| Detector | KP (img1) | Matches | Inliers (%) | OK? |
|----------|-----------|---------|-------------|-----|
| SIFT     | —         | —       | —           | —   |
| ORB      | —         | —       | —           | —   |

Discussão: panorama ficou bom nos dois? Onde há diferença visível? Quando preferiria ORB (sistema embarcado/tempo real)?

### Conclusão (Markdown)
- Qualidade do panorama SIFT vs ORB.
- Limitações da composição simples (sem blending, sem correção de translação negativa).
- Inlier rate como indicador de qualidade do stitching.

## Critérios de avaliação cobertos

1. **Fundamentação** — homografia, RANSAC, USAC_MAGSAC na base teórica.
2. **Implementação** — pipeline funcional com warp e composição; panorama gerado e exibido.
3. **Análise Comparativa** — tabela SIFT×ORB preenchida com evidências; diferenças discutidas.
4. **Apresentação** — panoramas exibidos com zoom na região de junção; conclusões fundamentadas.

## Fora de escopo

- `Tarefa3.pptx` — o aluno fará ao final.
- Blending avançado (alpha blending, multi-band) — não requerido pelo enunciado.
- Correção automática de translação negativa — mencionada como limitação na conclusão.
