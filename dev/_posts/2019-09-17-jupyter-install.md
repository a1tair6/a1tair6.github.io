# jupyter 설치
- dockerhub에 올라와 있는 공식 image 사용
- https://hub.docker.com/u/jupyter/
- local jupyter에서 run하는 것을 서버로(local 날려 먹은 적도 있고, 소스 관리도 안되고 있음)

## Environment Variables
```
CHOWN_EXTRA=/home/jovyan/work
CHOWN_HOME=yes
CHOWN_HOME_OPTS=-R
GRANT_SUDO=yes
NB_GID=0
NB_UID=0
NB_USER=root
```
