# "Dockerizing" Multiple Services 

## Notes

This is a reminder to use the -it flag when you test the React container after building from the Dockerfile in the next lecture:
```
docker run -it IMAGE_ID
```
- .

### Dockerizing a React App - again!
The React App can be found inside the client dir \
We copy all dependecies and install them before copying over the code to take advantage of the cache\

change to the client dir and build the docker image:
```
docker build -f Dockerfile.dev -t client . 
```
- define the build context with . 
- use -f to define the name of the Dockerfile (not needed if the name is just Dockerfile)
- use -t to tag your image 

run the client container you just created:
```
docker run client
```
- alternatively, you could use the docker id to start it

### Dockerizing Generic Node Apps 
change to the server dir \
package.json contains nodemon \ 
nodemon is a command line tool that will restart your application when a change to your source code is made \
when this container starts up, 'npm run dev' will be executed (command to run nodemon, see package.json line 8)

change to the server dir and build your server image:
```
cd ../server ; docker build -t server -f Dockerfile.dev .
```

change to the worker dir and build your worker image:
```
cd ../worker ; docker build -t worker -f Dockerfile.dev .
```

### Adding Postgres as a Service
docker-compose.yml will define how our services talk to each other \
a password environment variable is required for Postgres
```
  postgres:
    image: 'postgres:latest'
    environment:
      - POSTGRES_PASSWORD=postgres_password
```

### Docker-compose config
the redis service:
```
  redis:
    image: 'redis:latest'
```

the api (server) service:
```
  api:
    build:
      dockerfile: Dockerfile.dev
      context: ./server
    volumes:
      - /app/node_modules
      - ./server:/app
```
- find a dir called server and look for Dockerfile.dev 
- inside the container, don't try to override /app/node_modules/
- look at the server dir and copy everything inside to our container's /app/ dir (this is where the source code is)


### Environment Variables with Docker Compose 
- environment variables must also be defined in our containers so that they can to talk to each other (see docker-compose.yml) \ 
```
  api:
    build:
      dockerfile: Dockerfile.dev
      context: ./server
    volumes:
      - /app/node_modules
      - ./server:/app
    environment:
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - PGUSER=postgres
      - PGHOST=postgres
      - PGDATABASE=postgres
      - PGPASSWORD=postgres_password
      - PGPORT=5432
```


.:
```
docker run -t postgres postgres:latest
```
- .

.:
```
.
```
- .

