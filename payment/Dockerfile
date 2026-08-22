# FROM python:3.9
# EXPOSE 8080
# ENV AMQP_USER=roboshop \
#     AMQP_PASS=roboshop123

# WORKDIR /app 
# COPY requirements.txt /app 
# RUN pip install -r requirements.txt
# COPY *.py /app/
# COPY payment.ini /app/ 
# CMD ["uwsgi", "--ini", "payment.ini"]

FROM python:3.9.25-alpine3.22 AS builder
WORKDIR /app
RUN apk add --no-cache build-base linux-headers pcre-dev
COPY requirements.txt /app
RUN pip3.9 install --prefix=/install -r requirements.txt

FROM python:3.9.25-alpine3.22
EXPOSE 8080
WORKDIR /app
RUN apk add --no-cache pcre
RUN addgroup -S roboshop && adduser -S roboshop -G roboshop
RUN chown -R roboshop:roboshop /app
COPY --from=builder /install /usr/local
COPY --chown=roboshop:roboshop payment.ini *.py requirements.txt /app
USER roboshop
CMD ["uwsgi", "--ini", "payment.ini"]
