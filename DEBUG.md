# Documentation du débogage — Checkpoint React Debugging

## Méthode

1. Installation de l'application démo (`npm ci`, `npm start`).
2. Lecture des fichiers de bugs dans `src/bugs/` : chaque fichier contient un composant, des tests intégrés (`useBugTest`) qui vérifient le comportement attendu, et une section « Fix issues to catch bug » qui indique le type de problème.
3. Diagnostic : pour chaque bug, comparaison entre le comportement attendu par les tests et le code (valeurs de props, état, dépendances des hooks, rendus).
4. Correction, un commit par bug, pour garder l'historique lisible (`git log`).
5. Vérification : build (`npm run build`) puis lecture de la page de chaque bug dans le navigateur (observations et statut « Catch »).

Remarque honnête : les corrections des 12 premiers bugs viennent des fichiers `.patch` fournis avec les modules du cours. Je les ai appliqués, mais je ne les ai pas écrits moi-même. Le bug StrictMode n'a pas de patch fourni ; je l'ai diagnostiqué et corrigé moi-même. Je n'ai pas utilisé l'extension React Developer Tools de façon interactive dans le navigateur : le diagnostic s'est fait par la lecture du code et des tests intégrés.

## Bugs

| Bug (fichier) | Symptôme | Cause | Correction |
|---|---|---|---|
| Unexpected Prop Types (`UnexpectedPropTypes.js`) | Popularité, stock et note incorrects | `popularity="trending"` (chaîne) comparé à `1` ; `rating={0}` rend un `0` à l'écran ; `inventoryCount={null}` traité comme « Only null left » | Props corrigées (`POPULARITY.trending`, `rating={3.5}`, `inventoryCount={10}`) et `{rating ? ... : null}` |
| Changing Props (`ChangingProps.js`) | Le « like » ne s'enregistre pas ; les stats changent de façon incorrecte | La valeur `likeValue` est une variable locale réinitialisée à chaque rendu ; `attributes` est muté directement | État local avec `useState` ; calcul des stats dans un `useEffect` sur une copie |
| Passing Props (`PassingProps.js`) | Le résumé d'achat ne reflète pas le choix ; « Buglearner Anonymous » reste dans la liste | Props passées dans le mauvais ordre (`{...props}` écrase les valeurs), noms de props incohérents (`level` / `purchaseLevel`) | Ordre des props corrigé, noms alignés, `{...props}` retiré du `Heading` |
| Unexpected State (`UnexpectedState.js`) | Le niveau ne change pas, le résumé est faux | `let currentLevel = useState(...)` : on récupère le tableau, pas la valeur ; la modification n'est jamais un état | `const [currentLevel, setCurrentLevel] = useState(...)` |
| Hook Side Effects (`HookSideEffects.js`, 3 parties) | Le sélecteur de quantité n'apparaît pas ; le compte à rebours et le suivi analytique se déclenchent en boucle ; `setInterval` jamais nettoyé | Dépendance `isFetching` mal utilisée, `useCallback` sans tableau de dépendances (recréé à chaque rendu), `setInterval` sans `clearInterval` | `isFetched` à la place, `useCallback(..., [])`, nettoyage du timer dans le `return` de `useEffect` |
| Strict Mode (`StrictMode.js`) | Le compteur de montages affiche 2 au lieu de 1 | En mode strict, React exécute l'effet deux fois au montage. La garde `hasMounted` est un state lu dans la fermeture : elle vaut encore `false` à la seconde exécution | Garde par `useRef` (persistante entre les deux exécutions), effet sans dépendance d'état |
| Cascading Render (`CascadingRender.js`) | Les cœurs d'animation se rejouent à chaque rendu | Le composant enfant se re-rend à chaque mise à jour du parent : fonctions recréées, pas de mémoïsation | `useCallback` sur les gestionnaires, `memo` sur `LikeButton` |
| Conditional Render (`ConditionalRender.js`) | Impossible de passer à une quantité élevée ; interface lente | Calcul et rendu conditionnels mal placés, `VolumeDiscount` monté ou non selon la quantité | Rendu inconditionnel et garde `if (discount <= 0) return null` dans `VolumeDiscount` |
| Context Render (`ContextRender.js`) | Tous les éléments de la liste se re-rendent quand on change de magasin | `homeStore` (valeur du contexte) passé à chaque élément | Calcul `isHomeStore` dans le parent, `StoreListItem` enveloppé dans `memo` |
| Delayed Render (`DelayedRender.js`) | La saisie de quantité est très lente | `parseCurrencyAsAmount` contient une boucle inutile de `Math.pow(amount, 2.8)` itérations qui bloque le thread | Suppression de la boucle |
| Excessive Render (`ExcessiveRender.js`) | Trop de requêtes et de re-rendus à chaque changement de quantité | Une fonction intermédiaire `handleOnQuantityChange` qui appelle `refetch` à chaque rendu | Passage direct de `refetch` à `onQuantityChange` |
| Deferred Render (`DeferredRender.js`) | La recherche est saccadée à la frappe | Le filtrage se fait à chaque frappe, sans priorité | `useDeferredValue` sur la valeur de recherche |
| Long Running Render (`LongRunningRender.js`) | L'onglet ne change pas tant que les avis ne sont pas chargés | Changement d'onglet traité comme une mise à jour urgente | `useTransition` autour de `setTabIndex` |

## Vérification

- `npm run build` : compilation réussie. Il reste des avertissements ESLint sur des imports inutilisés (`isVolumeDiscount`, `useEffect` dans `PassingProps.js`). Ils n'empêchent pas le build.
- Dans le navigateur (`npm start`, `http://localhost:3000`), les pages suivantes affichent le statut « Catch » une fois les observations validées sans interaction : **Unseen Umbrella Moth**, **Shy Spider**, **Silent Stag Beetle** (StrictMode) et **Extravagant Emerald Ash Borer**.
- Les autres bugs (Crimson Caterpillar, Pilfering Pillbug, Huggable Honeybee, Curious Centipede, Confounding Cricket, Cheerful Cicada, Dapper Dragonfly, Lovable Ladybug, Defiant Damselfly) demandent une interaction (cliquer sur « like », changer la quantité, taper dans la recherche, changer d'onglet). Je n'ai pas encore vérifié ces interactions une par une : c'est la prochaine étape de la vérification.
