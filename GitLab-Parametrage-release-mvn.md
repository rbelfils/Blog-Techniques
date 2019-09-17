# Paramétrage d'une release mvn

1. Generer une clef RSA pub/prive
  https://docs.gitlab.com/ee/ssh/
2. Rajouter la clef public dans le menu Deploy Key de l'administration
   https://<HOST>/admin/deploy_keys
3. Rajouter la clef privée dans une variable.
  * Se connecter au repository et accéder au menu Settings -> CI / CD -> Environment variables et ajouter le contenu de la clef privée surtout avec 
  - le header : -----BEGIN RSA PRIVATE KEY-----
  - le footer :  -----END RSA PRIVATE KEY-----
4. Exemple de script CD/CI :

.gitlab-ci.yml
```
image: docker:latest

cache:
  key: "$CI_PROJECT_NAMESPACE_$CI_PROJECT_NAME"
  paths:
    - .m2/

services:
  - docker:dind

variables:
  MAVEN_OPTS: "-Dmaven.repo.local=.m2"
  DOCKER_DRIVER: overlay

stages:
  - test
  - build
  - release

test:
  image: maven:3.3-jdk-8-alpine
  stage: test
  script: "mvn clean test"

maven-build:
  image: maven:3.3-jdk-8-alpine
  stage: build
  script: "mvn clean package -DskipTests"
  artifacts:
    name: "$CI_JOB_STAGE-$CI_COMMIT_REF_NAME"
    paths:
      - "alertes-api/target/*.jar"
      - "alertes-client/target/*.jar"
    expire_in: 1 week

maven-release:
  image: maven:3.3-jdk-8-alpine
  stage: release
  when: manual
  script:
    - "which ssh-agent || ( apk update && apk upgrade && apk add git && apk add openssh-client)"
    - mkdir -p ~/.ssh
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' > ~/.ssh/id_rsa
    - chmod 700 ~/.ssh/id_rsa
    - eval $(ssh-agent -s)
    - ssh-add ~/.ssh/id_rsa
    - git config --global user.email "qa@tennaxia.com" && git config --global user.name "QA"
    - mvn release:prepare --batch-mode -Dmaven.test.skip=true
    - mvn release:perform
  artifacts:
    paths:
      - "alertes-api/target/*.jar"
      - "alertes-client/target/*.jar"
 ```

