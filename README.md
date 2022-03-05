# La Fraude Magecart - Le Code qui vous vole votre Carte de Crédit

Cette semaine je suis tombé sur une compagnie de Québec qui avait malheureusement du code malicieux sur leur site.
Ça parraît anondin, mais ce code servait a frauder des numéros de Cartes de crédit. Donc, ceci signifie que tous leurs clients qui ont fait une transaction sur leur site depuis que ce code est présent vont se faire frauder leur numéro de carte de crédit.
Le commerçant n'était pas au courant de l'attaque lorsque je leur ai divulgué et on prit action en moins de 24h pour enlever ce code.

# Mage-Quoi ? Magento ? MageCart ? Kossé ça ?
Pour ceux qui s'intéressent un peu a la sécurité informatique, vous savez que MageCart est une expression qui est utilisée parce que c'est groupe de hackers visent particulièrement la plateforme MAGEnto construite par Adobe. Cart vient du fait que c'est ceux qui ont une fonction 'Panier' sur le site web qui sont ciblés.
Au même titre que quelqu'un pourrait pirater une machine de paiement dans un centre d'achat, les pirates le font directement sur le site web du marchant, a son insu et n'ont même pas besoin de se déplacer.
Plusieurs groupes de 'MageCart' différents existent, qui ont tous des tactiques différentes et des infrastructures de contrôle propres a eux.
Mais ils ont tous un objectif commun : amasser le plus de numéros de cartes de crédit possible afin de les frauder. Ce sont littéralement des 'Card Skimmers'.
### Personne n'est a l'abri voici quelques victimes notables :
-La compagnie canadienne NewEgg vendeur d'équipement informatique Canadien 15 lignes de codes nécessaires : https://www.volexity.com/blog/2018/09/19/magecart-strikes-again-newegg/  (2018)

-La compagnie British Airways (380 000 victimes) : https://www.riskiq.com/blog/external-threat-management/magecart-british-airways-breach/ 
-L'entreprise de billetteries TicketMaster : https://www.riskiq.com/blog/external-threat-management/magecart-ticketmaster-breach/

-Fitness Depot qui se sont fait poursuivre en justice pour avoir tenté de cacher un tel cas.

#### Comment se protéger
Sur ce sujet, je ne peux pas insister assez que tous les commerçants doivent faire leurs mises-a-jours fournies par les fabricants de leur plateformes (Magento, Wordpress, Shopify et toutes les autres). Ceux qui ne font pas leurs mises-a-jours seront tôt ou tard ciblés par ce type d'attaque... C'est trop facile! Après un certain temps a ne pas avoir fait de mises-a-jour la question n'est pas de savoir si ça arrivera, mais plutôt "quand?". Évidemment, toutes les autres mesures d'hygiènes de Cybersécurité sont de mises aussi (avoir un antivirus, changer son mot de passe et en avoir un complexe et toute la ritournelle habituelle)

#### Pourquoi c'est autant intéressant ? La chasse est ouverte!
Ce qui rend la chose si intéressante n'est pas le fait qu'ils tentent de voler des numéros de cartes..... même si c'est vraiment un fléeau!!!
Ce qui est passionnant est dû au fait qu'ils tentent tant bien que mal de se cacher et que la chasse est ouverte a tous les jours sur internet pour ceux qui veulent les trouver!
(sans violence évidemment!!)
C'est pourquoi aujourd'hui j'ai décidé de partager ma dernière découverte qui était une des meilleures tentatives que j'ai vue pour obfusquer du JavaScript.

#### Modus Operandi et chasse a l'ours
Tout d'abord, si par malheur vous tombez sur un site infecté par ce code, vous ne le saurez pas au premier coup d'oeil.
Les sites ressemblent en tout point a un site de commerce en ligne normal et le code est "caché" afin que le client ne se rendent compte de rien.

Le code est vraiment fou a comprendre a désobfusquer!! En voici la preuve :

## Étape 1 : Mettre le code en bas du code source de la page et l'encoder
Le site a l'air normal :

![cart](https://user-images.githubusercontent.com/16509773/156863112-a26c5fe2-df03-4fc5-8b70-1d27606b70da.jpg)


Cependant, en vérifiant le code, a la fin du code source on y trouve ceci :

![dev](https://user-images.githubusercontent.com/16509773/156863217-242255bd-7cc8-400f-b3cf-c24d1eae9c01.jpg)

Je suis loin d'être un développeur, mais je pense que tous ceux qui ont déja lu une ligne s'entendent pour dire qu'aucun développeur saint d'esprit n'aurait fait un code comme ça sans bonne raison. 

C'est ce qu'on appelle l'obfuscation, l'art de rendre un code illisible afin de cacher ses fonctions!

Avec un peu d'acharnement et l'aide d'un collègue on a découvert que celui-ci est l'une des versions les plus avancées de MageCart et probablement un nouveau groupe en plein émergence (je continuerai sur le sujet plus tard).

Tout d'abord, il faut décoder tout ce bloc en Base64, ceci leur permet d'éviter certaines détections, mais aussi de faire disparaître des mots comprommetant du genre 'CC_Value'.
Voici donc ce que ça donne après une première tentative :

![the-heck](https://user-images.githubusercontent.com/16509773/156863430-16218217-35ba-4e7a-aa2d-8733ff040e9b.jpg)
![the-heck2](https://user-images.githubusercontent.com/16509773/156863435-392317a3-804f-4d4c-b256-2cbdaef0540e.jpg)
C'est déja mieux... mais j'aurais tendance a dire que "c'est pas y'iable" encore... Par contre on voit déja que des fonctions 'lisibles sont en place'.
On a encore du CharCode est des fonctions très bizarres. En tentant de décoder ceux-ci je ne suis pas arrivé a mes fins...

Et c'est ici que ça devient vraiment intéressant!! 
En tentant de décoder ce qui reste, on se rend compte que le code 's'auto-utilise' pour décoder le reste des fonctions (merci a mon collègue de m'avoir fait découvrir ça!).
Le code utilise des petites parties de son propre code pour se désobfusquer, ce qui nécessite l'utilisation d'un debugger étape par étape pour comprendre ce qui se passe.
![step-by-step](https://user-images.githubusercontent.com/16509773/156865048-3fb37b25-a33f-415f-bce5-26c6ae8bdfa4.jpg)

A voir ces informations, on peut donc déduire qu'il utilise l'information remplie sur un formulaire de la page et qui la soumets sous forme de Requête POST en lettres minuscules lorsque le client clique sur la souris (donc avant de le soumettre au vrai marchant)!!!

Auttre fait intéressant, ce script n'est toujours pas détecté par la plupart des antivirus sur le marché, ni le site vers lequel il est envoyé!


De plus le certificat "Let's Encrypt' du site malicieux a été émis le 1er Février 2022, 
Le Domaine est protégé par un PrivacyService (on ne sait pas qui est le propriétaire), 
Le nom de domaine a été mis-a-jour le 1er Mars dernier.
Le site est aussi hébergé loin loin du Québec..., il n'y a aucune raison que le commerçant fasse affaire dans cette partie du monde.


En espérant que ça change bientôt et que vous avez aimé mon blog!

🔥🔥🔥

