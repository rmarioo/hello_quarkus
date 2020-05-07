from https://blog.doubleslash.de/how-to-deploy-a-native-quarkus-application-on-heroku/

How to deploy a (native) Quarkus Application on Heroku

mvn io.quarkus:quarkus-maven-plugin:1.0.1.Final:create \
    -DprojectGroupId=org.acme \
    -DprojectArtifactId=hello-quarkus \
    -DclassName="org.acme.quickstart.GreetingResource" \
    -Dpath="/hello"
	
	
Test:
 cd hello-quarkus
$ mvn quarkus:dev

curl localhost:8080/hello

CTRL-C to quit

Build native image 
mvn package -Pnative -Dquarkus.native.container-build=true

Modify src/main/docker/Dockerfile.native

in this way

FROM registry.access.redhat.com/ubi8/ubi-minimal
WORKDIR /work/
COPY target/*-runner /work/application
RUN chmod 775 /work
CMD ./application -Dquarkus.http.host=0.0.0.0 -Dquarkus.http.port=${PORT}	
	
	
Build docker image 
docker build -f src/main/docker/Dockerfile.native -t quarkus/hello-quarkus .

Run with docker

docker run -i --rm --name hello_quarkus --env PORT=8081 -p 8081:8081 quarkus/hello-quarkus

Test 
curl localhost:8081/hello


Deploying the app to Heroku  ( i used myplaygroundapp as  <app-name>)

heroku create <app-name> --region eu
heroku container:login
docker tag quarkus/hello-quarkus registry.heroku.com/<app-name>/web
docker push registry.heroku.com/<app-name>/web
heroku container:release web -a <app-name>
curl https://<app-name>.herokuapp.com/hello
