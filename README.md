# scalingo-buildpack-bun

Buildpack Scalingo qui installe **bun** au build, en remplacement du `nodejs-buildpack`,
pour une app dont toute la chaîne JS est bun (package manager + bundler). **bun-only**
(pas de node). Inspiré du buildpack bun de
[`proconnect-gouv/hyyypertool`](https://github.com/proconnect-gouv/hyyypertool/tree/scalingo/buildpack).

Utilisé par `yespark/yespark-rails`.

## Ce qu'il fait (`bin/compile`)

1. Installe bun (version lue dans le `.bun-version` de l'app, cache réutilisé si identique).
2. `bun install --frozen-lockfile`.
3. Écrit un fichier `export` → met bun sur le `PATH` des **buildpacks suivants**, pour que le
   `ruby-buildpack` trouve `bun` quand `assets:precompile` lance `bun run build` (jsbundling-rails).
   > ⚠️ Le `.profile.d` seul ne suffit pas : il n'est sourcé qu'au runtime, pas entre les
   > buildpacks au build. Le fichier `export` est la pièce essentielle.
4. Écrit `.profile.d/bun.sh` → `PATH` bun au runtime.

## Usage

Dans le `.buildpacks` de l'app, **avant** le `ruby-buildpack` (où tourne `assets:precompile`) :

```
https://github.com/yespark/scalingo-buildpack-bun.git
https://github.com/Scalingo/ruby-buildpack.git
```

Ce repo doit rester **public** : le builder Scalingo clone l'URL d'un buildpack sans les
credentials git de l'app.
