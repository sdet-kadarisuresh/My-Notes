# PART 23: REAL-TIME FRAMEWORK (Complete Build)

**Q: Can you walk me through the architecture and overview of the automation framework you built in your project?** 🔥
**A:** In my project, we built a robust, scalable test automation framework from scratch. The way I handle this is by clearly separating concerns. We use Java with Selenium WebDriver, driven by TestNG for execution and Maven for dependency management. The architecture is primarily based on the Page Object Model (POM). 

What we typically do is divide the framework into distinct layers. At the base layer, we have our `DriverFactory` managing thread-safe WebDriver instances using `ThreadLocal`, and a `ConfigReader` to pull settings from a `config.properties` file. Above that is the Page Layer, where we have a `BasePage` containing all our reusable Selenium wrapper methods (like `click()`, `type()`, `waitForElement()`). Every specific page class, like `LoginPage`, extends this `BasePage`. 

Then we have the Test Layer. Our tests extend a `BaseTest` class which handles the setup (`@BeforeMethod`) and teardown (`@AfterMethod`), including capturing screenshots on failure. Finally, for reporting, we integrated Extent Reports via a TestNG `ITestListener`. Log4j handles our execution logs. This layered approach means if the UI changes, we only update the Page Objects; if we need to change how we launch a browser, we only touch the DriverFactory. It's built for maintainability.

### 2. Complete pom.xml
**Q: How do you manage dependencies and what does your pom.xml look like?**
**A:** We use Maven. From my experience, keeping versions centralized in the `<properties>` block is best practice. Here's exactly what our `pom.xml` looks like. We include `selenium-java`, `testng`, `extentreports`, `poi-ooxml` for Excel data reading, and `log4j` for logging. We also use the `maven-surefire-plugin` to trigger our `testng.xml` from the command line, which is crucial for CI/CD integration.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.automation</groupId>
  <artifactId>SeleniumFramework</artifactId>
  <version>1.0-SNAPSHOT</version>

  <properties>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
    <selenium.version>4.18.1</selenium.version>
    <testng.version>7.9.0</testng.version>
    <extent.version>5.1.1</extent.version>
    <poi.version>5.2.3</poi.version>
    <log4j.version>2.20.0</log4j.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.seleniumhq.selenium</groupId>
      <artifactId>selenium-java</artifactId>
      <version>${selenium.version}</version>
    </dependency>
    <dependency>
      <groupId>org.testng</groupId>
      <artifactId>testng</artifactId>
      <version>${testng.version}</version>
    </dependency>
    <dependency>
      <groupId>com.aventstack</groupId>
      <artifactId>extentreports</artifactId>
      <version>${extent.version}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.poi</groupId>
      <artifactId>poi-ooxml</artifactId>
      <version>${poi.version}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.logging.log4j</groupId>
      <artifactId>log4j-core</artifactId>
      <version>${log4j.version}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.logging.log4j</groupId>
      <artifactId>log4j-api</artifactId>
      <version>${log4j.version}</version>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>3.1.2</version>
        <configuration>
          <suiteXmlFiles>
            <suiteXmlFile>src/test/resources/testng.xml</suiteXmlFile>
          </suiteXmlFiles>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```

### 3. config.properties
**Q: How do you externalize configuration data?**
**A:** In my project, we never hardcode environment URLs or browser names. We keep them in a `config.properties` file located in `src/test/resources`. This makes it incredibly easy to switch environments or browsers without touching the code.

```properties
browser=chrome
url=https://opensource-demo.orangehrmlive.com/
implicitWait=10
explicitWait=15
headless=false
environment=qa
```

### 4. ConfigReader.java
**Q: How do you read data from the properties file?**
**A:** I implemented a Singleton `ConfigReader` class. What we typically do is load the properties file once into memory during the initial setup. The Singleton pattern ensures we don't open multiple file streams, which is an important performance consideration when running hundreds of tests.

```java
package com.framework.utils;

import java.io.FileInputStream;
import java.util.Properties;

public class ConfigReader {
    private static Properties properties;

    public static void initialize() {
        if (properties == null) {
            try {
                FileInputStream file = new FileInputStream("src/test/resources/config.properties");
                properties = new Properties();
                properties.load(file);
            } catch (Exception e) {
                e.printStackTrace();
                throw new RuntimeException("Could not load config.properties");
            }
        }
    }

    public static String getProperty(String key) {
        if (properties == null) initialize();
        return properties.getProperty(key);
    }
}
```

### 5. DriverFactory.java 🔥
**Q: How do you handle WebDriver initialization, especially for parallel execution?** 🔥
**A:** This is a critical part of the framework. In my project, we use a `ThreadLocal` variable to store the WebDriver instance. `ThreadLocal` ensures that every thread in TestNG gets its own isolated WebDriver instance, making parallel execution completely thread-safe. We have an `initDriver()` method that reads the browser type from config and spins up the respective driver.

```java
package com.framework.driver;

import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.edge.EdgeDriver;
import org.openqa.selenium.firefox.FirefoxDriver;

public class DriverFactory {
    private static ThreadLocal<WebDriver> driverTL = new ThreadLocal<>();

    public static void initDriver(String browser) {
        if (driverTL.get() == null) {
            WebDriver driver;
            boolean headless = Boolean.parseBoolean(com.framework.utils.ConfigReader.getProperty("headless"));
            
            switch (browser.toLowerCase()) {
                case "chrome":
                    ChromeOptions options = new ChromeOptions();
                    if (headless) options.addArguments("--headless");
                    driver = new ChromeDriver(options);
                    break;
                case "firefox":
                    driver = new FirefoxDriver();
                    break;
                case "edge":
                    driver = new EdgeDriver();
                    break;
                default:
                    throw new IllegalArgumentException("Invalid browser: " + browser);
            }
            driver.manage().window().maximize();
            driverTL.set(driver);
        }
    }

    public static WebDriver getDriver() {
        return driverTL.get();
    }

    public static void quitDriver() {
        if (driverTL.get() != null) {
            driverTL.get().quit();
            driverTL.remove();
        }
    }
}
```

### 6. BasePage.java
**Q: How do you avoid writing WebDriver commands repeatedly in every Page Object?**
**A:** The way I handle this is by creating a `BasePage` class that all other Page Objects extend. This class encapsulates all native Selenium commands inside explicit waits. So instead of doing `driver.findElement(By).click()`, my page classes call a generic `click()` method that waits for the element to be clickable first. This drastically reduces `ElementNotInteractableException` flakiness in our pipelines.

```java
package com.framework.pages;

import com.framework.driver.DriverFactory;
import org.openqa.selenium.By;
import org.openqa.selenium.JavascriptExecutor;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.Select;
import org.openqa.selenium.support.ui.WebDriverWait;
import java.time.Duration;

public class BasePage {
    protected WebDriver driver;
    protected WebDriverWait wait;

    public BasePage() {
        this.driver = DriverFactory.getDriver();
        int explicitWaitTime = Integer.parseInt(com.framework.utils.ConfigReader.getProperty("explicitWait"));
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(explicitWaitTime));
    }

    protected void click(By locator) {
        waitForElementClickable(locator).click();
    }

    protected void type(By locator, String text) {
        WebElement element = waitForElementVisible(locator);
        element.clear();
        element.sendKeys(text);
    }

    protected String getText(By locator) {
        return waitForElementVisible(locator).getText();
    }

    protected boolean isDisplayed(By locator) {
        try {
            return waitForElementVisible(locator).isDisplayed();
        } catch (Exception e) {
            return false;
        }
    }

    protected WebElement waitForElementVisible(By locator) {
        return wait.until(ExpectedConditions.visibilityOfElementLocated(locator));
    }

    protected WebElement waitForElementClickable(By locator) {
        return wait.until(ExpectedConditions.elementToBeClickable(locator));
    }
    
    protected void selectDropdownByText(By locator, String text) {
        Select select = new Select(waitForElementVisible(locator));
        select.selectByVisibleText(text);
    }

    protected void scrollToElement(By locator) {
        WebElement element = driver.findElement(locator);
        ((JavascriptExecutor) driver).executeScript("arguments[0].scrollIntoView(true);", element);
    }
}
```

### 7. BaseTest.java
**Q: How do you handle setup and teardown for your test scripts?**
**A:** From my experience, you should never put setup code inside individual test classes. I use a `BaseTest` class where I utilize TestNG's `@BeforeMethod` to initialize the driver and navigate to the application URL, and `@AfterMethod` to tear down the driver. We also pass the browser parameter from the TestNG XML, allowing for cross-browser execution.

```java
package com.framework.tests;

import com.framework.driver.DriverFactory;
import com.framework.utils.ConfigReader;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Optional;
import org.testng.annotations.Parameters;

public class BaseTest {

    @Parameters("browser")
    @BeforeMethod(alwaysRun = true)
    public void setup(@Optional("chrome") String browser) {
        ConfigReader.initialize();
        // Fallback to properties if parameter is not supplied via testng.xml
        String browserName = System.getProperty("browser") != null ? System.getProperty("browser") : browser;
        
        DriverFactory.initDriver(browserName);
        DriverFactory.getDriver().get(ConfigReader.getProperty("url"));
    }

    @AfterMethod(alwaysRun = true)
    public void tearDown() {
        DriverFactory.quitDriver();
    }
}
```

### 8. Page Classes (Complete)
**Q: Can you show me an example of how you structure a Page Object class?**
**A:** Absolutely. In my project, we follow strict encapsulation. Locators are kept `private` as `By` objects, and we expose public action methods that tests can call. We don't write assertions in page classes; page classes only return data or perform actions. Here's a real example of a `LoginPage`.

```java
package com.framework.pages;

import org.openqa.selenium.By;

public class LoginPage extends BasePage {
    
    // Locators
    private By usernameField = By.name("username");
    private By passwordField = By.name("password");
    private By loginBtn = By.xpath("//button[@type='submit']");
    private By errorMessage = By.cssSelector(".oxd-alert-content-text");

    // Actions
    public void enterUsername(String username) {
        type(usernameField, username);
    }

    public void enterPassword(String password) {
        type(passwordField, password);
    }

    public DashboardPage clickLogin() {
        click(loginBtn);
        return new DashboardPage();
    }
    
    public DashboardPage loginAs(String user, String pass) {
        enterUsername(user);
        enterPassword(pass);
        return clickLogin();
    }

    public String getErrorMessage() {
        return getText(errorMessage);
    }
}
```

```java
package com.framework.pages;

import org.openqa.selenium.By;

public class DashboardPage extends BasePage {

    private By dashboardHeader = By.xpath("//h6[text()='Dashboard']");
    private By userDropdown = By.cssSelector(".oxd-userdropdown-name");

    public boolean isDashboardDisplayed() {
        return isDisplayed(dashboardHeader);
    }

    public String getLoggedUserName() {
        return getText(userDropdown);
    }
}
```

### 9. Test Classes (Complete)
**Q: How do you write the actual test methods using these Page Objects?**
**A:** Our tests are extremely clean and readable because all the heavy lifting is done by the page classes. What we typically do is instantiate the page object, perform actions, and then use TestNG's `Assert` class to validate the state. 

```java
package com.framework.tests;

import com.framework.pages.DashboardPage;
import com.framework.pages.LoginPage;
import org.testng.Assert;
import org.testng.annotations.Test;

public class LoginTest extends BaseTest {

    @Test(groups = {"smoke", "regression"})
    public void validLoginTest() {
        LoginPage loginPage = new LoginPage();
        DashboardPage dashboardPage = loginPage.loginAs("Admin", "admin123");
        
        Assert.assertTrue(dashboardPage.isDashboardDisplayed(), "Dashboard was not displayed after login!");
    }

    @Test(groups = {"regression"})
    public void invalidLoginTest() {
        LoginPage loginPage = new LoginPage();
        loginPage.loginAs("Invalid", "User");
        
        String error = loginPage.getErrorMessage();
        Assert.assertEquals(error, "Invalid credentials", "Error message mismatch!");
    }
}
```

### 10-14. Reporting, Listeners, and Utilities
**Q: How do you handle Extent Reports and Screenshots on failure?** 🔥
**A:** In my project, we use an `ITestListener` to automatically capture screenshots and log failures into Extent Reports. We don't clutter our `@AfterMethod` with reporting logic. Instead, when `onTestFailure` is triggered in the listener, it calls our `ScreenshotUtils` to take a base64 screenshot and attaches it directly to the Extent Report log.

```java
// ExtentReportManager.java
package com.framework.utils;

import com.aventstack.extentreports.ExtentReports;
import com.aventstack.extentreports.reporter.ExtentSparkReporter;
import com.aventstack.extentreports.reporter.configuration.Theme;

public class ExtentReportManager {
    private static ExtentReports extent;

    public static ExtentReports getReporter() {
        if (extent == null) {
            ExtentSparkReporter spark = new ExtentSparkReporter("target/ExtentReports/Spark.html");
            spark.config().setDocumentTitle("Automation Report");
            spark.config().setReportName("Functional Test Execution");
            spark.config().setTheme(Theme.DARK);

            extent = new ExtentReports();
            extent.attachReporter(spark);
            extent.setSystemInfo("Environment", ConfigReader.getProperty("environment"));
            extent.setSystemInfo("OS", System.getProperty("os.name"));
        }
        return extent;
    }
}
```

```java
// TestNGListener.java
package com.framework.listeners;

import com.aventstack.extentreports.ExtentReports;
import com.aventstack.extentreports.ExtentTest;
import com.aventstack.extentreports.Status;
import com.framework.driver.DriverFactory;
import com.framework.utils.ExtentReportManager;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.testng.ITestContext;
import org.testng.ITestListener;
import org.testng.ITestResult;

public class TestNGListener implements ITestListener {
    private ExtentReports extent = ExtentReportManager.getReporter();
    private static ThreadLocal<ExtentTest> testTL = new ThreadLocal<>();

    @Override
    public void onTestStart(ITestResult result) {
        ExtentTest test = extent.createTest(result.getMethod().getMethodName());
        testTL.set(test);
    }

    @Override
    public void onTestSuccess(ITestResult result) {
        testTL.get().log(Status.PASS, "Test Passed");
    }

    @Override
    public void onTestFailure(ITestResult result) {
        testTL.get().log(Status.FAIL, result.getThrowable());
        String base64Screenshot = ((TakesScreenshot) DriverFactory.getDriver()).getScreenshotAs(OutputType.BASE64);
        testTL.get().addScreenCaptureFromBase64String(base64Screenshot, "Failure Screenshot");
    }

    @Override
    public void onFinish(ITestContext context) {
        extent.flush();
    }
}
```

### 16. testng.xml
**Q: How do you trigger parallel execution and grouping?**
**A:** We manage this entirely through our `testng.xml`. By setting `parallel="tests"` or `parallel="methods"`, we tell TestNG to spin up different threads. Since we use `ThreadLocal` for WebDriver, this works seamlessly.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd">
<suite name="Regression Suite" parallel="methods" thread-count="4">
    <listeners>
        <listener class-name="com.framework.listeners.TestNGListener"/>
    </listeners>

    <test name="Chrome Tests">
        <parameter name="browser" value="chrome"/>
        <groups>
            <run>
                <include name="regression"/>
            </run>
        </groups>
        <classes>
            <class name="com.framework.tests.LoginTest"/>
        </classes>
    </test>
</suite>
```

---

# PART 24: REAL-TIME SCENARIOS 🔥

### 20. E-Commerce: Search + Add to Cart + Checkout
**Q: Can you explain how you would automate an end-to-end flow for an e-commerce checkout?**
**A:** In my project, we automated complex flows like this by chaining page object methods. The main challenge here is synchronization—waiting for the cart overlay to disappear before proceeding to checkout. What we typically do is write an E2E test that traverses multiple pages.

First, we search for the product on the HomePage. The search returns a SearchResultsPage. From there, we click "Add to Cart" and wait for the success notification. Then we navigate to the CartPage and finally the CheckoutPage. 

```java
@Test
public void checkoutE2ETest() {
    HomePage homePage = new HomePage();
    
    // Search and add
    SearchResultsPage results = homePage.searchProduct("MacBook Pro");
    results.clickAddToCart("MacBook Pro");
    
    // Verify toast message
    Assert.assertTrue(results.isSuccessMessageDisplayed());
    
    // Go to cart and checkout
    CartPage cart = results.goToCart();
    Assert.assertTrue(cart.verifyProductInCart("MacBook Pro"));
    
    CheckoutPage checkout = cart.proceedToCheckout();
    checkout.enterBillingDetails(billingInfo);
    checkout.confirmOrder();
    
    Assert.assertTrue(checkout.isOrderSuccessful());
}
```

### 23. Multiple Windows Handling 🔥
**Q: How do you handle switching between multiple browser windows or tabs?** 🔥
**A:** This is a very common scenario. In my project, clicking certain reports opened them in a new tab. The way I handle this is by grabbing the main window handle using `driver.getWindowHandle()`. After the click, I use `driver.getWindowHandles()` to get all open handles. I iterate through the set, and switch to the handle that is not equal to the main window. 

After performing my actions and assertions in the child window, I close it using `driver.close()` (not `quit()`), and switch back to the parent window so the test can continue.

```java
public void verifyReportInNewTab() {
    String parentWindow = driver.getWindowHandle();
    
    // Action that opens new tab
    click(viewReportBtn);
    
    Set<String> allWindows = driver.getWindowHandles();
    for (String window : allWindows) {
        if (!window.equals(parentWindow)) {
            driver.switchTo().window(window);
            break;
        }
    }
    
    // Now we are in the child tab
    Assert.assertTrue(getText(reportHeader).contains("Q3 Financials"));
    
    // Close child tab and switch back
    driver.close();
    driver.switchTo().window(parentWindow);
}
```

### 24. Iframe Handling 🔥
**Q: How do you interact with elements inside an iframe?** 🔥
**A:** In my project, our payment gateway integration was loaded inside a third-party iframe. You cannot interact with iframe elements directly; you'll get a `NoSuchElementException`. What we typically do is switch context to the iframe using `driver.switchTo().frame()`. 

You can switch by index, name/ID, or WebElement. I always prefer switching by WebElement because it's the most robust. After entering the credit card details, it's critical to switch back to the main document using `switchTo().defaultContent()`.

```java
public void enterPaymentDetails() {
    // Switch to iframe using WebElement
    WebElement paymentFrame = waitForElementVisible(By.cssSelector("iframe.payment-widget"));
    driver.switchTo().frame(paymentFrame);
    
    // Now interact with elements inside the frame
    type(By.id("cardNumber"), "4111 1111 1111 1111");
    type(By.id("cvv"), "123");
    click(By.id("submitPayment"));
    
    // Switch back to the main page!
    driver.switchTo().defaultContent();
    
    // Verify success message on main page
    Assert.assertTrue(isDisplayed(successMessage));
}
```

### 25. Dynamic Dropdown / Auto-Suggest
**Q: How do you handle dynamic auto-suggestion dropdowns, like Google Search or flight booking destinations?**
**A:** We had exactly this scenario in our travel module for airport searches. You can't use the `Select` class here. The strategy is to type the partial text, wait for the dropdown list (usually a `<ul>` or a list of `<div>`s) to become visible, fetch all the suggestions into a `List<WebElement>`, iterate through them, and click the one that matches the desired text.

```java
public void selectAirport(String inputCity, String desiredAirport) {
    type(cityInputBox, inputCity);
    
    // Wait for the suggestion list container to be visible
    waitForElementVisible(By.cssSelector("ul.auto-suggest-list"));
    
    // Find all list items
    List<WebElement> suggestions = driver.findElements(By.cssSelector("ul.auto-suggest-list > li"));
    
    for (WebElement suggestion : suggestions) {
        if (suggestion.getText().trim().equalsIgnoreCase(desiredAirport)) {
            suggestion.click();
            break; // Important to break after clicking
        }
    }
}
```

### 27. WebTable Data Extraction 🔥
**Q: How do you handle dynamic web tables and extract data from a specific row?** 🔥
**A:** Handling web tables is something we do daily in my project for reporting grids. If I need to find a specific row—say, finding an employee named "John" and clicking the "Delete" button next to his name—I write a dynamic XPath. 

I don't hardcode row indexes because the data order changes. Instead, I locate the cell containing "John", navigate up to the parent `<tr>`, and then find the delete button within that specific row.

```java
public void deleteEmployee(String empName) {
    // Dynamic XPath: Find cell with name -> go to parent row -> find delete button
    String xpath = "//td[text()='" + empName + "']/parent::tr//button[@title='Delete']";
    
    // Wait for it and click
    WebElement deleteBtn = wait.until(ExpectedConditions.elementToBeClickable(By.xpath(xpath)));
    deleteBtn.click();
    
    // Accept alert if any
    driver.switchTo().alert().accept();
}

public List<String> getAllColumnData(int columnIndex) {
    List<String> colData = new ArrayList<>();
    List<WebElement> cells = driver.findElements(By.xpath("//table[@id='empTable']//tbody/tr/td[" + columnIndex + "]"));
    
    for (WebElement cell : cells) {
        colData.add(cell.getText());
    }
    return colData;
}
```

### 28. Broken Links Validation
**Q: How do you verify if there are any broken links on a webpage?**
**A:** In my project, part of our regression suite checks for dead links on landing pages. The strategy is to fetch all elements with an `<a>` tag, get their `href` attribute, and then make a fast HTTP GET or HEAD request using Java's `HttpURLConnection`. If the response code is 400 or greater, it's a broken link. We collect all broken links in a list and fail the test at the end if the list isn't empty using SoftAssert.

```java
public void verifyBrokenLinks() {
    List<WebElement> links = driver.findElements(By.tagName("a"));
    List<String> brokenLinks = new ArrayList<>();
    
    for (WebElement link : links) {
        String url = link.getAttribute("href");
        if (url != null && !url.isEmpty()) {
            try {
                HttpURLConnection connection = (HttpURLConnection) new URL(url).openConnection();
                connection.setConnectTimeout(5000);
                connection.setRequestMethod("HEAD");
                connection.connect();
                
                if (connection.getResponseCode() >= 400) {
                    brokenLinks.add(url + " - " + connection.getResponseCode());
                }
            } catch (Exception e) {
                brokenLinks.add(url + " - Exception occurred");
            }
        }
    }
    Assert.assertTrue(brokenLinks.isEmpty(), "Found broken links: " + brokenLinks);
}
```

### 32 & 33. OTP and CAPTCHA Handling Strategy 🔥
**Q: How do you automate scenarios involving OTPs or CAPTCHAs?** 🔥
**A:** This is a classic interview question. The definitive answer is: **You cannot and should not automate them directly.** The entire purpose of OTP and CAPTCHA is to prevent automated bots, so trying to automate them defeats the security mechanism.

In my project, the way we handle this depends on the environment:
1. **For Test/QA Environments:** We work with developers to **disable** the CAPTCHA entirely. For OTPs, we ask them to configure a static OTP (like `123456`) or we query the test database directly via JDBC to retrieve the generated OTP. Sometimes, they expose an internal API endpoint that we can hit to get the latest OTP for a specific test user.
2. **For Production Environments:** If we absolutely must run a sanity suite in Prod, we use test accounts that are whitelisted in the backend to bypass OTP/CAPTCHA, or we inject an API key in the request headers that the backend recognizes as the automation framework.

If they force you to ask how to handle CAPTCHA *if forced*, you can mention third-party tools like 2Captcha or optical character recognition (Tesseract OCR), but strongly emphasize that these are flaky, slow, and not recommended for enterprise pipelines.
