# See CKAN docs on installation from Docker Compose on usage
FROM debian:buster
MAINTAINER Marsdd

# Install required system packages
RUN apt-get -q -y update \
    && DEBIAN_FRONTEND=noninteractive apt-get -q -y upgrade \
    && apt-get -q -y install \
        python-dev \
        python-pip \
        python-virtualenv \
        python-wheel \
        python3-dev \
        python3-pip \
        python3-virtualenv \
        python3-wheel \
        libpq-dev \
        libxml2-dev \
        libxslt-dev \
        libgeos-dev \
        libssl-dev \
        libffi-dev \
        postgresql-client \
        build-essential \
        git-core \
        vim \
        wget \
        curl \
    && apt-get -q clean \
    && rm -rf /var/lib/apt/lists/*

RUN apt-get update && apt-get install -y redis-tools

# Define environment variables
ENV CKAN_HOME /usr/lib/ckan
ENV CKAN_VENV $CKAN_HOME/venv
ENV CKAN_CONFIG /etc/ckan
ENV CKAN_STORAGE_PATH=/var/lib/ckan
ENV CKAN_INI=/etc/ckan/production.ini

# Build-time variables specified by docker-compose.yml / .env
ARG CKAN_SITE_URL
ARG MARS_PLUGIN_VERSION
ARG MARS_MULTILANG_PLUGIN_VERSION
ARG GAID

# Create ckan user
RUN useradd -r -u 900 -m -c "ckan account" -d $CKAN_HOME -s /bin/false ckan

# Setup virtual environment for CKAN
RUN mkdir -p $CKAN_VENV $CKAN_CONFIG $CKAN_STORAGE_PATH && \
  virtualenv $CKAN_VENV && \
  ln -s $CKAN_VENV/bin/pip /usr/local/bin/ckan-pip &&\
  ln -s $CKAN_VENV/bin/paster /usr/local/bin/ckan-paster &&\
  ln -s $CKAN_VENV/bin/ckan /usr/local/bin/ckan

RUN ckan-pip install -e git+https://github.com/ckan/ckanext-googleanalytics.git#egg=ckanext-googleanalytics && \
  ckan-pip install -r $CKAN_VENV/src/ckanext-googleanalytics/requirements.txt

ADD ./requirement-setuptools.txt /ckan-temp/requirement-setuptools.txt
ADD ./requirements-py2.txt /ckan-temp/requirements-py2.txt
RUN ckan-pip install -U pip && \
  ckan-pip install --upgrade --no-cache-dir -r /ckan-temp/requirement-setuptools.txt && \
  ckan-pip install --upgrade --no-cache-dir -r /ckan-temp/requirements-py2.txt

RUN rm -Rf /ckan-temp
# Setup CKAN
# I didn't add ckanext-pages to this because it uses different mechanism for database updates.  I prefer it to use
# same alembic, but it doesn't do that, it does it live as it's loading the module for the first time, it works, but not the best.
# this should change.
ADD . $CKAN_VENV/src/ckan/
RUN ckan-pip install -e $CKAN_VENV/src/ckan/ && \
  ckan-pip install https://github.com/marsdd/ckanext-marsavin/archive/$MARS_PLUGIN_VERSION.zip && \
  ckan-pip install https://github.com/marsdd/ckanext-multilang/archive/$MARS_MULTILANG_PLUGIN_VERSION.zip && \
  ln -s $CKAN_VENV/src/ckan/ckan/config/who.ini $CKAN_CONFIG/who.ini && \
  cp -v $CKAN_VENV/src/ckan/contrib/docker/ckan-bg/ckan-entrypoint.sh /ckan-entrypoint.sh && \
  chmod +x /ckan-entrypoint.sh && \
  chown -R ckan:ckan $CKAN_HOME $CKAN_VENV $CKAN_CONFIG $CKAN_STORAGE_PATH

RUN ckan generate config "$CKAN_INI"

# m.m. - replace Google Analytics ID
RUN sed -i "s/GAID/$GAID/g" "$CKAN_INI"

ENTRYPOINT ["/ckan-entrypoint.sh"]

USER ckan

CMD ["ckan","jobs","worker"]
