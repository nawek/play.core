# Revue automatique du dépôt

> Généré depuis le homelab le 2026-10-19 par un modèle gratuit OpenCode. Peut se tromper : vérifie avant d'agir.

## Points d'attention

### Critiques

- **Exécution de code distant arbitraire** — `src/programs/demos/hotlink.js:11` : le programme fait `fetch(...raw.githubusercontent.com...)` puis `new Function("return " + e)()`. Le contenu téléchargé est évalué tel quel au démarrage : si le dépôt distant (ou le réseau) est compromis, c'est une exécution de code arbitraire (RCE / risque de chaîne d'approvisionnement). Aucun épinglage de version ni d'intégrité (SRI). Le commentaire « Don't do this :) » le reconnaît mais ne le corrige pas.

### Importants

- **Export par défaut incohérent** — `src/modules/num.js:7` : l'objet `export default` liste `map, fract, clamp, sign, mix, smoothstep, smootherstep` mais omet `step` et `mod`, qui sont pourtant exportés nommément. Tout `import Num from ...; Num.mod(...)` / `Num.step(...)` renverra `undefined` → `TypeError`. `src/modules/sdf.js` importe en nommé (donc épargné), mais l'API par défaut est trompeuse.
- **`restore()` sans `try/catch`** — `src/core/storage.js:15` : `JSON.parse(localStorage.getItem(key))` lève une exception si la valeur stockée est corrompue (contrairement à `store()` qui, lui, est protégé). Un `localStorage` altéré casse le démarrage.
- **Erreurs silencieusement avalées** — `src/modules/load.js` : `image()` fait `resolve(null)` en cas d'échec, `text()` renvoie `''`, `json()` renvoie `{}` au lieu de `reject`. Conséquence : le `.catch()` montré dans la doc du même fichier ne se déclenche jamais, et le `.catch()` de `src/modules/image.js` est du code mort (les échecs ne remontent pas). La gestion d'erreur est incohérente entre la doc et le code.
- **Type MIME erroné** — `src/modules/filedownload.js:16` : `'jpg' : 'text/jpeg'` devrait être `'image/jpeg'`. Le fichier téléchargé aura un type incorrect.

### Mineurs

- **Couleur CSS invalide** — `src/programs/contributed/emoji_wave.js:11` : `backgroundColor : 'rgb(100, 0, 300)'` ; la composante bleue dépasse 255 (clampée par le navigateur, comportement non déterministe selon le moteur).
- **`mod` mal nommé** — `src/modules/num.js` : `return a % b` est commenté « GLSL modulo » mais diffère du `mod` GLSL pour les valeurs négatives. `map()` divise par zéro si `inB === inA`.
- **Modulo par zéro possible** — `src/programs/basics/name_game.js:23` : `% (f % context.cols)` ; si `f` est un multiple de `cols` (ex. `cols = 10`, `f = 10`), `f % cols == 0` → `NaN` → index `undefined`.
- **Code mort / API dépréciée** — `src/modules/camera.js` : `return` après un `throw` (inatteignable) ; `video.src = window.URL.createObjectURL(stream)` est déprécié, et dans ce chemin ancien le callback reçoit un `video.srcObject` potentiellement `undefined`.
- **`char` non‑chaîne** — `src/programs/demos/performance_test.js` : `char : context.frame % 10` renvoie un nombre alors que les autres programmes renvoient une chaîne (`String` implicite ; rendu non vérifié).
- **Imports inutilisés** — `src/programs/demos/chromaspiral.js` : `sort` et `addN` importés mais non utilisés.
- **Scripts shell fragiles** — `src/programs/addheader.sh` et `src/modules/list.sh` : variables non quotées (`$1`, `$TARGET_PATH/$folder`), parsing via backticks/`find`, construction d'URL par `sed` fragile (casse si espaces ou noms inattendus).
- **Documentation hétérogène** : `src/core/storage.js` n'a pas d'en-tête `@module/@desc/@category` contrairement aux autres modules fournis.

## Améliorations suggérées

- **Tests automatisés** : les seuls tests fournis (`tests/*.html`, `tests/browser_bugs/*.html`) sont manuels et dépendent d'un navigateur ; aucun framework (Vitest/Jest/Playwright) ni `package.json` visible. Ajouter des tests unitaires pour les modules purs (`num.js`, `sdf.js`, `vec2/vec3`, `sort.js`, `storage.js`).
- **CI** : aucun workflow visible ; mettre une CI qui exécute lint + tests et vérifie au minimum les erreurs de syntaxe sur `src/**/*.js` (des fichiers comme `tests/promise_chain.html` contiennent volontairement des erreurs, à exclure explicitement).
- **Lint/format** : ajouter ESLint/Prettier (les imports en fin de fichier, ex. `import { drawInfo }` après `main`/`post`, déclencheraient `import/first`).
- **Sécurité** : supprimer ou isoler `hotlink.js` derrière une case documentée ; à défaut, épingler un commit et un hash, jamais du code évalué depuis `master`.
- **Dépendances** : aucune dépendance externe runtime visible (bon point) ; documenter ce choix et figer la version Node pour les scripts.
- **Structure/documentation** : `README.md`, `LICENSE`, `src/makefile`, `src/modules/{sort,color,canvas,vec2,vec3,buffer,drawbox,exportframe}.js`, `src/run.js`, `src/core/{canvasrenderer,textrenderer}.js` **non fournis → non vérifiés** ; uniformiser les en-têtes de module (corriger `storage.js`, retirer les `import` en fin de fichier) et documenter le contrat exact de `main()` (chaîne vs nombre vs objet `{char,...}`).

## Ce qui est bien

- **Séparation claire** en trois couches (`core/`, `modules/`, `programs/`) avec de nombreux programmes pédagogiques courts et commentés, ce qui rend le projet lisible et testable manuellement.
- **Gestion d'erreurs présente dans les points critiques d'I/O** : `load.js`, `camera.js` et `image.js` utilisent des `try/catch`, `.catch()` et des messages explicites (à l'exception notable des erreurs avalées, cf. ci‑dessus).
- **Tests navigateur dédiés** (`tests/browser_bugs/*.html`) documentant des bugs WebKit précis (numéros de bug cités) — démarche utile pour la compatibilité.

*Note : les fichiers non listés ci‑dessus (README, LICENSE, makefile, sort.js, color.js, canvas.js, vec2.js, vec3.js, buffer.js, drawbox.js, exportframe.js, run.js, renderers core, `*.old.txt`) ne m'ont pas été fournis ; tout élément les concernant est « non vérifié ».*
