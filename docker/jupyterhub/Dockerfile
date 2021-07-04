FROM jupyter/datascience-notebook:ubuntu-20.04

ENV SPARK_VERSION=3.0.0
ENV HADOOP_VERSION=2.7
ENV HADOOP_VERSION_LONG=2.7.4
USER root
RUN sudo apt-get update && \
    DEBIAN_FRONTEND=noninteractive && \ 
    sudo apt-get -y install openjdk-8-jdk


# Install Spark and Hadoop
RUN wget https://archive.apache.org/dist/spark/spark-${SPARK_VERSION}/spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz  -O /tmp/spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz && \
    tar -zxf /tmp/spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz -C /tmp && \
    rm /tmp/spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz && \
    mv /tmp/spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION} /usr/local && \
    ln -s /usr/local/spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION} /usr/local/spark && \
    wget https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-aws/2.7.4/hadoop-aws-2.7.4.jar -O /tmp/hadoop-aws-2.7.4.jar && \
    mv /tmp/hadoop-aws-2.7.4.jar /usr/local/spark/jars && \
    wget https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk/1.7.4/aws-java-sdk-1.7.4.jar -O /tmp/aws-java-sdk-1.7.4.jar && \
    mv /tmp/aws-java-sdk-1.7.4.jar /usr/local/spark/jars && \
    wget https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-azure/2.7.4/hadoop-azure-2.7.4.jar -O /tmp/hadoop-azure-2.7.4.jar && \
    mv /tmp/hadoop-azure-2.7.4.jar /usr/local/spark/jars && \
    wget https://repo1.maven.org/maven2/com/microsoft/azure/azure-storage/8.6.6/azure-storage-8.6.6.jar -O /tmp/azure-storage-8.6.6.jar && \
    mv /tmp/azure-storage-8.6.6.jar  /usr/local/spark/jars

# Copy pyspark kernel
COPY kernel.json /opt/conda/share/jupyter/kernels/pypsark/kernel.json