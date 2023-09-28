### Why Migrate Application to Liberty 

 
#### We tend to migrate application to Liberty due to the following reasons. 
 
 
1. Even with the highest version of WAS, it has support only up to Java 8. Far I know IBM have no plan to enhance it further.  So, if Java 8 becomes out of complaint, then eventually you may need to choose another app server. 

Refer here -> https://www.ibm.com/support/pages/verify-java-sdk-version-shipped-ibm-websphere-application-server-fix-packs. 
	
	
2. Please be noted that WAS is not supported in AKS architecture, I can be used only in traditional VM.   

Refer here for more info -> https://learn.microsoft.com/en-us/azure/developer/java/ee/websphere-family 
 
![image](https://github.com/mjameer/LibertyMigrationSteps/assets/11364104/dc84920e-4547-4644-9ff3-9fdf1f0297b3)

3. IBM® WebSphere® Liberty is a modern, and ideal for building new cloud-native applications and modernizing existing applications. Liberty allows you to take advantage of cloud-centric environments and support for serverless or traditional deployments, in the cloud. So, in future, it’s easy to migrate an application hosted in Liberty to Azure.

4. IBM liberty is compatible with Oracle/Open and IBM JDK and can support up to Java 17. 

5. Finally Liberty eliminates the need of NDM server. 

##### Additional Reference 

https://www.ibm.com/garage/method/practices/learn/ibm-transformation-advisor/

https://ac-gm-static-files-server.lahgrqm5xee.au-syd.codeengine.appdomain.cloud/cloud/architecture/files/app-modernization-field-guide.pdf


### LibertyMigrationSteps


If you are planning to migrate to Liberty, then following are its steps
 
1.	Mavenize the application, test it end to end if its working in existing WAS.

   
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
Sample - http://www.setgetweb.com/p/WAS85x/xx30.html 

4.	Run the app using liberty and test them
 
####### To Start Application 

```
clean install liberty:run
```
 
####### To Stop Application

```
clean install liberty:stop
```

