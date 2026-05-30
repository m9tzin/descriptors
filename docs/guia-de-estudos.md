# Guia de Estudos — Atividade 3: Pontos de Interesse e Descritores
**TECA2 20261 · Tarefas 1–4**

> Organizado por tarefa. Cada bloco traz a teoria do notebook, as observações esperadas dos experimentos, as análises e as perguntas mais prováveis de aparecer numa apresentação — com resposta.

---

## Tarefa 1 — Espaço de Escala, Visualização e Análise de Descritores

### Base teórica

**Espaço de escala**
Um objeto aparece em tamanhos diferentes conforme a distância da câmera. Para detectar a mesma estrutura independentemente do tamanho, gera-se uma família de versões progressivamente borradas com filtro Gaussiano. Borrar com σ maior equivale a "afastar a câmera": detalhes finos somem, estruturas grandes permanecem.

**Pirâmide Gaussiana**
Organiza o espaço de escala em *oitavas* e *escalas*:
- **Escalas**: dentro de cada oitava, a resolução é fixa mas σ cresce — `σₛ = σ₀ · kˢ`.
- **Oitavas**: a imagem é subamostrada pela metade entre oitavas — economiza computação e foca em estruturas maiores.
- Resultado visual: grade onde colunas = mais borramento, linhas = menos resolução.

**DoG ≈ LoG**
O detector ideal de blobs seria o Laplaciano do Gaussiano (LoG), mas é computacionalmente caro. Lowe mostrou que a diferença entre duas imagens borradas com σ vizinhos aproxima o LoG:

`DoG = G(kσ) − G(σ) ≈ (k−1)σ²·∇²G`

Os keypoints SIFT são **extremos locais** nessa pilha DoG: cada pixel é comparado com 8 vizinhos no mesmo nível e 9+9 nos níveis adjacentes (26 vizinhos no total).

**Descritor SIFT de 128 dimensões**
Janela 16×16 ao redor do keypoint (na escala e orientação detectadas) → 4×4 sub-regiões → histograma de 8 direções em cada → **4×4×8 = 128 valores**.
- Cada barra do histograma = soma das magnitudes de gradiente naquela direção.
- O vetor é normalizado para resistir a mudanças de iluminação.
- Invariante a rotação: o histograma é calculado em relação à orientação dominante do keypoint.

> Referências: Corke (2023) Cap. 12, Seção 12.3.2 · Gonzalez & Woods (2018) Cap. 9 · Lowe (2004).

---

### Item 1 — Pirâmide Gaussiana e DoG

**O que foi feito:** Pirâmide manual com 3 oitavas e 5 escalas por oitava.

**Interpretação dos resultados:**
- Na pirâmide: cada linha (oitava) parte de uma imagem com metade da resolução da anterior; dentro de cada linha o borramento aumenta da esquerda para a direita.
- Nas imagens DoG: regiões claras/escuras realçam bordas e blobs em cada escala.
- Estruturas finas (textura, janelas) respondem nas oitavas iniciais; estruturas grandes respondem nas oitavas finais.

---

### Item 2 — Visualização rica de keypoints (SIFT vs AKAZE)

**O que foi feito:** Detecção em 3 imagens com flag `DRAW_RICH_KEYPOINTS` — círculo proporcional à escala, raio indica orientação.

**Observações:**
- O **céu** (região plana, sem gradiente) não recebe keypoints — pontos de interesse exigem estrutura local.
- **Fachada** e **árvores** ficam densamente cobertas (muita textura e cantos).
- **SIFT detecta mais keypoints que o AKAZE** na mesma imagem (~9600 vs ~6400 no prédio). O AKAZE é mais seletivo.
- Descritor SIFT: **128 dimensões (float)**. AKAZE: **61 bytes (binário)**.

---

### Item 3 — Análise do descritor SIFT de 128 componentes

**O que foi feito:** Um keypoint de escala mediana foi selecionado e seus 128 valores plotados em barras, com linhas vermelhas separando as 16 sub-regiões.

**Interpretação:**
- Cada grupo de 8 barras = histograma de orientações de gradiente de uma sub-região 4×4.
- Picos altos indicam direção de borda dominante naquela sub-região.
- 128 não é número arbitrário: **4×4 sub-regiões × 8 direções = 128**. Esse arranjo equilibra discriminação e robustez a pequenas deformações.

---

### Item 4 — Efeito da escala (invariância do SIFT)

**O que foi feito:** Mona Lisa redimensionada para 100%, 50% e 25%; keypoints SIFT detectados em cada versão.

**Discussão — invariância a escala:**
O número de keypoints cai conforme a imagem encolhe (~1066 → ~330 → ~142). Isso **não contradiz** a invariância a escala do SIFT — ilustra seus limites:
- O SIFT é invariante a escala no sentido de que *um mesmo ponto detectado numa escala consegue ser casado com o mesmo ponto em outra escala*, porque o descritor é calculado numa janela proporcional à σ detectada.
- Quando a imagem é fisicamente reduzida, **detalhes finos deixam de existir nos pixels** (subamostragem os elimina). Esses keypoints de alta frequência somem; sobram apenas os de estruturas maiores.
- A invariância garante que os pontos *que sobrevivem* serão reconhecíveis entre escalas — não que todos sobreviverão à perda de resolução.

---

### Conclusão da Tarefa 1

- Construímos a pirâmide Gaussiana e a DoG, visualizando a estrutura multiescala que fundamenta a detecção SIFT.
- Comparamos SIFT (mais keypoints, descritor 128-float) e AKAZE (mais seletivo, descritor 61-binário) em três texturas; confirmamos que regiões planas não geram keypoints.
- Interpretamos o descritor de 128 dim como 16 histogramas de 8 orientações (4×4×8).
- Verificamos empiricamente que a invariância a escala do SIFT preserva keypoints de estruturas grandes entre escalas, mas detalhes finos se perdem com redução de resolução.

---

### Perguntas de apresentação — Tarefa 1

**Por que usar pirâmide e não aplicar só um Gaussiano com σ grande?**
> A pirâmide preserva a estrutura em múltiplas escalas simultaneamente; um único σ grande só capturaria estruturas daquela escala específica.

**Por que subamostrar entre oitavas?**
> Eficiência computacional: estruturas finas já foram detectadas nas oitavas iniciais; reduzir a resolução descarta detalhes redundantes e concentra processamento em estruturas maiores.

**O que uma região escura no DoG representa?**
> Uma região onde a intensidade local é menor que a média das escalas vizinhas — geralmente o centro de um blob escuro ou borda de uma estrutura.

**Por que o SIFT tem 128 dimensões e não outra quantidade?**
> 4×4 sub-regiões × 8 direções de gradiente = 128. Esse arranjo foi escolhido por equilibrar poder de discriminação com robustez a pequenas deformações.

**O que acontece com os keypoints SIFT quando a imagem encolhe para 25%?**
> O número cai porque detalhes finos desaparecem com a subamostragem. A invariância a escala garante que um keypoint detectado em escala 1 pode ser casado com o mesmo ponto em escala 0.5 — não que todos os keypoints serão detectados em qualquer resolução.

**Por que regiões planas (céu) não geram keypoints?**
> Keypoints exigem variação de intensidade em múltiplas direções (canto, blob). Região plana → gradiente ≈ 0 → sem extremo no DoG → nenhum keypoint.

---

## Tarefa 2 — Descritores Float vs. Binários: Comparação Prática

### Base teórica

**Descritores float (SIFT)**
Medem magnitudes contínuas de gradiente. Similaridade = distância euclidiana (`NORM_L2`): quanto menor, mais parecidos. 128 floats = 512 bytes por keypoint.

**Descritores binários (ORB, AKAZE)**
Armazenam bits comparando pares de pixels. Similaridade = distância de Hamming (`NORM_HAMMING`): conta quantos bits diferem — calculada com XOR, ordens de magnitude mais rápida que L2. ORB: 256 bits (32 bytes). AKAZE: 488 bits (61 bytes).

**BFMatcher — Brute Force Matcher**
Compara cada descritor de img1 com todos os de img2. `knnMatch(k=2)` retorna os 2 melhores candidatos para cada keypoint.

**Ratio Test de Lowe**
Para cada keypoint, compara o melhor match (`d1`) com o segundo melhor (`d2`). Aceita se `d1 < ratio × d2`.
- Intuição: se o melhor é muito melhor que o segundo, o match é confiante; se os dois são próximos, o match é ambíguo e descartado.
- `ratio=0.75` (padrão Lowe): elimina ~90% dos falsos matches preservando ~90% dos verdadeiros — valor empírico da pesquisa original.

> Referências: Corke (2023) Seção 13.3 · Lowe (2004) Seção 7.1 · Slides TECA2 20261, slides 12–16.

---

### Item 2 — Pipeline de matching

**O que foi feito:** `pipeline_match()` detecta, descreve, aplica BFMatcher com norma correta e filtra com Ratio Test.

**Observações:**
- SIFT: mais keypoints, descritores contínuos, matches de alta precisão graças à distância euclidiana.
- ORB e AKAZE: distância de Hamming calculada com XOR bit-a-bit — muito mais rápida.
- Matches ruins formam linhas cruzadas e caóticas na visualização; o Ratio Test elimina a maioria.

---

### Item 3 — Efeito do Ratio Test (SIFT)

**O que foi feito:** Ratio variado de 0.50 a 0.90 com SIFT, observando quantidade e qualidade visual dos matches.

**Trade-off quantidade × qualidade:**
- **ratio=0.50**: poucos matches, quase todos corretos — linhas paralelas e ordenadas.
- **ratio=0.65**: número moderado; qualidade ainda alta.
- **ratio=0.75** (padrão Lowe): equilíbrio empírico.
- **ratio=0.90**: muitos matches, linhas cruzadas — falsos positivos visíveis.

Regra prática: para estimativa de homografia com poucos inliers → 0.65–0.70. Para cobertura máxima em cenas difíceis → 0.80–0.85.

---

### Item 4 — Tabela comparativa

| Método | Distância | KPs (img1) | Match (%) |
|--------|-----------|-----------|-----------|
| SIFT | L2 (float) | 5535 | 1.8% |
| ORB | Hamming (bin.) | 2000 | 0.8% |
| AKAZE | Hamming (bin.) | 1636 | 2.2% |

*Match (%) = matches bons ÷ total de pares candidatos (ratio=0.75)*

**Análise:**
- O ORB limita-se a `nfeatures=2000` por parâmetro; o SIFT detecta conforme a estrutura da imagem; o AKAZE é o mais seletivo.
- Percentual alto = descritores discriminativos (poucas ambiguidades). Percentual baixo pode indicar descritores menos discriminativos *ou* pouca sobreposição entre imagens.
- **Quando usar cada um:** SIFT → precisão crítica (Structure from Motion, AR). ORB/AKAZE → embarcado ou tempo real (robô, drone).

---

### Conclusão da Tarefa 2

- A métrica de distância deve ser compatível com o tipo de descritor: L2 para float, Hamming para binário.
- O Ratio Test de Lowe é essencial: sem ele matches ambíguos dominam; com `ratio=0.75` as correspondências são visualmente coerentes.
- Trade-off fundamental: precisão (SIFT, float, mais lento) vs. velocidade (ORB/AKAZE, binário, mais rápido).

---

### Perguntas de apresentação — Tarefa 2

**Por que não usar L2 com descritores binários?**
> Distância euclidiana em bits não tem sentido semântico. Hamming — contagem de bits diferentes via XOR — é a distância natural para vetores binários e é eficiente em hardware.

**Por que o Ratio Test é necessário?**
> Sem ele, o melhor match entre dois descritores é aceito mesmo quando há outro candidato quase igualmente bom — o que indica ambiguidade (regiões repetitivas, fundo uniforme). O ratio test descarta esses casos ambíguos.

**O que acontece visualmente quando ratio=0.90?**
> Muitos matches são aceitos, mas linhas cruzadas na visualização revelam correspondências incorretas — falsos positivos que o threshold mais permissivo deixou passar.

**Qual método foi mais preciso no par ursinho e por quê?**
> SIFT teve mais matches absolutos (97) porque seus 128 floats capturam gradientes com mais granularidade. ORB foi mais rápido mas menos discriminativo (17 matches). AKAZE teve o maior percentual relativo (2.2%) sendo o mais seletivo.

**Qual escolher para um drone com processador ARM?**
> ORB: usa XOR bit-a-bit, roda em tempo real mesmo em hardware limitado, sem dependência de ponto flutuante, footprint de memória mínimo.

---

## Tarefa 3 — Aplicação: Montagem de Panorama (Image Stitching)

### Base teórica

**Homografia**
Transformação projetiva 3×3 que mapeia pontos de um plano de imagem para outro, preservando linhas retas. `cv2.findHomography` estima a matriz H tal que `p2 ≈ H · p1` (coordenadas homogêneas). Para o stitching, H reprojeta (`warp`) img1 no sistema de coordenadas de img2. Requer no mínimo **4 correspondências** (cada par dá 2 equações; H tem 8 graus de liberdade).

**RANSAC**
Estima H de forma robusta na presença de outliers (matches incorretos que passaram pelo Ratio Test):
1. Sorteia subconjunto mínimo de 4 pares.
2. Estima H com esses 4 pontos.
3. Conta inliers: pares cuja distância de reprojeção < threshold.
4. Repete muitas vezes, guarda o modelo com mais inliers.
5. Reestima H final usando todos os inliers.

**USAC_MAGSAC**
Variante moderna (`cv2.USAC_MAGSAC` em OpenCV ≥ 4.5): mais robusto que RANSAC clássico especialmente quando a proporção de outliers é alta. O restante do código permanece idêntico.

**Taxa de inliers**
`inliers / total_matches_enviados_ao_RANSAC`. Acima de 60% = estimativa confiável (critério do enunciado).

> Referências: Corke (2023) Seções 14.1–14.2.4 · Brown & Lowe (2007) AutoStitch · Slides TECA2 20261, slides 15–18.

---

### Item 2 — Pipeline SIFT completo

**Pipeline:** detecção + descrição → Ratio Test → `findHomography(USAC_MAGSAC)` → `warpPerspective` → composição.

**Resultado:**

| Detector | KP (img1) | Matches | Inliers (%) | OK? |
|----------|-----------|---------|-------------|-----|
| SIFT | 2377 | 336 | 32.7% | Não |
| ORB | 2000 | 383 | 61.4% | Sim |

**Composição simples:** sobrepõe img2 reprojetada sobre o canvas sem blending — a costura pode ter uma linha visível. O zoom na junção permite avaliar o alinhamento.

---

### Item 3 — Pipeline ORB

**Discussão:**
- **SIFT:** 336 matches, mas taxa de inliers 32.7% — abaixo do limiar de 60%. Alguns matches incorretos influenciaram a homografia; alinhamento pode apresentar leve deslocamento.
- **ORB:** 383 matches e 61.4% de inliers — acima do limiar. Os padrões binários capturaram bem as texturas da mesa neste par específico. Alinhamento mais preciso na junção.
- **Quando preferiria ORB em sistema real?** Em embarcados (robô, drone): ordens de magnitude mais rápido que SIFT por usar XOR em vez de aritmética float; neste caso obteve qualidade superior.

**Limitações da composição simples:**
1. Não corrige translação negativa — se img1 mapeada tiver coordenadas negativas, partes são cortadas.
2. Sem blending na sobreposição — costura visível.
3. Sem ajuste automático do canvas pela bounding box das imagens warpadas.

---

### Conclusão da Tarefa 3

- Pipeline completo: detecção → matching → homografia USAC_MAGSAC → warp → composição simples.
- ORB superou SIFT em taxa de inliers (61.4% vs 32.7%) neste par, demonstrando que o melhor detector depende da cena.
- A taxa de inliers é o indicador mais confiável de qualidade do stitching: acima de 60% o resultado costuma ser visualmente bom.

---

### Perguntas de apresentação — Tarefa 3

**O que é uma homografia e quantos pontos são necessários?**
> Transformação projetiva 3×3. Cada correspondência dá 2 equações; H tem 8 graus de liberdade → mínimo de 4 pares de pontos.

**Por que usar RANSAC em vez de calcular H com todos os matches?**
> Os matches incluem outliers (correspondências erradas). Se todos forem usados, os outliers corrompem a estimativa. RANSAC isola os inliers antes de ajustar H.

**O que significa taxa de inliers de 32.7%?**
> Dois terços dos matches enviados ao RANSAC são outliers — a homografia foi baseada em apenas um terço dos dados. Isso fragiliza o alinhamento.

**Por que o ORB teve mais inliers que o SIFT neste par?**
> Os padrões binários do ORB (rBRIEF) capturaram bem as texturas da mesa neste par específico. Não é regra geral: o melhor detector depende das características visuais da cena.

**Quais são as limitações da composição simples implementada?**
> (1) Sem blending — costura visível. (2) Sem correção de translação negativa — partes podem ser cortadas. (3) Sem ajuste automático do canvas.

**O que seria necessário para uma costura perfeita?**
> Multi-band blending ou feather blending na região de sobreposição, correção de exposição entre as imagens e cálculo automático do canvas pela bounding box.

---

## Tarefa 4 — Descritores Neurais com Kornia (DISK)

### Base teórica

**DISK — DIscrete Keypoints**
Modelo neural que aprende detector e descritor conjuntamente por aprendizado por reforço, otimizando diretamente a qualidade do matching. Ao contrário de SIFT/ORB (regras matemáticas fixas sobre gradientes), o DISK aprende — a partir de dados — o que constitui um bom keypoint e um bom descritor. Gera descritores float de **128 dimensões** — mesma dimensão do SIFT, mas aprendidos.

**Por que neurais superam clássicos em cenários difíceis?**
Mudanças drásticas de iluminação (dia/noite), estações do ano, pontos de vista muito distantes — situações em que gradientes locais mudam radicalmente, mas a semântica da cena permanece. Clássicos falham porque dependem de gradientes estáveis; neurais aprendem representações mais invariantes.

**Por que ratio=0.90 para DISK e não 0.75?**
Descritores neurais são mais discriminativos — há menos ambiguidade entre os dois melhores candidatos. Um ratio maior ainda filtra bem os falsos positivos sem descartar os verdadeiros.

**Por que DISK e não R2D2 ou SuperPoint?**
O DISK tem boa integração com Kornia e tutorial oficial disponível. R2D2 e SuperPoint resolvem o mesmo problema com filosofias diferentes; em produção a escolha depende do dataset, da plataforma e da licença.

**Tensores PyTorch — formato `[B, C, H, W]`**
- B = batch, C = canais (3 para RGB), H = altura, W = largura.
- Valores em `[0, 1]` (float32 normalizado).
- `pad_if_not_divisible=True`: necessário porque o DISK usa arquitetura U-Net com stride 16, que exige dimensões múltiplas de 16.

> Referências: Torralba, Freeman, Isola (2024) Cap. 11 · Tutorial Kornia DISK · Slides TECA2 20261, slide 20.

---

### Itens 1 e 2 — Carregar tensores e rodar DISK

**O que foi feito:** Imagens carregadas como arrays OpenCV e como tensores `[1,3,H,W]`. DISK pré-treinado (`depth` checkpoint, ~30 MB) extraiu até 2048 keypoints por imagem.

**Resultado:** 2048 kp por imagem | descritores shape `(2048, 128)`.

---

### Item 3 — Matching e visualização

**O que foi feito:** Keypoints DISK convertidos para `cv2.KeyPoint` para usar `drawMatches`. BFMatcher NORM_L2, ratio=0.90.

**Resultado:** 58 matches bons de 2048 pares candidatos.

---

### Item 4 — Tabela comparativa e reflexão crítica

| Método | Tipo | KP | Matches |
|--------|------|----|---------|
| SIFT | Clássico / float | 5535 | 97 (ratio 0.75) |
| ORB | Clássico / binário | 2000 | 17 (ratio 0.75) |
| AKAZE | Clássico / binário | 1636 | 36 (ratio 0.75) |
| DISK | Neural / float | 2048 | 58 (ratio 0.90) |

> Nota: SIFT/ORB/AKAZE usaram ratio=0.75; DISK usou ratio=0.90. Limiares diferentes permitem mais matches — comparação de contagens deve levar isso em conta.

**Reflexão 1 — Em que cenário o DISK foi melhor que o SIFT? Em que foram parecidos?**
O DISK obteve 58 matches com ratio=0.90, contra 97 do SIFT com ratio=0.75. Considerando o threshold mais permissivo do DISK, o desempenho é comparável — ambos funcionam bem neste par com iluminação estável e variação leve de ângulo. A real vantagem do DISK apareceria em pares com variação drástica de iluminação ou ponto de vista extremo, onde gradientes mudam radicalmente e o SIFT falha.

**Reflexão 2 — Desvantagens do DISK vs. ORB em sistema embarcado:**
O DISK requer PyTorch e pesos pré-treinados (~30 MB), tornando o deployment em microcontroladores inviável. A inferência em CPU é significativamente mais lenta que o ORB, que usa XOR bit-a-bit e roda em tempo real mesmo em hardware limitado. O ORB não depende de conexão para baixar modelos e seu footprint de memória é mínimo.

**Reflexão 3 — Escolha para vigilância indoor em tempo real:**
SIFT como baseline clássico, ou DISK se houver GPU disponível. Com GPU, o DISK ganha em robustez a variações de iluminação típicas de câmeras de segurança. Sem GPU, o SIFT é preferível: mais rápido em CPU e sem dependência de PyTorch.

---

### Conclusão da Tarefa 4

- O DISK usa a mesma métrica do SIFT (NORM_L2) mas com representações aprendidas — mais robusto em cenários difíceis.
- Tabela final ilustra o espectro completo: binários rápidos (ORB/AKAZE) → float clássico (SIFT) → neural moderno (DISK), cada um com trade-offs de velocidade, tamanho e qualidade.
- A escolha do método ideal depende de: latência exigida, hardware disponível, variabilidade visual da cena.

---

### Perguntas de apresentação — Tarefa 4

**O que diferencia o DISK do SIFT conceitualmente?**
> O SIFT calcula gradientes com regras matemáticas fixas; o DISK aprende por reforço o que é um bom keypoint e um bom descritor, otimizando diretamente a qualidade do matching — não uma proxy como invariância a gradientes.

**Por que o DISK usa NORM_L2 e não NORM_HAMMING?**
> Os descritores DISK são vetores float de 128 dimensões (não binários). A distância euclidiana é a métrica natural para vetores contínuos; os descritores são L2-normalizados, o que equivale a similaridade por cosseno.

**Por que ratio=0.90 no DISK e 0.75 no SIFT?**
> Descritores neurais são mais discriminativos: o melhor candidato tende a ser muito mais próximo que o segundo. Um threshold mais alto ainda filtra bem sem descartar matches corretos.

**Em que situação você escolheria ORB sobre DISK?**
> Sistemas embarcados, tempo real sem GPU, restrição de memória. O ORB não tem dependência de deep learning, é calculado em microsegundos e cabe em microcontroladores.

**O que é `pad_if_not_divisible=True` e por que é necessário?**
> O DISK usa uma arquitetura U-Net com stride 16, que exige que H e W sejam múltiplos de 16. O padding automático evita erros quando as imagens não satisfazem essa restrição.

---

## Tabela-resumo geral — os 4 métodos

| Método | Tipo | Dimensão | Métrica | KP (ursinho) | Matches | Velocidade | Cenário ideal |
|--------|------|----------|---------|-------------|---------|-----------|--------------|
| SIFT | Float clássico | 128 floats | NORM_L2 | 5535 | 97 (r=0.75) | Lento | Precisão, offline |
| ORB | Binário clássico | 256 bits | NORM_HAMMING | 2000 | 17 (r=0.75) | Muito rápido | Tempo real, embarcado |
| AKAZE | Binário clássico | 488 bits | NORM_HAMMING | 1636 | 36 (r=0.75) | Rápido | Equilíbrio |
| DISK | Float neural | 128 floats | NORM_L2 | 2048 | 58 (r=0.90) | Lento (CPU) | Cenários difíceis, GPU |

---

## Perguntas transversais

**O que é um keypoint?**
> Ponto na imagem com localização (x, y), escala e orientação, detectado por possuir estrutura local discriminativa (canto, blob, extremo de gradiente).

**O que é um descritor?**
> Vetor numérico que resume a aparência local ao redor de um keypoint de forma compacta e (idealmente) invariante a transformações.

**O que é o Ratio Test de Lowe?**
> Filtro que aceita um match só quando o melhor candidato é significativamente mais próximo que o segundo: `d1 < ratio × d2`. Elimina correspondências ambíguas. `ratio=0.75` elimina ~90% dos falsos preservando ~90% dos verdadeiros.

**Float vs. binário: qual escolher?**
> Float (SIFT, DISK): maior precisão, mais memória, distância euclidiana. Binário (ORB, AKAZE): menor memória, distância Hamming, muito mais rápido. Escolha depende de latência, hardware e variabilidade da cena.

**Quando um método clássico falha e um neural ajuda?**
> Mudanças drásticas de iluminação, estação ou ponto de vista extremo — situações onde gradientes mudam radicalmente mas a semântica persiste. Clássicos dependem de gradientes estáveis; neurais aprendem representações mais invariantes a partir de dados reais.

**Qual a diferença entre RANSAC e Ratio Test?**
> Ratio Test filtra matches ambíguos *antes* de estimar a geometria. RANSAC filtra outliers *durante* a estimativa da homografia, usando consistência geométrica (não apenas distância de descritor).
