
# Рекомендации по переводу проектов на SpringBoot и JDK11
peristance.xml: убрать xmlns,оставить `<persistence version="2.2">`, иначе ругается  eclipselink-maven-plugin

xxx-api удалить профиль site
```xml
        <profile>
            <id>site</id>
```            
xxx-web

если есть профиль `<id>server</id>`, все перенести в основную ветку без профиля

удалить плагины
`maven-dependency-plugin`, `maven-war-plugin` (переносятся в профиль jdk-8)

javaee-web-api версия 8.0
`javax.ws.rs-api` версия 2.1.1

убрать dependency  
1. `xxx-api` с  `<classifier>javadoc</classifier>`
2. `eclipselink`
3. `com.google.guava` 


убрать профили

1. java2wadl (статическая генерация wadl из java классов, убрать если не используется)
2. standalone
3. java2swagger - устарело, генеряется по новому в xxx-api
4. swaggerclient - устарело, генеряется по новому  генеряется в xxx-api
5. schemagen - первичная генерация схем из модели данных. можно выкинуть если не используется
6. modelgen - генерация мета-модели по JPA сущностям, у нас не прижилось
7. site - если используется, возможно следует заменить на ком.строку
8. static-aspectj - замена на профили для jdk 8 и jdk 11

предпочтительно включить профиль static-weaving, если не включен, настроить по инструкции внутри модуля
                    

убрать весию в зависимостях spring, junit и др.
`<version>${org.springframework.version}</version>`

netbeans пишет, что будет использована managed version - нам подойдет



добавить 
```
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring.boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```    
    
```    
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>${spring.boot.version}</version>
            </plugin>    
```

```
<dependencies>
...
        <dependency>
            <groupId>org.apache.cxf</groupId>
            <artifactId>cxf-spring-boot-starter-jaxrs</artifactId>
            <version>${apache-cxf.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-jdbc</artifactId>
        </dependency>

           <dependency>
            <groupId>javax.xml.bind</groupId>
            <artifactId>jaxb-api</artifactId>
        </dependency>
 ```
    
```         
        <profile>
            <id>static-weaving</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <build>
                <plugins>
                    <!-- Static weaver for EclipseLink
                    1. comment <context:load-time-weaver /> [beans.xml]                    2. uncomment <property name="eclipselink.weaving" value="static" /> [persistance.xml]
                    3. switch activeByDefault=true
                    -->
                    <plugin>
                        <groupId>com.ethlo.persistence.tools</groupId>
                        <artifactId>eclipselink-maven-plugin</artifactId>
                        <version>2.7.4.1</version>
                        <executions>
                            <execution>
                                <phase>process-classes</phase>
                                <goals>
                                    <goal>weave</goal>
                                </goals>
                            </execution>
                        </executions>
                        <dependencies>
                            <dependency>
                                <groupId>org.glassfish.jaxb</groupId>
                                <artifactId>jaxb-runtime</artifactId>
                                <version>2.3.1</version>
                            </dependency>
                        </dependencies>
                    </plugin>
                </plugins>
            </build>
        </profile>
            
        <profile>
            <id>java-8</id>
            <activation>
                <jdk>1.8</jdk>
            </activation>
            <build>
                <plugins>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-war-plugin</artifactId>
                        <version>3.2.2</version>
                        <configuration>
                            <packagingExcludes>WEB-INF/lib/org.eclipse.persistence*.jar,WEB-INF/lib/javax.persistence-*.jar,WEB-INF/lib/jakarta.persistence-*.jar,WEB-INF/lib/mysql-connector-java-*.jar</packagingExcludes>
                        </configuration>
                    </plugin>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-dependency-plugin</artifactId>
                        <version>2.10</version>
                        <executions>
                            <execution>
                                <id>unpack</id>
                                <phase>validate</phase>
                                <goals>
                                    <goal>unpack</goal>
                                </goals>
                                <configuration>
                                    <artifactItems>
                                        <!-- packaging org.eclipse.persistence.jaxb.rs.MOXyJsonProvider required becouse of javax.ws.rs-api conflicts -->
                                        <artifactItem>
                                            <groupId>org.eclipse.persistence</groupId>
                                            <artifactId>org.eclipse.persistence.moxy</artifactId>
                                            <version>${eclipselink.version}</version>
                                            <type>jar</type>
                                            <overWrite>false</overWrite>
                                            <outputDirectory>${project.build.directory}/classes</outputDirectory>
                                            <includes>org/eclipse/persistence/jaxb/rs/*.class</includes>
                                        </artifactItem>
                                    </artifactItems>
                                </configuration>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
        </profile>
        <profile>
            <id>static-weaving-jdk8</id>
            <activation>
                <jdk>1.8</jdk>
            </activation>
            <build>
                <plugins>
                    <plugin>
                        <groupId>org.codehaus.mojo</groupId>
                        <artifactId>aspectj-maven-plugin</artifactId>
                        <version>1.11</version>
                        <configuration>
                            <complianceLevel>${maven.compiler.source}</complianceLevel>
                            <source>${maven.compiler.source}</source>
                            <target>${maven.compiler.target}</target>
                            <xmlConfigured>src/main/resources/META-INF/aop.xml</xmlConfigured>
                            <aspectLibraries>
                                <aspectLibrary>
                                    <groupId>org.springframework</groupId>
                                    <artifactId>spring-aspects</artifactId>
                                </aspectLibrary>
                                <!--                                <aspectLibrary>
                                    <groupId>com.jcabi</groupId>
                                    <artifactId>jcabi-aspects</artifactId>
                                </aspectLibrary>
                                <aspectLibrary>
                                    <groupId>ru.ilb.aspectlock</groupId>
                                    <artifactId>redisson-aspect-lock</artifactId>
                                </aspectLibrary>-->
                            </aspectLibraries>
                            <verbose>true</verbose>
                            <showWeaveInfo>true</showWeaveInfo>
                            <privateScope>true</privateScope>
                        </configuration>
                        <executions>
                            <execution>
                                <goals>
                                    <goal>compile</goal>
                                    <goal>test-compile</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
        </profile>
        <profile>
            <id>static-weaving-jdk11</id>
            <activation>
                <jdk>11</jdk>
            </activation>
            <build>
                <plugins>
                    <plugin>
                        <!-- https://github.com/mojohaus/aspectj-maven-plugin/pull/45 -->
                        <groupId>com.nickwongdev</groupId>
                        <artifactId>aspectj-maven-plugin</artifactId>
                        <version>1.12.1</version>
                        <configuration>
                            <complianceLevel>${maven.compiler.source}</complianceLevel>
                            <source>${maven.compiler.source}</source>
                            <target>${maven.compiler.target}</target>
                            <xmlConfigured>src/main/resources/META-INF/aop.xml</xmlConfigured>
                            <aspectLibraries>
                                <aspectLibrary>
                                    <groupId>org.springframework</groupId>
                                    <artifactId>spring-aspects</artifactId>
                                </aspectLibrary>
                                <!--                                <aspectLibrary>
                                    <groupId>com.jcabi</groupId>
                                    <artifactId>jcabi-aspects</artifactId>
                                </aspectLibrary>
                                <aspectLibrary>
                                    <groupId>ru.ilb.aspectlock</groupId>
                                    <artifactId>redisson-aspect-lock</artifactId>
                                </aspectLibrary>-->
                            </aspectLibraries>
                            <verbose>true</verbose>
                            <showWeaveInfo>true</showWeaveInfo>
                            <privateScope>true</privateScope>
                        </configuration>
                        <executions>
                            <execution>
                                <goals>
                                    <goal>compile</goal>
                                    <goal>test-compile</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
        </profile>
```            

            
            
beans.xml - если приято решение оставить

cacheBuilder, cacheManager от guava заменить на
`<bean id="cacheManager" class="org.springframework.cache.caffeine.CaffeineCacheManager"/>`



            
            
            
Spring Boot

Настроить файлы resources:

1. application.properties
2. swagger.properties


Настроить файлы classes: 
1. Application
2. ServletInitializer


Настроить файл web.xml

закомментировть 

1. context-param
2. listener
3. servlet
4. servlet-mapping


beans.xml
`<context:spring-configured/>` заменяется на `@EnableSpringConfigured`


`<jaxrs-client:client` заменяется на `@EnableJaxRsProxyClient`
http://cxf.apache.org/docs/jaxrsclientspringboot.html



cors
https://cxf.apache.org/docs/jax-rs-cors.html

`task:scheduled-tasks` на `@EnableScheduling` и `@Scheduled(cron = "${scheduleImportItemsSource}")`
