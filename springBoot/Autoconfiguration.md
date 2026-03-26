### @SpringbootApplication -> configuration + EnableAutoConfigration + componentScan 
- it tell the sping that autoconfiguret the wholeapplication based on the classpath(datasource,MVC,jackson and scan Application for beans.

  
### uscases
1. @EnableAutoConfiguration(exclude = { CassendraAutoconfiguration.class ...}) keep the defualt autoconfigurtion enabled for everything except three cassendra related auto config classes
- to avoid the unneccesssary cost of opening connectoi for the cassendra service
  
2. this class only loaded when the elasticsearch=false . it disble all the elasticsearch related auto-configured classess.
- allow same code base to run in local dev where elastic search is not availablewithout failing on missing beans.
  @CondiftionalOnProprty(name="elasticsearch.enabled", havingvalue="false")
  @EnableAutoConfiguration(exclude={ElasticSearchDataSourceConfiguration.class ...)
  

3. when elasticsearch is enable then it enable the elasticsearch data search repo in application
 @CondiftionalOnProprty(name="elasticsearch.enabled", havingvalue="true")
@EnableElasticSearchRepossitories()
   
  
  

