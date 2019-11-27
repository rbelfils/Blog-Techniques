# Add sonar to a java project

Based on a project using :
- Java 1.8
- Maven 3.6.0
- Gitlab 12.2.5 and Gitlab runner with docker executor and forked project
- Sonarqube 7.1

### Configure a sonar reference build

Here we configure a reference sonar job on which will be compared MR builds.
This job will update a reference project on the master branch.

- In gitlab, Settings > CI/CD > Variables or at the group level, configure the following variables :
    - SONAR_HOST_URL
    - SONAR_AUTH_TOKEN
- In .gitlab-ci.yml :


    stages:
      - sonar
    
    maven-scheduled-sonar:
      image: maven:3.3-jdk-8-alpine
      stage: sonar
      script:
        - mvn clean verify sonar:sonar -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.branch=master
      only:
        - master
        - schedules

To schedule it once a day at 22:00, configure a scheduled pipeline in CI/CD > Schedules with the following values :

- Interval pattern : select Custom and type in "0 22 * * *"
- Target branch : master

The sidekiq worker used by gitlab to manage the scheduling is by default configured to execute each hour at the 19th minute.
To change this behavior, modify in gitlab.rb the following (here set to execute every 10 minutes) :

    pipeline_schedule_worker_cron: '*/10 * * * *'


### Configure the MR build to execute sonar against the reference

For this job, I faced several issues :
- Execute the build on the merge result is only available from the premium version of gitlab : https://docs.gitlab.com/ee/ci/merge_request_pipelines/pipelines_for_merged_results/index.html
- For security reasons, Gitlab does not pass variables from the main repository to the forked one. To follow : https://gitlab.com/gitlab-org/gitlab/issues/11934
 
So, to build on the merge result, I manually merge the source and destination branches.
To be able to do that, I need to add on my profile (or a user dedicated to runners) the public SSH key of the runner 
and create a variable on the forked repository containing the private key : GITLABRUNNER_PRIVATE_KEY.

- In gitlab, for the forked repository, Settings > CI/CD > Variables, configure the following variables :
    - GITLABRUNNER_PRIVATE_KEY
    - SONAR_HOST_URL
    - SONAR_AUTH_TOKEN
- In .gitlab-ci.yml :


    .merge-result: &merge-result
      variables:
        ORIGIN_REPO: ssh://git@$CI_SERVER_HOST/$CI_MERGE_REQUEST_PROJECT_PATH.git
      before_script:
        - "which ssh-agent || ( apk update && apk upgrade && apk add git && apk add openssh-client)"
        - chmod 600 $GITLABRUNNER_PRIVATE_KEY                                                                     #The private key needs to be accessed only by the current user, otherwise an error is raised
        - git config core.sshCommand "ssh -o StrictHostKeyChecking=no -i $GITLABRUNNER_PRIVATE_KEY -F /dev/null"  #To avoid the unknown host error "-o StrictHostKeyChecking=no" needs to be added to the ssh command 
        - grep -q $ORIGIN_REPO .git/config || git remote add the_repo $ORIGIN_REPO                                #Add the destination repository only if it does not exist. It can exist if already added in a previous build
        - git fetch the_repo
        - git merge the_repo/$CI_MERGE_REQUEST_TARGET_BRANCH_NAME
    
    maven-build:
      image: maven:3.3-jdk-8-alpine
      stage: build
      <<: *merge-result
      script:
        - mvn clean verify sonar:sonar --batch-mode -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.branch=develop -Dsonar.analysis.mode=preview -Dsonar.issuesReport.console.enable=true -Dsonar.gitlab.commit_sha=$CI_COMMIT_SHA -Dsonar.gitlab.ref_name=master -Dsonar.gitlab.disable_global_comment=true -Dsonar.gitlab.only_issue_from_commit_file=true -Dsonar.gitlab.project_id=$CI_PROJECT_ID
      artifacts:
        name: "$CI_JOB_STAGE-$CI_COMMIT_REF_NAME"
        paths:
          - "module-api/target/*.jar"
          - "module-client/target/*.jar"
        reports:
          junit:
            - module-api/target/failsafe-reports/TEST-*.xml
            - module-service/target/surefire-reports/TEST-*.xml
        expire_in: 1 week
      only:
        - merge_requests
      except:
        - schedules

### Sources

- https://docs.gitlab.com/ee/ci/merge_request_pipelines/
- https://docs.gitlab.com/ee/ci/merge_request_pipelines/pipelines_for_merged_results/index.html
- https://gitlab.com/gitlab-org/gitlab/issues/11934
- https://docs.gitlab.com/ee/user/project/pipelines/schedules.html#advanced-configuration