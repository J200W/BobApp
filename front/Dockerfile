FROM node:latest AS build

WORKDIR /usr/local/app
COPY ./ /usr/local/app/

RUN npm i && npm i -g @angular/cli && npm run build
 
FROM nginx:latest AS production

COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build /usr/local/app/dist/bobapp /usr/share/nginx/html

EXPOSE 80