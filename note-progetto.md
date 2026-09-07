# Progetto Astro — Blog minimalista

## Obiettivo

Creare un blog personale con Astro, semplice, leggibile e facile da modificare.

Caratteristiche concordate:

- tema esclusivamente dark;
- design minimalista;
- layout responsive per desktop e smartphone;
- articoli scritti in Markdown;
- organizzazione degli articoli tramite tag;
- pagina `/tags/` con tutti i tag e il numero di articoli associati;
- pagina `/tags/[tag]` con gli articoli di un determinato tag;
- footer con link social;
- font gestiti tramite Fontsource;
- favicon con una zampa di cane;
- metadata Open Graph da aggiungere.

## Stack

- Astro 7.3.1
- TypeScript 6.x
- pnpm
- Markdown
- Fontsource
- CSS senza framework UI

Il progetto è stato creato con:

```bash
mkdir my-blog
cd my-blog
pnpm create astro@latest .
```

Scelte effettuate:

- template Empty;
- installazione delle dipendenze;
- TypeScript;
- modalità Strict.

È stato inoltre installato MDX:

```bash
pnpm astro add mdx
```

MDX non è necessario per gli articoli `.md`, ma può rimanere installato.

## Struttura del progetto

```text
src/
├── content.config.ts
├── content/
│   └── posts/
│       └── primo-articolo.md
├── layouts/
│   └── BlogLayout.astro
├── pages/
│   ├── index.astro
│   ├── posts/
│   │   └── [slug].astro
│   └── tags/
│       ├── index.astro
│       └── [tag].astro
└── styles/
    └── global.css

public/
└── favicon.svg
```

## Content Layer

File `src/content.config.ts`:

```ts
import { defineCollection } from 'astro:content';
import { glob } from 'astro/loaders';
import { z } from 'astro/zod';

const posts = defineCollection({
  loader: glob({
    pattern: '**/*.md',
    base: './src/content/posts',
  }),
  schema: z.object({
    title: z.string(),
    description: z.string(),
    pubDate: z.coerce.date(),
    tags: z.array(z.string()).default([]),
  }),
});

export const collections = {
  posts,
};
```

## Esempio di articolo

File `src/content/posts/primo-articolo.md`:

```markdown
---
title: "Il mio primo articolo"
description: "Un piccolo esperimento con Astro."
pubDate: 2026-09-05
tags:
  - astro
  - web
---

Questo è il mio primo articolo.

Sto costruendo questo blog con Astro, cercando di mantenerlo il più semplice possibile.
```

Il titolo viene visualizzato dalla pagina Astro, quindi non è necessario aggiungere un ulteriore titolo H1 nel corpo Markdown.

## Rendering degli articoli

Con Astro 7 si usa:

```ts
import { getCollection, render } from 'astro:content';
```

e:

```ts
const { Content } = await render(post);
```

Non si usa il vecchio metodo:

```ts
post.render()
```

## Homepage

La homepage:

- recupera gli articoli con `getCollection('posts')`;
- li ordina dal più recente al più vecchio;
- mostra data, titolo, descrizione e tag;
- collega ogni articolo a `/posts/[slug]`.

## Pagina del singolo articolo

La pagina dinamica è:

```text
src/pages/posts/[slug].astro
```

Utilizza `getStaticPaths()` per generare una pagina per ogni articolo.

Mostra:

- data di pubblicazione;
- titolo;
- descrizione;
- contenuto Markdown;
- tag;
- collegamento per tornare alla homepage.

## Pagina di tutti i tag

La pagina è:

```text
src/pages/tags/index.astro
```

I tag vengono raccolti da tutti gli articoli e contati con una `Map`.

Ordinamento:

1. numero di articoli decrescente;
2. ordine alfabetico in caso di parità.

La visualizzazione usa un contenitore flex con `flex-wrap`, così i tag vanno automaticamente a capo:

```text
#astro (3)   #web (2)   #linux (1)   #scrittura (1)
```

## Pagina del singolo tag

La pagina dinamica è:

```text
src/pages/tags/[tag].astro
```

Mostra:

- il nome del tag;
- il numero di articoli;
- gli articoli associati;
- i relativi tag;
- il collegamento alla pagina di tutti i tag.

## Layout principale

File:

```text
src/layouts/BlogLayout.astro
```

Il layout contiene:

- `<head>`;
- titolo della pagina;
- descrizione;
- header con il nome del blog;
- collegamento alla pagina Tags;
- contenuto tramite `<slot />`;
- footer.

Il riferimento alla favicon sarà:

```astro
<link rel="icon" type="image/svg+xml" href="/favicon.svg" />
```

## Footer

Il footer contiene:

- copyright;
- link a Mastodon;
- link a Pixelfed.

Link:

- Mastodon: `https://infosec.exchange/@linuzzz`
- Pixelfed: `https://pixelfed.social/paolinus`

## Tema grafico

Il sito usa esclusivamente il tema dark.

Variabili CSS principali:

```css
:root {
  color-scheme: dark;

  --background: #0d0d0d;
  --surface: #151515;
  --text: #e8e8e8;
  --text-muted: #888;
  --accent: #d4a373;
  --border: #292929;

  --font-body: "Inter Variable", sans-serif;
  --font-heading: "Inter Variable", sans-serif;
}
```

Non è prevista una modalità light.

## Font

È stato installato Fontsource: --> ricordare di installare i fonts!!!

```bash
pnpm add @fontsource-variable/inter
```

Nel CSS:

```css
@import "@fontsource-variable/inter";
```

L'import deve trovarsi all'inizio del file CSS.

I font sono configurati tramite variabili separate:

```css
--font-body: "Inter Variable", sans-serif;
--font-heading: "Inter Variable", sans-serif;
```

In futuro sarà possibile scegliere un font diverso per il testo e uno per i titoli.

## Responsive design

Il CSS è mobile-first.

Contenitore principale:

```css
width: min(100% - 2rem, 800px);
```

Su schermi più grandi:

```css
width: min(100% - 4rem, 800px);
```

Breakpoint principale:

```css
@media (min-width: 700px)
```

Per i titoli viene usato `clamp()`:

```css
font-size: clamp(2rem, 6vw, 3.2rem);
```

La sintassi è:

```text
clamp(minimo, valore preferito, massimo)
```

`vw` significa viewport width, cioè percentuale della larghezza della finestra.

## Correzione dei margini del titolo

Era presente questa regola:

```css
header,
main {
  width: min(100% - 2rem, 800px);
  margin-inline: auto;
}
```

Il problema era che `header` selezionava anche l'header interno dell'articolo, `.post-header`, centrandolo nuovamente e causando un disallineamento del margine sinistro.

La correzione è:

```css
.site-header,
main {
  width: min(100% - 2rem, 800px);
  margin-inline: auto;
}
```

Anche nella media query:

```css
@media (min-width: 700px) {
  .site-header,
  main {
    width: min(100% - 4rem, 800px);
  }

  .site-header {
    padding-block: 3rem;
  }

  main {
    padding-bottom: 7rem;
  }
}
```

In questo modo `.post-header` rimane dentro `main` e il suo margine sinistro coincide con quello del contenuto.

## Spaziatura degli articoli

Le regole principali sono:

```css
.post-preview {
  padding-block: 1.25rem;
  border-top: 1px solid var(--border);
}
```

Su desktop:

```css
@media (min-width: 700px) {
  .post-preview {
    padding-block: 1.5rem;
  }
}
```

Quando si modifica lo spazio tra i titoli, bisogna considerare anche la regola presente nella media query, perché può sovrascrivere quella mobile.

## Favicon

È stata scelta una grafica a forma di zampa di cane.

Il file dovrà essere copiato in:

```text
public/favicon.svg
```

Nel layout va aggiunto:

```astro
<link rel="icon" type="image/svg+xml" href="/favicon.svg" />
```

Astro serve automaticamente i file presenti nella cartella `public`.

Dopo aver aggiunto la favicon, se non compare subito nel browser, è possibile eseguire un hard refresh:

```text
Ctrl + Shift + R
```

## Stato del progetto

Completato:

- [x] Astro;
- [x] TypeScript;
- [x] Markdown;
- [x] Content Layer;
- [x] homepage;
- [x] pagina del singolo articolo;
- [x] tag negli articoli;
- [x] pagina con tutti i tag;
- [x] conteggio degli articoli per tag;
- [x] ordinamento dei tag;
- [x] pagina del singolo tag;
- [x] tema dark-only;
- [x] responsive design;
- [x] Fontsource;
- [x] footer;
- [x] link Mastodon;
- [x] link Pixelfed;
- [x] correzione dei margini del titolo e del contenuto.

Da completare:

- [ ] aggiungere la favicon;
- [ ] implementare Open Graph;
- [ ] scegliere eventualmente un'immagine Open Graph;
- [ ] valutare due font diversi;
- [ ] valutare i callout Obsidian;
- [ ] eventuali rifiniture grafiche.

## Filosofia del progetto

Il principio guida è:

> Semplice, leggibile, minimale.

Evitare di aggiungere framework o librerie quando Astro e CSS sono sufficienti.

## Note aggiuntive
per cloudflare digitando
`pnpm astro add cloudflare`
fa tutto da solo...crea i files, etc etc...bisogna poi solo aggiungerli a git, fare commit e push

Per il deploy su cloudflare occorre provare in locale che sia tutto ok:
`npx astro build && npx wrangler dev`

se tutto funziona appare:
```
[wrangler:info] Ready on http://localhost:8787
```

Occorre anche ricordare che c'è un bug e che per fare funzionare la build su cloudflare occorre rimuovere il file pnpm-workspace.yaml
che è inutile se non si usano i workspace di pnpm

su .gitignore ho aggiunto alcune cose:
```
# VS Code files
.vscode/

# Wrangler
.wrangler/

```

Siccome però prima di aggiungere wrangler a gitignore avevo già fatto una build e poi `git add .` ho dovuto poi rimuovere quei files da git:
```
git rm -r .wrangler/
git commit -m "provo a rimuovere i file di build di wrangler"
git push origin main
```

Infine per inizializzare il git locale:
```
git config --global init.defaultBranch main
git init
git add .
git commit -m "First commit"
git remote add origin https://github.com/linuzzz/linuzzz-blog.git
git branch -a
git push origin main
```


