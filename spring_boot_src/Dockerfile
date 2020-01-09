FROM java:1.8.0_151

ARG JARS_NAME
ARG RUNS_OPTS
ARG RUNS_PROFILES
ENV RUNS $RUNS_OPTS
ENV PROFILES $RUNS_PROFILES
ADD $JARS_NAME /app.jar
RUN bash -c 'touch /app.jar'

ENTRYPOINT [ "sh", "-c", "java $RUNS   -Djava.security.egd=file:/dev/./urandom -jar /app.jar  --spring.profiles.active=$PROFILES" ]
