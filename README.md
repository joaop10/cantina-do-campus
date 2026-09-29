# Cantina do Campus — Atividade do Encontro 4

Correção dos seis erros plantados em `cantina/style.css` e conclusão do layout
com flexbox.

- **Arquivo corrigido:** [`cantina/style.css`](cantina/style.css)
- **HTML:** [`cantina/index.html`](cantina/index.html) — **não foi alterado**
  (aparece uma única vez no histórico, no commit inicial)
- **Referência:** [`cantina/referencia.png`](cantina/referencia.png)
- **Sem `!important`:** nenhuma ocorrência no arquivo de estilo

Para abrir, basta abrir `cantina/index.html` no navegador. O histórico tem um
commit por erro corrigido, na ordem da tabela do enunciado.

---

## Resumo dos seis erros

| # | Tema | O que estava errado | Por que acontecia | Como corrigi |
|---|---|---|---|---|
| 1 | Cascata | `h1 { color: #14213d !important }` | A marcação de prioridade tirava a declaração da cascata normal, mas nenhuma outra regra disputava a cor do `h1` — ela vencia uma disputa que não existia | Removi a marcação. A cor final é exatamente a mesma, e o título volta a ser ajustável por qualquer regra futura |
| 2 | Especificidade | O preço em promoção aparecia azul, não vermelho | `#cardapio .preco` pontua **1,1,0** e `.promocao` pontua **0,1,0**. Um ID nunca perde para classes, então `.promocao` ficava riscada no DevTools | Troquei `#cardapio .preco` por `.preco` (**0,1,0**). Os dois empatam e vence quem vem depois no arquivo: `.promocao` |
| 3 | Box model | Os cartões estouravam a página e criavam rolagem lateral | `.cartao` juntava `width: 100%` com `padding: 20px` e `border: 2px`. No `content-box` padrão esses 44px entram **por fora**: 1052 + 44 = **1096px** dentro de um espaço de 1052px | Apliquei `box-sizing: border-box` no reset (`*, *::before, *::after`), fazendo a largura declarada já incluir padding e borda |
| 4 | Contraste | O subtítulo quase sumia no fundo branco | `#c4c4c4` sobre branco mede **1,74:1**. A WCAG exige **4,5:1** para texto normal — estava a menos de metade do mínimo | Troquei por `#4b5563`, que mede **7,56:1** (passa em AA e AAA) e é a cor usada na `referencia.png` |
| 5 | Foco | Navegando com Tab, o foco ficava invisível no menu | `.menu a:focus { outline: none }` apagava o único indicador de posição do teclado, sem colocar nada no lugar | Troquei por um anel âmbar de 3px com `outline-offset: 4px`, em `:focus-visible` (navegação por teclado) e `:focus` (compatibilidade). Mede **7,9:1** sobre o azul do topo |
| 6 | Alinhamento | O menu estava empurrado para a direita "no chute" | `.menu li { margin-left: 610px }` é um número mágico: só acertava nesta largura exata de tela, saía da página em janelas menores e, por valer para **cada** `li`, separava os três links | Removi a regra inteira. A posição passou a ser calculada pelo flexbox (erro 6 + Passo 4) |

---

## Detalhe de cada correção

### 1. Cascata — a declaração que gritava mais alto do que precisava

```css
/* antes */
h1 {
  color: #14213d !important;
  margin-top: 0;
}

/* depois */
h1 {
  color: #14213d;
  margin-top: 0;
}
```

`!important` promove a declaração para uma camada acima da cascata normal, onde
regras comuns não conseguem alcançá-la. O detalhe é que **nenhuma outra regra
da folha seleciona `h1`** — não havia conflito nenhum para resolver. O efeito
visual é zero; o efeito real é dívida técnica: quem tentasse ajustar o título
depois ia descobrir que só outro `!important` funcionaria.

### 2. Especificidade — por que o vermelho perdia

O HTML do prato em promoção é `class="preco promocao"`, ou seja, as duas regras
casam com o mesmo elemento. A disputa é decidida pela pontuação do seletor,
comparada da esquerda para a direita:

| Seletor | IDs | Classes | Elementos | Resultado |
|---|---|---|---|---|
| `#cardapio .preco` | **1** | 1 | 0 | vencia → pintava de azul |
| `.promocao` | **0** | 1 | 0 | perdia → ficava riscada no DevTools |

Como a primeira coluna já decide, nenhuma quantidade de classes empata com um
ID. Baixando o seletor para `.preco`, os dois ficam em `0,1,0` e o desempate
passa a ser a ordem do código — por isso **`.promocao` precisa continuar depois
de `.preco` no arquivo**, e deixei um comentário avisando disso.

Os outros dois preços continuam escuros porque só têm a classe `.preco`.

### 3. Box model — a conta que criava a rolagem lateral

`.conteudo` tem `padding: 24px`, então em uma janela de 1100px sobram 1052px de
espaço interno. A largura real do cartão era:

```
1052 (width: 100%) + 20 + 20 (padding) + 2 + 2 (border) = 1096px
```

44px maiores que os 1052 disponíveis, começando em x=24 → a borda direita caía
em 1120px numa janela de 1100px. Daí a rolagem.

`box-sizing: border-box` inverte o sentido da conta: padding e borda passam a
ser descontados **para dentro** da largura declarada, e 100% volta a significar
100%. Apliquei no reset universal porque a mesma armadilha esperaria em
qualquer componente novo, não só neste cartão.

Medido no navegador a 760px, simulando o erro e desfazendo:

| Estado | `scrollWidth` | `clientWidth` | Rolagem lateral? |
|---|---|---|---|
| Com `content-box` (erro original) | 765 | 745 | **sim, 20px** |
| Com `border-box` (corrigido) | 745 | 745 | não |

### 4. Contraste — medindo em vez de chutar

A razão de contraste é calculada a partir da **luminância relativa**, não da
distância entre os códigos hexadecimais — por isso um cinza que "parece ok"
pode reprovar com folga.

| Cor | Contra o branco | Mínimo AA (4,5:1) |
|---|---|---|
| `#c4c4c4` (antes) | 1,74:1 | reprova |
| `#4b5563` (depois) | 7,56:1 | passa (AA e AAA) |

Conferi também o resto da página: rodapé `#c7cee0` sobre `#14213d` = 10,1:1,
links do menu em branco sobre o topo = 16,0:1, botão `#14213d` sobre `#fca311`
= 7,9:1 e o preço em promoção `#b91c1c` sobre branco = 6,5:1. Todos passam.

### 5. Foco — devolvendo o indicador de teclado

```css
/* antes */
.menu a:focus {
  outline: none;
}

/* depois */
.menu a:focus,
.menu a:focus-visible {
  outline: 3px solid #fca311;
  outline-offset: 4px;
  border-radius: 2px;
}
```

`outline: none` é a regressão de acessibilidade mais comum em CSS: apaga a única
pista que quem navega por teclado tem de onde está na página. Normalmente é
escrita para esconder o anel no clique do mouse — e `:focus-visible` resolve
exatamente isso, mostrando o anel na navegação por teclado e escondendo no
clique. Mantive também `:focus` para navegadores sem suporte a `:focus-visible`.

O âmbar `#fca311` foi escolhido por já ser a cor de destaque da página e por
medir 7,9:1 sobre o azul do topo.

### 6. Alinhamento — trocando o número mágico pelo cálculo

`margin-left: 610px` só "funciona" na largura em que foi medido no olho. Em uma
janela de 800px o menu saía da tela; e como a margem valia para **cada** `li`,
os três links ficavam separados por 610px cada um, e não alinhados à direita.

A regra saiu por completo. O espaço entre os itens virou `gap: 24px` no
`.menu ul` e a posição do menu virou `justify-content: space-between` no
`.topo` — ambos calculados a partir do espaço disponível, em qualquer largura.

---

## Passo 4 — o layout com flexbox

### Topo: marca de um lado, menu do outro

```css
.topo {
  display: flex;
  align-items: center;             /* alinha marca e menu pelo meio */
  justify-content: space-between;  /* joga o espaço livre para o meio */
  flex-wrap: wrap;                 /* menu desce em telas bem estreitas */
  gap: 12px;
}

.menu ul {
  display: flex;
  flex-wrap: wrap;
  gap: 24px;                       /* no lugar das margens */
}
```

`space-between` gruda o primeiro item na borda esquerda e o último na direita,
distribuindo todo o resto no meio. É o que o `margin-left: 610px` tentava imitar
— só que calculado, e portanto correto em qualquer largura.

### Cartões: três lado a lado, quebrando quando falta espaço

```css
.cartoes {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
}

.cartao {
  flex: 1 1 260px;   /* grow / shrink / basis */
}
```

`flex: 1 1 260px` significa: cada cartão *quer* 260px (`flex-basis`), pode
crescer para ocupar a sobra (`flex-grow: 1`) e pode encolher se faltar espaço
(`flex-shrink: 1`). Com `flex-wrap: wrap`, quando três cartões de 260px mais os
gaps não cabem mais, o excedente desce para a linha de baixo — sem nenhuma
media query.

Uma porcentagem como `width: 33%` não daria conta, porque teria que embutir na
conta o tamanho dos gaps. O `flex-grow` resolve isso sozinho: a 1100px os três
cartões ficam com **340px** cada (3 × 340 + 2 × 16 = 1052 ✓), idêntico à
referência.

O `gap` também substituiu o `margin-bottom: 16px` do cartão: margem espaça por
um lado só e sobra depois do último item, enquanto `gap` é espaçamento do
container, aplicado **apenas entre** os itens.

| Largura da janela | Cartões por linha |
|---|---|
| 1100px | 3 |
| 760px | 2 |
| 420px | 1 |

---

## Lista de verificação

| Item | Como verifiquei | Resultado |
|---|---|---|
| Sem `!important` | busca no arquivo de estilo | 0 ocorrências |
| Promoção | `getComputedStyle` dos três preços | `#b91c1c` no da promoção, `#14213d` nos outros dois |
| Largura | `scrollWidth` vs `clientWidth` a 1100, 760 e 420px | iguais nas três — sem rolagem lateral |
| Contraste | razão calculada para todos os textos | mínimo da página = 6,5:1 (todos ≥ 4,5:1) |
| Foco | Tab no menu + `getComputedStyle` | anel sólido de 3px `#fca311`, visível |
| Topo | medição das caixas | marca em x=24, menu terminando em x=1076, alinhados pelo flexbox |
| Cartões | medição em três larguras | 3 / 2 / 1 por linha; 340px cada a 1100px |
| HTML intacto | `git log -- cantina/index.html` | aparece só no commit inicial |
| README | este arquivo | uma explicação para cada um dos seis erros |

### Comparação com a referência

Renderizei a página a 1100×791 e comparei pixel a pixel com `referencia.png`:
**todas as bordas dos cartões batem exatamente** (0 pixels de diferença nas
linhas y=255, 256, 445, 446 e nas colunas x=24, 25, 363, 380, 1075). As
diferenças restantes são apenas franjas de antisserrilhamento subpixel nas
letras (ex.: `#fffff2` → `#ffe7ba`), próprias do rasterizador de fontes do
sistema, e não de CSS.
