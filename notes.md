COLIN Noé 
BUT3 TD2-APP

Réponses aux questions :


**Q1.** Le fichier déclare quatre services alors que le TP d'aujourd'hui n'en utilise qu'un. Pourquoi, à votre avis ?

Pour que chaque service soit indépendant.

**Q2.** Trois services ont une section `volumes`, un seul n'en a pas. Lequel, et qu'est-ce que ça implique pour ses données ?

C'est redis, quand le service redis est supprimé, les données ne sont pas sauvegardées donc les données sont perdues.

**Q3.** Que signifie la ligne `"15432:5432"` ? Les deux nombres ne désignent pas la même chose.

15432 : le port de la machine hôte
5432 : le port utilisé par PostgreSQL dans le conteneur.

**Q4.** Les mots de passe sont écrits en clair dans le fichier. Est-ce acceptable ici ? Le serait-ce sur un serveur de production ?

Oui, c’est acceptable ici pour un exercice ou un environnement de test.
En production, non : il faut utiliser des secrets ou des variables d’environnement sécurisées pour éviter d’exposer les mots de passe.

**Q5.** Les commandes ci-dessus utilisent `docker compose exec`. Qu'est-ce que ça fait exactement ? Où s'exécute la commande `redis-cli` ?

docker compose exec exécute une commande à l’intérieur d’un conteneur en cours d’exécution.

redis-cli s’exécute dans le conteneur Redis.

**Q6.** La base de données tourne dans un conteneur Docker, et la chaîne de connexion dit `Host=localhost`. Pourquoi est-ce que ça fonctionne ? (Indice : relisez votre réponse à la Q3.)

Ça fonctionne car localhost désigne la machine hôte, où le port 15432 est redirigé vers PostgreSQL sur le port 5432 du conteneur.

Port : 127.0.0.1:15432

**Q7.** Arrêtez l'application (Ctrl+C), relancez-la. Les trois joueurs sont-ils dupliqués ? Pourquoi ?

Non, les trois joueurs ne sont pas dupliqués, car les données sont stockées dans PostgreSQL, qui conserve les données même lorsque l’application est arrêtée puis relancée.

**Q8.** Dans votre code, que vaut `player.Id` *avant* l'appel à `SaveChangesAsync()` ? Et après ? Qui a choisi la valeur 4 : votre code, EF Core ou PostgreSQL ?

Avant SaveChangesAsync(), player.Id vaut 0. Après, il vaut 4, car c’est PostgreSQL qui génère automatiquement l’ID, avec l’aide d’EF Core.

**Q9.** Que valent les soldes après le `ROLLBACK` ? Imaginez maintenant que le serveur s'éteigne entre les deux `UPDATE`, *sans* transaction : dans quel état serait la base, et qui y perdrait ?

Après le ROLLBACK, Nova revient à 1200 pièces et Krayz à 350 pièces.
Sans transaction, si le serveur s'éteint entre les deux UPDATE, Nova aurait perdu 100 pièces sans que Krayz les reçoive : les 100 pièces seraient donc perdues.


**Q10.** Pendant la transaction de A, quel solde de Nova voyaient B et l'API ? Pourquoi l'`UPDATE` de B a-t-il dû attendre ? Imaginez que A et B soient deux achats lancés au même instant par Nova sur deux téléphones : que pourrait-il se passer si la base ne bloquait pas B ?


B et l’API voyaient 1200 pièces, car la modification de A n’était pas encore validée (COMMIT).
L’UPDATE de B doit attendre car PostgreSQL verrouille la ligne modifiée ; sans ce blocage, deux achats simultanés pourraient modifier le même solde et provoquer un solde incorrect.

**Q11.** Qui a refusé l'achat d'Ombre : votre code C# ou la base ? Pourquoi le `SELECT` suivant a-t-il échoué lui aussi ? Après le `ROLLBACK`, la contrainte existe-t-elle toujours, et qu'est-ce que ça vous apprend sur PostgreSQL ?

C’est PostgreSQL qui a refusé l’achat car le solde aurait été négatif.
Après l’erreur, la transaction est bloquée jusqu’au ROLLBACK, qui supprime aussi la contrainte ajoutée.

**Q12.** Que renvoie `TTL temporaire` au fil des secondes, puis une fois les 10 secondes écoulées ? Et `GET temporaire` ? Citez une donnée de PixelHub qui gagnerait à disparaître toute seule.

TTL temporaire renvoie le nombre de secondes restantes, puis une fois les 10 secondes écoulées, il renvoie -2 car la clé n'existe plus. GET temporaire renvoie alors (nil).

**Q13.** `INCR` lit, ajoute 1 et réécrit en **une seule** commande. Pourquoi est-ce plus sûr que de faire un `GET`, d'ajouter 1 dans le code C#, puis un `SET`, si 200 joueurs déclenchent le compteur au même moment ? (Pensez à ce que vous avez vu en 5.2.)

INCR est plus sûr car il fait l'ajout en une seule fois, donc aucun joueur ne peut écraser le compteur d'un autre.

**Q14.** Les deux jeux sont dans la même collection, mais n'ont pas les mêmes champs (`plateformes` est une liste, `multijoueur` un objet imbriqué). Aurait-on pu ranger les deux dans une même table PostgreSQL ? À quel prix ? Rapprochez votre réponse de l'exercice du catalogue de jeux (section 2 du cours `CM1_etudiants_Panorama.md`).

Oui. On aurait pu, mais il aurait fallu adapter la table PostgreSQL avec plusieurs colonnes, des valeurs NULL ou des tables supplémentaires pour gérer les listes et objets imbriqués.

**Q15.** La requête « ami d'un ami » se lit presque comme un dessin. Comment l'écririez-vous en SQL, avec une table `Amities(joueur_id, ami_id)` ? Combien de jointures faudrait-il pour « ami d'un ami d'un ami » ?

SELECT a2.ami_id
FROM Amities a1
INNER JOIN Amities a2 ON a1.ami_id = a2.joueur_id
WHERE a1.joueur_id = 1;
Pour « ami d'un ami d'un ami », il faudrait 3 jointures.

**Q16.** Pourquoi `DELETE` seul a-t-il été refusé ? Que fait `DETACH` de plus ?

DELETE est refusé car le nœud possède encore des relations.
DETACH DELETE supprime le nœud et ses relations en même temps.


**Q17.** La clé `survivant` a-t-elle survécu au `stop` ? Au `down` ? Et les joueurs de PostgreSQL ? Expliquez la différence avec le mot **volume**.

Après stop : la clé survivant est toujours là, car le conteneur est seulement arrêté.
Après down : la clé Redis disparaît, car Redis n'a pas de volume.
PostgreSQL : les joueurs restent présents après down, car PostgreSQL utilise un volume.
Un volume permet de conserver les données même si le conteneur est supprimé.