# Dockerizing Angular Apps
Avoid `npm install` to be run every time the image is build by placing it before copying the source code to the container.

As indicated in [Scotch.io](https://scotch.io/tutorials/create-a-mean-app-with-angular-2-and-docker-compose#dockerizing-angular-2-client-app)

    WORKDIR /usr/src/app
    COPY package.json /usr/src/app
    RUN npm install
    COPY . /usr/src/app

As indicated in [Hummingbird Client](https://github.com/hummingbird-me/hummingbird-client/blob/the-future/Dockerfile)

    WORKDIR /opt/kitsu/client
    COPY package.json /opt/kitsu/client
    RUN yarn install
    COPY . /opt/kitsu/client

As indicated in [Dockerized Angular App Example](https://github.com/karlkori/dockerized-angular-app/blob/master/Dockerfile)

    WORKDIR /var/www/app/current
    ADD package.json /var/www/app/current
    RUN npm install
    ADD . /var/www/app/current

As indicated by [Laurent Brodoux](https://lbroudoux.wordpress.com/2016/07/05/dockerize-your-mean-application/)

    WORKDIR /usr/src/app
    ADD /dist/package.json /usr/src/app
    RUN npm install
    ADD dist /usr/src/app

As indicated by [Daniel Popescu](https://dpopescu.me/2017/03/13/running-angular-applications-inside-a-docker-container-part-1/)

    WORKDIR /usr/src/app
    ADD package.json /usr/src/app
    RUN npm install
    ADD . /usr/src/app

