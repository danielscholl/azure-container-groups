FROM node:8.9.3-alpine
RUN mkdir -p /usr/src/app
COPY ./src/* /usr/src/app/
WORKDIR /usr/src/app
RUN npm install
CMD node /usr/src/app/index.js