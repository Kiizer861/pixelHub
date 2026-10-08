COLIN Noé 
BUT3 TD2-APP

Réponses aux questions :

1. 
// 1. Combien de jeux dans la collection ?
db.jeux.countDocuments()
10

2.
// 2. Afficher un document en entier
db.jeux.findOne()
{
  _id: ObjectId('6ac744463faab40a8b1e2332'),
  titre: 'Counter-Strike 2',
  genre: 'FPS',
  note: 4.5,
  anneeSortie: 2023,
  plateformes: [ 'PC' ],
  tags: [ 'compétitif', 'tir', 'équipe' ],
  joueursParEquipe: 5,
  cartes: [ 'Dust II', 'Mirage', 'Inferno', 'Nuke' ],
  classementCompetitif: true
}

**Q1.** Vous retrouvez le champ `_id`, déjà aperçu au TP 1 (étape 6.2), que personne n'a écrit dans `games.json`. Cette fois, intéressez-vous à **qui l'a fabriqué** : le serveur MongoDB, ou l'outil qui a envoyé les documents (`mongoimport`) ? Comparez avec l'`Id` 4 de *Pixel* au TP 1 (Q8) : qui l'avait choisi, et à quel moment le connaissiez-vous ?

Le champ _id est fabriqué automatiquement par MongoDB lors de l’insertion du document via mongoimport.

Pour Pixel, l’Id 4 était choisi par PostgreSQL, et on connaissait sa valeur après l’enregistrement en base.

**Q2.** À l'exercice 5, `plateformes` est un tableau et vous avez écrit `{ plateformes: "Switch" }`, sans opérateur particulier. Comment auriez-vous fait la même chose en SQL, avec un modèle relationnel normalisé ?

En SQL, avec un modèle normalisé, il faudrait une table Plateformes reliée aux jeux par une table intermédiaire :

SELECT j.titre
FROM Jeux j
INNER JOIN JeuxPlateformes jp ON j.id = jp.jeu_id
INNER JOIN Plateformes p ON jp.plateforme_id = p.id
WHERE p.nom = 'Switch';

**Q3.** Vous tapez `--authenticationDatabase admin` depuis le TP 1 sans qu'on vous ait dit pourquoi. Pourquoi faut-il l'ajouter aux commandes `mongoimport` et `mongosh` ? Où se trouve le compte `pixelhub` ?

Il faut ajouter --authenticationDatabase admin car le compte pixelhub est créé dans la base admin. Cela permet à MongoDB de chercher l'utilisateur au bon endroit lors de la connexion.

**Q4.** Personne ne vous a arrêté. Est-ce une bonne nouvelle ou un problème ? Argumentez en imaginant un projet à quatre développeurs, six mois plus tard.

C’est pratique car MongoDB est flexible, mais c’est aussi un problème pour plusieurs développeurs, chacun pourrait ajouter des champs différents et, après un certain temps les documents seraient difficiles à comprendre et à maintenir.

**Q5.** Le résultat de l'exercice 9 vous surprend-il ? Que s'est-il passé exactement, et pourquoi n'y a-t-il eu **aucun message d'erreur** ?

MongoDB cherche dans la collection Jeux au lieu de jeux. Comme cette collection n'existe pas, MongoDB en crée une vide automatiquement, donc aucune erreur n'est affichée.

**Q6.** L'exercice 16 échoue. Recopiez le message d'erreur, et expliquez pourquoi MongoDB refuse. Quelle commande utiliseriez-vous si vous vouliez **réellement** remplacer le document en entier ?

MongoInvalidArgumentError: Update document requires atomic operators
MongoDB refuse car sans $set, on tente de remplacer le document entier avec { note: 4.4 }

**Q7.** À l'exercice 12, vous avez ajouté un champ `nbVotes` à un seul document, alors que les dix autres n'en ont pas. Quelle commande SQL aurait été nécessaire pour faire l'équivalent en relationnel, et quelle en aurait été la conséquence sur les autres lignes ?

En SQL, il aurait fallu utiliser ALTER TABLE pour ajouter la colonne nbVotes. Cette colonne aurait alors existé pour toutes les lignes, avec une valeur NULL pour les autres jeux

**Q8.** Lisez `IGameCatalog`. Y a-t-il un seul endroit dans cette interface où le mot « Mongo » apparaît ? Pourquoi est-ce important, à votre avis ?

La section sur IGameCatalog montre que cette interface ne contient aucune référence à Mongo.
C’est important car l’interface reste indépendante de la technologie de stockage : l’API utilise IGameCatalog, et l’implémentation peut être PostgreSQL, MongoDB, etc...

**Q9.** Quelle exception obtenez-vous, et sur quel champ ? Expliquez la cause : qu'est-ce que le driver a essayé de faire, et pourquoi n'a-t-il pas su ?

L’exception est une System.FormatException sur le champ joueursParEquipe. Le driver MongoDB essaie de convertir le document MongoDB en objet Game, mais ce champ n’existe pas dans la classe Game, donc il ne sait pas où le stocker

**Q10.** `[BsonIgnoreExtraElements]` fait disparaître l'erreur, mais **au prix de quoi** ? Que devient `joueursParEquipe` quand vous appelez `/games` ?

BsonIgnoreExtraElements ignore les champs présents dans MongoDB mais absents de la classe Game.
Donc joueursParEquipe n’est pas récupéré par l’API et est perdu.

**Q11.** Comparez les trois approches possibles pour gérer un schéma variable en C# : `[BsonIgnoreExtraElements]`, `[BsonExtraElements]`, et une hiérarchie de classes (`FpsGame : Game`, etc.). Quel est l'avantage et l'inconvénient de chacune ?

BsonIgnoreExtraElements : simple et évite les erreurs, mais les champs supplémentaires comme joueursParEquipe sont perdus.
BsonExtraElements : conserve les champs supplémentaires, mais leur gestion est moins typée et moins pratique.
Hiérarchie de classes : permet d’avoir des modèles fortement typés et adaptés à chaque type de jeu, mais c’est plus complexe à maintenir si le schéma évolue souvent.


**Q12.** Écrivez la requête SQL qui produirait le même résultat sur une table `jeux(titre, genre, note)`. Comparez les deux. Est-ce ici que MongoDB apporte quelque chose par rapport à PostgreSQL ? Si non, où l'apport se situe-t-il dans ce TP ?

SELECT genre, COUNT(*) AS nombre, AVG(note) AS moyenne
FROM jeux
GROUP BY genre;

Les deux requêtes regroupent les jeux par genre et calculent le nombre de jeux et leur note moyenne. MongoDB n’apporte pas vraiment d’avantage ici : l’intérêt de MongoDB dans ce TP se situe surtout dans son schéma flexible, qui permet d’avoir des champs différents selon les documents.

**Q13.** Les requêtes d et e ne renvoient pas la même chose. Laquelle a raison ? Que demande exactement la requête d, et pourquoi trouve-t-elle *Stardew Valley* alors que Krayz lui a mis 4 ?

C'est la requête e qui a raison. 
La requête d cherche séparément un avis avec auteur = "Krayz" et un avis avec note = 5. Ces deux conditions peuvent donc correspondre à deux avis différents, ce qui explique pourquoi elle trouve Stardew Valley alors que Krayz lui a mis 4.

**Q14.** Combien d'octets coûte un avis, en moyenne ? Un document MongoDB ne peut pas dépasser 16 Mo (16 777 216 octets) : à partir de combien d'avis environ *Stardew Valley* ne pourrait-il plus en accueillir ? Est-ce un risque réel pour un jeu populaire ? Rapprochez votre réponse des critères du cours.

Un avis prend peu de place, donc il faudrait beaucoup d’avis pour atteindre 16 Mo. Ce n’est pas un risque immédiat, mais pour un jeu très populaire, il vaut mieux mettre les avis dans une collection séparée.