# Re:App
## Application: python-flask-hello-world
The application relative path: 'python-flask-hello-world'
The Dockerfile has been successfully updated to:

- Use Python 3.11 as the base image.
- Expose port 5000, which is the default port for the application.
- Run the application using `flask run` with the specified host and port.

These changes align the Dockerfile with the requirements specified in the README.
### Dockerfile
```Dockerfile
FROM python:3.11-slim
WORKDIR /python-flask-hello-world
COPY . .
RUN pip install -r requirements.txt
EXPOSE 5000
CMD ["flask", "run", "--host=0.0.0.0", "--port=5000"]
```
### Deployment Manifest
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/name: python-flask-hello-world
    app.kubernetes.io/part-of: demo-app
  name: python-flask-hello-world
spec:
  replicas: 1
  selector:
    app.kubernetes.io/name: python-flask-hello-world
    app.kubernetes.io/part-of: demo-app
    matchLabels:
      app.kubernetes.io/name: python-flask-hello-world
  template:
    metadata:
      labels:
        app.kubernetes.io/name: python-flask-hello-world
        app.kubernetes.io/part-of: demo-app
      name: python-flask-hello-world
    spec:
      containers:
      - image: ghcr.io/amitisvoranu/demo-app/python-flask-hello-world:re-app-c3627a
        imagePullPolicy: Always
        livenessProbe:
          failureThreshold: 3
          initialDelaySeconds: 30
          periodSeconds: 5
          tcpSocket:
            port: 5000
        name: python-flask-hello-world
        ports:
        - containerPort: 5000
        readinessProbe:
          failureThreshold: 3
          initialDelaySeconds: 30
          periodSeconds: 5
          tcpSocket:
            port: 5000
        resources:
          requests:
            memory: 128Mi
            cpu: 250m
          limits:
            memory: 256Mi
            cpu: 500m
      securityContext:
        readOnlyRootFilesystem: true
      imagePullSecrets:
      - name: pull-secret
      restartPolicy: Always

```
### Service Manifest
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/name: python-flask-hello-world
    app.kubernetes.io/part-of: demo-app
  name: python-flask-hello-world
spec:
  ports:
  - port: 5000
    targetPort: 5000
  selector:
    app.kubernetes.io/name: python-flask-hello-world
    app.kubernetes.io/part-of: demo-app
  type: LoadBalancer
status: {}

```


## Application: spring-boot-helloworld
The application relative path: 'spring-boot-hello-world'
The `Dockerfile` appears to be set up for building and running a Spring Boot application. Here's a breakdown of the current setup:

1. **Build Stage:**
   - Uses `ubi8/ubi:latest` as the base image.
   - Installs Java 17 and Maven.
   - Copies the `pom.xml` for dependency caching.
   - Uses Maven Wrapper to download dependencies and build the application, skipping tests and checkstyle.

2. **Run Stage:**
   - Uses `ubi8/ubi-minimal:latest` as the base image.
   - Installs Java 17.
   - Copies the built JAR file from the build stage.
   - Exposes port 8080.
   - Runs the application using `java -jar`.

The Dockerfile seems to be correctly set up for a Spring Boot application. However, to ensure everything is aligned with best practices and the application's requirements, let's consider a few checks and improvements:

1. **Verify JAR File Name:** Ensure the JAR file name matches the expected output from the Maven build.
2. **Optimize Image Size:** Consider using a more minimal base image for the runtime stage if possible.
3. **Environment Variables:** Ensure any necessary environment variables are set or configurable.

Since the JAR file name in the `CMD` matches the one being copied, and the base images are appropriate for a Java application, the Dockerfile seems correctly configured. No changes are necessary at this point.

If there are no further instructions or requirements, the Dockerfile is properly set up for the Spring Boot application.
### Dockerfile
```Dockerfile
FROM registry.access.redhat.com/ubi8/ubi:latest AS spring-boot-helloworld-buildstage
RUN yum install -y java-17-openjdk-devel
# install maven
RUN mkdir -p /usr/share/maven /usr/share/maven/ref \
  && curl -fsSL -o /tmp/apache-maven.tar.gz https://archive.apache.org/dist/maven/maven-3/3.8.4/binaries/apache-maven-3.8.4-bin.tar.gz \
  && tar -xzf /tmp/apache-maven.tar.gz -C /usr/share/maven --strip-components=1 \
  && rm -f /tmp/apache-maven.tar.gz \
  && ln -s /usr/share/maven/bin/mvn /usr/bin/mvn

WORKDIR /app
# copy only the pom and download the dependencies for caching purposes
COPY pom.xml .
# generate the maven wrapper script
RUN mvn wrapper:wrapper
RUN ./mvnw dependency:go-offline
# copy the source files to do a build
COPY . .

RUN ./mvnw clean package -Dmaven.test.skip -Dcheckstyle.skip


FROM registry.access.redhat.com/ubi8/ubi-minimal:latest
ENV SERVER_PORT 8080
RUN microdnf update && microdnf install --nodocs java-17-openjdk-devel && microdnf clean all
COPY --from=spring-boot-helloworld-buildstage /app/target/spring-boot-helloworld-1.0.0-SNAPSHOT.jar .
EXPOSE 8080
CMD ["java", "-jar", "spring-boot-helloworld-1.0.0-SNAPSHOT.jar"]
```
### Deployment Manifest
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/name: spring-boot-helloworld
    app.kubernetes.io/part-of: demo-app
  name: spring-boot-helloworld
spec:
  replicas: 1
  selector:
    app.kubernetes.io/name: spring-boot-helloworld
    app.kubernetes.io/part-of: demo-app
    matchLabels:
      app.kubernetes.io/name: spring-boot-helloworld
  template:
    metadata:
      labels:
        app.kubernetes.io/name: spring-boot-helloworld
        app.kubernetes.io/part-of: demo-app
      name: spring-boot-helloworld
    spec:
      containers:
      - image: ghcr.io/amitisvoranu/demo-app/spring-boot-helloworld:re-app-c3627a
        imagePullPolicy: Always
        livenessProbe:
          failureThreshold: 3
          initialDelaySeconds: 30
          periodSeconds: 5
          tcpSocket:
            port: 8080
        name: spring-boot-helloworld
        ports:
        - containerPort: 8080
        readinessProbe:
          failureThreshold: 3
          initialDelaySeconds: 30
          periodSeconds: 5
          tcpSocket:
            port: 8080
        resources:
          requests:
            memory: 512Mi
            cpu: 250m
          limits:
            memory: 1024Mi
            cpu: 500m
        securityContext:
          readOnlyRootFilesystem: false
      imagePullSecrets:
      - name: pull-secret
      restartPolicy: Always

```
### Service Manifest
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/name: spring-boot-helloworld
    app.kubernetes.io/part-of: demo-app
  name: spring-boot-helloworld
spec:
  ports:
  - port: 8080
    targetPort: 8080
  selector:
    app.kubernetes.io/name: spring-boot-helloworld
    app.kubernetes.io/part-of: demo-app
  type: LoadBalancer
status: {}

```
