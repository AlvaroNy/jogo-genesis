# 🌌 Gênesis — Documentação do Projeto

Jogo de cartas de "criação" onde você joga **3 cartas em ordem** e a **sequência** decide qual dos **8 finais** você desbloqueia. São **8 mundos** (temas diferentes) + um **Nível Secreto**. Coleção de emblemas, feito para compartilhar (roda no PC e celular).

- **Jogar:** https://alvarony.github.io/jogo-genesis/
- **Repo:** https://github.com/AlvaroNy/jogo-genesis
- **Stack:** 1 arquivo único `index.html`, self-contained (HTML + CSS + JS puro, **sem dependências, sem build**). Som via Web Audio (sintetizado), partículas via `<canvas>`. Google Analytics (gtag `G-DPY08X7CVD`).

---

## Como rodar / publicar

```bash
# Rodar local (preview)
python -m http.server 8512 --directory C:\Claude\jogo-elementa
# abrir http://localhost:8512

# Publicar: é só commitar e dar push na branch main.
# O GitHub Pages reconstrói sozinho em ~1 min.
git add index.html
git -c user.name="AlvaroNy" -c user.email="alvaronayder1993@gmail.com" commit -m "..."
git push origin main
```

> O repo não tem git config global — use `-c user.name/user.email` no commit (ou configure).
> Google Analytics só funciona no site publicado (não no localhost).

---

## Arquitetura (dentro do `index.html`)

### Telas (screens)
Geridas por `showScreen(id)` (mostra uma `.screen`, esconde o resto). Ao abrir `s-menu` também chama `refreshSecretBtn()` e `renderMenuOrbit()`.
- `#s-menu` — menu inicial (título GÊNESIS, globo/orbe animado, botões).
- `#s-worlds` — seleção dos 8 mundos (grid 2×4 via array `WORLDS`).
- `#s-game` — a tela de jogo (data-driven, serve qualquer mundo).
- Overlays: `#endOverlay` (resultado), `#galleryOverlay` (emblemas), `#hintOverlay` (dica ao toque), `#introOverlay` (onboarding), `#eggOverlay` (easter egg).

### Motor data-driven
Cada mundo é um objeto no registro **`GAMES = {0:SECRET, 1:WORLD1, ... 8:WORLD8}`**. Shape de um mundo:
```js
{
  id, name, lvl, cards, plays:3, initialEmoji, storyStart,
  stats: [[icon, stateKey], ...],   // chips de status exibidos
  newState(), apply(prev, el), classify(s),
  emoji(s, done), bg(s), react(prev, el),
  fates: { fateId: {badge, title, kind:'win'|'lose', color, hint, text} },
  fateOrder: [...8 ids...],
  // secret only: secret:true, noRepeat:true
}
```
- `apply(prev, el)` retorna o novo estado (a **ordem importa** porque o efeito depende do estado atual / da carta anterior).
- `classify(state)` mapeia o estado final para 1 dos 8 `fateId`.
- `G` = mundo atual. `enterWorld(id)` seta G, monta a mão (`renderHand`), reseta.
- Funções centrais: `renderHand`, `renderStats`, `renderSlots`, `playCard`, `finish`, `openGallery`, `openHint`, `reset`.

### Progresso (localStorage)
- `genesis_progress` = `{ "1": { fateId: {seq:[...]}, ... }, ..., "0": {...secreto} }`
- `genesis_muted` = "1"/"0" (som). `genesis_intro` = "1" (já viu onboarding).
- `medalCount(id)` = nº de finais de um mundo. `totalMedals()` = soma dos mundos 1..8 (máx **64**; o secreto NÃO conta). `completeWorlds()` = quantos mundos estão 8/8.
- (Migra o antigo `genesis_unlocked` → mundo 1.)

### Som e partículas
- `Audio` (Web Audio): `element(fx)`, `win/lose/boom/unlock/click`. `fx` de cada carta ∈ `up|down|heavy|spread`.
- `FX` (canvas `#fx`): `spawn(fx,big,cor1,cor2)` por jogada, `burst(cor,n)` no final. Ao entrar num mundo chama `FX.resize()` (o canvas estava escondido).

### Verificação obrigatória ao criar/alterar mundo
Enumerar as combinações e conferir que os **8 finais são alcançáveis**:
```js
// no console, com o jogo aberto:
const W = window.__genesis.GAMES[N];
// itere todas as sequências de 3 cartas (com repetição p/ mundos normais;
// 3 distintas p/ o secreto) e conte W.classify(W.apply...) por fateId.
```
`window.__genesis = {GAMES, showScreen}`.

---

## Os 8 mundos

Todos: 4 cartas, joga 3 em ordem, 8 finais. A coluna "Segredo da ordem" é o que faz a sequência importar.

### Mundo 1 — 🌍 Os Elementos
Cartas: 🔥 Fogo · 💧 Água · 🌍 Terra · 💨 Ar. Estado `{heat,water,land,life,chaos}`.
Finais: `eden`🌳 `inferno`🌋 `diluvio`🌊 `gelo`❄️ `deserto`🏜️ `pedra`🗿 `cataclismo`💥(perde) `vazio`🕳️(perde).
Segredo: fogo+água=vapor/caos; terra molhada→vida; água pura demais→congela; excesso de colisão→cataclismo.

### Mundo 2 — 🧪 Química da Vida
Cartas (cores CPK): 🔴 Oxigênio · ⚫ Carbono · ⚪ Hidrogênio · 🔵 Nitrogênio. Estado `{c,h,o,n,energia}`.
Finais: `vida`🧬 `agua`💧 `acucar`🍬 `gascarbonico`🫧 `metano`🔥 `amonia`🧴 `explosivo`🧨(perde) `inerte`💨(perde).
Segredo: oxigênio sobre combustível gera energia; nitrogênio em meio energético = instável. **C→N→O = Vida, C→O→N = Explosão.**

### Mundo 3 — ❤️ O Coração
Cartas: ❤️ Amor · 😨 Medo · 😡 Ódio · 🔍 Curiosidade. Estado `{amor,medo,odio,sabedoria,obsessao,teveAmor}` (amor vai negativo).
Finais: `amorpleno`💖 `iluminacao`🌟 `furia`🔥(perde) `frieza`🖤(perde) `medo`😱(perde) `obsessao`🌀(perde) `partido`💔(perde) `indiferenca`😶(perde).
Segredo: amor acalma o medo; ódio destrói o amor; curiosidade+amor=sabedoria, curiosidade+escuridão=obsessão; `teveAmor` + amor destruído por ódio = Coração Partido.

### Mundo 4 — 🎨 As Cores
Cartas: 🔴 Vermelho · 🔵 Azul · 🟡 Amarelo · ⚪ Prata. Estado `{r,b,y,prata,brilho,mud}`.
Finais: `arcoiris`🌈 `purpura`🟣 `laranja`🟠 `verde`🟢 `metalico`✨ `prateado`⚪ `marrom`🟤(perde) `monocromatico`🎨(perde).
Segredo: **prata POR CIMA de 2 cores = brilho → Arco-Íris; prata embaixo fica escondida.** 3 primárias = lama (marrom). Ex: 🔴🔵⚪=Arco-Íris, ⚪🔴🔵=Púrpura.

### Mundo 5 — 🎼 A Composição
Cartas: 🥁 Ritmo · 🎵 Melodia · 🎹 Harmonia · 🎺 Timbre. Estado `{ritmo,melodia,harmonia,timbre,dissonancia}`.
Finais: `sinfonia`🎼 `cancao`🎵 `jazz`🎷 `balada`🎹 `batuque`🥁 `solo`🎤 `cacofonia`🔊(perde) `textura`🎺(perde).
Segredo: melodia/harmonia sem base de ritmo geram dissonância → cacofonia.

### Mundo 6 — 🎧 A Música
Cartas: 🎸 Grave · 🎻 Agudo · 🥁 Ritmo · 🤫 Silêncio. Estado `{grave,agudo,ritmo,silencio,dinamica}`.
Finais: `obra`🎶 `espectro`🎵 `groove`🎸 `grave`🔉 `agudo`🔊 `batida`🥁 `estrondo`💥(perde) `silencio`🤫(perde).
Segredo: **silêncio DEPOIS de som = dinâmica → Obra-Prima.** 3 sons sem silêncio = estrondo (muralha de ruído).

### Mundo 7 — 🍬 Os Sabores
Cartas: 🍬 Doce · 🧂 Salgado · 🍋 Azedo · ☕ Amargo. Estado `{doce,salgado,azedo,amargo,enjoo}`.
Finais: `banquete`🥘 `agridoce`🍯 `amargodoce`🍫 `doce`🍬 `azedo`🍋 `amargo`☕ `salgado`🧂 `enjoo`🤢(perde).
Segredo: azedo+amargo se chocam (enjôo), mas **doce no fim reduz o enjôo e salva** → banquete. Ex: 🍋☕🍬=Banquete, 🍬🍋☕=Enjôo. Banquete = 3+ sabores distintos sem enjôo.

### Mundo 8 — 🎬 Hollywood
Cartas: 😂 Humor · 🏕️ Sobrevivência · 🔍 Mistério · 🧭 Exploração. Estado `{humor,sobrev,misterio,explor,quebra}`.
Finais (6 pares + obra-prima + fracasso): `blockbuster`🎬 `suspense`😱 `comediamisterio`🕵️ `comediaterror`🧟 `aventura`🗺️ `ficcao`🚀 `epico`🏔️ `bomba`📉(perde).
Segredo: os 6 PARES de ingredientes = 6 gêneros; 3 ingredientes = Blockbuster; **1 ingrediente raso OU humor cedo demais (antes de 2 tensões) = Bomba.** Ex: 🏕️🔍😂=Blockbuster, 😂🏕️🔍=Bomba.

### Nível Secreto — 🌌 A Criação Final (`GAMES[0]`)
- **8 cartas = os 8 mundos.** Cada carta libera só quando aquele mundo está **8/8** (senão 🔒). Usa **3, sem repetir** (`noRepeat`).
- Destrava no menu quando `completeWorlds() >= 3` (botão mostra "🔒 Nível Secreto (X/3)" até lá).
- Cada mundo tem uma **Força**: 🌍 Matéria, 💗 Alma, 🎨 Arte, 🧠 Mente (ver `SEC_DATA`, `pts`/`set`). A **1ª carta é a "semente"** (peso dobrado → define o núcleo). `harmonia` = pares adjacentes que compartilham força.
- 8 universos: `supremo`🌌 `materia`🪐 `alma`💗 `arte`🎨 `mente`🧠 `dualidade`☯️ `equilibrio`🌈 `colapso`🕳️(perde). Todos alcançáveis nas 336 combinações (3 distintas de 8).
- Progresso em `progress[0]`, NÃO conta no /64.

---

## Menu e UX

- **Botões do menu:** ▶ Jogar → 🌌 Nível Secreto → Compartilhar no WhatsApp. (Emblemas e Som foram removidos do menu; Som fica no in-game, Emblemas fica dentro de cada mundo.)
- **Compartilhar no WhatsApp:** abre `wa.me` com mensagem pronta + link público (usuário escolhe o contato e envia).
- **Globo orbitado:** em vez de um 🔥 fixo, cada mundo completo 8/8 vira um **planeta** (emoji do mundo) orbitando o universo — 0 completos = 🔥, 8/8 = sistema solar. `renderMenuOrbit()` (contra-rotação mantém emojis "de pé").
- **Easter egg:** 5 cliques seguidos no globo (`#menuOrb`, reset após 1.6s) abrem a **Sala Secreta** (`#eggOverlay`, créditos).
- **Onboarding:** `#introOverlay` "Como jogar" (3 passos: 3 cartas, ordem importa, 8 finais) na 1ª vez que entra num mundo; botão ❓ (na seleção de mundos) reabre.
- **Progresso & comemoração:** contador **X/64** na seleção; mundos completos com borda dourada + "✓ 8/8"; banner "🏆 MUNDO COMPLETO" (8/8), "🎉 GÊNESIS 100%" (64/64) e "🌌 NÍVEL SECRETO COMPLETO" + confete.
- **Galeria com dica ao toque:** emblemas sem dica visível; **tocar** abre caixa com a Dica; a **Solução** (sequência) só aparece se já descoberto (senão "🔒 Descubra jogando").
- **Polimento:** partículas por elemento + burst; sons por carta + stingers; esfera do mundo com brilho/anel; animações de slot/stat/história/ring; botão de mudo.

---

## Receita: adicionar um novo mundo

1. Criar `WORLDn` (mesmo shape acima): definir `cards`, `newState/apply/classify/emoji/bg/react`, `fates` (8) e `fateOrder`. **Fazer a ordem importar** (efeito depende do estado/carta anterior).
2. Adicionar em `GAMES` e marcar `playable:true` no array `WORLDS` (com emoji + tag).
3. **Enumerar as 64 combinações** (`window.__genesis.GAMES[n]`) e ajustar os limiares do `classify` até os 8 finais serem alcançáveis (cuidado com balde "catch-all" pegando casos errados).
4. Playtest na UI, limpar `localStorage` de teste, `commit` + `push` (Pages rebuilda ~1min).

---

## Ideias futuras (não feitas)
Eventos personalizados no Analytics (mundo completo / universo criado / easter egg / compartilhou); níveis 2+ por mundo; arte própria nos emblemas; tela de 100%; stats visuais (barrinhas); vibração no celular; ranking.
