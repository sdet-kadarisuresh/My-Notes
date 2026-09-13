# Enterprise Test Automation Framework Architecture 🔥

A complete, production-grade, enterprise-level Java + Selenium WebDriver + TestNG + Maven automation framework design guide.

---

## 1. High-Level Architecture & Directory Structure

### Framework Layer Diagram

```
+-----------------------------------------------------------------------------------+
|                                  TEST EXECUTION                                   |
|                        (TestNG / Maven Surefire / Jenkins)                         |
+-----------------------------------------------------------------------------------+
                                          |
                                          v
+-----------------------------------------------------------------------------------+
|                                     TEST LAYER                                    |
|   - Test Classes (LoginTest, CheckoutTest)                                        |
|   - BaseTest (@BeforeMethod, @AfterMethod)                                        |
|   - DataProviders (@DataProvider -> ExcelUtils / JSONReader)                     |
+-----------------------------------------------------------------------------------+
        |                                   |                                   |
        v                                   v                                   v
+-----------------------+   +-------------------------------+   +-------------------+
|      PAGE LAYER       |   |       CORE FRAMEWORK LAYER    |   |  UTILITIES LAYER  |
| - BasePage            |   | - DriverFactory (ThreadLocal) |   | - WaitUtils       |
| - Page Object Classes |   | - ConfigReader (Properties)   |   | - ScreenshotUtils |
|   (LoginPage, etc.)   |   | - FrameworkConstants          |   | - ExcelUtils      |
+-----------------------+   | - Environment Handling        |   | - JSONReader      |
                            +-------------------------------+   +-------------------+
                                          |                                   |
                                          v                                   v
+-----------------------------------------------------------------------------------+
|                             LISTENERS & REPORTING LAYER                           |
|   - TestListener (ITestListener) -> Captures Screenshots on Failure              |
|   - ExtentReportManager (ThreadLocal ExtentTest)                                  |
|   - Log4j2 Logging Engine                                                         |
+-----------------------------------------------------------------------------------+
```

---

### Folder Structure (Standard Maven Project)

```
SeleniumFramework/
├── src/
│   ├── main/
│   │   └── java/
│   │       └── com/automation/
│   │           ├── config/
│   │           │   └── ConfigReader.java
│   │           ├── constants/
│   │           │   └── FrameworkConstants.java
│   │           ├── driver/
│   │           │   ├── DriverFactory.java
│   │           │   └── DriverManager.java
│   │           ├── enums/
│   │           │   ├── ConfigProperties.java
│   │           │   └── EnvironmentType.java
│   │           ├── listeners/
│   │           │   ├── AnnotationTransformer.java
│   │           │   ├── RetryAnalyzer.java
│   │           │   └── TestListener.java
│   │           ├── pages/
│   │           │   ├── BasePage.java
│   │           │   ├── HomePage.java
│   │           │   └── LoginPage.java
│   │           ├── reports/
│   │           │   ├── ExtentLogger.java
│   │           │   └── ExtentReportManager.java
│   │           └── utils/
│   │               ├── ExcelUtils.java
│   │               ├── JSONReader.java
│   │               ├── LogUtils.java
│   │               ├── ScreenshotUtils.java
│   │               └── WaitUtils.java
│   └── test/
│       ├── java/
│       │   └── com/automation/tests/
│       │       ├── BaseTest.java
│       │       ├── HomeTest.java
│       │       └── LoginTest.java
│       └── resources/
│           ├── config/
│           │   ├── config.properties
│           │   ├── dev-config.properties
│           │   └── qa-config.properties
│           ├── testdata/
│           │   ├── testdata.xlsx
│           │   └── user_data.json
│           ├── log4j2.xml
│           └── testng.xml
├── pom.xml
└── README.md
```

---

## 2. Component-by-Component Implementation

---

### 1. Framework Constants (`FrameworkConstants.java`)

Centralized storage of immutable application path constants and configuration timeouts. Avoids hardcoded strings throughout the project.

```java
package com.automation.constants;

public final class FrameworkConstants {

    private FrameworkConstants() {
        // Private constructor to prevent instantiation
    }

    private static final String USER_DIR = System.getProperty("user.dir");
    private static final String RESOURCES_PATH = USER_DIR + "/src/test/resources";
    
    private static final String CONFIG_FILE_PATH = RESOURCES_PATH + "/config/config.properties";
    private static final String EXCEL_PATH = RESOURCES_PATH + "/testdata/testdata.xlsx";
    private static final String JSON_DATA_PATH = RESOURCES_PATH + "/testdata/user_data.json";
    private static final String EXTENT_REPORT_PATH = USER_DIR + "/target/extent-reports/";

    private static final int EXPLICIT_WAIT = 15;
    private static final int IMPLICIT_WAIT = 10;

    public static String getConfigFilePath() { return CONFIG_FILE_PATH; }
    public static String getExcelPath() { return EXCEL_PATH; }
    public static String getJsonDataPath() { return JSON_DATA_PATH; }
    public static String getExtentReportPath() { return EXTENT_REPORT_PATH; }
    public static int getExplicitWait() { return EXPLICIT_WAIT; }
    public static int getImplicitWait() { return IMPLICIT_WAIT; }
}
```

---

### 2. Configuration Reader & Environment Handling (`ConfigReader.java`)

Loads configuration settings dynamically based on the target environment (QA, DEV, STAGING) passed via command line flags or default properties file.

#### `config.properties`
```properties
browser=chrome
environment=qa
headless=false
timeout=10
```

#### `qa-config.properties`
```properties
url=https://qa.application.com
username=qa_user@test.com
password=QAPassword123!
```

#### `ConfigReader.java`
```java
package com.automation.config;

import com.automation.constants.FrameworkConstants;
import java.io.FileInputStream;
import java.io.IOException;
import java.util.Properties;

public final class ConfigReader {

    private static final Properties properties = new Properties();

    private ConfigReader() {}

    static {
        try {
            // Load base configuration
            FileInputStream baseFile = new FileInputStream(FrameworkConstants.getConfigFilePath());
            properties.load(baseFile);
            baseFile.close();

            // Override environment dynamically via CLI flag: -Denvironment=dev
            String env = System.getProperty("environment", properties.getProperty("environment", "qa"));
            String envFilePath = System.getProperty("user.dir") + "/src/test/resources/config/" + env.toLowerCase() + "-config.properties";

            FileInputStream envFile = new FileInputStream(envFilePath);
            properties.load(envFile); // Overrides/adds env-specific keys
            envFile.close();

        } catch (IOException e) {
            throw new RuntimeException("Failed to load configuration properties file!", e);
        }
    }

    public static String get(String key) {
        // Priority: System Property > Properties File
        String systemProperty = System.getProperty(key);
        if (systemProperty != null && !systemProperty.trim().isEmpty()) {
            return systemProperty;
        }
        String value = properties.getProperty(key);
        if (value == null) {
            throw new RuntimeException("Property key '" + key + "' not found in configuration files.");
        }
        return value.trim();
    }
}
```

---

### 3. Driver Thread Safety & Manager (`DriverManager.java`)

ThreadLocal storage wrapper ensuring thread isolation during parallel test execution.

```java
package com.automation.driver;

import org.openqa.selenium.WebDriver;

public final class DriverManager {

    private DriverManager() {}

    private static final ThreadLocal<WebDriver> dr = new ThreadLocal<>();

    public static WebDriver getDriver() {
        return dr.get();
    }

    public static void setDriver(WebDriver driverRef) {
        dr.set(driverRef);
    }

    public static void unload() {
        dr.remove();
    }
}
```

---

### 4. Driver Factory (`DriverFactory.java`)

Implements the **Factory Pattern** to initialize `Chrome`, `Firefox`, `Edge`, or `RemoteWebDriver` based on configuration parameters.

```java
package com.automation.driver;

import com.automation.config.ConfigReader;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.edge.EdgeDriver;
import org.openqa.selenium.firefox.FirefoxDriver;
import java.time.Duration;

public final class DriverFactory {

    private DriverFactory() {}

    public static void initDriver() {
        if (DriverManager.getDriver() == null) {
            String browser = ConfigReader.get("browser").toLowerCase();
            boolean isHeadless = Boolean.parseBoolean(ConfigReader.get("headless"));
            WebDriver driver;

            switch (browser) {
                case "chrome":
                    ChromeOptions options = new ChromeOptions();
                    if (isHeadless) {
                        options.addArguments("--headless=new");
                    }
                    options.addArguments("--start-maximized");
                    options.addArguments("--disable-notifications");
                    driver = new ChromeDriver(options);
                    break;

                case "firefox":
                    driver = new FirefoxDriver();
                    break;

                case "edge":
                    driver = new EdgeDriver();
                    break;

                default:
                    throw new IllegalArgumentException("Invalid browser specified: " + browser);
            }

            driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
            driver.get(ConfigReader.get("url"));
            DriverManager.setDriver(driver);
        }
    }

    public static void quitDriver() {
        if (DriverManager.getDriver() != null) {
            DriverManager.getDriver().quit();
            DriverManager.unload();
        }
    }
}
```

---

### 5. Synchronization Utilities (`WaitUtils.java`)

Wraps Explicit Waits (`WebDriverWait`) and `FluentWait` to eliminate flaky assertions.

```java
package com.automation.utils;

import com.automation.constants.FrameworkConstants;
import com.automation.driver.DriverManager;
import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import java.time.Duration;

public final class WaitUtils {

    private WaitUtils() {}

    private static WebDriverWait getWait() {
        return new WebDriverWait(DriverManager.getDriver(), Duration.ofSeconds(FrameworkConstants.getExplicitWait()));
    }

    public static WebElement waitUntilVisible(By locator) {
        return getWait().until(ExpectedConditions.visibilityOfElementLocated(locator));
    }

    public static WebElement waitUntilClickable(By locator) {
        return getWait().until(ExpectedConditions.elementToBeClickable(locator));
    }

    public static boolean waitUntilDisappear(By locator) {
        return getWait().until(ExpectedConditions.invisibilityOfElementLocated(locator));
    }
}
```

---

### 6. Screenshot Utilities (`ScreenshotUtils.java`)

Captures base64 strings and physical PNG images for failure debugging in Extent Reports.

```java
package com.automation.utils;

import com.automation.driver.DriverManager;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;

public final class ScreenshotUtils {

    private ScreenshotUtils() {}

    public static String getBase64Image() {
        return ((TakesScreenshot) DriverManager.getDriver()).getScreenshotAs(OutputType.BASE64);
    }
}
```

---

### 7. Excel Data Reader (`ExcelUtils.java`)

Uses Apache POI to parse test matrices into TestNG `@DataProvider` format (`Object[][]`).

```java
package com.automation.utils;

import com.automation.constants.FrameworkConstants;
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import java.io.FileInputStream;
import java.io.IOException;

public final class ExcelUtils {

    private ExcelUtils() {}

    public static Object[][] getTestData(String sheetName) {
        Object[][] data = null;
        try (FileInputStream fis = new FileInputStream(FrameworkConstants.getExcelPath());
             Workbook workbook = new XSSFWorkbook(fis)) {

            Sheet sheet = workbook.getSheet(sheetName);
            int rowCount = sheet.getLastRowNum();
            int colCount = sheet.getRow(0).getLastCellNum();

            data = new Object[rowCount][colCount];

            DataFormatter formatter = new DataFormatter();

            for (int i = 0; i < rowCount; i++) {
                Row row = sheet.getRow(i + 1); // Skip Header
                for (int j = 0; j < colCount; j++) {
                    Cell cell = row.getCell(j);
                    data[i][j] = formatter.formatCellValue(cell);
                }
            }
        } catch (IOException e) {
            throw new RuntimeException("Could not read Excel file!", e);
        }
        return data;
    }
}
```

---

### 8. JSON Reader (`JSONReader.java`)

Parses JSON data files using Jackson ObjectMapper into POJO objects or Map structures.

```java
package com.automation.utils;

import com.fasterxml.jackson.databind.ObjectMapper;
import java.io.File;
import java.io.IOException;
import java.util.Map;

public final class JSONReader {

    private JSONReader() {}

    public static Map<String, Object> getJsonDataAsMap(String filePath) {
        try {
            ObjectMapper mapper = new ObjectMapper();
            return mapper.readValue(new File(filePath), Map.class);
        } catch (IOException e) {
            throw new RuntimeException("Failed to read JSON file at: " + filePath, e);
        }
    }
}
```

---

### 9. Logging Engine (`LogUtils.java` & `log4j2.xml`)

Configures Log4j2 logger for console and rolling file logging.

#### `log4j2.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} [%t] %-5level %logger{36} - %msg%n"/>
        </Console>
        <File name="FileAppender" fileName="target/logs/automation.log" append="false">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} [%t] %-5level %logger{36} - %msg%n"/>
        </File>
    </Appenders>
    <Loggers>
        <Root level="info">
            <AppenderRef ref="Console"/>
            <AppenderRef ref="FileAppender"/>
        </Root>
    </Loggers>
</Configuration>
```

#### `LogUtils.java`
```java
package com.automation.utils;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

public final class LogUtils {

    private LogUtils() {}

    private static final Logger logger = LogManager.getLogger(LogUtils.class);

    public static void info(String message) {
        logger.info(message);
    }

    public static void error(String message, Throwable t) {
        logger.error(message, t);
    }

    public static void warn(String message) {
        logger.warn(message);
    }
}
```

---

### 10. Reporting Layer (`ExtentReportManager.java` & `ExtentLogger.java`)

Thread-safe ExtentReports 5.x integration with automatic failure screenshot capture.

```java
package com.automation.reports;

import com.aventstack.extentreports.ExtentReports;
import com.aventstack.extentreports.ExtentTest;
import com.aventstack.extentreports.reporter.ExtentSparkReporter;
import com.aventstack.extentreports.reporter.configuration.Theme;
import com.automation.constants.FrameworkConstants;

public final class ExtentReportManager {

    private ExtentReportManager() {}

    private static ExtentReports extent;
    private static final ThreadLocal<ExtentTest> extentTest = new ThreadLocal<>();

    public static void initReports() {
        if (extent == null) {
            extent = new ExtentReports();
            ExtentSparkReporter spark = new ExtentSparkReporter(FrameworkConstants.getExtentReportPath() + "TestReport.html");
            spark.config().setTheme(Theme.DARK);
            spark.config().setDocumentTitle("Automation Execution Report");
            spark.config().setReportName("Regression Suite Results");
            extent.attachReporter(spark);
        }
    }

    public static void createTest(String testName) {
        ExtentTest test = extent.createTest(testName);
        extentTest.set(test);
    }

    public static ExtentTest getTest() {
        return extentTest.get();
    }

    public static void unload() {
        extentTest.remove();
    }

    public static void flushReports() {
        if (extent != null) {
            extent.flush();
        }
    }
}
```

---

### 11. TestNG Listeners (`TestListener.java`)

Intercepts test start, pass, failure, and skip events. On failure, captures base64 screenshot and attaches it directly to the HTML report.

```java
package com.automation.listeners;

import com.aventstack.extentreports.MediaEntityBuilder;
import com.automation.reports.ExtentReportManager;
import com.automation.utils.LogUtils;
import com.automation.utils.ScreenshotUtils;
import org.testng.ITestContext;
import org.testng.ITestListener;
import org.testng.ITestResult;

public class TestListener implements ITestListener {

    @Override
    public void onStart(ITestContext context) {
        ExtentReportManager.initReports();
    }

    @Override
    public void onFinish(ITestContext context) {
        ExtentReportManager.flushReports();
    }

    @Override
    public void onTestStart(ITestResult result) {
        ExtentReportManager.createTest(result.getMethod().getMethodName());
        LogUtils.info("Starting test: " + result.getMethod().getMethodName());
    }

    @Override
    public void onTestSuccess(ITestResult result) {
        ExtentReportManager.getTest().pass("Test Passed Successfully.");
        ExtentReportManager.unload();
    }

    @Override
    public void onTestFailure(ITestResult result) {
        LogUtils.error("Test Failed: " + result.getMethod().getMethodName(), result.getThrowable());
        String base64Image = ScreenshotUtils.getBase64Image();
        ExtentReportManager.getTest().fail(result.getThrowable(),
                MediaEntityBuilder.createScreenCaptureFromBase64String(base64Image).build());
        ExtentReportManager.unload();
    }

    @Override
    public void onTestSkipped(ITestResult result) {
        ExtentReportManager.getTest().skip("Test Skipped.");
        ExtentReportManager.unload();
    }
}
```

---

### 12. Page Object Base Layer (`BasePage.java`)

Contains common encapsulated Selenium action wrappers with built-in explicit waits.

```java
package com.automation.pages;

import com.automation.utils.LogUtils;
import com.automation.utils.WaitUtils;
import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;

public class BasePage {

    protected void click(By locator, String elementName) {
        WebElement element = WaitUtils.waitUntilClickable(locator);
        element.click();
        LogUtils.info("Clicked on: " + elementName);
    }

    protected void type(By locator, String value, String elementName) {
        WebElement element = WaitUtils.waitUntilVisible(locator);
        element.clear();
        element.sendKeys(value);
        LogUtils.info("Typed '" + value + "' in: " + elementName);
    }

    protected String getText(By locator) {
        return WaitUtils.waitUntilVisible(locator).getText();
    }
}
```

---

### 13. Page Object Classes (`LoginPage.java`)

Implements actual UI locators and user workflows.

```java
package com.automation.pages;

import org.openqa.selenium.By;

public class LoginPage extends BasePage {

    private final By txtUsername = By.id("user-name");
    private final By txtPassword = By.id("password");
    private final By btnLogin = By.id("login-button");

    public LoginPage enterUsername(String username) {
        type(txtUsername, username, "Username Field");
        return this; // Method chaining
    }

    public LoginPage enterPassword(String password) {
        type(txtPassword, password, "Password Field");
        return this;
    }

    public HomePage clickLogin() {
        click(btnLogin, "Login Button");
        return new HomePage();
    }
}
```

---

### 14. Test Class Base Layer (`BaseTest.java`)

Manages setup and teardown hooks (`@BeforeMethod`, `@AfterMethod`).

```java
package com.automation.tests;

import com.automation.driver.DriverFactory;
import com.automation.listeners.TestListener;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Listeners;

@Listeners(TestListener.class)
public class BaseTest {

    @BeforeMethod
    public void setUp() {
        DriverFactory.initDriver();
    }

    @AfterMethod
    public void tearDown() {
        DriverFactory.quitDriver();
    }
}
```

---

### 15. Concrete Test Classes (`LoginTest.java`)

Contains high-level business workflow assertions, free from low-level driver calls.

```java
package com.automation.tests;

import com.automation.pages.LoginPage;
import com.automation.utils.ExcelUtils;
import org.testng.Assert;
import org.testng.annotations.DataProvider;
import org.testng.annotations.Test;

public class LoginTest extends BaseTest {

    @DataProvider(name = "loginData")
    public Object[][] getLoginData() {
        return ExcelUtils.getTestData("LoginSheet");
    }

    @Test(dataProvider = "loginData", groups = {"smoke", "regression"})
    public void verifyValidLogin(String username, String password) {
        LoginPage loginPage = new LoginPage();
        
        boolean isDashboardDisplayed = loginPage.enterUsername(username)
                                               .enterPassword(password)
                                               .clickLogin()
                                               .isDashboardHeaderDisplayed();

        Assert.assertTrue(isDashboardDisplayed, "Login failed for user: " + username);
    }
}
```

---

## 3. Summary Matrix of Design Patterns Used

| Design Pattern | Where Used in Framework | Benefit |
| :--- | :--- | :--- |
| **Singleton Pattern** | `ConfigReader`, `ExtentReportManager` | Prevents redundant file reading and memory overhead. |
| **Factory Pattern** | `DriverFactory` | Encapsulates browser driver instantiation logic. |
| **ThreadLocal Pattern** | `DriverManager`, `ExtentReportManager` | Guarantees thread safety and isolation during parallel execution. |
| **Page Object Model** | `LoginPage`, `HomePage` | Separates UI locators and actions from test logic. |
| **Fluent / Chain Pattern** | Page Methods (`enterUsername().enterPassword().clickLogin()`) | Improves readability of test methods. |
| **Observer Pattern** | `TestListener` (TestNG `ITestListener`) | Automatically listens for test failures and attaches screenshots. |

---
