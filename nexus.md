1. download nexus Docker image 
```
docker run -d --name nexus -p 8081:8081 sonatype/nexus3
```

2. Edit Pom.XML file adding the repo :
```
<repository>
   		 <id>maven-releases</id>
   		 <url>http://172.234.178.64:8081/repository/maven-releases/</url>
</repository>
```
3. Add Distribution Managment in pom.xml :
```
<distributionManagement>
  <repository>
    <id>maven-releases</id>
    <!-- CHANGE HERE by your team Nexus server -->
    <url>http://13.126.124.159:8081/repository/maven-releases/</url>
  </repository>
  <snapshotRepository>
    <id>maven-snapshots</id>
    <!-- CHANGE HERE by your team Nexus server -->
    <url>http://13.126.124.159:8081/repository/maven-releases/</url>
  </snapshotRepository>
</distributionManagement>
```
