+++
title = "Zero-Knowledge : construire une preuve SNARK (Partie 2/3)"
date = 2026-04-17
updated = 2026-04-27
description = "Une cérémonie de confiance, huit engagements sur la courbe et cinq vérifications par couplage : comment transformer une identité polynomiale en SNARK à connaissance nulle fonctionnel."

[taxonomies]
tags = ["crypto", "computer-science"]

[extra]
katex = true
social_media_card = "/img/zk-proofs-part2-banner.webp"
+++

![Grues jumelles en origami gravées d'équations de couplage, se rejoignant sur un faisceau lumineux focal](/img/zk-proofs-part2-banner.webp)

L'équation (3) de la [Partie 1](@/blog/zk-computation-to-polynomials.fr.md) a réduit un programme à une seule vérification polynomiale : pour tout $\tau$ aléatoire, un témoin valide satisfait

$$L(\tau) \cdot R(\tau) - O(\tau) = H(\tau) \cdot Z(\tau)$$

C'est un énoncé algébrique limpide, mais ce n'est pas encore une preuve. Deux lacunes subsistent. Le prouveur doit évaluer en un $\tau$ qu'il ne connaît pas, et l'évaluation doit provenir des vrais polynômes du QAP, pas de polynômes arbitraires fabriqués pour passer la vérification.

Cet article comble ces deux lacunes, plus quelques autres qu'on découvrira en chemin. À la fin, on aura un SNARK complet : huit points de courbe, cinq vérifications par couplage, et une preuve qui ne révèle rien du témoin.

## Déplacer l'évaluation sur la courbe

La première lacune se comble en cachant $\tau$ dans des points de courbe. Une **cérémonie de confiance** choisit un scalaire aléatoire $\tau$ et publie ses puissances dans deux groupes d'ordre premier sur courbe elliptique :

$$G_1,\ \tau G_1,\ \tau^2 G_1,\ \ldots,\ \tau^d G_1$$

dans $\mathbb{G}_1$, plus les puissances parallèles $G_2, \tau G_2, \ldots, \tau^d G_2$ dans $\mathbb{G}_2$, puis détruit le scalaire lui-même. Plusieurs participants contribuent chacun à l'aléa de $\tau$, et tant qu'au moins l'un d'entre eux détruit honnêtement sa part, personne ne peut le reconstituer. L'aléa détruit s'appelle les **déchets toxiques**. Ce qui survit, c'est la liste de points ci-dessus : la **Common Reference String (CRS)**, suivant le même schéma que la [cérémonie de confiance](@/blog/verkle-trees-polynomial-commitments.fr.md#trusted-setup) de l'article Verkle.

![La cérémonie de confiance grave τ dans les rails CRS des points G₁ et G₂ ; quatre coefficients polynomiaux s'associent aux points dorés et convergent vers un engagement scellé à la cire P(τ)·G₁](/img/zk-trusted-setup-and-commitment.webp)

Quiconque détient les coefficients d'un polynôme $P$ peut désormais calculer $P(\tau) \cdot G_1$ sans apprendre $\tau$ : il multiplie chaque $\tau^k G_1$ publié par le coefficient $p_k$ et somme le tout. Le résultat est un **engagement** sur la valeur du polynôme au point caché, et c'est l'outil central qui permet au prouveur de transmettre au vérificateur des valeurs dépendantes du témoin sans révéler le témoin.

Cette CRS continuera de croître tout au long de cet article à mesure que de nouvelles vérifications réclameront de nouveaux points fixes. Chaque ajout suit la même recette : une expression dans les scalaires secrets, évaluée durant la cérémonie et publiée uniquement sous forme de point de courbe.

## Lier les engagements aux polynômes du QAP

Commençons par trois engagements : un par polynôme agrégé $L$, $R$ et $O$. Notons $\pi_A$ l'engagement sur $L(\tau)$, défini dans la Partie 1 comme la somme des polynômes par variable $L_i$ pondérés par le témoin :[^letters]

$$\pi_A = L(\tau) \cdot G_1 = \sum_i s_i \cdot L_i(\tau) \cdot G_1$$

où les $s_i$ sont les valeurs du témoin. L'**hypothèse de la connaissance de l'exposant (KoE)** force le prouveur à publier un $\pi_A$ calculé à l'aide de l'équation ci-dessus :

Étant donné une paire $(P, \alpha P)$ — un point de courbe et lui-même décalé par un scalaire inconnu $\alpha$ —, KoE affirme que la seule façon de produire une autre paire ayant le même décalage est de multiplier la paire d'origine par un second scalaire connu $c$, ce qui donne $(cP, c \cdot \alpha P) = (cP, \alpha \cdot cP)$.[^koe] KoE s'étend naturellement aux combinaisons linéaires : si l'on publie $(P_1, \alpha P_1), \ldots, (P_n, \alpha P_n)$, le prouveur peut renvoyer une autre paire α-décalée $(C, \alpha C)$ si et seulement si $C = \sum_j c_j P_j$ pour des coefficients $c_j$ que le prouveur connaît.

Le prouveur *doit* donc partir des $P_j$ publiés, ce qui est précisément la propriété nécessaire pour lier $\pi_A$ aux $L_i$. Comme le prouveur et le vérificateur connaissent tous deux les polynômes $L_i$, la cérémonie peut les inscrire en dur dans la CRS : pour chaque variable $i$, elle publie la paire $\big(L_i(\tau) \cdot G_1,\ \alpha_A \cdot L_i(\tau) \cdot G_1\big)$ avec un scalaire secret $\alpha_A$. Le prouveur combine ces paires avec les poids du témoin $s_i$, en utilisant les moitiés non décalées pour former $\pi_A$ et les moitiés α-décalées pour former $\pi_A'$.

Avec ces paires α-décalées et $\alpha_A \cdot G_2$ également dans la CRS, le vérificateur peut effectuer une vérification par couplage confirmant que $(\pi_A, \pi_A')$ a le bon décalage :

$$e(\pi_A',\ G_2) = e(\pi_A,\ \alpha_A \cdot G_2)$$

Un **couplage** $e(P, Q)$ prend deux points de courbe et produit un élément d'un groupe cible multiplicatif $\mathbb{G}_T$. La bilinéarité donne $e(aP, bQ) = e(P, Q)^{ab}$, donc les deux côtés ci-dessus se réduisent à $e(\pi_A, G_2)^{\alpha_A}$, et la vérification passe si et seulement si $\pi_A' = \alpha_A \cdot \pi_A$.[^pairings]

La même construction avec de nouveaux scalaires $\alpha_B$ et $\alpha_C$ donne $(\pi_B, \pi_B')$ et $(\pi_C, \pi_C')$. $\pi_B$ est construit dans $G_2$ plutôt que dans $G_1$ pour pouvoir ensuite être couplé avec $\pi_A$, tandis que $\pi_C$ reste dans $G_1$.

En résumé :

$$\begin{aligned}
\pi_A &= \sum_i s_i \cdot L_i(\tau) \cdot G_1 \\\\
\pi_B &= \sum_i s_i' \cdot R_i(\tau) \cdot G_2 \\\\
\pi_C &= \sum_i s_i'' \cdot O_i(\tau) \cdot G_1
\end{aligned} \tag{1}$$

avec leurs compagnons α-décalés :

$$\begin{aligned}
\pi_A' &= \alpha_A \cdot \pi_A \\\\
\pi_B' &= \alpha_B \cdot \pi_B \\\\
\pi_C' &= \alpha_C \cdot \pi_C
\end{aligned} \tag{2}$$

Rien encore ne force $s_i = s_i' = s_i''$ ; on pourrait avoir trois témoins candidats passés en parallèle.

## Lier les engagements à un témoin unique

Un prouveur qui passe chaque vérification sans détenir un seul témoin cohérent n'a rien prouvé. La solution est un quatrième engagement $\pi_K$ qui permet au vérificateur de s'assurer que les trois ensembles de témoins concordent.

Pour chaque variable $i$, la CRS publie un seul point qui regroupe les trois côtés sous un secret $\beta$ :

$$\beta \big(L_i(\tau) + R_i(\tau) + O_i(\tau)\big) \cdot G_1$$

Le prouveur multiplie chaque paquet par un seul poids $r_i$, un par variable, pour former :

$$\begin{aligned}
\pi_K = \beta \cdot G_1 \cdot \sum_i r_i \cdot \big({}&L_i(\tau) + R_i(\tau) \\\\
&{}+ O_i(\tau)\big) \tag{3}
\end{aligned}$$

Avec $\beta \cdot G_1$ et $\beta \cdot G_2$ également ajoutés à la CRS, le vérificateur effectue une vérification par couplage :

$$\begin{aligned}
e(\pi_K, G_2) &= e(\pi_A + \pi_C,\ \beta \cdot G_2) \\\\
&\quad \cdot e(\beta \cdot G_1,\ \pi_B)
\end{aligned}$$

$\pi_B$ se couple séparément puisqu'il appartient à $G_2$ tandis que $\pi_A, \pi_C$ restent dans $G_1$. Par bilinéarité, les deux côtés se réduisent à la même puissance de $e(G_1, G_2)$. On identifie les exposants, on simplifie β, et on égale terme à terme dans la somme :

$$\begin{aligned}
&r_i \cdot \big(L_i(\tau) + R_i(\tau) + O_i(\tau)\big) \\\\
&\quad = s_i \cdot L_i(\tau) + s_i' \cdot R_i(\tau) \\\\
&\quad\quad {} + s_i'' \cdot O_i(\tau), \quad \forall i
\end{aligned}$$

En supposant que les polynômes par variable $L_i, R_i, O_i$ sont linéairement indépendants, la vérification force $r_i$ à coïncider avec chacun des $s_i, s_i', s_i''$ pour chaque $i$, ramenant les trois premiers engagements à un témoin unique.

$\alpha$ lie chaque élément de preuve aux polynômes du QAP ; β lie les trois éléments au même témoin. On a maintenant un prouveur cohérent. Il reste une lacune : son témoin doit satisfaire les contraintes des portes.

## Lier les engagements à un témoin satisfaisant

La Partie 1 a montré que le polynôme quotient $H(t) = (L(t) \cdot R(t) - O(t)) / Z(t)$ se divise sans reste si et seulement si chaque porte est satisfaite, où $Z$ est le polynôme cible qui s'annule à toutes les racines des portes. Le prouveur engage $H(\tau)$ en utilisant les puissances de $\tau$ de la CRS, vues plus haut :

$$\pi_H = H(\tau) \cdot G_1 = \sum_j h_j \cdot (\tau^j \cdot G_1) \tag{4}$$

Une fois $Z(\tau) \cdot G_2$ ajouté à la CRS, le vérificateur peut effectuer :

$$e(\pi_A,\ \pi_B) = e(\pi_H,\ Z(\tau) \cdot G_2) \cdot e(\pi_C,\ G_2)$$

Les deux côtés se réduisent à des puissances de $e(G_1, G_2)$ ; par bilinéarité, identifier les exposants donne :

$$L(\tau) \cdot R(\tau) = H(\tau) \cdot Z(\tau) + O(\tau)$$

C'est l'identité QAP $L \cdot R - O = H \cdot Z$ évaluée en $\tau$. Par [Schwartz-Zippel](@/blog/zk-computation-to-polynomials.fr.md#ce-qu-une-seule-equation-apporte), si l'identité tient en un $\tau$ aléatoire caché, elle tient en tant qu'identité polynomiale avec une probabilité écrasante.

## Un noyau fonctionnel, deux lacunes restantes

Les équations (1) à (4) fournissent les huit engagements représentés à gauche, reliés aux cinq vérifications par couplage à droite :

![Huit sceaux de cire pour les éléments de preuve (six dorés G₁, deux argentés G₂) reliés à cinq clauses de vérification par couplage, codées par famille : KoE en doré, cohérence β en terracotta, divisibilité en cramoisi](/img/zk-snark-architecture.webp)

Ce qu'on a, c'est un SNARK fonctionnel, à un correctif près au niveau de la CRS qui referme une dernière faille de correction (ρ, traité dans l'annexe). Deux lacunes restent sur le chemin principal :

- **Entrées publiques.** Rien n'épingle encore la sortie revendiquée à ce que le vérificateur attend. La section suivante ajoute un découpage public/privé.
- **Connaissance nulle.** Les engagements pourraient laisser fuiter de l'information sur le témoin. La dernière section ajoute du masquage pour combler cette lacune.

## Fixer les variables publiques

Un énoncé typique d'un SNARK ressemble à ceci : « ce programme, exécuté sur mes entrées privées, produit cette sortie publique ». Le vérificateur a la sortie. Il veut une preuve que l'exécution a été honnête. Or, les cinq vérifications par couplage acceptent *n'importe quel* témoin cohérent : le prouveur peut choisir la sortie qu'il veut et passer quand même.

On corrige ça en découpant le vecteur témoin pour que le vérificateur possède la moitié publique :

$$\mathbf{s} = [\underbrace{s_0, s_1, \ldots, s_\ell}\_{\text{public}} \mid \underbrace{s_{\ell+1}, \ldots, s_n}\_{\text{privé}}]$$

La moitié publique contient la constante 1 et toutes les valeurs que le vérificateur doit voir, en général la sortie revendiquée. La moitié privée contient tout le reste.

À partir d'ici, $\pi_A$ ne porte plus que sur les indices privés ; $\pi_B$ et $\pi_C$ restent tels que définis, puisqu'un seul ancrage côté public suffit :[^aside]

$$\pi_A = \sum_{i \text{ privé}} s_i \cdot L_i(\tau) \cdot G_1$$

La moitié publique manquante revient au vérificateur, et la CRS contient déjà les $L_i(\tau) \cdot G_1$ dont il a besoin (issus de la cérémonie KoE). Le vérificateur sélectionne ceux qui correspondent aux indices publics et les combine avec les valeurs revendiquées de son côté :

$$L_\text{pub}(\tau) \cdot G_1 = \sum_{i \text{ public}} s_i \cdot L_i(\tau) \cdot G_1$$

Partout où les équations de vérification utilisaient le $L(\tau) \cdot G_1$ complet, elles utilisent désormais $L_\text{pub}(\tau) \cdot G_1 + \pi_A$ : la moitié du vérificateur plus celle du prouveur, recomposées sur la courbe.

La vérification de divisibilité devient :

$$\begin{aligned}
e\big(L_\text{pub}(\tau) \cdot G_1 + \pi_A,\ \pi_B\big) ={}& e(\pi_H,\ Z(\tau) \cdot G_2) \\\\
&\cdot e(\pi_C,\ G_2)
\end{aligned}$$

Le même argument que précédemment force l'identité QAP, désormais sur $L = L_\text{pub} + L_\text{priv}$ : elle tient si et seulement si $\pi_A, \pi_B, \pi_C, \pi_H$ ont été calculés à partir d'un témoin correspondant au $L_\text{pub}$ du vérificateur. Le vérificateur lie maintenant la preuve à la sortie qu'il spécifie : une preuve ne passe que si elle a été calculée pour cette valeur exacte.

## Masquer les engagements

Tout ce qui précède est **correct** : toute preuve que le vérificateur accepte signifie qu'un témoin valide existe pour l'énoncé. Mais la correction n'implique pas la confidentialité. $\pi_A, \pi_B, \pi_C$ sont des fonctions déterministes du témoin, donc quiconque a un témoin candidat en tête peut recalculer $\pi_A$ à partir de la CRS publique et le comparer à la preuve publiée : une correspondance confirme la supposition, une discordance l'écarte. La connaissance nulle exige que la preuve ne révèle *rien* du témoin.

La solution : randomiser chaque engagement avec un décalage que les équations de vérification absorbent. Le prouveur tire de nouveaux $\delta_L, \delta_R, \delta_O$ à chaque preuve :[^threedeltas]

$$\begin{aligned}
\pi_A &= L(\tau) \cdot G_1 + \delta_L \cdot Z(\tau) \cdot G_1 \\\\
\pi_B &= R(\tau) \cdot G_2 + \delta_R \cdot Z(\tau) \cdot G_2 \\\\
\pi_C &= O(\tau) \cdot G_1 + \delta_O \cdot Z(\tau) \cdot G_1
\end{aligned}$$

La vérification de divisibilité utilise déjà $Z(\tau) \cdot G_2$. Ajouter $Z(\tau) \cdot G_1$ à la CRS permet au prouveur de former chaque décalage sur la courbe.

La divisibilité survit. Développons les produits et substituons $L \cdot R = H \cdot Z + O$ pour le terme $LR$ nu. Le $O$ s'annule avec $-O$, et tout ce qui reste est multiplié par $Z$ :

$$\begin{aligned}
&\underbrace{(L + \delta_L Z)}\_{L_{\text{nouveau}}}\underbrace{(R + \delta_R Z)}\_{R_{\text{nouveau}}} - \underbrace{(O + \delta_O Z)}\_{O_{\text{nouveau}}} \\\\
&\quad = Z \cdot \underbrace{(H + \delta_L R + \delta_R L - \delta_O + \delta_L \delta_R Z)}\_{H_{\text{nouveau}}}
\end{aligned}$$

Le prouveur calcule maintenant $\pi_H = H_{\text{nouveau}}(\tau) \cdot G_1$. Les vérifications α et β s'équilibrent par la même astuce : la CRS ajoute des décalages assortis ($\alpha_A Z(\tau) G_1$, $\beta Z(\tau) G_1$, etc.) et les éléments compagnons les absorbent.

![Le même témoin produit trois engagements identiques en cire dorée sans masquage (la loupe de l'attaquant correspond), contre trois sceaux distincts avec des gouttes de masquage indépendantes (la loupe de l'attaquant ne voit qu'un brouillard de candidats)](/img/zk-blinding.webp)

Pour tout témoin fixé, $\pi_A, \pi_B, \pi_C$ sont uniformément distribués dans leurs groupes ($\delta$ est uniforme, $Z(\tau) \neq 0$). L'attaquant qui essayait de recalculer $\pi_A$ à partir d'un témoin candidat ne voit plus que du bruit : huit points de courbe, cinq vérifications par couplage, et zéro bit sur le témoin.

## Et ensuite

Une fois ajoutés le correctif ρ et le verrou γ de l'annexe, le schéma qu'on vient de dériver est **Pinocchio/BCTV14** (2013–2014), la construction que Zcash a livrée avec son pool blindé d'origine en 2016. Les raffinements modernes ont poussé le motif bien plus loin : un circuit de paiement privé compte environ 100 000 contraintes et tient dans une preuve de quelques centaines d'octets ; un rollup zkEVM, une chaîne L2 qui regroupe l'exécution Ethereum dans une preuve unique, agrège des dizaines de millions de contraintes et se situe autour de la même taille après encapsulation.[^wrapping]

La Partie 3 reprendra deux fils. D'abord, les trois protocoles qui ont façonné les systèmes de preuve ZK modernes : Groth16 réduit la preuve SNARK à 3 engagements et 1 vérification par couplage, PLONK substitue à la cérémonie propre à chaque circuit une cérémonie universelle qui débloque les zkVMs, et les STARKs quittent entièrement la famille des SNARKs, remplaçant les couplages par des engagements à base de hachages pour supprimer la cérémonie de confiance et obtenir la sécurité post-quantique. Ensuite, ce que les systèmes ZK de production prouvent en pratique : les transferts blindés de Zcash, les zkVMs généralistes qui transforment des programmes arbitraires en preuves, l'inférence ML vérifiable, et les systèmes d'identité ZK.

---

## Annexe

Deux annexes : ρ (la dépendance linéaire entre les $L_i, R_i, O_i$) et une fiche de référence dans la notation de la littérature.

<details>
<summary>Pourquoi ρ est nécessaire</summary>

L'argument de la β-vérification dans le corps de l'article reposait sur l'indépendance linéaire de $\{L_i, R_i, O_i\}$. Cette hypothèse ne tient pas en pratique : avec $N$ variables et $d$ portes, les $3N$ polynômes par variable appartiennent à un espace de dimension $d$, donc dès que $3N > d$ (la plupart des circuits), les relations linéaires sont la norme.

**Ce qui échoue quand l'indépendance échoue.** La β-vérification se réduit à :

$$\begin{aligned}
&\sum_i (r_i - s_i) L_i(t) \\\\
&\quad + (r_i - s_i') R_i(t) \\\\
&\quad + (r_i - s_i'') O_i(t) \equiv 0
\end{aligned}$$

Si l'indépendance tient, la seule solution est $s_i = s_i' = s_i'' = r_i$ : un témoin unique. La dépendance ouvre des solutions non triviales ; la divisibilité en rejette certaines mais pas toutes, et toute solution qui passe les deux vérifications est une preuve invalide que le vérificateur accepte.

**Le correctif ρ.** Deux scalaires secrets $\rho_A$ et $\rho_B$ pondèrent les trois côtés différemment : chaque $L_i$ devient $\rho_A L_i$, chaque $R_i$ devient $\rho_B R_i$, chaque $O_i$ devient $\rho_A \rho_B O_i$. Les engagements héritent de ces étiquettes :

$$\begin{aligned}
\pi_A &= \rho_A L(\tau) \cdot G_1 \\\\
\pi_B &= \rho_B R(\tau) \cdot G_2 \\\\
\pi_C &= \rho_A \rho_B O(\tau) \cdot G_1
\end{aligned}$$

L'identité β, réécrite avec ces étiquettes :

$$\begin{aligned}
&\sum_i (r_i - s_i) \rho_A L_i(t) \\\\
&\quad + (r_i - s_i') \rho_B R_i(t) \\\\
&\quad + (r_i - s_i'') \rho_A \rho_B O_i(t) \equiv 0
\end{aligned}$$

Comme $\rho_A, \rho_B$ sont secrets et algébriquement indépendants, des ρ-monômes distincts ne peuvent pas s'annuler : chaque coefficient doit s'annuler séparément. Cela force $r_i = s_i$ (via $\rho_A$), $r_i = s_i'$ (via $\rho_B$), $r_i = s_i''$ (via $\rho_A \rho_B$). Témoin unique, restauré.[^within]

Les autres vérifications restent équilibrées. Pour les α-vérifications, les deux côtés sont multipliés uniformément par $\rho_A$ ou $\rho_B$, donc les relations KoE tiennent toujours. La divisibilité fait apparaître un facteur commun $\rho_A \rho_B$ des deux côtés : la CRS publie $\rho_A \rho_B Z(\tau) \cdot G_2$ au lieu de $Z(\tau) \cdot G_2$, et le facteur s'annule.

</details>

<details>
<summary>Référence : spécification complète de BCTV14 dans la notation de la littérature</summary>

Les articles utilisent la notation par crochets $[x]_j = x \cdot G_j$ et nomment les polynômes par variable $A_i, B_i, C_i$ plutôt que $L_i, R_i, O_i$. Les scalaires grecs ($\alpha, \beta, \gamma, \rho, \delta, \tau$) gardent leurs noms d'un article à l'autre. Cette annexe reformule le protocole complet dans cette notation et associe chaque pièce à son équivalent dans le corps de l'article.

On suppose ρ (de l'annexe précédente) et γ actifs. Sous ces hypothèses, les expressions du corps héritent de facteurs ρ (le $\pi_A$ du corps devient $\rho_A L_\text{priv}(\tau) \cdot G_1$) et la β-vérification est verrouillée par γ. La table ci-dessous traduit la notation de cet article vers celle de la littérature.

**Table de notation.** Colonne de gauche : la notation du présent article ; colonne de droite : ce qu'on trouve dans BCTV14, Groth16 et la plupart des dérivés.

{% table(wide=true) %}
| Cet article | Littérature | Ce que c'est |
|---|---|---|
| $L_i, R_i, O_i$ | $A_i, B_i, C_i$ | polynômes par variable |
| $L, R, O$ | $A, B, C$ | polynômes agrégés (pondérés par le témoin) |
| $P(\tau) \cdot G_j$ | $[P(\tau)]_j$ | engagement sur $P$ dans le groupe $j$ |
| $\pi_A$ (privé seulement) | $[A\_{\text{mid}}]_1$ | engagement côté A du prouveur |
| $\pi_B, \pi_C$ | $[B]_2, [C]_1$ | engagements côtés B et C |
| $\pi_A', \pi_B', \pi_C'$ | $[A'\_{\text{mid}}]_1, [B']_1, [C']_1$ | compagnons α-décalés (tous dans $G_1$) |
| $\pi_K$ | $[K]_1$ | combinaison du β-paquet liant le témoin entre L, R, O |
| $\pi_H$ | $[H]_1$ | engagement sur le quotient |
| $L_\text{pub}(\tau) \cdot G_1$ | $\mathbf{vk}\_x$ | reconstruction des entrées publiques par le vérificateur |
| $\mathbf{s}$ | $\mathbf{a}$, $\mathbf{w}$ (varie) | vecteur témoin / d'assignation |
| $Z(\tau)$ | $Z(\tau)$, $t(\tau)$ (varie) | polynôme cible en τ |
{% end %}

Un point subtil sur $\pi_B'$ : le corps de l'article écrit $\pi_B' = \alpha_B \pi_B \in G_2$ par souci de concision, tandis que BCTV14 place $[B']_1$ dans $G_1$ via une paire KoE séparée. Les deux vérifient la même relation de décalage α ; la PK ci-dessous porte les éléments $G_1$ dont la construction de BCTV14 a besoin.

**Secrets de la cérémonie** (détruits après celle-ci) :

- $\tau$ : point d'évaluation caché
- $\alpha_A, \alpha_B, \alpha_C$ : décalages KoE
- $\beta$ : scalaire de cohérence du témoin
- $\gamma$ : verrou de la β-vérification[^gamma]
- $\rho_A, \rho_B$ : étiquettes distinctes pour L, R, O, empêchant les termes croisés entre familles de s'annuler dans l'identité β

Le prouveur tire de nouveaux $\delta_L, \delta_R, \delta_O$ à chaque preuve.

**Clé de preuve** (utilisée par le prouveur) :

- Puissances de $\tau$ : $[\tau^k]_1, [\tau^k]_2$ pour $k = 0, \ldots, d$
- Par variable privée $i$ :
  - Côté A : $[\rho_A A_i(\tau)]_1, [\alpha_A \rho_A A_i(\tau)]_1$
  - Côté B :
    - $[\rho_B B_i(\tau)]_1$
    - $[\rho_B B_i(\tau)]_2$
    - $[\alpha_B \rho_B B_i(\tau)]_1$
  - Côté C : $[\rho_A \rho_B C_i(\tau)]_1, [\alpha_C \rho_A \rho_B C_i(\tau)]_1$
  - β-paquet : $[\beta(\rho_A A_i(\tau) + \rho_B B_i(\tau) + \rho_A \rho_B C_i(\tau))]_1$

**Clé de vérification** (utilisée par le vérificateur) :

- $[1]_2$
- $[\alpha_A]_2, [\alpha_B]_1, [\alpha_C]_2$
- $[\gamma]_2, [\beta\gamma]_1, [\beta\gamma]_2$
- $[\rho_A \rho_B Z(\tau)]_2$
- Par variable publique $i$ (y compris $i = 0$ pour la constante 1) : $\mathbf{vk}\_{\text{IC},i} = [\rho\_A A\_i(\tau)]\_1$. Le vérificateur forme $\mathbf{vk}\_x = \sum\_{i \in \text{pub}} s\_i \cdot \mathbf{vk}\_{\text{IC},i}$.

**Preuve** (8 éléments de groupe) :

$$\begin{aligned}
\pi = \big( &[A\_{\text{mid}}]_1,\ [A'\_{\text{mid}}]_1,\ [B]_2,\ [B']_1, \\\\
&[C]_1,\ [C']_1,\ [K]_1,\ [H]_1 \big)
\end{aligned}$$

**Équations de vérification** (5 vérifications par couplage) :

1. **α pour A** (KoE) : $e([A'\_{\text{mid}}]_1, [1]_2) = e([A\_{\text{mid}}]_1, [\alpha_A]_2)$
2. **α pour B** (KoE) : $e([B']_1, [1]_2) = e([\alpha_B]_1, [B]_2)$
3. **α pour C** (KoE) : $e([C']_1, [1]_2) = e([C]_1, [\alpha_C]_2)$
4. **Cohérence β (verrouillée par γ) :** $e([K]_1, [\gamma]_2) = e([A\_{\text{mid}}]_1 + [C]_1, [\beta\gamma]_2) \cdot e([\beta\gamma]_1, [B]_2)$
5. **Divisibilité :** $e(\mathbf{vk}\_x + [A\_{\text{mid}}]_1, [B]_2) = e([H]_1, [\rho_A \rho_B Z(\tau)]_2) \cdot e([C]_1, [1]_2)$

Pour les éléments CRS exacts du masquage ZK et l'encodage des entrées publiques, voir l'implémentation de référence `r1cs_ppzksnark` de libsnark.

</details>

---

[^letters]: Pinocchio et BCTV14 écrivent les polynômes par variable $A_i, B_i, C_i$. Cet article utilise $L_i, R_i, O_i$ pour rester cohérent avec la Partie 1 et avec les agrégés $L, R, O$.
[^aside]: Découper les trois côtés triplerait le nombre de points CRS par entrée publique sans rien apporter de plus.
[^pairings]: Pour un exemple détaillé montrant la dérivation de l'identité polynomiale à la vérification bilinéaire, voir la section [vérification de la preuve par couplage](@/blog/verkle-trees-polynomial-commitments.fr.md#verification-de-la-preuve-par-couplage) de l'article Verkle.
[^koe]: KoE va au-delà de la dureté standard du logarithme discret. La dureté DL dit « on ne peut pas trouver $\alpha$ étant donné $\alpha G$ », ce qui est falsifiable : n'importe qui peut essayer. KoE affirme quelque chose de plus fort : *à chaque fois* qu'un prouveur renvoie une paire α-décalée correcte $(C, \alpha C)$, il existe un algorithme (un **extracteur**) qui récupère les coefficients que le prouveur a utilisés pour construire $C$. C'est une affirmation sur l'existence d'un algorithme, pas une borne computationnelle, donc aucune expérience ne peut la réfuter. Si KoE casse, les SNARKs cassent.
[^threedeltas]: Trois scalaires indépendants, pas un seul partagé. Un δ partagé corrélerait les engagements : on déduit δ de l'un, on prédit les autres.
[^wrapping]: Les zkEVMs prouvent en général le gros de leur travail avec un STARK (rapide, sans cérémonie de confiance, grosse preuve), puis re-prouvent cette vérification STARK à l'intérieur d'un circuit Groth16. L'« encapsulation » réduit la grosse preuve STARK à une petite preuve publiée sur la chaîne.
[^within]: Les identités par monôme exigent que chacune des trois familles ($L_i$, $R_i$, $O_i$), prise sur tous les indices $i$, soit linéairement indépendante en elle-même. Une dépendance interne à une famille (par exemple parmi les $L_i$) peut encore exister, mais elle ne pose problème que si elle implique des indices d'entrée publique ou traverse la frontière public/privé, ce que la construction QAP exclut via des mécanismes hors du périmètre de cet article.
[^gamma]: γ est un artefact de génération de CRS issu de la réduction de correction héritée de GGPR/Pinocchio, pas une défense au niveau du protocole. Le détail complet figure dans [GGPR §5.3](https://eprint.iacr.org/2012/215.pdf) (EUROCRYPT 2013 ; ePrint 2012/215) ; dans le modèle du groupe générique, γ n'est pas strictement nécessaire ([Gabizon 2019](https://eprint.iacr.org/2019/119.pdf)).
