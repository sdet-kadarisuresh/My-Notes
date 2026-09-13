# PART 15: TestNG + Selenium ⭐⭐⭐⭐⭐

### 1. What is TestNG? Why TestNG?
**Q: What is TestNG and why do we use it in automation frameworks?** 🔥
**A:** In my project, whenever we set up a new automation framework from scratch, TestNG is our absolute go-to execution engine. TestNG stands for "Test Next Generation", and it's an open-source testing framework for Java, inspired by JUnit but specifically designed to overcome its limitations for end-to-end and integration testing. 

When you write standard Java code, execution always starts from the `main` method. But in a massive automation suite with hundreds of test cases, you can't just throw everything into a `main` method. You need a structured way to organize your tests, control the exact order they execute, run them in parallel to save time, pass dynamic data to them, and generate comprehensive reports. That's exactly the role TestNG plays—it's the orchestrator.

What we typically do is use TestNG to manage our test lifecycle. The main reasons we prefer it are its advanced annotations, native support for parallel execution via XML configuration, and incredible data-driven testing capabilities using `@DataProvider`. For instance, if I need to run a regression suite across multiple browsers concurrently, I just tweak a few attributes in the `testng.xml` file. It also generates a default HTML report which, while basic, is a lifesaver when you quickly need to see what failed after a Jenkins job finishes.

**Q: What are the differences between TestNG and JUnit?** 🔥
| Feature | TestNG | JUnit (4/5) |
|---------|--------|-------------|
| **Primary Focus** | End-to-end, Integration, and UI testing | Unit testing for developers |
| **Grouping** | Natively supported (`groups={"smoke"}`) | Supported via Tags/Categories but less intuitive |
| **Data-Driven** | Excellent with `@DataProvider` | Parameterization exists but is more complex |
| **Parallel Execution** | Very simple via `testng.xml` properties | Complex and requires extra configuration |
| **Test Dependency** | `dependsOnMethods`, `dependsOnGroups` | Not natively supported or discouraged |
| **Reporting** | Generates default HTML/XML reports | Needs third-party plugins (like Surefire) |

**Verbal explanation:** If an interviewer asks me this, I’d explain: "From my experience, JUnit is fantastic for developers writing isolated unit tests. However, in UI automation, we deal with complex workflows. For example, my 'Checkout' test might strictly depend on my 'Login' test passing. TestNG handles this gracefully with `dependsOnMethods`. Furthermore, in our project, we maintain a single test script for user registration but test it with 50 different data combinations. TestNG's `@DataProvider` makes this extremely clean to implement, whereas doing the same in JUnit feels like a workaround."

### 2. TestNG Annotations (ALL of them with execution order)
**Q: Can you explain the various TestNG annotations and their execution order?** 🔥
**A:** Absolutely. The way I handle test setup and teardown in my framework heavily relies on understanding the exact hierarchy of TestNG annotations. TestNG provides a rich set of pre-condition and post-condition annotations that wrap around your actual `@Test` methods.

The standard execution order from highest to lowest scope is:
1. `@BeforeSuite`
2. `@BeforeTest`
3. `@BeforeClass`
4. `@BeforeMethod`
5. `@Test`
6. `@AfterMethod`
7. `@AfterClass`
8. `@AfterTest`
9. `@AfterSuite`

In my project, we use these very specifically:
- `@BeforeSuite`: We use this to set up global configurations, like initializing the ExtentReports engine or establishing a database connection that the entire test run will use.
- `@BeforeTest`: (Note: "Test" here refers to the `<test>` tag in `testng.xml`, not the `@Test` method). We use this to read specific environment configurations or set up test-data files for a specific module.
- `@BeforeClass`: This is where we usually initialize our WebDriver if we want one browser instance per class.
- `@BeforeMethod`: We use this to launch the browser, navigate to the base URL, or perform a login before *every single* test case.
- `@Test`: This contains our actual test logic and assertions.
- The `@After...` annotations just execute in the exact reverse order to clean up resources, like taking screenshots on failure (`@AfterMethod`), quitting the driver (`@AfterClass`), and flushing the reports (`@AfterSuite`).

Here is a code snippet showing the order:
```java
public class AnnotationHierarchyTest {

    @BeforeSuite
    public void beforeSuite() {
        System.out.println("1. @BeforeSuite: Setup global Extent Report, DB connections.");
    }

    @BeforeTest
    public void beforeTest() {
        System.out.println("2. @BeforeTest: Setup module-specific data.");
    }

    @BeforeClass
    public void beforeClass() {
        System.out.println("3. @BeforeClass: Initialize Page Objects for this class.");
    }

    @BeforeMethod
    public void beforeMethod() {
        System.out.println("4. @BeforeMethod: Launch browser and login.");
    }

    @Test
    public void testMethod1() {
        System.out.println("5. @Test: Executing Actual Test Case 1");
    }

    @AfterMethod
    public void afterMethod() {
        System.out.println("6. @AfterMethod: Capture screenshot, logout, close browser.");
    }

    @AfterClass
    public void afterClass() {
        System.out.println("7. @AfterClass: Nullify Page Objects.");
    }

    @AfterTest
    public void afterTest() {
        System.out.println("8. @AfterTest: Clear module-specific data.");
    }

    @AfterSuite
    public void afterSuite() {
        System.out.println("9. @AfterSuite: Flush Extent Reports, close DB connection.");
    }
}
```

**Q: What is the difference between @BeforeMethod, @BeforeClass, @BeforeTest, and @BeforeSuite?** 🔥
| Annotation | Scope | Real-time Project Usage |
|------------|-------|-------------------------|
| `@BeforeSuite` | Runs exactly ONCE before the entire XML suite execution starts. | Setting up Extent Reports, establishing DB/API connections. |
| `@BeforeTest` | Runs ONCE before any class inside a specific `<test>` tag in XML. | Loading specific test data sheets, setting parallel execution variables. |
| `@BeforeClass` | Runs ONCE before the first `@Test` method in the current class. | Initializing Page Object classes or setting up a shared WebDriver for the class. |
| `@BeforeMethod` | Runs BEFORE EVERY SINGLE `@Test` method in the class. | Launching the browser, navigating to the URL, logging in, clearing cookies. |

**Verbal explanation:** "In interviews, people often confuse `@BeforeTest` and `@BeforeMethod`. I always clarify it this way: In my framework, if I have 5 test cases (`@Test` methods) in a class, `@BeforeMethod` will execute 5 times—once before each test. But `@BeforeTest` is entirely tied to the XML configuration; it runs once per `<test>` block defined in the `testng.xml`, regardless of how many classes or methods are inside it. We use `@BeforeMethod` for browser isolation so each test starts fresh, and `@BeforeSuite` for one-time heavy lifting."

### 3. @Test Annotation
**Q: How do you use the @Test annotation and its attributes in your framework?** 🔥
**A:** The `@Test` annotation is the heart of TestNG. Without it, TestNG simply ignores the method. But what makes it incredibly powerful are its attributes. In my project, we rarely just use a plain `@Test`; we almost always configure its behavior using attributes to handle real-world complexities.

Let me break down the ones we use most frequently:

1. **`priority`**: By default, TestNG runs tests in alphabetical order. But often, we have sequential flows. For example, I can't test "Add to Cart" if "Search Product" hasn't run. We use `priority` to dictate the order. Lower numbers run first.
2. **`enabled`**: Sometimes a feature is broken in the current build, or a test is flaky, and we don't want it to fail the CI/CD pipeline. Instead of commenting out the code, we just set `enabled = false`.
3. **`description`**: We always add a brief description of what the test does. This text appears in the Extent Reports, making it easier for manual testers and managers to understand the report.
4. **`dependsOnMethods`**: This is crucial. If my `loginTest` fails, there is absolutely no point in running `checkoutTest`. We use this to skip downstream tests if the prerequisite fails, saving massive execution time.
5. **`invocationCount`**: Sometimes we need to test if a specific API endpoint or a UI button can handle rapid successive clicks (basic load testing), or we want to verify the stability of a flaky test. We set `invocationCount = 5` to run that specific test 5 times back-to-back.

Here is a real-world code example demonstrating all of these:

```java
public class CheckoutTests {

    @Test(priority = 1, description = "Verify user can login successfully")
    public void loginTest() {
        System.out.println("Executing Login");
        // Assert.fail("Simulating failure to show dependency");
    }

    @Test(priority = 2, dependsOnMethods = {"loginTest"}, description = "Search for a laptop")
    public void searchProductTest() {
        System.out.println("Executing Product Search");
    }

    @Test(priority = 3, dependsOnMethods = {"searchProductTest"}, alwaysRun = true)
    public void addToCartTest() {
        System.out.println("Executing Add to Cart - always runs even if search fails");
    }

    @Test(enabled = false, description = "This feature is currently deprecated")
    public void legacyPaymentTest() {
        System.out.println("This will not execute");
    }

    @Test(invocationCount = 3, threadPoolSize = 3, timeOut = 5000)
    public void concurrentServerLoadTest() {
        System.out.println("Running 3 times concurrently to check stability");
    }
}
```
*Best Practice:* While `priority` is useful, we try to keep our tests independent as much as possible. Independent tests can be run in parallel without issues. We only use `dependsOnMethods` for strict end-to-end journey tests.

### 4. Assertions
**Q: Can you explain Assertions in TestNG? What is the difference between Hard Assert and Soft Assert?** 🔥
**A:** Assertions are the validation checkpoints in our automation. Without assertions, an automation script is just a bot clicking around; it doesn't actually "test" anything. In my project, we strictly enforce that every `@Test` must have at least one assertion. TestNG provides two main types: Hard Asserts and Soft Asserts.

**Hard Asserts (`Assert` class):**
This is the standard assertion. The moment a hard assert fails, a `java.lang.AssertionError` is thrown, and the execution of that specific `@Test` method completely stops right there. Any code after the failed assertion is skipped. We use this for critical validations. For example, if I'm testing a login functionality and the login fails (the user is not on the homepage), there is no point in verifying if the user profile picture is displayed. The test should stop immediately.

**Soft Asserts (`SoftAssert` class):**
This is a non-blocking assertion. If a soft assert fails, it logs the failure but *continues* executing the rest of the lines in the `@Test` method. At the very end of the test, we MUST call `softAssert.assertAll()`. This method collates all the failures that occurred and fails the test case at the end. We use this when verifying multiple non-critical elements on a page. For instance, on a product page, I want to verify the Title, the Price, the Image, and the Description. If the Image is missing, I still want to check if the Price is correct in the same test run.

**Q: Hard Assert vs Soft Assert?** 🔥
| Feature | Hard Assert (`Assert`) | Soft Assert (`SoftAssert`) |
|---------|------------------------|----------------------------|
| **Execution Behavior** | Stops execution of the method immediately upon failure. | Continues execution even if an assertion fails. |
| **Class** | Uses static methods from `org.testng.Assert` | Requires creating an object of `org.testng.asserts.SoftAssert` |
| **Mandatory Step** | No extra step needed. | Must call `assertAll()` at the end of the method. |
| **Use Case** | Critical validations (e.g., Login success). | Multiple UI validations on a single page (e.g., verifying labels). |

```java
public class AssertionExample {

    @Test
    public void hardAssertionRealScenario() {
        System.out.println("Step 1: Enter Username");
        System.out.println("Step 2: Enter Password");
        System.out.println("Step 3: Click Login");
        
        boolean isLoggedIn = false; // Simulating login failure
        // Execution stops here!
        Assert.assertTrue(isLoggedIn, "Critical Failure: User could not login!");
        
        System.out.println("Step 4: This line will NEVER be executed if login fails.");
    }

    @Test
    public void softAssertionRealScenario() {
        SoftAssert softAssert = new SoftAssert();
        
        System.out.println("Navigated to Product Page");
        
        String actualTitle = "iPhone 15";
        String expectedTitle = "iPhone 15 Pro";
        
        // This will fail, but execution CONTINUES
        softAssert.assertEquals(actualTitle, expectedTitle, "Title mismatch");
        System.out.println("Step executed despite title failure");
        
        boolean isImageDisplayed = true;
        softAssert.assertTrue(isImageDisplayed, "Image is missing");
        
        // CRITICAL: Must call assertAll() to report the failures
        softAssert.assertAll(); 
    }
}
```

### 5. Groups
**Q: How do you manage executing different subsets of tests, like Smoke or Regression, using TestNG?** 🔥
**A:** This is one of the most practical features we use daily. In our CI/CD pipeline, we don't always have the time to run the entire suite of 2000 tests. When a developer merges a small PR, we just want to run a quick Sanity or Smoke suite. When we do a nightly build, we run the full Regression suite.

We achieve this using TestNG Groups. The way I handle this is by assigning the `groups` attribute to every `@Test` method. A test can belong to multiple groups. For example, a basic Login test is usually part of both "smoke" and "regression".

In the `testng.xml`, we use the `<groups>` and `<run>` tags to specify exactly which groups to include and which to exclude. If a test is flaky or currently being reworked, we might put it in a "broken" group and explicitly `<exclude>` that group in the XML.

Here is how we implement it:

```java
public class CheckoutGroupsTest {

    @Test(groups = {"smoke", "regression"})
    public void validLoginTest() {
        System.out.println("Login test - runs in both smoke and regression");
    }

    @Test(groups = {"regression"})
    public void paymentWithCreditCardTest() {
        System.out.println("Credit card payment - regression only");
    }

    @Test(groups = {"regression", "broken"})
    public void paymentWithCryptoTest() {
        System.out.println("Crypto payment - currently failing, marked as broken");
    }
}
```

And the corresponding `testng.xml` to run ONLY smoke tests, but specifically exclude anything marked as broken:
```xml
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd">
<suite name="E-commerce Automation Suite">
    <test name="Smoke Tests">
        <groups>
            <run>
                <include name="smoke"/>
                <exclude name="broken"/>
            </run>
        </groups>
        <classes>
            <class name="com.project.tests.CheckoutGroupsTest"/>
        </classes>
    </test>
</suite>
```
*Real-time tip:* We enforce a rule in our team that every new test written MUST have at least the "regression" group tag, otherwise, it might get orphaned and never executed in our CI pipeline.

### 6. DataProvider
**Q: What is Data-Driven testing and how do you achieve it in TestNG using DataProvider?** 🔥
**A:** Data-driven testing is a design pattern where we separate our test logic from our test data. Instead of hardcoding usernames and passwords into the test script, we write the script once and loop it over an external dataset. In my framework, we use TestNG's `@DataProvider` heavily for this.

A `@DataProvider` is simply a Java method that returns a 2D array of Objects (`Object[][]`). The `@Test` method then references this provider using the `dataProvider` attribute. TestNG will automatically execute the `@Test` method as many times as there are rows in that 2D array.

What we typically do in a real project is we don't hardcode the data in the DataProvider method itself. Instead, the DataProvider method uses Apache POI to read data from an Excel file, converts that data into an `Object[][]`, and passes it to the test. We also keep our DataProviders in a separate utility class so multiple test classes can reuse them.

Here is a complete, real-world example showing how we connect a Test class to a separate DataProvider class:

```java
// 1. The DataProvider Class
public class LoginDataProviders {

    @DataProvider(name = "loginCredentials")
    public Object[][] getLoginData() {
        // In a real project, this data comes from an Excel file via Apache POI
        return new Object[][] {
            {"admin", "admin123", "Valid Login"},
            {"invalidUser", "admin123", "Invalid Username"},
            {"admin", "wrongPass", "Invalid Password"},
            {"", "", "Blank Credentials"}
        };
    }
}

// 2. The Test Class
public class LoginTest {

    // Reference the provider name, and specify the class where it resides
    @Test(dataProvider = "loginCredentials", dataProviderClass = LoginDataProviders.class)
    public void verifyLogin(String username, String password, String scenarioDescription) {
        System.out.println("Testing Scenario: " + scenarioDescription);
        System.out.println("Entering Username: " + username);
        System.out.println("Entering Password: " + password);
        System.out.println("Clicking Login Button");
        System.out.println("-----------------------------------");
        
        // Actual Selenium code would go here
        // driver.findElement(By.id("user")).sendKeys(username);
    }
}
```
*Best Practice:* Always add a "scenario description" column in your test data. It makes the TestNG logs and Extent Reports instantly readable because you know exactly *which* data row failed without having to open the Excel sheet.

### 7. Parameters
**Q: How do you pass parameters from testng.xml to your tests?**
**A:** While DataProvider is great for testing a script against multiple rows of test data, sometimes we just want to pass a single environment configuration variable at runtime. The most classic example in my project is the browser type (Chrome, Firefox, Edge) or the environment URL (QA, UAT, PROD). We handle this using the `@Parameters` annotation in TestNG.

We define the `<parameter>` tag in the `testng.xml` file. Then, in our base test class (usually inside a `@BeforeMethod` or `@BeforeClass`), we use the `@Parameters` annotation to catch that value and initialize the WebDriver accordingly.

Here is exactly how we do cross-browser testing using parameters:

```java
public class BaseTest {
    WebDriver driver;

    @Parameters({"browser", "envURL"})
    @BeforeMethod
    public void setup(String browserName, String environment) {
        System.out.println("Navigating to environment: " + environment);
        
        if (browserName.equalsIgnoreCase("chrome")) {
            System.out.println("Initializing Chrome Driver");
            // driver = new ChromeDriver();
        } else if (browserName.equalsIgnoreCase("firefox")) {
            System.out.println("Initializing Firefox Driver");
            // driver = new FirefoxDriver();
        } else {
            throw new RuntimeException("Unsupported Browser: " + browserName);
        }
    }

    @Test
    public void sampleTest() {
        System.out.println("Executing test in the initialized browser.");
    }
}
```

And the `testng.xml` that drives this:
```xml
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd">
<suite name="Cross Browser Suite">
    <!-- Global parameter for the whole suite -->
    <parameter name="envURL" value="https://qa.myapp.com"/>

    <test name="Chrome Test Execution">
        <parameter name="browser" value="chrome"/>
        <classes>
            <class name="com.project.tests.BaseTest"/>
        </classes>
    </test>

    <test name="Firefox Test Execution">
        <parameter name="browser" value="firefox"/>
        <classes>
            <class name="com.project.tests.BaseTest"/>
        </classes>
    </test>
</suite>
```
*Edge Case:* If you declare `@Parameters` in your code but forget to provide it in the XML (or if you run the class directly as a Java Application instead of through TestNG), TestNG will throw a `TestNGException`. We often use `@Optional("chrome")` next to the parameter in code to provide a fallback default.

### 8. testng.xml
**Q: Can you explain the complete structure of the testng.xml file?** 🔥
**A:** The `testng.xml` is the steering wheel of the entire automation framework. Without it, you are just running isolated classes. In my projects, we maintain multiple XML files—one for Smoke, one for Regression, one for Cross-browser. 

The structure strictly follows a hierarchy:
1. `<suite>`: The root tag. It represents a collection of tests. We usually configure parallel execution here.
2. `<listeners>`: We register our custom reporting and screenshot listeners here.
3. `<test>`: A logical grouping of classes. This is important because `@BeforeTest` and `@AfterTest` are scoped to this tag.
4. `<parameter>`: Can be placed at the suite level or test level to pass variables to the code.
5. `<classes>` & `<class>`: Specifies which actual Java classes to execute.
6. `<methods>`: Inside a class, we can `<include>` or `<exclude>` specific `@Test` methods.

Here is a comprehensive template representing how a real enterprise `testng.xml` looks:

```xml
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd">
<!-- Run tests in parallel classes, max 3 threads -->
<suite name="Enterprise Automation Suite" parallel="classes" thread-count="3">

    <!-- Listeners for Extent Reports and failure screenshots -->
    <listeners>
        <listener class-name="com.project.listeners.TestListener"/>
        <listener class-name="com.project.listeners.RetryTransformer"/>
    </listeners>

    <parameter name="environment" value="QA"/>

    <test name="Authentication Module">
        <parameter name="browser" value="chrome"/>
        <!-- Executing based on groups -->
        <groups>
            <run>
                <include name="regression"/>
            </run>
        </groups>
        <classes>
            <class name="com.project.tests.LoginTest">
                <!-- Granular control: exclude a specific method -->
                <methods>
                    <exclude name="forgotPasswordTest"/>
                </methods>
            </class>
            <class name="com.project.tests.RegistrationTest"/>
        </classes>
    </test>

</suite>
```

### 9. Parallel Execution
**Q: How do you achieve parallel execution in TestNG and what are the challenges?** 🔥
**A:** Parallel execution is critical. When our regression suite grew to 500+ tests, it took 4 hours to run sequentially. We had to implement parallel execution to bring the time down to 45 minutes for CI/CD feedback.

The way I handle this is entirely through the `testng.xml` file. We don't need complex multi-threading Java code. We just add `parallel` and `thread-count` attributes to the `<suite>` tag.

Options for `parallel`:
- `methods`: Runs individual `@Test` methods in parallel.
- `classes`: (Most preferred in my project) Runs all tests inside a single class sequentially, but runs multiple classes in parallel. This ensures logical flow within a class is maintained.
- `tests`: Runs multiple `<test>` blocks in the XML in parallel.

**The major challenge:** Thread Safety.
When you run in parallel, multiple threads are executing simultaneously. If you use a single `public static WebDriver driver` variable, Thread 1 will initialize Chrome, Thread 2 will initialize Firefox, and Thread 1 will suddenly start sending commands to Thread 2's Firefox instance. The session crashes with `NoSuchSessionException`.

**The Solution: ThreadLocal**
To fix this, in our `BaseTest`, we wrap the WebDriver in a `ThreadLocal<WebDriver>`. This creates a separate, isolated instance of the WebDriver for each executing thread.

Code implementation:
```java
public class BaseTest {
    // ThreadLocal ensures thread safety during parallel execution
    protected static ThreadLocal<WebDriver> threadLocalDriver = new ThreadLocal<>();

    @BeforeMethod
    public void setup() {
        WebDriver driver = new ChromeDriver();
        // Set the driver for the current thread
        threadLocalDriver.set(driver); 
    }

    // Custom getter to use the driver in tests/pages
    public static WebDriver getDriver() {
        return threadLocalDriver.get();
    }

    @AfterMethod
    public void tearDown() {
        getDriver().quit();
        // Crucial: remove to prevent memory leaks
        threadLocalDriver.remove(); 
    }
}
```
And the XML configuration:
```xml
<suite name="Parallel Suite" parallel="classes" thread-count="4">
    <!-- Configuration goes here -->
</suite>
```

### 10. Retry Mechanism
**Q: How do you handle flaky tests? Can you explain the TestNG Retry mechanism?** 🔥
**A:** In any UI automation framework, tests will occasionally fail due to network blips, slow server responses, or DOM rendering delays. These are "flaky" failures. If we fail the build immediately, developers lose trust in the automation. What we do is implement a Retry Mechanism that automatically re-runs a failed test a specified number of times before officially marking it as "Failed".

To do this, we implement the `IRetryAnalyzer` interface provided by TestNG.

Here is the exact code we use:

```java
// 1. Create the Retry Analyzer
public class RetryAnalyzer implements IRetryAnalyzer {
    int counter = 0;
    int retryLimit = 2; // We retry 2 times

    @Override
    public boolean retry(ITestResult result) {
        if (counter < retryLimit) {
            System.out.println("Retrying test " + result.getName() + " with status "
                    + getResultStatusName(result.getStatus()) + " for the " + (counter + 1) + " time(s).");
            counter++;
            return true; // Tells TestNG to retry
        }
        return false; // Tells TestNG to stop retrying and fail
    }

    private String getResultStatusName(int status) {
        if (status == 1) return "SUCCESS";
        if (status == 2) return "FAILURE";
        if (status == 3) return "SKIP";
        return null;
    }
}
```

Now, we could attach this to a test like `@Test(retryAnalyzer = RetryAnalyzer.class)`, but doing that for 500 tests is a maintenance nightmare. Instead, we use `IAnnotationTransformer` to apply this retry logic globally to ALL tests dynamically at runtime.

```java
// 2. Global Annotation Transformer
public class RetryAnnotationTransformer implements IAnnotationTransformer {
    @Override
    public void transform(ITestAnnotation annotation, Class testClass, Constructor testConstructor, Method testMethod) {
        // Automatically attach the RetryAnalyzer to every @Test
        annotation.setRetryAnalyzer(RetryAnalyzer.class);
    }
}
```
Then, we simply register `RetryAnnotationTransformer` in the `<listeners>` block of our `testng.xml`.

### 11. TestNG Listeners
**Q: What are Listeners in TestNG? How do you use them in your framework?** 🔥
**A:** Listeners are interfaces that "listen" to events during the test execution and allow us to trigger custom code based on those events. In my project, listeners are the backbone of our reporting and debugging infrastructure.

The most common interface we implement is `ITestListener`. It has methods that trigger automatically when a test starts, passes, fails, or skips.
The way I handle failure screenshots is entirely through `onTestFailure`. Instead of putting `try-catch` blocks in every test method, the listener catches the failure globally, grabs the driver instance, takes a screenshot, and attaches it to the Extent Report.

Here is a simplified real-world implementation:

```java
public class MyTestListener implements ITestListener {

    @Override
    public void onTestStart(ITestResult result) {
        System.out.println("STARTED: " + result.getMethod().getMethodName());
        // Here we create a new node in Extent Reports
    }

    @Override
    public void onTestSuccess(ITestResult result) {
        System.out.println("PASSED: " + result.getMethod().getMethodName());
        // Here we log PASS in Extent Reports
    }

    @Override
    public void onTestFailure(ITestResult result) {
        System.out.println("FAILED: " + result.getMethod().getMethodName());
        System.out.println("Reason: " + result.getThrowable().getMessage());
        
        // Pseudo-code for screenshot:
        // WebDriver driver = BaseTest.getDriver();
        // File src = ((TakesScreenshot)driver).getScreenshotAs(OutputType.FILE);
        // attach to Extent Report...
    }

    @Override
    public void onTestSkipped(ITestResult result) {
        System.out.println("SKIPPED: " + result.getMethod().getMethodName());
    }
}
```
We register this in the `testng.xml`:
```xml
<listeners>
    <listener class-name="com.project.listeners.MyTestListener"/>
</listeners>
```

---

# PART 16: PAGE OBJECT MODEL ⭐⭐⭐⭐⭐

### 13. What is POM? Why POM?
**Q: What is the Page Object Model (POM) and why is it so important?** 🔥
**A:** In my project, we strictly adhere to the Page Object Model design pattern. If you don't use POM, you end up writing procedural code where your Selenium locator strategies (like `By.xpath`), your driver actions (`driver.click()`), and your actual test assertions (`Assert.assertEquals()`) are all mixed together in the same test method. 

This causes a massive maintenance nightmare. Imagine you have 50 test cases that interact with the Login Page. If the developers change the ID of the username field from `user_id` to `login_username`, you would have to find and update that locator in 50 different test files. 

POM solves this by enforcing separation of concerns. In POM, we create a separate Java class for every web page in our application (e.g., `LoginPage.java`, `HomePage.java`). 
- The **Page Class** acts as a repository. It contains ONLY the locators (WebElements) for that page and the methods to interact with those locators (e.g., `enterUsername()`, `clickLogin()`).
- The **Test Class** contains ONLY the test data and the assertions. It calls the methods from the Page Class.

The benefits we see every day are:
1. **Reusability:** The `login()` method in `LoginPage` can be reused by `CheckoutTest`, `ProfileTest`, etc.
2. **Maintainability:** If the UI changes, we update the locator in exactly ONE place (the Page class). All 50 tests automatically work again.
3. **Readability:** Our test classes read like plain English.

### 14. POM Structure
**Q: How do you structure a Page Object Model framework?**
**A:** The architecture of our framework is highly modularized. A clean directory structure is critical for scaling. Here is exactly how we set up our packages:

- `src/main/java/com/project/pages`: Contains all the Page classes.
- `src/main/java/com/project/base`: Contains `BaseTest` (WebDriver setup/teardown) and `BasePage` (Common UI interactions).
- `src/test/java/com/project/tests`: Contains all the Test classes with TestNG annotations.
- `src/main/java/com/project/utils`: Contains utilities like `WaitUtils`, Excel readers, config readers.

In the Page class itself, the structure is strictly:
1. **Private Locators:** Using `By` variables or `@FindBy`.
2. **Constructor:** Requires a `WebDriver` to be passed in to initialize the page.
3. **Public Action Methods:** Methods like `enterText()`, `clickButton()`, or composite methods like `doLogin(user, pass)`.

### 15. Complete POM Implementation (Without PageFactory)
**Q: Can you write a complete, working example of POM from scratch?** 🔥
**A:** Yes. We currently prefer standard POM using `By` locators over PageFactory because it avoids the `StaleElementReferenceException` issues common with lazy initialization. Here is exactly how we code it.

**1. LoginPage.java (The Page Class)**
```java
package com.project.pages;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;

public class LoginPage {
    private WebDriver driver;

    // 1. Locators (Private to prevent direct access from tests)
    private By usernameField = By.id("user-name");
    private By passwordField = By.id("password");
    private By loginButton = By.id("login-button");
    private By errorMessage = By.cssSelector("[data-test='error']");

    // 2. Constructor
    public LoginPage(WebDriver driver) {
        this.driver = driver;
    }

    // 3. Page Actions (Public)
    public void enterUsername(String username) {
        driver.findElement(usernameField).sendKeys(username);
    }

    public void enterPassword(String password) {
        driver.findElement(passwordField).sendKeys(password);
    }

    public void clickLogin() {
        driver.findElement(loginButton).click();
    }

    // Composite method for convenience
    public HomePage doLogin(String username, String password) {
        enterUsername(username);
        enterPassword(password);
        clickLogin();
        // Returning the next page object (Method Chaining / Fluent Page Object)
        return new HomePage(driver);
    }

    public String getErrorMessage() {
        return driver.findElement(errorMessage).getText();
    }
}
```

**2. HomePage.java**
```java
package com.project.pages;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;

public class HomePage {
    private WebDriver driver;

    private By headerTitle = By.className("title");

    public HomePage(WebDriver driver) {
        this.driver = driver;
    }

    public boolean isHeaderDisplayed() {
        return driver.findElement(headerTitle).isDisplayed();
    }
}
```

**3. LoginTest.java (The Test Class)**
```java
package com.project.tests;
import org.testng.Assert;
import org.testng.annotations.Test;
import com.project.pages.LoginPage;
import com.project.pages.HomePage;

public class LoginTest extends BaseTest {

    @Test
    public void validLoginTest() {
        // Test logic is completely separated from WebDriver API
        LoginPage loginPage = new LoginPage(driver);
        
        HomePage homePage = loginPage.doLogin("standard_user", "secret_sauce");
        
        Assert.assertTrue(homePage.isHeaderDisplayed(), "Home page was not displayed after login!");
    }

    @Test
    public void invalidLoginTest() {
        LoginPage loginPage = new LoginPage(driver);
        loginPage.doLogin("locked_out_user", "secret_sauce");
        
        String actualError = loginPage.getErrorMessage();
        Assert.assertTrue(actualError.contains("locked out"), "Error message mismatch");
    }
}
```

### 16. BasePage Class
**Q: What is a BasePage and why do you need it?** 🔥
**A:** In a large application, you don't want to rewrite `driver.findElement(By...).click()` in every single page class. More importantly, you rarely just click an element in modern web apps; you usually need to wait for it to be clickable first. 

What we do is create a `BasePage` class. This class acts as a wrapper for Selenium's native commands. It contains robust methods for clicking, sending keys, and getting text, which automatically include explicit waits and exception handling. Every other Page class (`LoginPage`, `HomePage`) `extends BasePage`.

Here is an example of our `BasePage.java`:

```java
package com.project.base;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import java.time.Duration;

public class BasePage {
    protected WebDriver driver;
    protected WebDriverWait wait;

    public BasePage(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    }

    // Custom robust click method
    protected void clickElement(By locator) {
        wait.until(ExpectedConditions.elementToBeClickable(locator)).click();
    }

    // Custom robust sendKeys method
    protected void enterText(By locator, String text) {
        WebElement element = wait.until(ExpectedConditions.visibilityOfElementLocated(locator));
        element.clear();
        element.sendKeys(text);
    }

    protected String getElementText(By locator) {
        return wait.until(ExpectedConditions.visibilityOfElementLocated(locator)).getText();
    }
}
```
Now, in `LoginPage`, instead of calling `driver.findElement(username).sendKeys(user)`, we simply call `enterText(usernameField, user)`. This makes the code drastically cleaner and virtually eliminates `ElementNotInteractable` exceptions.

### 17. PageFactory & @FindBy
**Q: What is PageFactory and how does it differ from traditional POM?**
**A:** PageFactory is a built-in extension of the Page Object Model provided by Selenium. Instead of defining locators using `By` class variables and writing `driver.findElement()`, PageFactory uses annotations like `@FindBy` to initialize WebElements directly.

When you use PageFactory, you must initialize the elements in the constructor using `PageFactory.initElements(driver, this)`. This implements "Lazy Initialization". The elements aren't actually searched for on the web page until the moment you call a method on them (like `.click()`).

Here is how the `LoginPage` looks using PageFactory:

```java
package com.project.pages.factory;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;

public class LoginPageFactory {
    WebDriver driver;

    // Locators defined via annotations
    @FindBy(id = "user-name")
    private WebElement usernameField;

    @FindBy(xpath = "//input[@id='password']")
    private WebElement passwordField;

    @FindBy(css = "#login-button")
    private WebElement loginButton;
    
    // @FindBys acts as an AND operator (must match both criteria)
    // @FindAll acts as an OR operator (must match either criteria)

    public LoginPageFactory(WebDriver driver) {
        this.driver = driver;
        // CRITICAL: Initializes the @FindBy elements
        PageFactory.initElements(driver, this);
    }

    public void doLogin(String user, String pass) {
        // Elements are only searched in the DOM at this exact moment
        usernameField.sendKeys(user);
        passwordField.sendKeys(pass);
        loginButton.click();
    }
}
```

### 18. POM vs PageFactory
**Q: What is the difference between standard POM (By locators) and PageFactory? Which do you prefer?** 🔥
| Feature | POM (Using `By` locators) | PageFactory (Using `@FindBy`) |
|---------|---------------------------|-------------------------------|
| **Locator Storage** | Stored as `By` objects (e.g., `By.id("btn")`) | Stored directly as `WebElement` proxies |
| **Initialization** | Uses `driver.findElement(By)` inside methods | Requires `PageFactory.initElements(driver, this)` |
| **DOM Searching** | Element is found precisely when `findElement` is executed | Lazy initialization via proxies; found when interacted with |
| **Stale Element Exception** | Less prone, as you can easily wrap dynamic waits around `By` locators | Highly prone if DOM updates (e.g., React/Angular apps) |
| **Wait Integration** | Very easy to pass `By` locators to Explicit Waits | Harder to pass `WebElement` directly to `ExpectedConditions` |

**Verbal explanation:** "In interviews, I am often asked this. In my current project, we actually moved *away* from PageFactory and strictly use standard `By` locators. Why? Modern web applications built on React or Angular constantly refresh the DOM. PageFactory caches the `WebElement` proxy. If the DOM refreshes after PageFactory initializes but before we click, we get a `StaleElementReferenceException`. With standard POM, we pass the `By` locator directly to our explicit waits in the `BasePage`, ensuring Selenium checks the fresh DOM right at the exact millisecond of interaction. PageFactory looks cleaner aesthetically, but standard POM is far more stable for dynamic apps."

### 19. POM Best Practices
**Q: What are the best practices you follow when designing a Page Object Model framework?** 🔥
**A:** In our code reviews, we strictly enforce several POM design rules to keep the framework scalable:

1. **Locators MUST be private:** Tests should never have access to `By` locators or WebElements. Tests should only call public action methods. This enforces encapsulation.
2. **Tests MUST NOT contain driver logic:** You should never see `driver.findElement` or `wait.until` inside a `@Test` method. The test class is purely for assertions.
3. **Methods should return the next Page Object (Fluent Interface):** If clicking the "Login" button takes me to the "Dashboard", the `clickLogin()` method should `return new DashboardPage(driver)`. This allows for method chaining: `loginPage.enterUser().enterPass().clickLogin().verifyDashboard()`.
4. **Use a BasePage:** Common behaviors like `waitForClickability`, `scrollToElement`, or `jsClick` belong in a `BasePage` that other pages extend.
5. **Meaningful Method Names:** Don't write `typeUser()`. Write `enterUsername()`. It must reflect business actions.

### 20. Real Project POM Structure Summary
In a mature framework, this all comes together beautifully. 
1. `config.properties` holds the base URL.
2. `testng.xml` triggers the suite.
3. `BaseTest` reads the config, launches Chrome, and passes the `driver` to the `@Test`.
4. The `@Test` instantiates the `LoginPage` and passes data from `@DataProvider`.
5. The `LoginPage` inherits `BasePage` and uses its explicit wait methods to safely interact with the `By` locators.
6. The test asserts the final state and `ITestListener` takes a screenshot if anything fails.

### 21. POM Interview Questions
**Q: Can you do assertions inside the Page Class?** 🔥
**A:** No, this is a strict anti-pattern. We never put `Assert.assertEquals()` inside `LoginPage.java`. The Page class's job is just to represent the state and behavior of the web page. It should return booleans or Strings (like `isErrorDisplayed()` or `getHeaderText()`). The `Assert` must always reside in the Test Class (`LoginTest.java`). Mixing them defeats the purpose of separation of concerns.

**Q: How do you handle common elements like a Navigation Header present on every page?**
**A:** We don't duplicate the locators for the Navigation Header in `HomePage`, `DashboardPage`, and `ProfilePage`. Instead, we create a specific component class called `HeaderComponentPage`. Or, if it's truly global, we place those locators in the `BasePage` so all inherited page classes have immediate access to the header actions.
