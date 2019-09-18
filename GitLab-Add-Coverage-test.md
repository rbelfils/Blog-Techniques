# Mettre la couverture de test sur un projet multi module

C'est une des solutions trouvées qui semble être la plus simple.

Vu sur le site : https://prismoskills.appspot.com/lessons/Maven/Chapter_06_-_Jacoco_report_aggregation.jsp
Projet maven :
```
Parent Pom
  |
  -- Module 1
  -- Module 2
  -- Module Rapport
  ```

## Pom parent

1. Rajouter  la dependance à Jacoco :
  ```
    <dependencies>
        <dependency>
            <groupId>org.jacoco</groupId>
            <artifactId>jacoco-maven-plugin</artifactId>
            <version>0.8.4</version>
        </dependency>
    </dependencies>
```
2. Rajouter dans la partie build -> plugins :
   ```
          <plugin>
            <groupId>org.jacoco</groupId>
            <artifactId>jacoco-maven-plugin</artifactId>
            <executions>
                <execution>
                    <id>prepare-agent</id>
                    <goals>
                        <goal>prepare-agent</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
   ```      
## Module 1
    Rien à rajouter

## Module 2
    Rien à rajouter

## Module Rapport

1. Rajouter les dependances des autres projets:
 ```
 <dependencies>
      <dependency>
          <groupId>GROUP_ID</groupId>
          <artifactId>Module1</artifactId>
          <version>${project.version}</version>
      </dependency>
      <dependency>
          <groupId>GROUP_ID</groupId>
          <artifactId>Module2</artifactId>
          <version>${project.version}</version>
      </dependency>
  </dependencies>
``` 
2. Rajouter le plugin Jacoco avec le goal 'report-aggregate'
```
   <build>
      <plugins>
          <plugin>
              <groupId>org.jacoco</groupId>
              <artifactId>jacoco-maven-plugin</artifactId>
              <version>${jacoco-maven-plugin.version}</version>
              <executions>
                  <execution>
                      <id>report-aggregate</id>
                      <phase>verify</phase>
                      <goals>
                          <goal>report-aggregate</goal>
                      </goals>
                  </execution>
              </executions>
          </plugin>
      </plugins>
  </build>
```    
## Résultat :
 Les couvertures de test de tous les modules sont réunis dans le dossier suivant :
 ```"<Module Rapport>/target/site/jacoco-aggregate"```

## Mettre en place le badge sous Gitlab CI
1. extraction de la couverture de test

Voir le site : https://jeankins.blogspot.com/2018/03/jacoco-on-gitlab-for-play-26.html

Cela consite, lors du lancement des tests, d'afficher le rapport HTML sur la console à l'aide de la commande 'cat'.

exemple gitlab-ci.tml:
```
image: docker:latest

cache:
  key: "$CI_PROJECT_NAMESPACE_$CI_PROJECT_NAME"
  paths:
    - ${CI_PROJECT_DIR}/.m2

services:
  - docker:dind
 ....
 
test:
  image: maven:3.3-jdk-8-alpine
  stage: test
  script:
    - mvn clean verify
    - cat alertes-report/target/site/jacoco-aggregate/index.html # Affichage du rapport Jacoco
  artifacts:
    reports:
      junit:
        - alertes-api/target/failsafe-reports/TEST-*.xml
        - alertes-service/target/surefire-reports/TEST-*.xml
        
....
```
2. Ajout d'un filtre pour capturer la couverture de test lors des pipelines
Aller dans 'Settings' du projet puis 'CI / CD' puis 'General pipelines' et renseigner la zone 'Test Coverage parsing' avec 
``` Total.*?([0-9]{1,3})% ```

3. Ajouter le badge de coverage test
Aller dans 'Settings' puis 'General project' puis 'Badges' et renseigner les zones suivantes :

Exemple pour le pipeline :
Link : https://gitlab.tennaxia.org/%{project_path}/badges/%{default_branch}/pipeline.svg
Base Url Image: https://gitlab.tennaxia.org/%{project_path}/badges/%{default_branch}/pipeline.svg

Exemple pour le coverage test :
Link: https://gitlab.tennaxia.org/rbelfils/alertes/badges/jacoco/coverage.svg
Base Url Image: https://gitlab.tennaxia.org/rbelfils/alertes/badges/jacoco/coverage.svg
