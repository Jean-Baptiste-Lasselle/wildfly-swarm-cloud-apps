
# wildfly-swarm-cloud-apps

Un repo de versionning pour deux codes sources d'une seule et même application, implémentée avec JAXRS et Ribbon https://github.com/Netflix/ribbon

### Description de l'application 

L'application fournira une API REST exposant les opération `CRUD` sur une catégorie d'objets que j'appellerai `Machine`(s).

L'obecjtif de cette application, est double : 

* produire une application permettant d'illustrer les principes des micros-services, et du "reactive programming"
* produire une application réellement utilisée, pour permettre la gestion d'un inventaire Hardaware. L'application fera plus tard l'objet de plsu amples développement, notamment pour permettre la gestion d'une étiquetage du matériel par QR-code, ou ce genre de truc scannables avec une appli mobile. Ce composant deviendra une partie de kytes.io

L'objetif no. 1, est d'implementer l'applciation, avec les

### Cahier des charges technique : Architecture

Le client sera en Angular 5, et sera toujours packagé dans un conteneur à part, avec un serveur HTTP le plus léger possible (essayer Undertow dans un conteneur?), avec confguration CORS.

L'ensemble des traitments serveur sera implémenté en Java, sous 3 formes : 

* Les 3 formes seront [**"swagger.io-compatibles"**](https://swagger.io).
* Une forme ayant recours à des objets mutables "à l'ancienne", avec un Rest Controller JAXRS classique, modifiant les entrées de la BDD avec une couche ORM JPA / Hibernate
* Une forme ayant recours aux principes du reactive manifesto, et à l'immutabilité avec RxJava côté serveur, et RxJs, côté client Angular 5
* Une forme ayant recours aux principes du reactive manifesto, et à l'immutabilité avec Vertx.io côté serveur, et RxJs, côté client Angular 5

Cela donnera soit 3 branches, et sur chaque branche, chaque releases correspond à une solution d'un exercice / un point de tutoriel. 

La release 1.0.0 de la forme "à l'ancienne" : 
* sera utilisée en premier, pour tester mes architectures infra (Hôte Docker / Free IPA server / Keycloak / Traefik.io / Gravitee.io + NGINX ... ).
* sera désignée dans cette documentation par "inventory.kytes.ioo-1.0.0.jar",
* sera et packagée avec springboot,
* sera implémentée avec spring data / springboot pour l'implémentation du design pattern DAO, et la génération des DTO avec mappers.



D'autres repo, créés et gérés dynamiquement par le cycle IAAC/girofle dans kytes, permettront de versionner les packaging possibles pour le déploiement de cette application : 

* Package en springboot simple, dans un conteneur docker, [repo](https://github.com/Jean-Baptiste-Lasselle/java-springboot-pack)
* Package avec un wildfly swarm hollowjar, le tout dans un conetneur docker, [repo](https://github.com/Jean-Baptiste-Lasselle/wildfly-swarm-hollowjar-pack)
* Package avec un wildfly swarm uberjar, dans un conteneur docker, [repo](https://github.com/Jean-Baptiste-Lasselle/wildfly-swarm-uberjar-pack)


 
### Cahier des charges technique : cible de déploiement

Je vais envoyer tout cela dans une cible de déploiement de tyoe **Hôte Docker**

Cette cible de déploiement comprendra : 

* Un reverse proxy à choisir (seront testés : traefik.io, nginx)
* Gravitee.io : juste pour apprendre ce que je peux faire avec load balancer ?
* Un Keycloak, pour la fédération d'identités
* Et tiens, j'essaietrai de vori comment je peux utiliser le STRAPI.io, dans ce contexte métier... (Headless CMS + Hardware inventory = ? ... cf. [jolokia](https://jolokia.org/), supervision JMX over REST APIs.... ce qui me permettrait d'avoir une supervision JMX des Endpoints de gestions ressources de l'inventaire...? ) =>> ce que je veux c'est aggréger les sources API de l'inventaire, comem si c'était un CMS, via le cleint Angular 5.
* La gestion des authentifications / auhtorisations est synchronisable avec un Keycloak dans un Free IPA server. OpenID explicit flow devra être utilisé, avec la plus forte option d'authentification avec certificats ssl clients.



Pour terminer, on tentera de migrer tout cela, plus les 3 versions d'application, vers K8s.


### Modèle de données

Chaque `Machine` possède les attributs suivants : 

* Un identifiant,(qui sera plmus tard à mapper avec les QR-code auto-collants apposés et scannés par les techniciens)
* Au moins une, sinon plusieurs adresses MAC (chaque adresse MAC correspond à une carte réseau)
* Une liste de composants, que j'appelerai des objets  `Module`
* Un lien vers la documentation globale de la machine (repo github versionnant les fichiers de docuementation téléchargés, et README.md rédigé par les équipes internes, pour compléter les documùentations, si nécessaire avec des liens vers N autres repos Github présentant les opérations standards de maintenance/supervision hardware).

Chaque `Module` possède les attributs suivants : 

* Un identifiant,(qui sera plmus tard à mapper avec les QR-code auto-collants apposés et scannés par les techniciens).
* Un lien vers la documentation du périphérique (repo github versionnant les fichiers de docuementation téléchargés, et README.md rédigé par les équipes internes, pour compléter les documùentations, si nécessaire avec des liens vers N autres repos Github présentant les opérations standards de maintenance/supervision hardware).

Un périphérique est une partie modulaire d'une machine, qui peut être désinstallée et ré-installée à volonté, selon une procédure documentée par le fabricant / vendeur du matériel. (Exemple: Cavium, Qualcomm, Dell...).

Exemples de `Module` : 

* Une carte réseau est un module, couramment qualifié de périphérique. 
* Les PSUs (**Power Supply Units**) sont des modules, que j'ai rarement (si ce n'est jamais) vu qualifiés de périphériques (et vous? cf. jean.baptiste.lasselle.it@gmail.com).
* Les disques durs hot swappable ou non, sont des modules, couramment qualifiés de périphériques.
* Les contrôleurs RAID, sont des modules, que j'ai rarement (si ce n'est jamais) vu qualifiés de périphériques (et vous? cf. jean.baptiste.lasselle.it@gmail.com).



## Décomposition des travaux

* Installation / configuration de l'hôte Docker
* Implémentation rapide de mon application de gestion d'inventaire hardware, avec un premier REST Endpoint testable. Springboot swagger.io et Base de données MongoDB derrière : 
  * cf. https://medium.com/@dersoz/spring-boot-vs-wildfly-swarm-in-the-land-of-enterprise-java-4ebd210fbc4a
  * cf. https://auth0.com/blog/automatically-mapping-dto-to-entity-on-spring-boot-apis/
  * cf.  swagger.io
* Installation / configuration du reverse proxy : NGINX d'abord pour faire très simple. Ensuite, premier essai rapide Traefik.io, avec docuementation du premier essai. but : pouvoir accédeer à mon application via un nom de domaine, du genre  https://hw-inventory.kytes.io 

* Conception / déploiement d'un premier conteneur avec Installation de STRAPI, et configuration d'une route "strapi-cms.kytes.io" dans le reverse proxy : strapi supprote-t-il le Scale UP ? J'ai vu des "promises" partout dans le code, ça vaut le coup d'essayer, de refaire mon conteneur STRAPI. 
* Le `strapi new MonAppliStrapi` doit se faire au build de l'image Docker, et non au run.
* Installation Configuration de Jenkins, et création du build de STRAPI, sur la base de la doc officielle STRAPI. Pluisieurs Pipeline pourront être créé, l'un pour les déploiement s de plugins, avec le fameux `npm run setup --plugins` à faire à la racine du projet STRAPI... Préalablement, et pour nettoyer le piepline de STRAPI, fair eun pur Jenkins pipeline pour une applciation purement NODEJS, en esayant de reproduire l'équivalent d'un `npm run setup --plugins` ...
* Faire un Jenkins pipeline pour le packaging du conteneur docker, en utilisdant les @Library Jenkins qui apparremment s'implémentent en ruby / groovy ...?
* Enfin, mon but premier : arriver à remonter un cycle devops complet avec chaîne complète de production industrielle, sur STRAPI : déploiement des plugins, déploiement des APIs, configuration de l'authentification, synchronsitation complète Keycloak , avec synchronisation du Keycloak sur un LDAP à l'ancienne (du typique pour les clients en ce moment). le tout doit se fair aussi avec un contexte reverse proxy et HTTPS partout.
* Installation / configuration de  Gravitee.io : ALors, ça m'apprte quoi, Gravitee, en plus du reverse proxy que 'jtuilise déjà? ET la propagation des certificats SSL, comment Gravitee.io l'assure-t-elle?
* Traefik,  : je trouverais comment faire le routage automatique (service discovery)  des conteneurs, et je trouverai comment fair le load balancing en changeant la cible de déploeiment pour un cluster Kubernetes, Traefik.io comme Ingress Controller K8s. Je consoliderai la réponse à la question : " La propagation des certificats SSL est-elle possible avec Traefik.io? " ?
* Faire une recette de provision d'un cluster Keycloak, avec derrière un cluster postgre SQL, à base de  `pg_pool` et `pg_master`. Le tout dans des conteneurs de grosse taille, sur " Hôtes Docker différents (et non des VMs comme au taff). Pour préparer, finir la rectte de provision d'un cluster wildfly.






