FROM docker.elastic.co/elasticsearch/elasticsearch:7.10.2
USER root
RUN bin/elasticsearch-plugin install analysis-kuromoji
RUN bin/elasticsearch-plugin install analysis-icu