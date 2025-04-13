# Chasser les failles, protéger le web : dans les coulisses du bug bounty

Bonjour, je m'appelle Benjamin Mamoud, étudiant à La Plateforme. Passionné de cybersécurité, j’ai découvert ce domaine assez tôt. Fasciné par les nouvelles technologies et leur fonctionnement, je me suis naturellement tourné vers le pentest, en me formant sur des plateformes de CTF comme Root-Me. Depuis, je ne cesse de progresser et d’en apprendre davantage chaque jour.

## Et si je vous disais qu’on pouvait être payé pour pirater des sites… légalement ?

C’est là qu’intervient le bug bounty, ou "prime à la faille". Ce programme, mis en place par des entreprises, consiste à inviter des hackers éthiques à tester leurs systèmes à la recherche de vulnérabilités. Plutôt que d’attendre qu’un cybercriminel exploite une faille, elles préfèrent qu’un chercheur la découvre en amont et le récompensent pour cela.
Une approche intelligente, collaborative, et surtout essentielle pour renforcer la sécurité du web, tout en valorisant les compétences des passionnés de cybersécurité.

## Les vulnérabilités

Pour performer en Bug Bounty il est obligatoire de connaitre la majorité des vulnérabilités qui s'appliquent aux applications, par exemple le OWASP Top 10 qui regroupe les 10 types de vulnérabilités les plus répendues sur les sites web, c'est sur ce genre de vulnérabilités que je m'attarderai ici puisque je me spécialise particulièrement en pentest sur les applications web.

## Ma découverte du monde des Bug Bountys

Cela s'est passé en janvier 2020, fatigué par une nuit trop courte, j’ai opté pour la facilité : commander à manger sur Uber Eats. Je me hâte donc sur le site et on me demande de me connecter, étonnamment je ne reçois pas le code après avoir entré mon numéro de téléphone. Curieux, je décide d'ouvrir les Devtools de chrome (qui me permettent de voir les requêtes envoyées aux serveurs de Uber.

Je commence donc a m'attarder sur la requête qui est censée m'envoyer ce fameux code par sms, je ne vois rien d'anormal, une réponse classique retournant une sorte d'ID de compte (inAuthSessionID) et beaucoup d'autre infos. Avec les Devtools toujours ouverts je suis donc allé sur la page de support uber et on me demande de me connecter ce que je ne pouvais pas faire mais je remarque que lorsque je clique sur me connecter une redirection est faite en passant par une requête qui me génère une session pour la connexion, je décide donc sur un coup de tête de changer un cookie, le cookie sessionid et j'y met la valeur que j'avais précédemment dans le inAuthSessionID.

Je reçois une erreur a laquelle je m'attendais, mais lorsque que je reviens après sur le site me voila connecté. Je me pose donc pleins de questions et décide de répéter le processus pour tester sur un autre compte et je réalise que je peux me connecter sur n'importe quel compte sans même recevoir de sms ou entrer de mot de passe.

Mon premier reflexe a été de vouloir comprendre comment cela pouvais marcher et pour cela nous devons nous pencher sur le protocole Oauth.
OAuth est un protocole d'autorisation qui permet à une application d'accéder à des ressources d’un utilisateur sans avoir besoin de son mot de passe (en passant par un code reçu par sms sur Uber). L’utilisateur approuve un accès limité (défini par des "scopes") à ses données via un jeton d’accès.

Normalement, un jeton d’accès n’autorise que ce que l’utilisateur a validé. Mais le service OAuth de Uber ne vérifiais pas correctement l’étendue autorisée, je pouvais donc abuser du système pour obtenir plus de permissions que prévu avec un jeton censé me donner seulement accès a la page de saisie du code sms
J'en ai donc parlé a un amis présent dans le domaine de la cyber sécurité qui me fait découvrir les Bug Bountys et m'envoie un lien vers celui de Uber.
Je signale la faille et quelques semaines plus tard j'apprend que Uber a choisi de me payer pour cette vulnérabilité.
Après cet évènement je me suis décidé a me lancer dans le signalement de vulnérabilités !

## Cairn.info - Comment une simple requete sql sur un sous domaine peut conduire au control total de l'infrastructure






Des failles, mais surtout des leçons






