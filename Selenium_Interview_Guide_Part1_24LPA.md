# 🚀 Selenium + Java SDET Interview Guide — 24 LPA Level

> **Covers**: Core Selenium, WebDriver internals, Frameworks, Design Patterns, CI/CD, Edge Cases, and Senior-level concepts.

---

## 📋 Table of Contents

1. [Selenium Architecture & Internals](#1-selenium-architecture--internals)
2. [WebDriver Setup & Configuration](#2-webdriver-setup--configuration)
3. [Locators — All Types & Best Practices](#3-locators--all-types--best-practices)
4. [Waits — Implicit, Explicit, Fluent](#4-waits--implicit-explicit-fluent)
5. [Actions Class — Mouse & Keyboard Events](#5-actions-class--mouse--keyboard-events)
6. [JavaScript Executor](#6-javascript-executor)
7. [Frames, iFrames & Windows](#7-frames-iframes--windows)
8. [File Upload & Download](#8-file-upload--download)
9. [Alerts, Popups & Dialogues](#9-alerts-popups--dialogues)
10. [Select Dropdown — WebElement & Custom](#10-select-dropdown--webelement--custom)
11. [Page Object Model (POM)](#11-page-object-model-pom)
12. [Page Factory & PageFactory Annotations](#12-page-factory--pagefactory-annotations)
13. [TestNG — Full Deep Dive](#13-testng--full-deep-dive)
14. [Data-Driven Testing](#14-data-driven-testing)
15. [Framework Design — Hybrid Framework](#15-framework-design--hybrid-framework)
16. [Screenshot & Reporting (Extent Reports)](#16-screenshot--reporting-extent-reports)
17. [Parallel Execution](#17-parallel-execution)
18. [Headless Browser Testing](#18-headless-browser-testing)
19. [Grid & Remote WebDriver](#19-grid--remote-webdriver)
20. [Handling Dynamic Elements & AJAX](#20-handling-dynamic-elements--ajax)
21. [Stale Element Reference Exception](#21-stale-element-reference-exception)
22. [Cross-Browser Testing Edge Cases](#22-cross-browser-testing-edge-cases)
23. [REST API Testing with RestAssured](#23-rest-api-testing-with-restassured)
24. [CI/CD — Jenkins Integration](#24-cicd--jenkins-integration)
25. [Design Patterns in Test Automation](#25-design-patterns-in-test-automation)
26. [Senior-Level Tricky Questions](#26-senior-level-tricky-questions)

---

## 1. Selenium Architecture & Internals

### Q: Explain Selenium WebDriver Architecture in detail?

**Answer:**

```
Test Script (Java) → WebDriver API → Browser Driver (ChromeDriver/GeckoDriver) → Browser
```

Selenium 4 uses **W3C WebDriver Protocol** (JSON Wire Protocol is deprecated).

```
[Test Code] --HTTP JSON--> [ChromeDriver] --CDP/BiDi--> [Chrome Browser]
```

- **Client Library**: Your Java test code
- **JSON Wire Protocol / W3C Protocol**: Communication format over HTTP
- **Browser Driver**: Translates commands to browser-specific instructions
- **Browser**: Executes commands

### Q: What is the difference between Selenium 3 and Selenium 4?

| Feature | Selenium 3 | Selenium 4 |
|---|---|---|
| Protocol | JSON Wire Protocol | W3C WebDriver (native) |
| CDP Support | No | Yes (Chrome DevTools Protocol) |
| Relative Locators | No | Yes |
| Grid | Requires Hub+Node | New Grid (Standalone/Node/Router) |
| `driver.manage().window()` | Basic | Enhanced (`getSize`, `setSize`) |
| Screenshots | Basic | Full page screenshot possible |

```java
// Selenium 4 — New Window/Tab
WebDriver driver = new ChromeDriver();
driver.get("https://www.google.com");

// Open new tab (Selenium 4 feature)
driver.switchTo().newWindow(WindowType.TAB);
driver.get("https://www.amazon.com");

// Open new window
driver.switchTo().newWindow(WindowType.WINDOW);
driver.get("https://www.flipkart.com");
```

---

## 2. WebDriver Setup & Configuration

### Q: How do you set up WebDriver with WebDriverManager?

```java
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.firefox.FirefoxDriver;
import org.openqa.selenium.edge.EdgeDriver;

public class DriverSetup {

    // Option 1: WebDriverManager (Auto manages driver binaries)
    public WebDriver getChromeDriver() {
        WebDriverManager.chromedriver().setup();
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--start-maximized");
        options.addArguments("--disable-notifications");
        options.addArguments("--disable-infobars");
        options.addArguments("--disable-extensions");
        // Bypass SSL errors
        options.addArguments("--ignore-certificate-errors");
        options.setAcceptInsecureCerts(true);
        return new ChromeDriver(options);
    }

    // Option 2: Selenium 4 built-in driver management (no external dependency)
    public WebDriver getChromeDriverSelenium4() {
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--start-maximized");
        return new ChromeDriver(options); // Selenium 4.6+ auto-downloads
    }
}
```

### Q: Explain DriverFactory / ThreadLocal pattern for parallel execution?

```java
public class DriverFactory {

    // ThreadLocal ensures each thread has its own WebDriver instance
    private static ThreadLocal<WebDriver> driver = new ThreadLocal<>();

    public static WebDriver getDriver() {
        return driver.get();
    }

    public static void setDriver(String browser) {
        WebDriver webDriver;
        switch (browser.toLowerCase()) {
            case "chrome":
                WebDriverManager.chromedriver().setup();
                ChromeOptions chromeOptions = new ChromeOptions();
                chromeOptions.addArguments("--start-maximized");
                webDriver = new ChromeDriver(chromeOptions);
                break;
            case "firefox":
                WebDriverManager.firefoxdriver().setup();
                webDriver = new FirefoxDriver();
                break;
            case "edge":
                WebDriverManager.edgedriver().setup();
                webDriver = new EdgeDriver();
                break;
            default:
                throw new IllegalArgumentException("Browser not supported: " + browser);
        }
        driver.set(webDriver);
    }

    public static void quitDriver() {
        if (driver.get() != null) {
            driver.get().quit();
            driver.remove(); // CRITICAL: Prevents memory leaks in thread pools
        }
    }
}
```

---

## 3. Locators — All Types & Best Practices

### Q: What are all types of locators in Selenium? Which is best?

```java
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.support.locators.RelativeLocator;

public class LocatorExamples {

    public void demonstrateAllLocators(WebDriver driver) {

        // 1. ID — FASTEST, most reliable
        WebElement byId = driver.findElement(By.id("username"));

        // 2. Name
        WebElement byName = driver.findElement(By.name("email"));

        // 3. Class Name — AVOID if multiple classes or dynamic classes
        WebElement byClass = driver.findElement(By.className("login-btn"));

        // 4. Tag Name — returns multiple usually
        WebElement byTag = driver.findElement(By.tagName("input"));

        // 5. Link Text — exact match for anchor tags
        WebElement byLinkText = driver.findElement(By.linkText("Sign In"));

        // 6. Partial Link Text
        WebElement byPartialLink = driver.findElement(By.partialLinkText("Sign"));

        // 7. CSS Selector — FAST, flexible, preferred over XPath
        WebElement byCss = driver.findElement(By.cssSelector("input[id='username']"));
        WebElement byCssClass = driver.findElement(By.cssSelector(".login-form .submit-btn"));
        WebElement byCssAttr = driver.findElement(By.cssSelector("input[placeholder='Enter email']"));
        WebElement byCssChild = driver.findElement(By.cssSelector("div.form > input:first-child"));
        WebElement byCssPseudo = driver.findElement(By.cssSelector("ul li:nth-child(3)"));

        // 8. XPath — most powerful, use when CSS can't do it
        WebElement byXpath = driver.findElement(By.xpath("//input[@id='username']"));
        WebElement byXpathText = driver.findElement(By.xpath("//button[text()='Login']"));
        WebElement byXpathContains = driver.findElement(By.xpath("//button[contains(text(),'Log')]"));
        WebElement byXpathParent = driver.findElement(By.xpath("//label[@for='email']/.."));
        WebElement byXpathSibling = driver.findElement(By.xpath("//label[@for='email']/following-sibling::input"));
        WebElement byXpathAnd = driver.findElement(By.xpath("//input[@type='text' and @name='user']"));
        WebElement byXpathOr = driver.findElement(By.xpath("//input[@type='text' or @name='user']"));
        WebElement byXpathIndex = driver.findElement(By.xpath("(//input[@type='text'])[2]")); // 2nd match

        // 9. Relative Locators (Selenium 4)
        WebElement passwordField = driver.findElement(By.id("password"));
        WebElement usernameField2 = driver.findElement(RelativeLocator.with(By.tagName("input"))
                .above(passwordField));

        WebElement submitBtn = driver.findElement(RelativeLocator.with(By.tagName("button"))
                .below(By.id("password"))
                .toRightOf(By.id("cancel")));
    }
}
```

### Q: CSS vs XPath — When to use which?

| Scenario | CSS | XPath |
|---|---|---|
| By attribute | `input[type='text']` | `//input[@type='text']` |
| By text content | ❌ Not possible | `//button[text()='Submit']` |
| Parent traversal | ❌ Not possible | `//span/..` (parent axis) |
| Following sibling | `label + input` (adjacent) | `//label/following-sibling::input` |
| Performance | Faster | Slightly slower |
| Readability | Cleaner | More expressive |

---

## 4. Waits — Implicit, Explicit, Fluent

### Q: Explain all types of waits and their differences. What are the anti-patterns?

```java
import org.openqa.selenium.*;
import org.openqa.selenium.support.ui.*;
import java.time.Duration;
import java.util.function.Function;

public class WaitsDeepDive {

    WebDriver driver;

    // ❌ ANTI-PATTERN: Thread.sleep — hardcoded, wastes time
    public void badWait() throws InterruptedException {
        Thread.sleep(5000); // Never do this in production
    }

    // 1. IMPLICIT WAIT — applies globally to ALL findElement calls
    // ⚠️ Problem: mixes with explicit wait causing unpredictable timeouts
    public void implicitWait() {
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
        WebElement el = driver.findElement(By.id("username")); // waits up to 10s
    }

    // 2. EXPLICIT WAIT — waits for specific condition on specific element
    public void explicitWait() {
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(15));

        // Wait for element to be clickable
        WebElement btn = wait.until(ExpectedConditions.elementToBeClickable(By.id("submit")));
        btn.click();

        // Wait for element visible
        WebElement el = wait.until(ExpectedConditions.visibilityOfElementLocated(By.xpath("//div[@class='result']")));

        // Wait for text
        wait.until(ExpectedConditions.textToBePresentInElementLocated(By.id("msg"), "Success"));

        // Wait for URL
        wait.until(ExpectedConditions.urlContains("dashboard"));

        // Wait for element to disappear
        wait.until(ExpectedConditions.invisibilityOfElementLocated(By.id("loader")));

        // Wait for number of windows
        wait.until(ExpectedConditions.numberOfWindowsToBe(2));

        // Wait for alert
        wait.until(ExpectedConditions.alertIsPresent());

        // Custom expected condition
        wait.until(driver -> {
            WebElement spinner = driver.findElement(By.id("spinner"));
            return !spinner.isDisplayed();
        });
    }

    // 3. FLUENT WAIT — explicit wait with polling interval and exception ignoring
    public void fluentWait() {
        Wait<WebDriver> fluentWait = new FluentWait<>(driver)
                .withTimeout(Duration.ofSeconds(30))
                .pollingEvery(Duration.ofMillis(500)) // check every 500ms
                .ignoring(NoSuchElementException.class) // ignore these exceptions
                .ignoring(StaleElementReferenceException.class)
                .withMessage("Element not found within 30 seconds");

        WebElement element = fluentWait.until(driver -> {
            WebElement el = driver.findElement(By.id("dynamicElement"));
            return el.isDisplayed() ? el : null;
        });
    }

    // 4. PAGE LOAD TIMEOUT
    public void pageLoadTimeout() {
        driver.manage().timeouts().pageLoadTimeout(Duration.ofSeconds(30));
        driver.manage().timeouts().scriptTimeout(Duration.ofSeconds(10)); // for async JS
    }

    // ⭐ BEST PRACTICE: Custom wait utility
    public WebElement waitForElement(By locator, int timeoutSecs) {
        return new WebDriverWait(driver, Duration.ofSeconds(timeoutSecs))
                .until(ExpectedConditions.visibilityOfElementLocated(locator));
    }

    // ⭐ EDGE CASE: Never mix implicit + explicit wait
    // Implicit(10s) + Explicit(5s) = could wait 15s on failure (not 5s)
}
```

---

## 5. Actions Class — Mouse & Keyboard Events

### Q: How do you handle mouse hover, drag and drop, right-click, double-click?

```java
import org.openqa.selenium.*;
import org.openqa.selenium.interactions.Actions;
import java.util.List;

public class ActionsClassDeepDive {

    WebDriver driver;
    Actions actions;

    public ActionsClassDeepDive(WebDriver driver) {
        this.driver = driver;
        this.actions = new Actions(driver);
    }

    // 1. Mouse Hover
    public void mouseHover() {
        WebElement menu = driver.findElement(By.id("mainMenu"));
        actions.moveToElement(menu).perform();
        // Now submenu appears
        WebElement subMenu = driver.findElement(By.id("subMenu"));
        subMenu.click();
    }

    // 2. Double Click
    public void doubleClick() {
        WebElement element = driver.findElement(By.id("doubleClickBtn"));
        actions.doubleClick(element).perform();
    }

    // 3. Right Click (Context Menu)
    public void rightClick() {
        WebElement element = driver.findElement(By.id("rightClickArea"));
        actions.contextClick(element).perform();
        // Then select option from context menu
        WebElement menuOption = driver.findElement(By.xpath("//li[text()='Copy']"));
        menuOption.click();
    }

    // 4. Drag and Drop — Method 1: dragAndDrop()
    public void dragAndDrop() {
        WebElement source = driver.findElement(By.id("draggable"));
        WebElement target = driver.findElement(By.id("droppable"));
        actions.dragAndDrop(source, target).perform();
    }

    // 5. Drag and Drop — Method 2: clickAndHold + release (more reliable)
    public void dragAndDropAlternative() {
        WebElement source = driver.findElement(By.id("draggable"));
        WebElement target = driver.findElement(By.id("droppable"));
        actions.clickAndHold(source)
               .moveToElement(target)
               .release()
               .perform();
    }

    // 6. Drag and Drop by Offset
    public void dragAndDropByOffset() {
        WebElement source = driver.findElement(By.id("slider"));
        actions.dragAndDropBy(source, 200, 0).perform(); // move 200px right
    }

    // 7. Keyboard Actions
    public void keyboardActions() {
        WebElement input = driver.findElement(By.id("search"));

        // Select all and delete
        actions.click(input)
               .keyDown(Keys.CONTROL)
               .sendKeys("a")
               .keyUp(Keys.CONTROL)
               .sendKeys(Keys.DELETE)
               .perform();

        // Type with keyboard
        actions.sendKeys(input, "Selenium Testing").perform();

        // Ctrl + C, Ctrl + V
        actions.keyDown(Keys.CONTROL).sendKeys("c").keyUp(Keys.CONTROL).perform();
        actions.keyDown(Keys.CONTROL).sendKeys("v").keyUp(Keys.CONTROL).perform();

        // Tab, Enter, Escape
        actions.sendKeys(Keys.TAB).perform();
        actions.sendKeys(Keys.ENTER).perform();
    }

    // 8. Scroll Actions (Selenium 4)
    public void scrollActions() {
        WebElement element = driver.findElement(By.id("footer"));
        // Scroll element into view
        actions.scrollToElement(element).perform();

        // Scroll by amount
        actions.scrollByAmount(0, 500).perform(); // scroll 500px down
    }

    // 9. Build complex action chain
    public void complexChain() {
        WebElement slider = driver.findElement(By.id("slider"));
        actions.clickAndHold(slider)
               .moveByOffset(100, 0)
               .pause(Duration.ofMillis(500))
               .moveByOffset(100, 0)
               .release()
               .perform();
    }
}
```

---

## 6. JavaScript Executor

### Q: When and how do you use JavascriptExecutor? Give real-world examples.

```java
import org.openqa.selenium.*;

public class JavaScriptExecutorExamples {

    WebDriver driver;
    JavascriptExecutor js;

    public JavaScriptExecutorExamples(WebDriver driver) {
        this.driver = driver;
        this.js = (JavascriptExecutor) driver;
    }

    // 1. Click element via JS (when normal click fails due to overlay)
    public void jsClick(WebElement element) {
        js.executeScript("arguments[0].click();", element);
    }

    // 2. Scroll into view
    public void scrollIntoView(WebElement element) {
        js.executeScript("arguments[0].scrollIntoView(true);", element);
    }

    // 3. Scroll to bottom of page
    public void scrollToBottom() {
        js.executeScript("window.scrollTo(0, document.body.scrollHeight)");
    }

    // 4. Scroll to top
    public void scrollToTop() {
        js.executeScript("window.scrollTo(0, 0)");
    }

    // 5. Type in read-only / disabled input
    public void typeInReadonlyField(WebElement element, String text) {
        js.executeScript("arguments[0].removeAttribute('readonly');", element);
        element.clear();
        element.sendKeys(text);
    }

    // 6. Set value directly
    public void setValue(WebElement element, String value) {
        js.executeScript("arguments[0].value='" + value + "';", element);
    }

    // 7. Get return value from JS
    public String getInnerText(WebElement element) {
        return (String) js.executeScript("return arguments[0].innerText;", element);
    }

    // 8. Get page title
    public String getPageTitle() {
        return (String) js.executeScript("return document.title;");
    }

    // 9. Highlight element (useful for debugging)
    public void highlightElement(WebElement element) {
        js.executeScript(
            "arguments[0].style.border='3px solid red';" +
            "arguments[0].style.backgroundColor='yellow';",
            element
        );
    }

    // 10. Check if page fully loaded
    public boolean isPageLoaded() {
        return js.executeScript("return document.readyState").equals("complete");
    }

    // 11. Open new tab
    public void openNewTab() {
        js.executeScript("window.open('https://www.google.com', '_blank')");
    }

    // 12. Async script (for AJAX calls)
    public void asyncScriptExample() {
        js.executeAsyncScript(
            "var callback = arguments[arguments.length - 1];" +
            "setTimeout(function() { callback('done'); }, 3000);"
        );
    }

    // ⭐ EDGE CASE: JS click vs normal click
    // Normal click → triggers real browser events (mousedown, mouseup, click)
    // JS click → directly calls the click handler, bypasses visual interaction
    // Use JS click only when absolutely needed (hidden element, covered element)
}
```

---

## 7. Frames, iFrames & Windows

### Q: How do you handle iFrames and multiple windows/tabs?

```java
import org.openqa.selenium.*;
import org.openqa.selenium.support.ui.*;
import java.time.Duration;
import java.util.*;

public class FramesAndWindows {

    WebDriver driver;

    // ==================== iFRAMES ====================

    // Switch by index
    public void switchToFrameByIndex() {
        driver.switchTo().frame(0); // first iframe on page
        // interact with elements inside iframe
        driver.findElement(By.id("innerElement")).click();
        // Switch back to main content
        driver.switchTo().defaultContent();
    }

    // Switch by name or id
    public void switchToFrameByNameOrId() {
        driver.switchTo().frame("myFrame"); // by name attribute
        driver.switchTo().frame("frameId"); // by id attribute
        driver.switchTo().defaultContent();
    }

    // Switch by WebElement
    public void switchToFrameByElement() {
        WebElement frameEl = driver.findElement(By.cssSelector("iframe.payment-frame"));
        driver.switchTo().frame(frameEl);
        driver.findElement(By.id("cardNumber")).sendKeys("4111111111111111");
        driver.switchTo().defaultContent();
    }

    // Nested iframes
    public void handleNestedIframes() {
        driver.switchTo().frame("outerFrame");
        driver.switchTo().frame("innerFrame"); // switch to nested frame
        driver.findElement(By.id("deepElement")).click();
        driver.switchTo().parentFrame(); // go up one level (Selenium 4)
        driver.switchTo().defaultContent(); // go back to main page
    }

    // ==================== WINDOWS & TABS ====================

    // Handle new window/tab
    public void handleNewWindow() {
        String mainWindow = driver.getWindowHandle();
        driver.findElement(By.linkText("Open New Window")).click();

        // Wait for new window
        new WebDriverWait(driver, Duration.ofSeconds(10))
            .until(ExpectedConditions.numberOfWindowsToBe(2));

        // Get all window handles
        Set<String> allWindows = driver.getWindowHandles();

        for (String window : allWindows) {
            if (!window.equals(mainWindow)) {
                driver.switchTo().window(window);
                System.out.println("New window title: " + driver.getTitle());
                // do work in new window
                break;
            }
        }

        // Close new window and switch back
        driver.close();
        driver.switchTo().window(mainWindow);
    }

    // Handle multiple windows with titles
    public void switchToWindowByTitle(String targetTitle) {
        Set<String> handles = driver.getWindowHandles();
        for (String handle : handles) {
            driver.switchTo().window(handle);
            if (driver.getTitle().equals(targetTitle)) {
                return; // found the window
            }
        }
        throw new RuntimeException("Window with title '" + targetTitle + "' not found");
    }

    // Selenium 4 — Open new tab programmatically
    public void openNewTabSelenium4() {
        String originalTab = driver.getWindowHandle();
        driver.switchTo().newWindow(WindowType.TAB);
        driver.get("https://www.google.com");
        // do stuff in new tab
        driver.close();
        driver.switchTo().window(originalTab);
    }

    // ⭐ EDGE CASE: Window handle order is not guaranteed in Set
    // Always switch by title, URL, or use the last handle trick
    public String getNewWindowHandle(String existingHandle) {
        Set<String> handles = driver.getWindowHandles();
        return handles.stream()
                      .filter(h -> !h.equals(existingHandle))
                      .findFirst()
                      .orElseThrow(() -> new RuntimeException("New window not found"));
    }
}
```

---

## 8. File Upload & Download

### Q: How do you handle file upload and download in Selenium?

```java
import org.openqa.selenium.*;
import java.io.*;
import java.util.HashMap;

public class FileUploadDownload {

    // ===== FILE UPLOAD =====

    // Method 1: sendKeys on input[type='file'] — MOST RELIABLE
    public void uploadFileWithSendKeys(WebDriver driver) {
        WebElement uploadInput = driver.findElement(By.cssSelector("input[type='file']"));
        // Must provide absolute path
        uploadInput.sendKeys("D:\\TestFiles\\document.pdf");
    }

    // Method 2: Upload hidden input (make it visible first via JS)
    public void uploadHiddenFileInput(WebDriver driver) {
        JavascriptExecutor js = (JavascriptExecutor) driver;
        WebElement fileInput = driver.findElement(By.cssSelector("input[type='file']"));

        // Remove display:none or visibility:hidden
        js.executeScript(
            "arguments[0].style.display='block';" +
            "arguments[0].style.visibility='visible';",
            fileInput
        );
        fileInput.sendKeys("D:\\TestFiles\\image.png");
    }

    // Method 3: AutoIT (Windows only, for OS-level dialogs)
    // Use when the file dialog is OS-level, not browser-level
    public void uploadWithAutoIT(WebDriver driver) throws Exception {
        driver.findElement(By.id("upload")).click();
        // Run AutoIT script
        Runtime.getRuntime().exec("D:\\Scripts\\upload.exe D:\\TestFiles\\file.pdf");
        Thread.sleep(2000);
    }

    // ===== FILE DOWNLOAD =====

    // Chrome — configure download directory
    public WebDriver setupChromeForDownload(String downloadPath) {
        HashMap<String, Object> prefs = new HashMap<>();
        prefs.put("download.default_directory", downloadPath);
        prefs.put("download.prompt_for_download", false);
        prefs.put("download.directory_upgrade", true);
        prefs.put("safebrowsing.enabled", true);
        prefs.put("plugins.always_open_pdf_externally", true); // auto-download PDFs

        ChromeOptions options = new ChromeOptions();
        options.setExperimentalOption("prefs", prefs);
        return new ChromeDriver(options);
    }

    // Verify file was downloaded
    public boolean isFileDownloaded(String downloadDir, String fileName, int timeoutSecs)
            throws InterruptedException {
        File directory = new File(downloadDir);
        long endTime = System.currentTimeMillis() + (timeoutSecs * 1000L);

        while (System.currentTimeMillis() < endTime) {
            File[] files = directory.listFiles();
            if (files != null) {
                for (File file : files) {
                    if (file.getName().equals(fileName) && !file.getName().endsWith(".crdownload")) {
                        return true; // fully downloaded
                    }
                }
            }
            Thread.sleep(500);
        }
        return false;
    }

    // Wait for download completion by checking file size stability
    public void waitForDownloadComplete(String filePath, int maxWaitSecs)
            throws InterruptedException {
        File file = new File(filePath);
        long previousSize = -1;
        int stableCount = 0;

        for (int i = 0; i < maxWaitSecs * 2; i++) {
            Thread.sleep(500);
            if (file.exists()) {
                long currentSize = file.length();
                if (currentSize == previousSize && currentSize > 0) {
                    stableCount++;
                    if (stableCount >= 3) return; // size stable for 1.5s = complete
                } else {
                    stableCount = 0;
                }
                previousSize = currentSize;
            }
        }
    }
}
```

---

## 9. Alerts, Popups & Dialogues

### Q: How do you handle different types of alerts in Selenium?

```java
import org.openqa.selenium.*;
import org.openqa.selenium.support.ui.*;
import java.time.Duration;

public class AlertsHandling {

    WebDriver driver;

    // 1. Simple Alert (OK only)
    public void handleSimpleAlert() {
        // Wait for alert
        new WebDriverWait(driver, Duration.ofSeconds(10))
            .until(ExpectedConditions.alertIsPresent());

        Alert alert = driver.switchTo().alert();
        System.out.println("Alert text: " + alert.getText());
        alert.accept(); // Click OK
    }

    // 2. Confirm Alert (OK + Cancel)
    public void handleConfirmAlert(boolean accept) {
        new WebDriverWait(driver, Duration.ofSeconds(10))
            .until(ExpectedConditions.alertIsPresent());

        Alert alert = driver.switchTo().alert();
        if (accept) {
            alert.accept(); // Click OK
        } else {
            alert.dismiss(); // Click Cancel
        }
    }

    // 3. Prompt Alert (text input + OK/Cancel)
    public void handlePromptAlert(String inputText) {
        new WebDriverWait(driver, Duration.ofSeconds(10))
            .until(ExpectedConditions.alertIsPresent());

        Alert alert = driver.switchTo().alert();
        alert.sendKeys(inputText); // Type in prompt
        alert.accept();
    }

    // 4. Safe alert handling (no exception if alert not present)
    public boolean isAlertPresent() {
        try {
            driver.switchTo().alert();
            return true;
        } catch (NoAlertPresentException e) {
            return false;
        }
    }

    // 5. Browser Authentication Popup (Basic Auth)
    public void handleBasicAuth() {
        // Method 1: Embed credentials in URL
        driver.get("https://admin:password@example.com/secure");

        // Method 2: Selenium 4 with CDP
        // (for Chrome DevTools Protocol authentication)
    }

    // ⭐ EDGE CASE: Handling alerts from beforeunload (navigation alert)
    public void handleNavigationAlert() {
        driver.findElement(By.id("navigateAway")).click();
        // Alert appears asking "Leave page?"
        driver.switchTo().alert().accept();
    }

    // ⭐ EDGE CASE: Alert with autofocus (alert appears before page loads)
    public void handleEarlyAlert() {
        driver.get("https://alertpage.com");
        // Use FluentWait with short poll to catch early alert
        new WebDriverWait(driver, Duration.ofSeconds(5))
            .until(ExpectedConditions.alertIsPresent());
        driver.switchTo().alert().accept();
    }
}
```

---

## 10. Select Dropdown — WebElement & Custom

### Q: How do you handle standard and custom dropdowns?

```java
import org.openqa.selenium.*;
import org.openqa.selenium.support.ui.Select;
import java.util.List;

public class DropdownHandling {

    WebDriver driver;

    // ===== STANDARD HTML SELECT =====

    public void handleStandardDropdown() {
        WebElement dropdownEl = driver.findElement(By.id("country"));
        Select dropdown = new Select(dropdownEl);

        // Select by visible text
        dropdown.selectByVisibleText("India");

        // Select by value attribute
        dropdown.selectByValue("IN");

        // Select by index (0-based)
        dropdown.selectByIndex(2);

        // Multi-select
        if (dropdown.isMultiple()) {
            dropdown.selectByVisibleText("India");
            dropdown.selectByVisibleText("USA");
            dropdown.deselectAll(); // deselect all
            dropdown.deselectByVisibleText("India");
        }

        // Get all options
        List<WebElement> options = dropdown.getOptions();
        for (WebElement option : options) {
            System.out.println(option.getText());
        }

        // Get selected option
        System.out.println("Selected: " + dropdown.getFirstSelectedOption().getText());
    }

    // ===== CUSTOM DROPDOWN (not <select> tag) =====

    public void handleCustomDropdown() {
        // Click to open
        driver.findElement(By.cssSelector(".custom-dropdown-trigger")).click();

        // Wait for options to appear
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(5));
        wait.until(ExpectedConditions.visibilityOfElementLocated(
            By.cssSelector(".dropdown-options")));

        // Select specific option
        List<WebElement> options = driver.findElements(
            By.cssSelector(".dropdown-options li"));

        for (WebElement option : options) {
            if (option.getText().equalsIgnoreCase("India")) {
                option.click();
                break;
            }
        }
    }

    // ===== AUTOCOMPLETE / TYPEAHEAD DROPDOWN =====

    public void handleAutocomplete() {
        WebElement searchInput = driver.findElement(By.id("autocomplete"));
        searchInput.sendKeys("Ind");

        // Wait for suggestions
        new WebDriverWait(driver, Duration.ofSeconds(5))
            .until(ExpectedConditions.visibilityOfElementLocated(
                By.cssSelector(".suggestions-list")));

        // Select from suggestions
        List<WebElement> suggestions = driver.findElements(
            By.cssSelector(".suggestions-list li"));

        for (WebElement suggestion : suggestions) {
            if (suggestion.getText().contains("India")) {
                suggestion.click();
                break;
            }
        }
    }

    // ===== REACT / ANGULAR SELECT (Material UI) =====

    public void handleMaterialUIDropdown() {
        // Click the mat-select component
        driver.findElement(By.cssSelector("mat-select[formcontrolname='country']")).click();

        // Wait for overlay panel
        new WebDriverWait(driver, Duration.ofSeconds(5))
            .until(ExpectedConditions.visibilityOfElementLocated(
                By.cssSelector("mat-option")));

        // Click the option
        List<WebElement> matOptions = driver.findElements(By.cssSelector("mat-option"));
        matOptions.stream()
                  .filter(o -> o.getText().equals("India"))
                  .findFirst()
                  .ifPresent(WebElement::click);
    }
}
```

---

## 11. Page Object Model (POM)

### Q: Design a complete POM framework from scratch.

```java
// ===== LoginPage.java =====
package pages;

import org.openqa.selenium.*;
import org.openqa.selenium.support.ui.*;
import java.time.Duration;

public class LoginPage {

    private WebDriver driver;
    private WebDriverWait wait;

    // Locators as private constants — easy to maintain
    private static final By USERNAME = By.id("username");
    private static final By PASSWORD = By.id("password");
    private static final By LOGIN_BTN = By.cssSelector("button[type='submit']");
    private static final By ERROR_MSG = By.cssSelector(".error-message");
    private static final By REMEMBER_ME = By.id("rememberMe");

    public LoginPage(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(15));
    }

    // ===== Page Actions =====

    public LoginPage enterUsername(String username) {
        wait.until(ExpectedConditions.visibilityOfElementLocated(USERNAME))
            .clear();
        driver.findElement(USERNAME).sendKeys(username);
        return this; // method chaining (Fluent interface)
    }

    public LoginPage enterPassword(String password) {
        driver.findElement(PASSWORD).sendKeys(password);
        return this;
    }

    public LoginPage checkRememberMe() {
        WebElement checkbox = driver.findElement(REMEMBER_ME);
        if (!checkbox.isSelected()) {
            checkbox.click();
        }
        return this;
    }

    public DashboardPage clickLogin() {
        driver.findElement(LOGIN_BTN).click();
        return new DashboardPage(driver); // returns next page
    }

    public LoginPage clickLoginExpectingFailure() {
        driver.findElement(LOGIN_BTN).click();
        return this; // stays on login page for negative test
    }

    // ===== Page Verifications =====

    public String getErrorMessage() {
        return wait.until(ExpectedConditions.visibilityOfElementLocated(ERROR_MSG))
                   .getText();
    }

    public boolean isLoginPageDisplayed() {
        return driver.getCurrentUrl().contains("/login");
    }

    // ===== Reusable Login Method =====

    public DashboardPage login(String username, String password) {
        return enterUsername(username)
               .enterPassword(password)
               .clickLogin();
    }
}

// ===== DashboardPage.java =====
package pages;

import org.openqa.selenium.*;
import org.openqa.selenium.support.ui.*;
import java.time.Duration;

public class DashboardPage {

    private WebDriver driver;
    private WebDriverWait wait;

    private static final By WELCOME_MSG = By.cssSelector(".welcome-header");
    private static final By USER_MENU = By.id("userMenu");
    private static final By LOGOUT_LINK = By.linkText("Logout");

    public DashboardPage(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(15));
        // Verify we are on dashboard page
        wait.until(ExpectedConditions.urlContains("/dashboard"));
    }

    public String getWelcomeMessage() {
        return wait.until(ExpectedConditions.visibilityOfElementLocated(WELCOME_MSG))
                   .getText();
    }

    public LoginPage logout() {
        driver.findElement(USER_MENU).click();
        wait.until(ExpectedConditions.elementToBeClickable(LOGOUT_LINK)).click();
        return new LoginPage(driver);
    }
}

// ===== LoginTest.java =====
package tests;

import org.testng.Assert;
import org.testng.annotations.*;
import pages.*;
import utils.DriverFactory;

public class LoginTest {

    private WebDriver driver;
    private LoginPage loginPage;

    @BeforeMethod
    public void setUp() {
        DriverFactory.setDriver("chrome");
        driver = DriverFactory.getDriver();
        driver.get("https://www.example.com/login");
        loginPage = new LoginPage(driver);
    }

    @Test(description = "Verify successful login with valid credentials")
    public void testSuccessfulLogin() {
        DashboardPage dashboard = loginPage.login("admin", "Admin@123");
        Assert.assertTrue(dashboard.getWelcomeMessage().contains("Welcome"),
                "Welcome message not displayed");
    }

    @Test(description = "Verify error message for invalid credentials")
    public void testInvalidLogin() {
        loginPage.enterUsername("wrong@email.com")
                 .enterPassword("wrongpass")
                 .clickLoginExpectingFailure();

        Assert.assertEquals(loginPage.getErrorMessage(),
                "Invalid username or password",
                "Error message mismatch");
    }

    @AfterMethod
    public void tearDown() {
        DriverFactory.quitDriver();
    }
}
```

---

## 12. Page Factory & PageFactory Annotations

### Q: What is Page Factory? How does it differ from POM?

```java
package pages;

import org.openqa.selenium.*;
import org.openqa.selenium.support.*;
import org.openqa.selenium.support.ui.*;
import java.time.Duration;

public class LoginPageWithFactory {

    private WebDriver driver;

    // @FindBy — lazy initialization, finds element when first used
    @FindBy(id = "username")
    private WebElement usernameInput;

    @FindBy(id = "password")
    private WebElement passwordInput;

    @FindBy(css = "button[type='submit']")
    private WebElement loginBtn;

    @FindBy(css = ".error-message")
    private WebElement errorMessage;

    // Multiple locators — finds first matching
    @FindAll({
        @FindBy(id = "loginBtn"),
        @FindBy(css = "button.submit"),
        @FindBy(xpath = "//button[@type='submit']")
    })
    private WebElement submitButton;

    // Find multiple elements
    @FindBy(css = ".menu-item")
    private List<WebElement> menuItems;

    // ⭐ CacheLookup — caches element reference (RISKY with dynamic pages)
    @FindBy(id = "staticHeader")
    @CacheLookup
    private WebElement header;

    public LoginPageWithFactory(WebDriver driver) {
        this.driver = driver;
        // MUST call this to initialize @FindBy annotations
        PageFactory.initElements(driver, this);

        // With custom timeout using AjaxElementLocatorFactory
        // PageFactory.initElements(new AjaxElementLocatorFactory(driver, 10), this);
    }

    public void login(String username, String password) {
        usernameInput.clear();
        usernameInput.sendKeys(username);
        passwordInput.sendKeys(password);
        loginBtn.click();
    }

    // ⭐ EDGE CASE: @CacheLookup causes StaleElementReferenceException
    // for elements that are recreated in DOM (AJAX, re-renders).
    // Don't use @CacheLookup for dynamic elements.

    // POM vs PageFactory:
    // POM = pattern (organize page elements in separate classes)
    // PageFactory = Selenium tool to implement POM with @FindBy annotations
    // Both work together. PageFactory reduces boilerplate but risks stale refs.
}
```

---

## 13. TestNG — Full Deep Dive

### Q: Explain all TestNG features used in automation frameworks.

```java
import org.testng.annotations.*;
import org.testng.*;
import org.testng.asserts.SoftAssert;

// ===== TestNG Annotations & Lifecycle =====
public class TestNGDeepDive {

    // SUITE → TEST → CLASS → METHOD order

    @BeforeSuite  // runs once before all tests in suite
    public void beforeSuite() { System.out.println("Before Suite"); }

    @AfterSuite
    public void afterSuite() { System.out.println("After Suite"); }

    @BeforeTest   // runs before each <test> tag in testng.xml
    public void beforeTest() { System.out.println("Before Test"); }

    @AfterTest
    public void afterTest() { System.out.println("After Test"); }

    @BeforeClass  // runs once before first method in this class
    public void beforeClass() { System.out.println("Before Class"); }

    @AfterClass
    public void afterClass() { System.out.println("After Class"); }

    @BeforeMethod // runs before EACH test method
    public void beforeMethod(Method method) {
        System.out.println("Starting test: " + method.getName());
    }

    @AfterMethod
    public void afterMethod(ITestResult result) {
        if (result.getStatus() == ITestResult.FAILURE) {
            System.out.println("Test FAILED: " + result.getName());
            // take screenshot here
        }
    }

    // ===== Test Method Features =====

    @Test(description = "Verify login",
          groups = {"smoke", "regression"},
          priority = 1,
          enabled = true,
          timeOut = 30000, // ms
          retryAnalyzer = RetryAnalyzer.class,
          dependsOnMethods = {"testOpenBrowser"},
          dataProvider = "loginData",
          threadPoolSize = 3,
          invocationCount = 5) // run 5 times
    public void testLogin(String user, String pass) {
        // test code
    }

    // ===== Data Provider =====

    @DataProvider(name = "loginData", parallel = true)
    public Object[][] loginData() {
        return new Object[][] {
            {"user1@test.com", "Pass@123"},
            {"user2@test.com", "Pass@456"},
            {"admin@test.com", "Admin@789"}
        };
    }

    // ===== Soft Assert =====
    @Test
    public void testSoftAssert() {
        SoftAssert softAssert = new SoftAssert();
        // Does NOT stop on failure — collects all failures
        softAssert.assertEquals(driver.getTitle(), "Dashboard");
        softAssert.assertTrue(element.isDisplayed(), "Element not visible");
        softAssert.assertEquals(textEl.getText(), "Welcome Admin");
        softAssert.assertAll(); // throws if ANY assertion failed
    }

    // ===== Hard Assert =====
    @Test
    public void testHardAssert() {
        Assert.assertEquals(actual, expected, "Mismatch message");
        // Stops execution immediately on failure
    }

    // ===== Parameters from testng.xml =====
    @Test
    @Parameters({"browser", "url"})
    public void testWithParams(String browser, String url) {
        System.out.println("Browser: " + browser + ", URL: " + url);
    }

    // ===== Retry Analyzer =====
    // (shown separately below)

    // ===== Groups =====
    @Test(groups = "smoke")
    public void smokeTest() {}

    @Test(groups = "regression")
    public void regressionTest() {}
}

// ===== RetryAnalyzer.java =====
import org.testng.IRetryAnalyzer;
import org.testng.ITestResult;

public class RetryAnalyzer implements IRetryAnalyzer {
    private int retryCount = 0;
    private static final int MAX_RETRY = 2;

    @Override
    public boolean retry(ITestResult result) {
        if (retryCount < MAX_RETRY) {
            retryCount++;
            System.out.println("Retrying test: " + result.getName() + " attempt " + retryCount);
            return true;
        }
        return false;
    }
}

// ===== testng.xml =====
/*
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd">
<suite name="RegressionSuite" parallel="tests" thread-count="3">

    <listeners>
        <listener class-name="listeners.ExtentReportListener"/>
    </listeners>

    <test name="ChromeTests">
        <parameter name="browser" value="chrome"/>
        <groups>
            <run>
                <include name="smoke"/>
                <include name="regression"/>
                <exclude name="skip"/>
            </run>
        </groups>
        <classes>
            <class name="tests.LoginTest"/>
            <class name="tests.DashboardTest"/>
        </classes>
    </test>

    <test name="FirefoxTests">
        <parameter name="browser" value="firefox"/>
        <classes>
            <class name="tests.LoginTest"/>
        </classes>
    </test>

</suite>
*/
```

---

## 14. Data-Driven Testing

### Q: How do you implement data-driven testing with Excel and CSV?

```java
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.*;
import org.testng.annotations.*;
import java.io.*;
import java.util.*;

public class DataDrivenTesting {

    // ===== Read from Excel (Apache POI) =====

    public static Object[][] readExcelData(String filePath, String sheetName) throws IOException {
        FileInputStream fis = new FileInputStream(filePath);
        XSSFWorkbook workbook = new XSSFWorkbook(fis);
        XSSFSheet sheet = workbook.getSheet(sheetName);

        int rowCount = sheet.getLastRowNum();
        int colCount = sheet.getRow(0).getLastCellNum();

        // Skip header row (start from row 1)
        Object[][] data = new Object[rowCount][colCount];

        for (int i = 1; i <= rowCount; i++) {
            XSSFRow row = sheet.getRow(i);
            for (int j = 0; j < colCount; j++) {
                XSSFCell cell = row.getCell(j);
                switch (cell.getCellType()) {
                    case STRING:
                        data[i - 1][j] = cell.getStringCellValue();
                        break;
                    case NUMERIC:
                        if (DateUtil.isCellDateFormatted(cell)) {
                            data[i - 1][j] = cell.getDateCellValue();
                        } else {
                            data[i - 1][j] = (int) cell.getNumericCellValue();
                        }
                        break;
                    case BOOLEAN:
                        data[i - 1][j] = cell.getBooleanCellValue();
                        break;
                    default:
                        data[i - 1][j] = "";
                }
            }
        }
        workbook.close();
        fis.close();
        return data;
    }

    // Write to Excel
    public static void writeExcelData(String filePath, String sheetName,
                                       String[] headers, List<String[]> rows) throws IOException {
        XSSFWorkbook workbook = new XSSFWorkbook();
        XSSFSheet sheet = workbook.createSheet(sheetName);

        // Write header
        XSSFRow headerRow = sheet.createRow(0);
        for (int i = 0; i < headers.length; i++) {
            headerRow.createCell(i).setCellValue(headers[i]);
        }

        // Write data rows
        int rowNum = 1;
        for (String[] rowData : rows) {
            XSSFRow row = sheet.createRow(rowNum++);
            for (int i = 0; i < rowData.length; i++) {
                row.createCell(i).setCellValue(rowData[i]);
            }
        }

        FileOutputStream fos = new FileOutputStream(filePath);
        workbook.write(fos);
        fos.close();
        workbook.close();
    }

    // ===== TestNG DataProvider using Excel =====

    @DataProvider(name = "loginDataFromExcel")
    public Object[][] getLoginData() throws IOException {
        return readExcelData("D:\\TestData\\LoginData.xlsx", "LoginSheet");
    }

    @Test(dataProvider = "loginDataFromExcel")
    public void testLoginFromExcel(String username, String password, String expectedResult) {
        // username, password, expectedResult from Excel columns
        System.out.println("Testing: " + username + " | " + password + " | " + expectedResult);
    }

    // ===== Read from CSV =====

    @DataProvider(name = "csvData")
    public Object[][] readCSVData() throws IOException {
        List<Object[]> records = new ArrayList<>();
        BufferedReader br = new BufferedReader(new FileReader("D:\\TestData\\data.csv"));
        String line;
        boolean isHeader = true;

        while ((line = br.readLine()) != null) {
            if (isHeader) { isHeader = false; continue; } // skip header
            String[] values = line.split(",");
            records.add(values);
        }
        br.close();
        return records.toArray(new Object[0][]);
    }

    // ===== Read from JSON (using Jackson) =====

    public List<Map<String, String>> readJSONData(String filePath) throws IOException {
        ObjectMapper mapper = new ObjectMapper();
        return mapper.readValue(new File(filePath),
                new TypeReference<List<Map<String, String>>>() {});
    }

    // ===== Read from Properties file =====

    public static Properties readProperties(String filePath) throws IOException {
        Properties props = new Properties();
        FileInputStream fis = new FileInputStream(filePath);
        props.load(fis);
        fis.close();
        return props;
    }
}
```

---

## 15. Framework Design — Hybrid Framework

### Q: Design a complete Hybrid Framework architecture.

```
src/
├── main/java/
│   ├── base/
│   │   └── BaseTest.java          ← WebDriver setup/teardown
│   ├── pages/
│   │   ├── LoginPage.java         ← Page Objects
│   │   └── DashboardPage.java
│   ├── utils/
│   │   ├── DriverFactory.java     ← ThreadLocal WebDriver
│   │   ├── ExcelUtils.java        ← Data utilities
│   │   ├── ScreenshotUtils.java   ← Capture screenshots
│   │   ├── WaitUtils.java         ← Custom wait helpers
│   │   ├── ConfigReader.java      ← Read config.properties
│   │   └── ReportManager.java     ← ExtentReports
│   ├── listeners/
│   │   └── TestListener.java      ← ITestListener impl
│   └── constants/
│       └── Constants.java         ← Magic strings/numbers
├── test/java/
│   └── tests/
│       ├── LoginTest.java
│       └── SearchTest.java
├── resources/
│   ├── config.properties          ← env-specific config
│   ├── testng.xml
│   └── testdata/
│       └── LoginData.xlsx
└── pom.xml
```

```java
// ===== BaseTest.java =====
package base;

import org.openqa.selenium.WebDriver;
import org.testng.annotations.*;
import utils.DriverFactory;
import utils.ConfigReader;

public class BaseTest {

    protected WebDriver driver;

    @BeforeMethod(alwaysRun = true)
    @Parameters({"browser"})
    public void setUp(@Optional("chrome") String browser) {
        String browserFromConfig = ConfigReader.get("browser");
        String activeBrowser = (browser != null) ? browser : browserFromConfig;

        DriverFactory.setDriver(activeBrowser);
        driver = DriverFactory.getDriver();
        driver.get(ConfigReader.get("base.url"));
        driver.manage().window().maximize();
    }

    @AfterMethod(alwaysRun = true)
    public void tearDown() {
        DriverFactory.quitDriver();
    }
}

// ===== ConfigReader.java =====
package utils;

import java.io.*;
import java.util.Properties;

public class ConfigReader {

    private static Properties properties;

    static {
        try {
            // Load from resources
            String env = System.getProperty("env", "qa"); // -Denv=prod
            FileInputStream fis = new FileInputStream(
                "src/main/resources/" + env + ".config.properties");
            properties = new Properties();
            properties.load(fis);
        } catch (IOException e) {
            throw new RuntimeException("Config file not found: " + e.getMessage());
        }
    }

    public static String get(String key) {
        String value = System.getProperty(key); // check command line first
        if (value == null) {
            value = properties.getProperty(key);
        }
        if (value == null) {
            throw new RuntimeException("Property not found: " + key);
        }
        return value;
    }

    public static int getInt(String key) {
        return Integer.parseInt(get(key));
    }
}
```

---

## 16. Screenshot & Reporting (Extent Reports)

### Q: How do you capture screenshots and generate reports?

```java
// ===== ScreenshotUtils.java =====
package utils;

import org.openqa.selenium.*;
import org.apache.commons.io.FileUtils;
import java.io.*;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public class ScreenshotUtils {

    // Full page screenshot
    public static String captureScreenshot(WebDriver driver, String testName) {
        String timestamp = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyyMMdd_HHmmss"));
        String screenshotPath = "test-output/screenshots/" + testName + "_" + timestamp + ".png";

        File src = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);
        File dest = new File(screenshotPath);

        try {
            FileUtils.copyFile(src, dest);
        } catch (IOException e) {
            e.printStackTrace();
        }
        return screenshotPath;
    }

    // Screenshot as Base64 (for embedding in HTML reports)
    public static String captureBase64Screenshot(WebDriver driver) {
        return ((TakesScreenshot) driver).getScreenshotAs(OutputType.BASE64);
    }

    // Element screenshot (Selenium 4)
    public static void captureElementScreenshot(WebElement element, String filePath) throws IOException {
        File src = element.getScreenshotAs(OutputType.FILE);
        FileUtils.copyFile(src, new File(filePath));
    }
}

// ===== ExtentReportManager.java =====
package utils;

import com.aventstack.extentreports.*;
import com.aventstack.extentreports.reporter.*;
import com.aventstack.extentreports.reporter.configuration.*;

public class ExtentReportManager {

    private static ExtentReports extent;
    private static ThreadLocal<ExtentTest> test = new ThreadLocal<>();

    public static ExtentReports getInstance() {
        if (extent == null) {
            ExtentSparkReporter reporter = new ExtentSparkReporter("test-output/ExtentReport.html");
            reporter.config().setDocumentTitle("Test Automation Report");
            reporter.config().setReportName("Regression Suite");
            reporter.config().setTheme(Theme.DARK);
            reporter.config().setTimeStampFormat("dd-MM-yyyy HH:mm:ss");

            extent = new ExtentReports();
            extent.attachReporter(reporter);
            extent.setSystemInfo("OS", System.getProperty("os.name"));
            extent.setSystemInfo("Browser", ConfigReader.get("browser"));
            extent.setSystemInfo("Tester", "SDET Team");
        }
        return extent;
    }

    public static ExtentTest getTest() { return test.get(); }

    public static void setTest(ExtentTest extentTest) { test.set(extentTest); }

    public static void flushReport() {
        if (extent != null) extent.flush();
    }
}

// ===== TestListener.java =====
package listeners;

import com.aventstack.extentreports.*;
import org.openqa.selenium.WebDriver;
import org.testng.*;
import utils.*;

public class TestListener implements ITestListener {

    @Override
    public void onTestStart(ITestResult result) {
        ExtentTest test = ExtentReportManager.getInstance()
                .createTest(result.getMethod().getMethodName(),
                            result.getMethod().getDescription());
        ExtentReportManager.setTest(test);
        ExtentReportManager.getTest().log(Status.INFO, "Test started");
    }

    @Override
    public void onTestSuccess(ITestResult result) {
        ExtentReportManager.getTest().log(Status.PASS, "Test PASSED");
    }

    @Override
    public void onTestFailure(ITestResult result) {
        ExtentReportManager.getTest().log(Status.FAIL,
                "Test FAILED: " + result.getThrowable().getMessage());

        // Attach screenshot on failure
        WebDriver driver = DriverFactory.getDriver();
        if (driver != null) {
            String base64Screenshot = ScreenshotUtils.captureBase64Screenshot(driver);
            ExtentReportManager.getTest()
                .addScreenCaptureFromBase64String(base64Screenshot, "Failure Screenshot");
        }
    }

    @Override
    public void onTestSkipped(ITestResult result) {
        ExtentReportManager.getTest().log(Status.SKIP, "Test SKIPPED");
    }

    @Override
    public void onFinish(ITestContext context) {
        ExtentReportManager.flushReport();
    }
}
```

---

## 17. Parallel Execution

### Q: How do you run tests in parallel? How do you handle thread safety?

```xml
<!-- testng.xml — Parallel at test/class/method level -->
<suite name="ParallelSuite" parallel="methods" thread-count="5">
    <test name="RegressionTests">
        <classes>
            <class name="tests.LoginTest"/>
            <class name="tests.SearchTest"/>
        </classes>
    </test>
</suite>

<!-- parallel="classes" → each class in separate thread -->
<!-- parallel="methods" → each test method in separate thread -->
<!-- parallel="tests"   → each <test> tag in separate thread -->
```

```java
// ⭐ KEY: Use ThreadLocal for WebDriver — one instance per thread
public class DriverFactory {
    private static ThreadLocal<WebDriver> driverThread = new ThreadLocal<>();
    // ... (shown earlier)
}

// ⭐ KEY: Don't use instance variables in test classes for shared state
public class LoginTest extends BaseTest {

    // ❌ WRONG — shared across threads
    private LoginPage loginPage;

    // ✅ CORRECT — use local variables or ThreadLocal
    @Test
    public void testLogin() {
        LoginPage loginPage = new LoginPage(driver); // local variable
        loginPage.login("user", "pass");
    }
}

// Maven parallel execution (Surefire plugin)
// pom.xml snippet:
/*
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <configuration>
        <suiteXmlFiles>
            <suiteXmlFile>src/test/resources/testng.xml</suiteXmlFile>
        </suiteXmlFiles>
        <parallel>methods</parallel>
        <threadCount>5</threadCount>
    </configuration>
</plugin>
*/
```

---

## 18. Headless Browser Testing

### Q: How do you run tests in headless mode? When to use it?

```java
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.firefox.FirefoxOptions;

public class HeadlessSetup {

    // Chrome Headless
    public WebDriver getChromeHeadless() {
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--headless=new"); // "new" headless mode (Chrome 112+)
        options.addArguments("--window-size=1920,1080"); // must set size in headless
        options.addArguments("--disable-gpu");
        options.addArguments("--no-sandbox");
        options.addArguments("--disable-dev-shm-usage");
        // Required for Docker/CI
        return new ChromeDriver(options);
    }

    // Firefox Headless
    public WebDriver getFirefoxHeadless() {
        FirefoxOptions options = new FirefoxOptions();
        options.addArguments("--headless");
        options.addArguments("--width=1920");
        options.addArguments("--height=1080");
        return new FirefoxDriver(options);
    }

    // ⭐ Headless Use Cases:
    // - CI/CD pipelines (no display server)
    // - Faster execution (no UI rendering)
    // - Running in Docker containers
    // - Server-side automation

    // ⭐ Headless Limitations:
    // - File upload dialogs may not work (use sendKeys method)
    // - Print dialogs may behave differently
    // - Some animations/transitions may differ
    // - Harder to debug (no visual feedback)
}
```

---

## 19. Grid & Remote WebDriver

### Q: How do you set up Selenium Grid and run tests remotely?

```java
import org.openqa.selenium.remote.*;
import java.net.URL;

public class GridSetup {

    // Connect to Selenium Grid Hub
    public WebDriver getRemoteDriver(String browser) throws Exception {
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--start-maximized");

        // Selenium 4 Grid — simplified URL format
        URL gridUrl = new URL("http://localhost:4444");
        return new RemoteWebDriver(gridUrl, options);
    }

    // With custom capabilities
    public WebDriver getRemoteWithCaps() throws Exception {
        ChromeOptions options = new ChromeOptions();
        options.setCapability("se:name", "My Test"); // Grid metadata
        options.setCapability("se:sessionTimeout", "300");

        return new RemoteWebDriver(new URL("http://selenium-hub:4444"), options);
    }

    // BrowserStack Remote
    public WebDriver getBrowserStack() throws Exception {
        ChromeOptions options = new ChromeOptions();
        HashMap<String, Object> bsOptions = new HashMap<>();
        bsOptions.put("os", "Windows");
        bsOptions.put("osVersion", "10");
        bsOptions.put("browserVersion", "latest");
        bsOptions.put("buildName", "Regression Build #1");
        bsOptions.put("sessionName", "Login Test");
        bsOptions.put("userName", "YOUR_USERNAME");
        bsOptions.put("accessKey", "YOUR_ACCESS_KEY");
        options.setCapability("bstack:options", bsOptions);

        return new RemoteWebDriver(
            new URL("https://hub.browserstack.com/wd/hub"), options);
    }
}
```

---

## 20. Handling Dynamic Elements & AJAX

### Q: How do you handle dynamic elements, AJAX calls, loading spinners?

```java
public class DynamicElementHandling {

    WebDriver driver;

    // 1. Wait for AJAX to complete
    public void waitForAjax() {
        new WebDriverWait(driver, Duration.ofSeconds(30)).until(driver -> {
            JavascriptExecutor js = (JavascriptExecutor) driver;
            return (Boolean) js.executeScript("return jQuery.active == 0");
        });
    }

    // 2. Wait for loading spinner to disappear
    public void waitForSpinnerToDisappear() {
        By spinner = By.cssSelector(".loading-spinner");
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(30));
        // Wait for spinner to appear (optional, ensures it started loading)
        try {
            wait.until(ExpectedConditions.visibilityOfElementLocated(spinner));
        } catch (TimeoutException ignored) {} // spinner may be too fast

        // Wait for spinner to disappear
        wait.until(ExpectedConditions.invisibilityOfElementLocated(spinner));
    }

    // 3. Dynamic table — find row by content
    public WebElement findTableRowByText(String targetText) {
        List<WebElement> rows = driver.findElements(By.cssSelector("table tbody tr"));
        for (WebElement row : rows) {
            if (row.getText().contains(targetText)) {
                return row;
            }
        }
        throw new NoSuchElementException("Row with text '" + targetText + "' not found");
    }

    // 4. Dynamic XPath with variable
    public WebElement findRowByDynamicXPath(String productName) {
        return driver.findElement(
            By.xpath("//td[text()='" + productName + "']/following-sibling::td/button[@class='edit']"));
    }

    // 5. Infinite scroll / lazy loading
    public void scrollAndLoad(int maxScrolls) throws InterruptedException {
        JavascriptExecutor js = (JavascriptExecutor) driver;
        long previousHeight = (long) js.executeScript("return document.body.scrollHeight");

        for (int i = 0; i < maxScrolls; i++) {
            js.executeScript("window.scrollTo(0, document.body.scrollHeight)");
            Thread.sleep(2000); // wait for content to load
            long newHeight = (long) js.executeScript("return document.body.scrollHeight");
            if (newHeight == previousHeight) break; // no more content
            previousHeight = newHeight;
        }
    }

    // 6. Wait for element count to change
    public void waitForMoreItemsToLoad(int expectedCount) {
        new WebDriverWait(driver, Duration.ofSeconds(30)).until(
            ExpectedConditions.numberOfElementsToBeMoreThan(
                By.cssSelector(".product-card"), expectedCount - 1));
    }
}
```

---

## 21. Stale Element Reference Exception

### Q: What is StaleElementReferenceException and how do you handle it?

```java
public class StaleElementHandling {

    WebDriver driver;

    // ⭐ Cause: Element was found, then DOM changed (AJAX reload, re-render, navigation)
    // The reference is now "stale" (pointing to deleted DOM node)

    // ❌ BAD: Leads to StaleElementReferenceException
    public void badExample() {
        WebElement button = driver.findElement(By.id("dynamicBtn"));
        driver.findElement(By.id("trigger")).click(); // causes DOM reload
        button.click(); // STALE! button reference is dead
    }

    // ✅ Method 1: Re-find element every time
    public void goodExample() {
        driver.findElement(By.id("trigger")).click();
        // Re-find after DOM change
        driver.findElement(By.id("dynamicBtn")).click();
    }

    // ✅ Method 2: Retry mechanism
    public void clickWithRetry(By locator, int maxRetries) {
        for (int i = 0; i < maxRetries; i++) {
            try {
                driver.findElement(locator).click();
                return; // success
            } catch (StaleElementReferenceException e) {
                if (i == maxRetries - 1) throw e;
                System.out.println("Stale element, retrying... attempt " + (i + 1));
            }
        }
    }

    // ✅ Method 3: Use ExpectedConditions.refreshed()
    public void clickRefreshed(By locator) {
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        WebElement element = wait.until(ExpectedConditions.refreshed(
                ExpectedConditions.elementToBeClickable(locator)));
        element.click();
    }

    // ✅ Method 4: Fluent wait with stale exception ignored
    public WebElement findWithFluentWait(By locator) {
        return new FluentWait<>(driver)
                .withTimeout(Duration.ofSeconds(20))
                .pollingEvery(Duration.ofMillis(300))
                .ignoring(StaleElementReferenceException.class)
                .until(d -> d.findElement(locator));
    }

    // ⭐ Avoid @CacheLookup in PageFactory for dynamic elements
}
```

---

## 22. Cross-Browser Testing Edge Cases

### Q: What are common cross-browser issues and how do you handle them?

```java
public class CrossBrowserHandling {

    // 1. CSS differences
    // - Firefox renders fonts differently
    // - IE/Edge may have different scrollbar behavior
    // - Safari strict same-origin policy

    // 2. JavaScript execution differences
    // Handle with try-catch or feature detection

    // 3. WebElement interaction differences
    public void crossBrowserClick(WebDriver driver, WebElement element) {
        try {
            element.click(); // normal click
        } catch (ElementClickInterceptedException e) {
            // Try JS click as fallback
            ((JavascriptExecutor) driver).executeScript("arguments[0].click();", element);
        }
    }

    // 4. sendKeys differences
    public void crossBrowserSendKeys(WebDriver driver, WebElement element, String text) {
        element.clear();
        // Some browsers need JS to set value
        try {
            element.sendKeys(text);
        } catch (Exception e) {
            ((JavascriptExecutor) driver).executeScript(
                "arguments[0].value='" + text + "';", element);
        }
    }

    // 5. Date input handling across browsers
    public void handleDateInput(WebDriver driver, WebElement dateInput, String date) {
        String browser = ((RemoteWebDriver) driver).getCapabilities().getBrowserName();

        if (browser.equalsIgnoreCase("chrome")) {
            dateInput.sendKeys(date); // MM/DD/YYYY
        } else if (browser.equalsIgnoreCase("firefox")) {
            dateInput.sendKeys(date); // YYYY-MM-DD
        } else {
            // Fallback: use JS
            ((JavascriptExecutor) driver).executeScript(
                "arguments[0].value='" + date + "';", dateInput);
        }
    }

    // 6. Screenshot on cross-browser failure
    public void takeCrossBrowserScreenshot(WebDriver driver, String testName) {
        String browser = ((RemoteWebDriver) driver).getCapabilities().getBrowserName();
        ScreenshotUtils.captureScreenshot(driver, testName + "_" + browser);
    }
}
```

---

## 23. REST API Testing with RestAssured

### Q: How do you integrate API testing in your Selenium framework?

```java
import io.restassured.RestAssured;
import io.restassured.response.Response;
import io.restassured.specification.RequestSpecification;
import static io.restassured.RestAssured.*;
import static org.hamcrest.Matchers.*;

public class APITestingWithRestAssured {

    // Base setup
    public void setup() {
        RestAssured.baseURI = "https://api.example.com";
        RestAssured.basePath = "/v1";
    }

    // GET request
    public void testGetRequest() {
        given()
            .header("Authorization", "Bearer " + getAuthToken())
            .queryParam("page", 1)
            .queryParam("size", 10)
        .when()
            .get("/users")
        .then()
            .statusCode(200)
            .body("data", hasSize(10))
            .body("data[0].name", notNullValue())
            .time(lessThan(2000L)); // response time < 2s
    }

    // POST request
    public Response testPostRequest() {
        String requestBody = """
            {
                "name": "Test User",
                "email": "test@example.com",
                "role": "user"
            }
            """;

        return given()
            .contentType("application/json")
            .header("Authorization", "Bearer " + getAuthToken())
            .body(requestBody)
        .when()
            .post("/users")
        .then()
            .statusCode(201)
            .body("id", notNullValue())
            .body("name", equalTo("Test User"))
            .extract().response();
    }

    // Extract value from response
    public String extractUserId() {
        Response response = testPostRequest();
        return response.jsonPath().getString("id");
    }

    // ⭐ API + UI Integration: Create test data via API, then test UI
    public void createUserViaApiThenTestUI() {
        // Step 1: Create user via API (fast)
        String userId = extractUserId();

        // Step 2: Login to UI and verify user exists
        driver.get("https://app.example.com/login");
        new LoginPage(driver).login("admin", "Admin@123");
        driver.get("https://app.example.com/users/" + userId);
        Assert.assertEquals(driver.findElement(By.id("userName")).getText(), "Test User");
    }

    // Auth token helper
    public String getAuthToken() {
        return given()
            .contentType("application/json")
            .body("{\"username\":\"admin\",\"password\":\"Admin@123\"}")
        .when()
            .post("/auth/login")
        .then()
            .statusCode(200)
            .extract()
            .path("token");
    }

    // Request specification (reusable)
    public RequestSpecification getAuthSpec() {
        return given()
            .baseUri("https://api.example.com")
            .contentType("application/json")
            .header("Authorization", "Bearer " + getAuthToken());
    }
}
```

---

## 24. CI/CD — Jenkins Integration

### Q: How do you integrate Selenium tests with Jenkins?

```groovy
// Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any

    parameters {
        choice(name: 'BROWSER', choices: ['chrome', 'firefox', 'edge'], description: 'Browser')
        choice(name: 'ENV', choices: ['qa', 'staging', 'prod'], description: 'Environment')
        string(name: 'SUITE', defaultValue: 'testng.xml', description: 'Test suite file')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/org/selenium-framework.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean compile -q'
            }
        }

        stage('Run Tests') {
            steps {
                sh """
                    mvn test \
                        -Dbrowser=${params.BROWSER} \
                        -Denv=${params.ENV} \
                        -DsuiteXmlFile=src/test/resources/${params.SUITE} \
                        -Dheadless=true
                """
            }
            post {
                always {
                    // Publish TestNG results
                    testNG reportFilenamePattern: '**/testng-results.xml'
                    // Publish Extent Report as artifact
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'test-output',
                        reportFiles: 'ExtentReport.html',
                        reportName: 'Extent HTML Report'
                    ])
                    // Archive screenshots
                    archiveArtifacts artifacts: 'test-output/screenshots/**', allowEmptyArchive: true
                }
            }
        }
    }

    post {
        failure {
            emailext (
                subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Check console output: ${env.BUILD_URL}",
                to: 'team@company.com'
            )
        }
    }
}
```

---

## 25. Design Patterns in Test Automation

### Q: What design patterns do you use in automation? Explain each.

```java
// ===== 1. SINGLETON — One WebDriver instance =====
public class DriverSingleton {
    private static WebDriver instance;

    private DriverSingleton() {}

    public static synchronized WebDriver getInstance() {
        if (instance == null) {
            instance = new ChromeDriver();
        }
        return instance;
    }
    // Note: Use ThreadLocal for parallel execution instead
}

// ===== 2. FACTORY — Create correct driver type =====
public interface BrowserFactory {
    WebDriver createDriver();
}

public class ChromeFactory implements BrowserFactory {
    public WebDriver createDriver() {
        ChromeOptions opts = new ChromeOptions();
        opts.addArguments("--start-maximized");
        return new ChromeDriver(opts);
    }
}

public class FirefoxFactory implements BrowserFactory {
    public WebDriver createDriver() { return new FirefoxDriver(); }
}

public class DriverManager {
    public static WebDriver createDriver(String browser) {
        Map<String, BrowserFactory> factories = Map.of(
            "chrome", new ChromeFactory(),
            "firefox", new FirefoxFactory()
        );
        return factories.getOrDefault(browser, new ChromeFactory()).createDriver();
    }
}

// ===== 3. BUILDER — Build complex test data =====
public class User {
    private String username, password, email, role;

    private User() {}

    public static class Builder {
        private User user = new User();

        public Builder username(String username) { user.username = username; return this; }
        public Builder password(String password) { user.password = password; return this; }
        public Builder email(String email) { user.email = email; return this; }
        public Builder role(String role) { user.role = role; return this; }
        public User build() { return user; }
    }
}

// Usage:
User testUser = new User.Builder()
    .username("testuser")
    .password("Test@123")
    .email("test@example.com")
    .role("admin")
    .build();

// ===== 4. STRATEGY — Interchangeable wait strategies =====
public interface WaitStrategy {
    WebElement waitForElement(WebDriver driver, By locator);
}

public class VisibilityWait implements WaitStrategy {
    public WebElement waitForElement(WebDriver driver, By locator) {
        return new WebDriverWait(driver, Duration.ofSeconds(10))
            .until(ExpectedConditions.visibilityOfElementLocated(locator));
    }
}

public class ClickabilityWait implements WaitStrategy {
    public WebElement waitForElement(WebDriver driver, By locator) {
        return new WebDriverWait(driver, Duration.ofSeconds(10))
            .until(ExpectedConditions.elementToBeClickable(locator));
    }
}

// ===== 5. DECORATOR — Add behavior to WebElement =====
public class HighlightingWebElement implements WebElement {
    private final WebElement element;
    private final WebDriver driver;

    public HighlightingWebElement(WebElement element, WebDriver driver) {
        this.element = element;
        this.driver = driver;
    }

    @Override
    public void click() {
        ((JavascriptExecutor) driver).executeScript(
            "arguments[0].style.border='3px solid red'", element);
        element.click();
    }

    // delegate all other methods to element...
    @Override public void sendKeys(CharSequence... keysToSend) { element.sendKeys(keysToSend); }
    // ... etc
}
```

---

## 26. Senior-Level Tricky Questions

### Q: What happens when you call `driver.findElement()` vs `driver.findElements()`?

```java
// findElement() — throws NoSuchElementException if not found
// findElements() — returns EMPTY LIST if not found (never throws)

// ✅ Use findElements to check element existence safely
public boolean isElementPresent(By locator) {
    return !driver.findElements(locator).isEmpty();
}

// ✅ Use findElements to get optional element
public Optional<WebElement> findOptional(By locator) {
    List<WebElement> elements = driver.findElements(locator);
    return elements.isEmpty() ? Optional.empty() : Optional.of(elements.get(0));
}
```

### Q: Difference between `close()` and `quit()`?

```java
// driver.close() → closes CURRENT browser window/tab only
// driver.quit()  → closes ALL browser windows + ends WebDriver session

// ⭐ Always use quit() in @AfterMethod
// close() can leave browser processes running → memory leak
// quit() also kills browser driver process (chromedriver.exe)
```

### Q: How do you handle an element that is visible but not clickable?

```java
public void handleElementNotClickable(By locator) {
    WebDriver driver = DriverFactory.getDriver();
    WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));

    // 1. Wait for clickable
    WebElement element = wait.until(ExpectedConditions.elementToBeClickable(locator));

    // 2. Scroll into view
    ((JavascriptExecutor) driver).executeScript("arguments[0].scrollIntoView(true);", element);

    // 3. Wait a bit after scroll
    try { Thread.sleep(300); } catch (InterruptedException ignored) {}

    // 4. Try click
    try {
        element.click();
    } catch (ElementClickInterceptedException e) {
        // Something is covering it — try JS click
        ((JavascriptExecutor) driver).executeScript("arguments[0].click();", element);
    }
}
```

### Q: How do you verify a broken image?

```java
public boolean isImageBroken(WebElement image) {
    return (Boolean) ((JavascriptExecutor) driver).executeScript(
        "return arguments[0].complete && " +
        "typeof arguments[0].naturalWidth != 'undefined' && " +
        "arguments[0].naturalWidth > 0;",
        image);
}

@Test
public void testAllImages() {
    List<WebElement> images = driver.findElements(By.tagName("img"));
    List<String> brokenImages = new ArrayList<>();

    for (WebElement img : images) {
        if (!isImageBroken(img)) {
            brokenImages.add(img.getAttribute("src"));
        }
    }

    Assert.assertTrue(brokenImages.isEmpty(),
            "Broken images found: " + String.join(", ", brokenImages));
}
```

### Q: How do you get all links and verify they return HTTP 200?

```java
import java.net.*;
import java.io.*;

@Test
public void testAllLinks() throws Exception {
    List<WebElement> links = driver.findElements(By.tagName("a"));
    List<String> brokenLinks = new ArrayList<>();

    for (WebElement link : links) {
        String url = link.getAttribute("href");
        if (url == null || url.isEmpty() || url.startsWith("mailto:") || url.startsWith("javascript:")) {
            continue;
        }

        HttpURLConnection conn = (HttpURLConnection) new URL(url).openConnection();
        conn.setRequestMethod("HEAD");
        conn.setConnectTimeout(3000);
        conn.setReadTimeout(3000);

        try {
            conn.connect();
            int responseCode = conn.getResponseCode();
            if (responseCode >= 400) {
                brokenLinks.add(url + " → " + responseCode);
            }
        } catch (Exception e) {
            brokenLinks.add(url + " → ERROR: " + e.getMessage());
        } finally {
            conn.disconnect();
        }
    }

    Assert.assertTrue(brokenLinks.isEmpty(), "Broken links: " + brokenLinks);
}
```

### Q: How do you handle Calendar/Date Picker?

```java
public void selectDateFromDatePicker(String targetDate) {
    // targetDate format: "15 September 2025"
    String[] parts = targetDate.split(" ");
    String day = parts[0], month = parts[1], year = parts[2];

    // Click date picker
    driver.findElement(By.id("datepicker")).click();

    // Navigate to correct year
    while (!driver.findElement(By.cssSelector(".year-display")).getText().equals(year)) {
        driver.findElement(By.cssSelector(".next-year")).click();
    }

    // Navigate to correct month
    while (!driver.findElement(By.cssSelector(".month-display")).getText().equals(month)) {
        driver.findElement(By.cssSelector(".next-month")).click();
    }

    // Click the day
    driver.findElement(By.xpath("//td[@data-day='" + day + "']")).click();
}
```

### Q: How do you handle cookies?

```java
public class CookieHandling {

    // Add cookie (for session bypass)
    public void addCookie(WebDriver driver, String name, String value) {
        Cookie cookie = new Cookie.Builder(name, value)
                .domain(".example.com")
                .path("/")
                .isHttpOnly(true)
                .isSecure(true)
                .build();
        driver.manage().addCookie(cookie);
        driver.navigate().refresh(); // apply cookie
    }

    // Get cookie
    public String getCookieValue(WebDriver driver, String name) {
        Cookie cookie = driver.manage().getCookieNamed(name);
        return (cookie != null) ? cookie.getValue() : null;
    }

    // Delete cookies
    public void clearCookies(WebDriver driver) {
        driver.manage().deleteAllCookies();
    }

    // ⭐ Login via cookie (skip UI login — speeds up tests significantly)
    public void loginViaCookie(WebDriver driver, String authToken) {
        driver.get("https://www.example.com");
        driver.manage().addCookie(new Cookie("auth_token", authToken));
        driver.navigate().refresh();
        // Now navigated to logged-in state
    }
}
```

### Q: What is the difference between `getText()` and `getAttribute("value")`?

```java
// getText()            → gets visible text content of element
// getAttribute("value") → gets "value" attribute (for input fields)
// getAttribute("innerHTML") → HTML inside element
// getAttribute("textContent") → all text (including hidden)

// Example:
WebElement input = driver.findElement(By.id("name"));
input.sendKeys("John");
System.out.println(input.getText()); // "" (empty — input has no text content)
System.out.println(input.getAttribute("value")); // "John" ✅

WebElement div = driver.findElement(By.id("message"));
System.out.println(div.getText()); // "Hello World"
System.out.println(div.getAttribute("value")); // null (div has no value attribute)
```

### Q: How do you perform visual testing / pixel comparison?

```java
// Using Ashot library for screenshot comparison
import ru.yandex.qatools.ashot.*;
import ru.yandex.qatools.ashot.comparison.*;
import ru.yandex.qatools.ashot.shooting.*;

public void visualTest(WebDriver driver) throws IOException {
    Screenshot screenshot = new AShot()
        .shootingStrategy(ShootingStrategies.viewportPasting(1000))
        .takeScreenshot(driver);

    // Compare with baseline
    BufferedImage baseline = ImageIO.read(new File("baseline.png"));
    ImageDiff diff = new ImageDiffer().makeDiff(screenshot.getImage(), baseline);

    if (diff.hasDiff()) {
        // Save diff image
        ImageIO.write(diff.getMarkedImage(), "PNG", new File("diff.png"));
        Assert.fail("Visual regression found! See diff.png");
    }
}
```

---

## 🎯 Most Asked Interview Questions Summary

| Level | Questions |
|---|---|
| **Junior** | Locators, Waits, Alerts, Dropdowns, basic POM |
| **Mid** | Frames/Windows, File upload, Actions, JS Executor, TestNG |
| **Senior** | ThreadLocal, Framework Design, Parallel, Grid, CI/CD |
| **Expert** | Design Patterns, Visual Testing, CDP, Performance, Custom ExpectedConditions |

## 💡 Key Things Interviewers Look For at 24 LPA

1. **Framework thinking** — not just test scripts
2. **Parallel execution** + **ThreadLocal** understanding
3. **Design patterns** — Factory, Builder, POM, Singleton
4. **CI/CD integration** — Jenkins, GitHub Actions
5. **Debugging skills** — Stale element, waits, timeouts
6. **API + UI testing** combination
7. **Code quality** — reusable, maintainable, readable
8. **Reporting** — ExtentReports, Allure
9. **Git** knowledge — branching, PR strategies
10. **Problem-solving** — edge case handling

---

> 💬 **Pro Tip**: Always explain the **WHY** not just the **WHAT** in interviews. "I use ThreadLocal because..." shows deeper understanding.
