# multi-step build process:
# start with a node alpine image with an alias of builder (you'll understand why later)
FROM node:alpine as builder
# define work dir
WORKDIR '/app'
# copy dependencies 
COPY ./package.json ./
# install dependecies 
RUN npm install  
# copy the apps files to the container
COPY . .
# build the production version of our frontend assets 
RUN npm run build 

# all of the production assets will be placed inside /app/build/ 
# we want to copy all the contents over to an nginx build

FROM nginx
# open port 3000
EXPOSE 3000 
# override the default nginx config file
COPY ./nginx/default.conf /etc/nginx/conf.d/default.conf
# copy production build assets from the build phase above
COPY --from=builder /app/build /usr/share/nginx/html

