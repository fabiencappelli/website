# Publication du site

Ce depot contient un site Jekyll publie avec GitHub Pages.

Le fichier `README.md` est exclu du build dans `_config.yml`, donc il reste une documentation de depot et n'est pas converti en page web.

## Structure utile

- `fr/` et `en/`: pages principales du site, par langue.
- `_posts/`: articles de blog.
- `_projects_fr/` et `_projects_en/`: fiches projets.
- `_layouts/`: gabarits HTML Jekyll.
- `_includes/`: fragments reutilisables.
- `assets/`: images, PDF et CSS.
- `.github/workflows/pages.yml`: workflow GitHub Actions qui construit et publie le site.

## Tester en local

Installer les dependances Ruby si necessaire:

```bash
bundle install
```

Lancer le site en local:

```bash
bundle exec jekyll serve
```

Le site est alors disponible sur l'adresse affichee par Jekyll, generalement:

```text
http://127.0.0.1:4000/
```

Pour faire uniquement un build local:

```bash
bundle exec jekyll build
```

Le resultat est genere dans `_site/`. Ce dossier est ignore par Git et ne doit pas etre modifie a la main.

## Publier une page

Les pages principales sont dans `fr/` et `en/`.

Exemple minimal:

```markdown
---
title: Titre de la page
layout: default
permalink: /fr/ma-page/
---

Contenu de la page.
```

Points importants:

- `layout: default` applique la mise en page du site.
- `permalink` fixe l'URL publique.
- Les pages dans `fr/` recoivent automatiquement `lang: fr`.
- Les pages dans `en/` recoivent automatiquement `lang: en`.

## Publier un article de blog

Ajouter un fichier Markdown dans `_posts/`.

Le nom du fichier doit suivre le format:

```text
YYYY-MM-DD-slug-lang.md
```

Exemple:

```text
2026-06-01-nouvel-article-fr.md
```

Front matter recommande:

```markdown
---
title: "Titre de l'article"
date: 2026-06-01
lang: fr
project: robie
image: /assets/images/blog/image.jpg
categories: [robie]
summary: "Resume court affiche dans la liste du blog."
---

Contenu de l'article.
```

Points importants:

- `lang` doit etre `fr` ou `en`.
- `summary` alimente les cartes du blog.
- `categories` permet le filtrage sur la page blog.
- `image` sert aussi au partage social.
- Pour publier l'article dans les deux langues, creer deux fichiers separes avec `lang: fr` et `lang: en`.

## Publier un projet

Ajouter une fiche dans la collection correspondant a la langue:

- `_projects_fr/mon-projet.md`
- `_projects_en/my-project.md`

Exemple:

```markdown
---
title: Mon projet
slug: mon-projet
summary: Description courte affichee dans la grille des projets.
stack: [Python, Jekyll]
order: 3
featured: true
github_repos:
  - url: https://github.com/fabiencappelli/mon-projet
    label: GitHub
---

Description detaillee du projet.
```

Points importants:

- Les projets francais sortent sous `/fr/projects/:name/`.
- Les projets anglais sortent sous `/en/projects/:name/`.
- `order` controle l'ordre d'affichage dans la grille.
- `stack` alimente les etiquettes techniques.

## Ajouter des images ou PDF

Placer les fichiers dans `assets/`.

Exemples:

- Images de blog: `assets/images/blog/`
- Images generales: `assets/images/`
- PDF: `assets/pdf/`

Dans un fichier Markdown, utiliser un chemin absolu depuis la racine du site:

```markdown
![Texte alternatif](/assets/images/blog/image.jpg)
```

ou:

```markdown
[Telecharger le PDF](/assets/pdf/fichier.pdf)
```

## Mettre en ligne

La publication se fait automatiquement via GitHub Actions.

Flux habituel:

```bash
git status
git add .
git commit -m "Ajoute un article"
git push origin main
```

Apres le push sur `main`, le workflow `.github/workflows/pages.yml`:

1. installe Ruby et les dependances Bundler;
2. lance `bundle exec jekyll build --baseurl ""`;
3. envoie le dossier `_site/` vers GitHub Pages;
4. publie le site.

Il est aussi possible de relancer manuellement le workflow depuis l'onglet Actions de GitHub avec `workflow_dispatch`.

## Verifications avant publication

Avant de pousser:

```bash
bundle exec jekyll build
git status
```

Verifier aussi:

- les liens internes;
- les chemins d'images;
- la presence de `lang` sur les articles;
- les dates des articles;
- les fichiers volumineux ajoutes dans `assets/`.
