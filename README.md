# WebDriver Factory Framework
A Simple Java based framework for creating WebDriver instances from a configuration file 
and injecting the created WebDriver(s) into the functional test cases.

This framework is broken down into two major components,
1. **webdriver-framework-core**: Which contains the code for reading the WebDriver capabilities from an external
configuration file and creating the WebDriver instances that are configured.
1. **webdriver-framework-${testframework}**: These are the glue code that are required to integrate the **webdriver-framework-core**
with the respective test framework that you are using. At present there are three implementations of this.
   1. **webdriver-framework-testng**
   1. **webdriver-framework-junit4**
   1. **webdriver-framework-junit5**
    
## TestNG WebDriver Framework
This is the testng specific framework library intended to be used by developers who are using TestNG as their Testing framework.
The steps to include this into your code is as follows,
1. **Add the dependency to your `pom.xml` **
```xml
        <dependency>
            <groupId>io.github.webdriver.testng</groupId>
            <artifactId>webdriver-framework-testng</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
```

2. **Create the Data Provider for injecting of the WebDriver**
TestNG enables you to pass in parameters into your test method via the use of [Data Providers](https://testng.org/doc/documentation-main.html#parameters-dataproviders). 
   We use the [@DataProvider](https://testng.org/doc/documentation-main.html#parameters-dataproviders) annotation on a method that is then responsible for 
providing the number of WebDrivers that are configured in the WebDriver config. 
   This data provider method can be added to a base class from which all your tests are extended and used as below,
    ```java
   public abstract class BaseTest {
       @DataProvider(name="webdriver", parallel = true)
        public static Iterator<Object[]> provideWebDrivers(Method testMethod) {
            return new LazyInitWebDriverIterator(testMethod.getName(),
                                                 WebDriverFactory.getInstance().getPlatforms(),
                                                 new Object[0]);
       }
   }
    ```
This class can then be extended by your test classes and the test written as below,
```java
public class SomeUsefulTestClass extends BaseTest {

    @Test(dataProvider = "webdriver")
    public void placeOrder(WebDriver webDriver) {
        // Write a useful test with the WebDriver
    }
}
```

3. **Add a WebDriver configuration file**  
The different web driver instances can be configured via a YAML configuration file as below,

```yaml
testEndpoint: https://bstackdemo.com

namedTestUrls:
  url_one: https://www.google.com
  url_two: https://www.yahoo.com

driverType: cloudDriver

onPremDriver:
  platforms:
    - name:
      driverPath: src/test/resources/chromedriver

cloudDriver:
  hubUrl: https://hub-cloud.browserstack.com/wd/hub
  user: BROWSERSTACK_USERNAME
  accessKey: BROWSERSTACK_ACCESSKEY
  localTunnel:
    enabled: false
  common_capabilities:
    project: BrowserStack Demo Repository
    buildPrefix: browserstack-examples-junit5
    capabilities:
      browserstack.debug: true
      browserstack.networkLogs: true
      browserstack.console: debug
  platforms:
    - name: Win10_IE11
      os: Windows
      os_version: '10'
      browser: Internet Explorer
      browser_version: '11.0'
      capabilities:
        browserstack.ie.arch: x32
        browserstack.selenium_version: 3.141.59
    - name: Win10_Chrome_Latest-1
      os: Windows
      os_version: '10'
      browser: Chrome
      browser_version: latest-1
      capabilities:
        browserstack.selenium_version: 3.141.59
    - name: OSX_BigSur_Chrome_Latest
      os: OS X
      os_version: Big Sur
      browser: Chrome
      browser_version: latest
      capabilities:
        browserstack.selenium_version: 3.141.59

```
