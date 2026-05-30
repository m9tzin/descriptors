# Guia de Estudos — Atividade 3: Pontos de Interesse e Descritores
**TECA2 20261 · Tarefas 1–4**

---

## Como usar este guia

Cada seção lista o que você precisa **entender** e o que precisa **conseguir explicar em voz alta**. As perguntas no final de cada bloco são as mais prováveis de aparecer numa apresentação.

---

## Bloco 1 — Espaço de Escala (Tarefa 1)

### O que estudar

**Espaço de escala**
- Um objeto aparece em tamanhos diferentes dependendo da distância da câmera.
- Para detectar a mesma estrutura independentemente do tamanho, aplica-se um filtro Gaussiano com σ crescente — criando uma sequência de versões cada vez mais borradas da imagem.
- σ (sigma) controla o grau de suavização. Cada valor de σ corresponde a uma "escala" diferente.

**Pirâmide Gaussiana**
- Estrutura de 3+ oitavas, cada uma com N escalas.
- Dentro de uma oitava: mesma resolução, σ crescente (σ multiplica por fator k a cada nível).
- Entre oitavas: a imagem é subamostrada pela metade (resolução cai).
- Resultado visual: uma grade de imagens onde colunas = mais borramento, linhas = menos resolução.

**DoG — Diferença de Gaussianas**
- Calculada subtraindo escalas adjacentes dentro de cada oitava: `DoG = G(σₙ₊₁) − G(σₙ)`.
- Aproxima o Laplaciano da Gaussiana (LoG) — um operador que realça blobs e bordas.
- Regiões claras/escuras no DoG indicam estruturas em determinada escala.
- O SIFT procura extremos locais (máximos e mínimos) no DoG em 3D (vizinhança espacial × escala) para definir keypoints.

### Perguntas de apresentação

- *Por que usar pirâmide e não aplicar só um Gaussiano com σ grande?*
  > A pirâmide preserva a estrutura em múltiplas escalas simultaneamente; um único σ grande só capturaria estruturas daquela escala específica.

- *Por que subamostrar entre oitavas?*
  > Eficiência computacional: estruturas finas já foram detectadas nas oitavas iniciais; reduzir a resolução descarta detalhes irrelevantes e foca em estruturas maiores.

- *O que uma região escura no DoG representa?*
  > Uma região onde a intensidade local é menor que a média das escalas vizinhas — geralmente corresponde ao centro de um blob escuro ou à borda de uma estrutura.

---

## Bloco 2 — Descritores Clássicos: SIFT e AKAZE (Tarefa 1)

### O que estudar

**SIFT — Scale-Invariant Feature Transform**

*Detecção:*
- Encontra extremos locais no DoG (máximo ou mínimo numa vizinhança 3×3×3 entre escalas).
- Descarta pontos de baixo contraste e pontos sobre bordas (usando autovalores da matriz Hessiana).
- Atribui orientação dominante ao keypoint (orientação do gradiente na vizinhança).

*Descritor (128 dimensões):*
- Janela 16×16 pixels ao redor do keypoint, dividida em 16 sub-regiões de 4×4.
- Em cada sub-região: histograma de 8 direções de gradiente → 8 valores.
- 16 sub-regiões × 8 direções = **128 floats**.
- Invariante a rotação (o histograma é calculado em relação à orientação dominante) e a escala (a janela é proporcional à σ do keypoint).

**AKAZE — Accelerated-KAZE**
- Detector e descritor baseado em espaço de escala não-linear (mais robusto a ruído que Gaussiano puro).
- Descritor **binário** (M-LDB): 61 bytes (488 bits) por keypoint.
- Mais seletivo que o SIFT (menos keypoints detectados).

**Visualização rica (`DRAW_RICH_KEYPOINTS`)**
- Círculo: tamanho proporcional à escala do keypoint.
- Raio: indica a orientação dominante.

**Por que regiões planas não geram keypoints?**
- Keypoints exigem variação de intensidade em múltiplas direções (estrutura de canto ou blob).
- Região plana (ex.: céu) → gradiente ≈ 0 → sem extremo no DoG.

### Perguntas de apresentação

- *Por que o SIFT tem 128 dimensões e não outra quantidade?*
  > 4×4 sub-regiões × 8 direções de gradiente = 128. É o arranjo que equilibra discriminação e robustez a pequenas deformações.

- *Qual a diferença entre SIFT e AKAZE em tipo de descritor?*
  > SIFT: float de 128 floats, distância euclidiana. AKAZE: binário de 488 bits, distância de Hamming.

- *O que acontece com os keypoints SIFT quando a imagem encolhe para 25%?*
  > O número cai porque detalhes finos (alta frequência) desaparecem com a subamostragem. A invariância a escala do SIFT significa que um keypoint encontrado em escala 1 pode ser casado com o mesmo ponto em escala 0.5 — não que todos os keypoints serão detectados em qualquer resolução.

---

## Bloco 3 — Matching: Float vs. Binário (Tarefa 2)

### O que estudar

**Tipos de descritor e métricas**

| Tipo | Exemplos | Representação | Métrica |
|------|----------|--------------|---------|
| Float | SIFT | 128 floats (512 bytes) | NORM_L2 (distância euclidiana) |
| Binário | ORB | 256 bits (32 bytes) | NORM_HAMMING (bits diferentes) |
| Binário | AKAZE | 488 bits (61 bytes) | NORM_HAMMING |

**Por que não usar L2 com descritores binários?**
- Distância euclidiana em bits não faz sentido semântico.
- Hamming = contagem de bits diferentes (eficiente com XOR bit-a-bit).

**BFMatcher — Brute Force Matcher**
- Compara cada descritor de img1 com todos os de img2.
- `knnMatch(k=2)`: retorna os 2 melhores candidatos para cada keypoint.

**Ratio Test de Lowe**
- Para cada keypoint, compara o melhor match (distância `d1`) com o segundo melhor (`d2`).
- Aceita o match se `d1 < ratio × d2`.
- Intuição: se o melhor match é muito melhor que o segundo, é provável que seja correto; se os dois são igualmente bons, o match é ambíguo.
- `ratio=0.75` é o valor padrão de Lowe: equilíbrio empírico entre recall e precisão.
- `ratio < 0.65`: muito restritivo, poucos mas bons matches.
- `ratio > 0.85`: permissivo, muitos matches, mais falsos positivos.

**ORB — Oriented FAST and Rotated BRIEF**
- Detector: FAST (rápido, baseado em comparação de pixels num anel).
- Descritor: rBRIEF rotacionado conforme a orientação do keypoint.
- Limita-se a 2000 keypoints por padrão.
- Muito mais rápido que SIFT — ideal para tempo real.

### Perguntas de apresentação

- *Por que o Ratio Test é necessário?*
  > Sem ele, o melhor match entre dois descritores é aceito mesmo quando há outro candidato quase igualmente bom — o que indica ambiguidade (regiões repetitivas, fundo uniforme). O ratio test descarta esses casos ambíguos.

- *Qual método foi mais preciso no par ursinho e por quê?*
  > SIFT teve mais matches absolutos (97 com ratio=0.75) porque seus 128 floats capturam gradientes com mais granularidade. ORB foi mais rápido mas menos discriminativo (17 matches).

- *Por que usar Hamming em vez de L2 para ORB?*
  > ORB produz vetores binários; a distância natural entre bits é a contagem de posições diferentes (Hamming), que se calcula com XOR — eficiente em hardware. L2 em binários não reflete a semelhança real.

---

## Bloco 4 — Image Stitching e Homografia (Tarefa 3)

### O que estudar

**Homografia**
- Transformação projetiva 3×3 que mapeia pontos de um plano de câmera para outro.
- Preserva linhas retas (mas não ângulos nem distâncias).
- Requer no mínimo 4 correspondências de pontos para ser estimada.
- Forma: `p' = H · p` (em coordenadas homogêneas).

**Por que precisamos de homografia no stitching?**
- Duas fotos da mesma cena tiradas de ângulos diferentes relacionam-se por uma homografia (quando a cena é aproximadamente plana ou a câmera só rotaciona).
- A homografia reprojeta uma imagem sobre o sistema de coordenadas da outra, alinhando-as.

**RANSAC — Random Sample Consensus**
- Problema: os matches incluem outliers (correspondências erradas).
- RANSAC sorteia subconjuntos mínimos de matches (4 pontos), estima uma homografia e conta quantos outros matches são consistentes com ela (inliers: erro reprojection < threshold).
- Repete muitas vezes e retém a hipótese com mais inliers.
- `USAC_MAGSAC`: variante moderna com pesos — mais robusto que o RANSAC clássico.

**Taxa de inliers**
- `inliers / total_matches_enviados_ao_RANSAC`.
- > 60% = estimativa confiável (critério do enunciado).
- Tarefa 3: SIFT = 32.7% (ruim), ORB = 61.4% (aceitável).

**Warp perspectivo**
- `cv2.warpPerspective(img, H, (W, H))` — aplica a homografia a todos os pixels de img.
- Canvas maior que as imagens originais para acomodar a imagem reprojetada.

**Composição simples vs. blending**
- Composição simples: copiar img2 reprojetada sobre o canvas; sobrepor img1 sem mistura → costura visível.
- Blending: mistura gradual na região de sobreposição (ex.: feather blending, multi-band blending) → costura suave.

**Por que ORB superou SIFT na Tarefa 3?**
- Neste par específico (mesa, textura homogênea), o ORB produziu matches mais consistentes geometricamente, resultando em taxa de inliers maior.
- Não é uma regra geral: o melhor detector depende da cena.

### Perguntas de apresentação

- *O que é uma homografia e quantos pontos são necessários para estimá-la?*
  > É uma transformação projetiva 3×3. Cada correspondência de pontos dá 2 equações; a homografia tem 8 graus de liberdade (9 parâmetros menos escala) → mínimo de 4 pares de pontos.

- *Por que usar RANSAC em vez de calcular a homografia com todos os matches?*
  > Os matches incluem outliers (correspondências erradas); se todos forem usados, os outliers corrompem a estimativa. RANSAC isola os inliers antes de ajustar a homografia final.

- *O que significa taxa de inliers de 32%?*
  > Dois terços dos matches enviados ao RANSAC são outliers — a estimativa da homografia foi baseada em apenas um terço dos dados. Isso indica que o matching gerou muitas correspondências incorretas, o que fragiliza o alinhamento.

- *Quais são as limitações da composição simples implementada?*
  > (1) Não corrige translações negativas — keypoints fora do canvas são cortados. (2) Sem blending — costura visível na junção. (3) Sem correção de exposição — diferenças de brilho entre as fotos permanecem.

---

## Bloco 5 — Descritores Neurais: DISK (Tarefa 4)

### O que estudar

**DISK — DIscrete Keypoints**
- Modelo neural que aprende detector e descritor conjuntamente, treinado por aprendizado por reforço.
- Objetivo de treinamento: maximizar diretamente a qualidade do matching (não um proxy como gradientes).
- Gera descritores float de **128 dimensões** — mesma dimensão do SIFT, mas aprendidos.
- Matching: `NORM_L2`, igual ao SIFT (descritores são L2-normalizados).

**Por que neurais superam clássicos em cenários difíceis?**
- Mudanças drásticas de iluminação (dia/noite), estações do ano, pontos de vista extremos.
- Métodos clássicos dependem de gradientes estáveis → falham quando a aparência muda muito.
- DISK aprende representações semânticas mais invariantes a partir de dados reais.

**Por que ratio=0.90 para DISK e não 0.75?**
- Descritores neurais são mais discriminativos: há menos ambiguidade entre o melhor e o segundo melhor candidato.
- Um limiar mais alto ainda filtra bem os falsos positivos sem descartar os verdadeiros.

**Tensores PyTorch — formato `[B, C, H, W]`**
- B = batch size (1 imagem = 1)
- C = canais (3 para RGB)
- H, W = altura e largura
- Valores em `[0, 1]` (float32 normalizado)

**DISK na prática (Tarefa 4)**
- `KF.DISK.from_pretrained('depth')`: carrega pesos pré-treinados (~30 MB, download automático).
- `disk(img_t, 2048, pad_if_not_divisible=True)`: extrai até 2048 keypoints.
- `pad_if_not_divisible=True`: necessário porque a rede tem stride 16 (arquitetura U-Net) e requer dimensões múltiplas de 16.

**Resultado observado (par ursinho)**
- DISK: 2048 kp | 58 matches (ratio=0.90).
- Comparável ao SIFT (97 matches, ratio=0.75) neste par de boa iluminação — a vantagem do DISK aparece em cenários mais difíceis.

**Desvantagens práticas do DISK**
- Requer PyTorch + pesos pré-treinados (~30 MB).
- Inferência em CPU muito mais lenta que ORB.
- Inviável em microcontroladores ou sistemas embarcados sem GPU.

### Perguntas de apresentação

- *O que diferencia o DISK do SIFT conceitualmente?*
  > O SIFT calcula gradientes locais com regras matemáticas fixas; o DISK aprende — a partir de dados com supervisão por reforço — o que constitui um bom keypoint e um bom descritor. O DISK otimiza diretamente a qualidade do matching, enquanto o SIFT otimiza uma proxy (invariância a gradientes).

- *Por que o DISK usa NORM_L2 e não NORM_HAMMING?*
  > Os descritores DISK são vetores float de 128 dimensões (não binários). A distância euclidiana é a métrica natural para vetores contínuos.

- *Em que situação você escolheria ORB sobre DISK?*
  > Sistemas embarcados (robótica, drones), aplicações em tempo real sem GPU, ambientes com restrição de memória. O ORB não tem dependência de deep learning, é calculado em microsegundos e cabe em microcontroladores.

---

## Tabela-resumo: os 4 métodos

| Método | Tipo | Dimensão | Métrica | KP (ursinho) | Matches | Velocidade | Cenário ideal |
|--------|------|----------|---------|-------------|---------|-----------|--------------|
| SIFT   | Float clássico | 128 floats | NORM_L2 | 5535 | 97 (r=0.75) | Lento | Precisão, offline |
| ORB    | Binário clássico | 256 bits | NORM_HAMMING | 2000 | 17 (r=0.75) | Muito rápido | Tempo real, embarcado |
| AKAZE  | Binário clássico | 488 bits | NORM_HAMMING | 1636 | 36 (r=0.75) | Rápido | Equilíbrio |
| DISK   | Float neural | 128 floats | NORM_L2 | 2048 | 58 (r=0.90) | Lento (CPU) | Cenários difíceis, GPU |

---

## Perguntas transversais (valem para qualquer tarefa)

**1. O que é um keypoint?**
> Um ponto na imagem com localização (x, y), escala e orientação, detectado por possuir estrutura local discriminativa (canto, blob, extremo de gradiente).

**2. O que é um descritor?**
> Um vetor numérico que resume a aparência local ao redor de um keypoint de forma compacta e (idealmente) invariante a transformações.

**3. O que é o Ratio Test de Lowe?**
> Filtro que aceita um match só quando o melhor candidato é significativamente mais próximo que o segundo melhor: `d1 < ratio × d2`. Elimina correspondências ambíguas.

**4. Float vs. binário: qual escolher?**
> Float (SIFT, DISK): maior precisão, mais memória, distância euclidiana. Binário (ORB, AKAZE): menor memória, distância Hamming, muito mais rápido. Escolha depende de latência, hardware e variabilidade da cena.

**5. Quando um método clássico falha e um neural ajuda?**
> Mudanças drásticas de iluminação, estação, ponto de vista extremo — situações onde gradientes mudam radicalmente mas a semântica persiste. Clássicos dependem de gradientes estáveis; neurais aprendem representações mais invariantes.
