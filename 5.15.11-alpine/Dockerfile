FROM openjdk:8-jre-alpine

ENV ACTIVEMQ_VERSION 5.15.11
ENV ACTIVEMQ apache-activemq-$ACTIVEMQ_VERSION
ENV ACTIVEMQ_TCP=61616 ACTIVEMQ_AMQP=5672 ACTIVEMQ_STOMP=61613 ACTIVEMQ_MQTT=1883 ACTIVEMQ_WS=61614 ACTIVEMQ_UI=8161
ENV SHA512_VAL=a3ca1a839ddb87eaf7468db1f130a1fe79be6423acc6b059ac2998d3d359b8cc369a45b62322bf29bb031e702113e4d1ac44ee43457a28e7bf22687aa107a37f

ENV ACTIVEMQ_HOME /opt/activemq

RUN set -x && \
    mkdir -p /opt && \
    apk --update add --virtual build-dependencies curl && \
    curl https://archive.apache.org/dist/activemq/$ACTIVEMQ_VERSION/$ACTIVEMQ-bin.tar.gz -o $ACTIVEMQ-bin.tar.gz

# Validate checksum
RUN if [ "$SHA512_VAL" != "$(sha512sum $ACTIVEMQ-bin.tar.gz | awk '{print($1)}')" ];\
    then \
        echo "sha512 values doesn't match! exiting."  && \
        exit 1; \
    fi;

RUN tar xzf $ACTIVEMQ-bin.tar.gz -C  /opt && \
    ln -s /opt/$ACTIVEMQ $ACTIVEMQ_HOME && \
    addgroup -S activemq && adduser -S -H -G activemq -h $ACTIVEMQ_HOME activemq && \
    chown -R activemq:activemq /opt/$ACTIVEMQ && \
    chown -h activemq:activemq $ACTIVEMQ_HOME && \
    apk del build-dependencies && \
    rm -rf /var/cache/apk/* && \
    sed -i -E 's#^([[:blank:]]*<transportConnector name="amqp" uri="amqp://).*("/>)#\1${amqp.transport.connector}\2#' /opt/activemq/conf/activemq.xml && \
    sed -i -E 's#^([[:blank:]]*ACTIVEMQ_OPTS="\$ACTIVEMQ_OPTS_MEMORY -Djava.util.logging.config.file=logging.properties -Djava.security.auth.login.config=\$ACTIVEMQ_CONF/login.config).*(")#\1 -Damqp.transport.connector=\\"${AMQP_TRANSP_CONN}\\"\2#' /opt/activemq/bin/env

USER activemq

WORKDIR $ACTIVEMQ_HOME
EXPOSE $ACTIVEMQ_TCP $ACTIVEMQ_AMQP $ACTIVEMQ_STOMP $ACTIVEMQ_MQTT $ACTIVEMQ_WS $ACTIVEMQ_UI

CMD ["/bin/sh", "-c", "bin/activemq console"]
