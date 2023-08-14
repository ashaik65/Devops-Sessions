FROM nginx:latest

RUN apt update -y

RUN apt install -y awscli jq

COPY index.html /usr/share/nginx/html/index.html
