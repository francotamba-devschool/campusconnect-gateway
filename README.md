# CampusConnect Gateway

Application Spring Cloud Gateway. Elle reçoit les appels HTTP et transmet les chemins configurés au Config Server vers les services enregistrés dans Eureka.

## Ce que signifie le code

- `spring.application.name: gateway` demande au Config Server la configuration `gateway-dev.yml`.
- Le fichier distant configure le port `8080` et les routes `/auth/**` et `/students/**`.
- `lb://auth-service` et `lb://student-service` sont des noms logiques. Eureka fournit les instances, puis Spring Cloud LoadBalancer en choisit une.
- Les chemins sont transmis tels quels : `/students/42` doit être accepté par le contrôleur Student sous ce même chemin.

Cette première version utilise le routage et le load balancing standards. Elle ne contient pas encore de filtre JWT personnalisé ni d'algorithme de load balancing maison.

## Lancement depuis l'IDE

Prérequis : Java 21, Maven et socle d'infrastructure démarré (Config Server sur 8888 et Eureka sur 8761).

Depuis ce dossier :

```bash
mvn spring-boot:run
```

Les valeurs par défaut utilisent le profil `dev` et `http://localhost:8888`.

## Lancement avec Docker

Démarrer d'abord le socle depuis son dépôt, puis depuis ce dossier :

```bash
docker compose -f compose.yaml up -d --build
```

La gateway rejoint le réseau externe `campus-net`, utilise `config-server:8888` et le profil `dev,docker`. Cette option suppose que le réseau a été créé par le compose du socle.

Arrêt :

```bash
docker compose -f compose.yaml down
```

## Vérification

- Santé : `http://localhost:8080/actuator/health`
- Eureka : `http://localhost:8761`
- Config distante : `http://localhost:8888/gateway/dev`
- Routage : envoyer une requête à `http://localhost:8080/auth/...` ou `http://localhost:8080/students/...` lorsque le service correspondant est lancé et enregistré dans Eureka.

Un service absent d'Eureka ne peut pas recevoir l'appel ; la gateway renverra une erreur de disponibilité.
