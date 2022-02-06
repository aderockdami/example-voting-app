FROM maven:amazoncorretto

WORKDIR /app 

COPY . . 

RUN mvn package -Dskiptest && \ 
    mv target/worker-jar-with-dependencies.jar /run/worker.jar && \
    rm -rf *


CMD java -jar /run/worker.jar
