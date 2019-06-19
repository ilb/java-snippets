# Java pom snippets

## Maven central requirements: name, description, url, licenses, developers, scm, distributionManagement, signature

```xml
    <name>jsonschema</name>
    <description>jsonschema objects</description>
    <url>https://github.com/ilb/jsonschema</url>
    <licenses>
        <license>
            <name>Apache 2.0</name>
            <url>http://www.apache.org/licenses/LICENSE-2.0.txt</url>
            <distribution>repo</distribution>
            <comments>A business-friendly OSS license</comments>
        </license>
    </licenses>
    <developers>
        <developer>
            <name>PJSC "Bystrobank"</name>
            <email>github@bystrobank.ru</email>
        </developer>
    </developers>
    <scm>
        <connection>scm:git:https://github.com/ilb/jsonschema.git</connection>
        <url>https://github.com/ilb/jsonschema</url>
    </scm>
    <distributionManagement>
        <snapshotRepository>
            <id>ossrh</id>
            <name>Sonatype Nexus snapshot repository</name>
            <url>https://oss.sonatype.org/content/repositories/snapshots</url>
        </snapshotRepository>
        <repository>
            <id>ossrh</id>
            <name>Sonatype Nexus release repository</name>
            <url>https://oss.sonatype.org/service/local/staging/deploy/maven2/</url>
        </repository>
    </distributionManagement>
    <build>
        <pluginManagement>
            <plugins>
                <plugin>
                    <artifactId>maven-deploy-plugin</artifactId>
                    <version>2.8.2</version>
                </plugin>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-release-plugin</artifactId>
                    <version>2.5.3</version>
                    <configuration>
                        <releaseProfiles>release-profile</releaseProfiles>
                        <autoVersionSubmodules>true</autoVersionSubmodules>
                    </configuration>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>
    <profiles>
        <profile>
            <id>release-profile</id>
            <build>
                <plugins>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-gpg-plugin</artifactId>
                        <version>1.6</version>
                        <executions>
                            <execution>
                                <id>sign-artifacts</id>
                                <phase>verify</phase>
                                <goals>
                                    <goal>sign</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
        </profile>
    </profiles>    
```
