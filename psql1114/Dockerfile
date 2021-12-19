# based on the
# https://github.com/sibsau/docker-postgres-pro-1c/blob/master/9.6/Dockerfile  
# & https://infostart.ru/public/1368214/  
# & https://github.com/alexanderfefelov/docker-1c-server

FROM ubuntu:bionic

# explicitly set user/group IDs
RUN groupadd -r postgres --gid=999 && useradd -r -g postgres --uid=999 postgres

# grab gosu for easy step-down from root
RUN set -x apt-get update && apt-get install -y --no-install-recommends gosu && rm -rf /var/lib/apt/lists/* 

# make the "ru_RU.UTF-8" locale so postgres will be utf-8 enabled by default
RUN apt-get update && apt-get install -y locales && rm -rf /var/lib/apt/lists/* \
	&& localedef -i ru_RU -c -f UTF-8 -A /usr/share/locale/locale.alias ru_RU.UTF-8
ENV LANG ru_RU.utf8

RUN mkdir /docker-entrypoint-initdb.d

RUN apt-key adv --fetch-keys http://repo.postgrespro.ru/keys/GPG-KEY-POSTGRESPRO

ENV PG_MAJOR 11.14

RUN echo 'deb http://repo.postgrespro.ru/pg1c-11/ubuntu bionic main' > /etc/apt/sources.list.d/postgrespro-1c.list

RUN apt-get update \
	&& apt-get install -y postgrespro-1c-11-server \
	&& sed -ri 's/#(create_main_cluster) .*$/\1 = false/' /etc/postgresql-common/createcluster.conf \
	&& rm -rf /var/lib/apt/lists/*

# make the sample config easier to munge (and "correct by default")
RUN mv -v /usr/share/postgresql/$PG_MAJOR/postgresql.conf.sample /usr/share/postgresql/ \
	&& ln -sv ../postgresql.conf.sample /usr/share/postgresql/$PG_MAJOR/ \
	&& sed -ri "s!^#?(listen_addresses)\s*=\s*\S+.*!\1 = '*'!" /usr/share/postgresql/postgresql.conf.sample

RUN mkdir -p /var/run/postgresql && chown -R postgres /var/run/postgresql

ENV PATH /usr/lib/postgresql/$PG_MAJOR/bin:$PATH
ENV PGDATA /var/lib/postgresql/data
VOLUME /var/lib/postgresql/data

COPY docker-entrypoint.sh /

ENTRYPOINT ["/docker-entrypoint.sh"]

EXPOSE 5432
CMD ["postgres"]
