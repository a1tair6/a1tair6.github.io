# 자주 사용하는 docker 명령어

```sh
docker images
docker run -p port:port image_id
docker exec -p port:port images_id
docker rmi image_id
docker rm container_id
docker build -e "config variables=value" --tag url/tag:version path
docker ps -a
docker logs container_id
```