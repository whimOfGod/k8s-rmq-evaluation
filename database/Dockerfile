FROM library/postgres:16-alpine

ENV POSTGRES_HOST_AUTH_METHOD trust

RUN chmod +x /docker-entrypoint-initdb.d/init-databases.sh
RUN echo "host all  all    0.0.0.0/0  md5" >> pg_hba.conf
RUN echo "listen_addresses='*'" >> postgresql.conf

EXPOSE 5432

VOLUME  ["/etc/postgresql", "/var/log/postgresql", "/var/lib/postgresql"]
