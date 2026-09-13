# 🚀 Selenium + Java SDET Interview Guide — PART 2 (Missing Topics)

> **Extends Part 1** with: BDD/Cucumber, Allure Reports, Log4j2, Shadow DOM, Database Testing, Docker Grid, Appium, EventListener, Proxy, TestNG Listeners, Faker, OAuth bypass, AssertJ, Maven setup, Git strategy, CAPTCHA handling, and more advanced edge cases.

---

## 📋 Additional Topics

27. [Cucumber BDD — Full Deep Dive](#27-cucumber-bdd--full-deep-dive)
28. [Allure Reporting](#28-allure-reporting)
29. [Log4j2 Logging in Framework](#29-log4j2-logging-in-framework)
30. [Shadow DOM Handling](#30-shadow-dom-handling)
31. [Database Testing with JDBC](#31-database-testing-with-jdbc)
32. [Docker + Selenium Grid 4](#32-docker--selenium-grid-4)
33. [Appium Basics (Mobile Awareness)](#33-appium-basics-mobile-awareness)
34. [WebDriver EventListener / EventFiringDecorator](#34-webdriver-eventlistener--eventfiringdecorator)
35. [Proxy Configuration in Selenium](#35-proxy-configuration-in-selenium)
36. [TestNG Listeners Deep Dive](#36-testng-listeners-deep-dive)
37. [Test Data Management — Faker Library](#37-test-data-management--faker-library)
38. [Handling OAuth / SSO / JWT in Tests](#38-handling-oauth--sso--jwt-in-tests)
39. [AssertJ — Fluent Assertions](#39-assertj--fluent-assertions)
40. [Maven pom.xml Full Setup](#40-maven-pomxml-full-setup)
41. [Git Strategy for QA Teams](#41-git-strategy-for-qa-teams)
42. [Handling CAPTCHA in Automation](#42-handling-captcha-in-automation)
43. [Dynamic Tables — Sort, Paginate, Search](#43-dynamic-tables--sort-paginate-search)
44. [Handling Tooltips & Hover Elements](#44-handling-tooltips--hover-elements)
45. [WebDriver Chrome DevTools Protocol (CDP)](#45-webdriver-chrome-devtools-protocol-cdp)
46. [Custom ExpectedConditions](#46-custom-expectedconditions)
47. [Handling OAuth/SSO Login via Cookies](#47-handling-multi-auth-flows)
48. [Commonly Misunderstood Concepts](#48-commonly-misunderstood-concepts)

---

## 27. Cucumber BDD — Full Deep Dive

### Q: Explain BDD with Cucumber. How do you integrate it with Selenium?

**BDD = Behavior Driven Development** — Tests written in plain English (Gherkin) so non-technical stakeholders can understand them.

```
Feature File (Gherkin) → Step Definitions (Java) → Page Objects → WebDriver
```

```gherkin
# src/test/resources/features/Login.feature
Feature: User Login Functionality
  As a registered user
  I want to login to the application
  So that I can access my account

  Background:
    Given the user is on the login page

  @smoke @regression
  Scenario: Successful login with valid credentials
    When the user enters username "admin@test.com" and password "Admin@123"
    And the user clicks on the Login button
    Then the user should be redirected to Dashboard
    And the welcome message should contain "Welcome"

  @regression @negative
  Scenario: Login fails with invalid password
    When the user enters username "admin@test.com" and password "Wrong@123"
    And the user clicks on the Login button
    Then an error message "Invalid username or password" should be displayed

  @datadriven
  Scenario Outline: Login with multiple users
    When the user enters username "<username>" and password "<password>"
    And the user clicks on the Login button
    Then the result should be "<result>"

    Examples:
      | username          | password    | result  |
      | admin@test.com    | Admin@123   | success |
      | user@test.com     | User@456    | success |
      | invalid@test.com  | wrong       | failure |
```

```java
// ===== StepDefinitions/LoginSteps.java =====
package stepdefinitions;

import io.cucumber.java.en.*;
import io.cucumber.java.Before;
import io.cucumber.java.After;
import io.cucumber.java.Scenario;
import org.testng.Assert;
import pages.*;
import utils.DriverFactory;

public class LoginSteps {

    private LoginPage loginPage;
    private DashboardPage dashboardPage;

    @Before
    public void setUp(Scenario scenario) {
        DriverFactory.setDriver("chrome");
        DriverFactory.getDriver().get("https://example.com/login");
        loginPage = new LoginPage(DriverFactory.getDriver());
    }

    @Given("the user is on the login page")
    public void theUserIsOnLoginPage() {
        Assert.assertTrue(loginPage.isLoginPageDisplayed(), "Not on login page!");
    }

    @When("the user enters username {string} and password {string}")
    public void theUserEntersCredentials(String username, String password) {
        loginPage.enterUsername(username);
        loginPage.enterPassword(password);
    }

    @And("the user clicks on the Login button")
    public void theUserClicksLogin() {
        loginPage.clickLoginButton();
    }

    @Then("the user should be redirected to Dashboard")
    public void theUserIsRedirectedToDashboard() {
        dashboardPage = new DashboardPage(DriverFactory.getDriver());
        Assert.assertTrue(DriverFactory.getDriver().getCurrentUrl().contains("dashboard"));
    }

    @Then("the welcome message should contain {string}")
    public void theWelcomeMessageContains(String expected) {
        Assert.assertTrue(dashboardPage.getWelcomeMessage().contains(expected));
    }

    @Then("an error message {string} should be displayed")
    public void errorMessageShouldBeDisplayed(String expectedError) {
        Assert.assertEquals(loginPage.getErrorMessage(), expectedError);
    }

    @Then("the result should be {string}")
    public void theResultShouldBe(String result) {
        if (result.equals("success")) {
            Assert.assertTrue(DriverFactory.getDriver().getCurrentUrl().contains("dashboard"));
        } else {
            Assert.assertTrue(loginPage.isErrorDisplayed());
        }
    }

    @After
    public void tearDown(Scenario scenario) {
        // Attach screenshot on failure
        if (scenario.isFailed()) {
            byte[] screenshot = ((TakesScreenshot) DriverFactory.getDriver())
                    .getScreenshotAs(OutputType.BYTES);
            scenario.attach(screenshot, "image/png", scenario.getName() + "_failure");
        }
        DriverFactory.quitDriver();
    }
}
```

```java
// ===== TestRunner.java =====
package runner;

import io.cucumber.testng.AbstractTestNGCucumberTests;
import io.cucumber.testng.CucumberOptions;
import org.testng.annotations.DataProvider;

@CucumberOptions(
    features = "src/test/resources/features",     // feature files location
    glue = {"stepdefinitions", "hooks"},           // step def packages
    tags = "@smoke or @regression",               // run only tagged scenarios
    plugin = {
        "pretty",                                  // console output
        "html:target/cucumber-reports/report.html",
        "json:target/cucumber-reports/report.json",
        "io.qameta.allure.cucumber7jvm.AllureCucumber7Jvm" // Allure plugin
    },
    monochrome = true,                             // cleaner console
    dryRun = false                                 // true to validate steps exist
)
public class TestRunner extends AbstractTestNGCucumberTests {

    // Enable parallel execution at scenario level
    @Override
    @DataProvider(parallel = true)
    public Object[][] scenarios() {
        return super.scenarios();
    }
}
```

```java
// ===== Hooks.java — Global Before/After =====
package hooks;

import io.cucumber.java.Before;
import io.cucumber.java.After;
import io.cucumber.java.BeforeStep;
import io.cucumber.java.AfterStep;
import io.cucumber.java.Scenario;

public class Hooks {

    // Runs before EVERY scenario
    @Before(order = 1) // order = runs first
    public void globalSetup() {
        System.out.println("=== Starting Scenario ===");
    }

    // Runs before specific tagged scenarios
    @Before("@requiresLogin")
    public void loginSetup() {
        // Pre-authenticate via cookie
    }

    @After(order = 1) // order = runs last
    public void globalTearDown(Scenario scenario) {
        System.out.println("=== Scenario Status: " + scenario.getStatus() + " ===");
    }

    @BeforeStep
    public void beforeEachStep(Scenario scenario) {
        System.out.println("Step: " + scenario.getName());
    }

    @AfterStep
    public void afterEachStep(Scenario scenario) {
        // Take screenshot after each step (for debug reports)
    }
}
```

```java
// ===== ScenarioContext.java — Share data between steps (Dependency Injection) =====
package context;

import java.util.HashMap;
import java.util.Map;

// Use PicoContainer for DI in Cucumber
public class ScenarioContext {

    private Map<String, Object> context = new HashMap<>();

    public void set(String key, Object value) {
        context.put(key, value);
    }

    public Object get(String key) {
        return context.get(key);
    }

    public String getString(String key) {
        return (String) context.get(key);
    }
}

// In Step Definition — inject via constructor
public class ProductSteps {
    private ScenarioContext scenarioContext;

    // PicoContainer injects this automatically
    public ProductSteps(ScenarioContext scenarioContext) {
        this.scenarioContext = scenarioContext;
    }

    @When("user adds product {string} to cart")
    public void addToCart(String productName) {
        // ... add product
        scenarioContext.set("addedProduct", productName); // share with next step
    }

    @Then("the cart should contain {string}")
    public void verifyCart(String expected) {
        String addedProduct = scenarioContext.getString("addedProduct"); // retrieve
        Assert.assertEquals(addedProduct, expected);
    }
}
```

---

## 28. Allure Reporting

### Q: How do you set up Allure Reports? Difference from ExtentReports?

```xml
<!-- pom.xml dependencies -->
<dependency>
    <groupId>io.qameta.allure</groupId>
    <artifactId>allure-testng</artifactId>
    <version>2.24.0</version>
</dependency>
```

```java
import io.qameta.allure.*;
import io.qameta.allure.model.Status;
import org.testng.annotations.*;

@Epic("User Management")           // top-level grouping
@Feature("Authentication")         // feature grouping
public class LoginTest {

    @Test
    @Story("US-101: User Login")   // user story
    @Severity(SeverityLevel.CRITICAL)
    @Description("Verify user can login with valid credentials")
    @Link(name = "JIRA", url = "https://jira.company.com/US-101")
    @Owner("John Doe")
    public void testValidLogin() {

        Allure.step("Navigate to login page", () -> {
            driver.get("https://example.com/login");
        });

        Allure.step("Enter username", () -> {
            loginPage.enterUsername("admin@test.com");
        });

        Allure.step("Enter password", () -> {
            loginPage.enterPassword("Admin@123");
        });

        Allure.step("Click login and verify dashboard", () -> {
            DashboardPage dashboard = loginPage.clickLogin();
            Assert.assertTrue(dashboard.getWelcomeMessage().contains("Welcome"));
        });
    }

    // Attach screenshot manually
    @Attachment(value = "Screenshot", type = "image/png")
    public byte[] captureScreenshot() {
        return ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
    }

    // Attach any text
    @Attachment(value = "Page Source", type = "text/html")
    public String capturePageSource() {
        return driver.getPageSource();
    }

    // Dynamic attachment
    public void attachScreenshotToReport(String name) {
        byte[] screenshot = ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
        Allure.addAttachment(name, "image/png", new ByteArrayInputStream(screenshot), "png");
    }
}

// Allure TestNG Listener — add to testng.xml:
// <listener class-name="io.qameta.allure.testng.AllureTestNg"/>

// Generate report:
// allure serve target/allure-results
```

| Feature | ExtentReports | Allure |
|---|---|---|
| Setup | Simple, pure Java | Requires Allure CLI |
| Look | Good, customizable | Modern, beautiful |
| History Trends | Manual | Built-in history/trends |
| Categories | Manual | Auto (broken, failed) |
| Steps | Manual logging | `@Step` annotation |
| CI Integration | Easy | Easy (allure plugin) |
| **Best For** | Quick reports | Full-featured dashboards |

---

## 29. Log4j2 Logging in Framework

### Q: How do you add proper logging to your test framework?

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.23.1</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.23.1</version>
</dependency>
```

```xml
<!-- src/main/resources/log4j2.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
    <Appenders>
        <!-- Console output -->
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} [%t] %-5level %logger{36} - %msg%n"/>
        </Console>
        <!-- Rolling file — creates new file daily, max 10 backups -->
        <RollingFile name="RollingFile" fileName="logs/automation.log"
                     filePattern="logs/automation-%d{yyyy-MM-dd}-%i.log.gz">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} [%t] %-5level %logger{36} - %msg%n"/>
            <Policies>
                <TimeBasedTriggeringPolicy interval="1"/>
                <SizeBasedTriggeringPolicy size="10MB"/>
            </Policies>
            <DefaultRolloverStrategy max="10"/>
        </RollingFile>
    </Appenders>
    <Loggers>
        <Root level="INFO">
            <AppenderRef ref="Console"/>
            <AppenderRef ref="RollingFile"/>
        </Root>
    </Loggers>
</Configuration>
```

```java
// ===== LogManager.java =====
package utils;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

public class Log {
    // Use class-specific loggers
    private static final Logger log = LogManager.getLogger(Log.class);

    public static void info(String message) { log.info(message); }
    public static void debug(String message) { log.debug(message); }
    public static void warn(String message) { log.warn(message); }
    public static void error(String message) { log.error(message); }
    public static void fatal(String message) { log.fatal(message); }

    public static void startTestCase(String testName) {
        log.info("========== Starting Test: " + testName + " ==========");
    }
    public static void endTestCase(String testName) {
        log.info("========== Finished Test: " + testName + " ==========");
    }
}

// ===== Usage in page/test classes =====
public class LoginPage {

    private static final Logger log = LogManager.getLogger(LoginPage.class);

    public DashboardPage login(String username, String password) {
        log.info("Logging in with username: " + username);
        enterUsername(username);
        enterPassword(password);
        log.debug("Clicking login button");
        loginBtn.click();
        log.info("Login successful, navigating to dashboard");
        return new DashboardPage(driver);
    }
}
```

---

## 30. Shadow DOM Handling

### Q: How do you handle Shadow DOM elements? (Commonly asked for modern web apps)

**Shadow DOM** = Encapsulated DOM tree attached to an element. Normal `findElement` **cannot** access it.

```java
public class ShadowDOMHandling {

    WebDriver driver;
    JavascriptExecutor js;

    // ===== Method 1: Using JS to access Shadow Root =====
    public WebElement getShadowElement(WebElement shadowHost, String cssSelector) {
        // Get shadow root via JS
        SearchContext shadowRoot = (SearchContext)
            ((JavascriptExecutor) driver).executeScript("return arguments[0].shadowRoot", shadowHost);
        return shadowRoot.findElement(By.cssSelector(cssSelector));
    }

    // Usage
    public void interactWithShadowElement() {
        // Find host element (the element that contains shadow root)
        WebElement shadowHost = driver.findElement(By.cssSelector("my-custom-component"));
        // Access element inside shadow root
        WebElement innerInput = getShadowElement(shadowHost, "input.email-field");
        innerInput.sendKeys("test@example.com");
    }

    // ===== Method 2: Selenium 4 Native Shadow Root =====
    public void nativeShadowRoot() {
        WebElement shadowHost = driver.findElement(By.cssSelector("custom-element"));
        // Selenium 4 direct shadow root access
        SearchContext shadowRoot = shadowHost.getShadowRoot();
        WebElement input = shadowRoot.findElement(By.cssSelector("#username"));
        input.sendKeys("testuser");
    }

    // ===== Nested Shadow DOM =====
    public void nestedShadowDOM() {
        // Level 1
        WebElement host1 = driver.findElement(By.cssSelector("outer-component"));
        SearchContext shadowRoot1 = host1.getShadowRoot();

        // Level 2
        WebElement host2 = shadowRoot1.findElement(By.cssSelector("inner-component"));
        SearchContext shadowRoot2 = host2.getShadowRoot();

        // Target element
        WebElement btn = shadowRoot2.findElement(By.cssSelector("button.submit"));
        btn.click();
    }

    // ⭐ EDGE CASE: Shadow DOM elements are NOT accessible by:
    // - XPath (XPath can't cross shadow boundaries)
    // - driver.findElement() directly
    // Use: CSS selectors + getShadowRoot() OR JS

    // ⭐ Open vs Closed Shadow DOM:
    // Open:   shadowRoot accessible via JS
    // Closed: shadowRoot returns null (very rare, used by browsers internally)
}
```

---

## 31. Database Testing with JDBC

### Q: How do you perform database validation in your Selenium framework?

```java
import java.sql.*;
import java.util.*;

public class DatabaseUtils {

    private Connection connection;
    private static final String DB_URL = "jdbc:mysql://localhost:3306/testdb";
    private static final String DB_USER = "root";
    private static final String DB_PASS = "password";

    // Initialize connection
    public void openConnection() throws SQLException {
        connection = DriverManager.getConnection(DB_URL, DB_USER, DB_PASS);
        System.out.println("DB Connected: " + !connection.isClosed());
    }

    public void closeConnection() throws SQLException {
        if (connection != null && !connection.isClosed()) {
            connection.close();
        }
    }

    // Execute SELECT query — returns list of maps
    public List<Map<String, String>> executeQuery(String query) throws SQLException {
        List<Map<String, String>> results = new ArrayList<>();
        Statement stmt = connection.createStatement();
        ResultSet rs = stmt.executeQuery(query);
        ResultSetMetaData meta = rs.getMetaData();
        int colCount = meta.getColumnCount();

        while (rs.next()) {
            Map<String, String> row = new LinkedHashMap<>();
            for (int i = 1; i <= colCount; i++) {
                row.put(meta.getColumnName(i), rs.getString(i));
            }
            results.add(row);
        }
        rs.close();
        stmt.close();
        return results;
    }

    // Get single value
    public String getSingleValue(String query) throws SQLException {
        Statement stmt = connection.createStatement();
        ResultSet rs = stmt.executeQuery(query);
        String value = rs.next() ? rs.getString(1) : null;
        rs.close();
        stmt.close();
        return value;
    }

    // Execute UPDATE/INSERT/DELETE
    public int executeUpdate(String query) throws SQLException {
        Statement stmt = connection.createStatement();
        int rowsAffected = stmt.executeUpdate(query);
        stmt.close();
        return rowsAffected;
    }

    // Prepared statement (prevents SQL injection)
    public boolean isUserExistsInDB(String email) throws SQLException {
        String query = "SELECT COUNT(*) FROM users WHERE email = ?";
        PreparedStatement ps = connection.prepareStatement(query);
        ps.setString(1, email);
        ResultSet rs = ps.executeQuery();
        rs.next();
        boolean exists = rs.getInt(1) > 0;
        rs.close();
        ps.close();
        return exists;
    }

    // ⭐ Real-world use case: UI + DB validation
    @Test
    public void verifyUserCreationInDB() throws Exception {
        DatabaseUtils db = new DatabaseUtils();
        db.openConnection();

        // Step 1: Create user via UI
        driver.get("https://example.com/register");
        new RegisterPage(driver).registerUser("testuser@mail.com", "Test@123");

        // Step 2: Verify in DB
        Assert.assertTrue(
            db.isUserExistsInDB("testuser@mail.com"),
            "User was not created in DB!"
        );

        // Step 3: Verify user role in DB
        String role = db.getSingleValue(
            "SELECT role FROM users WHERE email = 'testuser@mail.com'"
        );
        Assert.assertEquals(role, "USER", "Wrong default role assigned");

        // Step 4: Cleanup — delete test user from DB
        db.executeUpdate("DELETE FROM users WHERE email = 'testuser@mail.com'");
        db.closeConnection();
    }
}
```

---

## 32. Docker + Selenium Grid 4

### Q: How do you set up Selenium Grid with Docker? (Very commonly asked at senior level)

```yaml
# docker-compose.yml — Selenium Grid 4
version: "3.8"
services:
  selenium-hub:
    image: selenium/hub:4.18.1
    container_name: selenium-hub
    ports:
      - "4442:4442"
      - "4443:4443"
      - "4444:4444"
    environment:
      - SE_NODE_MAX_SESSIONS=5

  chrome-node:
    image: selenium/node-chrome:4.18.1
    container_name: chrome-node
    shm_size: '2gb'
    depends_on:
      - selenium-hub
    environment:
      - SE_EVENT_BUS_HOST=selenium-hub
      - SE_EVENT_BUS_PUBLISH_PORT=4442
      - SE_EVENT_BUS_SUBSCRIBE_PORT=4443
      - SE_NODE_MAX_SESSIONS=3
      - SE_NODE_SESSION_TIMEOUT=300
    volumes:
      - /dev/shm:/dev/shm

  firefox-node:
    image: selenium/node-firefox:4.18.1
    container_name: firefox-node
    shm_size: '2gb'
    depends_on:
      - selenium-hub
    environment:
      - SE_EVENT_BUS_HOST=selenium-hub
      - SE_EVENT_BUS_PUBLISH_PORT=4442
      - SE_EVENT_BUS_SUBSCRIBE_PORT=4443
      - SE_NODE_MAX_SESSIONS=3

  # VNC for debugging — see browser in real time
  chrome-node-debug:
    image: selenium/node-chrome:4.18.1
    shm_size: '2gb'
    depends_on:
      - selenium-hub
    environment:
      - SE_EVENT_BUS_HOST=selenium-hub
      - SE_EVENT_BUS_PUBLISH_PORT=4442
      - SE_EVENT_BUS_SUBSCRIBE_PORT=4443
      - SE_VNC_NO_PASSWORD=1
    ports:
      - "7900:7900"  # noVNC — open http://localhost:7900 in browser
```

```bash
# Start the grid
docker-compose up -d

# Check status — http://localhost:4444/ui
# View VNC — http://localhost:7900

# Stop grid
docker-compose down
```

```java
// Connect Java tests to Docker Grid
public class DockerGridSetup {

    public WebDriver getDockerGridDriver(String browser) throws Exception {
        ChromeOptions chromeOptions = new ChromeOptions();
        FirefoxOptions firefoxOptions = new FirefoxOptions();

        MutableCapabilities options = browser.equalsIgnoreCase("firefox")
            ? firefoxOptions : chromeOptions;

        // Connect to Docker Grid hub
        return new RemoteWebDriver(new URL("http://localhost:4444"), options);
    }
}

// Dockerfile for running tests IN Docker
/*
FROM maven:3.9-openjdk-17-slim
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
CMD ["mvn", "test", "-Dbrowser=chrome", "-Denv=qa", "-Dheadless=true"]
*/
```

---

## 33. Appium Basics (Mobile Awareness)

### Q: What is Appium? How is it different from Selenium? (Awareness level expected)

```java
import io.appium.java_client.*;
import io.appium.java_client.android.*;
import io.appium.java_client.ios.*;
import org.openqa.selenium.remote.DesiredCapabilities;
import java.net.URL;

public class AppiumBasics {

    // ===== Android Setup =====
    public AndroidDriver setupAndroid() throws Exception {
        DesiredCapabilities caps = new DesiredCapabilities();
        caps.setCapability("platformName", "Android");
        caps.setCapability("platformVersion", "13");
        caps.setCapability("deviceName", "Pixel_6_API_33");
        caps.setCapability("automationName", "UiAutomator2"); // Appium driver
        caps.setCapability("appPackage", "com.example.app");
        caps.setCapability("appActivity", ".MainActivity");
        caps.setCapability("noReset", true); // don't reset app state

        return new AndroidDriver(new URL("http://localhost:4723"), caps);
    }

    // ===== iOS Setup =====
    public IOSDriver setupIOS() throws Exception {
        DesiredCapabilities caps = new DesiredCapabilities();
        caps.setCapability("platformName", "iOS");
        caps.setCapability("platformVersion", "17.0");
        caps.setCapability("deviceName", "iPhone 15");
        caps.setCapability("automationName", "XCUITest");
        caps.setCapability("bundleId", "com.example.app");
        caps.setCapability("udid", "device-udid-here");

        return new IOSDriver(new URL("http://localhost:4723"), caps);
    }

    // ===== Find Elements (Mobile) =====
    public void mobileLocators(AndroidDriver driver) {
        // Android-specific locators
        driver.findElement(By.id("com.example.app:id/login_btn")); // resource-id
        driver.findElement(By.xpath("//android.widget.Button[@text='Login']"));
        driver.findElement(AppiumBy.accessibilityId("Login Button")); // accessibility label
        driver.findElement(AppiumBy.androidUIAutomator(
            "new UiSelector().text(\"Login\").className(\"android.widget.Button\")"
        ));
    }

    // ===== Mobile Gestures =====
    public void mobileGestures(AndroidDriver driver) {
        // Scroll
        driver.findElement(AppiumBy.androidUIAutomator(
            "new UiScrollable(new UiSelector().scrollable(true))" +
            ".scrollIntoView(new UiSelector().text(\"Privacy Policy\"))"
        ));

        // Swipe (Selenium Actions in Appium 2)
        PointerInput finger = new PointerInput(PointerInput.Kind.TOUCH, "finger");
        Sequence swipe = new Sequence(finger, 0)
            .addAction(finger.createPointerMove(Duration.ZERO, PointerInput.Origin.viewport(), 500, 700))
            .addAction(finger.createPointerDown(PointerInput.MouseButton.LEFT.asArg()))
            .addAction(finger.createPointerMove(Duration.ofMillis(600), PointerInput.Origin.viewport(), 500, 200))
            .addAction(finger.createPointerUp(PointerInput.MouseButton.LEFT.asArg()));
        driver.perform(Arrays.asList(swipe));

        // Long press
        driver.findElement(By.id("com.example:id/item")).click();
        // (use Actions or TouchAction for long press)
    }

    /* ===== Selenium vs Appium =====
     *
     * Selenium                          Appium
     * ─────────────────────────────     ─────────────────────────────────
     * Web browsers                      Mobile apps (native/hybrid/web)
     * ChromeDriver, GeckoDriver         UiAutomator2, XCUITest
     * W3C WebDriver                     W3C WebDriver (extended)
     * WebElement                        MobileElement (extended WebElement)
     * CSS, XPath locators               accessibility-id, Android UIAutomator
     * No device needed                  Real device or emulator needed
     */
}
```

---

## 34. WebDriver EventListener / EventFiringDecorator

### Q: How do you implement event listeners to auto-log/screenshot every action?

```java
import org.openqa.selenium.support.events.*;
import org.openqa.selenium.*;

// ===== CustomEventListener.java =====
public class CustomEventListener implements WebDriverListener {

    private static final Logger log = LogManager.getLogger(CustomEventListener.class);

    // Before and after clicking
    @Override
    public void beforeClick(WebElement element) {
        log.info("Clicking element: " + getElementDescription(element));
    }

    @Override
    public void afterClick(WebElement element) {
        log.info("Clicked successfully");
    }

    // Before and after typing
    @Override
    public void beforeSendKeys(WebElement element, CharSequence... keysToSend) {
        log.info("Typing into: " + getElementDescription(element) +
                 " | Value: " + String.join("", (CharSequence[]) keysToSend));
    }

    // Before navigation
    @Override
    public void beforeGet(WebDriver driver, String url) {
        log.info("Navigating to: " + url);
    }

    @Override
    public void afterGet(WebDriver driver, String url) {
        log.info("Page title: " + driver.getTitle());
    }

    // On exception — auto-screenshot
    @Override
    public void onException(WebDriver driver, Throwable throwable) {
        log.error("Exception occurred: " + throwable.getMessage());
        // Auto-capture screenshot
        File screenshot = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);
        log.error("Screenshot saved: " + screenshot.getAbsolutePath());
    }

    // Before finding element
    @Override
    public void beforeFindElement(WebDriver driver, By locator) {
        log.debug("Finding element: " + locator);
    }

    private String getElementDescription(WebElement element) {
        try {
            return element.getTagName() + "[" + element.getAttribute("id") + "]";
        } catch (Exception e) {
            return "unknown element";
        }
    }
}

// ===== Wrap driver with EventFiringDecorator =====
public class DriverFactory {

    public static WebDriver setDriver(String browser) {
        WebDriver rawDriver = createRawDriver(browser);

        // Wrap with event listener
        EventFiringDecorator<WebDriver> decorator =
            new EventFiringDecorator<>(new CustomEventListener());
        WebDriver decoratedDriver = decorator.decorate(rawDriver);

        driverThread.set(decoratedDriver);
        return decoratedDriver;
    }
}
```

---

## 35. Proxy Configuration in Selenium

### Q: How do you configure proxy settings for Selenium? (Corporate networks, traffic capture)

```java
import org.openqa.selenium.Proxy;

public class ProxySetup {

    // ===== Manual Proxy (Corporate / BrowserMob) =====
    public WebDriver setupWithProxy() {
        Proxy proxy = new Proxy();
        proxy.setHttpProxy("proxy.company.com:8080");
        proxy.setSslProxy("proxy.company.com:8080");
        // Bypass proxy for these hosts
        proxy.setNoProxy("localhost,127.0.0.1,.internal.company.com");

        ChromeOptions options = new ChromeOptions();
        options.setProxy(proxy);
        return new ChromeDriver(options);
    }

    // ===== BrowserMob Proxy — Capture network traffic =====
    public WebDriver setupBrowserMobProxy() throws Exception {
        BrowserMobProxy bmpProxy = new BrowserMobProxyServer();
        bmpProxy.start(0); // random port

        // Enable request/response capture
        bmpProxy.enableHarCaptureTypes(CaptureType.REQUEST_CONTENT, CaptureType.RESPONSE_CONTENT);
        bmpProxy.newHar("test-traffic");

        // Add request filter (e.g., add auth header)
        bmpProxy.addRequestFilter((request, contents, messageInfo) -> {
            request.headers().add("X-Custom-Header", "automation");
            return null;
        });

        // Add response filter
        bmpProxy.addResponseFilter((response, contents, messageInfo) -> {
            System.out.println("Response: " + response.getStatus() + " - " + messageInfo.getUrl());
        });

        Proxy seleniumProxy = ClientUtil.createSeleniumProxy(bmpProxy);
        ChromeOptions options = new ChromeOptions();
        options.setProxy(seleniumProxy);
        WebDriver driver = new ChromeDriver(options);

        // ... run tests ...

        // Get HAR file (HTTP Archive — all network traffic)
        Har har = bmpProxy.getHar();
        har.writeTo(new File("network-traffic.har"));
        bmpProxy.stop();

        return driver;
    }

    // ===== Chrome DevTools Protocol — Network Interception (Selenium 4) =====
    public void interceptNetwork(WebDriver driver) {
        DevTools devTools = ((ChromeDriver) driver).getDevTools();
        devTools.createSession();

        // Enable network monitoring
        devTools.send(Network.enable(Optional.empty(), Optional.empty(), Optional.empty()));

        // Listen to all requests
        devTools.addListener(Network.requestWillBeSent(), request -> {
            System.out.println("Request: " + request.getRequest().getUrl());
        });

        // Listen to responses
        devTools.addListener(Network.responseReceived(), response -> {
            System.out.println("Response: " + response.getResponse().getStatus());
        });

        driver.get("https://example.com");
    }
}
```

---

## 36. TestNG Listeners Deep Dive

### Q: Explain all TestNG listeners and when to use each.

```java
// ===== 1. ITestListener — most commonly used =====
public class TestListener implements ITestListener {
    void onTestStart(ITestResult result) {}        // before test method
    void onTestSuccess(ITestResult result) {}      // test passed
    void onTestFailure(ITestResult result) {}      // test failed
    void onTestSkipped(ITestResult result) {}      // test skipped
    void onTestFailedButWithinSuccessPercentage(ITestResult result) {} // partial failure
    void onStart(ITestContext context) {}          // before any test in class
    void onFinish(ITestContext context) {}         // after all tests in class
}

// ===== 2. ISuiteListener — suite-level =====
public class SuiteListener implements ISuiteListener {
    void onStart(ISuite suite) {}    // before entire suite
    void onFinish(ISuite suite) {}   // after entire suite
}

// ===== 3. IMethodInterceptor — change test execution order =====
public class PriorityInterceptor implements IMethodInterceptor {
    @Override
    public List<IMethodInstance> intercept(List<IMethodInstance> methods, ITestContext context) {
        // Sort methods by @Priority annotation value
        methods.sort(Comparator.comparingInt(m -> {
            Priority p = m.getMethod().getConstructorOrMethod().getMethod().getAnnotation(Priority.class);
            return (p != null) ? p.value() : Integer.MAX_VALUE;
        }));
        return methods;
    }
}

// ===== 4. IInvokedMethodListener — before/after every method =====
public class MethodListener implements IInvokedMethodListener {
    @Override
    public void beforeInvocation(IInvokedMethod method, ITestResult testResult) {
        System.out.println("About to run: " + method.getTestMethod().getMethodName());
    }

    @Override
    public void afterInvocation(IInvokedMethod method, ITestResult testResult) {
        System.out.println("Finished: " + method.getTestMethod().getMethodName()
                + " | Status: " + testResult.getStatus());
    }
}

// ===== 5. IRetryAnalyzer — retry failed tests =====
// (shown in Part 1)

// ===== 6. IAnnotationTransformer — modify test annotations at runtime =====
public class RetryTransformer implements IAnnotationTransformer {
    @Override
    public void transform(ITestAnnotation annotation, Class testClass,
                          Constructor testConstructor, Method testMethod) {
        // Apply RetryAnalyzer to ALL tests automatically (without adding to each @Test)
        annotation.setRetryAnalyzer(RetryAnalyzer.class);
    }
}

// Register ALL listeners in testng.xml:
/*
<listeners>
    <listener class-name="listeners.TestListener"/>
    <listener class-name="listeners.SuiteListener"/>
    <listener class-name="listeners.RetryTransformer"/>
    <listener class-name="listeners.MethodListener"/>
</listeners>
*/
```

---

## 37. Test Data Management — Faker Library

### Q: How do you generate dynamic, realistic test data?

```java
import com.github.javafaker.Faker;
import java.util.Locale;

public class TestDataFactory {

    private static final Faker faker = new Faker(new Locale("en-IN")); // Indian locale

    // Generate random user
    public static Map<String, String> generateUser() {
        Map<String, String> user = new HashMap<>();
        user.put("firstName", faker.name().firstName());
        user.put("lastName", faker.name().lastName());
        user.put("email", faker.internet().emailAddress());
        user.put("password", faker.internet().password(8, 16, true, true, true));
        user.put("phone", faker.phoneNumber().cellPhone());
        user.put("dob", faker.date().birthday(18, 60).toString());
        user.put("address", faker.address().streetAddress());
        user.put("city", faker.address().city());
        user.put("pinCode", faker.address().zipCode());
        return user;
    }

    // Unique email (avoid duplication)
    public static String uniqueEmail() {
        return "auto_" + System.currentTimeMillis() + "@testmail.com";
    }

    // Credit card data (for payment testing)
    public static Map<String, String> generateCreditCard() {
        Map<String, String> card = new HashMap<>();
        card.put("number", faker.finance().creditCard(CreditCardType.VISA));
        card.put("name", faker.name().fullName());
        card.put("expiry", "12/28");
        card.put("cvv", "123");
        return card;
    }

    // Generate specific domain data
    public static String generateProductName() {
        return faker.commerce().productName();
    }

    public static double generatePrice() {
        return faker.commerce().price(10.0, 10000.0);
    }

    public static String generateCompanyName() {
        return faker.company().name();
    }

    // ⭐ Usage in test:
    @Test
    public void testUserRegistration() {
        Map<String, String> user = TestDataFactory.generateUser();

        new RegisterPage(driver)
            .enterFirstName(user.get("firstName"))
            .enterLastName(user.get("lastName"))
            .enterEmail(user.get("email"))
            .enterPassword(user.get("password"))
            .submitRegistration();

        Assert.assertTrue(new DashboardPage(driver).isLoaded());
    }
}
```

---

## 38. Handling OAuth / SSO / JWT in Tests

### Q: How do you handle OAuth2 / SSO login in Selenium tests?

```java
import io.restassured.RestAssured;

public class AuthenticationHandling {

    // ===== Method 1: Get JWT via API, set as cookie =====
    public void loginViaJWT(WebDriver driver) {
        // Step 1: Get auth token via REST API (bypasses UI login)
        String token = RestAssured
            .given()
                .contentType("application/json")
                .body("{\"username\":\"admin\",\"password\":\"Admin@123\"}")
            .when()
                .post("https://api.example.com/auth/login")
            .then()
                .statusCode(200)
                .extract().path("access_token");

        // Step 2: Navigate to app
        driver.get("https://app.example.com");

        // Step 3: Set auth cookie/localStorage
        driver.manage().addCookie(new Cookie("auth_token", token,
                ".example.com", "/", null, true, true));

        // Or set in localStorage via JS
        ((JavascriptExecutor) driver).executeScript(
                "localStorage.setItem('access_token', '" + token + "');");

        // Step 4: Refresh to apply
        driver.navigate().refresh();
        // Now you're logged in without going through UI login!
    }

    // ===== Method 2: OAuth2 Authorization Code Flow =====
    public void handleOAuthFlow(WebDriver driver) {
        driver.get("https://app.example.com/login");
        driver.findElement(By.id("oauth-login-btn")).click();

        // Handle OAuth provider page (Google, Azure, Okta)
        new WebDriverWait(driver, Duration.ofSeconds(10))
            .until(ExpectedConditions.urlContains("accounts.google.com"));

        driver.findElement(By.id("identifierId")).sendKeys("testuser@gmail.com");
        driver.findElement(By.id("identifierNext")).click();
        // ... complete OAuth flow
    }

    // ===== Method 3: Windows Authentication (NTLM) =====
    public void handleNTLM(String username, String password) {
        // Embed credentials in URL
        driver.get("https://" + username + ":" + password + "@app.internal.com");
    }

    // ⭐ Best Practice: Never use UI login for every test
    // - UI login: ~3-5 seconds per test
    // - API token: ~200ms per test
    // → Huge time saving in large suites (500+ tests)
}
```

---

## 39. AssertJ — Fluent Assertions

### Q: What is AssertJ? Why is it better than TestNG/JUnit assertions?

```java
import static org.assertj.core.api.Assertions.*;

public class AssertJExamples {

    @Test
    public void testWithAssertJ() {

        // ===== String assertions =====
        String title = driver.getTitle();
        assertThat(title)
            .isNotNull()
            .isNotEmpty()
            .startsWith("Dashboard")
            .containsIgnoringCase("welcome")
            .hasSize(15);

        // ===== Collection assertions =====
        List<String> menuItems = getMenuItems();
        assertThat(menuItems)
            .isNotEmpty()
            .hasSize(5)
            .contains("Home", "Profile")
            .containsExactlyInAnyOrder("Home", "Profile", "Settings", "Orders", "Logout")
            .doesNotContain("Admin Panel")
            .allMatch(item -> !item.isEmpty());

        // ===== Number assertions =====
        int itemCount = getCartItemCount();
        assertThat(itemCount)
            .isGreaterThan(0)
            .isLessThanOrEqualTo(10)
            .isBetween(1, 10);

        // ===== Object assertions =====
        User user = new User("John", "john@test.com", "admin");
        assertThat(user)
            .isNotNull()
            .extracting("name", "email", "role")
            .containsExactly("John", "john@test.com", "admin");

        // ===== Boolean assertions =====
        assertThat(element.isDisplayed()).isTrue();
        assertThat(element.isSelected()).isFalse();

        // ===== SoftAssertions (collect all failures) =====
        SoftAssertions softly = new SoftAssertions();
        softly.assertThat(driver.getTitle()).isEqualTo("Dashboard");
        softly.assertThat(welcomeMsg).contains("Welcome");
        softly.assertThat(cartCount).isZero();
        softly.assertAll(); // throws if any failed

        // ===== Custom error messages =====
        assertThat(driver.getCurrentUrl())
            .as("After login, should redirect to dashboard")
            .contains("dashboard");

        // ===== Exception assertions =====
        assertThatThrownBy(() -> driver.findElement(By.id("nonExistent")))
            .isInstanceOf(NoSuchElementException.class)
            .hasMessageContaining("nonExistent");
    }
}
```

---

## 40. Maven pom.xml Full Setup

### Q: Show a complete pom.xml for a Selenium framework.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.company</groupId>
    <artifactId>selenium-framework</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <properties>
        <java.version>17</java.version>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <selenium.version>4.18.1</selenium.version>
        <testng.version>7.9.0</testng.version>
        <webdrivermanager.version>5.7.0</webdrivermanager.version>
        <extentreports.version>5.1.1</extentreports.version>
        <allure.version>2.24.0</allure.version>
        <restassured.version>5.4.0</restassured.version>
        <poi.version>5.2.5</poi.version>
        <log4j.version>2.23.1</log4j.version>
        <faker.version>1.0.2</faker.version>
        <assertj.version>3.25.3</assertj.version>
        <cucumber.version>7.15.0</cucumber.version>
        <suiteFile>testng.xml</suiteFile>
        <browser>chrome</browser>
        <env>qa</env>
    </properties>

    <dependencies>
        <!-- Selenium -->
        <dependency>
            <groupId>org.seleniumhq.selenium</groupId>
            <artifactId>selenium-java</artifactId>
            <version>${selenium.version}</version>
        </dependency>

        <!-- WebDriverManager -->
        <dependency>
            <groupId>io.github.bonigarcia</groupId>
            <artifactId>webdrivermanager</artifactId>
            <version>${webdrivermanager.version}</version>
        </dependency>

        <!-- TestNG -->
        <dependency>
            <groupId>org.testng</groupId>
            <artifactId>testng</artifactId>
            <version>${testng.version}</version>
        </dependency>

        <!-- Cucumber -->
        <dependency>
            <groupId>io.cucumber</groupId>
            <artifactId>cucumber-java</artifactId>
            <version>${cucumber.version}</version>
        </dependency>
        <dependency>
            <groupId>io.cucumber</groupId>
            <artifactId>cucumber-testng</artifactId>
            <version>${cucumber.version}</version>
        </dependency>
        <dependency>
            <groupId>io.cucumber</groupId>
            <artifactId>cucumber-picocontainer</artifactId>
            <version>${cucumber.version}</version>
        </dependency>

        <!-- ExtentReports -->
        <dependency>
            <groupId>com.aventstack</groupId>
            <artifactId>extentreports</artifactId>
            <version>${extentreports.version}</version>
        </dependency>

        <!-- Allure -->
        <dependency>
            <groupId>io.qameta.allure</groupId>
            <artifactId>allure-testng</artifactId>
            <version>${allure.version}</version>
        </dependency>

        <!-- RestAssured -->
        <dependency>
            <groupId>io.rest-assured</groupId>
            <artifactId>rest-assured</artifactId>
            <version>${restassured.version}</version>
        </dependency>

        <!-- Apache POI (Excel) -->
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-ooxml</artifactId>
            <version>${poi.version}</version>
        </dependency>

        <!-- Log4j2 -->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>${log4j.version}</version>
        </dependency>

        <!-- Faker -->
        <dependency>
            <groupId>com.github.javafaker</groupId>
            <artifactId>javafaker</artifactId>
            <version>${faker.version}</version>
        </dependency>

        <!-- AssertJ -->
        <dependency>
            <groupId>org.assertj</groupId>
            <artifactId>assertj-core</artifactId>
            <version>${assertj.version}</version>
        </dependency>

        <!-- Commons IO (screenshots) -->
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.15.1</version>
        </dependency>

        <!-- Jackson (JSON) -->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.17.0</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- Compiler -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.12.1</version>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                </configuration>
            </plugin>

            <!-- Surefire — runs TestNG tests -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.2.5</version>
                <configuration>
                    <suiteXmlFiles>
                        <suiteXmlFile>src/test/resources/${suiteFile}</suiteXmlFile>
                    </suiteXmlFiles>
                    <systemPropertyVariables>
                        <browser>${browser}</browser>
                        <env>${env}</env>
                    </systemPropertyVariables>
                    <testFailureIgnore>false</testFailureIgnore>
                </configuration>
            </plugin>

            <!-- Allure Maven plugin -->
            <plugin>
                <groupId>io.qameta.allure</groupId>
                <artifactId>allure-maven</artifactId>
                <version>2.12.0</version>
                <configuration>
                    <reportVersion>${allure.version}</reportVersion>
                </configuration>
            </plugin>
        </plugins>
    </build>

    <!-- Run specific browser: mvn test -Dbrowser=firefox -Denv=staging -->
</project>
```

---

## 41. Git Strategy for QA Teams

### Q: What branching strategy do you follow for test automation?

```
Main Branches:
main          → production-ready tests (mirrors app release)
develop       → integration branch for new tests

Feature Branches:
feature/US-101-login-tests     → new test cases
fix/SDET-45-flaky-login-test   → fixing unstable tests
chore/upgrade-selenium-4       → dependency upgrades
```

```bash
# Daily workflow
git checkout develop
git pull origin develop
git checkout -b feature/US-201-payment-tests

# Write tests...

git add .
git commit -m "feat(payment): add credit card checkout tests US-201"
git push origin feature/US-201-payment-tests
# Create PR → review → merge to develop

# Commit message conventions:
# feat(scope): add new tests
# fix(scope): fix flaky test
# refactor: improve page object
# chore: update dependencies
# docs: update README

# Useful commands
git log --oneline --graph    # visual branch tree
git stash                    # save uncommitted changes
git cherry-pick <commit-id>  # apply specific commit to another branch
git bisect                   # find which commit introduced flakiness
```

---

## 42. Handling CAPTCHA in Automation

### Q: How do you handle CAPTCHA in Selenium tests?

```java
// IMPORTANT: Selenium CANNOT bypass real CAPTCHA (by design)

public class CaptchaStrategies {

    // ===== Strategy 1: Disable CAPTCHA in TEST environment =====
    // Ask dev team to add a flag that disables CAPTCHA for QA/test env
    // Most recommended approach for functional testing

    // ===== Strategy 2: Test environment excludes CAPTCHA =====
    // QA environment config: CAPTCHA_ENABLED=false

    // ===== Strategy 3: Third-party CAPTCHA solving service =====
    // Services: 2captcha.com, anti-captcha.com
    // AVOID for regular functional tests — slow and costs money

    public String solve2Captcha(String apiKey, String siteKey, String pageUrl) throws Exception {
        // Submit CAPTCHA to solving service
        String requestId = RestAssured
            .given()
                .param("key", apiKey)
                .param("method", "userrecaptcha")
                .param("googlekey", siteKey)
                .param("pageurl", pageUrl)
            .when()
                .post("http://2captcha.com/in.php")
            .asString().split("\\|")[1];

        // Poll for solution (usually takes 10-30 seconds)
        Thread.sleep(15000);
        String solution = RestAssured
            .given()
                .param("key", apiKey)
                .param("action", "get")
                .param("id", requestId)
            .when()
                .get("http://2captcha.com/res.php")
            .asString().split("\\|")[1];

        // Inject solution via JS
        ((JavascriptExecutor) driver).executeScript(
            "document.getElementById('g-recaptcha-response').innerHTML='" + solution + "';"
        );
        return solution;
    }

    // ===== Strategy 4: Mock CAPTCHA for unit/component tests =====
    // Use Mockito or WireMock to mock CAPTCHA validation endpoint

    // ===== Strategy 5: Cookie-based bypass (if app supports it) =====
    public void bypassWithCookie(WebDriver driver) {
        driver.manage().addCookie(new Cookie("captcha_bypass", "enabled"));
    }

    /* ⭐ Interview Answer:
     * "CAPTCHA is designed to block automation. In our framework, we handle it by:
     *  1. Requesting the dev team to disable CAPTCHA in QA environment
     *  2. Using test accounts whitelisted from CAPTCHA verification
     *  3. For payment flows, testing with sandboxed payment gateways that skip CAPTCHA
     *  We never test CAPTCHA itself — only that the CAPTCHA widget renders correctly."
     */
}
```

---

## 43. Dynamic Tables — Sort, Paginate, Search

### Q: How do you handle complex table operations?

```java
public class DynamicTableHandling {

    WebDriver driver;

    // ===== Read entire table =====
    public List<Map<String, String>> readTable(String tableSelector) {
        List<Map<String, String>> tableData = new ArrayList<>();

        WebElement table = driver.findElement(By.cssSelector(tableSelector));
        List<WebElement> headers = table.findElements(By.cssSelector("thead th"));
        List<WebElement> rows = table.findElements(By.cssSelector("tbody tr"));

        for (WebElement row : rows) {
            Map<String, String> rowData = new LinkedHashMap<>();
            List<WebElement> cells = row.findElements(By.tagName("td"));

            for (int i = 0; i < headers.size() && i < cells.size(); i++) {
                rowData.put(headers.get(i).getText().trim(), cells.get(i).getText().trim());
            }
            tableData.add(rowData);
        }
        return tableData;
    }

    // ===== Find specific row by column value =====
    public WebElement findRowByColumnValue(String columnName, String targetValue) {
        List<WebElement> headers = driver.findElements(By.cssSelector("table thead th"));
        int colIndex = -1;

        for (int i = 0; i < headers.size(); i++) {
            if (headers.get(i).getText().trim().equalsIgnoreCase(columnName)) {
                colIndex = i + 1; // XPath is 1-indexed
                break;
            }
        }

        if (colIndex == -1) throw new RuntimeException("Column not found: " + columnName);

        return driver.findElement(By.xpath(
            "(//table//tbody//tr//td[" + colIndex + "])[text()='" + targetValue + "']/../.."));
    }

    // ===== Click action button in specific row =====
    public void clickActionInRow(String productName, String actionType) {
        driver.findElement(By.xpath(
            "//td[text()='" + productName + "']/following-sibling::td" +
            "//button[contains(@class,'" + actionType + "')]"))
        .click();
    }

    // ===== Verify column is sorted =====
    public boolean isColumnSortedAscending(String columnSelector) {
        List<WebElement> cells = driver.findElements(By.cssSelector(columnSelector));
        List<String> values = cells.stream()
                                   .map(WebElement::getText)
                                   .collect(Collectors.toList());

        List<String> sorted = new ArrayList<>(values);
        Collections.sort(sorted);
        return values.equals(sorted);
    }

    // ===== Handle pagination =====
    public void navigateAllPages(String nextPageSelector, String rowSelector) {
        int pageNum = 1;
        while (true) {
            System.out.println("Page " + pageNum + ":");
            List<WebElement> rows = driver.findElements(By.cssSelector(rowSelector));
            rows.forEach(row -> System.out.println("  " + row.getText()));

            List<WebElement> nextBtn = driver.findElements(By.cssSelector(nextPageSelector));
            if (nextBtn.isEmpty() || !nextBtn.get(0).isEnabled()) {
                break; // no more pages
            }
            nextBtn.get(0).click();
            pageNum++;
        }
    }

    // ===== Search and verify results =====
    public void searchAndVerify(String searchTerm) {
        driver.findElement(By.id("searchInput")).sendKeys(searchTerm);
        driver.findElement(By.id("searchBtn")).click();

        new WebDriverWait(driver, Duration.ofSeconds(10))
            .until(ExpectedConditions.invisibilityOfElementLocated(By.cssSelector(".spinner")));

        List<WebElement> results = driver.findElements(By.cssSelector("table tbody tr"));
        Assert.assertTrue(results.size() > 0, "No search results found");

        results.forEach(row ->
            Assert.assertTrue(row.getText().toLowerCase().contains(searchTerm.toLowerCase()),
                "Row does not contain search term: " + row.getText()));
    }
}
```

---

## 44. Handling Tooltips & Hover Elements

### Q: How do you verify tooltip text in Selenium?

```java
public class TooltipHandling {

    WebDriver driver;
    Actions actions = new Actions(driver);

    // ===== Method 1: HTML title attribute tooltip =====
    public String getHtmlTooltip(WebElement element) {
        return element.getAttribute("title");
    }

    // ===== Method 2: CSS/JS tooltip (appears on hover) =====
    public String getCssTooltip(By triggerLocator, By tooltipLocator) {
        WebElement trigger = driver.findElement(triggerLocator);
        actions.moveToElement(trigger).perform(); // hover

        WebElement tooltip = new WebDriverWait(driver, Duration.ofSeconds(5))
            .until(ExpectedConditions.visibilityOfElementLocated(tooltipLocator));

        return tooltip.getText();
    }

    // ===== Method 3: aria-label attribute =====
    public String getAriaTooltip(WebElement element) {
        return element.getAttribute("aria-label");
    }

    // ===== Method 4: data-tooltip attribute =====
    public String getDataTooltip(WebElement element) {
        return element.getAttribute("data-tooltip");
    }

    // ===== Method 5: Bootstrap/Material UI tooltip =====
    public String getMaterialTooltip() {
        WebElement infoIcon = driver.findElement(By.cssSelector(".info-icon"));

        // Hover to trigger tooltip
        actions.moveToElement(infoIcon).pause(Duration.ofMillis(500)).perform();

        // Wait for tooltip to appear (usually role="tooltip")
        WebElement tooltip = new WebDriverWait(driver, Duration.ofSeconds(5))
            .until(ExpectedConditions.visibilityOfElementLocated(
                By.cssSelector("[role='tooltip']")));

        return tooltip.getText();
    }

    // ⭐ EDGE CASE: Tooltip disappears before you can read it
    // Solution: use JavaScript to freeze tooltip or check attributes instead
}
```

---

## 45. WebDriver Chrome DevTools Protocol (CDP)

### Q: What is CDP in Selenium 4? Give examples.

```java
import org.openqa.selenium.devtools.*;
import org.openqa.selenium.devtools.v120.*;

public class CDPExamples {

    ChromeDriver driver = new ChromeDriver();
    DevTools devTools = driver.getDevTools();

    public void cdpSetup() {
        devTools.createSession();
    }

    // 1. Block network requests (e.g., ads, analytics)
    public void blockNetworkRequests() {
        devTools.send(Network.enable(Optional.empty(), Optional.empty(), Optional.empty()));
        devTools.send(Network.setBlockedURLs(Arrays.asList(
            "*google-analytics*", "*facebook-pixel*", "*ads*"
        )));
        driver.get("https://example.com"); // page loads without blocked resources
    }

    // 2. Emulate network conditions (offline, slow 3G)
    public void emulateSlowNetwork() {
        devTools.send(Network.emulateNetworkConditions(
            false,       // offline
            100,         // latency in ms
            1000,        // download throughput (bytes/s)
            500,         // upload throughput
            Optional.of(Network.ConnectionType.CELLULAR3G)
        ));
    }

    // 3. Override geolocation
    public void setGeolocation(double latitude, double longitude) {
        devTools.send(Emulation.setGeolocationOverride(
            Optional.of(latitude),
            Optional.of(longitude),
            Optional.of(1.0)
        ));
        driver.get("https://google.com/maps");
    }

    // 4. Override device (mobile emulation)
    public void emulateDevice() {
        devTools.send(Emulation.setDeviceMetricsOverride(
            375,    // width
            812,    // height
            2.0,    // device scale factor (retina)
            true,   // mobile
            Optional.empty(), Optional.empty(),
            Optional.empty(), Optional.empty(),
            Optional.empty(), Optional.empty(),
            Optional.empty(), Optional.empty(),
            Optional.empty()
        ));
    }

    // 5. Capture console logs
    public void captureConsoleLogs() {
        devTools.send(Log.enable());
        devTools.addListener(Log.entryAdded(), entry -> {
            System.out.println("Console: [" + entry.getLevel() + "] " + entry.getText());
        });
    }

    // 6. Capture JS exceptions
    public void captureJSExceptions() {
        devTools.send(Runtime.enable());
        devTools.addListener(Runtime.exceptionThrown(), event ->
            System.err.println("JS Error: " + event.getExceptionDetails().getText()));
    }

    // 7. Set HTTP headers on all requests
    public void setExtraHeaders() {
        devTools.send(Network.enable(Optional.empty(), Optional.empty(), Optional.empty()));
        devTools.send(Network.setExtraHTTPHeaders(new Headers(
            Map.of("X-Test-Mode", "true", "Authorization", "Bearer test-token")
        )));
    }
}
```

---

## 46. Custom ExpectedConditions

### Q: How do you create custom ExpectedConditions for complex waits?

```java
import org.openqa.selenium.support.ui.ExpectedCondition;

public class CustomConditions {

    // 1. Wait for element attribute to have specific value
    public static ExpectedCondition<Boolean> attributeToBe(
            By locator, String attribute, String value) {
        return driver -> {
            try {
                String attrValue = driver.findElement(locator).getAttribute(attribute);
                return value.equals(attrValue);
            } catch (NoSuchElementException e) {
                return false;
            }
        };
    }

    // 2. Wait for element count to stabilize (stops changing)
    public static ExpectedCondition<Integer> elementCountToStabilize(
            By locator, int stabilizeForMs) {
        return new ExpectedCondition<Integer>() {
            int lastCount = -1;
            long stableFrom = 0;

            @Override
            public Integer apply(WebDriver driver) {
                int currentCount = driver.findElements(locator).size();
                if (currentCount != lastCount) {
                    lastCount = currentCount;
                    stableFrom = System.currentTimeMillis();
                }
                if (System.currentTimeMillis() - stableFrom >= stabilizeForMs) {
                    return currentCount;
                }
                return null;
            }
        };
    }

    // 3. Wait for ALL of multiple conditions
    public static ExpectedCondition<Boolean> allOf(
            ExpectedCondition<?>... conditions) {
        return driver -> {
            for (ExpectedCondition<?> condition : conditions) {
                if (condition.apply(driver) == null ||
                    Boolean.FALSE.equals(condition.apply(driver))) {
                    return false;
                }
            }
            return true;
        };
    }

    // 4. Wait for text to match regex
    public static ExpectedCondition<Boolean> textMatchesRegex(By locator, String regex) {
        return driver -> {
            try {
                return driver.findElement(locator).getText().matches(regex);
            } catch (NoSuchElementException e) {
                return false;
            }
        };
    }

    // 5. Wait for page to finish all network requests
    public static ExpectedCondition<Boolean> noActiveNetworkRequests() {
        return driver -> {
            JavascriptExecutor js = (JavascriptExecutor) driver;
            Object result = js.executeScript(
                "return window.performance.getEntriesByType('resource')" +
                ".filter(r => !r.responseEnd).length === 0;"
            );
            return Boolean.TRUE.equals(result);
        };
    }

    // ===== Usage =====
    @Test
    public void useCustomConditions() {
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(20));

        // Wait for CSS class to change
        wait.until(CustomConditions.attributeToBe(
            By.id("statusBtn"), "class", "btn btn-success"));

        // Wait for dropdown count to stabilize (after AJAX)
        int count = wait.until(CustomConditions.elementCountToStabilize(
            By.cssSelector("select#city option"), 500));
        System.out.println("City options loaded: " + count);
    }
}
```

---

## 47. Handling Multi-Auth Flows

### Q: How do you handle multi-factor authentication (MFA) in test automation?

```java
public class MFAHandling {

    // ===== Option 1: Disable MFA for test accounts =====
    // Most recommended: test environment has MFA disabled for automation users

    // ===== Option 2: TOTP (Time-based OTP) using TOTP library =====
    // Add dependency: dev.samstevens.totp:totp:1.7.1
    public String generateTOTP(String secretKey) {
        // secretKey from app's MFA setup (stored securely in config)
        TimeBasedOneTimePasswordGenerator totp = new TimeBasedOneTimePasswordGenerator();
        SecretKey key = new SecretKeySpec(
            Base32.decode(secretKey), totp.getAlgorithm());
        return totp.generateOneTimePassword(key, Instant.now());
    }

    public void loginWithMFA() {
        // Step 1: Normal login
        loginPage.enterUsername("mfa_user@test.com");
        loginPage.enterPassword("Test@123");
        loginPage.clickLogin();

        // Step 2: MFA page appears — enter TOTP
        String otpCode = generateTOTP(ConfigReader.get("mfa.secret.key"));
        driver.findElement(By.id("mfaCode")).sendKeys(otpCode);
        driver.findElement(By.id("verifyBtn")).click();
    }

    // ===== Option 3: Mock OTP endpoint in test environment =====
    // Dev team creates a test API: GET /test/otp?userId=x → returns current OTP

    // ===== Option 4: Email OTP via IMAP =====
    public String getOTPFromEmail(String email, String emailPassword) throws Exception {
        Properties props = new Properties();
        props.put("mail.imap.host", "imap.gmail.com");
        props.put("mail.imap.port", "993");
        props.put("mail.imap.ssl.enable", "true");

        Session session = Session.getInstance(props);
        Store store = session.getStore("imap");
        store.connect(email, emailPassword);

        Folder inbox = store.getFolder("INBOX");
        inbox.open(Folder.READ_ONLY);

        // Get latest email
        Message[] messages = inbox.getMessages();
        String content = messages[messages.length - 1].getContent().toString();

        // Extract OTP using regex
        java.util.regex.Matcher matcher =
            Pattern.compile("\\b(\\d{6})\\b").matcher(content);
        return matcher.find() ? matcher.group(1) : null;
    }
}
```

---

## 48. Commonly Misunderstood Concepts

### Q: What are the most common misconceptions in Selenium?

```java
public class CommonMisconceptions {

    // ❌ MYTH 1: Selenium can automate CAPTCHA
    // ✅ TRUTH: Selenium cannot. Use test env without CAPTCHA.

    // ❌ MYTH 2: Selenium tests = automated testing
    // ✅ TRUTH: Selenium = UI automation tool.
    //           Automated testing includes unit, API, performance tests too.

    // ❌ MYTH 3: More @Test annotations = better test coverage
    // ✅ TRUTH: Coverage = what paths/scenarios are tested, not count.

    // ❌ MYTH 4: Thread.sleep() is fine for waiting
    // ✅ TRUTH: Always use Explicit/Fluent waits.

    // ❌ MYTH 5: findElements() throws exception if not found
    // ✅ TRUTH: Returns empty List. Only findElement() throws.

    // ❌ MYTH 6: driver.close() ends session
    // ✅ TRUTH: close() only closes current window. quit() ends session.

    // ❌ MYTH 7: @CacheLookup is always good
    // ✅ TRUTH: Causes StaleElementReferenceException for dynamic elements.

    // ❌ MYTH 8: Implicit + Explicit wait work well together
    // ✅ TRUTH: NEVER mix them. Causes unpredictable wait times.

    // ❌ MYTH 9: XPath is always slower than CSS
    // ✅ TRUTH: Performance difference is negligible in modern browsers.
    //           Choose based on what you need (text search → XPath, others → CSS).

    // ❌ MYTH 10: POM is the same as PageFactory
    // ✅ TRUTH: POM = design pattern. PageFactory = Selenium implementation tool.
    //           You can do POM without PageFactory.

    // ❌ MYTH 11: Selenium 4 supports image comparison natively
    // ✅ TRUTH: Use AShot, Percy.io, or Applitools for visual testing.

    // ❌ MYTH 12: Headless tests are always faster
    // ✅ TRUTH: Headless is faster, but some JS-heavy apps behave differently.

    /* ===== IMPORTANT SELENIUM INTERVIEW ANSWERS =====

    Q: What is Selenium WebDriver?
    A: An open-source, W3C-standard API to automate browser interactions.
       NOT a testing tool — it's an automation tool. Testing logic is handled by
       TestNG/JUnit.

    Q: Can Selenium handle Windows applications?
    A: No. Selenium handles only web browsers. For desktop apps: use Appium/WinAppDriver.

    Q: Is Selenium free?
    A: Yes. Apache 2.0 license.

    Q: What is the default page load timeout?
    A: 300 seconds (5 minutes) by default.

    Q: What happens if implicit wait and explicit wait are both set?
    A: UNPREDICTABLE. They INTERFERE. Explicit can end up waiting
       implicitTimeout + explicitTimeout on failure.

    Q: How many parameters does @DataProvider return?
    A: Object[][] — 2D array. First dimension = test runs, second = parameters.

    Q: What is the difference between get() and navigate().to()?
    A: get() waits for page load. navigate().to() is same but supports
       navigate().back(), navigate().forward(), navigate().refresh().

    Q: What is the difference between isDisplayed(), isEnabled(), isSelected()?
    A: isDisplayed() → visible on page (not hidden, not visibility:hidden)
       isEnabled()   → not disabled (can be interacted with)
       isSelected()  → for checkboxes/radio buttons/options — is checked/selected
    */
}
```

---

## 🎯 Final Checklist — What to Know at 24 LPA

### Technical Skills
- [ ] Complete Selenium 4 API (all methods, all waits)
- [ ] TestNG / Cucumber BDD
- [ ] POM + PageFactory design
- [ ] ThreadLocal for parallel execution
- [ ] ExtentReports + Allure
- [ ] RestAssured for API testing
- [ ] Jenkins CI/CD pipeline
- [ ] Docker + Grid
- [ ] Design patterns (at least Factory, Builder, Singleton, Strategy)
- [ ] Shadow DOM, iFrames, Windows
- [ ] JDBC database validation

### Behavioral / Process Skills
- [ ] Git branching strategy
- [ ] Code review practices
- [ ] Test planning and estimation
- [ ] Defect lifecycle management
- [ ] BDD collaboration (3 Amigos)
- [ ] Sprint planning involvement
- [ ] Shift-left testing mindset

### Must-Know Numbers
- Selenium latest: **4.18+**
- TestNG latest: **7.9+**
- Java LTS: **17 / 21**
- Grid default port: **4444**
- Default implicit wait: **0 seconds**
- Default page load timeout: **300 seconds**

---

> 💡 **Final Pro Tip**: At 24 LPA level, interviewers look for **framework architects**, not just test writers. Be ready to design a framework from scratch on a whiteboard and explain every design decision with justification.
