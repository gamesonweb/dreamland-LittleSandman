---
slug: music
title: 🎻 Musique multipiste
authors:
  - name: Björn LAI
tags: [musique]
---

Bonjour à tous !

Dans cet article, nous expliquerons comment nous avons réalisé la musique du premier niveau du jeu,
ainsi que les problèmes rencontrés au cours de cette étape.

La particularité de cette musique est qu'elle commence avec peu d'instruments (le piano et le clavecin),
et d'autres s'ajoutent au fur et à mesure qu'on progresse afin de créer un effet de tension.


## 🎶 Composition avec MuseScore

La musique a été réalisée à l'aide de MuseScore, un logiciel gratuit permettant de composer des morceaux en utilisant la 
notation musicale standard, sur une partition.

![musescore image](/img/musescore.png)

L'utilisation de ce logiciel est très pratique, car elle offre de nombreuses options d'exports,
et notamment celle qui nous intéresse : exporter les pistes séparément.

Cependant, un autre problème nous attend...

## ♾️ Problème de loop

En général, dans les jeux-vidéos, la musique doit pouvoir tourner en boucle sans transition brusque,
et sans blanc à la fin. Et c'est ce qu'on souhaite pour notre musique.

Cependant, une particularité de MuseScore est que lorsqu'on exporte une musique, un blanc est rajouté à la fin pour permettre
aux dernières notes de résonner jusqu'à leur évanouissement complet, et il n'existe aucune option pour empêcher cela.

Après l'export, on obtient sept fichiers audio au format ogg, un pour chaque instrument, mais comme ils ont chacun
un blanc à la fin, la boucle musicale ne peut malheureusement pas poursuivre de façon fluide.

Il faut trouver un moyen de garantir une boucle parfaite !

## 💿 Raccourcissement avec un script python

Notre objectif est de couper la fin de chaque piste, en supprimant tout le temps écoulé après la mesure finale. 

On peut faire cela en utilisant n'importe quel éditeur audio, comme Audacity.

Toutefois, le nombre de pistes étant relativement
élevé, cela peut s'avérer fastidieux. Et de plus, un découpage manuel peut entrainer des imprécisions, ce qui entrainerait 
une terrible désynchronisation des pistes après quelques tours de boucle.

Il devient donc particulièrement intéressant d'automatiser ce processus.

Pour ce faire, nous avons utilisé un script python utilisant le module de traitement audio _pydub_.
Cela a permis d'automatiser efficacement le découpage des pistes, en garantissant une précision à la milliseconde près !


## 🔊 Chargement des sons dans Babylon.js

Il ne reste plus qu'à importer les sons dans Babylon. Mais prudence, il reste un dernier piège à éviter.

Si on charge successivement toutes les pistes, puis, qu'on essaie immédiatement de les jouer, on peut obtenir le problème suivant : 
aucun son ne joue, car le chargement des sons n'est pas encore terminé et est toujours en train de continuer de manière asynchrone.

Pour éviter cela, on utilise le paramètre optionnel _readyToPlayCallback_ qui appellera une fonction à chaque fois qu'un son finit de charger.
Cette dernière fonction a simplement pour effet d'incrémenter un compteur et de démarrer simultanément tous les sons, si le quota de sons à charger est atteint.

Ce procédé est démontré dans ce [playground](https://playground.babylonjs.com/#PCY1J#6) de la doc de Babylon.js.

Et voilà ! Une fois que tous les sons sont démarrés en même temps, on n'a plus qu'à couper le son des pistes qui ne doivent pas encore jouer, 
et à les réactiver petit à petit dès que le joueur atteint un objectif dans le niveau. 