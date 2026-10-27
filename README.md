# JenkinsTp

Projet de TP d'intégration continue : une application Java « Hello World » construite avec Maven, compilée et testée par un pipeline Jenkins, puis publiée sur un dépôt Nexus.

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

Jenkins et Nexus tournent dans des conteneurs reliés par le réseau `cicd`, ce qui permet à Jenkins de joindre Nexus via `http://nexus:8081`.

```bash
docker network create cicd
docker network connect cicd jenkins
docker run -d --name nexus --network cicd -p 8081:8081 \
  -v nexus-data:/nexus-data sonatype/nexus3:3.70.1
```

Mot de passe admin initial de Nexus :

```bash
docker exec nexus cat /nexus-data/admin.password
```

## Configuration Jenkins

- **Tools** : une installation Maven nommée `Maven3`
- **Credentials** : un identifiant *Username with password* d'ID `nexus-credentials` contenant le compte Nexus
- **Job** : type *Pipeline*, *Pipeline script from SCM*, dépôt `https://github.com/Fitiafenohaja/JenkinsTp.git`, branche `*/main`, script `Jenkinsfile`

## Lancer en local

```bash
mvn clean package
java -cp target/classes HelloWorld
```

## Accès

- Jenkins : http://localhost:8080
- Nexus : http://localhost:8081 (artefacts dans `maven-snapshots/mg/eni/hello-world/`)
