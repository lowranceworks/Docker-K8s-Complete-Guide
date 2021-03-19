

```
.
```

build the docker image:
```
docker build .
```

build the docker image with a tag:
```
docker build -t lowranceworks/simpleweb . 
```

run the docker container on port 5000 on your local host:
```
docker run -p 5000:8080 lowranceworks/simpleweb
```
- the app listens on port 8080, so we map localhost:5000 to container:8080
- visit localhost:5000 on your browser to see the application