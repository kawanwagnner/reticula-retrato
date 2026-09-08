# Retícula de Retrato

Transforma uma foto em retrato de meio-tom: cada bloco de pixels vira um ponto cujo
raio é a luminância daquele bloco. Um arquivo, sem build, sem dependência.

Abra o `index.html` — ou sirva a pasta com `npx serve .`.

## Como usar

Solte um **PNG com fundo transparente**. É o canal alpha que diz onde a pessoa
termina; numa foto com fundo, a retícula preenche o quadro inteiro (a página avisa
quando detecta isso).

Os sete controles são os parâmetros reais do algoritmo — mexer neles é a melhor
forma de entender o que cada etapa faz:

| Controle | O que muda |
|---|---|
| **Bloco** | quantos pixels viram um ponto. Maior = grade grossa, cara de cartaz |
| **Raio máximo** | tamanho do ponto no branco puro |
| **Gama do raio** | acima de 1 engole os meios-tons e limpa a sombra |
| **Contraste** | abre a faixa antes de medir a luz |
| **Nitidez** | quanto da diferença contra o borrado volta — crava olho e boca |
| **Corte de alpha** | o quanto do recorte conta como fundo |
| **Duração** | tempo da animação de entrada |

A foto é lida no navegador e não sai dele: nada é enviado a lugar nenhum.

## O algoritmo

Seis etapas, em canvas 2D puro:

1. **Luminância** — `0.299R + 0.587G + 0.114B`, a fórmula que pesa o verde mais que
   o azul porque o olho enxerga assim.
2. **Contraste** — `(L − 128) × 1.4 + 128`. Sem isso a retícula sai cinzenta e sem forma.
3. **Nitidez** — compara com uma cópia borrada em 2 px e devolve a diferença
   amplificada em 1.8×. É o que faz olho, boca e contorno sobreviverem à grade.
4. **Blocos** — cada quadrado de 4×4 px vira uma célula com a média da luz e do alpha.
   Aqui a foto deixa de ser foto e vira grade.
5. **Raio** — `8 × luz^1.3`. Claro vira bolinha gorda, escuro encolhe até sumir, e é
   o sumiço que desenha a sombra.
6. **Recorte** — célula com alpha abaixo de 140 é pulada.

A entrada anima assim: cada ponto nasce numa posição aleatória
(`(1.3 × random − 0.15) × dimensão`), tem um atraso próprio de até `0.45`, e voa até
o seu lugar com easing cúbico de saída, crescendo de 45% a 100% do raio e de 12% a
100% da opacidade.

A dissolvida na base do retrato não faz parte do desenho — é CSS:
`mask-image: linear-gradient(to bottom, #000 58%, transparent 82%)`.

## Detalhes de implementação

- A foto é reduzida a **520 px** no maior lado antes de virar grade, para a densidade
  de pontos não depender do tamanho do arquivo de entrada.
- O canvas de trabalho é **4× maior** que a origem, e o resultado é exibido reduzido:
  é o que deixa a borda do ponto lisa em vez de serrilhada.
- A página abre com um busto sintético desenhado em código, só para mostrar o efeito
  funcionando. Não é foto de ninguém, e some quando você solta a sua.
- O botão de salvar usa uma âncora `download` comum. Dentro do visualizador de
  artefatos do Claude, onde a âncora é inerte, ele usa a capacidade `downloads`
  quando ela existe.

## Crédito

A técnica é do retrato do hero de [arif-hasan.vercel.app](https://arif-hasan.vercel.app/about).
Os números vieram de ler o bundle daquele site — os padrões aqui são exatamente os
dele, e o botão "voltar aos números do original" devolve todos de uma vez. Esta
ferramenta é uma reimplementação para estudo; nenhum código nem asset de lá foi copiado.
