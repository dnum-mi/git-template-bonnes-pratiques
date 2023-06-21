
# ==== CONFIGURE =====
# Use a Node 16 base image
FROM node:18-alpine as build

ARG http_proxy
ARG https_proxy
ARG npm_registry
ARG no_proxy

# Set the working directory to /app inside the container
WORKDIR /app

# use proxy & private npm registry
# With internal npm repo (autosigned) disable strict ssl : strict-ssl false
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo "Europe/Paris" > /etc/timezone ; \
    if [ ! -z "$http_proxy" ] ; then \
    npm config delete proxy; \
    npm config set proxy $http_proxy; \
    npm config set https-proxy $https_proxy ; \
    npm config set no-proxy $no_proxy; \
    fi ; \
    [ -z "$npm_registry" ] || npm config set registry=$npm_registry ; \
    [ -z "$npm_registry" ] || npm config set strict-ssl false

# copy the package.json to install dependencies
COPY package.json package-lock.json ./

# Copy app files
COPY . .

# ==== BUILD =====
# Install dependencies (npm ci makes sure the exact versions in the lockfile gets installed)
RUN npm install

# Build the app
RUN yarn build

# ==== RUN =======

FROM nginx:1.23.3

ENV PORT 8080

COPY ./nginx/default.conf.template /etc/nginx/conf.d/default.conf
#COPY ./nginx/nginx.conf.template /etc/nginx/nginx.conf

## Remove default nginx index page
RUN rm -rf /usr/share/nginx/html/*

COPY --from=build /app/build /usr/share/nginx/html

CMD ["nginx", "-g", "daemon off;"]
