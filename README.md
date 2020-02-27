
Quarkus is not handling annontation @Interceptors properly when 1 or more classes
are declared in it.

There is a common thread between this issue and 
  - https://github.com/quarkusio/quarkus/issues/7292 Quarkus is not handling 
        annontation @JsonSubTypes properly
  - https://github.com/quarkusio/quarkus/issues/7293 Quarkus is not handling 
        annontation @JsonView properly

All three of these are annotations which allow 1 or more classes to be declared
within them.  The declared classes are not handled properly.

In standard resteasy testing which uses wildfly, application of the classes 
specified in annotations @JsonSubTypes and @JsonView are handled by 
org.eclipse.yasson.  My guess is @Interceptors are similarly handled.
Quarkas is not handling this.


This test has many supporting classes.  The classes of interest are
  InterceptorTest
  InterceptorBookReader
  InterceptorOne
  InterceptorTwo
  InterceptorResource
  InterceptorVisitList    ;; maintains a static array which is referenced in this test

-------------------------


Before running the test install this archive in your local repo.
Here is the cmd to do it.

mvn install:install-file \
   -Dfile=___FIX_THIS___/lib/utils-arquillian-utils-4.5.0-SNAPSHOT.jar \
   -DgroupId=org.jboss.resteasy \
   -DartifactId=utils-arquillian-utils \
   -Dversion=4.5.0-SNAPSHOT \
   -Dpackaging=jar 

Execution env.
    jdk 11.0.2
    mvn 3.6.0
    quarkus 999-SNAPSHOT
    
 
Set breakpoints in 
  InterceptorVisitList line 9
  InterceptorTest line 104, 111, 119
  InterceptorResource 89, 148

run test

InterceptorTest line 104
  classes with @Interceptor annotation and no class declared in it are
  registered with InterceptorVisitList

InterceptorTest line  111  same as above

InterceptorTest line  119   InterceptorResource endpoint test() (line 88) is
executed.  The count of expectedList entries is wrong because classes
InterceptorOne and  InterceptorTwo have not been add to the list as expected.
 

  
  Compile project:
    mvn clean test -DskipTests=true
  
  Run tests
    mvn test
    or
    mvn test -Dmaven.surefire.debug
  