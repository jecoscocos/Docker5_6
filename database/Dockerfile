FROM postgres:15

COPY init.sql /docker-entrypoint-initdb.d/

# Используем аргументы сборки
ARG POSTGRES_DB
ARG POSTGRES_USER
ARG POSTGRES_PASSWORD

# Устанавливаем переменные окружения из аргументов
ENV POSTGRES_DB=$POSTGRES_DB
ENV POSTGRES_USER=$POSTGRES_USER
ENV POSTGRES_PASSWORD=$POSTGRES_PASSWORD

EXPOSE 5432 