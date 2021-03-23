# "Dockerizing" Multiple Services 

### Dockerizing a React App - again!
the React App can be found inside the client dir \
we copy all dependecies and install them before copying over the code to take advantage of the cache\

change to the client dir and build the docker image:
```
cd ./client ; docker build -f Dockerfile.dev -t client . 
```
- define the build context with . 
- use -f to define the name of the Dockerfile (not needed if the name is just Dockerfile)
- use -t to tag your image 

run the client container you just created:
```
docker run -it client 
```
- alternatively, you could use the docker id to start it

### Dockerizing Generic Node Apps 
package.json contains a section titled 'scripts' and calls on nodemon \ 
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
environment variables must also be defined in our containers so that they can to talk to each other (see docker-compose.yml) \ 
environment variables set variables in the container at **run time** (when the container is started up) \
the image does NOT have any memory of the environment variables you define \ 
values for environment variables can be taken from a file on your computer (for secrets that shouldn't be uploaded to SCM) \
values for environment variables can be defined inside the docker-compose.yml (see example below) \
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
- the values for these keys will be used in ./server/keys.js
- you have to look at the documentation docker repoistory to find out what environment variables are needed 

### The Worker and Client services 

specify the Dockerfile name \
specify the build context \
create two volumes to accomplish the following:
1. ensure the node_modules directory does not get overwritten
2. grant the container access to the code needed for the application 

```
  client:
    stdin_open: true
    build:
      dockerfile: Dockerfile.dev
      context: ./client
    volumes:
      - /app/node_modules
      - ./client:/app
```

The same should be done for our worker service (also needs environment variables):

```
  worker:
    build:
      dockerfile: Dockerfile.dev
      context: ./worker
    volumes:
      - /app/node_modules
      - ./worker:/app
    environment:
      - REDIS_HOST=redis
      - REDIS_PORT=6379 
```

One crucial thing that is missing in this project is port mapping:
- There are no ports exposed inside docker-compose.yml 
- There's no port mapping to expose the server service to the outside world
- There's no port mapping to expose the client service to the outside world 

We will use Nginx to map our traffic to the server and client services

### Nginx path routing
right now, none of the services can be reached by a web browser \
we have no ports exposed for our services \
Nginx will be handling our requests and mapping traffic to our services 

**Nginx has one job:** \
read what the request starts with ('/api/' or '/') and route traffic to either the React server or the Express server:
- if ( request starts with '/' ) route to { React server (client service) }
- if ( request starts with '/api/' ) route to { Express server (server service) }

Why are we doing it this way?.. why not expose a port? - several reasons: 
- in a production environment, you don't want to worry about exposing ports 
- it would not be convient to append ':3000' or ':5000' to all of the functions of the code
- the port can very easily change
- it's easier to expose a 'token' (/api/ or /) and rely on Nginx to do the routing for us 

Nginx will chop off '/api/' from the beginnning of each request once it's processed
- this is an optional setting defined in default.conf (line 17) 
```
cat .\nginx\default.conf
```

### Routing with Nginx

### Building a custom Nginx image

### Starting up Docker Compose
it's highly likely that the application will crash the first time you try to run this application \
do not be discouraged, this is because our code inside ./server and ./worker are trying to call on the redis service before the image has been created \
we also expect nginx to crash the first time we run it, but with the restart:always flag this shouldn't be a problem \
build and run the containers:
```
docker-compose up --build
```
- stop the containers with ctrl + c

spin them back up:
```
docker-compose up
```

visit localhost:3050 on your browser,  

### Troubleshooting Startup Bugs
The console reveals the following error: 
**webpackHotDevClient.js:60 WebSocket connection to 'ws://localhost:3050/sockjs-node' failed:**
this is happening because anytime our react application boots in development mode, it wants to keep an active connection to the developer server so that it can get updated source code if changes are made \

by not setting up this web socket connection, we're unable to quickly query the index in our app \
- the network console reveals the issue if you try to search for an index 
- refresh the page and it will work as expected  


### Opening Websocket Connections

Nginx is currently not allowed to allow websocket connections so we will need to update the default.conf file \
we will expose one route in Nginx to allow websocket connections \ 
review the error message in the console closely: \ 
**webpackHotDevClient.js:60 WebSocket connection to 'ws://localhost:3050/sockjs-node' failed:** \
inside default.conf, we will add a new location called sockjs-node:
```
  location /sockjs-node {
    proxy_pass http://client;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
  }
```
as a reminder, if errors occur press ctrl + c to cancel and restart all of the containers again. \
the api container starting up before the redis and postgres container is what is causing this issue. \ 
now when you enter your index value, the error will no longer occur \ 
refresh the page and see the value that has been calculated !

### Changes made

**docker-compose.yml** 

- container_name \
the container name is defined for each container in docker-compose.yml \
this change was applied because '9_' (the directory name) was appended to the beginning of the name and '_1' was appended to end of the name of each container (e.g. 9_api_01) \

- depends_on \
nginx and api should be created and started after redis and postgres 


