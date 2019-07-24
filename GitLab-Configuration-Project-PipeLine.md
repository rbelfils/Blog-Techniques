## Activer le pipeline pour le projet

Aller dans Project Settings -> Permission ->  Activate Pipeline.
@see https://docs.gitlab.com/ee/ci/enable_or_disable_ci.html

## Configuration de son executor gitlab runner
https://docs.gitlab.com/ee/ci/docker/using_docker_build.html

Mettre en place le partage volume '/cache' pour ne pas redescendre les dépendances tout le temps.


### configuration du fichier config.toml de gitlab Runner
1) Mettre docker en privilegié
2) Rajouter les serveurs de DNS pour que Docker in Docker puisse retrouver l'url de GITLab.

config.toml
```  [runners.docker]
    tls_verify    = false
    image         = "docker:dind"
    privileged    = true
    disable_cache = false
    volumes       = ["/cache"]
    dns           = ["192.168.xxx.xxx","192.168.xxx.xxx"]
    extra_hosts   = ["git.my-company.com:192.168.xxx.xxx"]
```
