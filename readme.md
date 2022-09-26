# ⭐ Spring Cloud Config Server (https://spring.io/projects/spring-cloud-config/)
스프링 클라우드 컨피그 서버란 다양한 벡엔드와 함께 일반적인 구성 관리 솔루션을 제공하는 오픈 소스 프로젝트이며, Git, Eureka, Consul 같은 백엔드와 통합 가능하다.

<br/> 

## 📋 spring-cloud-starter-config 라이브러리 추가

### ✏ 라이브러리
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.thoughtmechanix</groupId>
    <artifactId>licensing-service</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>Eagle Eye Licensing Service</name>
    <description>Licensing Service</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.4</version>
    </parent>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Finchley.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-client</artifactId>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
        </dependency>

        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <version>42.2.4</version>
            <!-- <version>9.4-1206-jdbc42</version> -->
            <!-- <version>9.4-1206-jdbc42</version> -->
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-rsa</artifactId>
        </dependency>
    </dependencies>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <start-class>com.thoughtmechanix.licenses.Application</start-class>
        <docker.image.name>johncarnell/tmx-licensing-service</docker.image.name>
        <docker.image.tag>chapter3</docker.image.tag>
    </properties>

    <build>
        <plugins>
            <!-- We use the Resources plugin to filer Dockerfile and run.sh, it inserts actual JAR filename -->
            <!-- The final Dockerfile will be created in target/dockerfile/Dockerfile -->
            <plugin>
                <artifactId>maven-resources-plugin</artifactId>
                <executions>
                    <execution>
                        <id>copy-resources</id>
                        <!-- here the phase you need -->
                        <phase>validate</phase>
                        <goals>
                            <goal>copy-resources</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>${basedir}/target/dockerfile</outputDirectory>
                            <resources>
                                <resource>
                                    <directory>src/main/docker</directory>
                                    <filtering>true</filtering>
                                </resource>
                            </resources>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>1.1.1</version>
                <configuration>
                    <imageName>${docker.image.name}:${docker.image.tag}</imageName>
                    <dockerDirectory>${basedir}/target/dockerfile</dockerDirectory>
                    <resources>
                        <resource>
                            <targetPath>/</targetPath>
                            <directory>${project.build.directory}</directory>
                            <include>${project.build.finalName}.jar</include>
                        </resource>
                    </resources>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

<br/>

| 키워드                                                          | 설명                        |
|:-------------------------------------------------------------|:--------------------------|
| <version>2.7.4</version>                                     | 사용할 스프링 부트 버전             |
| spring-cloud-dependencies                                    | 사용할 스프링 클라우드 버전           |
| spring-cloud-config-server <br/> spring-cloud-starter-config | 이 서비스에 사용할 스프링 클라우드 프로젝트들 |
| com.thoughtmechanix.licenses.Application                                                             | 컨피그 서버용 부트스트랩 클래스         |

<br/>

컨피그 서버를 동작하게 하려면 여러 파일을 설정해야 함.
예를 들어 application.yml 파일에는 Spring Cloud Config Service가 수신 대기할 포트, 구성 데이터를 제공하는 벡엔드 위치 등의 정보를 명시

#### ✏️ confsvr/../resources/application.yml
```yaml
server:
  port: 8888
spring:
  cloud:
    config:
      server:
        # encrypt.enabled should moved to bootstrap.yml
        # encrypt.enabled: false
        git:
          uri: https://github.com/klimtever/config-repo/
          searchPaths: licensingservice,organizationservice
          username: native-cloud-apps
          password: 0ffended
```

<br/>
<br/>
<br/>

## 📋 구성 데이터를 보관할 백엔드 저장소 지정
빌드한 라이선싱 서비스를 Spring Cloud Config를 사용하는데 활용할 것인데, 간결하게 만들기 위해 로컬, 개발, 운영 환경을 위한 구성 데이터를 설정해보자

<br/>

### ✔ 각 환경에 두 가지 구성 프로퍼티 설정
> - 라이선싱 서비스가 직접 사용할 예제 프로퍼티
> - 라이선싱 서비스의 데이터가 저장될 Postgres 데이터베이스를 위한 데이터베이스 구성

Config Service를 구축하면 그 환경에 실행되는 마이크로서비스가 추가되며, 
구축되고 나서 서비스 콘텐츠는 HTTP/REST 엔드포인트로 액세스할 수 있음.

![](readmefile/img.png)

<br/>

애플리케이션 구성 파일의 명명 규칙은 appname-env.yml이다. 환경 이름은 URL에 그대로 반환되어 구성 정보를 조회하는 데 사용됨.

![](readmefile/img_1.png)

#### ✏ licensingservice.yml
```yaml
example.property: "I AM IN THE DEFAULT"
spring.jpa.database: "POSTGRESQL"
spring.datasource.platform:  "postgres"
spring.jpa.show-sql: "true"
spring.database.driverClassName: "org.postgresql.Driver"
spring.datasource.url: "jdbc:postgresql://database:5432/eagle_eye_local"
spring.datasource.username: "postgres"
spring.datasource.password: "{cipher}4788dfe1ccbe6485934aec2ffeddb06163ea3d616df5fd75be96aadd4df1da91"
spring.datasource.testWhileIdle: "true"
spring.datasource.validationQuery: "SELECT 1"
spring.jpa.properties.hibernate.dialect: "org.hibernate.dialect.PostgreSQLDialect"
redis.server: "redis"
redis.port: "6379"
signing.key: "345345fsdfsf5345"
```

















































