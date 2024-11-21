FROM node:22.11.0 AS build

WORKDIR /usr/local/app
COPY ./ /usr/local/app/

RUN yarn \
&& npm run build
 
FROM nginx:latest AS production

COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build /usr/local/app/dist/bobapp /usr/share/nginx/html

EXPOSE 80