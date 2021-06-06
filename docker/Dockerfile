FROM python:alpine

ENV SPARK_VERSION=3.0.0
ENV HADOOP_VERSION=2.7

RUN apk add --no-cache --virtual=.build-dependencies wget ca-certificates && \
    apk add --no-cache bash curl openjdk8-jre

#RUN ln -s /usr/lib/jvm/java-8-openjdk-amd64 /usr/lib/jvm/default-java

RUN wget https://archive.apache.org/dist/spark/spark-${SPARK_VERSION}/spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz  -O /tmp/spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz && \
    wget https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-aws/2.7.4/hadoop-aws-2.7.4.jar -O /tmp/hadoop-aws-2.7.4.jar && \
    wget https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk/1.11.1030/aws-java-sdk-1.11.1030.jar -O /tmp/aws-java-sdk-1.11.1030.jar && \
    wget https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-azure/2.7.4/hadoop-azure-2.7.4.jar -O /tmp/hadoop-azure-2.7.4.jar
#COPY conf/hadoopsparkvars.sh /tmp/hadoopsparkvars.sh
#COPY conf/spark-defaults.conf /tmp/spark-defaults.conf
#COPY conf/spark-env.sh /tmp/spark-env.sh

# Copy hadoopsparkvars.sh to /etc/profile.d
#RUN cp /tmp/hadoopsparkvars.sh /etc/profile.d
#RUN chmod 755 /etc/profile.d/hadoopsparkvars.sh

# Install Spark
RUN tar -zxf /tmp/spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz -C /tmp && \
    mv /tmp/spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION} /usr/local && \
    ln -s /usr/local/spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION} /usr/local/spark && \
    mv /tmp/hadoop-aws-2.7.4.jar /usr/local/spark/jars && \
    mv /tmp/aws-java-sdk-1.11.1030.jar /usr/local/spark/jars && \
    mv /tmp/hadoop-azure-2.7.4.jar /usr/local/spark/jars

# Copy Spark config files
#RUN cp /tmp/spark-defaults.conf /usr/local/spark/conf/spark-defaults.conf && \
#    cp /tmp/spark-env.sh /usr/local/spark/conf/spark-env.sh && \
#    chmod 755 /usr/local/spark/conf/spark-env.sh && \
#    mv /usr/local/spark/conf/log4j.properties.template /usr/local/spark/conf/l$

# Create the Spark working directories
RUN mkdir -p /srv/spark
#sudo chown -R spark:bigdata /srv/spark
RUN chmod 775 /srv/spark
RUN mkdir -p /srv/spark/tmp
RUN chmod 4755 /srv/spark/tmp