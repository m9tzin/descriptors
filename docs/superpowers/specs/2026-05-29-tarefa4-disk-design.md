---
name: tarefa4-disk-design
description: Design do Tarefa4.ipynb — Descritores Neurais com Kornia (DISK), comparação final SIFT/ORB/AKAZE/DISK
metadata:
  type: project
---

# Design — Tarefa 4: Exploração — Descritores Neurais com Kornia (DISK)

## Contexto

Parte da Atividade 3 (TECA2 20261). Entrega via Google Classroom até 30/05/2026. Arquivo de entrega: `Tarefa4.ipynb`. Apresentação fica por conta do aluno.

## Ambiente

- torch 2.12.0+cpu e kornia 0.8.3 já instalados no `.venv/` via `uv sync`
- Dispositivo: CPU (sem CUDA disponível localmente)
- Par de imagens: `images/ursinho1.jpeg` e `images/ursinho2.jpeg` (mesmo par da Tarefa 2)

## Estrutura do notebook (Opção A — espelha Tarefa1/2/3)

### Cabeçalho e base teórica (Markdown)
- Título, alunos, descrição.
- **DISK (DIscrete Keypoints)**: modelo neural que aprende detector e descritor conjuntamente por aprendizado por reforço, otimizando diretamente a qualidade do matching. Ao contrário de SIFT/ORB que usam regras matemáticas fixas, o DISK aprende a partir de dados o que é um bom keypoint.
- **Por que neurais superam os clássicos em cenários difíceis**: mudanças drásticas de iluminação, diferentes estações do ano, pontos de vista muito distantes — situações onde gradientes locais mudam radicalmente mas a semântica da cena permanece.
- **Ratio test com limiares mais altos (0.85–0.95)**: descritores neurais são mais discriminativos (menos ambiguidade entre vizinhos), então um ratio maior ainda filtra bem os falsos positivos.
- Referências: Torralba et al. (2024) Cap. 11; tutorial Kornia DISK.

### Setup (código)
- Imports: `cv2`, `numpy as np`, `matplotlib.pyplot as plt`, `torch`, `kornia`, `kornia.feature as KF`.
- Prints: versões de OpenCV, torch, kornia.
- Dispositivo: `device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')`.
- Constantes: `PATH_IMG1 = 'images/ursinho1.jpeg'`, `PATH_IMG2 = 'images/ursinho2.jpeg'`.
- Função `load(path)` → `(img_bgr, gray)` (mesma das tarefas anteriores).
- Função `load_tensor(path, device)`: lê com cv2, converte BGR→RGB, float32/255, `torch.from_numpy`, permute HWC→CHW, unsqueeze(0), move para device.

### Item 1 — Carregar imagens como tensores (código + markdown)
- Carrega img1/img2 como arrays OpenCV e como tensores.
- Exibe par lado a lado em RGB.
- Imprime shape dos tensores (esperado: `[1, 3, H, W]`).

### Item 2 — Rodar o DISK (código + markdown)
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
Markdown: explicação de que `n=2048` é o número máximo de keypoints por imagem e que o modelo baixa os pesos na primeira execução (~30 MB).

### Item 3 — Matching e visualização (código + markdown)
- Converte keypoints DISK para `cv2.KeyPoint`:
  ```python
  kp1_cv = [cv2.KeyPoint(float(x), float(y), 1) for x, y in kp1_disk]
  kp2_cv = [cv2.KeyPoint(float(x), float(y), 1) for x, y in kp2_disk]
  ```
- BFMatcher NORM_L2, knnMatch k=2, ratio=0.90.
- `cv2.drawMatches` com `NOT_DRAW_SINGLE_POINTS`.
- Imprime contagem de matches bons.
- Exibe figura de matches.

Markdown: por que ratio=0.90 em vez de 0.75 (descritores neurais mais discriminativos).

### Item 4 — Tabela comparativa final e reflexão (código + markdown)

**Código**: imprime tabela com valores reais (KP img1 e matches bons) para DISK, além dos valores das tarefas anteriores (hardcoded do par ursinho):

| Método | Tipo             | KP    | Matches |
|--------|-----------------|-------|---------|
| SIFT   | Clássico/float  | 5535  | 97      |
| ORB    | Clássico/binário| 2000  | 17      |
| AKAZE  | Clássico/binário| 1636  | 36      |
| DISK   | Neural/float    | —     | —       |

**Markdown**: tabela preenchida com valores reais + 3 perguntas de reflexão (máx. 1 parágrafo cada):
1. Em que cenário o DISK foi visivelmente melhor que o SIFT? Em que cenário os resultados foram parecidos?
2. Quais são as desvantagens práticas de usar DISK em comparação com ORB em sistema embarcado (câmera de robô, drone)?
3. Se você fosse escolher um único método para um sistema de vigilância indoor em tempo real, qual escolheria e por quê?

### Conclusão (Markdown)
Parágrafo curto comparando os 4 métodos com base nos dados observados. Destaca o trade-off neural vs. clássico.

## Critérios de avaliação cobertos

1. **Contextualização** — entendimento do DISK sem teoria de redes na base teórica.
2. **Implementação** — Kornia funcionando; keypoints e descritores extraídos corretamente.
3. **Comparação** — tabela final preenchida; matching visualizado para DISK e SIFT.
4. **Reflexão Crítica** — 3 perguntas respondidas com base nos resultados observados.

## Fora de escopo

- `Tarefa4.pptx` — o aluno fará ao final.
- LightGlue matcher (bônus opcional mencionado no enunciado) — não implementado.
- Teoria de redes neurais (backpropagation, arquitetura) — explicitamente excluída pelo enunciado.
