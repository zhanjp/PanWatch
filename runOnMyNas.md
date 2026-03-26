# 构建镜像
```shell
sudo docker build -t panwatch:latest .
```
### 运行容器
```shell
sudo docker rm -f panwatch
sudo docker run -d --name panwatch -p 8000:8000 -v panwatch_data:/app/data panwatch:latest
````
