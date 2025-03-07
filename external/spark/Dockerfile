#
# Copyright (C) 2016-2019 Lightbend Inc. <https://www.lightbend.com>
#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

FROM adoptopenjdk/openjdk8

ARG spark_uid=185

COPY jars /opt/spark/jars
COPY bin /opt/spark/bin
COPY sbin /opt/spark/sbin
COPY kubernetes/dockerfiles/spark/entrypoint.sh /opt/
COPY examples /opt/spark/examples
COPY kubernetes/tests /opt/spark/tests
COPY data /opt/spark/data
COPY metrics.properties /etc/metrics/conf/metrics.properties
COPY prometheus.yaml /etc/metrics/conf/prometheus.yaml

# Before building the docker image, first build and make a Spark distribution following
# the instructions in http://spark.apache.org/docs/latest/building-spark.html.
# If this docker file is being used in the context of building your images from a Spark
# distribution, the docker build command should be invoked from the top level directory
# of the Spark distribution. E.g.:
# docker build -t spark:latest -f kubernetes/dockerfiles/spark/Dockerfile .
ENV TINI_VERSION v0.18.0
RUN set -ex && \
    apt-get update && \
    ln -s /lib /lib64 && \
    apt install -y bash libc6 libpam-modules krb5-user libnss3 && \
    mkdir -p /opt/spark/work-dir && \
    touch /opt/spark/RELEASE && \
    rm /bin/sh && \
    ln -sv /bin/bash /bin/sh && \
    echo "auth required pam_wheel.so use_uid" >> /etc/pam.d/su && \
    chgrp root /etc/passwd && chmod ug+rw /etc/passwd && \
    rm -rf /var/cache/apt/* && \
    curl -L https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini -o /sbin/tini && \
    chmod +x /sbin/tini && \
    groupadd -r cloudflow -g 185 && \
    useradd -u 185 -r -g root -G cloudflow -m -d /home/cloudflow \
    -s /sbin/nologin -c CloudflowUser cloudflow && \
    mkdir -p /prometheus && \
    curl  https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/0.11.0/jmx_prometheus_javaagent-0.11.0.jar -o /prometheus/jmx_prometheus_javaagent-0.11.0.jar && \
    chmod ug+rwX /home/cloudflow && \
    chgrp -R 0 /opt/spark  && \
    chmod -R g=u /opt/spark && \
    chmod -R u+x /opt/spark/bin && \
    chmod -R u+x /opt/spark/sbin && \
    chgrp -R 0 /prometheus  && \
    chmod -R g=u /prometheus && \
    chgrp -R 0 /etc/metrics/conf  && \
    chmod -R g=u /etc/metrics/conf && \
    chown cloudflow:root /opt && \
    chmod 775 /opt


ENV SPARK_HOME /opt/spark

WORKDIR /opt/spark/work-dir
RUN chmod g+w /opt/spark/work-dir

ENTRYPOINT [ "/opt/entrypoint.sh" ]

# Specify the User that the actual main process will run as
USER ${spark_uid}
