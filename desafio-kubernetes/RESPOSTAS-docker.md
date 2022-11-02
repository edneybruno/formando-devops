### 1.

```
docker container run --rm alpine hostname
```

### 2.

docker container run --name webserver -d -p 8080:80 nginx:1.22

### 3.

Foi criado um arquivo de configuração default.conf no diretório ${PWD} com a porta alterada pela que foi pedida na questão, então montei como somente leitura no diretório de configuração do nginx.


´´´
docker container run --name webserver -d -p 8080:90 --mount type=bind,src=${PWD},dst=/etc/nginx/conf.d/,ro nginx:1.22
´´´

### 4.

Arquivo: hello.py

```
def main():
	print('Hello World in Python!')

if __name__ == '__main__':
	main()
```

Arquivo: Dockfile
```
FROM python as build
WORKDIR /app
ADD ./hello.py /app
ENTRYPOINT python hello.py
```

```
docker build -t app-python .
docker container run --name python app-python:latest
```

### 5.

docker container run --name webserver -d --memory 128m --cpus 0.5 nginx

### 6.

docker system prune

### 7.

docker image inspect [ID_IMAGE]
