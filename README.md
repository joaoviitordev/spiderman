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
- [Ajustes rápidos](#ajustes-rápidos)
- [Acessibilidade](#acessibilidade)
- [Pendências conhecidas](#pendências-conhecidas)

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

```
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

```
spiderMan/
├── index.html                # as duas seções: hero e elenco
├── style.css                 # layout fluido, shapes e estados do player
├── main.js                   # todas as timelines GSAP
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
([main.js:175-182](main.js#L175-L182)). O frame do vídeo já nasce com a altura
final, então o trailer não estica: só é revelado.

### O player

Tem dois modos ([main.js:190-325](main.js#L190-L325)):

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

| Ator | Personagem |
| --- | --- |
| Tom Holland | Peter Parker / Homem-Aranha |
| Zendaya | MJ |
| Jacob Batalon | Ned Leeds |
| Jon Bernthal | Frank Castle / Justiceiro |
| Sadie Sink | — |
| Mark Ruffalo | Bruce Banner / Hulk |

## Ajustes rápidos

**Trocar os nomes do elenco** — o JS não tem nenhum nome escrito nele. Edite só
o HTML, em três lugares que precisam ficar na mesma ordem:

1. os blocos `.cast__slide` (nome + papel);
2. os `<li class="cast__item">` da lista lateral;
3. o `alt` de cada `<img>` em `.cast__media`.

Para mudar a **quantidade** de atores, acrescente/remova um `.cast__slide`, um
`.cast__item` e um `.cast__shot` (com a foto em `assets/elenco/`) — o
`STOPS` se ajusta sozinho ([main.js:499](main.js#L499)).

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

## Acessibilidade

- `SplitText` roda com `aria: "auto"`, então leitores de tela continuam lendo os
  parágrafos originais, não as letras soltas.
- O seek do player é um `role="slider"` navegável por teclado, com
  `aria-valuenow` atualizado.
- Arte decorativa marcada com `aria-hidden` e `alt=""`.
- O nome ativo do elenco recebe `aria-current="true"`.
