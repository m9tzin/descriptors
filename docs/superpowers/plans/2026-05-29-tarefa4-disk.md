# Tarefa 4 — Descritores Neurais DISK: Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Criar `Tarefa4.ipynb` que usa Kornia/DISK para detectar keypoints e extrair descritores neurais no par ursinho1/2.jpeg, comparando com SIFT/ORB/AKAZE numa tabela final com reflexão crítica.

**Architecture:** Notebook linear com 4 itens numerados espelhando Tarefa1/2/3. DISK roda via `kornia.feature.DISK.from_pretrained('depth')` em CPU; keypoints convertidos para cv2.KeyPoint para reutilizar drawMatches. Tabela final consolida valores das tarefas anteriores (hardcoded) com DISK (computado em tempo de execução).

**Tech Stack:** Python 3.12, OpenCV 4.13, PyTorch 2.12+cpu, Kornia 0.8.3 — `.venv/` do projeto.

---

## Arquivos

- **Criar:** `Tarefa4.ipynb`
- **Imagens:** `images/ursinho1.jpeg` (img1), `images/ursinho2.jpeg` (img2) — mesmo par da Tarefa 2

---

### Task 1: Cabeçalho, base teórica e setup

**Files:**
- Create: `Tarefa4.ipynb`

- [ ] **Step 1: Criar notebook com célula markdown de cabeçalho**

```markdown
# Tarefa 4 — Exploração: Descritores Neurais com Kornia (DISK)
**Atividade 3 · TECA2 20261 · Pontos de Interesse e seus Descritores**

**Alunos:** Henryque Oliveira, Matheus Marinho e Rodrigo Oliveira

Este notebook usa a biblioteca Kornia para carregar o modelo DISK pré-treinado, extrair keypoints e descritores neurais no mesmo par de imagens da Tarefa 2 e compará-los com os métodos clássicos.

### Base teórica

- **DISK (DIscrete Keypoints)**: modelo neural que aprende detector e descritor conjuntamente por aprendizado por reforço, otimizando diretamente a qualidade do matching. Ao contrário de SIFT/ORB, que aplicam regras matemáticas fixas sobre gradientes, o DISK aprende — a partir de dados — o que constitui um bom keypoint e um bom descritor.
- **Por que neurais superam os clássicos em cenários difíceis**: mudanças drásticas de iluminação (dia/noite), diferentes estações do ano, pontos de vista muito distantes — situações em que gradientes locais mudam radicalmente, mas a semântica da cena permanece. Modelos clássicos falham porque dependem de gradientes estáveis; modelos neurais aprendem representações mais invariantes.
- **Ratio test com limiares mais altos (0.85–0.95)**: descritores neurais são mais discriminativos — há menos ambiguidade entre os dois melhores candidatos — portanto um ratio maior ainda filtra bem os falsos positivos sem descartar os verdadeiros.
- **Por que DISK e não R2D2 ou SuperPoint?** O DISK tem boa integração com Kornia e tutorial oficial disponível. R2D2 e SuperPoint resolvem o mesmo problema com filosofias diferentes; em produção a escolha depende do dataset, da plataforma e da licença.

> Referências: Torralba, Freeman, Isola (2024) Cap. 11 · Tutorial Kornia DISK · Slides TECA2 20261, slide 20.
```

- [ ] **Step 2: Adicionar célula de código de setup**

```python
import cv2
import numpy as np
import matplotlib.pyplot as plt
import torch
import kornia
import kornia.feature as KF

print('OpenCV:', cv2.__version__)
print('torch:', torch.__version__)
print('kornia:', kornia.__version__)

device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
print('Dispositivo:', device)

PATH_IMG1 = 'images/ursinho1.jpeg'
PATH_IMG2 = 'images/ursinho2.jpeg'

def load(path):
    img = cv2.imread(path, cv2.IMREAD_COLOR)
    if img is None:
        raise FileNotFoundError(path)
    gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
    return img, gray

def load_tensor(path, device):
    img = cv2.imread(path)
    if img is None:
        raise FileNotFoundError(path)
    img = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
    t = torch.from_numpy(img).float() / 255.0
    return t.permute(2, 0, 1).unsqueeze(0).to(device)
```

- [ ] **Step 3: Executar e verificar**

```bash
cd "/home/m9t/Documents/ufg/computer-vision/Atividade 3/descriptors" && \
  .venv/bin/jupyter nbconvert --to notebook --execute --inplace \
  --ExecutePreprocessor.timeout=120 Tarefa4.ipynb 2>&1 | tail -3
```

Saída esperada:
```
OpenCV: 4.13.0
torch: 2.12.0+cpu
kornia: 0.8.3
Dispositivo: cpu
```

- [ ] **Step 4: Commit**

```bash
git add "Tarefa4.ipynb"
git commit -m "feat(tarefa4): adiciona cabeçalho, base teórica e setup"
```

---

### Task 2: Item 1 — Carregar imagens como tensores

**Files:**
- Modify: `Tarefa4.ipynb`

- [ ] **Step 1: Adicionar célula markdown**

```markdown
## Item 1 — Carregar imagens como tensores

O Kornia opera sobre tensores PyTorch no formato `[B, C, H, W]` com valores em `[0, 1]`. Carregamos o mesmo par de imagens da Tarefa 2 tanto como arrays OpenCV (para visualização com cv2.drawMatches) quanto como tensores (para o DISK).
```

- [ ] **Step 2: Adicionar célula de código**

```python
img1, gray1 = load(PATH_IMG1)
img2, gray2 = load(PATH_IMG2)
img1_t = load_tensor(PATH_IMG1, device)
img2_t = load_tensor(PATH_IMG2, device)

print(f'img1 OpenCV: {img1.shape[1]}×{img1.shape[0]} px')
print(f'img2 OpenCV: {img2.shape[1]}×{img2.shape[0]} px')
print(f'img1 tensor: {img1_t.shape}')
print(f'img2 tensor: {img2_t.shape}')

fig, axes = plt.subplots(1, 2, figsize=(12, 5))
axes[0].imshow(cv2.cvtColor(img1, cv2.COLOR_BGR2RGB))
axes[0].set_title('ursinho1')
axes[0].axis('off')
axes[1].imshow(cv2.cvtColor(img2, cv2.COLOR_BGR2RGB))
axes[1].set_title('ursinho2')
axes[1].axis('off')
plt.suptitle('Par de imagens (mesmo da Tarefa 2)', fontsize=13)
plt.tight_layout()
plt.show()
```

- [ ] **Step 3: Executar e verificar**

```bash
cd "/home/m9t/Documents/ufg/computer-vision/Atividade 3/descriptors" && \
  .venv/bin/jupyter nbconvert --to notebook --execute --inplace \
  --ExecutePreprocessor.timeout=120 Tarefa4.ipynb 2>&1 | tail -3
```

Saída esperada: dimensões impressas + `torch.Size([1, 3, 1280, 960])` + figura do par.

- [ ] **Step 4: Commit**

```bash
git add "Tarefa4.ipynb"
git commit -m "feat(tarefa4): adiciona Item 1 - carregamento como tensores"
```

---

### Task 3: Item 2 — Rodar o DISK

**Files:**
- Modify: `Tarefa4.ipynb`

- [ ] **Step 1: Adicionar célula markdown do Item 2**

```markdown
## Item 2 — Rodar o DISK

Carregamos o modelo DISK pré-treinado (`depth` checkpoint) e extraímos até 2048 keypoints por imagem. O download dos pesos ocorre automaticamente na primeira execução (~30 MB). O DISK gera descritores float de 128 dimensões — mesma dimensão do SIFT, mas aprendidos em vez de calculados manualmente.
```

- [ ] **Step 2: Adicionar célula de código**

```python
disk = KF.DISK.from_pretrained('depth').to(device).eval()

with torch.inference_mode():
    feats1 = disk(img1_t, 2048, pad_if_not_divisible=True)[0]
    feats2 = disk(img2_t, 2048, pad_if_not_divisible=True)[0]

kp1_disk = feats1.keypoints.cpu().numpy()
d1_disk  = feats1.descriptors.cpu().numpy()
kp2_disk = feats2.keypoints.cpu().numpy()
d2_disk  = feats2.descriptors.cpu().numpy()

print(f'DISK img1: {len(kp1_disk)} kp | desc shape: {d1_disk.shape}')
print(f'DISK img2: {len(kp2_disk)} kp | desc shape: {d2_disk.shape}')
```

- [ ] **Step 3: Executar e verificar**

```bash
cd "/home/m9t/Documents/ufg/computer-vision/Atividade 3/descriptors" && \
  .venv/bin/jupyter nbconvert --to notebook --execute --inplace \
  --ExecutePreprocessor.timeout=300 Tarefa4.ipynb 2>&1 | tail -3
```

Saída esperada (valores aproximados):
```
DISK img1: XXXX kp | desc shape: (XXXX, 128)
DISK img2: XXXX kp | desc shape: (XXXX, 128)
```
O número de keypoints será ≤ 2048. O shape do descritor deve ser `(N, 128)`.

- [ ] **Step 4: Commit**

```bash
git add "Tarefa4.ipynb"
git commit -m "feat(tarefa4): adiciona Item 2 - extração de keypoints DISK"
```

---

### Task 4: Item 3 — Matching e visualização

**Files:**
- Modify: `Tarefa4.ipynb`

- [ ] **Step 1: Adicionar célula markdown do Item 3**

```markdown
## Item 3 — Matching e visualização

Convertemos os keypoints DISK (tensores float `[N, 2]`) para `cv2.KeyPoint` a fim de usar `cv2.drawMatches`. O matching usa `NORM_L2` (descritores float) com ratio=0.90 — limiar mais alto que os clássicos pois os descritores neurais são mais discriminativos.
```

- [ ] **Step 2: Adicionar célula de código**

```python
# Converter keypoints DISK para cv2.KeyPoint
kp1_cv = [cv2.KeyPoint(float(x), float(y), 1) for x, y in kp1_disk]
kp2_cv = [cv2.KeyPoint(float(x), float(y), 1) for x, y in kp2_disk]

# Matching com ratio=0.90
bf = cv2.BFMatcher(cv2.NORM_L2)
pairs = bf.knnMatch(d1_disk, d2_disk, k=2)
good_disk = [m for m, n in pairs if m.distance < 0.90 * n.distance]
print(f'DISK: {len(kp1_cv)} kp | {len(good_disk)}/{len(pairs)} bons (ratio=0.90)')

img_matches = cv2.drawMatches(
    img1, kp1_cv, img2, kp2_cv, good_disk, None,
    flags=cv2.DrawMatchesFlags_NOT_DRAW_SINGLE_POINTS)
plt.figure(figsize=(14, 5))
plt.imshow(cv2.cvtColor(img_matches, cv2.COLOR_BGR2RGB))
plt.title(f'DISK — {len(good_disk)} matches bons (ratio=0.90)')
plt.axis('off')
plt.tight_layout()
plt.show()
```

- [ ] **Step 3: Executar e verificar**

```bash
cd "/home/m9t/Documents/ufg/computer-vision/Atividade 3/descriptors" && \
  .venv/bin/jupyter nbconvert --to notebook --execute --inplace \
  --ExecutePreprocessor.timeout=300 Tarefa4.ipynb 2>&1 | tail -3
```

Saída esperada: linha de contagem impressa + figura de matches sem erros.

- [ ] **Step 4: Commit**

```bash
git add "Tarefa4.ipynb"
git commit -m "feat(tarefa4): adiciona Item 3 - matching e visualização DISK"
```

---

### Task 5: Item 4 — Tabela comparativa final, reflexão e conclusão

**Files:**
- Modify: `Tarefa4.ipynb`

- [ ] **Step 1: Adicionar célula markdown do Item 4**

```markdown
## Item 4 — Tabela comparativa final e reflexão crítica
```

- [ ] **Step 2: Adicionar célula de código que imprime a tabela**

Os valores de SIFT/ORB/AKAZE vêm da Tarefa 2 (par ursinho, ratio=0.75). DISK usa os valores computados neste notebook.

```python
# Valores Tarefa 2 (par ursinho, ratio=0.75)
sift_kp, sift_matches = 5535, 97
orb_kp,  orb_matches  = 2000, 17
akaze_kp, akaze_matches = 1636, 36

print('Tabela comparativa final:')
print(f'{"Método":<8} {"Tipo":<20} {"KP":>6} {"Matches":>8}')
print('-' * 48)
rows = [
    ('SIFT',  'Clássico / float',   sift_kp,       sift_matches),
    ('ORB',   'Clássico / binário', orb_kp,        orb_matches),
    ('AKAZE', 'Clássico / binário', akaze_kp,      akaze_matches),
    ('DISK',  'Neural / float',     len(kp1_disk), len(good_disk)),
]
for name, tipo, kp, m in rows:
    print(f'{name:<8} {tipo:<20} {kp:>6} {m:>8}')
```

- [ ] **Step 3: Executar a célula e anotar os valores reais do DISK**

Saída esperada (exemplo):
```
Tabela comparativa final:
Método   Tipo                    KP  Matches
------------------------------------------------
SIFT     Clássico / float      5535       97
ORB      Clássico / binário    2000       17
AKAZE    Clássico / binário    1636       36
DISK     Neural / float        XXXX     XXXX
```
Anote os valores reais de KP e Matches do DISK para preencher o markdown a seguir.

- [ ] **Step 4: Adicionar célula markdown com tabela preenchida, reflexão e conclusão**

Use os valores reais do DISK obtidos no Step 3. Substitua os `XXXX` pelos números reais:

```markdown
| Método | Tipo              | KP   | Matches |
|--------|-------------------|------|---------|
| SIFT   | Clássico / float  | 5535 | 97      |
| ORB    | Clássico / binário| 2000 | 17      |
| AKAZE  | Clássico / binário| 1636 | 36      |
| DISK   | Neural / float    | XXXX | XXXX    |

### Reflexão crítica

**1. Em que cenário o DISK foi visivelmente melhor que o SIFT? Em que cenário os resultados foram parecidos?**

[Preencher após execução: se DISK teve mais matches bons que o SIFT (97), descreva que o modelo neural foi mais eficaz neste par de imagens com mudança de ângulo leve. Se os resultados foram similares, descreva que para cenas com boa iluminação e textura estável os clássicos já são suficientes.]

**2. Quais são as desvantagens práticas de usar DISK em comparação com ORB em sistema embarcado (câmera de robô, drone)?**

O DISK requer PyTorch e pesos pré-treinados (~30 MB), tornando o deployment em microcontroladores inviável. A inferência em CPU é significativamente mais lenta que o ORB, que usa operações bit-a-bit e roda em tempo real mesmo em hardware limitado. O ORB não depende de conexão para baixar modelos e seu footprint de memória é mínimo.

**3. Se você fosse escolher um único método para um sistema de vigilância indoor em tempo real, qual escolheria e por quê?**

[Preencher após execução: se hardware disponível tem GPU → DISK; se embarcado → ORB. Justificar com os dados da tabela observados.]

## Conclusão da Tarefa 4

- O DISK, como descritor neural float, utiliza a mesma métrica de distância do SIFT (NORM_L2), porém com representações aprendidas em vez de calculadas — o que o torna mais robusto em cenários difíceis.
- A tabela final ilustra o espectro completo: binários rápidos (ORB/AKAZE) → float clássico (SIFT) → neural moderno (DISK), cada um com trade-offs distintos de velocidade, tamanho e qualidade.
- A escolha do método ideal depende dos requisitos do sistema: latência, hardware disponível, variabilidade das condições visuais e necessidade de precisão.
```

**IMPORTANTE:** Após executar o notebook e ver os valores reais do DISK, edite a célula markdown acima para:
- Substituir `XXXX` pelos números reais na tabela
- Preencher as partes `[Preencher após execução: ...]` das perguntas 1 e 3 com análise baseada nos dados observados (1 parágrafo cada)

- [ ] **Step 5: Executar notebook completo**

```bash
cd "/home/m9t/Documents/ufg/computer-vision/Atividade 3/descriptors" && \
  .venv/bin/jupyter nbconvert --to notebook --execute --inplace \
  --ExecutePreprocessor.timeout=300 Tarefa4.ipynb 2>&1 | tail -3
```

Saída esperada: todas as células executam sem erros.

- [ ] **Step 6: Commit final**

```bash
git add "Tarefa4.ipynb"
git commit -m "feat(tarefa4): adiciona Item 4 - tabela comparativa final e reflexão"
```

---

## Self-Review

**Cobertura da spec:**
- ✅ Base teórica: DISK, neurais vs clássicos, ratio 0.85–0.95 — Task 1
- ✅ Setup com torch, kornia, load_tensor — Task 1
- ✅ Item 1: carregar como tensores [1,3,H,W] — Task 2
- ✅ Item 2: DISK.from_pretrained('depth'), 2048 kp, imprimir shape — Task 3
- ✅ Item 3: conversão para cv2.KeyPoint, BFMatcher NORM_L2, ratio=0.90, drawMatches — Task 4
- ✅ Item 4: tabela final com SIFT/ORB/AKAZE/DISK, 3 perguntas de reflexão — Task 5
- ✅ Conclusão — Task 5

**Consistência de tipos:**
- `kp1_disk` numpy array (N,2), `d1_disk` numpy array (N,128) definidos em Task 3, usados em Tasks 4 e 5 ✅
- `good_disk` lista de cv2.DMatch definida em Task 4, usada em Task 5 ✅
- `kp1_cv` lista de cv2.KeyPoint definida em Task 4, usada em drawMatches Task 4 ✅

**Sem placeholders de código:** instruções de preenchimento das respostas qualitativas são explícitas (preencher com dados observados), não são "TBD" ✅
