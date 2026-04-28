+++
title = "Zero-Knowledge : transformer un calcul en polynômes (Partie 1/3)"
date = 2026-03-04
updated = 2026-04-28
description = "D'un programme à quelques équations polynomiales : comment les SNARKs encodent un calcul par aplatissement, R1CS et la transformation QAP"

[taxonomies]
tags = ["crypto", "computer-science"]

[extra]
katex = true
social_media_card = "/img/zk-computation-to-polynomials-banner.webp"
+++

![Grue en origami pliée à partir d'une feuille de notation mathématique](/img/zk-computation-to-polynomials-banner.webp)

Les preuves à connaissance nulle reviennent sans cesse. Balaji [les décrit](https://x.com/balajis/status/2022462579713675506) comme le contrepoids de l'IA : « L'intelligence artificielle, c'est l'attaque. Le zero knowledge, c'est la défense. » [Podcasts](https://zeroknowledge.fm/), conférences, [feuilles de route crypto](https://strawmap.org/) : le ZK est partout. Je voulais comprendre ce qui se passe réellement sous le capot, alors [comme promis](@/blog/verkle-trees-polynomial-commitments.fr.md#et-ensuite), j'ai retracé les mathématiques depuis zéro. Ce qui m'a le plus enthousiasmé, ce n'était pas la cryptographie. C'était l'étape d'avant : comment prend-on un programme pour le transformer en quelque chose sur lequel la cryptographie peut opérer ?

C'est le sujet de cet article. On va prendre un petit programme, le décomposer en opérations élémentaires, encoder ces opérations sous forme de matrices, et tout condenser en une seule équation qu'un SNARK[^snark] peut vérifier. La structure et l'exemple suivent [le guide QAP de Vitalik (2016)](https://medium.com/@VitalikButerin/quadratic-arithmetic-programs-from-zero-to-hero-f6d558cea649), qui reste l'une des introductions les plus claires sur le sujet. Il s'agit de la Partie 1/3 ([la Partie 2](@/blog/zk-proofs-part2.fr.md) couvre le protocole de preuve, la Partie 3 couvrira les applications). Je supposerai une familiarité avec les [corps finis](@/blog/math-behind-private-key.fr.md#les-corps-des-nombres-avec-de-l-arithmetique) et les [engagements polynomiaux](@/blog/verkle-trees-polynomial-commitments.fr.md) de mes articles précédents.

## Prouver sans montrer

Alice affirme connaître une valeur $x$ telle que $f(x) = y$. Bob veut en être convaincu, mais il ne veut pas exécuter $f$ lui-même (peut-être que c'est coûteux), et Alice ne veut pas révéler $x$ (peut-être que c'est secret). Une **preuve à connaissance nulle** donne à chacun ce qu'il veut :

- **Confidentialité** : Alice ne révèle pas $x$.
- **Concision** : le travail de vérification de Bob est minuscule comparé à l'exécution de $f$.

Notre exemple fil rouge pour tout l'article :

$$f(x) = x^3 + x + 5$$

Alice affirme connaître une entrée qui produit 35. À la fin de l'article, on aura transformé cette affirmation en une simple vérification de divisibilité polynomiale. On utilisera la vraie solution $x = 3$ à chaque étape, mais rappelons-le : dans une vraie preuve, cette valeur est cachée à Bob.

Première étape : décomposer le calcul en morceaux assez petits pour être encodés sous forme de contraintes.

## Aplatissement : décomposer un calcul en portes

L'idée est de réécrire $f(x) = 35$ comme une séquence d'opérations élémentaires, chacune de la forme `résultat = gauche (op) droite`. On peut voir ça comme le démontage d'une formule en ses opérations les plus simples :

```
sym1 = x * x          // porte 1 : mise au carré
sym2 = sym1 * x        // porte 2 : mise au cube
sym3 = sym2 + x        // porte 3 : addition
out  = sym3 + 5        // porte 4 : ajout d'une constante
```

Chaque ligne est une **porte**. L'ensemble complet est un **circuit arithmétique**. Le processus d'encodage d'un calcul sous forme de portes s'appelle l'**arithmétisation** : c'est la première étape du pipeline SNARK. La forme aplatie encode exactement la même logique que $f(x)$. Juste une représentation différente.

![Arbre d'expression pour x³ + x + 5 aplati en quatre portes séquentielles](/img/zk-flattening.webp)

Notre exemple est une équation cubique, mais cette technique d'aplatissement se généralise à tout programme borné. L'addition et la multiplication sur un corps fini sont suffisamment expressives pour encoder des conditions, des comparaisons et des opérations bit à bit. Par exemple, étant donné une variable $b$ valant 0 ou 1, l'expression $y = 7b + 9(1 - b)$ choisit l'une des deux valeurs : un `if/else` construit à partir d'arithmétique. Les boucles se déroulent jusqu'à une profondeur fixe. Le nombre de portes croît avec la complexité du programme, mais l'encodage reste le même.

Revenons à notre exemple. On a quatre portes, et l'étape suivante consiste à exprimer chacune comme une contrainte qu'un vérificateur peut contrôler.

## R1CS : des contraintes comme produits scalaires

Chaque porte devient une contrainte sur un unique vecteur partagé. Le format s'appelle un **Rank-1 Constraint System (R1CS)**. Toute l'arithmétique à partir d'ici se fait sur un [corps fini](@/blog/math-behind-private-key.fr.md#les-corps-des-nombres-avec-de-l-arithmetique) $\mathbb{F}_p$. On utilise de petits entiers pour garder l'exemple lisible, mais en pratique $p \sim 2^{255}$.

Le vecteur $\mathbf{s}$ est la liste à plat de toutes les variables du circuit :

$$\mathbf{s} = [1,\ x,\ \text{out},\ \text{sym1},\ \text{sym2},\ \text{sym3}] \tag{1}$$

Avec $x = 3$ :

$$\mathbf{s} = [1,\ 3,\ 35,\ 9,\ 27,\ 30]$$

Ce $1$ en tête n'est pas une variable : c'est un terme constant qu'on verra en action quand la $\text{porte } 4$ encodera le « + 5 » de l'équation cubique. Ce vecteur, avec toutes ses valeurs intermédiaires remplies, est le **témoin** : c'est tout ce que le prouveur a calculé.

Chaque porte devient une contrainte de la forme :

$$(\mathbf{L}_i \cdot \mathbf{s}) \times (\mathbf{R}_i \cdot \mathbf{s}) = \mathbf{O}_i \cdot \mathbf{s} \tag{2}$$

où $\mathbf{L}_i$ (**l**eft, entrée gauche), $\mathbf{R}_i$ (**r**ight, entrée droite) et $\mathbf{O}_i$ (**o**utput, sortie) sont les $i$-èmes lignes des matrices $L$, $R$ et $O$. Chaque vecteur ligne « sélectionne » les bonnes variables dans $\mathbf{s}$.

### Portes 1 et 2 : encoder la multiplication

La $\text{porte } 1$ correspond à $\text{sym1} = x \times x$. En regardant $\mathbf{s}$ dans l'équation $(1)$, il faut des vecteurs qui sélectionnent $x$ à gauche, $x$ à droite et $\text{sym1}$ comme résultat :

$$\mathbf{L}_1 = [0, 1, 0, 0, 0, 0] \quad \text{(sélectionne } x \text{)}$$

$$\mathbf{R}_1 = [0, 1, 0, 0, 0, 0] \quad \text{(sélectionne } x \text{)}$$

$$\mathbf{O}_1 = [0, 0, 0, 1, 0, 0] \quad \text{(sélectionne sym1)}$$

Vérification : $(\mathbf{L}_1 \cdot \mathbf{s}) \times (\mathbf{R}_1 \cdot \mathbf{s}) = 3 \times 3 = 9 = (\mathbf{O}_1 \cdot \mathbf{s})$. Ça marche.

La $\text{porte } 2$ ($\text{sym2} = \text{sym1} \times x$) suit le même schéma de multiplication : $\mathbf{L}_2$ sélectionne $\text{sym1}$, $\mathbf{R}_2$ sélectionne $x$ et $\mathbf{O}_2$ sélectionne $\text{sym2}$.

### Portes 3 et 4 : encoder l'addition

La $\text{porte } 3$ ne contient pas de multiplication : $\text{sym3} = \text{sym2} + x$. On l'encode comme $(\text{sym2} + x) \times 1 = \text{sym3}$, en intégrant l'addition dans le vecteur $\mathbf{L}$. Le côté droit est simplement la constante $1$ :

$$\mathbf{L}_3 = [0, 1, 0, 0, 1, 0]$$

$$\mathbf{R}_3 = [1, 0, 0, 0, 0, 0]$$

$$\mathbf{O}_3 = [0, 0, 0, 0, 0, 1]$$

Vérification : $(27 + 3) \times 1 = 30$. La $\text{porte } 4$ ($\text{out} = \text{sym3} + 5$) fonctionne de la même façon, avec $\mathbf{L}_4 = [5, 0, 0, 0, 0, 1]$. Vérification : $(5 + 30) \times 1 = 35$.

Le point essentiel : les additions n'ajoutent pas de contraintes. Elles s'intègrent dans les vecteurs $\mathbf{L}_i$ ou $\mathbf{O}_i$ des portes existantes. Si le calcul d'origine avait été $(\text{sym2} + x) \times 2$ plutôt qu'une pure addition, cela n'aurait quand même fait qu'une seule porte : $\mathbf{L}_i$ sélectionne $\text{sym2} + x$, $\mathbf{R}_i$ sélectionne la constante $2$, et l'addition ne coûte rien de plus. Le nombre de contraintes est dicté par les multiplications ; les additions ne coûtent rien.

### Les matrices complètes

Chaque ligne est une porte, chaque colonne une entrée de $\mathbf{s}$.

![Les trois matrices R1CS L, R et O avec étiquettes de lignes et entrées non nulles mises en évidence](/img/zk-r1cs-matrices.webp)

Un témoin valide satisfait les quatre instances de l'équation $(2)$ simultanément. Un témoin invalide (par exemple, des valeurs intermédiaires fausses, une sortie incorrecte) en viole au moins une. On a maintenant établi que le R1CS fonctionne, mais aussi que chaque porte doit être vérifiée séparément : quatre vérifications pour quatre portes, chacune nécessitant trois produits scalaires et une multiplication.

## QAP : des produits scalaires aux polynômes

La transformation en **Programme Arithmétique Quadratique (QAP)** convertit ces quatre vérifications R1CS séparées en une seule vérification de divisibilité polynomiale.

La technique : prendre chaque *colonne* des matrices L, R, O et la transformer en polynôme via [l'interpolation de Lagrange](@/blog/verkle-trees-polynomial-commitments.fr.md#des-valeurs-a-un-polynome). L'article sur les arbres Verkle couvrait déjà cela : encoder des valeurs discrètes comme évaluations d'un polynôme. Même procédé, données différentes.

Concrètement, la colonne $j$ de la matrice $L$ contient 4 valeurs (une par porte). On les traite comme des évaluations aux points $t = 1, 2, 3, 4$ et on interpole un polynôme de degré 3 $L_j(t)$.[^tvar] Par exemple, la colonne $x$ de $L$ a les valeurs $[1, 0, 1, 0]$, donc $L_1(t)$ est l'unique polynôme de degré 3 passant par les points $(1, 1),\ (2, 0),\ (3, 1),\ (4, 0)$.

On répète pour chaque colonne de L, R et O. Chaque colonne correspond à une entrée de $\mathbf{s}$, donc 6 par côté ; appelons ces 18 polynômes les **polynômes par variable**.

On veut un unique polynôme qui, à chaque point de porte $t = i$, s'évalue au produit scalaire $\mathbf{L}_i \cdot \mathbf{s}$. Une propriété utile rend cela possible : une somme pondérée de polynômes est elle-même un polynôme. Les poids ne font que multiplier et combiner les coefficients. On peut donc utiliser les entrées du témoin $s_j$ comme poids sur les polynômes par variable :

$$L(t) = \sum_{j=0}^{5} s_j \cdot L_j(t)$$

$$R(t) = \sum_{j=0}^{5} s_j \cdot R_j(t)$$

$$O(t) = \sum_{j=0}^{5} s_j \cdot O_j(t)$$

À chaque point de porte $t = i$, $L(i)$ s'évalue à $\mathbf{L}_i \cdot \mathbf{s}$, le produit scalaire gauche pour la porte $i$. Idem pour $R(i)$ et $O(i)$. **Un seul polynôme par côté (3 au total), encodant les quatre portes d'un coup.**

Pour un témoin valide, chaque contrainte de porte est satisfaite, donc $L(t) \cdot R(t) - O(t) = 0$ en $t = 1, 2, 3, 4$. Appelons cette expression $T(t)$. C'est un polynôme ayant des racines en $1, 2, 3, 4$. Définissons le **polynôme cible** $Z(t) = (t-1)(t-2)(t-3)(t-4)$, qui a des racines aux mêmes quatre points.

Puisque $T(t)$ et $Z(t)$ partagent les mêmes racines, $Z(t)$ divise $T(t)$ sans reste. Le prouveur peut effectuer une division polynomiale pour obtenir un **polynôme quotient** $H(t)$ tel que :

$$T(t) = H(t) \cdot Z(t) \tag{3}$$

Si même une seule contrainte de porte était violée, $T(t)$ perdrait une racine, la division laisserait un reste, et aucun polynôme $H(t)$ n'existerait.

![Pipeline du programme à la vérification de divisibilité polynomiale : Programme, Aplatissement, R1CS, QAP, Vérification](/img/zk-pipeline.webp)

## Ce qu'une seule équation apporte

Le **lemme de Schwartz-Zippel** rend l'équation $(3)$ puissante : évaluer l'égalité des deux côtés en un point aléatoire $\tau$ suffit à vérifier un calcul ; si les polynômes diffèrent, la probabilité de passer par accident est négligeable.[^sz] Une seule évaluation aléatoire remplace les $n$ vérifications de portes. Les circuits de production ont des millions de portes (Zcash Sapling en utilise environ 1,5 million, les rollups zkEVM des dizaines de millions). C'est de là que vient le « succinct » dans SNARK.

L'approche naïve serait d'envoyer $T(t)$ et $H(t)$ au vérificateur et de le laisser choisir $\tau$. Mais ces polynômes ont un degré proportionnel au nombre de portes : $O(n)$ données sur le réseau, et $O(n)$ travail par vérificateur pour les évaluer. On vient de condenser $n$ vérifications de portes en une seule évaluation ; envoyer les polynômes annulerait ces gains. On veut donc que le prouveur évalue et n'envoie que les valeurs ponctuelles.

## Ce qui manque

Deux problèmes se dressent entre le QAP et une preuve véritable.

**Le prouveur ne doit pas connaître $\tau$.** S'il le connaît, il peut calculer $Z(\tau)$ (qui est public), choisir n'importe quel $H(\tau)$, poser $T(\tau) = H(\tau) \cdot Z(\tau)$, et satisfaire l'équation $(3)$ sans témoin valide. Le prouveur doit évaluer en $\tau$ sans jamais apprendre ce qu'est $\tau$.

**Le prouveur doit être lié aux polynômes du circuit.** Même avec un $\tau$ caché, rien n'oblige le prouveur à évaluer les vrais polynômes du QAP. Il pourrait en fabriquer d'autres, sans rapport, qui passent la vérification au point caché.

[La Partie 2](@/blog/zk-proofs-part2.fr.md) ajoute la cryptographie qui résout ces deux problèmes : une cérémonie de confiance qui cache $\tau$, des vérifications par couplage qui lient le prouveur aux polynômes du circuit, et une étape de masquage qui fait en sorte que la preuve ne révèle rien du témoin.

---

[^snark]: **S**uccinct **N**on-interactive **AR**gument of **K**nowledge (argument de connaissance succinct et non interactif).
[^tvar]: On utilise $t$ pour la variable polynomiale afin d'éviter la collision avec $x$, qui est déjà l'entrée du circuit.
[^sz]: Si $P$ et $Q$ sont des polynômes distincts de degré au plus $d$ sur $\mathbb{F}_p$, leur différence $D = P - Q$ est non nulle et de degré au plus $d$, donc elle a au plus $d$ racines. Un point d'évaluation aléatoire tombe sur l'une de ces racines avec une probabilité d'au plus $d/p$. Avec $p \sim 2^{255}$ et $d$ de l'ordre du millier, cette probabilité est négligeable.
