
# ==== CONFIGURE =====
# Use a Node 16 base image
FROM node:18-alpine as build

ARG http_proxy
ARG https_proxy
ARG npm_registry
ARG no_proxy

# Set the working directory to /app inside the container
WORKDIR /git-template-test

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


# Copy app files
COPY . .
# ==== BUILD =====
# Install dependencies (npm ci makes sure the exact versions in the lockfile gets installed)
RUN npm install

# Set the env to "production"

ENV NODE_ENV production
ENV PORT 8080

# Build the app
RUN npm run build
# ==== RUN =======

FROM nginx:alpine

COPY ./nginx/nginx.conf /etc/nginx/nginx.conf

## Remove default nginx index page
RUN rm -rf /usr/share/nginx/html/*

COPY --from=build /git-template-test/build /usr/share/nginx/html

CMD ["nginx", "-g", "daemon off;"]
