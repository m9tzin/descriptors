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
  code { background: #f0f4f8; padding: 2px 6px; border-radius: 3px; }
  blockquote { border-left: 4px solid #2e6da4; background: #f0f4f8; padding: 8px 12px; }
  footer { font-size: 0.75em; color: #555; }
---

# Atividade 3 — Pontos de Interesse e Descritores

**TECA2 20261**

Henryque Oliveira · Matheus Marinho · Rodrigo Oliveira

---
<!-- _footer: "🎤 Henryque" -->

## Agenda

| Tarefa | Tema | Apresenta |
|--------|------|-----------|
| **T1** | Espaço de Escala, Pirâmide, DoG | Henryque → Matheus → Rodrigo |
| **T2** | Descritores Float vs. Binários | Henryque → Matheus → Rodrigo |
| **T3** | Montagem de Panorama (Stitching) | Henryque → Matheus → Rodrigo |
| **T4** | Descritores Neurais (DISK) | Henryque → Matheus → Rodrigo |

---

<!-- _class: lead -->
# Tarefa 1
## Espaço de Escala, Visualização e Análise de Descritores

---
<!-- _footer: "🎤 Henryque" -->

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
<!-- _footer: "🎤 Henryque" -->

## T1 — Resultado: Pirâmide e DoG

**3 oitavas · 5 escalas por oitava · gerado manualmente**

| Oitava | Resolução | σ cresce → | Estruturas visíveis |
|--------|-----------|-----------|-------------------|
| 1 | 100% | baixo → alto | textura fina, janelas |
| 2 | 50% | baixo → alto | bordas, cantos |
| 3 | 25% | baixo → alto | estruturas grandes |

> Estruturas finas respondem nas oitavas iniciais · estruturas grandes nas finais

---
<!-- _footer: "🎤 Matheus" -->

## T1 — Resultado: Keypoints SIFT vs AKAZE

**3 imagens · `DRAW_RICH_KEYPOINTS` (tamanho = escala · raio = orientação)**

| Imagem | SIFT (kp) | AKAZE (kp) |
|--------|-----------|------------|
| Prédio (building2) | **9.711** | 6.368 |
| Mona Lisa | **815** | 277 |
| Fig0438 (Gonzalez) | **686** | 264 |

- SIFT detecta mais keypoints — menos seletivo
- AKAZE usa espaço de escala não-linear → mais restritivo
- Regiões planas (céu) → gradiente ≈ 0 → **nenhum keypoint**

---
<!-- _footer: "🎤 Rodrigo" -->

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
<!-- _footer: "🎤 Rodrigo" -->

## T1 — Resultado: Invariância a Escala

**Mona Lisa redimensionada → keypoints SIFT por versão**

| Escala | Resolução | Keypoints |
|--------|-----------|-----------|
| 100% | 960×1280 | **815** |
| 50% | 480×640 | **240** |
| 25% | 240×320 | **138** |

> A queda **não contradiz** a invariância a escala:
> garante que os pontos *que sobrevivem* são reconhecíveis entre escalas —
> detalhes finos deixam de existir nos pixels quando a resolução cai.

---

<!-- _class: lead -->
# Tarefa 2
## Descritores Float vs. Binários: Comparação Prática

---
<!-- _footer: "🎤 Henryque" -->

## T2 — Float vs. Binário

| Característica | Float (SIFT) | Binário (ORB / AKAZE) |
|---------------|-------------|----------------------|
| Representação | 128 floats (512 B) | 256–488 bits (32–61 B) |
| Métrica | NORM_L2 (euclidiana) | NORM_HAMMING (XOR) |
| Velocidade | Lento | **Muito rápido** |
| Precisão | **Alta** | Moderada |

**Ratio Test de Lowe:** aceita match se `d1 < ratio × d2`
- Elimina correspondências ambíguas (dois candidatos igualmente próximos)
- `ratio=0.75` → ~90% falsos eliminados · ~90% verdadeiros preservados

---
<!-- _footer: "🎤 Matheus" -->

## T2 — Resultado: Matching (ratio = 0.75)

**Par: ursinho1 × ursinho2 · 960×1280 px**

| Método | Distância | KPs | Matches bons | Match (%) |
|--------|-----------|-----|-------------|-----------|
| SIFT | L2 (float) | 5.535 | **97** | 1.8% |
| ORB | Hamming | 2.000 | 17 | 0.8% |
| AKAZE | Hamming | 1.636 | 36 | **2.2%** |

- SIFT: mais keypoints e mais matches absolutos
- AKAZE: menor volume, maior taxa relativa de matches bons
- ORB: mais rápido, menos discriminativo neste par

---
<!-- _footer: "🎤 Rodrigo" -->

## T2 — Resultado: Efeito do Ratio Test (SIFT)

| Ratio | Matches aceitos | Qualidade visual |
|-------|----------------|-----------------|
| 0.50 | **5** (0.1%) | Perfeita — linhas paralelas |
| 0.65 | 35 (0.6%) | Alta |
| **0.75** | **97 (1.8%)** | **Equilíbrio (padrão Lowe)** |
| 0.90 | 993 (17.9%) | Degradada — linhas cruzadas |

> Quanto maior o ratio → mais permissivo → mais falsos positivos
>
> Homografia com poucos inliers: **0.65–0.70** · Cobertura máxima: **0.80–0.85**

---

<!-- _class: lead -->
# Tarefa 3
## Montagem de Panorama — Image Stitching

---
<!-- _footer: "🎤 Henryque" -->

## T3 — Pipeline de Stitching

```
Detecção (SIFT/ORB)  →  Ratio Test  →  findHomography (USAC_MAGSAC)
        ↓                                        ↓
 warpPerspective  →  Composição no canvas  →  Panorama
```

**Homografia (H 3×3)**
Transformação projetiva que mapeia pontos de um plano para outro.
Requer mínimo de **4 correspondências** · preserva linhas retas.

**USAC_MAGSAC**
Variante robusta do RANSAC: sorteia subconjuntos de 4 pares, estima H, conta inliers (erro reprojeção < threshold), repete — descarta outliers automaticamente.

---
<!-- _footer: "🎤 Matheus" -->

## T3 — Resultado: SIFT vs ORB no Stitching

**Par: mesa_esq × mesa_dir · 960×1280 px**

| Detector | KP (img1) | Matches | Inliers | Taxa | Qualidade |
|----------|-----------|---------|---------|------|-----------|
| SIFT | 2.377 | 336 | 110 | **32.7%** | ❌ abaixo de 60% |
| ORB | 2.000 | 383 | 235 | **61.4%** | ✅ acima de 60% |

**ORB superou SIFT neste par** — padrões binários capturaram melhor as texturas da mesa.

> Taxa de inliers > 60% = homografia confiável.
> Não há vencedor universal: o melhor detector depende da cena.

---
<!-- _footer: "🎤 Rodrigo" -->

## T3 — Limitações da Composição Simples

A implementação usa sobreposição direta (sem blending):

1. **Costura visível** — sem mistura gradual na região de sobreposição
2. **Corte de bordas** — translações negativas cortam partes reprojetadas
3. **Sem ajuste de exposição** — diferenças de brilho entre as fotos permanecem

Para costura suave em produção: *multi-band blending* ou *feather blending*.

---

<!-- _class: lead -->
# Tarefa 4
## Descritores Neurais com Kornia (DISK)

---
<!-- _footer: "🎤 Henryque" -->

## T4 — O que é DISK?

**DISK — DIscrete Keypoints** (Kornia / PyTorch)

- Modelo neural que aprende **detector e descritor conjuntamente**
- Treinado por **aprendizado por reforço**, otimizando diretamente a qualidade do matching
- Gera descritores **float de 128 dimensões** — mesma dimensão do SIFT, mas aprendidos
- Matching: `NORM_L2` (descritores são L2-normalizados)

**Por que ratio = 0.90 e não 0.75?**
Descritores neurais são mais discriminativos — menos ambiguidade entre os dois melhores candidatos. Threshold mais alto ainda filtra bem sem descartar matches corretos.

---
<!-- _footer: "🎤 Matheus" -->

## T4 — Resultado: Extração e Tabela Final

**Par ursinho · CPU · kornia 0.8.3**

```
DISK: 2048 kp por imagem  |  desc shape: (2048, 128)
Matching (ratio=0.90): 58 bons / 2048 pares candidatos
```

| Método | Tipo | KP | Matches | Ratio |
|--------|------|----|---------|-------|
| SIFT | Clássico / float | 5.535 | **97** | 0.75 |
| ORB | Clássico / binário | 2.000 | 17 | 0.75 |
| AKAZE | Clássico / binário | 1.636 | 36 | 0.75 |
| **DISK** | **Neural / float** | **2.048** | **58** | **0.90** |

> DISK ≈ SIFT neste par de boa iluminação. Vantagem real aparece em cenas difíceis.

---
<!-- _footer: "🎤 Rodrigo" -->

## T4 — DISK vs. ORB em Sistemas Embarcados

| Critério | DISK | ORB |
|----------|------|-----|
| Dependência | PyTorch + ~30 MB pesos | Nenhuma |
| Velocidade (CPU) | Lenta | **Muito rápida** (XOR) |
| Hardware mínimo | GPU recomendada | **Microcontrolador** |
| Download de modelo | Necessário | Não necessário |
| Cenário ideal | GPU disponível, cenas difíceis | Embarcado, tempo real |

---
<!-- _footer: "🎤 Rodrigo" -->

## Visão Geral — Os 4 Métodos

| Método | Tipo | Dimensão | Métrica | KP | Matches | Uso ideal |
|--------|------|----------|---------|-----|---------|-----------|
| SIFT | Float clássico | 128 floats | L2 | 5.535 | 97 | Precisão, offline |
| ORB | Binário | 256 bits | Hamming | 2.000 | 17 | Embarcado, real time |
| AKAZE | Binário | 488 bits | Hamming | 1.636 | 36 | Equilíbrio |
| DISK | Float neural | 128 floats | L2 | 2.048 | 58 | Cenas difíceis, GPU |

---
<!-- _footer: "🎤 Rodrigo" -->

## Conclusão

- **Espaço de escala** é a base teórica de todos os detectores clássicos: DoG aproxima o LoG e localiza extremos multiescala.
- **Float vs. binário** define a métrica de matching: L2 para gradientes contínuos, Hamming para XOR bit-a-bit.
- **Ratio Test de Lowe** (0.75) é o filtro padrão de ambiguidade — ajustável conforme a aplicação.
- **Homografia + RANSAC** conecta correspondências pontuais à geometria da cena; inliers > 60% indica qualidade.
- **DISK** demonstra que descritores neurais são competitivos, mas custo de inferência e dependência de PyTorch os tornam inadequados para hardware limitado.

---

# Obrigado

**Repositório:** `m9tzin/descriptors`
**Entrega:** Atividade 3 · TECA2 20261

*Henryque Oliveira · Matheus Marinho · Rodrigo Oliveira*
