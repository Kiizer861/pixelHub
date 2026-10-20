COLIN Noé 
BUT3 TD2-APP

Réponses aux questions :

**Q1.** `INCR` est une opération **atomique**. Qu'est-ce que ça veut dire concrètement, et pourquoi est-ce important si mille joueurs terminent une partie en même temps ? Comment auriez-vous fait la même chose en SQL, et quelle différence cela fait-il ?

INCR est atomique : l'incrémentation se fait en une seule opération.
Avec 1 000 joueurs qui terminent une partie en même temps, le compteur augmente correctement de 1 000.
En SQL, on utilise UPDATE ... SET valeur = valeur + 1. PostgreSQL gère aussi les accès concurrents, mais Redis est souvent plus rapide pour les compteurs simples.

**Q2.** On aurait pu sérialiser tout le joueur en JSON dans une simple `String`. Décrivez les étapes qu'il aurait alors fallu faire pour ajouter 250 XP, et le problème qui apparaît si deux requêtes le font en même temps.

sérialiser et désérialiser tout le joueur en JSON 
Les données seront perdues 

**Q3.** Les deux dernières commandes ne renvoient pas la même chose. Expliquez la différence, et dites laquelle vous utiliserez pour afficher un « top 3 ».

ZREVRANGE affiche les joueurs du score le plus élevé au plus faible.
ZRANGE affiche les joueurs du score le plus faible au plus élevé.
Pour afficher le top 3, on utilise ZREVRANGE.

**Q4.** `ZREVRANK` renvoie un nombre. Si vous affichez ce nombre tel quel au joueur, que va-t-il lire ? Que faut-il faire ?

ZREVRANK commence à 0. Le joueur verra donc un rang décalé d'une place.
Il faut ajouter 1 au résultat pour afficher le classement correctement.

**Q5.** Qui sort de la file au `RPOP`, et pourquoi ? Qu'aurait donné `LPOP` à la place ?

RPOP sort Nova, car il a été ajouté en premier avec LPUSH.
LPOP aurait sorti Ombre, le dernier joueur ajouté.

**Q6.** Quelles valeurs obtenez-vous ? Que renvoie `TTL` sur une clé sans expiration (testez avec `compteur:parties`) ? Que signifie chacune des valeurs de retour ?

Après 30 secondes : TTL renvoie -2 et EXISTS renvoie 0 ,la clé a expiré.
Pour compteur:parties, TTL renvoie -1 ,la clé n'a pas d'expiration.
TTL positif : temps restant en secondes.

**Q7.** Personne n'a lancé de tâche de nettoyage. Quel code auriez-vous dû écrire pour obtenir le même comportement avec PostgreSQL, et quels risques ce code introduit-il ?

Il aurait fallu utiliser une tâche périodique SQL avec DELETE pour supprimer les joueurs dont la date d’expiration est dépassée.
Risques : la tâche peut prendre du temps, consommer des ressources et supprimer les données en retard si elle ne s'exécute pas correctement. Redis le fait automatiquement grâce au TTL.

**Q8.** Ce service est enregistré en **singleton**. Cherchez dans la documentation de StackExchange.Redis ce qu'est un `ConnectionMultiplexer` et ce qui se passerait si on en créait un à chaque requête HTTP. Pourquoi ce bug serait-il **invisible** aujourd'hui en TP ?

ConnectionMultiplexer gère les connexions à Redis et doit être partagé entre les requêtes.
Si on en crée un à chaque requête HTTP, cela consomme trop de ressources et crée trop de connexions.
Le bug est invisible en TP car il y a peu de requêtes simultanées.

**Q9.** Supprimez `Order.Descending` des deux méthodes, relancez, et regardez `/leaderboard/top/5`. Le résultat est-il visiblement faux ? Pourquoi ce bug est-il particulièrement dangereux ?

Oui, le classement sera inversé : les joueurs avec le moins de points apparaîtront en premier.
Ce bug est dangereux car le classement semble fonctionner, mais affiche des résultats incorrects sans provoquer d'erreur.

**Q12.** Combien de lignes avez-vous modifiées dans vos **endpoints** pour ajouter ce cache ? Pourquoi ? Quel choix de la séance 2 rend cela possible ?

Aucune ligne des endpoints n’a été modifiée, car le cache est géré dans CachedGameCatalog.
Le pattern Decorator permet d’ajouter le cache sans modifier les endpoints, grâce à l’interface IGameCatalog.

**Q13.** Relevez vos mesures. Le gain est-il spectaculaire ? Quelle conclusion honnête pouvez-vous en tirer, et dans quelles conditions un cache devient-il réellement décisif ?

"nombre": 11,
"millisecondes": 1.7997

"nombre": 11,
"millisecondes": 8.5127

Non, le gain n’est pas spectaculaire : les mesures varient entre 1,8 ms et 8,5 ms. Il faut faire plus de tests pour comparer correctement. Le cache devient surtout utile avec beaucoup de données et de requêtes simultanées.

**Q14.** Un administrateur corrige le titre d'un jeu directement dans MongoDB. Combien de temps les joueurs verront-ils l'ancien titre, et pourquoi ? Proposez deux stratégies pour réduire ce délai, avec leur inconvénient.

Les joueurs verront l’ancien titre pendant 60 secondes maximum, car le cache expire après ce délai.

Deux stratégies :
- Réduire le TTL : les données sont actualisées plus vite, mais MongoDB est davantage sollicité.
- Vider le cache après une modification : les données sont actualisées immédiatement, mais il faut gérer l’invalidation du cache.