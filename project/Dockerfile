FROM python:3.7.4-alpine3.9
WORKDIR /build
RUN apk update && apk add bash curl wget
RUN curl ${1:-https://s3.amazonaws.com/conceptnet/downloads/2018/edges/conceptnet-assertions-5.6.0.csv.gz} --output conceptnet-assertions.csv.gz
RUN pip3 install nltk
COPY wordnet_conceptnet_to_neo4j.py .
RUN python3 wordnet_conceptnet_to_neo4j.py

FROM openjdk:8-jre-alpine
ENV PATH=$PATH:/neo4j/bin/:.
RUN apk update && apk add bash curl wget
RUN wget http://dist.neo4j.org/neo4j-community-3.5.4-unix.tar.gz
RUN tar -xvf neo4j-community-3.5.4-unix.tar.gz -C .
RUN mv neo4j-community-3.5.4 neo4j
COPY --from=0 /build/neo4j_csv_imports/. /imports/.
RUN neo4j-admin import --database=graph.db --relationships=/imports/relationships.csv --nodes=/imports/words.csv --nodes=/imports/synsets.csv
RUN rm -rf /imports

FROM openjdk:8-jre-alpine
EXPOSE 7474 7473 7687
ENV PATH=$PATH:/neo4j/bin/:.
RUN apk update && apk add bash
COPY --from=1 /neo4j .
CMD neo4j console
