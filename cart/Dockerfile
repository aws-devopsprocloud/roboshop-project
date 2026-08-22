# FROM node:20
# EXPOSE 8080 
# WORKDIR /opt/server
# ENV REDIS_HOST="redis"
# ENV CATALOGUE_HOST="catalogue"
# ENV CATALOGUE_PORT="8080"
# COPY server.js /opt/server
# COPY package.json /opt/server
# RUN npm install 
# CMD ["node", "server.js"]

FROM node:20.20.2-alpine3.23 AS builder
WORKDIR /app
COPY package.json .
COPY *.js .
RUN npm install


FROM node:20.20.2-alpine3.23
WORKDIR /app
EXPOSE 8080
COPY --from=builder /app /app 
# ENV REDIS_HOST="redis" \
#     CATALOGUE_HOST="catalogue" \
#     CATALOGUE_PORT="8080"
RUN addgroup -S roboshop && adduser -S roboshop -G roboshop
RUN chown -R roboshop:roboshop /app
USER roboshop
CMD ["server.js"]
ENTRYPOINT ["node"]