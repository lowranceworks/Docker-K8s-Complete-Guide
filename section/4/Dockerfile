# Specify a base image
FROM node:alpine

WORKDIR /usr/app

# Install some depenendencies
COPY ./package.json ./
RUN npm install

# Copy everything in the host's working dir to the container's working dir
COPY ./ ./

# Default command
CMD ["npm", "start"]