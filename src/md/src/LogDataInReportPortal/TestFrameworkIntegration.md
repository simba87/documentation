# Test framework integration

## TestNG

TestNG provides support for attaching custom listeners, reporters, annotation transformers and method interceptors to your tests.

### Installation

Add to POM.xml

**dependency**

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ xml
<repositories>
     <repository>
        <snapshots>
          <enabled>false</enabled>
        </snapshots>
        <id>bintray-epam-reportportal</id>
        <name>bintray</name>
        <url>http://dl.bintray.com/epam/reportportal</url>
     </repository>
</repositories>



<dependency>
  <groupId>com.epam.reportportal</groupId>
  <artifactId>agent-java-testng</artifactId>
  <version>3.0.0</version>
</dependency>
<dependency>
  <groupId>com.epam.reportportal</groupId>
  <artifactId>logger-java-logback</artifactId>
  <version>3.0.0</version>
</dependency>
<dependency>
  <groupId>com.epam.reportportal</groupId>
  <artifactId>logger-java-log4j</artifactId>
  <version>3.0.0</version>
</dependency>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

### Install listener

Download package [here](<https://bintray.com/epam/reportportal/agent-java-testng>).
Choose latest version.

By default, TestNG attaches a few basic listeners to generate HTML and XML
reports. For reporting TestNG test events (ie start of test, successful finish
of test, test fail) to Report Portal user should add Report Portal TestNg
listener to run and configure input parameters. Description of listeners input
parameters and how to configure it see “Parameters” in [Configuration section](#documentation/TestNG>configuration).

**Listener class:** *com.epam.reportportal.testng.ReportPortalTestNGListener*

There are several ways how to install listener:

- Maven Surefire plugin

- In TestNG configuration file

- Register listener in code

- Via command line

- Via \@Listeners annotation

- Via ServiceLoader class

> Please note, that listener must be configured in a single place only.
> Configuring multiple listeners will lead to incorrect application behavior.

**Maven Surefire plugin**

Maven surefire plugin allows configuring multiple custom listeners. For logging
to Report Portal user should add Report Portal listener to “Listener” property.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ xml
<plugins>
    [...]
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>2.15</version>
        <configuration>
          <properties>
            <property>
              <name>usedefaultlisteners</name>
              <value>false</value> <!-- disabling default listeners is optional -->
            </property>
            <property>
              <name>listener</name>
              <value>com.epam.reportportal.testng.ReportPortalTestNGListener</value>
            </property>
          </properties>
        </configuration>
      </plugin>
    [...]
</plugins>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

>   If you use maven surefire plugin set report portal listener only in
>   "listener" property in pom.xml.

**Specify listener in testng.xml**

Here is how you can define report portal listener in your testng.xml file.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ xml
<suite>
   
  <listeners>
    <listener class-name="com.example.MyListener" />
    <listener class-name="com.epam.reportportal.testng.ReportPortalTestNGListener" />
  </listeners>
.....
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Custom runner**

TestNG allows users to create own runners. In this case user can instantiate
listener by himself and add it to TestNG object.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ java
ReportPortalTestNGListener listener = new ReportPortalTestNGListener();
     TestNG testNg = new TestNG();
     List<String> lSuites = Lists.newArrayList();
     Collections.addAll(lSuites, “testng_suite.xml”);
     testNg.setTestSuites(lSuites);
     testNg.addListener((Object) listener);
     testNg.run();
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Using command line**

Assuming that you have TestNG and Report Portal client jars in your class path,
you can run TestNG tests with Report Portal listener as follows:

*java org.testng.TestNG testng1.xml –listener
com.epam.reportportal.testng.ReportPortalTestNGListener*

**Using \@Listeners annotation**

Report Portal listener can be set using \@Listener annotation.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ java
@Listeners({ReportPortalTestNGListener.class})
public class FailedParameterizedTest {
…..
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

\@Listeners annotation will apply to your entire suite file, just as if you had
specified it in a testng.xml file. Please do not restrict its scope.

**Using ServiceLoader**

JDK offers a very elegant mechanism to specify implementations of interfaces on
the class path via the
[ServiceLoader](<http://docs.oracle.com/javase/8/docs/api/java/util/ServiceLoader.html>)
class. With ServiceLoader, all you need to do is create a jar file that contains
your listener(s) and a few configuration files, put that jar file on the
classpath when you run TestNG and TestNG will automatically find them. You can
use service loaded mechanism for adding Report Portal listener to test
execution. Please see detailed information here:

<http://testng.org/doc/documentation-main.html#testng-listeners> 5.17.2 -
Specifying listeners with ServiceLoader.

### Configuration

Copy your configuration from UI of Report Portal at [User Profile](<#user-profile>) section

or

In order to start using an agent, user should configure property file
"reportportal.properties" in such format:

**reportportal.properties**

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ java
rp.endpoint = https://rp.epam.com/
rp.username = default
rp.uuid = 8967de3b-fec7-47bb-9dbc-2aa4ceab8b1e
rp.launch = default_TEST_EXAMPLE
rp.project = default_project

## OPTIONAL PARAMETERS
rp.tags = TAG1;TAG2
rp.keystore.resource = reportportal-client-v2.jks
rp.keystore.password = reportportal

rp.batch.size.logs = 5
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


**Parameters**

User should provide next parameters to agent.

| **Parameter**                                 | **Description**      | **Required**|
|-----------------------------------------------|----------------------|-------------|
|rp.enable                                      |Enable/Disable logging to Report Portal: rp.enable=true - enable log to RP server.  Any other value means 'false': rp.enable=false - disable log to RP server.  If parameter is absent in  properties file then automation project results will be posted on RP. |No |
|rp.username                                    |User name |Yes |
|rp.password                                    |User password. **We strongly recommend to use UUID** or separate ReportPortal internal users password here to avoid domain password publishing. |Yes |
|rp.uuid                                        |UUID of user. |Yes |
|rp.endpoint                                    |URL of web service, where requests should be send |Yes |
|rp.launch                                      |The unique name of Launch (Run). Based on that name a history of runs will be created for particular name |Yes |
|rp.project                                     |Project name to identify scope |Yes |
|rp.tags                                        |Set of tags for specifying additional meta information for current launch. Format: tag1;tag2;build:12345-6. Tags should be separated by “;”. There are one special tag- build – it should be used for specification number of build for launch. |No |
|rp.batch.size.logs                             |In order to rise up performance and reduce number of requests to server |Yes |
|rp.keystore.resource                           |Put your JKS file into resources and specify path to it | |
|rp.keystore.password                           |Access password for JKS (certificate storage) package, mentioned above | |
|rp.convertimage                                |Colored log images can be converted to grayscale for reducing image size. Values: ‘true’ – will be converted. Any other value means ‘false’. |No |
|rp.mode                                        |Report Portal provides possibility to specify visibility of executing launch. Currently two modes are supported: DEFAULT  - all users from project can see this launch; DEBUG - only owner can see this launch (in debug sub tab). Note: for all java based clients (TestNG, Junit) mode will be set automatically to "DEFAULT" if it is not specified. |No |
|rp.skipped.issue                               |Report Portal provides feature to mark skipped tests as not 'To Investigate' items on WS side. Parameter could be equal boolean values: *TRUE* - skipped tests considered as issues and will be marked as 'To Investigate' on Report Portal. *FALSE* - skipped tests will not be marked as 'To Investigate' on application. |No |


Launch name can be edited once, and should be edited once, before first
execution. As usual, parts of launches are fixed for a long time. Keeping the
same name for launch, here we will understand a fixed list of suites under
launch, will help to have a history trend, and on UI instances of the same
launch will be saved with postfix "\#number", like "Test Launch \#1", "Test
Launch \#2" etc.

>   If mandatory properties are missed client throw exception
>   IllegalArgumentException.

**Proxy configuration**

The client uses standard java proxy mechanism. If you are new try [Java networking and proxies](<http://docs.oracle.com/javase/8/docs/technotes/guides/net/proxies.html>) page.

Ways to set up properties:

a. reportportal.properties file

b. command line properties (-Dhttps.proxyHost=localhost)

**How to provide parameters**

There are two ways to load parameters.

- Load from properties file

Properties file should have name: “reportportal.properties”. Properties file can
be situated on the class path (in the project directory).

If listener can’t find properties file it throws FileNotFoundException. By
default “reportportal.properties” exists in the reportportall-client.jar, but
user can create his own “reportportal.properties” file and put in class path.

- Use system variables

**Parameters loading order**

Client loads properties in the next order (every next level overrides previous):

a.  Properties file. Client loads all known to him properties (specified in the
    "Parameters" section) from "reportportal.properties" file.

b.  Environment variables. If environment variables with names specified in the
    "Parameters" section exist client reloads existing properties from
    environment variables.


### Description

Handling events

TestNG agent can handle next events:

-   Start launch

-   Finish launch

-   Start suite

-   Finish suite

-   Start test

-   Finish test

-   Start test step

-   Successful finish of test step

-   Fail of test step

-   Skip of test step

-   Start configuration (All «before» and «after» methods)

-   Fail of configuration

-   Successful finish of configuration

-   Skip configuration

### Objects interrelation

| **TestNG object**    | **ReportPortal object**       |
|----------------------|-------------------------------|
| LAUNCH               |LAUNCH                         |
| BEFORE_SUITE         |TestItem (type = BEFORE_SUITE) |
| BEFORE_GROUPS        |TestItem (type = BEFORE_GROUPS)|
| SUITE                |TestItem (type = SUITE)        |
| BEFORE_TEST          |TestItem (type = BEFORE_TEST)  |
| TEST                 |TestItem (type = TEST)         |
| BEFORE_CLASS         |TestItem (type = BEFORE_CLASS) |
| CLASS                |Not in structure. Avoid it     |
| BEFORE_METHOD        |TestItem (type = BEFORE_METHOD)|
| METHOD               |TestItem (type = STEP)         |
| AFTER_METHOD         |TestItem (type = AFTER_METHOD) |
| AFTER_CLASS          |TestItem (type = AFTER_CLASS)  |
| AFTER_TEST           |TestItem (type = AFTER_TEST)   |
| AFTER_SUITE          |TestItem (type = AFTER_SUITE)  |
| AFTER_GROUPS         |TestItem (type = AFTER_GROUPS) |

TestItem – report portal specified object for representing:  suite, test, method objects in different test systems. Used as tree structure and can be recursively placed inside himself.


## JUnit

### Installation

Download package [here](<https://bintray.com/epam/reportportal/agent-java-junit>)

Report Portal (hereinafter: RP) provides two client modules for valid tracking
of [JUnit](<http://junit.org/>) script events:

-   Report Portal listener

-   Report Portal custom runner

The Listener provides information for RP in appropriate form and updates launch
structure in accordance with developed template.

The Runner was created for *\@BeforeClass*, *\@Before*, *\@After* and
*\@AfterClass* methods notifications rising. These notifications are not
top-level events for native [JUnit](<http://junit.org/>) runners\\listeners.


### Report Portal Listener

The Listener could be included in the project via following scripts running
method.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ java
import org.junit.runner.JUnitCore;
import com.epam.reportportal.junit.ReportPortalListener;

public class JUnitScriptsRunner {
    public static void main(String[] args) {
        JUnitCore core= new JUnitCore();
        core.addListener(new ReportPortalListener());
        core.run(UserTestClass1.class, UserTestClass2.class, UserTestClass3.class);
    }
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If [Maven](<http://maven.apache.org/index.html>) has been used by test project
so listener could be included in
[maven-surefire-plugin](<http://maven.apache.org/surefire/maven-surefire-plugin/index.html>)
configuration in pom.xml.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ xml
<plugin>
   <groupId>org.apache.maven.plugins</groupId>
   <artifactId>maven-surefire-plugin</artifactId>
   <version>2.15</version>
   <configuration>
      <properties>
         <property>
            <name>listener</name>
            <value>com.epam.reportportal.junit.ReportPortalListener</value>
         </property>
      </properties>
   </configuration>
</plugin>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


### Report Portal Runner

The RP runner could be used via following ways:

-   Via \@RunWith annotation

Usage

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ java
@RunWith(com.epam.reportportal.junit.CustomJUnitRunner.class)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-   Via Suite class extension

If test project has using Suite extended class so you could include RP runner
like in example below.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ java
import java.util.List;
import org.junit.internal.runners.InitializationError;
import org.junit.runner.Runner;
import org.junit.runners.Suite;
import org.junit.runners.model.RunnerBuilder;
import com.epam.reportportal.junit.CustomJUnitRunner;

public class MySuite extends Suite {
   private static Class<?>[] getAnnotatedClasses(Class<?> klass) throws InitializationError {
      Suite.SuiteClasses annotation = klass.getAnnotation(Suite.SuiteClasses.class);
      if (annotation == null) {
         throw new InitializationError(String.format("class '%s' must have a SuiteClasses annotation", klass.getName()));
      }
      return annotation.value();
   }

   public MySuite(Class<?> klass, RunnerBuilder builder) throws InitializationError {
      super(klass, getRunners(getAnnotatedClasses(klass)));
   }

   public static List<Runner> getRunners(Class<?>[] classes) throws InitializationError {
      List<Runner> runners = new LinkedList<Runner>();
      for (Class<?> klazz : classes) {
         runners.add(new CustomJUnitRunner(klazz)); //Initialization of RP custom runner
      }
      return runners;
   }
 }
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

>   If you use custom runner, please extend CustomJUnitRunner class instead of
>   native BlockJUnit4ClassRunner JUnit4 class!

>   Also make sure that running order keep in safe after yours extensions.

### Configuration

In order to start using of agent, user should configure property file
“reportportal.properties” in such format.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ java
rp.username=user_name
rp.uuid=user_token
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


### Parameters

User should provide next parameters to agent.

| **Parameter**                     | **Description**      | **Required**        |
|-----------------------------------|----------------------|----------------------|
| rp.enable                         |Enable/Disable logging to Report Portal. rp.enable=true - enable log to RP server. Any other value means 'false' rp.enable=false - disable log to RP server |No |
| rp.username                       |User name |Yes |
| rp.password                       |**UUID** or domain password |Yes |
| rp.endpoint                       |URL of web service, where requests should be send |Yes |
| rp.launch                         |The unique name of Launch (Run). Based on that name a history of runs will be created for particular name |Yes |
| rp.project                        |Project name, to identify scope |Yes |
| rp.tags                           |Set of tags for specifying additional meta information for current launch. Format: tag1;tag2;build:12345-6. Tags should be separated by “;”. There are one special tag- build – it should be used for specification number of build for launch. |No |
| rp.batch.size.logs                |In order to rise up performance and reduce number of requests to server |Yes|
| rp.keystore.resource              |Put your JKS file into resources and specify path to it | |
| rp.keystore.password              |Access password for JKS (certificate storage) package, mentioned above | |
| rp.convertimage                   |Colored log images can be converted to grayscale for reducing image size. Values: ‘true’ – will be converted.  Any other value means ‘false’. |No|
| rp.mode                           |Report Portal provides possibility to specify visibility of executing launch. Currently two modes are supported:     DEFAULT  - all users from project can see this launch;  DEBUG - only owner can see this launch (in debug sub tab). Note: for all java based clients (TestND, Junit) mode will be set automatically to  "DEFAULT" if it is not specified. |No |


Launch name can be edited once, and should be edited once, before first
execution. As usual, parts of launches are fixed for a long time. Keeping the
same name for launch, here we will understand a fixed list of suites under
launch, will help to have a history trend, and on UI instances of the same
launch will be saved with postfix "\#number", like "Test Launch \#1", "Test
Launch \#2" etc.

>   If mandatory properties is missed client throw exception
>   IllegalArgumentException.


### How to provide parameters

There are two ways to load parameters.

-   Load from properties file

Properties file should have name: "reportportal.properties". Properties file can
be situated on the class path (in the project directory).

If listener can’t find properties file it throws FileNotFoundException. By
default "reportportal.properties" exists in the reportportall-client.jar, but
user can create his own “reportportal.properties” file and put in class path.

-   Use system variables

Read [here](#documentation/Tuning-CI-tool) how to do it by Jenkins


### Parameters loading order

Client loads properties in the next order (every next level overrides previous):

1. Properties file. Client loads all known to him properties (specified in the
    "Parameters" section) from "reportportal.properties" file.

2. Environment variables. If environment variables with names specified in the
    "Parameters" section exist client reloads existing properties from
    environment variables.


## Cucumber


### Installation and usage

There are several options to use Cucumber formatter:

-  Add *gem 'yarpc'* to your *Gemfile* and point either to locally cloned YARPC
    sources or to Git repository (refer to Bundler documentation for more
    details):

*gem 'yarpc', :path => '<local path to sources>'*

or

*gem 'yarpc', :git => '<URL to Git repo>'*

Then run Cucumber with -f/--format switch:

*cucumber <other options> -f YARPC::Cucumber::Formatter*

-  Clone Git repository to some local folder and require yarpc.rb in Cucumber command line:

*cucumber <other options> -r <local path to sources>\lib\yarpc.rb -f YARPC::Cucumber::Formatter*

### Configuration

Create report\_portal.yml configuration file in one of the following folders of
your Cucumber project: '.', './.config', './config' (see
report\_portal.yaml.example). Alternatively specify path to configuration file



| **Property**             | **Description**      |
|--------------------------|----------------------|
| username                 | ReportPortal username|
| password                 | ReportPortal password|             
| endpoint                 | ReportPortal endpoint|      
| launch                   | launch name          |        
| project                  | project name         |
| tags                     | array of tags for the launch|
| group_by_folder          | set to true to enable grouping by folder (feature folders will be represented as nested suites in ReportPortal. Not fully supported yet by ReportPortal UI). In single-threaded mode folder suites are closed when all their children are closed. In parallel mode (with YARPCParallel formatter) all folder suites remain open until the very end, and are closed along with the launch.|   
| debug_mode               | set to true to mark the launch as 'DEBUG'            |
| thread_cool_off_timeout  | timeout in seconds for the last thread in the bunch to wait for other threads to start. If no new threads start during this interval, the launch is closed. Launches via parallel_tests do not need this parameter because no new threads will be started when the old ones are completed; however it may be useful to group together the results of several independent CI jobs.|
| disable_ssl_verification | set to true to disable SSL verification on connect to ReportPortal (potential security hole!) In single-threaded mode folder suites are closed when all their children are closed. In parallel mode (with ParallelFormatter) all folder suites remain open until the very end, and are closed along with the launch.|
| use_standard_logger      | set to true to enable logging via standard Ruby Logger class to ReportPortal. Note that log messages are transformed to strings before sending.


Each of these settings can be overridden by an environment variable with the same name and 'rp.' prefix (e.g. 'rp.username' for 'username').

### Usage

Run Cucumber with -f/--format switch:

*cucumber <other options> -f RP::Cucumber::YARPC*

### Supported events

-   before\_features (start launch)

-   after\_features (finish launch)

-   before\_feature (start feature suite and suites for its parent folders if
    group\_by\_folder == true)

-   after\_feature (finish feature suite and suites for its parent folders if
    group\_by\_folder == true unless there are other unfinished features in
    them)

-   before\_feature\_element (start test (either scenario or scenario outline))

-   after\_feature\_element (finish test)

-   before\_step (start test step for regular scenario)

-   after\_step (finish test step for regular scenario)

-   before\_table\_row (start test step for example row of scenario outline)

-   after\_table\_row (finish test step for example row of scenario outline)

-   puts

-   embed

-   exception

**Notes:**

1. Steps of scenario outlines are always reported as 'Passed'; the actual pass/fail status is reported for individual example rows.

2. Background steps are reported within the scenarios as 'BEFORE\_TEST' items.

### Logging

Experimental support for three common logging frameworks was added:

-   Logger (part of standard Ruby library)

-   [Logging](<https://rubygems.org/gems/logging>)

-   [Log4r](<https://rubygems.org/gems/log4r>)

To use Logger, set use\_standard\_logger parameter to true (see Configuration
chapter). For the other two corresponding appenders/outputters are available
under yarpc/logging.

### Parallel formatter

YARPC::Cucumber::ParallelFormatter can be used instead for tests started via
parallel\_tests gem.

>   To facilitate parallel reporting from independent processes (as is the case with
>   parallel\_tests gem), ParallelFormatter formatter creates
>   'parallel\_launch\_id.lck' and a number of 'parallel\_launch\_process\*.lck'
>   files in the TEMP directory. If, for whatever reason (e.g. because the previous
>   launch was aborted due to exception), any of these files are present when new
>   launch is invoked, results may be reported incorrectly.

**Notes:**

1. Rows of scenario outlines are reported as individual scenarios with row
    number in brackets.

2. To get more readable logs for scenario outlines, start Cucumber with
    -x/--expand switch. However, even in this case log messages for outlines may
    appear in wrong order - this is due to Cucumber limitations and will not be
    fixed in YARPC.

3. Without --expand switch, all steps of scenario outlines are reported before
    other log messages, so it is not easy to see which log message belongs to
    which step. This is due to Cucumber limitations and will not be fixed in
    YARPC.

4. Feature background is reported once at the beginning as 'Before Class' item
    and the first scenario does not include background steps. Remaining
    scenarios are 'augmented' with background steps and there are no separate
    'Before Class/Test' items for them. This reflects the way Cucumber actually
    executes scenarios: if the very first background fails, all scenarios in the
    feature are skipped, even though on second iteration this background might
    pass.

5. ReportPortal launch created by YARPC has root test suite as a single direct
    descendant. This is workaround for current ReportPortal UI implementation
    and will be removed when UI 2.0 is deployed.

## Cucumber-JVM (Java)


### Installation

Add to POM.xml

**dependency**

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ xml
<repositories>
     <repository>
        <snapshots>
          <enabled>false</enabled>
        </snapshots>
        <id>bintray-epam-reportportal</id>
        <name>bintray</name>
        <url>http://dl.bintray.com/epam/reportportal</url>
     </repository>
</repositories>



<dependency>
  <groupId>com.epam.reportportal</groupId>
  <artifactId>agent-java-cucumber</artifactId>
  <version>2.6.1</version>  
</dependency>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


Located in the main [RP repository](<https://github.com/reportportal/agent-java-cucumber>).

Configuration is identical to JUnit RP client.

It is implemented as a custom Cucumber reporter and is enabled in the same way
as other custom reporters
([http://cukes.info](<https://cucumber.io>)).

There are two versions: **ScenarioReporter** and **StepReporter**;

the difference is in granularity level, ScenarioReporter being probably the
recommended option. 


TestItem – report portal specified object for representing: suite, test, method
objects in different test systems. Used as tree structure and can be recursively
placed inside himself.


## Node.js Client (JavaScript)

Only Cucumber listener is available at the moment, and it was tested only in **Protractor/Cucumber combo** (although it should work in vanilla Cucumber-JS as well).

### Installation

To install directly from Git, run

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ powershell
npm install git+<HTTPS or SSH repository URL>.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To install from local path, clone repository and run

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ powershell
npm install <local path>.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

See *npm -?* for more details.

Cucumber listener can be enabled by adding to world.js something like this:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ javascript
this.registerListener(require('node-rp-client').Cucumber({}));
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


### Configuration

Create reportportal.json in the working directory or specify desired config file in the environment variable rp_config (again, relative to the working directory).

The following properties are supported:

-   username - Report Portal username (string, required)
-   password - Report Portal password (string, required)
-   endpoint - Report Portal API endpoint (string, required)
-   launch - launch name (string, required)
-   project - project name (string, required)
-   mode - launch mode ("DEFAULT" or "DEBUG", optional, defaults to "DEFAULT")
-   tags - launch tags (array of strings, optional)
-   root_folder - name (not path!) of the folder where all tests are located (string, optional)

Each of these settings can be overridden by an environment variable with the same name and 'rp_' prefix (e.g. 'rp_username' for 'username'). Environment variables take precedence over JSON configuration. 'tags' property is parsed as JSON array (i.e. it has to be defined like this: set rp_tags=["xxx", "yyy", "zzz"]); all other properties are used literally.

**Important notes:**

1. Certificate verification is disabled in HTTPS agent because I do not have enough time left to figure how to supply the whole certificate chain to the client. If you know how of are willing to investigate, please do fix this security hole.

2. As of the date of writing this, the latest version of node-rest-client, 1.4.3, (one of this module's dependencies) is unable to send binary data in HTTP requests which is required to upload attachments such as screenshots. The workaround is provided in [this pull request](https://github.com/aacerox/node-rest-client/pull/58); until it is merged, attachments will appear corrupted in Report Portal.

To use Report Portal client while this pull request is not merged, the following workaround can be taken:

- install patched REST client using *npm install git+* [https://github.com/nebehr/node-rest-client.git#write-buffers](https://github.com/nebehr/node-rest-client.git#write-buffers)
- now install Report Portal client as described above. Since REST client is already installed, the patch will not be overwritten.


## NUnit

### Installation

There are 2 ways how to install NUnit add-in. The first way is directed to
install add-in for NUnit runner, the second is only for your assembly with
tests. So the test results will be reported to Report Portal if tests are
executed by NUnit runner with installed add-in, or only if specific test
assembly is executed and not depend on what NUnit runner is used.

### Windows Installer Package

Download the latest [Windows Installer package](https://www.nuget.org/packages/ReportPortal.Client/) package on your machine and follow
instructions during installation.

>   **Warning!**

>   You have to install add-in into *bin/addins* folder that is located in the
>   folder where NUnit (nunit.exe) runner is.

>   Example: *C:\\Program Files (x86)\\NUnit 2.6.4\\bin\\addins*

When installation process is completed please verify that add-in is installed
successfully. Execute NUnit GUI runner, go to menu *Tools -\> Addins*.

[ ![Image](Images/NUnit_1.png) ](Images/NUnit_1.png)

[ ![Image](Images/NUnit_2.png) ](Images/NUnit_2.png)

If add-in is successfully loaded beginning from now all tests that will be
executed by this NUnit runner (exe) will be represented on Report Portal.

### NuGet Package

Install **ReportPortal.NUnit** NuGet package into your project with features
files. Missed dependencies will be installed automatically.

>   Read [here](<http://docs.nuget.org/consume/package-manager-dialog>) how to
>   manage NuGet packages.

NuGet package is compatible only with NUnit runner **v2.6.4**. This limitation is
related to NUnit architecture restrictions. If you use some other version of
NUnit launcher and you are not able to upgrade/downgrade to v2.6.4, please
contact us and we will provide compatible NuGet package with NUnit runner which
you use.

### Configuration

You can configure Report Portal plugin. All settings are stored in
*EPAM.ReportPortal.Addins.NUnit.dll.config* file. You can find this file by path
where you installed add-in: installation path of MSI package or in VS project in
other case.

| **Property**             | **Description**      |
|--------------------------|----------------------|
| enabled                  |Enable/Disable reporting to Report Portal server. |
| server - url             |The base URI to Report Portal REST web service. |
| server - project         |Name of project |
| server - proxy - server  |Proxy server for all requests to Report Portal services. This element is optional. |
| authentication - username|Name of user. |
| authentication - password|Password of user. UID can be used instead of opened password. You can find it on user's profile page. |
| launch - debugMode       |Turn on/off debugging of your tests. Only you have access for test results if test execution is proceeded in debug mode. |
| launch - name            |Name of test execution. |
| launch - tags            |Comma separated tags for test execution. |
| logConsoleOutput         |Specify whether console output also should be reported |


See example of valid configuration file below.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ xml
<configuration>
  <configSections>
    <section name="reportPortal" type="EPAM.ReportPortal.Addins.NUnit.ReportPortalSection, EPAM.ReportPortal.Addins.NUnit, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  </configSections>
  <reportPortal enabled="true" logConsoleOutput="true">
    <server url="https://{HOST}:{PORT}/api/v1/" project="default_project">
      <authentication username="default" password="1q2w3e" />
      <proxy server="yourproxy.com:port"/>
    </server>
    <launch name="NUnit Demo Launch" debugMode="true" tags="t1,t2" />
  </reportPortal>
</configuration>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

## SoapUI

### Installation

You can download SoapUI agent from
[here](<https://github.com/reportportal/agent-java-soapui>).

**Option 1: Desktop SoapUI**

SoapUI agent is provided as zip archive. Unpack this archive into SoapUI root folder, and **override** conflicted files.
And remove the below jars from */lib* directory because old versions:

-   log4j
-   guava
    ... and any other found in /lib folder

To verify that agent is successfully installed start SoapUI and see logs:

[ ![Image](Images/pic_234.jpg) ](Images/pic_234.jpg)

Now agent is plugged to SoapUI.

**HTTPS**

For access to servers working via HTTPS protocol - JKS keystore, provided by RP, is required.
Keystore for access to RP is available [here](#user-profile). Please place it under ${SOAPUI-HOME}/bin

**Option 2: via Maven Plugin in external automation**

Add following repository to your pom.xml.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ xml
     <repository>
        <snapshots>
          <enabled>false</enabled>
        </snapshots>
        <id>bintray-epam-reportportal</id>
        <name>bintray</name>
        <url>http://dl.bintray.com/epam/reportportal</url>
     </repository>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

And dependency (please check latest available version under specified artifactory and use it instead provided):

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ xml
<dependency>
   <groupId>com.epam.ta.reportportal</groupId>
   <artifactId>soapui-client</artifactId>
   <version>2.1.9</version>
</dependency>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Exclude conflicted dependencies from soapui-maven-plugin.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ xml
<groupId>eviware</groupId>
<artifactId>maven-soapui-plugin</artifactId>
<version>4.5.1</version>
<exclusions>
   <exclusion>
      <artifactId>httpclient</artifactId>
      <groupId>org.apache.httpcomponents</groupId>
   </exclusion>
   <exclusion>
      <artifactId>httpcore</artifactId>
      <groupId>org.apache.httpcomponents</groupId>
   </exclusion>
   <exclusion>
      <artifactId>commons-codec</artifactId>
      <groupId>commons-codec</groupId>
   </exclusion>
   <exclusion>
      <artifactId>httpmime</artifactId>
      <groupId>org.apache.httpcomponents</groupId>
   </exclusion>
   <exclusion>
      <artifactId>log4j</artifactId>
      <groupId>log4j</groupId>
   </exclusion>
</exclusions>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Add the property to the plugin section.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ xml
<property>
   <name>soapui.ext.listeners</name>
   <value>PATH TO FOLDER CONTAINS reportportal-listeners.xml FILE</value>
</property>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Put your soapui-log4j.xml in project directory (or made it available under classpath).


### Configuration

ReportPortal listeners described under ${SOAPUI-HOME}/bin/listeners/reportportal-listeners.xml for Desktop SoapUI version, or in user-specified file for maven plugin using:

**reportportal-listeners.xml**
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ xml
 <?xml version="1.0" encoding="UTF-8"?> <tns:soapui-listeners xmlns:tns="http://eviware.com/soapui/config">
<tns:listener id="TestStepListener" listenerClass="com.epam.ta.reportportal.soapui.listeners.TestStepListener"
   listenerInterface="com.eviware.soapui.model.testsuite.TestRunListener" />
<tns:listener id="TestCaseListener" listenerClass="com.epam.ta.reportportal.soapui.listeners.TestCaseListener"
   listenerInterface="com.eviware.soapui.model.testsuite.TestSuiteRunListener" />
<tns:listener id="TestSuiteListener" listenerClass="com.epam.ta.reportportal.soapui.listeners.TestSuiteListener"
   listenerInterface="com.eviware.soapui.model.testsuite.ProjectRunListener" />
</tns:soapui-listeners>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

## Desktop SoapUI

For configuring SoapUI agent user has to set the follows properties into project custom properties or set them via system variables. For example:

[ ![Image](Images/soapui-properties.png) ](Images/soapui-properties.png)

And in text format:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ java
rp.username = andrei_ramanchuk
rp.uuid = 00000000-0000-0000-0000-000000000001
rp.endpoint = https://rp.epam.com

rp.launch = soapui_TEST_EXAMPLE
rp.project = default_project
rp.tags = TAG1;TAG2
rp.batch.size.logs = 1
rp.mode = DEFAULT

## Optionally you can provide JKS keystore with certificate
rp.keystore.resource = your_keystore.jks
rp.keystore.password = keystore_password
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

### Logging

If you need to change logging you may override soapui-log4j.xml. Use "REPORTPORTAL" appender if you want to track messages to Report Portal.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">

<!-- ===================================================================== -->
<!-- -->
<!-- This is an example of a Log4j XML configuration file. -->
<!-- -->
<!-- ===================================================================== -->

<log4j:configuration xmlns:log4j="http://jakarta.apache.org/log4j/"
    debug="false">

    <!-- =================== -->
    <!-- Appenders -->
    <!-- =================== -->

    <appender name="CONSOLE" class="org.apache.log4j.ConsoleAppender">
        <errorHandler class="org.apache.log4j.helpers.OnlyOnceErrorHandler" />
        <param name="Target" value="System.out" />
        <param name="Threshold" value="DEBUG" />
        <layout class="org.apache.log4j.PatternLayout">
            <!-- The default pattern: Date Priority [Category] Message\n -->
            <param name="ConversionPattern" value="%d{ABSOLUTE} %-5p [%c{1}] %m%n" />
        </layout>
    </appender>

    <appender name="FILE" class="org.apache.log4j.RollingFileAppender">
        <errorHandler class="org.apache.log4j.helpers.OnlyOnceErrorHandler" />
        <param name="File" value="soapui.log" />
        <param name="Threshold" value="INFO" />
        <param name="Append" value="false" />
        <param name="MaxFileSize" value="5000KB" />
        <param name="MaxBackupIndex" value="50" />
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%d %-5p [%c{1}] %m%n" />
        </layout>
    </appender>

    <appender name="ERRORFILE" class="org.apache.log4j.RollingFileAppender">
        <errorHandler class="org.apache.log4j.helpers.OnlyOnceErrorHandler" />
        <param name="File" value="soapui-errors.log" />
        <param name="Threshold" value="INFO" />
        <param name="Append" value="true" />
        <param name="MaxFileSize" value="5000KB" />
        <param name="MaxBackupIndex" value="50" />
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%d %-5p [%c{1}] %m%n" />
        </layout>
    </appender>

    <appender name="SOAPUI" class="com.eviware.soapui.support.log.SoapUIAppender">
        <errorHandler class="org.apache.log4j.helpers.OnlyOnceErrorHandler" />
    </appender>
    <appender name="REPORTPORTAL"
        class="com.epam.ta.reportportal.soapui.log.SoapUILogAppender">
        <errorHandler class="org.apache.log4j.helpers.OnlyOnceErrorHandler" />
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%d %-5p [%c{1}] %m%n" />
        </layout>
    </appender>
    <appender name="GLOBAL_GROOVY_LOG" class="org.apache.log4j.FileAppender">
        <errorHandler class="org.apache.log4j.helpers.OnlyOnceErrorHandler" />
        <param name="File" value="global-groovy.log" />
        <param name="Threshold" value="DEBUG" />
        <param name="Append" value="true" />
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%d %-5p [%c{1}] %m%n" />
        </layout>
    </appender>


    <!-- =============== -->
    <!-- Loggers -->
    <!-- =============== -->
    <logger name="groovy.log">
        <level value="INFO" />
        <appender-ref ref="GLOBAL_GROOVY_LOG" />
        <appender-ref ref="REPORTPORTAL" />
    </logger>

    <logger name="com.eviware.soapui">
        <level value="DEBUG" />
        <appender-ref ref="SOAPUI" />
        <appender-ref ref="CONSOLE" />
        <appender-ref ref="REPORTPORTAL" />
    </logger>

    <logger name="soapui.errorlog">
        <level value="DEBUG" />
        <appender-ref ref="ERRORFILE" />
        <appender-ref ref="REPORTPORTAL" />
    </logger>

    <logger
        name="com.eviware.soapui.impl.wsdl.support.http.SoapUIMultiThreadedHttpConnectionManager">
        <level value="ERROR" />
        <appender-ref ref="ERRORFILE" />
        <appender-ref ref="REPORTPORTAL" />
    </logger>

    <logger name="com.eviware.soapui.impl.wsdl.WsdlSubmit">
        <level value="ERROR" />
        <appender-ref ref="ERRORFILE" />
        <appender-ref ref="REPORTPORTAL" />
    </logger>

    <root>
        <priority value="INFO" />
        <appender-ref ref="FILE" />
        <appender-ref ref="REPORTPORTAL" />
    </root>

</log4j:configuration>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

## SpecFlow

### Installation

Install **ReportPortal.SpecFlow** NuGet package into your project with features
files. Missed dependencies will be installed automatically.

>   Read [here](<http://docs.nuget.org/consume/package-manager-dialog>) how to
>   manage NuGet packages.

After installing NuGet package your App.config will be modified. Report Portal
plugin and step assembly will be registered in the specFlow section.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ xml
<specFlow>
  ...
  <plugins>
    <add name="EPAM.ReportPortal.Addins" type="Runtime" />
  </plugins>
  ...
  <stepAssemblies>
    <stepAssembly assembly="EPAM.ReportPortal.Addins.SpecFlowPlugin" />
  </stepAssemblies>
  ...
</specFlow>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

### Configuration

You can configure Report Portal plugin. All settings are stored in
*EPAM.ReportPortal.Addins.SpecFlowPlugin.dll.config* file. This file is added
into VS project with NuGet package installation.

| **Property**              | **Description**                                                                                                        |
|---------------------------|------------------------------------------------------------------------------------------------------------------------|
| enabled                   |Enable/Disable reporting to Report Portal server                                                                        |
| server - url              |The base URI to Report Portal REST web service                                                                          |
| server - project          |Name of project                                                                                                         |
| authentication - username |Name of user                                                                                                            |
| authentication - password |Password of user. UID can be used instead of opened password. You can find it on user's profile page                    |
| launch - debugMode        |Turn on/off debugging of your tests. Only you have access for test results if test execution is proceeded in debug mode |
| launch - name             |Name of test execution                                                                                                  |
| launch - tags             |Comma separated tags for test execution                                                                                 |



Example of valid configuration file is below.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ xml
<configuration>
  <configSections>
    <section name="reportPortal" type="EPAM.ReportPortal.Addins.SpecFlowPlugin.ReportPortalSection, EPAM.ReportPortal.Addins.SpecFlowPlugin, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  </configSections>
  <reportPortal enabled="true">
    <server url="https://{SERVER}:{PORT}/api/v1/" project="default_project">
      <authentication username="default" password="1q2w3e" />
    </server>
    <launch name="SpecFlow Demo Launch" debugMode="true" tags="t1,t2" />
  </reportPortal>
</configuration>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

## ScalaTest

### Installation
1) add this library to your project as dependency
   * SBT
    >
    >      libraryDependencies += "com.epam.reportportal" %% "agent-scala-scalatest" % "2.6.0" % "test"
    >

   * Maven
    >      <dependency>
    >        <groupId>com.epam.reportportal</groupId>
    >        <artifactId>agent-scala-scalatest_2.11</artifactId>
    >        <version>2.6.0</version>
    >        <scope>test</scope>
    >      </dependency>

   * Gradle
    >
    >      testCompile group: 'com.epam.reportportal', name: 'agent-scala-scalatest_2.11', version: '2.6.0'
    >

2) add your reportportal.properties file to test/resources (See Configuration section)

3) set reporter in your build tool:
   * _maven_ (Configuration options / Reporters): 
  
    http://www.scalatest.org/user_guide/using_the_scalatest_maven_plugin
   * _sbt_
  
    In your build.sbt
   ```scala
   testOptions in Test += Tests.Argument(TestFrameworks.ScalaTest, "-C", "com.epam.reportportal.scalatest.RPReporter")
   ```
   * _gradle:_ in your test task: 
   ```groovy
   args = ['-C', 'com.epam.reportportal.scalatest.RPReporter']
   ```
4) define bintray repository
   * _sbt_
     ```scala
       resolvers ++= Seq(
         "EPAM bintray" at "http://dl.bintray.com/epam/reportportal"
       )
     ```
   * _maven_  
      [Resolving Artifacts](https://bintray.com/docs/usermanual/formats/formats_mavenrepositories.html#anchorMavenResolve)  
      URL parameter is https://dl.bintray.com/epam/reportportal/
   * _gradle_
     ```groovy
     repositories {
         jcenter()
         mavenLocal()
         maven { url "http://dl.bintray.com/epam/reportportal" }
     }
    ```
    
### Configuration

Copy you configuration from UI of Report Portal at _User Profile_ section

or

In order to start using of agent, user should configure property file
"reportportal.properties" in such format:

**reportportal.properties**

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ java
rp.endpoint = https://rp.epam.com/
rp.username = default
rp.uuid = 8967de3b-fec7-47bb-9dbc-2aa4ceab8b1e

rp.launch = default_TEST_EXAMPLE
rp.project = default_project
rp.tags = TAG1;TAG2

rp.keystore.resource = reportportal-client-v2.jks #optional
rp.keystore.password = reportportal #optional

rp.batch.size.logs = 5
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

*Keystore parameters are optional.*

**Parameters**

User should provide next parameters to agent.

| **Parameter**                                 | **Description**      | **Required**|
|-----------------------------------------------|----------------------|-------------|
|rp.enable                                      |Enable/Disable logging to Report Portal: rp.enable=true - enable log to RP server.  Any other value means 'false': rp.enable=false - disable log to RP server.  If parameter is absent in  properties file then automation project results will be posted on RP. |No |
|rp.username                                    |User name |Yes |
|rp.password                                    |User password. **We strongly recommend to use UUID** or separate ReportPortal internal users password here to avoid domain password publishing. |Yes |
|rp.uuid                                        |UUID of user. |Yes |
|rp.endpoint                                    |URL of web service, where requests should be send |Yes |
|rp.launch                                      |The unique name of Launch (Run). Based on that name a history of runs will be created for particular name |Yes |
|rp.project                                     |Project name to identify scope |Yes |
|rp.tags                                        |Set of tags for specifying additional meta information for current launch. Format: tag1;tag2;build:12345-6. Tags should be separated by “;”. There are one special tag- build – it should be used for specification number of build for launch. |No |
|rp.batch.size.logs                             |In order to rise up performance and reduce number of requests to server |Yes |
|rp.keystore.resource                           |Put your JKS file into resources and specify path to it | |
|rp.keystore.password                           |Access password for JKS (certificate storage) package, mentioned above | |
|rp.convertimage                                |Colored log images can be converted to grayscale for reducing image size. Values: ‘true’ – will be converted. Any other value means ‘false’. |No |
|rp.mode                                        |Report portal provides possibility to specify visibility of executing launch. Currently two modes are supported: DEFAULT  - all users from project can see this launch; DEBUG - only owner can see this launch (in debug sub tab). Note: for all java based clients (TestNG, Junit) mode will be set automatically to "DEFAULT" if it is not specified. |No |
|rp.skipped.issue                               |Report Portal provides feature to mark skipped tests as not 'To Investigate' items on WS side. Parameter could be equal boolean values: *TRUE* - skipped tests considered as issues and will be marked as 'To Investigate' on Report Portal. *FALSE* - skipped tests will not be marked as 'To Investigate' on application. |No |


## Starting Logging

To start logging with Report Portal, add a specific listener to your automation
framework, called an agent.

An agent consists of 2 parts:

-   Structure listener

-   Log listener (appender)

Use the configuration suggestion at the **Profile** page; it will help you with
the required parameters.

[ ![Image](Images/testFrameworkIntegration/startingLogging/userProfilePage.png) ](Images/testFrameworkIntegration/startingLogging/userProfilePage.png)

To configure logging, follow these steps:

1. Configure a structure listener according to the information in one of the topics above.

2. Configure a log appender, according to the description at the Logging Integration page.

3. Configure the properties file as described in the topic above.

4. Don't forget about the JKS (Java Key Storage) file for a Java-based project.

5. Execute your automation tests.

6. Select your project as described above.