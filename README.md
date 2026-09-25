[requirements.txt](https://github.com/user-attachments/files/32669744/requirements.txt)

1. Создание папки:
mkdir docker-nginx; cd docker-nginx

2. Создание образа по написанному коду
docker build . -t nginx-project

3. Создание и запуск контейнера
docker run -d -p 8888:80 nginx-project


