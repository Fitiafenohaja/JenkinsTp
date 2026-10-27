# JenkinsTp

Projet de TP d'intégration continue : une application Java « Hello World » construite avec Maven, compilée et testée par un pipeline Jenkins, puis publiée sur un dépôt Nexus.

## À propos

L'application elle-même est volontairement minimale : elle affiche seulement `Hello World`. Le but du TP est la chaîne d'outils autour du code, pas le code lui-même.

À chaque lancement du pipeline, Jenkins :

1. récupère le code depuis GitHub ;
2. le compile et le package en `.jar` avec Maven ;
3. exécute le programme pour vérifier qu'il fonctionne ;
4. publie le `.jar` sur Nexus, le dépôt d'artefacts où sont stockées les versions construites.

```
GitHub ──► Jenkins (build + run) ──► Nexus (maven-snapshots)
```

Ce schéma est celui qu'on retrouve en entreprise : le développeur pousse son code, le serveur d'intégration continue le construit et le vérifie automatiquement, et le résultat est rangé dans un dépôt d'où il peut être récupéré ou déployé.

## Contenu

| Fichier | Rôle |
|---|---|
| `src/main/java/HelloWorld.java` | Programme Java qui affiche `Hello World` |
| `pom.xml` | Configuration Maven (`mg.eni:hello-world:1.0-SNAPSHOT`, Java 21) et dépôt Nexus de destination |
| `Jenkinsfile` | Pipeline déclaratif Jenkins |

## Pipeline

1. **Checkout** : récupération du code depuis GitHub
2. **Build** : `mvn clean package`
3. **Run** : exécution de `HelloWorld`
4. **Deploy to Nexus** : `mvn deploy` vers `maven-snapshots`
5. **Post** : archivage du `.jar` dans Jenkins

## Prérequis

- Java 21 et Maven 3
- Jenkins avec les plugins Pipeline, Git et Credentials Binding
- Nexus Repository 3

## Environnement Docker

Jenkins et Nexus tournent dans des conteneurs reliés par le réseau `cicd`, ce qui permet à Jenkins de joindre Nexus via `http://nexus:8081`. Les données sont conservées dans les volumes `jenkins_home` et `nexus-data`.

### Première installation

```bash
# Réseau commun
docker network create cicd

# Jenkins
docker run -d --name jenkins --network cicd --restart unless-stopped \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /usr/local/bin/docker:/usr/bin/docker \
  jenkins/jenkins:2.555.2-lts

# Nexus
docker run -d --name nexus --network cicd --restart unless-stopped \
  -p 8081:8081 \
  -v nexus-data:/nexus-data \
  sonatype/nexus3:3.70.1
```

Si le conteneur `jenkins` existe déjà sans le réseau : `docker network connect cicd jenkins`.

### Mots de passe initiaux

```bash
# Jenkins (assistant de première connexion)
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword

# Nexus (utilisateur admin, à changer à la première connexion)
docker exec nexus cat /nexus-data/admin.password
```

Nexus met 2 à 3 minutes à démarrer. Il est prêt quand cette commande renvoie `200` :

```bash
curl -s -o /dev/null -w "%{http_code}\n" http://localhost:8081/service/rest/v1/status
```

### Démarrer, arrêter, consulter

```bash
docker start jenkins nexus     # démarrer
docker stop jenkins nexus      # arrêter
docker restart jenkins         # redémarrer Jenkins
docker ps                      # vérifier qu'ils tournent
docker logs -f jenkins         # logs Jenkins
docker logs -f nexus           # logs Nexus
```

## Configuration Jenkins

- **Tools** : une installation Maven nommée `Maven3`
- **Credentials** : un identifiant *Username with password* d'ID `nexus-credentials` contenant le compte Nexus
- **Job** : type *Pipeline*, *Pipeline script from SCM*, dépôt `https://github.com/Fitiafenohaja/JenkinsTp.git`, branche `*/main`, script `Jenkinsfile`
- **Lancer un build** : ouvrir le job `JenkinsTp` puis *Build Now*. Le job ne se déclenche pas automatiquement à chaque push.

Si le mot de passe Nexus change, mettre à jour l'identifiant `nexus-credentials` dans Jenkins, sinon l'étape *Deploy to Nexus* échoue.

## Lancer en local

```bash
mvn clean package
java -cp target/classes HelloWorld
```

## Accès

- Jenkins : http://localhost:8080
- Nexus : http://localhost:8081 (artefacts dans `maven-snapshots/mg/eni/hello-world/`)
