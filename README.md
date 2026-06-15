# NetCorp S.L. — Documentació ASIX 2n (Ciberseguretat)

Lloc de documentació amb [MkDocs](https://www.mkdocs.org/) per als projectes
ABP del curs ASIX 2n, itinerari Ciberseguretat.

## Estructura

```
docs/
├── index.md                  # Pàgina d'inici
├── projectes/
│   ├── index.md              # Visió general dels 6 projectes
│   ├── p1.html ... p6.html    # Webs interactives de cada projecte (HTML autònom)
└── teoria/
    └── m08-cap1.md           # Fonaments teòrics M08 (Projecte 1)
```

Les webs `p1.html`–`p6.html` són fitxers HTML autònoms (amb el seu propi CSS/JS),
i MkDocs els serveix tal qual, enllaçats des del menú lateral.

## Veure-ho en local

```bash
pip install -r requirements.txt
mkdocs serve
```

Obre `http://127.0.0.1:8000`.

## Publicar a GitHub Pages

### Opció A — Automàtica (recomanada)

1. Crea un repositori a GitHub (per exemple `netcorp-docs`) i puja tot aquest
   contingut a la branca `main`.
2. A la configuració del repositori → **Settings → Pages**, selecciona com a
   font la branca **`gh-pages`** (es crearà sola la primera vegada que
   s'executi el workflow).
3. El workflow `.github/workflows/deploy.yml` ja inclòs construirà i publicarà
   el lloc automàticament cada vegada que facis `push` a `main`.

### Opció B — Manual des del teu ordinador

```bash
pip install -r requirements.txt
mkdocs gh-deploy
```

Això construeix el lloc i el puja directament a la branca `gh-pages` del repo.

## Afegir més contingut

- Nous documents de teoria: afegeix un `.md` a `docs/teoria/` i referencia'l a
  `mkdocs.yml` dins de `nav:`.
- Noves webs de projecte: copia l'HTML a `docs/projectes/` i afegeix l'entrada
  corresponent a `nav:`.
