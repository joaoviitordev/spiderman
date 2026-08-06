# Spider-Man: Um Novo Dia — Landing Page

Landing page conceitual (fan-made) do novo filme do Homem-Aranha, construída em
HTML, CSS e JavaScript puros, com todas as animações dirigidas pela rolagem via
**GSAP**.

> Projeto de estudo/portfólio. Não tem vínculo com a Marvel, a Sony ou a Disney;
> as marcas e o material audiovisual pertencem aos seus respectivos donos.

---

## Sumário

- [Stack](#stack)
- [Como rodar](#como-rodar)
- [Estrutura](#estrutura)
- [Como a página funciona](#como-a-página-funciona)
- [Elenco](#elenco)
- [Performance](#performance)
- [Ajustes rápidos](#ajustes-rápidos)
- [Acessibilidade](#acessibilidade)
- [Movimento reduzido](#movimento-reduzido)
- [Redimensionamento](#redimensionamento)

---

## Stack

| Camada | O que foi usado |
| --- | --- |
| Marcação | HTML5 semântico, SVG inline (`clipPath`) |
| Estilo | CSS puro — `container queries`, unidade fluida `--u`, `mix-blend-mode`, `clip-path` |
| Comportamento | JavaScript (ES6+), sem build e sem framework |
| Animação | [GSAP 3.13](https://gsap.com/) + `ScrollTrigger`, `ScrollSmoother`, `SplitText` (via CDN) |
| Mídia | Canvas 2D para a sequência de frames, `<video>` com player customizado |
| Tipografia | `Good Times` (local, `.otf`) + `Poppins` (Google Fonts) |
| IA | Apoio na geração/tratamento de arte e no desenvolvimento |

Sem `package.json`, sem bundler, sem dependência instalada: é só abrir.

## Como rodar

Precisa de um servidor local — o vídeo, os frames e a fonte local não carregam
bem em `file://`.

**VS Code + Live Server** (já configurado na porta `5501` em
[.vscode/settings.json](.vscode/settings.json)):

```text
Botão direito em index.html → "Open with Live Server"
```

**Ou qualquer servidor estático:**

```bash
npx serve .
# ou
python -m http.server 5501
```

Depois acesse `http://localhost:5501`.

## Estrutura

```text
spiderMan/
├── index.html                # as duas seções: hero e elenco
├── style.css                 # layout fluido, shapes e estados do player
├── main.js                   # timelines GSAP + modo de movimento reduzido
├── README.md
└── assets/
    ├── frames/               # 59 JPGs da sequência controlada pelo scroll
    ├── elenco/               # img1..img5.webp — fotos do elenco
    ├── trailer.mp4           # trailer usado no reveal
    ├── titulo-spider-man.svg # logo do filme
    ├── logotipo-marvel.svg   # marca no canto superior
    ├── logotipo-aranha.svg
    ├── mao-homem-aranha.svg  # selo "role a página"
    ├── texto-circular.svg
    ├── aranha-indicador.svg  # aranha da barra de progresso do elenco
    ├── bg.webp / bg-container.webp
    └── good-times.otf
```

## Como a página funciona

A rolagem inteira passa pelo `ScrollSmoother` (2s de suavização), por isso todo
o conteúdo vive dentro de `#smooth-wrapper > #smooth-content`.

O que vem a seguir descreve o comportamento padrão; quem pede
[movimento reduzido](#movimento-reduzido) recebe uma versão parada da mesma
página.

### Seção 1 — Hero

A hero fica **pinnada** por 600% de altura de tela, numa única timeline dividida
em dois atos:

**Ato 1 — a sequência de frames**
Os 59 JPGs são desenhados num `<canvas>` em modo *cover*. O GSAP não troca de
imagem: ele anima um valor fracionário (`playhead.frame`, ex.: `12.4`) e o
`render()` mistura o frame atual com o seguinte por `globalAlpha`. É esse
crossfade que faz 59 imagens parecerem vídeo.

No mesmo intervalo, as três sinopses se trocam **letra a letra em ordem
aleatória** (`SplitText` + `stagger: { from: "random" }`). Como a arte começa
clara e termina escura, dois tweens curtos viram a cor do texto (~44% da
timeline) e invertem o selo circular (~68%) — valores medidos nos próprios
frames.

**Ato 2 — o reveal do trailer**
Uma máscara abre do centro: primeiro a altura (vira uma faixa horizontal),
depois a largura. O logo e a sinopse não são animados por tempo — eles são
**empurrados** pela borda da máscara: enquanto a borda não os alcança, ficam
parados; a partir do encontro, andam colados nela
([main.js:224-232](main.js#L224-L232)). O frame do vídeo já nasce com a altura
final, então o trailer não estica: só é revelado.

### O player

Tem três estados:

- **dormindo** — `preload="none"`, nada baixado. O `<video>` fica assim até a
  rolagem chegar a 85% do ato 1, quando `player.arm()` libera o download;
- **prévia** — mudo, em loop, com overlay escuro; é o que roda enquanto a
  máscara abre;
- **assistindo** — começa do zero, com som, overlay some e os controles entram.

O botão de play só fica clicável com a máscara 100% aberta. Sair da hero (para
cima ou para baixo) devolve o trailer à prévia, para o áudio nunca continuar
fora da tela. Controles: play/pause, seek (mouse, toque e setas do teclado),
mute, tela cheia e espaço para pausar.

### Seção 2 — Elenco

Também pinnada. Cada troca move três coisas em sincronia: a foto é revelada de
cima para baixo (quem cresce é a máscara — a imagem já nasce no tamanho final),
o nome faz crossfade vertical, e a barra de progresso preenche enquanto a aranha
desce até o nome ativo.

A posição de cada parada da aranha é **medida no DOM** (`stopPositions`), não
chutada: mudar a quantidade de nomes não quebra o alinhamento.

## Elenco

| # | Ator | Personagem | Foto |
| --- | --- | --- | --- |
| 1 | Tom Holland | Peter Parker / Homem-Aranha | `elenco/img1.webp` |
| 2 | Zendaya | Michelle “MJ” Jones | `elenco/img2.webp` |
| 3 | Jacob Batalon | Ned Leeds | `elenco/img3.webp` |
| 4 | Jon Bernthal | Frank Castle / Justiceiro | `elenco/img4.webp` |
| 5 | Sadie Sink | Personagem não revelado | card de mistério |
| 6 | Mark Ruffalo | Bruce Banner / Hulk | `elenco/img5.webp` |

O papel da Sadie Sink não foi divulgado, então o slot 5 usa um **card de
mistério** no lugar da foto: fundo em degradê com o logotipo da aranha e a
legenda "personagem não revelado". Para trocar por uma foto de verdade, basta
substituir o `.cast__shot--mystery` por um `.cast__shot` com `<img>`, igual aos
outros.

## Performance

O gargalo era o `trailer.mp4`: **18 MB**, contra 2,3 MB de todos os 59 frames
somados. Três medidas atacam isso:

- **Trailer sob demanda** — o `<video>` nasce com `preload="none"` e sem
  `autoplay`. O download só começa quando a rolagem passa de `ARM_AT` (85% do
  ato 1), com folga para bufferizar antes de a máscara abrir. Quem só olha a
  hero e sai não baixa 18 MB.
- **Frames em fila** — em HTTP/2 os 59 `src` disparados de uma vez baixam todos
  em paralelo, e os primeiros frames (os que aparecem primeiro) chegam junto
  com os últimos. A fila mantém a ordem com no máximo `PARALLEL_LOADS = 4`
  downloads simultâneos, então o começo da sequência fica pronto quase de
  imediato.
- **Fotos do elenco** — `loading="lazy"` da segunda em diante; a primeira é
  `eager` com `fetchpriority="low"` para não competir com a hero.

O `<video>` também ganhou `playsinline`, sem o qual o iOS abriria o trailer em
tela cheia por conta própria.

## Ajustes rápidos

**Trocar os nomes do elenco** — o JS não tem nenhum nome escrito nele. Edite só
o HTML, em três lugares que precisam ficar na mesma ordem **e com a mesma
quantidade de itens**:

1. os blocos `.cast__slide` (nome + papel);
2. os `<li class="cast__item">` da lista lateral;
3. os `.cast__shot` em `.cast__media` (e o `alt` de cada `<img>`).

Para mudar a **quantidade** de atores, acrescente/remova um item em cada um dos
três — o `STOPS` se ajusta sozinho ([main.js:615](main.js#L615)) e a aranha
continua parando na altura certa, porque as posições são medidas no DOM.

**Ritmo das animações** — constantes no topo de cada bloco de
[main.js](main.js):

| Constante | Efeito |
| --- | --- |
| `ACT_1` / `ACT_2` | proporção entre a sequência de frames e o reveal |
| `SCROLL_PER_UNIT` | quanto de rolagem vale uma unidade de timeline |
| `CHAR_FADE` / `CHAR_SPREAD` | velocidade e espalhamento da troca letra a letra |
| `CAST_DURATION` | duração total da seção de elenco |
| `FRAME_COUNT` | número de imagens em `assets/frames/` |
| `FOCUS_X` / `FOCUS_Y` | enquadramento do canvas (equivale a `background-position`) |

**Escala** — o CSS usa `--u: calc(100cqw / 1920)`: o design foi feito em
1920px e tudo escala proporcionalmente. Há um breakpoint em `700px`.

**Desvio do Figma** — dois valores da coluna de elenco mudaram para acomodar
nomes reais: a coluna passou de 333u para 453u e o `.cast__name` de 62u para
56u. Em 62u/333u, "Jacob Batalon" quebrava em duas linhas por cima do seletor.

## Acessibilidade

- `SplitText` roda com `aria: "auto"`, então leitores de tela continuam lendo os
  parágrafos originais, não as letras soltas.
- O seek do player é um `role="slider"` navegável por teclado, com
  `aria-valuenow` atualizado.
- Arte decorativa marcada com `aria-hidden` e `alt=""`.
- O nome ativo do elenco recebe `aria-current="true"`.
- Toda a experiência tem um caminho alternativo sem movimento — abaixo.

## Movimento reduzido

Com `prefers-reduced-motion: reduce` no sistema, a página serve o **mesmo
conteúdo sem sequestrar a rolagem**. Nada de `ScrollSmoother`, de `pin` nem de
`scrub`:

| | Normal | Reduzido |
| --- | --- | --- |
| Rolagem | suavizada (2s) | nativa |
| Hero | 600% pinnada, 59 frames | pôster estático (1º frame) |
| Sinopse | troca letra a letra | os três parágrafos empilhados, de uma vez |
| Trailer | reveal no ato 2 | abre pelo link **TRAILER** do menu, fecha no `Esc` |
| Elenco | pinnado, troca por scroll | lista clicável (mouse, `Tab`, `Enter`/espaço) |

Nenhuma informação some: as três sinopses passam a ser um texto corrido só (que
é como elas realmente se encadeiam) e os seis atores ficam a um clique.

## Redimensionamento

O `resize` refaz o canvas e a altura da foto do elenco **sempre**, mas só
reconstrói as timelines (que matam o `SplitText` e remedem tudo) quando a
**largura** muda — no mobile, a barra de endereço entrando e saindo muda a
altura o tempo todo e não deve disparar rebuild. O rebuild é debounced em 250ms,
e `orientationchange` tem um atraso próprio de 300ms porque nem sempre o evento
chega com as medidas novas.
