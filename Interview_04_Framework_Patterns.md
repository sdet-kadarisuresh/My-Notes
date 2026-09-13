# COMPLETE Framework, Design Patterns & Scenario Q&A (Infosys L2 SDET)

## PART A: FRAMEWORK DESIGN

### Section 1: Page Object Model (POM)

**Q: 1. 🔥 What is Page Object Model? Why do we use it?**
**A:** "In my project, we extensively use the Page Object Model, which is basically a design pattern where we create an object repository for our web UI elements. The way I explain it is: for every webpage in our application (like a Login page or Dashboard), we create a corresponding Java class. This class holds two things: the locators (like XPaths or IDs) for that page, and the action methods that interact with those locators. 
What we typically do is separate the test logic from the page logic. Before we used POM, if a developer changed an element's ID, we had to go through 50 different test scripts to update it. Now, we just update it in one single Page class, and all tests using that page automatically work. It solves the biggest nightmare in automation: maintenance. From my experience, POM makes the code extremely readable, reusable, and cuts down maintenance time by at least 70%."

**Q: 2. 🔥 How do you implement POM in your project? Explain with code**
**A:** "The way I handle this is by creating a standard structure. First, I create a `LoginPage.java` class. I define the locators at the top using `By` locators. Then, I pass the WebDriver via a constructor. Finally, I write the action methods.

```java
public class LoginPage {
    private WebDriver driver;
    
    // Locators
    private By usernameField = By.id("user");
    private By passwordField = By.id("pass");
    private By loginBtn = By.id("login");

    // Constructor
    public LoginPage(WebDriver driver) {
        this.driver = driver;
    }

    // Actions
    public void loginToApp(String user, String pass) {
        driver.findElement(usernameField).sendKeys(user);
        driver.findElement(passwordField).sendKeys(pass);
        driver.findElement(loginBtn).click();
    }
}
```
In the test class, I simply instantiate `LoginPage login = new LoginPage(driver);` and call `login.loginToApp("admin", "admin123");`. It keeps the test incredibly clean."

**Q: 3. 🔥 What is the difference between POM and PageFactory?**
| Feature | Page Object Model (POM) | PageFactory |
|---------|-------------------------|-------------|
| **Definition** | A design pattern for creating Object Repositories | An inbuilt class in Selenium to implement POM |
| **Locators** | Uses `By` locators (e.g., `By.id("xyz")`) | Uses `@FindBy` annotations |
| **Initialization**| Locators are evaluated when `driver.findElement()` is called | Requires `PageFactory.initElements(driver, this)` |
| **Lazy Eval** | Elements evaluated at runtime | Uses `AjaxElementLocatorFactory` for lazy initialization |

**Verbal explanation:** "From my experience, people often confuse these two. POM is just a concept or a design pattern. You can write POM without PageFactory. PageFactory, on the other hand, is an actual class provided by Selenium that gives you a cleaner way to implement POM using `@FindBy` annotations. In my recent projects, I actually prefer standard POM over PageFactory because standard POM with `By` locators gives better control over explicit waits, whereas PageFactory can sometimes throw `StaleElementReferenceException` if the DOM updates."

**Q: 4. What is @FindBy annotation? What are its types?**
**A:** "When we use PageFactory, `@FindBy` is the annotation we use to locate web elements. It replaces the traditional `driver.findElement(By...)`. What we typically do is declare a WebElement at the class level and annotate it. 
```java
@FindBy(id = "username")
private WebElement usernameField;

@FindBys({ // Acts like AND condition
    @FindBy(className = "input-field"),
    @FindBy(name = "user")
})
private WebElement specificField;

@FindAll({ // Acts like OR condition
    @FindBy(id = "login"),
    @FindBy(name = "loginBtn")
})
private WebElement loginButton;
```
In my project, we use `@FindBy` for standard elements, but if an element locator is dynamic (like an XPath containing a variable string), we can't use `@FindBy` because annotations require constant expressions. In those edge cases, I fall back to standard `By` locators."

**Q: 5. What is initElements()? What does PageFactory.initElements(driver, this) do?**
**A:** "If you're using `@FindBy`, you absolutely need `initElements()`. The way I explain it is: `@FindBy` just tells Selenium *how* to find the element, but it doesn't actually find it right away. If you try to interact with that element, you'll get a `NullPointerException`. 
By calling `PageFactory.initElements(driver, this)` inside the Page class constructor, you are instructing Selenium to initialize all the `@FindBy` annotated WebElements in that class using the provided driver instance. 'this' refers to the current class object. In my base page classes, I always put this in the constructor so child pages automatically inherit the initialization."

**Q: 6. 🔥 What are the advantages of POM? What problems does it solve?**
**A:** "In my 7 years of automation, POM is a lifesaver for three main reasons. First: Maintenance. If a locator changes, I update it in exactly one place (the Page class), not in 100 test files. Second: Readability. My test classes read like plain English: `loginPage.enterCredentials()`, `dashboardPage.verifyWelcomeMessage()`. Third: Reusability. Multiple test scripts can use the same page methods.
Before POM, we used to have "Spaghetti Code" where locators and test logic were mixed together. Tests were extremely brittle. POM solves the problem of code duplication and makes the framework highly scalable when the application grows to hundreds of pages."

**Q: 7. How do you handle common elements (header, footer, navigation) in POM?**
**A:** "In my project, we have a top navigation bar that is present on every single page. Instead of duplicating those locators in every Page class, what we typically do is create a `BasePage` or a `CommonElementsPage` class. 
All our specific page classes (like `InventoryPage` or `ProfilePage`) extend this `BasePage`. This way, methods like `clickLogout()` or `searchItem()` are written once in the BasePage, and they are inherited by all child pages. It perfectly follows the DRY (Don't Repeat Yourself) principle."

**Q: 8. What is BasePage? What methods does it contain?**
**A:** "BasePage is the parent class for all Page Object classes. In my framework, BasePage holds the `WebDriver` instance and the `PageFactory.initElements()` call. More importantly, it contains common wrapper methods.
Instead of using standard Selenium methods directly, I write custom wrappers in BasePage. For example, a `clickElement(WebElement ele)` method that incorporates an explicit wait for the element to be clickable before actually clicking it. Or a `sendKeysWithClear()` method. This ensures that every page in my framework automatically handles synchronization, drastically reducing flaky tests."

### Section 2: Framework Architecture 

**Q: 9. 🔥🔥🔥 Explain your automation framework. What is the architecture?**
**A:** "In my current project, I've designed a Hybrid Data-Driven Framework using Selenium WebDriver, Java, TestNG, and Maven. Let me break down the architecture layer by layer.
1. **Core Layer:** We have a `DriverFactory` class implementing the Singleton and Factory patterns to manage thread-safe WebDriver instances using `ThreadLocal`, which is crucial for parallel execution.
2. **Page Layer (POM):** We use the Page Object Model. Every web page has a corresponding class containing locators and action methods. They all extend a `BasePage` which has explicit wait wrappers.
3. **Test Layer:** We use TestNG. All test classes extend a `BaseTest` class where we handle setup (`@BeforeMethod` for browser launch) and teardown. Tests only contain assertions and page method calls.
4. **Data Layer:** For test data, we use Apache POI to read from Excel files, and Jackson API for JSON data. We pass this data to tests using TestNG `@DataProvider`.
5. **Config Layer:** Environment variables (URL, browser type) are kept in a `config.properties` file, read via a `ConfigReader` utility.
6. **Reporting & CI/CD:** We use ExtentReports for beautiful HTML reports, integrated via TestNG `ITestListener` to automatically capture screenshots on failure. Everything is orchestrated through Jenkins, pulling the code from Git and executing Maven commands (`mvn clean test`)."

**Q: 10. 🔥 What is BaseTest class? What does it contain?**
**A:** "The `BaseTest` class is the foundation of the test execution layer. Every test class in my framework extends it. What we typically do is place all the common setup and teardown logic here.
```java
public class BaseTest {
    protected WebDriver driver;

    @BeforeMethod
    public void setUp(ITestResult result) {
        // Read browser from properties, initialize via DriverFactory
        driver = DriverFactory.getDriver("chrome");
        driver.get(ConfigReader.getProperty("url"));
    }

    @AfterMethod
    public void tearDown(ITestResult result) {
        if(ITestResult.FAILURE == result.getStatus()) {
            ScreenshotUtil.takeScreenshot(driver, result.getName());
        }
        DriverFactory.quitDriver();
    }
}
```
This ensures my actual Test classes are completely clean—they only contain `@Test` methods. No driver initialization logic clutters the test cases."

**Q: 11. 🔥 What is DriverFactory? How do you manage WebDriver instances?**
**A:** "DriverFactory is where we handle browser initialization. In an enterprise framework, you can't just write `new ChromeDriver()` everywhere. What I do is use a combination of Factory Pattern and `ThreadLocal` for thread safety during parallel execution.
```java
public class DriverFactory {
    private static ThreadLocal<WebDriver> tlDriver = new ThreadLocal<>();

    public static WebDriver initDriver(String browser) {
        if(browser.equalsIgnoreCase("chrome")) {
            tlDriver.set(new ChromeDriver());
        } else if (browser.equalsIgnoreCase("firefox")) {
            tlDriver.set(new FirefoxDriver());
        }
        tlDriver.get().manage().window().maximize();
        return getDriver();
    }

    public static synchronized WebDriver getDriver() {
        return tlDriver.get();
    }
}
```
By using `ThreadLocal`, if I run 5 tests in parallel, each thread gets its own isolated WebDriver instance, preventing session ID conflicts."

**Q: 12. 🔥 How do you read configuration from properties file?**
**A:** "Hardcoding URLs or credentials is a bad practice. I keep environment-specific variables like `qa_url`, `browser`, and `timeouts` in a `config.properties` file. I created a `ConfigReader` utility to load this into memory once.
```java
public class ConfigReader {
    private static Properties prop;
    static {
        try {
            FileInputStream ip = new FileInputStream("./src/test/resources/config.properties");
            prop = new Properties();
            prop.load(ip);
        } catch (Exception e) { e.printStackTrace(); }
    }
    public static String get(String key) {
        return prop.getProperty(key);
    }
}
```
In my code, whenever I need the URL, I just call `ConfigReader.get("url")`. If the environment changes from QA to Staging, I just change one line in the properties file."

**Q: 13. How do you manage test data?**
**A:** "From my experience, test data management is critical. We use a hybrid approach. For static configuration data (like admin credentials or API keys), we use `properties` files. 
For large datasets driving data-driven tests (like testing a form with 20 different user profiles), we use Excel spreadsheets read via Apache POI. Recently, for complex nested data structures (like API payloads), we transitioned to JSON files and read them using the Jackson or Gson libraries. All this data is seamlessly fed into our TestNG `@Test` methods using `@DataProvider`."

**Q: 14. 🔥 How do you generate reports?**
**A:** "In my projects, standard TestNG reports are not presentable to management. We use Extent Reports (Version 5). What we typically do is create an `ExtentManager` class to configure the HTML reporter (setting the report name, theme, document title).
We integrate this with TestNG Listeners (`ITestListener`). In the `onTestStart()` method, we create an ExtentTest node. In `onTestSuccess()`, we log a pass. In `onTestFailure()`, we log the exception, take a screenshot, and attach the Base64 screenshot directly into the Extent Report. This gives us a highly interactive dashboard with pie charts showing passed/failed tests, which the clients absolutely love."

**Q: 15. How do you handle logging?**
**A:** "System.out.println is strictly banned in our framework. We use Log4j2 for all logging. I configure a `log4j2.xml` file to define appenders—one for the console and one for a rolling file appender that creates a new log file every day.
In every page and test class, I initialize the logger: `private static final Logger log = LogManager.getLogger(LoginPage.class);`. We use different log levels: `log.info()` for standard steps ('Clicked login button'), `log.debug()` for detailed locators, and `log.error()` inside catch blocks when exceptions occur. This makes debugging CI/CD failures infinitely easier."

**Q: 16. 🔥 How do you handle multiple environments (QA, staging, production)?**
**A:** "The way I handle this is by passing the environment dynamically at runtime via Maven command line arguments. In my `config.properties`, I have URLs for different envs: `qa.url`, `stage.url`.
In Maven, I run: `mvn clean test -Denv=stage`.
In my framework, my `ConfigReader` checks for the system property first: `String env = System.getProperty("env");`. If it's null, it falls back to a default value in the properties file. Based on this value, it fetches the corresponding URL. This allows Jenkins to easily trigger runs across different environments without touching the code."

**Q: 17. How do you handle parallel execution in your framework?**
**A:** "Parallel execution is handled primarily through TestNG and ThreadLocal. In the `testng.xml` file, I set `parallel="tests"` or `parallel="methods"` and `thread-count="4"`. 
But the magic happens in the `DriverFactory`. By using `ThreadLocal<WebDriver>`, I ensure that each parallel thread gets its own separate instance of WebDriver. Without ThreadLocal, multiple threads would try to use the same driver session, causing exceptions like `SessionNotFound`. I also ensure that my ExtentReports implementation is thread-safe using `ThreadLocal<ExtentTest>` so logs don't overwrite each other."

**Q: 18. 🔥 What is your test execution flow?**
**A:** "The flow starts from Jenkins. Jenkins pulls the latest code from GitHub and triggers a Maven command like `mvn clean test -Dsuite=regression.xml`.
Maven triggers Surefire plugin, which reads the `testng.xml`. 
TestNG starts the execution. It first hits the `BaseTest` `@BeforeSuite` to initialize ExtentReports. Then `@BeforeMethod` triggers `DriverFactory` to launch the browser and navigate to the URL.
The `@Test` method executes, calling Page Object methods and performing Assertions.
If it fails, the TestNG Listener catches it, takes a screenshot, and attaches it to the report.
Finally, `@AfterMethod` quits the driver, and `@AfterSuite` flushes the ExtentReport. Jenkins then emails this report to the stakeholders."

**Q: 19. How do you handle test data cleanup?**
**A:** "Data cleanup is vital to avoid flaky tests. In my project, we follow the 'Create your own data, clean your own data' rule. 
If an automation script creates a user in the system, we handle the cleanup in two ways: 
1. **UI Cleanup:** In the `@AfterMethod`, write logic to delete the created user via the UI.
2. **API/DB Cleanup (Preferred):** UI cleanup is slow. What we typically do is make a REST API call in the teardown method to delete the test data directly from the backend, or run a JDBC query. It's much faster and more reliable than relying on the UI to clean up."

**Q: 20. 🔥 What is the role of Listeners in your framework?**
**A:** "Listeners actively monitor the execution flow. In my framework, I implement the `ITestListener` interface. 
The main reason we use it is for reporting and screenshots. Without a listener, taking a screenshot on failure requires putting `try-catch` blocks in every single test, which is terrible practice. 
With a listener, I simply override the `onTestFailure(ITestResult result)` method. Inside it, I extract the WebDriver instance from the context, capture the screenshot, and attach it to ExtentReports. I also use `onTestSkipped` to log skipped tests. We just register the listener once in `testng.xml`, and it handles the entire suite automatically."

### Section 3: Data-Driven Testing

**Q: 21. 🔥 What is data-driven testing? How do you implement it?**
**A:** "Data-Driven Testing is when we run the exact same test script multiple times with different sets of input data. Instead of hardcoding values, we separate the data from the logic.
In my framework, I implement this using TestNG's `@DataProvider` combined with Apache POI. I maintain an Excel sheet with columns like 'Username', 'Password', and 'ExpectedResult'. A utility method reads this Excel file, converts it into a 2D Object array `Object[][]`, and feeds it to the `@DataProvider`. The `@Test` method receives these variables as parameters and executes sequentially for every row in the sheet."

**Q: 22. How do you read data from Excel?**
**A:** "I use the Apache POI library. What we typically do is create an `ExcelUtil` class.
We use `FileInputStream` to load the `.xlsx` file, then initialize an `XSSFWorkbook` object. From there, we get the `XSSFSheet` by name. 
To iterate through the data, I find the total rows (`sheet.getLastRowNum()`) and total columns (`sheet.getRow(0).getLastCellNum()`). I use nested `for` loops to read cell values using `cell.getStringCellValue()`. I store this in a 2D Object array and return it. One edge case we handle is cell formatting—if a cell has a numeric value like '123', POI might throw an exception if we expect a string, so we use `DataFormatter` to format everything as a string."

**Q: 23. How do you read data from JSON file?**
**A:** "Recently, JSON has become preferred over Excel because it's lighter and handles complex hierarchical data better. In my project, we use the Jackson library (or Gson).
We create a POJO (Plain Old Java Object) class that maps to the JSON structure. Then, we use the `ObjectMapper` class from Jackson:
```java
ObjectMapper mapper = new ObjectMapper();
TestData[] data = mapper.readValue(new File("data.json"), TestData[].class);
```
We can then pass this array to a TestNG DataProvider. It’s significantly faster than Apache POI and integrates beautifully with REST API payloads as well."

**Q: 24. How do you read data from CSV file?**
**A:** "For CSV files, I use the OpenCSV library. It's much simpler than Apache POI. You use a `CSVReader` object, pass a `FileReader` pointing to your `.csv` file, and use the `readAll()` method. This returns a `List<String[]>` where each `String[]` represents a row. We then easily convert this List into a 2D Array to supply to the TestNG DataProvider."

**Q: 25. 🔥 How do you use TestNG DataProvider for data-driven testing?**
**A:** "A DataProvider is a TestNG method annotated with `@DataProvider` that returns a 2D array: `Object[][]`. 
```java
@DataProvider(name = "loginData")
public Object[][] getData() {
    return new Object[][] { 
        {"admin", "pass123"}, 
        {"invalidUser", "wrongPass"} 
    };
}

@Test(dataProvider = "loginData")
public void loginTest(String username, String password) {
    loginPage.login(username, password);
}
```
In my framework, the DataProvider doesn't hardcode data; it calls my `ExcelUtil.getExcelData("LoginSheet")` method. This allows the test to run multiple times seamlessly without changing the code."

## PART B: DESIGN PATTERNS

**Q: 26. 🔥 What design patterns have you used in your framework?**
**A:** "In my enterprise framework, I utilize several design patterns:
1. **Page Object Model (POM):** To separate UI locators/actions from test logic.
2. **Singleton Pattern:** Used in reading properties files and managing the database connection, ensuring only one instance exists in memory.
3. **Factory Pattern:** Used in `DriverFactory` where I pass a string like 'chrome' and the factory handles the instantiation logic of the correct browser driver.
4. **Facade Pattern:** Sometimes used in Base classes to hide complex setup logic behind a simple interface.
These patterns make the framework scalable, thread-safe, and highly maintainable."

**Q: 27. 🔥 What is Singleton pattern? How do you use it?**
**A:** "The Singleton pattern ensures that a class has only one instance and provides a global point of access to it. We implement this by making the constructor `private` and creating a `public static` method to return the single instance.
In my framework, I use Singleton for `ConfigReader` and `ExtentManager`. You don't want multiple threads creating multiple reporting instances and overwriting each other.
```java
public class ExtentManager {
    private static ExtentReports extent;
    private ExtentManager() {} // Private constructor
    
    public static synchronized ExtentReports getInstance() {
        if(extent == null) {
            extent = new ExtentReports();
            // setup reporters...
        }
        return extent;
    }
}
```"

**Q: 28. 🔥 What is Factory pattern? How do you use it?**
**A:** "The Factory design pattern is used to create objects without exposing the instantiation logic to the client. In automation, the best example is the `DriverFactory`.
Instead of the test class deciding how to configure Chrome options or Firefox profiles, it simply calls `DriverFactory.getDriver("chrome")`. The Factory class contains a `switch` statement that hides all the complex logic of initializing `ChromeDriver`, adding arguments like `--headless` or `--disable-gpu`. If tomorrow we need to add Edge browser support, I only modify the Factory class, not my 500 test scripts."

**Q: 29. What is Builder pattern? Where do you use it?**
**A:** "The Builder pattern is used when we need to construct a complex object step by step. In my framework, I use it heavily for Test Data generation, especially when dealing with APIs or filling out complex registration forms.
Instead of having a constructor with 15 parameters, I use Lombok's `@Builder` annotation on my POJO classes. In the test, I can construct data like:
`User user = User.builder().firstName("John").lastName("Doe").email("j@test.com").build();`
It makes the code incredibly readable and allows me to only specify the fields I actually need for a specific test scenario."

**Q: 30. What is Strategy pattern? Give a framework example**
**A:** "The Strategy pattern enables selecting an algorithm at runtime. In automation, I've used it for element locating strategies or environment setups. 
For example, if my tests run on Desktop Web, I need a standard WebDriver, but if they run on Mobile Web, I need AppiumDriver. I create a `DriverStrategy` interface with a `createDriver()` method. Then I create concrete classes like `ChromeStrategy` and `MobileStrategy`. Based on the runtime configuration, the framework dynamically selects the appropriate strategy to initialize the driver."

**Q: 31. 🔥 What is ThreadLocal? Why is it important for parallel execution?**
**A:** "`ThreadLocal` is a Java class that creates variables which can only be read and written by the same thread. It's absolutely critical for parallel execution in Selenium.
If you declare `public static WebDriver driver;`, it acts as a global variable. If two test threads run simultaneously, Thread 2 might overwrite Thread 1's driver instance, causing Thread 1 to fail with `NoSuchSessionException`. 
By wrapping it as `ThreadLocal<WebDriver> driver = new ThreadLocal<>();`, Java creates an isolated copy of the WebDriver for every execution thread. Thread 1 gets its driver, Thread 2 gets its own, and they never collide."

**Q: 32. What is Dependency Injection? How does it relate to frameworks?**
**A:** "Dependency Injection (DI) is a concept where an object's dependencies are provided externally rather than the object creating them itself. In POM, passing the `WebDriver` instance to the Page classes via their constructor is a prime example of Constructor Injection.
In more advanced frameworks, we use frameworks like PicoContainer (often with Cucumber BDD) or Spring to inject state/context between step definitions, preventing the need for massive static variables. It keeps classes loosely coupled and highly testable."

## PART C: CI/CD & GRID

**Q: 33. 🔥 What is CI/CD? How do you integrate Selenium with Jenkins?**
**A:** "CI/CD stands for Continuous Integration and Continuous Deployment. It means every time a developer commits code, automated checks (like our Selenium suite) run immediately to ensure nothing broke.
To integrate with Jenkins, I create a Jenkins Freestyle or Pipeline job. I configure the Source Code Management to point to our Git repository. In the Build Triggers, I can set it to run daily at midnight or trigger on a webhook when a PR is merged. In the Build Steps, I use `Invoke top-level Maven targets` and run `clean test -Dsuite=sanity.xml`. After execution, Jenkins uses the Extent Reports plugin or Email-ext plugin to publish the results."

**Q: 34. 🔥 How do you run Selenium tests in Jenkins? Step by step**
**A:** "First, ensure Java, Maven, and Git are installed on the Jenkins server. 
1. Create a new Pipeline/Freestyle job.
2. Link the GitHub repository and provide credentials.
3. In the Build step, select 'Invoke top-level Maven targets' and type `clean test` (and any parameters like `-Dbrowser=chrome`).
4. Since Jenkins often runs on a headless Linux server, I ensure my framework passes the `--headless` argument to Chrome, otherwise the browser will fail to launch due to lack of a GUI.
5. In Post-build actions, I configure the email notification to attach the Extent Report and send it to the QA team."

**Q: 35. How do you schedule test runs in Jenkins?**
**A:** "Under the Build Triggers section in Jenkins, we select 'Build periodically'. It uses a cron expression format with 5 asterisks: `MINUTE HOUR DOM MONTH DOW`.
For example, to run the nightly regression suite at 2:00 AM every single day, I would write `0 2 * * *`. To run it only on weekdays at midnight, I'd use `0 0 * * 1-5`. This completely automates the trigger process without human intervention."

**Q: 36. How do you handle test failures in CI/CD pipeline?**
**A:** "When tests fail in CI/CD, the first step is analyzing the Jenkins console output and the Extent Report screenshots. Often, failures in CI/CD (which passes locally) are due to speed/network issues. To handle this proactively, I implement an `IRetryAnalyzer` in TestNG. If a test fails, the framework automatically retries it once or twice. If it passes on the retry, it's flagged as a flaky test rather than a hard failure. If it fails consistently, I raise a Jira ticket and link the Jenkins build URL and screenshot."

**Q: 37. How do you generate and publish reports in Jenkins?**
**A:** "In the framework, Extent Reports saves an HTML file in the `target/reports` folder. In Jenkins, I use the HTML Publisher Plugin. In the Post-build actions, I tell Jenkins the directory (`target/reports`) and the file name (`extent.html`). Jenkins creates a clickable link on the Job dashboard. I also configure the Editable Email Notification plugin to zip the report and email it to stakeholders upon job completion."

**Q: 38. 🔥 What is Selenium Grid? How do you use it for cross-browser testing?**
**A:** "Selenium Grid allows us to distribute our test execution across multiple machines and different browsers simultaneously. It has a Hub-and-Node architecture. The Hub receives the test requests, and the Nodes (which could be Windows, Mac, or Linux machines) execute them.
In code, instead of using `new ChromeDriver()`, we use `RemoteWebDriver(new URL("http://hub-ip:4444/wd/hub"), options)`. We pass `ChromeOptions` or `FirefoxOptions` to tell the Hub what browser we need. Grid routes the test to the appropriate available Node, allowing us to test Chrome on Windows and Safari on Mac in parallel."

**Q: 39. What is Docker? How do you use Docker with Selenium Grid?**
**A:** "Docker is a containerization platform. Setting up Selenium Grid manually on virtual machines is a pain—maintaining Java versions, browser versions, and driver executables. 
With Docker, I can spin up a Selenium Grid in seconds. We use a `docker-compose.yml` file that pulls the official `selenium/hub` image, and several node images like `selenium/node-chrome` and `selenium/node-firefox`. Running `docker-compose up` instantly creates a fully functional Grid. It ensures our test execution environment is perfectly clean, isolated, and identical every single time."

**Q: 40. What is the difference between Selenium Grid 3 and Grid 4?**
| Feature | Selenium Grid 3 | Selenium Grid 4 |
|---------|----------------|----------------|
| **Architecture** | Hub and Node only | Standalone, Hub/Node, Fully Distributed (Router, Distributor, Session Map) |
| **Setup** | Required downloading jar and manual start | Comes natively with the Selenium Server jar, easier Docker integration |
| **Observability**| Minimal logging | Native support for OpenTelemetry for tracing and debugging |
| **Browser Support**| Required manual driver binaries | Uses Selenium Manager to auto-download drivers |

**Verbal explanation:** "Selenium 4 completely revamped Grid. In Grid 3, it was a simple hub-node setup that was hard to scale in cloud environments. Grid 4 is built for modern Kubernetes and Cloud infrastructure. It can be broken down into microservices like Router, Session Map, and Distributor. Also, Grid 4 natively supports Docker, making it much easier to deploy."

## PART D: SCENARIO-BASED QUESTIONS 

**Q: 41. 🔥 Your test passes locally but fails in Jenkins. How do you troubleshoot?**
**A:** "This is a classic issue. First, I check the Jenkins Extent report screenshot. If it's a `TimeoutException` or `NoSuchElementException`, it means the application loaded slower in the CI environment (which usually has fewer resources than a dev laptop). I would check my Explicit Waits and increase the timeout. 
Second, I check screen resolution. Jenkins usually runs in headless mode, which might default to a tiny mobile-like resolution, causing elements to hide behind hamburger menus. I fix this by setting `options.addArguments("--window-size=1920,1080")`. 
Third, I verify the environment variables. Local might be pointing to QA, but Jenkins might be pointing to a Staging DB with different data. I always check logs, wait times, and resolutions first."

**Q: 42. 🔥 You have 1000 test cases. 100 are failing randomly (flaky). What is your approach?**
**A:** "Having 10% flaky tests destroys trust in automation. First, I would isolate these 100 tests into a separate `flaky_suite.xml` and remove them from the main CI pipeline so the main build goes green.
Then, I'd analyze the root causes. From my experience, 80% of flakiness is due to improper synchronization—using `Thread.sleep()` instead of Explicit Waits. I would replace all hard sleeps with `WebDriverWait` for specific conditions (`elementToBeClickable`, `visibilityOf`). 
The other 20% is usually test data collision (tests sharing the same login/data and running in parallel) or DOM updates (React/Angular apps causing `StaleElementReferenceException`). I would implement data isolation and a retry-wrapper for Stale elements."

**Q: 43. 🔥 The application is highly dynamic — elements change IDs on every refresh. How do you write stable locators?**
**A:** "Dynamic IDs (like `id="ext-gen1045"`) are common in Salesforce or React apps. I absolutely avoid using IDs in these cases. 
Instead, I rely on stable attributes. If `name` or `data-testid` is available, I use those. If not, I use XPath functions like `contains()` or `starts-with()`. For example, `//*[starts-with(@id, 'user-')]`. 
Another highly effective strategy is DOM traversal—finding a stable parent element, and navigating to the child: `//div[@class='stable-parent']//input`. I also collaborate with the dev team and ask them to add custom attributes like `data-cy` or `data-automation-id` specifically for testing, which permanently solves the problem."

**Q: 44. 🔥 You need to automate a new module. Walk me through your approach from scratch**
**A:** "First, I would manually test the module to deeply understand the business flow and identify edge cases. I'll review the test cases and categorize them into Sanity, Regression, and Data-Driven.
Next, I'll go to the IDE. I'll create the necessary Page Object classes, capturing robust locators and writing action methods.
Then, I'll create the Test class extending `BaseTest`. I'll write the `@Test` methods utilizing the POM classes and implement hard and soft assertions. 
If the module requires specific data, I'll update the JSON/Excel files and hook up a `DataProvider`. 
Finally, I run the suite locally. Once stable, I merge the PR into Git, update `testng.xml` to include this new module, and monitor the Jenkins run."

**Q: 45. 🔥 Your test execution time is 4 hours. How would you reduce it to 1 hour?**
**A:** "To cut execution time by 75%, I would implement a multi-layered approach:
1. **Parallel Execution:** I would update `testng.xml` to run tests/methods in parallel, utilizing `ThreadLocal` in my DriverFactory. Running 4 threads simultaneously cuts the time drastically.
2. **Headless Execution:** Running browsers in headless mode uses less memory and speeds up execution by about 10-20%.
3. **API for Setup/Teardown:** Instead of using the UI to log in or create test data (which takes 30 seconds per test), I inject cookies or use REST API calls to set up state instantly.
4. **Remove Hard Sleeps:** I would search the entire codebase for `Thread.sleep()` and replace them with Explicit waits. 
5. **Selenium Grid/Cloud:** If hardware is the bottleneck, I'd distribute the load using Selenium Grid on AWS or BrowserStack."

**Q: 46. 🔥 How would you design a scalable framework from scratch?**
**A:** "I would choose Java, Maven, Selenium, and TestNG. 
For structure, I'd use POM to ensure UI changes don't break tests. For parallel execution, I'd use `ThreadLocal` in a `DriverFactory`. 
For data, I'd use Jackson for JSON parsing and TestNG DataProviders. Environment variables would be in a properties file.
For reporting, I'd integrate ExtentReports via an `ITestListener` to auto-capture screenshots.
The code would reside in GitHub, and Jenkins would pull and execute it on a Dockerized Selenium Grid. This architecture is robust, highly parallelizable, and completely agnostic to environment changes."

**Q: 47. A developer changed the UI and 50 tests broke. How do you handle this efficiently?**
**A:** "Because my framework strictly adheres to the Page Object Model, this is a minor issue. I wouldn't need to touch the 50 test files. 
I would identify which web page the UI change occurred on. I go into that specific Page class (e.g., `DashboardPage.java`), update the locator (maybe the ID changed to a class name), and save it. All 50 tests that call that page method will instantly start passing again. This scenario is exactly why we use POM."

**Q: 48. You need to test a feature across Chrome, Firefox, and Edge. How?**
**A:** "I handle Cross-Browser testing using TestNG parameters. In the `testng.xml`, I create different `<test>` tags, passing a `<parameter name="browser" value="chrome"/>` for one, and 'firefox' for another.
In my `BaseTest` `@BeforeMethod`, I use the `@Parameters("browser")` annotation to receive this value and pass it to my `DriverFactory.initDriver(browser)`. The Factory pattern spins up the correct driver. By setting `parallel="tests"`, TestNG will run the Chrome, Firefox, and Edge executions simultaneously, drastically saving time."

**Q: 49. 🔥 How do you handle test data? Static vs dynamic data?**
**A:** "For static data (admin URLs, timeouts, DB host), I use a `config.properties` file. 
For bulk repetitive data (like 100 invalid login attempts), I use Excel or JSON with `@DataProvider`.
However, for dynamic data (like an Email address which must be unique every time), hardcoding fails. I use Java libraries like `Faker` to generate random names, emails, and phone numbers at runtime: `String email = faker.internet().emailAddress();`. If I need to pass dynamic data generated in Test A (like an Order ID) to Test B, I store it in a singleton Context class or a `ThreadLocal` map to maintain thread safety."

**Q: 50. Your framework needs to support both web and API testing. How do you design it?**
**A:** "In my current project, we have a Hybrid framework covering both. I keep the core utilities (ConfigReader, DB helpers, ExtentReports) shared.
For Web, I have the standard POM classes and WebDriver setup.
For API, I integrate RestAssured. I create an `ApiEndpoints` class storing URIs and a `RestUtils` class for GET/POST actions. 
The real power is combining them: I can write a test that creates a user via RestAssured API (fast), then logs into the Web UI using Selenium to verify the user appears in the dashboard. This approach is highly efficient and provides end-to-end coverage."

**Q: 51. How do you decide what to automate and what NOT to automate?**
**A:** "Automation is not a silver bullet. I prioritize automating Sanity/Smoke tests, highly repetitive regression tests, data-driven tests, and complex calculations.
I explicitly DO NOT automate:
1. Tests that require human observation (like visual design, UX, or color checking).
2. Features that are highly unstable or actively changing in current sprints.
3. Edge cases that only happen once in a blue moon—the ROI on writing automation for it is too low.
4. Captchas or Multi-Factor Authentication (MFA) unless we have a backdoor API to bypass them."

**Q: 52. 🔥 Explain a challenging automation problem you faced and how you solved it**
**A:** "In a recent project, we had to automate a dashboard that heavily used shadow DOM elements (Web Components). Standard Selenium `By.xpath` or `By.cssSelector` simply could not see inside the shadow root.
It caused a lot of failures. I solved it by using JavaScriptExecutor to pierce the shadow root. I wrote a custom utility method `getShadowElement(WebElement host, String css)` that executed `return arguments[0].shadowRoot.querySelector(arguments[1])`. Later, when we migrated to Selenium 4, I updated the framework to use the native `SearchContext shadow = element.getShadowRoot();` feature. It was a great learning experience."

**Q: 53. How do you maintain your automation suite? What is your regression strategy?**
**A:** "Maintenance is an ongoing activity. I run the suite nightly via Jenkins. Every morning, the QA team triages the failures. If a test fails due to a UI change, we update the POM locator. If it's flaky, we fix the synchronization. 
For regression, we categorize tests using TestNG `@Test(groups = {"smoke", "regression"})`. For minor releases, we trigger the smoke suite. For major releases, we trigger the full regression suite. We also regularly prune the suite—if a feature is deprecated, we delete the automation code to keep the framework lightweight."

**Q: 54. How do you handle environment-specific configurations?**
**A:** "I use maven profiles and properties files. I maintain multiple config files: `qa.properties`, `staging.properties`. 
In my maven command, I pass `mvn test -Denv=qa`. My `ConfigReader` utility reads this system property. If `env` is 'qa', it loads `qa.properties`. This dynamically sets the `app_url`, database strings, and API endpoints for that specific environment. This completely decoupled design allows Jenkins to execute tests on any environment without changing a single line of Java code."

**Q: 55. 🔥 Tell me about a time when your automation caught a critical bug**
**A:** "We had an e-commerce application. I had automated the end-to-end checkout flow and scheduled it to run every night on the Staging environment. 
One morning, the automation report showed a failure at the payment gateway step. A developer had pushed a backend optimization late at night that accidentally stripped the authentication token from the payment API payload. Manual QA wouldn't have caught it until days later during the manual regression phase. Because my automation caught it immediately, the dev rolled back the change before the release candidate went to Production, saving the company from major revenue loss."

## PART E: IMPORTANT DIFFERENCES (QUICK REFERENCE)

**Q: 56. close() vs quit()**
| Feature | driver.close() | driver.quit() |
|---------|----------------|---------------|
| **Action** | Closes only the current browser window | Closes all windows and ends the WebDriver session |
| **Session ID** | Session ID remains active | Session ID becomes null |
| **Usage** | When handling multiple pop-up windows | Used in `@AfterMethod` to tear down completely |

**Q: 57. get() vs navigate().to()**
| Feature | driver.get("url") | driver.navigate().to("url") |
|---------|-------------------|-----------------------------|
| **Wait** | Waits till the page loads completely | Does not necessarily wait for page load |
| **History** | Does not maintain browser history | Maintains history |
| **Actions** | No forward/back actions | Supports `back()`, `forward()`, `refresh()` |

**Q: 58. findElement() vs findElements()**
| Feature | findElement() | findElements() |
|---------|---------------|----------------|
| **Returns** | A single `WebElement` | A `List<WebElement>` |
| **If not found**| Throws `NoSuchElementException` | Returns an empty List (size 0), no exception |
| **Usage** | Interacting with single buttons/inputs | Counting rows in a table or verifying lists |

**Q: 59. Implicit vs Explicit vs Fluent Wait**
| Wait Type | Characteristics |
|-----------|-----------------|
| **Implicit** | Global wait applied to `driver`. Polls DOM for specified time. Fails if element not found. |
| **Explicit** | Applied to specific elements. Waits for conditions (e.g., `elementToBeClickable`). Best practice. |
| **Fluent** | Advanced Explicit wait where you can define polling frequency and ignore specific exceptions (e.g., `NoSuchElementException`). |

**Q: 60. POM vs PageFactory**
*(See Question 3 for the detailed table)*

**Q: 61. Assert vs Verify (Hard vs Soft Assert)**
| Feature | Hard Assert (`Assert.assertEquals`) | Soft Assert (`SoftAssert.assertEquals`) |
|---------|-------------------------------------|-----------------------------------------|
| **Execution** | Test execution aborts immediately on failure | Test execution continues even if assertion fails |
| **Usage** | Critical validations (e.g., Login success) | Minor validations (e.g., multiple text checks on a page) |
| **Requirement**| None | Must call `softAssert.assertAll()` at the end |

**Q: 62. TestNG Annotations Hierarchy**
**A:** The execution order is:
`@BeforeSuite` -> `@BeforeTest` -> `@BeforeClass` -> `@BeforeMethod` -> `@Test` -> `@AfterMethod` -> `@AfterClass` -> `@AfterTest` -> `@AfterSuite`.

**Q: 63. Selenium 3 vs Selenium 4**
| Feature | Selenium 3 | Selenium 4 |
|---------|------------|------------|
| **Architecture** | Used JSON Wire Protocol | W3C Standardized (Direct communication, no encoding/decoding) |
| **Locators** | Standard locators | Introduced Relative Locators (`above`, `below`, `toLeftOf`) |
| **Windows** | Hard to handle new tabs | Native `switchTo().newWindow(WindowType.TAB)` |
| **Grid** | Hub & Node | Fully distributed architecture with Docker support |

**Q: 64. TestNG vs JUnit**
| Feature | TestNG | JUnit (4/5) |
|---------|--------|-------------|
| **Annotations**| `@BeforeMethod`, `@DataProvider` | `@BeforeEach`, `@ParameterizedTest` |
| **Grouping** | Supported natively (`groups="smoke"`) | Handled via `@Tag` or Suites |
| **Parallelism**| Built-in via XML | Requires external configuration |
| **Listeners** | Native robust Listener support | Less intuitive listener support |

**Q: 65. Maven vs Gradle**
| Feature | Maven | Gradle |
|---------|-------|--------|
| **File format** | `pom.xml` (XML based) | `build.gradle` (Groovy/Kotlin DSL) |
| **Performance** | Standard execution | Highly optimized, incremental builds, much faster |
| **Flexibility** | Highly structured, rigid | Highly customizable |

**Q: 66. Git merge vs Git rebase**
| Feature | Git Merge | Git Rebase |
|---------|-----------|------------|
| **History** | Creates a new merge commit. History is non-linear. | Rewrites history. Creates a perfectly linear, clean history. |
| **Conflict Handling**| Handle all conflicts at once in the merge commit | Handle conflicts commit-by-commit |
| **Safety** | Non-destructive | Can be destructive if rebasing public branches |

**Q: 67. API testing vs UI testing**
| Feature | API Testing (RestAssured) | UI Testing (Selenium) |
|---------|---------------------------|-----------------------|
| **Speed** | Extremely fast (milliseconds) | Slow (seconds to minutes) |
| **Stability**| Highly stable, immune to UI changes | Prone to flakiness due to DOM changes/rendering |
| **Focus** | Business logic & Data layer | Presentation layer & User experience |

**Q: 68. Smoke vs Sanity vs Regression**
| Feature | Smoke Testing | Sanity Testing | Regression Testing |
|---------|---------------|----------------|--------------------|
| **Purpose** | Verify basic build stability | Verify specific component bug fixes | Verify entire app after changes |
| **Scope** | Shallow and wide | Deep and narrow | Deep and wide |
| **Execution** | Done by QA/Dev on initial deployment | Done by QA on minor builds | Done by QA before release |

**Q: 69. BDD vs TDD**
| Feature | TDD (Test-Driven Dev) | BDD (Behavior-Driven Dev) |
|---------|-----------------------|---------------------------|
| **Focus** | Implementation/Code logic | System behavior/Business requirements |
| **Language**| Programming language (Java) | Plain English (Gherkin: Given, When, Then) |
| **Audience**| Developers | Devs, QA, Product Owners, Business Analysts |

**Q: 70. Static testing vs Dynamic testing**
| Feature | Static Testing | Dynamic Testing |
|---------|----------------|-----------------|
| **Execution** | Code is NOT executed | Code IS executed |
| **Methods** | Reviews, Walkthroughs, SonarQube | Unit, Integration, UI Automation tests |
| **Phase** | Verification phase (Early SDLC) | Validation phase (Later SDLC) |
| **Cost** | Cheap to fix defects found here | Expensive to fix defects found here |

---
**End of Document. Good luck with your Infosys L2 Interview!**
