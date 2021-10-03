# LAB 5 : Routage des appels vers les modules Gainde avec Spring Cloud Gateway

## Objectif

Cet atelier à pour but de configurer une gateway avec Spring Cloud Gateway permettant de définir un point d'entrée unique aux modules.
Les appels vers la **gateway-service** sont routées vers le bon module.
La gateway permet aussi d'assurer la sécurité et monitoring des modules


## les points à voir

+ Utilisation de Spring Cloud Gateway
+ Intégration avec Spring Cloud Discovery Client
+ Intégration avec Spring Cloud Config
+ Pattern de développement : API Gateway ou Proxy


## Initialisation du projet gateway-service

Il s'agit de générer un projet spring boot en se basant sur un modèle disponible sur Spring Initializr. Pour cela, aller sur [Spring Initializr](https://start.spring.io/) et générer une application en respectant toutes les informations ci dessous.

| Element | Valeur |
|--------|---------------|
| Project|  Maven Project |
| Language | Java |
|Spring Boot| 2.5.5|
|GroupID| com.jc.gainde|
|Artifact|gateway-service|
|Name|gateway-service|
|Description|Gateway Service|
|Package name|com.jc.gainde.gatewayservice|
|Packaging|Jar|
|Java|11|

**Dependencies**

Ajouter Config Server dans la zone Dependencies
```
Config Client
Eureka Discovery Client
Gateway
Actuator
```

Generer **gateway-service** en appuyant sur le bouton **GENERATE** ensuite décompresser l'archive et importer le projet dans l'éditeur de code.

## Intégration avec discovery-service

Au niveau de la classe principale **GatewayServiceApplication** rajouter l'annotation suivante
```
@EnableEurekaClient
```

au niveau du **pom.xml** rajouter la dependance suivante
```
<dependency>
  <groupId>javax.inject</groupId>
  <artifactId>javax.inject</artifactId>
  <version>1</version>
</dependency>
```

## Intégration avec config-service

Dans le fichier **application.properties** rajouter les élements suivant
```
spring.config.import=configserver:http://localhost:8888
spring.application.name=gateway-service
spring.profiles.active=dev
```


## Vérification de la configuration de la gateway service

Aller à l'url suivante pour voir si la configuration du **gateway-service** est disponible à travers le **config-service**

```
http://localhost:8888/gateway-service/dev
```
**dev** représente l'environnement cible.

**détails et explications**

```
server.port=8072
spring.application.name=gateway-service

# eureka client configuration
eureka.client.fetchRegistry=true
eureka.client.registerWithEureka=true
eureka.client.serviceUrl.defaultZone=http://localhost:8070/eureka/
eureka.instance.preferIpAddress=true

# actuator configuration
info.app.version=1.0
management.endpoint.gateway.enabled=true
management.endpoints.web.exposure.include=*

# gateway configuration
spring.cloud.gateway.discovery.locator.enabled=true
spring.cloud.gateway.discovery.locator.lowerCaseServiceId=true
```

## Compilation du projet

A l'aide de l'invite de commande, taper la commande
```
mvn clean install -DskipTests
```

## Lancement de la gateway-service

Pour lancer la gateway, utiliser la commande
```
mvn spring-boot:run
```

Vérifier dans les logs de démarrage que le service **gateway-service** démarre correctement et écoute sur le port **8072**


## Voir la liste des routes via actuator

Pour lister l'ensemble des routes du **gateway-service**
```
http://localhost:8072/actuator/gateway/routes
```

## Test de la route vers le module manifeste-service

Afin de récuperer tous les manifeste, on fait un appel au module **manifeste-service** en passant par cette url :
```
http://localhost:8080/v1/manifeste
```

Pour avoir la même chose en passant par la gateway, il suffit de lancer cette url
```
http://localhost:8072/manifeste-service/v1/manifeste
```

L'url est de la forme : http://adresse_gateway/nom_du_service/chemin_ressources

```
http://localhost:8072/manifeste-service/v1/manifeste -> http://localhost:8080/v1/manifeste
```

## Pour résumer

+ Utilisation de Spring Cloud Gateway pour la mise en place d'un point d'entrée unique à l'application
+ Intégration avec Spring Cloud Discovery Client pour identifier les routes vers les modules
+ Intégration avec Spring Cloud Config pour l'externalisation de la configuration
+ Intégration avec Actuator pour l'exposition des routes aux outils ops
