# PART 12: ADVANCED SELENIUM

### 1. Shadow DOM
**Q: What is Shadow DOM, and how do you handle it in Selenium?** 🔥
**A:** "In my project, we started seeing this modern web architecture where developers use Web Components. Shadow DOM is essentially a 'DOM within a DOM' – it's an encapsulated, hidden DOM tree attached to a regular DOM element. Developers use it to keep their component's CSS and structure separate from the main page, preventing styling conflicts.

The tricky part? Standard Selenium locators like XPath or CSS Selectors simply cannot pierce through this shadow boundary. If you try to do `driver.findElement(By.id("my-btn"))` and it's inside a shadow root, Selenium will throw a `NoSuchElementException` because it's completely isolated.

In older Selenium versions, we had to heavily rely on JavaScript execution to fetch elements from the shadow DOM. But since Selenium 4, they've introduced native support which is fantastic. 

**How we handle it now (Selenium 4):**
What we typically do is first locate the *Shadow Host* (the regular element that hosts the shadow tree), get its shadow root, and then locate the element inside it using CSS Selectors (XPath still doesn't work inside Shadow DOM).

```java
// 1. Locate the shadow host
WebElement shadowHost = driver.findElement(By.id("shadow-host-id"));

// 2. Get the Shadow Root
SearchContext shadowRoot = shadowHost.getShadowRoot();

// 3. Find the element INSIDE the shadow root using CSS Selector
WebElement shadowElement = shadowRoot.findElement(By.cssSelector(".shadow-btn-class"));

// Now interact with it
shadowElement.click();
```
*Edge Case:* If the shadow root is 'closed' (`mode: closed`), even `getShadowRoot()` won't work easily, but in 99% of enterprise apps, they use 'open' mode."

### 2. SVG Elements
**Q: How do you handle SVG (Scalable Vector Graphics) elements in Selenium?** 🔥
**A:** "SVG elements are a classic pain point in automation. In my current project, our analytics dashboard is full of graphs and custom icons drawn using SVGs. 

The issue is that standard XPath like `//svg` or `//path` doesn't work because SVG elements are in a different XML namespace compared to standard HTML elements. If you try normal XPath, Selenium won't find them.

**From my experience, the standard way to locate SVGs is using the `local-name()` or `name()` functions in XPath.**

Let's say you have a `<svg>` tag with a class 'chart-icon':
```java
// WRONG WAY - Will throw NoSuchElementException
// driver.findElement(By.xpath("//svg[@class='chart-icon']"));

// RIGHT WAY - Using local-name()
WebElement svgIcon = driver.findElement(By.xpath("//*[local-name()='svg' and @class='chart-icon']"));
svgIcon.click();

// If you need to click a specific <path> inside the SVG
WebElement svgPath = driver.findElement(By.xpath("//*[local-name()='svg']/*[name()='path' and @id='series-1']"));
```
*Best Practice:* Whenever I see an icon or chart, I immediately check if it's an SVG. If it is, I default to this `local-name()` approach. CSS selectors also work fine (`css="svg.chart-icon"`), but for complex DOM traversal around SVGs, XPath with `local-name()` is our go-to."

### 3. Dynamic DOM / Stale Elements
**Q: How do you handle Dynamic DOM where elements become stale?** 🔥
**A:** "This is probably the most frequent issue in modern Single Page Applications (SPAs) like React or Angular. 

**Dynamic DOM** means the structure of the webpage is constantly updating without a full page reload. An element is created, destroyed, and recreated dynamically.

When you find an element, Selenium gets a reference ID for it. But if the page's JavaScript updates that part of the DOM, the old element is destroyed and a visually identical one is created. Your old reference is now dead. If you try to click it, Selenium throws a `StaleElementReferenceException` because the element is 'stale' (no longer attached to the current DOM).

**The way I handle this is usually a mix of three strategies:**

1. **Re-find the element right before using it:**
Don't find elements and store them for long periods. Find them immediately before the action.
```java
// Instead of storing it early:
// WebElement btn = driver.findElement(By.id("submit"));
// ... do 10 other things ...
// btn.click(); // Might be stale!

// Do this:
driver.findElement(By.id("submit")).click();
```

2. **Use Explicit Waits (ExpectedConditions.refreshed):**
If I know a DOM update is happening, I tell the wait to expect the element to become stale and refreshed.
```java
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
WebElement dynamicElement = wait.until(ExpectedConditions.refreshed(
    ExpectedConditions.elementToBeClickable(By.id("dynamic-btn"))
));
dynamicElement.click();
```

3. **The Try-Catch Retry Loop (The bulletproof enterprise way):**
In our framework, we have a custom click method that retries if it hits a stale exception.
```java
public void robustClick(By locator) {
    int attempts = 0;
    while(attempts < 3) {
        try {
            driver.findElement(locator).click();
            break; // Success, exit loop
        } catch (StaleElementReferenceException e) {
            attempts++;
            System.out.println("Element stale, retrying... Attempt: " + attempts);
            // Optional brief sleep here
        }
    }
}
```
"

### 4. Element Interception / Overlays
**Q: What is ElementClickInterceptedException and how do you resolve it?** 🔥
**A:** "In almost every e-commerce or modern app I've tested, we get the `ElementClickInterceptedException`. This happens when Selenium tries to click element A, but element B is physically floating or layered on top of it.

**Common Culprits:**
- Cookie consent banners at the bottom.
- 'Loading...' spinners or overlays that block the whole screen.
- Sticky headers/footers.
- Chatbots hovering over the submit button.

Selenium tries to click the precise center of your target element. If a spinner is over it, Selenium hits the spinner, fails, and throws the exception.

**How I solve it:**

1. **Wait for the overlay to disappear (Best Practice):**
If it's a loading spinner, wait for it to be invisible.
```java
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
// Wait for the spinner overlay to vanish
wait.until(ExpectedConditions.invisibilityOfElementLocated(By.className("loading-overlay")));
// Now click the target
driver.findElement(By.id("submit-btn")).click();
```

2. **Scroll the element into view:**
If a sticky header is covering it, scrolling it to the middle of the viewport usually fixes it.
```java
WebElement element = driver.findElement(By.id("submit-btn"));
((JavascriptExecutor) driver).executeScript("arguments[0].scrollIntoView({block: 'center'});", element);
element.click();
```

3. **JavaScript Click (The fallback):**
If the overlay is something trivial or buggy, a JS click bypasses the UI layer entirely and fires the click event at the DOM level. I only use this if standard clicks consistently fail due to UI quirks.
```java
WebElement element = driver.findElement(By.id("submit-btn"));
((JavascriptExecutor) driver).executeScript("arguments[0].click();", element);
```
"

### 5. AJAX Applications
**Q: How do you automate AJAX-heavy applications?**
**A:** "AJAX stands for Asynchronous JavaScript and XML. It allows a web page to fetch data from the server in the background and update parts of the page without doing a full page refresh.

**The Challenge:**
Selenium's default `driver.get()` or click actions usually wait for the browser's `document.readyState` to be 'complete'. But with AJAX, the browser *thinks* it's done loading, while background calls are still fetching data. If you try to find an element before the AJAX call finishes rendering it, you get a `NoSuchElementException`.

**The Solution:**
We never rely on implicit waits or `Thread.sleep()` for AJAX. The only reliable way is to use **Explicit Waits** tied to specific conditions.

```java
// Example: Clicking a filter triggers an AJAX call to load new products
driver.findElement(By.id("category-shoes")).click();

// Wait explicitly for the AJAX-loaded elements to be present/visible
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(15));
WebElement firstShoeProduct = wait.until(
    ExpectedConditions.visibilityOfElementLocated(By.xpath("//div[@class='product-card']"))
);

System.out.println("AJAX content loaded: " + firstShoeProduct.getText());
```
*Pro-tip:* Sometimes, we also execute a JS script to check if jQuery/AJAX calls are active (`return jQuery.active == 0`), but explicit waits on the actual UI elements are usually much more reliable."

### 6. Lazy Loading
**Q: Have you handled Lazy Loading in your automation?**
**A:** "Yes, absolutely. Lazy loading is an optimization technique where images or content are only loaded when they enter the user's viewport (when you scroll down to them). It saves bandwidth.

If you try to assert an image or interact with an element at the bottom of a lazy-loaded page without scrolling, it simply won't be in the DOM, or its `src` attribute will be a placeholder.

**My Strategy:**
I use JavaScript to smoothly scroll down the page, wait a moment for the network to fetch the lazy chunk, and then locate the element.

```java
// Scroll down by a specific pixel amount or to the bottom
JavascriptExecutor js = (JavascriptExecutor) driver;

// Smoothly scroll down the page in increments
for(int i = 0; i < 5; i++) {
    js.executeScript("window.scrollBy(0, 500);");
    try {
        Thread.sleep(1000); // Give time for lazy load to trigger and render
    } catch(Exception e) {}
}

// Now wait for the specific lazy-loaded element
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
WebElement footerImage = wait.until(
    ExpectedConditions.visibilityOfElementLocated(By.id("heavy-footer-image"))
);
```
"

### 7. Infinite Scrolling
**Q: How do you test infinite scrolling functionality?**
**A:** "Infinite scrolling is common in social media feeds or product listing pages. There's no pagination button; as you scroll to the bottom, it makes an API call and appends more items.

To test this, we need a `while` loop that keeps scrolling until a certain condition is met—either we find our target element, or we reach the true end of the page.

**The implementation:**
```java
JavascriptExecutor js = (JavascriptExecutor) driver;
boolean isElementFound = false;
int maxScrolls = 10; // Failsafe to prevent infinite loops
int scrollCount = 0;

while(!isElementFound && scrollCount < maxScrolls) {
    try {
        // Try to find the target element
        WebElement target = driver.findElement(By.xpath("//h2[text()='Target Post']"));
        isElementFound = true;
        System.out.println("Element found!");
        break;
    } catch (NoSuchElementException e) {
        // If not found, scroll to the bottom of the page
        js.executeScript("window.scrollTo(0, document.body.scrollHeight);");
        
        // Wait for the loader to appear and disappear, or just wait for network
        try { Thread.sleep(2000); } catch(Exception ignored) {}
        
        scrollCount++;
    }
}

if(!isElementFound) {
    System.out.println("Element not found after " + maxScrolls + " scrolls.");
}
```
"

### 8. Hidden Elements
**Q: How do you verify and interact with hidden elements?** 🔥
**A:** "Hidden elements exist in the DOM but are invisible on the screen. This is usually done via CSS like `display: none;`, `visibility: hidden;`, or `opacity: 0;`.

**The behavior:**
- `driver.findElement()` WILL find the element (it's in the DOM).
- `element.isDisplayed()` will return `false`.
- If you try to `click()` or `sendKeys()`, Selenium throws an `ElementNotInteractableException` because a real user couldn't interact with an invisible element.

**How to handle them:**
1. **Fetching hidden text:** Standard `element.getText()` returns empty for hidden elements. To get the text, I use `getAttribute("textContent")`.
```java
WebElement hiddenElement = driver.findElement(By.id("secret-token"));
// This gets the text even if display:none
String text = hiddenElement.getAttribute("textContent"); 
```

2. **Interacting (Clicking/Typing):** If I *must* interact with a hidden element (e.g., a hidden file input for uploading, or a custom styled dropdown), I bypass Selenium's visibility check using JavaScriptExecutor.
```java
WebElement hiddenBtn = driver.findElement(By.id("hidden-submit"));
JavascriptExecutor js = (JavascriptExecutor) driver;
js.executeScript("arguments[0].click();", hiddenBtn);
```
"

### 9. Dynamic Attributes
**Q: How do you automate elements where IDs or classes change dynamically on every page load?** 🔥
**A:** "In modern frameworks like React, ExtJS, or Salesforce, developers use auto-generated IDs. You'll see an ID like `ext-gen-1024` on one load, and `ext-gen-3089` on the next. You absolutely cannot use standard `By.id` here.

**Strategies I use in my projects:**

1. **XPath `contains()` or `starts-with()`:**
If the ID has a static prefix, I match that part.
```java
// ID is like 'submit-btn-93849'
WebElement btn = driver.findElement(By.xpath("//button[starts-with(@id, 'submit-btn-')]"));
```

2. **Rely on robust, static attributes:**
I ignore the dynamic ID entirely and look for attributes that don't change, like `name`, `data-testid`, `aria-label`, or custom attributes provided by devs.
```java
// Using a custom data attribute which devs put specifically for QA
WebElement input = driver.findElement(By.cssSelector("[data-testid='username-input']"));
```

3. **Parent-Child traversal / XPath Axes:**
If the element itself has no stable attributes, I find a stable parent or sibling and navigate from there.
```java
// Find the stable label, then find the following sibling input
WebElement dynamicInput = driver.findElement(By.xpath("//label[text()='First Name']/following-sibling::input"));
```
"

---

# PART 13: SCREENSHOTS & EVIDENCE

### 10. TakesScreenshot Interface
**Q: How do you capture a screenshot in Selenium? Explain the TakesScreenshot interface.** 🔥
**A:** "Capturing screenshots is crucial for debugging failures in CI/CD pipelines. In Selenium, we use the `TakesScreenshot` interface.

Because the `WebDriver` interface doesn't have the screenshot methods directly, we have to downcast the driver object to `TakesScreenshot`. Then we use the `getScreenshotAs()` method, specifying that we want the output as a `FILE`. Finally, we use a utility class like Apache Common's `FileUtils` or Selenium's `FileHandler` to copy that temporary file to a physical location on our disk.

**Here's the exact code we use:**
```java
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.io.FileHandler;
import java.io.File;

public void takeScreenshot() {
    // 1. Cast driver to TakesScreenshot
    TakesScreenshot ts = (TakesScreenshot) driver;
    
    // 2. Capture screenshot as a temporary File object
    File srcFile = ts.getScreenshotAs(OutputType.FILE);
    
    // 3. Define the destination path
    File destFile = new File("./screenshots/homepage.png");
    
    try {
        // 4. Copy from temp location to our folder
        FileHandler.copy(srcFile, destFile);
        System.out.println("Screenshot saved successfully.");
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```
"

### 11. Full Page Screenshot
**Q: How do you take a full-page screenshot (including the scrollable area)?**
**A:** "Standard screenshots only capture the visible viewport. If the page is long, the bottom is cut off.

In Selenium 4, Firefox supports full-page screenshots natively. But for Chrome (which we use 90% of the time), we typically rely on third-party libraries like AShot, or use Chrome DevTools Protocol (CDP).

**Firefox Selenium 4 native approach:**
```java
// Cast to FirefoxDriver specifically
FirefoxDriver firefoxDriver = (FirefoxDriver) driver;
File src = firefoxDriver.getFullPageScreenshotAs(OutputType.FILE);
FileHandler.copy(src, new File("./fullpage.png"));
```

**For Chrome (Using AShot library - common in enterprise):**
```java
// AShot stitches the scrollable areas together
Screenshot fpScreenshot = new AShot()
    .shootingStrategy(ShootingStrategies.viewportPasting(1000))
    .takeScreenshot(driver);
    
ImageIO.write(fpScreenshot.getImage(), "PNG", new File("./chrome-fullpage.png"));
```
"

### 12. Element Screenshot (Selenium 4)
**Q: How do you take a screenshot of a specific web element?**
**A:** "One of the best features added in Selenium 4 is element-level screenshots. In the past, we had to take a full screenshot, get the coordinates of the element, and crop the image manually using Java's ImageIO.

Now, the `WebElement` interface itself extends `TakesScreenshot`.

**Real-time use case:**
We use this to capture just the QR code element on an order confirmation page, or to grab a specific chart for visual validation.

```java
// 1. Locate the specific element
WebElement chartElement = driver.findElement(By.id("monthly-sales-chart"));

// 2. Call getScreenshotAs directly on the WEBELEMENT, not the driver
File src = chartElement.getScreenshotAs(OutputType.FILE);

// 3. Save it
File dest = new File("./element-screenshots/sales-chart.png");
FileHandler.copy(src, dest);
```
"

### 13. Screenshot on Test Failure
**Q: How do you automatically capture screenshots ONLY when a test fails?** 🔥
**A:** "This is a standard framework requirement. We don't want to clutter our storage with screenshots of passing tests. We only want evidence when a test fails.

To achieve this, we use TestNG's `@AfterMethod` annotation combined with the `ITestResult` interface. `ITestResult` stores the status of the test that just finished execution.

**Here is how we implement it in our BaseClass:**
```java
@AfterMethod
public void tearDown(ITestResult result) {
    // Check if the test result is FAILURE
    if (ITestResult.FAILURE == result.getStatus()) {
        try {
            TakesScreenshot ts = (TakesScreenshot) driver;
            File src = ts.getScreenshotAs(OutputType.FILE);
            
            // Generate a dynamic name: TestMethodName + Timestamp
            String testName = result.getName();
            String timestamp = new SimpleDateFormat("yyyyMMdd_HHmmss").format(new Date());
            String destPath = "./failed_tests/" + testName + "_" + timestamp + ".png";
            
            FileHandler.copy(src, new File(destPath));
            System.out.println("Captured screenshot for failed test: " + testName);
            
        } catch (Exception e) {
            System.out.println("Exception while taking screenshot " + e.getMessage());
        }
    }
    
    // driver.quit(); // Close browser
}
```
"

### 14. Attaching Screenshots to Reports
**Q: How do you embed screenshots into your Extent Reports?**
**A:** "Saving screenshots to a folder is good, but stakeholders want to see the screenshot directly inside the HTML report. In our framework, we use ExtentReports.

Instead of saving the file locally and linking it, I prefer getting the screenshot as a Base64 string and attaching it directly. This makes the HTML report portable (you can email it without sending a zip folder of images).

```java
// Method to get Base64 screenshot
public String getBase64Screenshot() {
    return ((TakesScreenshot) driver).getScreenshotAs(OutputType.BASE64);
}

// Inside a TestNG Listener or @AfterMethod on failure:
if (result.getStatus() == ITestResult.FAILURE) {
    // Log the failure reason
    extentTest.fail(result.getThrowable());
    
    // Attach the Base64 screenshot directly to the report
    extentTest.addScreenCaptureFromBase64String(getBase64Screenshot(), "Failure Evidence");
}
```
"

### 15. Screenshot Best Practices
**Q: What are the best practices for handling screenshots in an enterprise framework?**
**A:** "Based on my experience, if you don't manage screenshots well, your Jenkins server will run out of disk space very quickly.

1. **Timestamping:** Never hardcode screenshot names. Always append `System.currentTimeMillis()` or a formatted date string. Otherwise, tests running in parallel will overwrite each other's screenshots.
2. **Folder organization:** Create a dynamic folder for every test run (e.g., `/Screenshots/Run_12_Oct_2023/`).
3. **Base64 for Reports:** As mentioned, use Base64 for Extent/Allure reports so the report is a single self-contained HTML file.
4. **Auto-Cleanup:** We have a pre-build script in Jenkins or a `@BeforeSuite` method in Java that deletes screenshots older than 7 days from the workspace to save space.

---

# PART 14: SELENIUM EXCEPTIONS ⭐⭐⭐⭐⭐

*Note for Interview:* Exceptional handling is the heart of an automation engineer's job. When asked about exceptions, always state: **What it is → Why it happens → How to fix it.**

### 16. NoSuchElementException 🔥
**What it is:** The Holy Grail of exceptions. WebDriver cannot find the element in the DOM.
**Why it happens:** 
1. The locator (XPath/ID) is completely wrong.
2. The element hasn't loaded into the DOM yet (timing issue).
3. The element is inside an `iframe` or `Shadow DOM` and you haven't switched to it.
**How to fix:**
"First, I manually verify the XPath in Chrome DevTools. If it's correct, it's usually a sync issue, so I apply an Explicit Wait. If it still fails, I check if the DOM structure has an iframe tag above the element."
```java
// Fix: Add explicit wait
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
wait.until(ExpectedConditions.presenceOfElementLocated(By.id("username")));
```

### 17. StaleElementReferenceException 🔥
**What it is:** The element was found previously, but is no longer attached to the current DOM.
**Why it happens:** The page refreshed, or an AJAX call updated the DOM, completely destroying the old HTML element and replacing it with a new one.
**How to fix:**
"The reference is dead. You cannot 'revive' it. You must tell WebDriver to search for the element again right at the moment you need it."
```java
// Problem:
WebElement btn = driver.findElement(By.id("submit"));
driver.navigate().refresh(); // DOM is rebuilt
btn.click(); // Throws StaleElementReferenceException!

// Fix: Re-initialize the element
btn = driver.findElement(By.id("submit"));
btn.click(); // Works perfectly
```

### 18. TimeoutException 🔥
**What it is:** A command did not complete in enough time (usually associated with Explicit Waits).
**Why it happens:** You told Selenium to wait 10 seconds for an element to become visible, but 10 seconds passed and it's still not visible. Usually caused by slow environments, server down, or the element fundamentally changing behavior.
**How to fix:**
"I check the application manually. If the app is genuinely slow, I increase the wait duration. If the element is actually there but the condition fails, maybe I'm waiting for 'Visibility' but the element is technically 'Hidden' (opacity: 0), so I switch my expected condition to `presenceOfElementLocated`."

### 19. ElementNotInteractableException 🔥
**What it is:** The element is present in the DOM, but its state prevents interaction.
**Why it happens:** 
1. The element is hidden (`display: none`).
2. The element is disabled (`disabled` attribute).
3. The element is physically off-screen and cannot be scrolled to automatically.
**How to fix:**
"I use an explicit wait to wait for `ExpectedConditions.elementToBeClickable()`. If it's deliberately hidden but I must interact with it, I use JavaScriptExecutor to force the click."
```java
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
wait.until(ExpectedConditions.elementToBeClickable(By.id("login-btn"))).click();
```

### 20. ElementClickInterceptedException 🔥
**What it is:** Selenium found the element, and it's visible, but clicking it would actually click a *different* element covering it.
**Why it happens:** Popups, sticky headers, loading overlays, or cookie banners are floating on top of the target element.
**How to fix:**
"Wait for the overlay to disappear, or use JavaScript to click directly on the DOM node."
```java
// Fix using JS
WebElement target = driver.findElement(By.id("target"));
JavascriptExecutor js = (JavascriptExecutor) driver;
js.executeScript("arguments[0].click();", target);
```

### 21. InvalidSelectorException
**What it is:** The locator strategy syntax is invalid.
**Why it happens:** You wrote a bad XPath or CSS selector. E.g., `//div[@class='abc']//` (trailing slashes) or missing brackets.
**How to fix:** "I copy the XPath, open Chrome DevTools (F12), hit Ctrl+F, paste it, and fix the syntax error."

### 22. NoSuchFrameException
**What it is:** WebDriver tries to switch to a frame that doesn't exist.
**Why it happens:** You provided the wrong frame name/ID/Index, or the iframe hasn't loaded yet.
**How to fix:** Use an explicit wait specifically designed for frames.
```java
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
wait.until(ExpectedConditions.frameToBeAvailableAndSwitchToIt("payment-frame"));
```

### 23. NoSuchWindowException
**What it is:** WebDriver tries to switch to a window handle that doesn't exist.
**Why it happens:** The popup window was closed by the application before you switched to it, or you are passing a stale window handle string.
**How to fix:** Always fetch the latest window handles dynamically using `driver.getWindowHandles()`.

### 24. WebDriverException
**What it is:** The parent exception for all Selenium exceptions. It acts as a generic failure.
**Why it happens:** Usually system-level issues: the browser crashed, the chromedriver executable is incompatible with your browser version, or the WebDriver session was unexpectedly terminated.
**How to fix:** "I check if my ChromeDriver version matches my Chrome browser version. I also ensure the browser isn't crashing out of memory (OOM)."

---

### 25. Exception Handling Best Practices
**Q: How do you design your framework to handle exceptions elegantly?**
**A:** "In a robust framework, we rarely let raw Selenium exceptions bubble up to the reports. We wrap our interactions.

1. **Wrapper Methods:** We create custom `click()`, `type()`, and `getElement()` methods.
2. **Try-Catch Blocks:** Inside these wrappers, we catch specific exceptions.
3. **Custom Logging:** If we catch a `TimeoutException`, we log a friendly message to Extent Reports: *"Failed to find username input after 15 seconds"*, rather than dumping a massive Java stack trace.
4. **Auto-Screenshots:** Our catch block automatically triggers the screenshot utility so we have immediate visual proof of the failure state."

---

### 26. MASTER EXCEPTION TABLE 🔥

| Exception | Root Cause | Solution Strategy |
|-----------|------------|-------------------|
| **NoSuchElementException** | Element not in DOM, wrong locator, inside iframe/shadow root | Check locator, use `presenceOfElementLocated` wait, switch to iframe |
| **StaleElementReferenceException** | DOM refreshed/updated, element ID changed | Re-locate the element immediately before interacting, use Try-Catch retry loop |
| **TimeoutException** | Explicit wait condition failed to meet within time | Increase wait time, verify the element actually meets the condition (e.g. visibility) |
| **ElementNotInteractableException** | Element in DOM but hidden or disabled | Wait for `elementToBeClickable`, or bypass UI using `JavascriptExecutor` |
| **ElementClickInterceptedException** | Another element (overlay/popup) covers the target | Wait for overlay invisibility, scroll element to center, or use JS click |
| **NoSuchFrameException** | Frame not loaded or wrong frame ID/name | Use `frameToBeAvailableAndSwitchToIt` explicit wait |
| **NoSuchWindowException** | Window closed or wrong handle | Iterate over `driver.getWindowHandles()` dynamically |
| **InvalidSelectorException** | Syntax error in XPath/CSS | Validate XPath manually in browser DevTools |
| **WebDriverException** | Browser crash, driver version mismatch, connection lost | Update driver/browser versions, check system memory |

---

### 27. Interview Questions on Exceptions

**Q1. What is the difference between NoSuchElementException and ElementNotInteractableException?**
**A:** "`NoSuchElementException` means the element physically does not exist in the HTML DOM structure. `ElementNotInteractableException` means the element *is* present in the DOM, but Selenium cannot interact with it because it is hidden, disabled, or off-screen."

**Q2. You get a StaleElementReferenceException. How exactly do you fix it in your code?**
**A:** "I remove any stored `WebElement` variables. I ensure that the `driver.findElement()` call is made on the exact line immediately before the `.click()` or `.sendKeys()` action. If the page is highly dynamic (like React), I wrap the find-and-click logic in a `while` loop that catches the exception and retries locating the element up to 3 times."

**Q3. If an element is covered by a 'Loading...' spinner, which exception is thrown?**
**A:** "It will throw an `ElementClickInterceptedException`. Selenium attempts to click the coordinate of the target element, but the 'Loading' overlay receives the click event instead."

**Q4. How do you handle TimeoutException? Does it mean the element doesn't exist?**
**A:** "Not necessarily. `TimeoutException` just means the specific `ExpectedCondition` was not met within the timeframe. The element might exist in the DOM (so no `NoSuchElementException`), but we might be waiting for it to be visible, and it remained hidden past the timeout. I fix it by checking what exact condition I'm waiting for and adjusting it, or increasing the wait time if the app is heavily loaded."

**Q5. Can we handle StaleElementReferenceException using Explicit Waits?**
**A:** "Yes. The `ExpectedConditions` class has a method called `refreshed()`. You can pass another condition inside it. For example: `wait.until(ExpectedConditions.refreshed(ExpectedConditions.elementToBeClickable(By.id("btn"))));`. This tells Selenium to expect the element to become stale and automatically redraw/re-locate it."

**Q6. What exception do you get if you don't switch to an iframe and try to locate an element inside it?**
**A:** "You will get a `NoSuchElementException`. Because without switching, WebDriver is only searching the main document HTML, and it is completely blind to the HTML inside the iframe."

**Q7. What happens if you try to select an option from a custom dropdown (made of divs and ul/li) using the Select class?**
**A:** "It will throw an `UnexpectedTagNameException`. The Selenium `Select` class is strictly designed to work ONLY with native HTML `<select>` tags. For custom dropdowns, we must click the dropdown to open it, and then click the `<li>` element directly."

**Q8. Why might isDisplayed() throw a NoSuchElementException?**
**A:** "Because `isDisplayed()` requires a `WebElement` to act upon. If `driver.findElement()` fails to locate the element in the DOM first, it immediately throws `NoSuchElementException` before `isDisplayed()` can even be evaluated. If you want to safely check presence without exceptions, you use `driver.findElements().size() > 0`."

**Q9. What is an UnhandledAlertException?**
**A:** "This happens when an unexpected JavaScript Alert/Prompt pops up on the browser. WebDriver blocks subsequent commands until the alert is dealt with. If you try to click a button while an alert is open, you get this exception. To fix it, you must switch to the alert and accept/dismiss it: `driver.switchTo().alert().accept();`"

**Q10. In a Try-Catch block handling exceptions in Selenium, what do you usually put in the Finally block?**
**A:** "In a standard test flow, I don't always use a `finally` block for every interaction. But at a framework level (like TestNG `@AfterMethod`), the equivalent of a `finally` block is where we ensure `driver.quit()` is executed, so that even if the test threw an unhandled exception and failed, the browser instance is properly closed, preventing memory leaks and zombie processes."
