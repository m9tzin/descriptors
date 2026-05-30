---
marp: true
theme: default
paginate: true
style: |
  section {
    font-family: 'Segoe UI', sans-serif;
    font-size: 22px;
  }
  h1 { color: #1a3a5c; font-size: 1.8em; }
  h2 { color: #1a3a5c; border-bottom: 2px solid #1a3a5c; padding-bottom: 6px; }
  h3 { color: #2e6da4; }
  table { width: 100%; font-size: 0.9em; }
  th { background: #1a3a5c; color: white; }
  tr:nth-child(even) { background: #f0f4f8; }
  .highlight { background: #fff3cd; padding: 4px 8px; border-radius: 4px; }
  code { background: #f0f4f8; padding: 2px 6px; border-radius: 3px; }
  blockquote { border-left: 4px solid #2e6da4; background: #f0f4f8; padding: 8px 12px; }
---

# Atividade 3 — Pontos de Interesse e Descritores

**TECA2 20261**

Henryque Oliveira · Matheus Marinho · Rodrigo Oliveira

---

## Agenda

1. **Tarefa 1** — Espaço de Escala, Pirâmide Gaussiana e DoG
2. **Tarefa 2** — Descritores Float vs. Binários: Comparação Prática
3. **Tarefa 3** — Montagem de Panorama (Image Stitching)
4. **Tarefa 4** — Descritores Neurais com Kornia (DISK)

---

<!-- _class: lead -->
# Tarefa 1
## Espaço de Escala, Visualização e Análise de Descritores

---

## T1 — Conceitos fundamentais

**Espaço de escala**
Família de imagens borradas com σ crescente — detecta a mesma estrutura independente do tamanho.

**Pirâmide Gaussiana**
- **Escalas**: dentro de cada oitava, σ cresce → mais borramento
- **Oitavas**: resolução cai pela metade entre oitavas

**DoG (Diferença de Gaussianas)**
Aproximação eficiente do Laplaciano da Gaussiana:

> `DoG = G(kσ) − G(σ)` → realça blobs e bordas em cada escala

Keypoints SIFT = **extremos locais do DoG** em 26 vizinhos (espaço + escala)

---

## T1 — Resultado: Pirâmide e DoG

**3 oitavas · 5 escalas por oitava · par Gaussiana + DoG gerado manualmente**

| Oitava | Resolução | σ cresce → | Estruturas visíveis |
|--------|-----------|-----------|-------------------|
| 1 | 100% | baixo → alto | textura fina, janelas |
| 2 | 50% | baixo → alto | bordas, cantos |
| 3 | 25% | baixo → alto | estruturas grandes |

> Estruturas finas respondem nas oitavas iniciais · estruturas grandes nas finais

---

## T1 — Resultado: Keypoints SIFT vs AKAZE

**3 imagens · flag `DRAW_RICH_KEYPOINTS` (tamanho = escala, raio = orientação)**

| Imagem | SIFT (kp) | AKAZE (kp) |
|--------|-----------|------------|
| Prédio (building2) | **9.711** | 6.368 |
| Mona Lisa | **815** | 277 |
| Fig0438 (Gonzalez) | **686** | 264 |

- SIFT detecta mais keypoints em todas as imagens — menos seletivo
- AKAZE é mais restritivo (espaço de escala não-linear)
- Regiões planas (céu) → gradiente ≈ 0 → **nenhum keypoint**

---

## T1 — Resultado: Descritor SIFT de 128 dimensões

**Keypoint analisado: escala 2.62 · orientação 353.8°**

```
Janela 16×16 → 4×4 sub-regiões → 8 direções por sub-região
4 × 4 × 8 = 128 valores (floats)
```

- Cada grupo de 8 barras = histograma de orientações de gradiente de uma sub-região
- Pico alto → direção de borda dominante naquela região
- Vetor normalizado → resistência a variações de iluminação

---

## T1 — Resultado: Invariância a Escala

**Mona Lisa redimensionada → keypoints SIFT detectados em cada versão**

| Escala | Resolução | Keypoints |
|--------|-----------|-----------|
| 100% | 960×1280 | **815** |
| 50% | 480×640 | **240** |
| 25% | 240×320 | **138** |

> A queda de keypoints **não contradiz** a invariância a escala:
> a invariância garante que os pontos *que sobrevivem* são reconhecíveis entre escalas —
> detalhes finos simplesmente deixam de existir nos pixels quando a resolução cai.

---

<!-- _class: lead -->
# Tarefa 2
## Descritores Float vs. Binários: Comparação Prática

---

## T2 — Float vs. Binário

| Característica | Float (SIFT) | Binário (ORB / AKAZE) |
|---------------|-------------|----------------------|
| Representação | 128 floats (512 B) | 256–488 bits (32–61 B) |
| Métrica | NORM_L2 (euclidiana) | NORM_HAMMING (XOR) |
| Velocidade | Lento | **Muito rápido** |
| Precisão | **Alta** | Moderada |

**Ratio Test de Lowe:** aceita match se `d1 < ratio × d2`
- Elimina correspondências ambíguas (dois melhores candidatos igualmente próximos)
- `ratio=0.75` → ~90% falsos eliminados · ~90% verdadeiros preservados

---

## T2 — Resultado: Matching (ratio = 0.75)

**Par: ursinho1.jpeg × ursinho2.jpeg · 960×1280 px**

| Método | Distância | KPs | Matches bons | Match (%) |
|--------|-----------|-----|-------------|-----------|
| SIFT | L2 (float) | 5.535 | **97** | 1.8% |
| ORB | Hamming | 2.000 | 17 | 0.8% |
| AKAZE | Hamming | 1.636 | 36 | **2.2%** |

- SIFT: mais keypoints e mais matches absolutos
- AKAZE: menor volume, maior taxa relativa de matches bons
- ORB: mais rápido, menos discriminativo neste par

---

## T2 — Resultado: Efeito do Ratio Test (SIFT)

| Ratio | Matches aceitos | Qualidade visual |
|-------|----------------|-----------------|
| 0.50 | **5** (0.1%) | Perfeita — linhas paralelas |
| 0.65 | 35 (0.6%) | Alta |
| **0.75** | **97 (1.8%)** | **Equilíbrio (padrão Lowe)** |
| 0.90 | 993 (17.9%) | Degradada — linhas cruzadas |

> Quanto maior o ratio → mais permissivo → mais falsos positivos
>
> Para homografia com poucos inliers: usar **0.65–0.70**
> Para cobertura máxima em cenas difíceis: usar **0.80–0.85**

---

<!-- _class: lead -->
# Tarefa 3
## Montagem de Panorama — Image Stitching

---

## T3 — Pipeline de Stitching

```
Detecção (SIFT/ORB)  →  Ratio Test  →  findHomography (USAC_MAGSAC)
     ↓                                         ↓
warpPerspective  →  Composição no canvas  →  Panorama
```

**Homografia (H 3×3)**
Transformação projetiva que mapeia pontos de um plano para outro.
Requer mínimo de **4 correspondências** · preserva linhas retas.

**USAC_MAGSAC**
Variante robusta do RANSAC: sorteia subconjuntos de 4 pares, estima H, conta inliers (erro reprojeção < threshold), repete — descarta outliers automaticamente.

---

## T3 — Resultado: SIFT vs ORB no Stitching

**Par: mesa_esq.jpeg × mesa_dir.jpeg · 960×1280 px**

| Detector | KP (img1) | Matches | Inliers | Taxa | Qualidade |
|----------|-----------|---------|---------|------|-----------|
| SIFT | 2.377 | 336 | 110 | **32.7%** | ❌ abaixo de 60% |
| ORB | 2.000 | 383 | 235 | **61.4%** | ✅ acima de 60% |

**ORB superou SIFT neste par** — os padrões binários capturaram melhor as texturas da mesa.

> Taxa de inliers > 60% = estimativa da homografia confiável.
> A escolha do melhor detector depende da cena, não há vencedor universal.

---

## T3 — Limitações da Composição Simples

A implementação usa sobreposição direta (sem blending):

1. **Costura visível** — sem mistura gradual na região de sobreposição
2. **Corte de bordas** — translações negativas cortam partes da imagem reprojetada
3. **Sem ajuste de exposição** — diferenças de brilho entre as fotos permanecem

Para costura suave: *multi-band blending* ou *feather blending* na região de sobreposição.

---

<!-- _class: lead -->
# Tarefa 4
## Descritores Neurais com Kornia (DISK)

---

## T4 — O que é DISK?

**DISK — DIscrete Keypoints** (Kornia / PyTorch)

- Modelo neural que aprende **detector e descritor conjuntamente**
- Treinado por **aprendizado por reforço**, otimizando diretamente a qualidade do matching
- Gera descritores **float de 128 dimensões** — mesma dimensão do SIFT, mas aprendidos
- Matching: `NORM_L2` (descritores são L2-normalizados)

**Por que ratio = 0.90 e não 0.75?**
Descritores neurais são mais discriminativos — há menos ambiguidade entre os dois melhores candidatos. Um threshold mais alto ainda filtra bem sem descartar matches corretos.

---

## T4 — Resultado: Extração com DISK

**Par: ursinho1.jpeg × ursinho2.jpeg · CPU (kornia 0.8.3)**

```
DISK img1: 2048 kp  |  desc shape: (2048, 128)
DISK img2: 2048 kp  |  desc shape: (2048, 128)

Matching (ratio=0.90):  58 bons  /  2048 pares candidatos
```

- Modelo pré-treinado `depth` checkpoint (~30 MB, download automático)
- `pad_if_not_divisible=True` — necessário pela arquitetura U-Net (stride 16)

---

## T4 — Tabela Comparativa Final

**Par ursinho · execução local · CPU**

| Método | Tipo | KP | Matches | Ratio |
|--------|------|----|---------|-------|
| SIFT | Clássico / float | 5.535 | **97** | 0.75 |
| ORB | Clássico / binário | 2.000 | 17 | 0.75 |
| AKAZE | Clássico / binário | 1.636 | 36 | 0.75 |
| **DISK** | **Neural / float** | **2.048** | **58** | **0.90** |

> DISK com ratio=0.90 é comparável ao SIFT com ratio=0.75 neste par de boa iluminação.
> A vantagem do DISK aparece em variações drásticas de iluminação ou ponto de vista extremo.

---

## T4 — DISK vs. ORB em Sistemas Embarcados

| Critério | DISK | ORB |
|----------|------|-----|
| Dependência | PyTorch + ~30 MB pesos | Nenhuma |
| Velocidade (CPU) | Lenta | **Muito rápida** (XOR) |
| Hardware mínimo | GPU recomendada | **Microcontrolador** |
| Download de modelo | Necessário | Não necessário |
| Cenário ideal | GPU disponível, cenas difíceis | Embarcado, tempo real |

---

## Visão Geral — Os 4 Métodos

| Método | Tipo | Dimensão | Métrica | KP | Matches | Uso ideal |
|--------|------|----------|---------|-----|---------|-----------|
| SIFT | Float clássico | 128 floats | L2 | 5.535 | 97 | Precisão, offline |
| ORB | Binário | 256 bits | Hamming | 2.000 | 17 | Embarcado, real time |
| AKAZE | Binário | 488 bits | Hamming | 1.636 | 36 | Equilíbrio |
| DISK | Float neural | 128 floats | L2 | 2.048 | 58 | Cenas difíceis, GPU |

---

## Conclusão

- **Espaço de escala** é a base teórica de todos os detectores clássicos: DoG aproxima o LoG e localiza extremos multiescala.
- **Float vs. binário** define a métrica de matching: L2 para gradientes contínuos, Hamming para comparação bit-a-bit.
- **Ratio Test de Lowe** (0.75) é o filtro padrão de ambiguidade — ajustável conforme a aplicação.
- **Homografia + RANSAC** conecta correspondências pontuais à geometria da cena; taxa de inliers > 60% indica qualidade.
- **DISK** demonstra que descritores neurais são viáveis e competitivos, mas com custo de inferência e dependência de framework que os tornam inadequados para hardware limitado.

---

# Obrigado

**Repositório:** `m9tzin/descriptors`
**Entrega:** Atividade 3 · TECA2 20261

*Henryque Oliveira · Matheus Marinho · Rodrigo Oliveira*
