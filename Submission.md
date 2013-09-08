## Which Categories Best Fit Your Submission and Why?
I would describe that the submission that I have done classifies both as a new feature and a quality related.
The new feature that I have added is a Arquillian extension that integrates with the Karyon.

## Describe your Submission
Basically I have done an JBoss Arquillian extension particular for the Netflix Karyon framework. Arquillian is an testing framework that allows for executing the tests (JUnit/TestNG) directly in the application container, such as servlet container or full JEE application server. More information can be found here: http://arquillian.org/.
The proposed by my patches solution adds an additional karyon-extension-testsuite that allows to plug in into the karyon lifecycle and the created the created Guice set up and use that to inject the Karyon autoscanned component directly into the test case. As already mentioned, the Karyon is a web framework and at the moment it requires a fully featured servlet container in order to bootstrap itself, this were Arquillian comes in handy. Arquillian is responsibility starts from creating the test deployment, and actually deploy that to the configured test container and finally on executing the tests. The tests are actually being executed in the container itself, so there allow to gain full access to the web application state.

The last thing I have done, based on our discussion: https://github.com/Netflix/karyon/pull/46#issuecomment-22710564. Is proposing a new functionality in the Google Guice, that would later on could be used with the Karyon extension for overriding the Guice bindings at the runtime - for instance to override the components with a mocks.
I have proposed the patch to the Google Guice directly: https://code.google.com/p/google-guice/issues/detail?id=770

### Writing Arquillian tests with Karyon Extension Test Suite

Writing the Arquillian test is pretty simple, it requires only setting up the project dependencies and adding additionally:
```
    testCompile 'org.jboss.arquillian.junit:arquillian-junit-container:1.1.0.Final'
    testCompile 'org.jboss.shrinkwrap:shrinkwrap-impl-base:1.1.2'
    testCompile 'org.jboss.shrinkwrap.resolver:shrinkwrap-resolver-impl-maven:2.0.0'
    testCompile 'org.jboss.shrinkwrap.descriptors:shrinkwrap-descriptors-impl-javaee:2.0.0-alpha-3'
```

To run the tests in the container we need to configure proper container on the classpath and add the proper container
adapter. For instance to run the tests on Tomcat, the only thing that need to be done is to add additional dependencies:
```
   testRuntime 'org.jboss.arquillian.container:arquillian-tomcat-embedded-7:1.0.0.CR5'
   testRuntime 'org.apache.tomcat.embed:tomcat-embed-core:7.0.42'
   testRuntime 'org.apache.tomcat.embed:tomcat-embed-logging-juli:7.0.42'
   testRuntime('org.apache.tomcat.embed:tomcat-embed-jasper:7.0.42') {
       exclude group: 'org.eclipse.jdt.core.compiler', module: 'ecj'
   }
```

Arquillian tests does not differ much from writing simple unit tests with JUnit or TestNG. The most important
difference is the fact that each test case is also responsible for creating the test deployment that will be deployed
by the framework on the target container. The Karyon extension adds additional functionality of injecting the Karyon
discovered components directly in the test case:

Example:
```
@RunWith(Arquillian.class)
@RunInKaryon(applicationId = "hello-netflix-oss")
public class HelloworldComponentTest {

    /**
     * Creates the test deployment.
     *
     * @return the test deployment
     */
    @Deployment
    public static Archive createTestArchive() {

        return Deployments.createDeployment();
    }

    /**
     * The injected component instance.
     */
    @Inject
    private HelloworldComponent instance;

    /**
     * Tests the {@link HelloworldComponent#getHelloString()} method.
     */
    @Test
    public void shouldRetrieveMessage() {

        // then
        assertEquals("I am a component", instance.getHelloString());
    }
}
```

### Building
Karyon: karyon-extension-testsuite
The test suite builds together with the karyon project. Simply running:
```
./gradlew clean build
```

Will build the test suite and it's unit tests and afterwards will run the hello-netflix-oss example application that
includes Arquillian integration tests.

Genie: genie-web-int-tests
The Genie integration tests will run with each build:
```
./gradlew clean build
```

## Provide Links to Github Repo's for your Submission
* https://github.com/jmnarloch/karyon
* https://github.com/jmnarloch/genie

You will find the all the changes in separate branches.