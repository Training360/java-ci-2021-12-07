## Lab 1

Wrapper:

```
mvn -N io.takari:maven:wrapper 
```

Következő parancs végrehajtása:

```
cd C:\
mkdir training
cd training
git clone https://github.com/Training360/java-ci-2021-12-07
cd java-ci-2021-12-07\hello-world
set JAVA_HOME=C:\Program Files\Java\jdk-17.0.1
mvnw package
```

## Lab 2

```
git pull
cd C:\training\java-ci-2021-12-07
mvnw package
mvnw spring-boot:run
```

Elérhető a `http://localhost:8080/swagger-ui.html` 

```
mvnw versions:display-dependency-updates
mvnw dependency:tree
```

## Lab 3

* Tesztlefedettség

```
git pull
mvnw package
```

## Lab 4

* Integrációs tesztekről, H2 embedded db-vel

```
git pull
mvnw package
mvnw verify
```

## Lab 5

Adminisztrátorként indított parancssorban:

```
net localgroup docker-users %USERDOMAIN%\%USERNAME% /add
```

Tesztelni:

```
docker run hello-world
```

Adatbázis:

```
docker run -d -e MYSQL_DATABASE=employees  -e MYSQL_USER=employees -e MYSQL_PASSWORD=employees -e MYSQL_ALLOW_EMPTY_PASSWORD=yes -p 3306:3306 --name employees-mariadb  mariadb
```

Átírni az `application.properties` fájlt.

## Lab 6

Integrációs teszt futtatása parancssori konfigurációval:

```
mvnw -Dspring.datasource.url=jdbc:mariadb://localhost/employees -Dspring.datasource.username=employees -Dspring.datasource.password=employees verify

mvnw -Dspring.datasource.url=jdbc:h2:mem:db;DB_CLOSE_DELAY=-1 -Dspring.datasource.username=sa -Dspring.datasource.password=sa verify
```

## Lab 7 - Docker

NGINX indítása volume-mal:

```
docker run -d -p8080:80 -v %CD%\nginx-share:/usr/share/nginx/html --name my-nginx nginx
```

## Lab 8 - Saját Docker image

```
cd hello-world
docker build -t hello-world-java .
docker run hello-world-java
```

## Lab 9 - Saját employees Docker image

```
git pull
mvnw package
docker build -t employees .
docker run -d -p8080:8080 --name my-employees employees
docker logs -f my-employees
```

## Lab 10 - Két kapcsolódó konténer

```
docker network create employees-net

docker run -d -e MYSQL_DATABASE=employees  -e MYSQL_USER=employees -e MYSQL_PASSWORD=employees -e MYSQL_ALLOW_EMPTY_PASSWORD=yes -p 3307:3306 --name employees-app-mariadb --network employees-net mariadb

docker run -d -p8081:8080 --name employees-app --network employees-net -e SPRING_DATASOURCE_URL=jdbc:mariadb://employees-app-mariadb/employees -e SPRING_DATASOURCE_USERNAME=employees -e SPRING_DATASOURCE_PASSWORD=employees employees
```

## Lab 11 - Docker Compose

```
cd employees-app
docker compose up -d
docker compose logs
docker compose down
```

## Lab 12 - Postman tesztesetek

```javascript
pm.test("Status code is 201", function () {
    pm.response.to.have.status(201);
});

pm.test("Has name", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.name).to.eql("John Doe");
});
```

```javascript
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});
```

```
docker compose up --abort-on-container-exit
```

## Lab 13 - GitLab image letöltése

```
docker pull gitlab/gitlab-ce:latest
docker pull gitlab/gitlab-runner:alpine
```

## Lab 14 - Layered

```
docker build -f Dockerfile.layered -t employees . 
```

## Lab 15 - GitLab infrastruktúra elindítása

```
cd gl
docker compose up -d
docker exec -it gl-gitlab-1 grep "Password:" /etc/gitlab/initial_root_password
```

Bejelentkezés: `root` felhasználóval

## Lab 16 - Runner regisztráció

```
docker exec -it gl-gitlab-runner-1 gitlab-runner register --non-interactive --url http://gl-gitlab-1 --registration-token ghe3_RWy7gqiAhYuQSBA --executor docker --docker-image docker:20.10.11 --docker-network-mode gl_default --clone-url http://gl-gitlab-1 --docker-volumes /var/run/docker.sock:/var/run/docker.sock
```

## Lab 17 - First pipeline

`.gitlab-ci.yml`

```
image: eclipse-temurin:17-focal

stages:
  - build

build-job:
  stage: build
  script:
    - echo "Hello Pipeline"
```

* Regisztráció a Docker Hubon (https://hub.docker.com/)
  
Parancs:

```
docker login
docker pull docker:20.10.11
docker pull eclipse-temurin:17-focal
git update-index --chmod=+x mvnw
```

```yaml
image: eclipse-temurin:17-focal

stages:
  - build

build-job:
  stage: build
  script:
    - echo "Hello Pipeline"
    - ./mvnw package
```

# Lab 17 - Pipeline cache

```yaml
variables:
   MAVEN_OPTS: "-Dmaven.repo.local=$CI_PROJECT_DIR/.m2/repository"

cache:
  paths:
    - .m2/repository
```

# Lab 18 - Pipeline 3 stage-gel

```yaml
variables:
   MAVEN_OPTS: "-Dmaven.repo.local=$CI_PROJECT_DIR/.m2/repository"

cache:
  paths:
    - .m2/repository

image: eclipse-temurin:17-alpine

stages:
  - build
  - test
  - image

build-job:
  stage: build
  script:
    - echo "Hello Pipeline"
    - ./mvnw package
  artifacts:
    paths:
      - target

test-job:
  stage: test
  script:
    - ./mvnw verify

image-job:
  stage: image
  image: docker:20.10.11
  script:
    - docker build -t hello-world-java .
```

# Lab 19 - SonarQube

```
docker run --name employees-sonarqube -d -p 9000:9000 sonarqube:lts
```

Felhasználónév, jelszó: `admin` / `admin`

Token: User > My Account > Security 

```
mvnw -Dsonar.login=70fe8dfd7d6fa4bb249aa648cd7b87afd912fc22 sonar:sonar
```

```
mvnw -Dsonar.host.url=http://sonar.internal:9000 -Dsonar.login=70fe8dfd7d6fa4bb249aa648cd7b87afd912fc22 sonar:sonar
```

## Lab 20 

```
docker build -t hello-world-java .
docker tag [username]/hello-world-java
docker push [username]/hello-world-java
```

```
docker run --rm training360/hello-world-java
```

## Lab 21 - Nexus indítása

```
docker run --name employees-nexus --detach --publish 8091:8081 --publish 8092:8082   --volume nexus-data:/nexus-data sonatype/nexus3
docker exec -it employees-nexus cat /nexus-data/admin.password
```

Username: `admin`

```
docker tag hello-world-java localhost:8092/hello-world-java
docker login localhost:8092
docker push localhost:8092/hello-world-java
```

## Lab 22 - Proxy Maven repo

`%HOME%\.m2\settings.xml`  tartalma:

```
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
   <mirrors>
    <mirror>
      <id>central</id>
      <name>central</name>
      <url>http://localhost:8091/repository/maven-public/</url>
      <mirrorOf>*</mirrorOf>
    </mirror>
  </mirrors>
   <profiles></profiles>
</settings>
```