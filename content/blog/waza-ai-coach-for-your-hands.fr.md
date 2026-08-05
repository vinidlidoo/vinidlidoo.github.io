+++
title = "Waza : un coach IA pour vos mains"
date = 2026-08-03
description = "Construire un coach IA qui voit ce que vous voyez et vous guide à la voix dans des savoir-faire concrets"

[taxonomies]
tags = ["waza"]

[extra]
stylesheets = ["css/video.css"]
social_media_card = "img/waza-ai-coach-for-your-hands-banner.webp"
+++

![Un atelier lumineux avec un établi blanc portant une boîte à outils rouge, une perceuse sans fil, un pot de peinture, un baluchon furoshiki rouge, de la laine et des baguettes. Au mur, un écusson noir peint à la main porte le logo Waza en enseigne au néon : une ligne blanche en zigzag reliant un objectif de caméra à des arcs de signal rouges lumineux.](/img/waza-ai-coach-for-your-hands-banner.webp)

Les débats les plus vifs du moment portent sur la question de savoir si, et à quelle vitesse, l'IA supprimera le travail humain par l'automatisation, et sur la façon dont la société devrait gérer cette transition. Au début du mois, des économistes, des lauréats du prix Nobel et des dirigeants de la tech ont signé une lettre intitulée [« We Must Act Now »](https://www.businessinsider.com/ai-job-impact-read-letter-economists-executives-openai-anthropic-google-2026-7), qui alerte sur des suppressions d'emplois à grande échelle ; la semaine dernière, Jensen Huang a rétorqué que l'IA [« tue des tâches, pas des emplois »](https://fortune.com/2026/07/28/nvidia-jensen-huang-ai-killing-tasks-not-jobs/), estimant que cette peur prend le problème exactement à l'envers. L'issue de ce débat compte, et je ne prétends pas savoir qui a raison. Ce que je sais, c'est que les deux derniers mois m'ont conduit vers un pan de l'IA où il y a beaucoup moins à craindre et beaucoup à gagner : élargir le champ de ce qu'une personne peut accomplir de ses propres mains, en toute confiance.

Cette confiance s'érode depuis un moment. Le travail intellectuel nous retient devant un écran la majeure partie de la journée, et nous avons réagi en confiant le versant physique de la vie à des spécialistes, une tâche à la fois. Quelqu'un d'autre cuisine (un restaurant, un service de repas préparés) ; un plombier se déplace pour une intervention de huit minutes ; un homme à tout faire pose l'étagère. Chaque délégation se justifie prise isolément. Additionnez-les, et notre capacité à composer avec le monde physique s'atrophie en silence, au point qu'un robinet qui goutte ressemble désormais à un coup de fil à passer plutôt qu'à la réparation de vingt minutes qu'il demande bien souvent.

Rien n'oblige à continuer ainsi. **Waza** (技, « technique » en japonais) est ma tentative d'infléchir cette courbe : une IA qui voit ce que vous voyez et vous guide dans la tâche que vous avez devant vous. Cet article passe en revue ce qu'elle fait aujourd'hui (vidéos de démonstration à l'appui) et ce vers quoi cela mène, à mon avis. Si vous utilisez un iPhone et possédez des lunettes Meta, une invitation vous attend à la fin.

## Le plafond actuel

Disons que vous vous attaquez au robinet. La technologie d'aujourd'hui vous offre deux recours, et chacun montre ses limites à sa façon. Un assistant conversationnel peut vous expliquer la réparation, mais il travaille à l'aveugle : lui montrer votre robinet, c'est s'arrêter pour se sécher les mains et prendre une photo, et les conseils reviennent sous forme de mots quand ce que vous voulez vraiment, c'est qu'on vous montre. YouTube a exactement le problème inverse : tout y est démonstration, par quelqu'un qui sait ce qu'il fait, mais la vidéo ne peut pas *vous* voir, donc elle ne peut pas vous dire ce que vous faites de travers. Quiconque a repassé en boucle les quinze mêmes secondes d'un tutoriel pendant que son projet traînait en pièces détachées connaît ce plafond.

Ce que j'aimerais avoir à la place ressemble à la scène de *Matrix* où Neo charge le kung-fu dans sa tête (peut-être sans le téléchargement instantané dans mon cortex moteur). Vous chargez un savoir-faire, et un coach vous regarde travailler, s'adapte à ce que vous savez déjà et vous corrige en images, pas en paragraphes, parce que les humains absorbent la vidéo bien plus vite que le texte. C'est la différence entre regarder quelqu'un changer un pneu et lire le manuel rangé dans la boîte à gants.

Un tel coach n'a sûrement pas sa place dans une fenêtre de chat. Il doit vous accompagner, garder vos mains libres et placer ces images là où un simple coup d'œil suffit à les capter. La technologie nécessaire à cette expérience arrivera sans doute par étapes : d'abord, un téléphone calé à côté de l'ouvrage, qui joue les clips ; ensuite, les mêmes clips en incrustation sur le verre de lunettes à affichage (la première génération est [déjà en rayon](https://www.meta.com/ai-glasses/meta-ray-ban-display-glasses-and-neural-band/)) ; et un jour, quand de véritables lunettes de réalité augmentée atteindront le grand public, des repères dessinés directement sur ce que vous êtes en train de faire. Chaque étape rapproche le retour visuel de vos yeux.

## Ce que Waza fait aujourd'hui

La question à laquelle j'ai voulu répondre en juin était donc : que peut-on construire de tout cela dès maintenant ? Une bonne partie, en fait. Waza est une app iOS qui se connecte à des lunettes Meta (ou à un téléphone en dépannage) et transmet en direct votre vue subjective au coach IA, qui vous guide pas à pas dans un savoir-faire et répond aux questions en une seconde environ.

Chaque étape d'un savoir-faire s'accompagne d'un court clip en vue subjective montrant le geste à accomplir, et le coach joue le bon au bon moment. Demandez à voir l'étape en action, ou ratez-la, et le clip apparaît sur votre téléphone. Celui-ci est calé à proximité ; vous y jetez un œil et gardez les deux mains sur l'ouvrage. C'est ce qui distingue un coach d'un tutoriel parlant. Le catalogue compte une douzaine de savoir-faire pour commencer, du nœud de cravate à la soudure de fils, en passant par le steak saisi à la poêle.

<div class="video-row">
  <video controls preload="metadata" playsinline poster="https://pub-94e31bf482a74272bb61e9559b598705.r2.dev/video/chopsticks-hero-poster.jpg">
    <source src="https://pub-94e31bf482a74272bb61e9559b598705.r2.dev/video/chopsticks-hero.mp4" type="video/mp4">
  </video>
  <video controls preload="metadata" playsinline poster="https://pub-94e31bf482a74272bb61e9559b598705.r2.dev/video/necktie-hero-poster.jpg">
    <source src="https://pub-94e31bf482a74272bb61e9559b598705.r2.dev/video/necktie-hero.mp4" type="video/mp4">
  </video>
  <video controls preload="metadata" playsinline poster="https://pub-94e31bf482a74272bb61e9559b598705.r2.dev/video/furoshiki-hero-poster.jpg">
    <source src="https://pub-94e31bf482a74272bb61e9559b598705.r2.dev/video/furoshiki-hero.mp4" type="video/mp4">
  </video>
</div>

(Si vous possédez déjà des lunettes Meta et voulez essayer vous-même, filez directement à l'[invitation](#invitation).)

## Enregistrer une fois, coacher pour toujours

Le coach ne vaut que par son catalogue de savoir-faire, donc la création doit être facile. Il s'avère que les modèles modernes de compréhension vidéo peuvent la rendre presque automatique. Pour créer un savoir-faire, vous vous filmez une fois en train d'accomplir la tâche, en commentant au fil de l'eau, puis vous importez la séquence. Le pipeline la découpe en étapes, ajuste chaque clip, zoome sur le geste clé et rédige le prompt de coaching étape par étape. Une démonstration de dix minutes devient un savoir-faire complet et coaché en deux minutes et demie environ, sans la moindre intervention humaine.

La friction ainsi levée est décisive. Un tutoriel YouTube digne d'être regardé est devenu une production : script, prises multiples, éclairage, puis des heures dans un logiciel de montage comme Final Cut Pro. Ce niveau d'exigence écarte la plupart des gens qui possèdent réellement ces savoir-faire. Waza demande une seule prise et se charge du montage. Votre voix brute n'est jamais publiée (seuls vos mots guident le coach) ; il n'y a donc aucune prestation à assurer non plus. Quiconque sait faire quelque chose (cuisine, bricolage, artisanat, technique professionnelle) peut l'enseigner en le faisant une fois devant la caméra.

<video class="video-solo" controls preload="metadata" playsinline poster="https://pub-94e31bf482a74272bb61e9559b598705.r2.dev/video/record-skill-demo-v2-poster.jpg">
  <source src="https://pub-94e31bf482a74272bb61e9559b598705.r2.dev/video/record-skill-demo-v2.mp4" type="video/mp4">
</video>

## Rien à toucher

Les apps sont encore pensées pour les pouces. Menus, barres de recherche, pages de détail : tout est conçu pour être touché du doigt, et c'est justement ce qu'on ne peut pas faire avec des mains mouillées, grasses ou déjà occupées. Dans Waza, le coach pilote aussi l'app. Demandez-lui de chercher dans le catalogue, de parcourir une catégorie ou d'ouvrir la page d'un savoir-faire, et tout se passe à l'écran pendant que vos mains restent sur l'ouvrage. Charger le savoir-faire, lancer la session, la mettre en pause : tout se fait à la voix.

Rien de tout cela n'était envisageable jusqu'à récemment. Les modèles vocaux comme GPT-Realtime-2.1 d'OpenAI savent désormais appeler des outils en pleine conversation, si bien que le même modèle qui vous coache sur une étape peut aussi appuyer sur les boutons de l'app. La commande vocale a cessé depuis longtemps d'être l'expérience fragile du « dites “menu” pour les options », pour ressembler davantage à une conversation avec quelqu'un qui tiendrait votre téléphone à votre place. C'est à mes yeux l'un des changements les plus sous-estimés de la tech cette année, et nulle part il ne compte autant que lorsque vos deux mains sont prises.

<video class="video-solo" controls preload="metadata" playsinline poster="https://pub-94e31bf482a74272bb61e9559b598705.r2.dev/video/voice-navigation-hot-tub-poster.jpg">
  <source src="https://pub-94e31bf482a74272bb61e9559b598705.r2.dev/video/voice-navigation-hot-tub.mp4" type="video/mp4">
</video>

## Le bon usage des lunettes à caméra

Aujourd'hui, les lunettes à caméra n'ont pas très bonne réputation, et ce n'est pas sans raison. Les histoires qui circulent parlent de porteurs filmant des inconnus qui n'ont jamais accepté d'être à l'image, et cette gêne colle à la catégorie partout où elle passe, depuis les Google Glass en 2013.

Le coaching est l'usage inverse. Une session pointe la caméra vers les objets dans vos mains, sur votre propre établi ou votre cuisinière, et pratiquement jamais vers une autre personne. Et le format se prête à la tâche comme s'il avait été conçu pour elle : vos mains restent libres, et la caméra regarde l'ouvrage sous le même angle que vous, suivant votre attention à mesure que vous tournez autour. Une seule app ne redorera pas le blason des lunettes à caméra, mais des usages comme celui-ci pourraient y parvenir avec le temps.

## Où cela mène

La prochaine étape consiste à découvrir si Waza est réellement utile sur le terrain, et où elle pèche. À court terme, cela veut dire : sortir des fonctionnalités qui fluidifient l'expérience (demander au coach d'ajouter ou de corriger un savoir-faire, ou se faire guider pour enregistrer sa propre démo), et étoffer le catalogue à mesure que les gens essaient des savoir-faire, créent les leurs et les font circuler. Chaque tour de cette boucle rend le coach plus utile.

Ce à quoi je reviens sans cesse, c'est ce qui se passera quand les utilisateurs et les savoir-faire atteindront une masse critique. Les savoir-faire sont faits d'étapes, et les étapes se recombinent. Imaginez un vélo électrique avec un pneu arrière à plat : atteindre la chambre à air suppose de s'occuper d'abord du câble du moteur, et disons qu'aucun savoir-faire ne couvre encore ce cas précis. Mais si le catalogue contient à la fois le savoir-faire « réparer une crevaison » d'un mécanicien vélo et le savoir-faire « connecteurs de câbles » d'un passionné d'électronique, l'IA pourrait combiner des étapes de l'un et de l'autre en un guide de réparation que personne n'a jamais écrit. C'est la **composabilité**, et elle rend le catalogue plus précieux que la somme de ses savoir-faire : mettez les lunettes, dites « j'aimerais réparer ça », et un savoir-faire personnalisé s'assemble sur-le-champ, repères visuels à l'appui.

L'échelle change aussi le versant humain. Un terrain que j'aimerais tester : la formation encadrée, dans des lieux qui enseignent déjà des savoir-faire manuels, avec des sessions animées à distance, un guide humain prêt à intervenir dès que l'IA atteint ses limites, et des centaines de personnes travaillant le même savoir-faire en même temps. La prise de relais, elle, fonctionne déjà dans Waza aujourd'hui (matière à un autre article).

Voilà un avenir que j'aurais plaisir à construire : plus de gens (enfants compris) qui fabriquent et réparent des choses réelles dans le monde réel, assez guidés pour réussir, au lieu de regarder un écran pendant que le logiciel agit à leur place. Quant à savoir s'il vaut la peine d'être construit, seuls les utilisateurs peuvent me le dire, et c'est à cela que sert l'invitation ci-dessous. On ne peut pas encore charger le kung-fu, mais le coaching pour tous est peut-être déjà à portée de main.

<a id="invitation"></a>

## Invitation

Je constitue un petit groupe de premiers testeurs : des gens que ce genre de technologie enthousiasme et que les aspérités ne rebutent pas. Il faut de toute façon commencer petit : Meta limite à une centaine le nombre de testeurs par app pour les lunettes, jusqu'à [l'ouverture de la publication en 2026](https://developers.meta.com/blog/introducing-meta-wearables-device-access-toolkit/). Si votre téléphone principal est un iPhone, que vous possédez des lunettes Meta (Ray-Ban ou Oakley) et que vous vous demandez quoi en faire au-delà des photos et de la musique, [écrivez-moi](https://vinidlidoo.github.io/fr/contact/) et je vous prépare un build TestFlight ainsi que l'invitation pour les lunettes. Pas de lunettes mais curieux ? Voici le [lien vers la boutique](https://www.meta.com/ai-glasses/) (sans commission ni affiliation).

Et si vous développez ou enseignez quelque chose de voisin (guidage en direct, vidéo égocentrée, capture de savoir-faire, enseignement pratique en tout genre), j'aimerais beaucoup qu'on échange. Même [page de contact](https://vinidlidoo.github.io/fr/contact/) ; dites-moi ce sur quoi vous travaillez.
