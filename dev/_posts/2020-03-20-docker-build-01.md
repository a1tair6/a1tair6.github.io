# docker build context size is very large

- docker build 할 때마다 7G 라 너무 많은 시간이 소요
- https://docs.docker.com/engine/reference/builder/#dockerignore-file
- docker file이 포함 되어 있는 directory 를 보기 때문에 build와 관련 없는 파일을 분리 하거나 .dockerignore 파일을 생성


## .dockerignore
```
# comment
*.tar.gz
```

- **한 번 build 하면 20분 걸리던게 30초 만에 끝남.**