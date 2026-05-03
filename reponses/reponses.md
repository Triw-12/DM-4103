## Réponses aux questions


## Question 2

Nous avons regardé trois graphes représentatifs (Caltech, MIT, Johns Hopkins). Tous sont relativement « sparse » mais présentent une forte organisation locale : Caltech a surtout une densité et un clustering bien plus élevés (coefficient global ≈ 0,29 ; clustering local moyen ≈ 0,41 ; densité ≈ 0,056) tandis que MIT et Johns Hopkins sont plus épars (densités ≈ 0,012 et 0,014) donc les étudiants sont moins interconnectés. Concrètement, cela se traduit par beaucoup de groupes locaux peu connectés et d’un petit nombre de nœuds très connectés servant de ponts entre ces groupes, qui est appelé un réseau « small world » selon les articles.

Le diagramme degré vs clustering confirment une tendance classique : les nœuds à fort degré ont un clustering local plus faible, cad qu’il ils font plutôt le lien entre communautés que partie intégrante d’un petit voisinage dense, ce qui est typique des réseaux sociaux.

## Question 3

Sur les 100 réseaux de Facebook100, l’assortativité par degré reste en moyenne proche de zéro, avec une forte variabilité d’un réseau à l’autre. Autrement dit, le degré n’explique pas du tout qui se connecte avec qui. En revanche, certains attributs sociaux (dortoir, genre, année, filière) montrent localement des effets d’homophilie importants, surtout pour "student_fac" et "dorm" : sur certaines universités le dortoir regroupe fortement les liens, sur d’autres il a peu d’impact.
Finalement, seul l'année academique (student_fac) semble avoir un effet d'homophilie relativement universel, ce qui est logique : les étudiants de la même année partagent souvent des cours et activités, favorisant les connexions.

## Question 4

J’ai testé Common Neighbors, Jaccard et Adamic/Adar. Le résultat est clair : Adamic/Adar surclasse les deux autres dans nos expériences (MIT, Caltech, différentes fractions d’arêtes retirées), ce qui est en adéquation avec les résultat du papier de recherche. Adamic/Adar fonctionne bien ici parce qu’il valorise les voisins communs peu connectés, ce qui évite de sur‑pondérer les hubs.

## Question 5

Le modèle implémenté est un Graph Convolutional Network (GCN) à deux couches avec : 

- Deux couches (comme dans le papier de référence) : Deux couches offrent un bon compromis entre information et rapidité. Deux couches permettent à chaque nœud d’accumuler l’information de ses voisins directs et indirects.
- Dimension cachée (d_hidden=50) : Choisie volontairement modérée
- Activation ReLU : Appliquée entre les deux couches pour introduire de la non-linéarité
- Dropout (p=0.5) : Pareil, paramètre classique et modéré.

Fonction de perte : cross-entropy.
Elle mesure la divergence entre distribution prédite et vraie label, ce qui est appropriée pour une classification multi-classe, comme notre cas.
En calculant la perte uniquement sur les nœuds étiquetés, elle ignore complètement les nœuds sans label, les traitant comme données non supervisées.

Entrainement : 100 epochs, ce qui est suffisant pour observer une convergence raisonnable sans surajuster, surtout avec un modèle simple.
Puis on fait 70% pour l'entrainement, 10% pour la validation et 20% pour le test, pour avoir un bon entraînement tout en gardant une part significative pour évaluer la performance finale.

Pour les résultats : `gender` est le plus facilement retrouvé (accuracy ≈ 0,54), `dorm` et surtout `major_index` sont beaucoup plus difficiles (accuracy ≈ 0,24 pour `dorm`, ≈ 0,215 pour `major_index`, MAE élevée pour ces derniers). Raisons probables : `gender` a peu de classes (normalement binaire) et une corrélation locale forte, alors que `major_index` a une cardinalité élevée et beaucoup de bruit. Le modèle utilisé est très simpliste, d'où le fait que les résultats ne sont vraiment pas assez précis (même s'il y a une amélioration par rapport au hasard). C'est peut-être dû à mon implémentation et des améliorations sont sûrement possibles.

## Question 6

Mon hypothèse est la suivante : Les étudiants vivant dans le même dortoir partagent des interactions quotidiennes accrues, ce qui favorise la formation de communautés en ligne denses, plus que les spécialités académiques.

Pour comparer les deux critères, je vais appliquer l’algorithme de Louvain pour détecter les communautés dans le graphe de MIT, puis évaluer la correspondance entre les communautés détectées et les attributs `dorm` et `major_index` à l’aide de métriques d’évaluation comme l’AMI (Adjusted Mutual Information) ou NMI (Normalized Mutual Information).

Les résultats sont les suivants :

```text
Résultats pour le graphe MIT8 :
Nombre de communautés détectées : 28
Score AMI pour dorm : 0.2256766487206284
Score NMI pour dorm : 0.24380203155185576
Score AMI pour major : 0.024867816848016564
Score NMI pour major : 0.04204440694417952

Résultats pour le graphe Caltech36 :
Nombre de communautés détectées : 11
Score AMI pour dorm : 0.5316098099084354
Score NMI pour dorm : 0.5429992934572379
Score AMI pour major : 0.01991634978566053
Score NMI pour major : 0.08095200922803535

Résultats pour le graphe Johns Hopkins55 :
Nombre de communautés détectées : 18
Score AMI pour dorm : 0.18633126044990428
Score NMI pour dorm : 0.2037545056946955
Score AMI pour major : 0.08396483553405747
Score NMI pour major : 0.1081917701228274
```

Ils montrent en effet que le dortoir a un impact plus fort que la spécialité académique pour expliquer les communautés détectées, surtout pour Caltech où le dortoir est un facteur structurant majeur. Cependant, l’effet n’est pas universel : sur MIT et Johns Hopkins, les scores sont plus faibles, suggérant que d’autres facteurs peuvent aussi jouer un rôle important dans la formation des communautés sur les réseaux sociaux universitaires.