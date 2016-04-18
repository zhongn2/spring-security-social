# Download #

spring-security-social is available from the following repositories. Add them to your pom.xml and then add the following dependency:

```
<dependency>
  <groupId>org.springframework.security.social</groupId>
  <artifactId>spring-security-social</artifactId>
  <version>1.0-SNAPSHOT</version>
</dependency>
```

Our corporate maven repository:

```
<repository>
	<id>acoveo</id>
	<url>http://www.acoveo.com/maven2/repo</url>
	<snapshots>
		<enabled>false</enabled>
	</snapshots>
</repository>
<repository>
	<id>acoveo-snapshots</id>
	<url>http://www.acoveo.com/maven2/snapshots</url>
	<releases>
		<enabled>false</enabled>
	</releases>
</repository>
```

## Source code ##
The source code of the project and the example is hosted on
https://acoveo.com/svnroot/business/public/trunk/.

## Example ##
Our example application, which demonstrates how to integrate spring-security-social into a Tapestry web application, is available from
```
https://acoveo.com/svnroot/business/public/trunk/spring-security-social-example
```

Simply check-out the project and run it with the maven jetty plug-in:
```
cd /tmp
svn co https://acoveo.com/svnroot/business/public/trunk/spring-security-social-example
cd spring-security-social-example
mvn jetty:run
```
Then visit http://localhost:8080/newapp/login and click on the Facebook button.

In addition to spring-security-social the example application also integrates spring-security-openid for a complete demonstration.