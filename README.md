### Why Migrate Application to Liberty 

 
#### We tend to migrate applications to Liberty due to the following reasons. 
 
 
1. Even with the highest version of WAS, it has support only up to Java 8. Far I know IBM has no plan to enhance it further.  So, if Java 8 becomes out of complaint, then eventually you may need to choose another app server. 

Refer here -> https://www.ibm.com/support/pages/verify-java-sdk-version-shipped-ibm-websphere-application-server-fix-packs. 
	
	
2. Please note that WAS is not supported in AKS architecture, it can be used only in traditional VM.   

Refer here for more info -> https://learn.microsoft.com/en-us/azure/developer/java/ee/websphere-family 
 
![image](https://github.com/mjameer/LibertyMigrationSteps/assets/11364104/7136d1ba-72ec-4898-83c9-a3927bd64a75)

3. IBM® WebSphere® Liberty is modern and ideal for building new cloud-native applications and modernizing existing applications. Liberty allows you to take advantage of cloud-centric environments and support for serverless or traditional deployments, in the cloud. So, in the future, it’s easy to migrate an application hosted in Liberty to Azure.

4. IBM Liberty is compatible with Oracle/Open and IBM JDK and can support up to Java 17. 

5. Finally Liberty eliminates the need for an NDM server. 

##### Additional Reference 

https://www.ibm.com/garage/method/practices/learn/ibm-transformation-advisor/

https://ac-gm-static-files-server.lahgrqm5xee.au-syd.codeengine.appdomain.cloud/cloud/architecture/files/app-modernization-field-guide.pdf


### LibertyMigrationSteps


If you are planning to migrate to Liberty, then the following are its steps
 
1.	Mavenize the application, and test it end to end if it's working in existing WAS.

   ![image](https://github.com/mjameer/LibertyMigrationSteps/assets/11364104/1a834e0f-a87d-49e1-a72e-7848fc1b9eef)

2. 	⁠Add the following plugin and configuration to your project.   
    
```
      <plugin>
        <groupId>io.openliberty.tools</groupId>
        <artifactId>liberty-maven-plugin</artifactId>
        <version>3.5.1</version>
        <configuration>
          <serverName>${serverName}</serverName>
          <configFile>${basedir}/src/main/liberty/config/server.xml</configFile>
          <bootstrapProperties>
            <default.http.port>${testServerHttpPort}</default.http.port>
            <default.https.port>${testServerHttpsPort}</default.https.port>
            <app.context.root> ApplicationName </app.context.root>
          </bootstrapProperties>
          <include>${packaging.type}</include>
        </configuration>
      </plugin>
```


```
  <properties>
 
    <!-- Liberty configuration -->
    <app.name>${project.name}</app.name>
    <testServerHttpPort>8091</testServerHttpPort>
    <testServerHttpsPort>9445</testServerHttpsPort>
    <warContext>${app.name}</warContext>
    <package.file>${project.build.directory}/${app.name}.zip</package.file>
    <packaging.type>usr</packaging.type>
    <serverName>${app.name}Server</serverName>
  </properties>
  
```

3.	Create server.xml under \src\main\liberty 
a. Sample - http://www.setgetweb.com/p/WAS85x/xx30.html 

b. For Springboot app

```
<server description="Liberty server">
	<!-- Enable features -->
	<featureManager>
		<feature>servlet-3.1</feature>
		<feature>jsp-2.3</feature>
		<feature>jndi-1.0</feature>
		<!-- <feature>jaxrs-2.1</feature> -->
	</featureManager>

	<httpEndpoint httpPort="${default.http.port}"
		httpsPort="${default.https.port}" httpOptionsRef="defaultHttpOptions"
		id="defaultHttpEndpoint" />

	<httpOptions id="defaultHttpOptions"
		maxKeepAliveRequests="-1" />

	<webApplication id="appName" contextRoot="/appName"
		location="appName-0.0.1-SNAPSHOT.war" />

</server>
```

c. MsSQL DB via a non-Spring boot app

```
<server description="appName">
	<featureManager>
		<feature>servlet-3.1</feature>
		<feature>jsp-2.3</feature>
		<feature>jndi-1.0</feature>
		<feature>jdbc-4.0</feature>
	</featureManager>
<logging
traceSpecification="com.ibm.ws.webcontainer*=all:com.ibm.wsspi.webcontainer*=all:HTTPChannel=all :GenericBNF=all:HTTPDispatcher=all"
		traceFileName="trace.log" maxFileSize="20" maxFiles="10"
		traceFormat="BASIC" />
	<httpEndpoint httpPort="${default.http.port}"
		httpsPort="${default.https.port}" id="defaultHttpEndpoint" host="*" />


	<library id="appNameJdbclib">
		<file name="/lib/mssql-jdbc-6.4.0.jre8.jar" />
	</library>

	<dataSource id="appNameDS" jndiName="jdbc/appName_db">
		<jdbcDriver libraryRef="appNameJdbclib" />
		
		<properties.microsoft.sqlserver databaseName="xxxxx" serverName="xxxx.cloud.com" 
			portNumber="0000" user="xxxxx" password="xxxxx" />
	</dataSource>
	<webApplication id="appName" location="appName-0.0.1-SNAPSHOT.war" contextRoot="appName" />
</server>
```

d. MsSQL Sample 2

```
<server description="appNameweb">
	<featureManager>
		<feature>servlet-3.1</feature>
		<feature>jsp-2.3</feature>
		<feature>jndi-1.0</feature>
		<feature>jdbc-4.0</feature>
	</featureManager>
	
	<httpDispatcher invokeFlushAfterService="false" />
	
<logging
traceSpecification="com.ibm.ws.webcontainer*=all:com.ibm.wsspi.webcontainer*=all:HTTPChannel=all :GenericBNF=all:HTTPDispatcher=all"
		traceFileName="trace.log" maxFileSize="20" maxFiles="10"
		traceFormat="BASIC" />

	<httpEndpoint httpPort="${default.http.port}"
		httpsPort="${default.https.port}" httpOptionsRef="defaultHttpOptions"
		id="defaultHttpEndpoint" />


	<library id="appNamewebJdbclib">
		<file name="C:\\lib\\someJar.jar" />
	</library>

	<dataSource id="appNamewebDS" jndiName="jdbc/appNameweb_db">
		<jdbcDriver libraryRef="appNamewebJdbclib" />
		<properties.microsoft.sqlserver
			databaseName="xxxx" serverName="xxxx.cloud.com"
			portNumber="1111" user="xxxx" password="xxxxx" />
	</dataSource>
	
	<webApplication id="appNameweb" location="appNameweb-0.0.1-SNAPSHOT.war"
		contextRoot="appName" />
</server>
```


4.	Run the app using liberty and test them
 
##### To Start Application 

```
clean install liberty:run
```
 
##### To Stop Application

```
clean install liberty:stop
```

