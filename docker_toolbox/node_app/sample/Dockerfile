FROM node:7.5.0

MAINTAINER qh4r

ENV NODE_ENV=production
ENV PORT=3000

COPY . /var/www 
# COPY [FROM] [TO]

WORKDIR /var/www

VOLUME ["/var/www"]
 # this will mount default to /var/www on container

RUN npm install
 # need to run AFTER volume is mounted

EXPOSE ${PORT}
 # expose only allows mapping

ENTRYPOINT ["npm", "start"]
