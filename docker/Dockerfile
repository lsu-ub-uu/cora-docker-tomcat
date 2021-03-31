FROM tomcat:9.0.43-jdk15-openjdk-buster

COPY /server.xml /usr/local/tomcat/conf/

COPY /log4j2.properties /usr/local/tomcat/lib/

#https://dzone.com/articles/build-docker-image-from-maven
#To compile and build the Docker image locally:

#mvn clean install 
#To compile, build, and push the image to a remote repo:

#mvn clean deploy -Ddocker.user=<username> -Ddocker.password=<passwd> 
#-Ddocker.url=<docker-registry-url> 