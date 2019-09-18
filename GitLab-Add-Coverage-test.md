# Mettre la couverture de test sur un projet multi module

C'est une des solutions trouvées qui semble être la plus simple.

Vu sur le site : https://prismoskills.appspot.com/lessons/Maven/Chapter_06_-_Jacoco_report_aggregation.jsp

Projet maven :

Parent Pom
  |
  -- Module 1
  -- Module 2
  -- Module Rapport
  
  
  ## Pom parent
  1. Rajouter  la dependance à Jacoco :
  
    <dependencies>
        <dependency>
            <groupId>org.jacoco</groupId>
            <artifactId>jacoco-maven-plugin</artifactId>
            <version>0.8.4</version>
        </dependency>
    </dependencies>

  2. Rajouter dans la partie build -> plugins :
        <plugin>
            <groupId>org.jacoco</groupId>
            <artifactId>jacoco-maven-plugin</artifactId>
            <executions>
                <execution>
                    <id>prepare-agent</id>
                    <goals>
                        <goal>prepare-agent</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
            
   ## Module 1
        Rien à rajouter
        
   ## Module 2
        Rien à rajouter
        
   ## Module Rapport
   
    1. Rajouter les dependances des autres projets:
     <dependencies>
        <dependency>
            <groupId>GROUP_ID</groupId>
            <artifactId>Module1</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>GROUP_ID</groupId>
            <artifactId>Module2</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
    
    2. Rajouter le plugin Jacoco avec le goal 'report-aggregate'
    <build>
        <plugins>
            <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
                <version>${jacoco-maven-plugin.version}</version>
                <executions>
                    <execution>
                        <id>report-aggregate</id>
                        <phase>verify</phase>
                        <goals>
                            <goal>report-aggregate</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
      
    
   ## Résultat :
   Les couvertures de test de tous les modules sont réunis dans le dossier suivant :
   "<Module Rapport>/target/site/jacoco-aggregate"
   
   
