# Teaching-HEIGVD-SEN-2022-Laboratoire-BITB


## Une petite note sur l'éthique

Il n'est absolument pas acceptable d'attaquer quelqu'un pour quelque raison que ce soit.

L'utilisation de ces outils à des fins autres que votre propre éducation et formation sans autorisation est strictement interdite par les politiques de ce cours et de l'école, ainsi que par les lois.

Le but de cet exercice est de vous permettre de vous familiariser avec les outils et comment ils peuvent être utilisés dans le contexte professionnel d'un pentest. Ça vous permettra aussi de comprendre les tactiques de l'adversaire afin de pouvoir les contrer par le biais de la politique, de l'éducation et de la formation.


## Introduction

Selon le [Internet Crime Report 2020](https://www.ic3.gov/Media/PDF/AnnualReport/2020_IC3Report.pdf) rendu publique par le Centre de plaintes pour les crimes sur Internet du FBI, le phishing est de loin l'attaque la plus courante des cybercriminels. En effet, on enregistre plus de deux fois plus d'incidents de phishing que tout autre type de criminalité informatique.

La raison est simple : une attaque par phishing ne nécessite pas forcément des connaissances techniques très sophistiquées ni un grand nombre de ressources.

Depuis l’apparition du terme « phishing » en 1995, la technique a évolué avec l’Internet. Les premières tentatives de phishing visaient le vol d’information et d’identité. Pourtant, les efforts se sont rapidement concentrés autour du vol d’argent. Le premier incident contre un système de paiement en ligne a été enregistré en 2001 et la première attaque contre une banque en 2003. Les choses n’ont pas beaucoup changé après 20 ans. 

Il est pourtant très facile d’identifier un site de phishing. Il suffit de regarder la barre d’adresse du navigateur ! (oui… il y a de méthodes qui utilisent un DNS pour contourner ces « détails » mais ça implique un travail considérable et un accès souvent local à la cible). Un coup d’œil à l’adresse du site suffit normalement pour dévoiler une attaque… 

Ce n’est plus le cas… 

En mars 2022, une nouvelle attaque baptisée [Browser in the Browser](https://mrd0x.com/browser-in-the-browser-phishing-attack/) propose d’utiliser JavaScript pour construire une fenêtre d’authentification capable de spoofer pas seulement les éléments visuels d’un site légitime, mais aussi le nom du domaine d'un fournisseur d'autnetification. Cette interface prend la forme d'une fenêtre émergente/flottante.

Il est clair que le site qui contient/lance cette fenêtre aura une adresse facilement identifiable comme étant pas légitime. Il faut donc comprendre le scénario d'utilisation de cet outil : un attaquant peut créer un site/service qui attire les victimes avec la promesse, par exemple, d'un cadeau. La seule condition pour obtenir le cadeau c'est la création d'un compte sur le site pour faciliter la livraison dudit cadeau. Pour cela, et pour faciliter la vie de la personne qui participe, le site met à disposition l'authentification de Google, ou Microsoft, ou Apple ou Facebook... ce qui est courant de nos jours. Au lieu de créer un compte de zéro pour chaque nouveau service, on utilise juste un fournisseur d'authentification. Pas besoin donc d'un username ou mot de passe, entre autres. Le site propose donc des icônes pour les différents services d'authentification disponibles. La cible clique sur l'icône du service authentification préféré et une fenêtre émergente demande les credentials de Google ou Microsoft ou Apple ou Facebook, etc... Sauf que cette fenêtre est un clone controlé par un attaquant.

Le résultat, c’est quelque chose comme ceci :

![BITB]( https://mrd0x.com/demo-c2b899d2175d71fb45e3f86a8ba80644.gif)

La fenêtre émergente/flottante que vous voyez sur l’animation est un clone ; ce n’est pas une vraie authentification Microsoft. Vous remarquerez également l’URL de la fenêtre, qui semble tout à fait légitime. Le domaine ```login.microsoftonline.com``` est bien le domaine réel appartenant à Microsoft pour l’authentification.

## Que faut-il faire ?

Voici les activités à réaliser dans ce laboratoire. Vous devez :

- Installer un serveur Web pour héberger votre attaque. Nous vous proposons une solution basée sur ```nginx``` et ```Docker```
- Télécharger les sources de l’attaque [BITB](https://github.com/mrd0x/BITB)
- Comprendre le fonctionnement et les différents paramètres 
- Implémenter votre attaque avec le site/système d’authentification de votre choix.

Le "rapport" de ce labo est très simple : **Pour chaque tâche, faites des captures d'écran de vos activités et répondez les éventuelles questions**.

## Docker Web Server

Nous allons utiliser une solution simple mais efficace utilisant l’[image Docker officielle du serveur nginx]( https://hub.docker.com/_/nginx). 

Vous pouvez lancer votre serveur à l’aide de la commande suivante :

```bash
docker run --name nginx -v /votre/site:/usr/share/nginx/html -p 8080:80 -d nginx
```

Ceci crée un container basé sur l’image ```nginx``` qui utilise le contenu du répertoire ```/votre/site``` comme homepage. Le port pour accéder au site depuis votre navigateur est le ```8080```. Vous pouvez évidement le modifier s'il est déjà utilisé par une autre application sur votre machine.


## Browser In The Browser Attack

Dans un répertoire de votre choix, clonez [le projet BITB disponible sur github](https://github.com/mrd0x/BITB). Par exemple :

```bash
git clone https://github.com/mrd0x/BITB.git
```
Le projet contient une série de répertoires qui génèrent des versions de fenêtres differentes en fonction de l’OS de la cible (windows ou macos). Des versions « dark » et « light » sont aussi proposées pour chaque OS. Il est possible de déterminer quelle version doit être montrée à la victime utilisant JavaScript. Vous pouvez même déterminer si la cible utilise dark ou light mode comme expliqué [ici](https://stackoverflow.com/questions/50840168/how-to-detect-if-the-os-is-in-dark-mode-in-browsers).

Dans mon cas (macos, dark), je lance mon container Docker depuis le répertoire ```BITB``` de la manière suivante :

```bash
docker run --name bitb-nginx -v $PWD/MacOS-Chrome-DarkMode:/usr/share/nginx/html -p 8080:80 -d nginx
```
Voici ce que j’obtiens comme résultat. Cette fenêtre est "flottante"; je peux donc la déplacer avec ma souris :

![Fenêtre basique BITB](images/bitb_basic.png)

---
#### Livrable : Capture d'écran de votre première fenêtre de BITB

![image-20220427231410214](images/image-20220427231410214.png)

---

Cette fenêtre est une manière assez utile de comprendre tout de suite les paramètres à configurer. Ces éléments sont facilement identifiables puisqu'ils prennent la forme ```XX-ELEMENT-A-CONFIGURER-XX```. Chacun de ces éléments correspond à une variable dans le fichier ```index.html``` de chaque répertoire (pour les différentes versions). Les variables à éditer sont donc les suivantes :

- XX-TITLE-XX - Le titre de la fenêtre
- XX-DOMAIN-NAME-XX - Le nom de domaine spoofé 
- XX-DOMAIN-PATH-XX - Le chemin (path) pour le domaine
- XX-PHISHING-LINK-XX - Le lien de phishing qui sera inséré dans votre fenêtre

Le fichier ```logo.svg``` dans le même répertoire vous permet de spoofer le logo du domaine. Il n'est pas visible sur la version Mac mais sur la version Windows uniquement. Vous pouvez changer le fichier ou éditer son nom dans ```index.html```.

Je procède à modifier les 3 premiers variables de la version Windows light. Je trouve aussi rapidement un logo correspondant à Outlook et voici le résultat :

![Auth heig](images/outlook_heig.png)

Vous pouvez voir que la fenêtre affiche maintenant l'adresse du service Outlook de l'école. Le message "Coming soon" vient tout simplement du fait que j'ai juste utilisé un fichier html avec le message en question. Je n'ai pas préparé un site de phishing... ceci est votre travail !

## Phishing Link

On arrive à la partie la moins triviale de l'exercice : vous devez donc implémenter votre attaque avec le site/système d’authentification de votre choix. En d'autres termes, vous allez produire votre "phishing link" pour la dernière variable du fichier ```ìndex.html```. Vous pouvez tenter de faire l'expérience avec le webmail de l'école, mais ça peut être plus intéressant avec un service qui utilise vraiment une fenêtre flottante/émergente (un exemple c'est Twitter). Vous aurez très probablement besoin d'un outil pour "aspirer"/cloner la fenêtre d'authentification que vous allez représenter ou vous pouvez essayer de la reproduire manuellement, ce qui sera plus ou moins facile à faire en fonction de l'objectif choisi. Il se peut que le site aspiré ne soit pas utilisable directement sans un peu de travail. A vous de voir. 

Votre travail pour ce TP sera limité à afficher la fenêtre avec le formulaire. La fonctionnalité pour récolter les credentials ne doit pas être implémentée. 

Evidement, ce travail peut être combiné avec des outils comme [Gophish](https://github.com/gophish/gophish/releases), ce qui risque de produire des résultats redoutables.

---
#### Livrable : Capture d'écran du site légitime que vous avez cloné.

![image-20220428011811091](images/image-20220428011811091.png)

---

#### Livrable : Capture d'écran de votre version.

![image-20220428013526650](images/image-20220428013526650.png)

![](images/bitb-demo.gif)

---

#### Question : quels sont les valeurs que vous avez attribués aux différentes variables ?

| Variable            | Contenu                                                      | Explication                                                  |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| XX-TITLE-XX         | `Sign-in - Google accounts`                                  | C'est le contenu de la balise `title`, qui définit le titre de l'onglet de la fausse fenêtre |
| XX-DOMAIN-NAME-XX   | `https://accounts.google.com`                                | Le nom de domaine de la fenêtre de login                     |
| XX-DOMAIN-PATH-XX   | `/o/oauth2/auth/identifier?redirect_uri=storagerelay%3A%2F%2Fhttps`... | La fin de l'URL du login, qui est grisée dans la fausse fenêtre. <br />J'ai pris soin de supprimer les tokens de la requête que j'ai prise pour générer le clone de google. |
| logo.svg            | Remplacé par le logo de Google                               | Afin de donner de plus de crédibilité au BTIB                |
| XX-PHISHING-LINK-XX | google-login.html                                            | Le chemin du fichier de faux login Google. À noter que les liens vers des domaines externes sont bloqués par la sécurité du navigateur (je ne me suis pas renseigné plus mais c'est le cas sous Firefox). |



---

#### Question : Y-a-t'il des différences remarquables entre le site original et votre version ? Si oui, lesquelles ?

Au niveau purement visuel, la seule différence est la bordure de la fenêtre BTIB. Je n'avais pas l'envie de jouer avec le CSS pour répliquer parfaitement Windows.
Mais surtout, c'est une barre de titre différente car elle est dépendante de l'OS et je suis sous Linux, qui n'a pas de template fourni par l'auteur. Pour corriger cela, il faudrait créer un template générique pour Linux (Ubuntu avec Gnome, pour rester dans les plus utilisés) et faire un script JS plus avancé qui détecte le navigateur de la victime (avec son User-Agent par exemple) et modifier la décoration de la fenêtre dynamiquement. 

Au niveau des fonctionnalités on peut évidemment noter que le formulaire de login ne fonctionne pas et que le bouton de connexion "Apple" envoie vers un lien qui n'est pas valide.

Le formulaire BTIB ne fonctionne par contre pas vraiment et on ne peut pas entrer de mot de passe après avoir validé le nom d'utilisateur. Cela serait faisable si on modifiait le code de la page.

---
#### Question : quel outil ou méthode avez-vous employé pour cloner le formulaire qui s'affiche sur votre fenêtre ? Comment avez-vous procédé ? Donnez-nous le plus grand nombre de détails possibles !

**Clonage de Reddit**

J'ai d'abord essayé un petit outil du nom de Goclone, que j'ai trouvé sur GitHub: https://github.com/imthaghost/goclone

Il est rapide et fonctionne bien pour les sites simples, il a un problème: il envoit trop de requêtes et se fait bloquer par les protections anti DDOS et anti bot des sites (Cloudflare par exemple). Après avoir passé du temps à chercher un site qui soit compatible avec cet outil, j'ai abandonné et trouvé un autre outil qui faisait parfaitement l'affaire.

L'outil en question est disponible en ligne à cette adresse : https://tool.minhclear.net/clonetemplate/

**Mise en place du BTIB**

Pour commencer, j'ai enlevé du code les appels à l'API reddit et les scripts qui n'étaient pas utiles pour la fausse page.

J'ai ensuite intégré les parties utiles du template dans la page clonée, c'est-à-dire le link CSS, la balise HTML de la fenêtre "window" ainsi que l'appel du script du BTIB.

Puis j'ai modifié les variables comme expliqué plus haut.

J'ai ensuite dû faire quelques modifications pour avoir une fenêtre BTIB propre et fonctionnelle:

1. Cacher la fenêtre au chargement de la page en lui donnant par défaut le style `display:none`

2. Afficher la fenêtre lorsqu'on appuie sur le bouton de connexion via Google. Dans la div, j'ai ajouté l'attribut `onclick` avec du Javascript qui modifie le style via le DOM.

   ![image-20220428022237181](images/image-20220428022237181.png)

3. Le contenu de la page commençait après la fenêtre BTIB, et non au sommet de la page. J'ai donc ajouté le CSS nécessaire: `position:absolute;top:0`

4. Lorsque la fenêtre était affichée, elle se mélangeait avec les composants de la page normale. J'ai mis sa position relative et permis la superposition avec le  `z-index` qui permet de définir des priorités sur les couches d'éléments.

5. Modifier l'emplacement de départ de la fenêtre

**Clonage du formulaire Google**

Pour cloner le formulaire de connexion Google, les outils que j'avais utilisé auparavant ne fonctionnaient pas avec l'URL de la page, j'ai donc utilisé la manière simple: Ctrl+S sur le navigateur.

Cette solution a fonctionné étonamment bien. J'ai donc pu sauvegarder la page, qui n'a pas de dépendance JS ni CSS et l'intégrer à la fenêtre BTIB.

Le résultat est identique à la page Google normale, mais ne fonctionne pas car il y a normalement un script qui va récupérer des tokens et faire des vérifications avant de pouvoir rentrer son mot de passe.

---
#### Pour finir, partagez avec nous vos conclusions.

Cette attaque Browser In The Browser est très puissante. Si elle est suffisamment bien implémentée, il est difficile de s'apercevoir qu'on est face à une fausse fenêtre de browser.

Je pense que les templates de l'auteur de l'attaque ne sont pas encore assez évolués. Par exemple, ma barre de titre "Windows"avait des soucis de CSS, ou avait des curseurs étranges. Ce sont des détails, mais des détails qui font perdre crédibilité face à des victimes attentives.

Cependant, un attaquant suffisamment motivé pourra forcément améliorer cela et ajouter par exemple des fonctionnalités pour s'adapter au browser et à l'OS de la victime, ou encore ajouter les fonctionnalités manquantes, comme la possibilité de redimensionner la fenêtre.

---

## Echeance

Le 12 mai 2022 à 10h25
