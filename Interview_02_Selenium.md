# Selenium Interview Q&A for Infosys L2 SDET

## Section 1: Selenium Basics

**Q: 1. 🔥 What is Selenium? What are its components?**
**A:** In my experience, when someone asks about Selenium, the best way to explain it is that it's not just a single tool, but rather a suite of tools for automating web browsers. In our projects, we primarily use it to simulate real user interactions on a web application to verify that it functions correctly. 
What we typically do is use **Selenium WebDriver**, which is the core component that communicates directly with the browser. 
The components include:
- **Selenium IDE:** A record-and-playback tool. I rarely use this in enterprise projects, but it's good for quick prototyping.
- **Selenium RC (Remote Control):** This is deprecated now. We don't use it anymore as it required a JavaScript injection server.
- **Selenium WebDriver:** This is our bread and butter. It provides a programming interface to create and execute test cases.
- **Selenium Grid:** We use this heavily for parallel execution. In my current project, we distribute tests across multiple machines and browsers to reduce execution time from hours to minutes.
From my experience, understanding this ecosystem helps in deciding the right tool for the job, but 99% of the time, we are working with WebDriver and Grid.

**Q: 2. 🔥 What are the advantages and disadvantages/limitations of Selenium?**
**A:** Let me break this down based on what I've seen in my projects.
**Advantages:**
First, it's open-source and free, which is why most companies prefer it over paid tools like UFT. Second, it supports multiple languages—Java, Python, C#—so our team can use what we're comfortable with (usually Java in my case). It also supports cross-browser testing (Chrome, Firefox, Edge) and integrates seamlessly with CI/CD tools like Jenkins and build tools like Maven.
**Disadvantages/Limitations:**
However, it's not a silver bullet. The biggest limitation we face is that it only supports web applications. We can't automate desktop or mobile apps directly (we use Appium for mobile). Also, there is no built-in reporting mechanism; we have to integrate third-party tools like ExtentReports or Allure. Another pain point is handling CAPTCHAs and barcodes, which Selenium simply cannot do natively. Lastly, we often have to deal with flaky tests due to synchronization issues, which requires careful implementation of waits.

**Q: 3. 🔥 What is the difference between Selenium 3 and Selenium 4?**
**A:** This is a crucial upgrade we recently handled in my project.

| Feature | Selenium 3 | Selenium 4 |
|---------|------------|------------|
| Architecture | Used JSON Wire Protocol to communicate between script and browser | W3C WebDriver Standardization. Direct communication. |
| Browser Drivers | Needed setup (e.g., System.setProperty) or WebDriverManager | Selenium Manager is built-in. No need to download drivers manually. |
| Locators | Standard locators | Introduced Relative Locators (above, below, near) |
| Grid | Complex setup with Hub and Node jars | Standalone mode, fully supports Docker |
| DevTools | No native support | Native Chrome DevTools Protocol (CDP) support |

**Verbal explanation:** The way I handle this explanation is by focusing on the W3C standardization. In Selenium 3, the code had to encode requests into JSON, send it to the browser driver, which then executed it. In Selenium 4, because of W3C compliance, the code talks directly to the browser, making execution faster and less flaky. Also, the native Selenium Manager in Selenium 4 has saved us so much time—we no longer have to worry about updating `chromedriver.exe` every time Chrome updates.

**Q: 4. 🔥 Explain Selenium WebDriver architecture — how does it work internally?**
**A:** In my project, understanding the architecture helps us debug failures faster. Here's how it works internally, especially in Selenium 4:
When we write a script (like `driver.get("url")`) in our IDE, the language bindings (Java, Python) convert this into an HTTP request.
Because Selenium 4 uses the **W3C WebDriver Protocol**, this HTTP request is sent directly to the respective Browser Driver (like ChromeDriver or GeckoDriver) via a local server.
The Browser Driver then communicates with the real browser using the browser's native HTTP/WebSockets communication. The browser executes the action (like clicking a button).
Finally, the browser sends the response back to the Browser Driver, which then sends the HTTP response back to our script. If the element isn't found, we get an exception like `NoSuchElementException`.
What we typically do is check the logs at the driver level if something goes fundamentally wrong, but mostly this W3C standard has made interactions much more stable than the old JSON Wire Protocol.

**Q: 5. What is the difference between Selenium IDE, RC, WebDriver, and Grid?**
**A:** From my experience, you can look at these as the evolution and scaling of Selenium.
**Selenium IDE** is just a browser extension for record and playback. We don't use it for robust frameworks because it generates brittle code.
**Selenium RC** is the old version (Selenium 1) that injected JavaScript into the browser to automate it. It's completely obsolete now.
**Selenium WebDriver** is what we use to write our automation scripts. It provides native OS-level control of the browser, making it fast and reliable.
**Selenium Grid** is our scaling solution. When I have 500 tests, running them on one machine takes too long. We use Grid to distribute these tests across multiple VMs and browsers concurrently. So WebDriver is for *creating* the tests, and Grid is for *executing* them at scale.

**Q: 6. Why do we write WebDriver driver = new ChromeDriver()? Explain the OOP concept behind it**
**A:** This is a classic question. The way I explain this is through the concept of **Upcasting** and **Interfaces** in Object-Oriented Programming.
`WebDriver` is an interface in Selenium. It contains all the abstract methods like `get()`, `findElement()`, `quit()`.
`ChromeDriver`, `FirefoxDriver`, etc., are implementation classes that provide the actual logic for these methods specific to their browsers.
By writing `WebDriver driver = new ChromeDriver();`, we are creating an object of the `ChromeDriver` class but storing its reference in an interface variable.
```java
WebDriver driver = new ChromeDriver();
// Later, we can switch the browser without changing the whole code:
driver = new FirefoxDriver();
```
In our framework, this allows us to achieve **Run-time Polymorphism**. We can pass the `WebDriver` object around, and depending on what was instantiated (based on our testng.xml parameter), it will execute on the correct browser. This makes our framework highly flexible and decoupled.

---

## Section 2: Locators

**Q: 7. 🔥 What are locators in Selenium? Name all 8 locators**
**A:** Locators are the mechanism Selenium uses to identify and interact with web elements on a page. In my projects, finding robust locators is 80% of the battle in writing stable tests.
Selenium provides 8 standard locator strategies. I rank them by preference:
1. `id` - The fastest and most reliable, if available.
2. `name` - Also very reliable.
3. `className` - Good, but often multiple elements share a class.
4. `tagName` - Used mostly when fetching multiple elements like all `<a>` links.
5. `linkText` - For exact text matches on anchor tags.
6. `partialLinkText` - For dynamic links where only part of the text is constant.
7. `cssSelector` - Very fast, heavily used in our frameworks.
8. `xpath` - The most powerful, but slightly slower. We use this for complex traversals.
In Selenium 4, we also have **Relative Locators** like `above()`, `below()`, `toLeftOf()`, etc., which are extremely handy for dynamic grids.

**Q: 8. 🔥 What is the difference between CSS Selector and XPath? Which is faster?**
**A:** We had a huge debate about this in my team when setting up our framework.

| Feature | CSS Selector | XPath |
|---------|-------------|-------|
| Speed | Faster and more native to browsers | Slightly slower due to DOM traversal engine |
| Traversal | Can only traverse top-down (parent to child) | Can traverse top-down AND bottom-up (child to parent) |
| Text Match | Does not support text-based locators natively | Supports text matches like `text()='Login'` |
| Syntax | Simpler and cleaner (e.g., `#submit`) | More complex (e.g., `//button[@id='submit']`) |

**Verbal explanation:** In my experience, **CSS Selector is faster** because browsers have native engines tuned for CSS rendering. However, the speed difference is negligible on modern browsers. What we typically do is prefer CSS Selectors for simple elements (like `#username`), but we heavily rely on XPath when we need to find an element based on its text or when we need to traverse backwards from a child element to a parent row in a dynamic web table.

**Q: 9. 🔥 What is the difference between Absolute XPath and Relative XPath?**
**A:** This is a fundamental concept for maintaining stable automation scripts.

| Feature | Absolute XPath | Relative XPath |
|---------|---------------|----------------|
| Syntax | Starts with a single slash `/` | Starts with a double slash `//` |
| Traversal | Starts from the root HTML node | Starts from any matching node in the DOM |
| Example | `/html/body/div[2]/form/input` | `//input[@id='email']` |
| Stability | Highly brittle. Breaks if any UI change occurs | Highly stable and robust |

**Verbal explanation:** In my project, we have a strict code review rule: **Never use Absolute XPath**. If a developer adds a simple `<div>` wrapper somewhere in the page, an absolute XPath will instantly break. We always use Relative XPath because it jumps directly to the element we want, regardless of where it is in the DOM hierarchy. It makes our tests incredibly resilient to minor UI changes.

**Q: 10. 🔥 How do you handle dynamic elements? Give 5 XPath strategies with code**
**A:** Handling dynamic elements—where IDs change on every refresh (like `btn-12345`)—is a daily task for an SDET. Here are the strategies I use in my framework:

1. **contains()**: Great when part of the attribute is constant.
```java
driver.findElement(By.xpath("//input[contains(@id, 'user_')]"));
```
2. **starts-with()**: Useful when the prefix is static but the suffix is dynamic.
```java
driver.findElement(By.xpath("//button[starts-with(@id, 'submit_')]"));
```
3. **text()**: Locating purely by the visible text, completely ignoring dynamic attributes.
```java
driver.findElement(By.xpath("//button[text()='Login']"));
```
4. **normalize-space()**: Similar to text(), but strips out leading/trailing whitespace which often causes flakiness.
```java
driver.findElement(By.xpath("//div[normalize-space()='Welcome User']"));
```
5. **Logical Operators (AND/OR)**: Combining multiple attributes to ensure uniqueness.
```java
driver.findElement(By.xpath("//input[@type='text' and contains(@class, 'form-control')]"));
```
What we typically do is combine `contains` with `text()` to build rock-solid locators that survive application updates.

**Q: 11. 🔥 Explain XPath axes — parent, child, following-sibling, preceding-sibling, ancestor, descendant**
**A:** XPath axes are lifesavers when dealing with complex structures like Web Tables. In my experience, you can't survive without them when automating modern SPAs.
- **parent::** Goes one level up. E.g., finding the `<tr>` from a `<td>`.
  `//td[text()='John']/parent::tr`
- **child::** Goes one level down.
  `//table[@id='users']/child::tbody`
- **following-sibling::** Finds elements at the same level, *after* the current node. We use this to find the "Action" button in the same row as a user name.
  `//td[text()='John']/following-sibling::td//button`
- **preceding-sibling::** Finds elements at the same level, *before* the current node. Useful for checking a checkbox before a user name.
  `//td[text()='John']/preceding-sibling::td/input`
- **ancestor::** Goes up multiple levels to find a specific parent wrapper.
  `//button[text()='Submit']/ancestor::form`
- **descendant::** Goes down multiple levels, skipping intermediate nodes.
  `//div[@id='container']/descendant::a`
The way I handle dynamic grids is almost exclusively using `following-sibling` to link a static label to its dynamic input field.

**Q: 12. 🔥 What XPath functions do you use? contains(), text(), starts-with(), normalize-space()**
**A:** Yes, these are the core functions I use daily. 
In my project, developers often add non-breaking spaces `&nbsp;` or newlines in the HTML, which makes standard `text()='Submit'` fail. 
- **normalize-space()** is my go-to function for this. It trims all leading, trailing, and duplicate spaces.
  `//button[normalize-space(text())='Submit']`
- **contains()** is essential for dynamic classes. Many modern UI libraries like React or Tailwind inject dynamic classes.
  `//div[contains(@class, 'MuiButton-root')]`
- **starts-with()** handles generated IDs, like `id="session_8839"`.
  `//input[starts-with(@id, 'session_')]`
From my experience, relying on these functions rather than exact attribute matches reduces our script maintenance effort by at least 40%.

**Q: 13. What is the difference between findElement() and findElements()?**
**A:** This is a core concept that often catches junior engineers off guard.

| Feature | `findElement()` | `findElements()` |
|---------|-----------------|------------------|
| Return Type | Returns a single `WebElement` object. | Returns a `List<WebElement>`. |
| Match | Returns the *first* matching element on the page. | Returns *all* matching elements. |
| Exception | Throws `NoSuchElementException` if not found. | Returns an empty list `[]` (no exception) if not found. |

**Verbal explanation:** What we typically do in our framework is use `findElement` for standard interactions like clicking a button. But `findElements` is a secret weapon for validation. For example, if I need to verify that an element does *not* exist on the page, using `findElement` will throw an exception and fail the test. Instead, I use `findElements(locator).size() == 0` to gracefully assert that the element is absent.

**Q: 14. 🔥 What locator strategy do you follow in your project? Best practices?**
**A:** In my current project, we have a very strict hierarchy documented in our coding guidelines.
1. First, we ask developers to add custom attributes, like `data-test-id` or `data-cy`. This is the absolute best practice because these attributes are immune to CSS or translation changes.
2. If that's not available, we use `id` or `name`.
3. If IDs are dynamic, we prefer `cssSelector` over XPath for performance.
4. We only use `XPath` when we need text-based location or relational traversal (like parent/sibling).
**Best Practices:**
- Never use Absolute XPath.
- Avoid indexing like `//div[3]/span` because UI ordering changes frequently.
- Keep locators short and robust.
The way I handle this is by maintaining all locators in a dedicated Page Object class so that if the UI changes, we only update the locator in one single place.

**Q: 15. How do you locate an element that has no ID, name, or class?**
**A:** This happens all the time with custom web components. In my experience, we have a few workarounds:
1. **Text-based XPath:** If it has visible text, I use `//tag[text()='Value']`.
2. **Relative DOM Traversal:** I find a stable nearby element (like a parent `div` with an ID) and traverse down or across. E.g., `//div[@id='stable_parent']//child_tag`.
3. **CSS Selectors with Attributes:** If it has some random attribute like `placeholder="Enter Name"`, I'll use `input[placeholder='Enter Name']`.
4. **Selenium 4 Relative Locators:** We can use `driver.findElement(with(By.tagName("input")).below(emailLabel))`. This is very readable and effective.
If absolutely none of this works, I will raise a PR or a Jira ticket asking the Dev team to add a `data-testid` attribute. That's the hallmark of a mature SDET approach.

---

## Section 3: Waits

**Q: 16. 🔥 What is synchronization in Selenium? Why do we need waits?**
**A:** Synchronization is the heart of a stable automation framework. In my projects, 90% of test flakiness comes from poor synchronization.
Selenium runs at the speed of code execution, but web applications run much slower—they rely on network latency, database queries, and rendering JavaScript (like React or Angular). If Selenium tries to click a button that hasn't rendered yet, it throws a `NoSuchElementException`.
What we typically do is implement **Waits** to pause the Selenium script execution until the application catches up. We sync the automation tool with the application state. Without waits, our scripts would be completely unreliable in a real-world CI/CD pipeline where server load varies constantly.

**Q: 17. 🔥 What is the difference between Implicit Wait, Explicit Wait, and Fluent Wait?**
**A:** This is critical. Let me compare them based on how we use them.

| Feature | Implicit Wait | Explicit Wait | Fluent Wait |
|---------|---------------|---------------|-------------|
| Scope | Global. Applies to all `findElement` calls. | Local. Applies only to specific elements. | Local. Highly customizable. |
| Condition | Only checks for the presence of the element in the DOM. | Checks specific conditions (visible, clickable, etc.). | Checks specific conditions with polling intervals. |
| Customization | Time-only (e.g., 10 seconds). | Time + Condition. | Time + Condition + Polling Frequency + Exception Ignoring. |

**Verbal explanation:** In my experience, Implicit Wait is a sledgehammer—you set it once, and it waits for elements to just appear in the DOM. The problem is, an element might be in the DOM but hidden or overlapped. That's why we rely heavily on Explicit Wait. It waits for the element to actually be *clickable* or *visible*. Fluent Wait is just an Explicit Wait on steroids, allowing us to define how often to check the DOM and what exceptions to ignore during the wait.

**Q: 18. 🔥 What are ExpectedConditions? Name the most commonly used ones**
**A:** `ExpectedConditions` is a class in Selenium that provides predefined conditions for Explicit Waits. In my framework's utility class, these are the ones I use constantly:
1. `visibilityOfElementLocated()` - Ensures the element is in the DOM *and* greater than 0x0 pixels.
2. `elementToBeClickable()` - Extremely common. Ensures the element is visible and not disabled.
3. `presenceOfElementLocated()` - Just checks if it's in the DOM. Good for hidden elements.
4. `invisibilityOf()` - We use this to wait for loading spinners or overlays to disappear before interacting with the page.
5. `textToBePresentInElement()` - Great for waiting for a status update, like waiting for "Pending" to change to "Completed".
The way I handle UI interactions is by always wrapping my `click()` methods in an `elementToBeClickable` wait to prevent interception errors.

**Q: 19. 🔥 Why should we NOT use Thread.sleep()? What happens if we do?**
**A:** `Thread.sleep()` is a Java concept, not a Selenium concept. It pauses the thread for a fixed amount of time, blindly.
From my experience, using `Thread.sleep()` is the worst practice in automation. Here is why:
If I put `Thread.sleep(5000)`, and the element loads in 1 second, I've just wasted 4 seconds. Multiply that by 500 test cases, and your test suite becomes bloated and slow. Conversely, if the network is slow and it takes 6 seconds, the test will still fail. 
What we typically do is use **Dynamic Waits** (Explicit/Fluent). They poll the DOM and proceed the millisecond the condition is met. This makes our framework both fast and resilient. I actually enforce a SonarQube rule in our repo to fail any PR that contains `Thread.sleep()`.

**Q: 20. 🔥 What is the difference between Implicit Wait and Explicit Wait? Can we use both together?**
**A:** We covered the definitions, but the second part of your question is the real catch.
**No, we should NEVER mix Implicit and Explicit Waits.** 
In my project, we learned this the hard way. The official Selenium documentation strongly advises against mixing them because it can cause unpredictable wait times. For example, if you have an Implicit Wait of 10s and an Explicit Wait of 15s for an element that doesn't exist, Selenium might wait 25 seconds, or it might throw a timeout at 10 seconds depending on how the local driver implements the timeout pooling.
What we typically do is set Implicit Wait to 0 (or don't define it at all) and exclusively use Explicit Wait in a centralized `WaitUtils` class.

**Q: 21. What is Fluent Wait? How is it different from Explicit Wait? When to use it?**
**A:** Fluent Wait is the most customizable wait in Selenium. 
While Explicit Wait polls the DOM every 500 milliseconds by default, Fluent Wait allows us to change that polling frequency. Furthermore, it allows us to ignore specific exceptions during the polling period.
```java
Wait<WebDriver> wait = new FluentWait<>(driver)
    .withTimeout(Duration.ofSeconds(30))
    .pollingEvery(Duration.ofSeconds(2))
    .ignoring(NoSuchElementException.class, StaleElementReferenceException.class);
```
In my experience, I use Fluent Wait in very specific scenarios—like generating a report. Sometimes a backend process takes a long time, and the UI occasionally refreshes, throwing `StaleElementReferenceException`. With Fluent Wait, I can poll every 2 seconds and safely ignore the stale exceptions until the download button finally appears.

**Q: 22. What is WebDriverWait? Write the syntax**
**A:** `WebDriverWait` is actually a subclass of `FluentWait`. It is the implementation we use when we talk about "Explicit Wait".
In Selenium 4, the syntax was updated to use the `Duration` class instead of long integers.
```java
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(15));
WebElement button = wait.until(ExpectedConditions.elementToBeClickable(By.id("submit")));
button.click();
```
In our framework, we encapsulate this syntax into a BasePage class. We have a method like `waitForClickable(By locator)` so that script writers don't have to instantiate `WebDriverWait` every time. It keeps the code clean and strictly follows DRY principles.

**Q: 23. 🔥 What wait strategy do you follow in your project? Explain your approach**
**A:** My wait strategy is highly centralized and entirely based on Explicit Waits.
1. We set Implicit Wait to zero.
2. We have a `WaitStrategy` enum (CLICKABLE, VISIBLE, PRESENCE, NONE).
3. We have an `ActionEngine` or `BasePage` class that handles all interactions. 
When a user calls `clickElement(locator, WaitStrategy.CLICKABLE)`, the framework internally instantiates a `WebDriverWait` and waits for `ExpectedConditions.elementToBeClickable`.
4. We also have a custom wait for page loading. After every major click, we wait for the `document.readyState` to be `complete` using JavaScript.
From my experience, this approach completely decouples wait logic from test scripts, ensuring that 100% of our interactions are perfectly synchronized without test writers needing to think about it.

---

## Section 4: Handling Web Elements

**Q: 24. 🔥 How do you handle dropdowns in Selenium?**
**A:** It depends on how the dropdown is built in the HTML.
If it is a native `<select>` tag, I use the Selenium `Select` class.
```java
Select dropdown = new Select(driver.findElement(By.id("country")));
dropdown.selectByVisibleText("India"); // I prefer this for readability
// dropdown.selectByValue("IND");
// dropdown.selectByIndex(1);
```
However, in modern applications (like React/Angular), dropdowns are usually built with `<div>` and `<ul>/<li>` tags. The `Select` class will throw an exception here. 
The way I handle these custom dropdowns is by performing two actions: First, I locate and click the main dropdown `div` to expand it. Then, I use a dynamic XPath or `findElements` to iterate through the `<li>` list, check the text, and click the one that matches my desired value.

**Q: 25. 🔥 How do you handle multiple windows/tabs in Selenium?**
**A:** Handling multiple windows is all about managing Window Handles, which are unique alphanumeric identifiers assigned by the browser.
Here is the approach we use:
```java
String parentWindow = driver.getWindowHandle();
// Action that opens a new tab
driver.findElement(By.id("newTabBtn")).click(); 

Set<String> allWindows = driver.getWindowHandles();
for(String window : allWindows) {
    if(!window.equals(parentWindow)) {
        driver.switchTo().window(window);
        break; // Switch to the new tab
    }
}
// Perform actions in new tab...
driver.close(); // Close only the new tab
driver.switchTo().window(parentWindow); // Return control to main tab
```
In Selenium 4, they also introduced `driver.switchTo().newWindow(WindowType.TAB)`, which is fantastic when we need to open a brand new blank tab and navigate somewhere without an explicit link click.

**Q: 26. 🔥 How do you handle alerts in Selenium?**
**A:** In my project, we deal with three types of Javascript alerts: simple alerts, confirmation alerts, and prompt alerts.
Because alerts are native browser popups and not part of the HTML DOM, we cannot use XPath to inspect them. We must switch the driver's context to the alert.
```java
// First, wait for the alert to be present
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(5));
wait.until(ExpectedConditions.alertIsPresent());

Alert alert = driver.switchTo().alert();
String text = alert.getText(); // Read the message
alert.sendKeys("Admin"); // If it's a prompt, enter text
alert.accept(); // Click OK
// alert.dismiss(); // Click Cancel
```
What we typically do is wrap this in a `try-catch` block for `NoAlertPresentException` in case the alert is conditional and doesn't always appear.

**Q: 27. 🔥 How do you handle frames/iframes?**
**A:** Just like alerts, elements inside an `iframe` are in a different document context. If you try to find an element inside an iframe from the main page, Selenium will throw a `NoSuchElementException`.
We have to switch into the frame first. We can do this in three ways:
1. By Index: `driver.switchTo().frame(0);` (I avoid this as indexes change).
2. By ID or Name: `driver.switchTo().frame("payment_frame");`
3. By WebElement: `driver.switchTo().frame(driver.findElement(By.xpath("//iframe[@class='embed']")));` (This is the most robust way in my experience).
Once we finish interacting with the iframe, we *must* return to the main document using `driver.switchTo().defaultContent();`. If it's nested frames, we can go up one level using `driver.switchTo().parentFrame();`.

**Q: 28. How do you handle mouse hover, right-click, double-click, drag-and-drop?**
**A:** For complex user gestures, we use the `Actions` class in Selenium. It relies on a builder pattern to compile advanced interactions.
In my e-commerce project, mouse-hovering over menus is a common scenario:
```java
Actions actions = new Actions(driver);
WebElement menu = driver.findElement(By.id("nav-menu"));
actions.moveToElement(menu).perform(); // Mouse hover
```
For other interactions:
- Right Click: `actions.contextClick(element).perform();`
- Double Click: `actions.doubleClick(element).perform();`
- Drag and Drop: `actions.dragAndDrop(sourceElement, targetElement).perform();`
The crucial part that people often forget in interviews is calling `.perform()` at the end. Without `.perform()`, the action is just built but never executed.

**Q: 29. How do you handle file upload and file download in Selenium?**
**A:** For **File Upload**, if the HTML uses an `<input type="file">` tag, it's incredibly simple. We completely bypass the Windows file explorer popup (because Selenium can't interact with OS windows) and just use `sendKeys()`.
```java
driver.findElement(By.id("uploadFile")).sendKeys("C:\\testdata\\image.jpg");
```
If it's not an `input` tag, we have to use third-party tools like Robot Class or AutoIt, but we try to avoid that by asking devs to expose the input tag.
For **File Download**, Selenium doesn't have native download handling. What we typically do is set ChromeOptions/FirefoxProfile to automatically download files to a specific directory without prompting, and then we write plain Java I/O code to check if the file exists in that directory and assert its size/extension.

**Q: 30. How do you scroll down a page?**
**A:** Selenium Actions class has a `scrollToElement` method now, but the most reliable way I've used throughout my career is injecting JavaScript using the `JavascriptExecutor` interface.
```java
JavascriptExecutor js = (JavascriptExecutor) driver;

// Scroll by pixels
js.executeScript("window.scrollBy(0,500)");

// Scroll to bottom of page
js.executeScript("window.scrollTo(0, document.body.scrollHeight)");

// Scroll directly to a specific element
WebElement element = driver.findElement(By.id("footer-link"));
js.executeScript("arguments[0].scrollIntoView(true);", element);
```
In our framework, I always use `scrollIntoView()` before interacting with elements that are at the bottom of a long page, as sometimes elements must be in the viewport to be considered interactable.

**Q: 31. How do you handle calendars/date pickers in Selenium?**
**A:** Date pickers can be tricky. There are generally two approaches we take in our projects:
1. **Direct Input (The easy way):** If the date field is an `<input>` tag and is not read-only, I simply use `sendKeys("12/08/2026")`. This saves so much execution time.
2. **Logic Handling (The realistic way):** If it's a readonly calendar popup, we must automate the clicks. 
   - First, click the date field to open the calendar.
   - We extract the desired Year and Month. We use a `while` loop to click the "Next" or "Prev" month button until the displayed Month/Year matches our target.
   - Finally, we use a dynamic XPath to select the exact day. Example: `//td[not(contains(@class,'disabled')) and text()='15']`.
Handling the disabled dates (like past dates) properly via XPath is key to a robust calendar logic.

**Q: 32. How do you handle auto-suggest/dynamic dropdowns?**
**A:** Auto-suggest dropdowns (like Google Search or flight booking destinations) populate dynamically based on what you type.
The way I handle this is:
1. Use `sendKeys()` on the input box to type a partial string (e.g., "Ban").
2. Implement an Explicit Wait to wait for the dropdown container or the list items (`<li>`) to become visible. This is crucial because it takes time for the backend API to return the suggestions.
3. Use `findElements` to capture all the suggested options in a `List<WebElement>`.
4. Iterate through the list using a `for` loop, get the text of each element, and if it matches the desired value (e.g., "Bangalore"), click it and `break` the loop.
This approach makes the script entirely dynamic and independent of hardcoded indexes.

---

## Section 5: Selenium Exceptions

**Q: 33. 🔥 What are common Selenium exceptions you have faced? How did you handle them?**
**A:** In my day-to-day work, I see a handful of exceptions repeatedly. 
1. `NoSuchElementException`: Handled by correcting locators or adding waits.
2. `StaleElementReferenceException`: Handled by re-initializing the element inside a try-catch block.
3. `ElementClickInterceptedException`: Handled by scrolling to the element or using JavaScript click.
4. `TimeoutException`: Handled by increasing explicit wait times or checking environment performance.
5. `ElementNotInteractableException`: Handled by ensuring the element is visible and not disabled before sending keys.
The way I structure my framework is by having a global exception handler in the Base Class, which automatically takes a screenshot whenever any of these exceptions occur, so we can debug exactly what the UI looked like at that exact millisecond.

**Q: 34. 🔥 What is NoSuchElementException? How to handle it?**
**A:** This is the most basic exception. It means WebDriver looked at the DOM and could not find an element matching the locator you provided.
From my experience, it happens for three reasons:
1. **Wrong Locator:** The ID or XPath changed. Fix: Update the locator.
2. **Timing Issue:** The element hasn't loaded yet. Fix: Use `WebDriverWait` with `ExpectedConditions.presenceOfElementLocated`.
3. **Context Issue:** The element is inside an iframe or another window. Fix: Switch to the correct iframe/window first.
I always check the screenshot first. If the element is on the screen, it's an iframe or timing issue. If the UI changed, it's a locator issue.

**Q: 35. 🔥 What is StaleElementReferenceException? Why does it occur? How to fix it?**
**A:** This is the most dreaded exception in modern SPA applications (React/Angular). 
It occurs when you locate an element and store it in a variable, but before you can interact with it, the DOM refreshes via AJAX. The reference you hold becomes "stale" (dead).
```java
WebElement btn = driver.findElement(By.id("save"));
driver.navigate().refresh(); // DOM is destroyed and rebuilt
btn.click(); // Throws StaleElementReferenceException
```
The way I fix this is using the **Retry Pattern**. I write a wrapper method that catches the exception and attempts to relocate the element:
```java
public void clickWithRetry(By locator) {
    for (int i = 0; i < 3; i++) {
        try {
            driver.findElement(locator).click();
            break;
        } catch (StaleElementReferenceException e) {
            // Element is stale, loop iterates and finds it again
        }
    }
}
```
Using the Page Factory `@CacheLookup` annotation makes this *worse*, so we avoid it for dynamic elements.

**Q: 36. 🔥 What is ElementClickInterceptedException? How to resolve it?**
**A:** I see this daily. It happens when the element you are trying to click is present and visible, but another element—like a sticky header, a chat widget, or a loading spinner overlay—is physically covering it on the screen.
When Selenium tries to click the center of your button, it ends up clicking the overlay instead, throwing this exception.
How I handle it:
1. Wait for the overlay (spinner) to disappear using `ExpectedConditions.invisibilityOf()`.
2. Scroll the element into view so it's not hidden under a sticky nav bar.
3. If it's a design quirk and I absolutely must click it, I bypass the UI layer entirely using JavaScript execution:
   `js.executeScript("arguments[0].click();", element);`
We prefer fixing the waits, but JS click is the ultimate fallback.

**Q: 37. What is ElementNotInteractableException? When does it occur?**
**A:** This occurs when an element is technically in the DOM, but it is in a state that cannot be interacted with. For example, trying to `sendKeys()` to a hidden input field, or clicking a button that has a `disabled` attribute.
In my project, we often see this with hidden file upload inputs or accordion menus that haven't expanded yet. 
To resolve it, I ensure I wait for `ExpectedConditions.visibilityOfElementLocated()`, or I make sure my script simulates the user action that triggers the element to become visible (like clicking the accordion expand icon first).

**Q: 38. What is TimeoutException? How to handle it?**
**A:** This is thrown specifically by `WebDriverWait` when the condition you are waiting for is not met within the specified time limit. For instance, waiting 15 seconds for an element to be visible, but it never shows up.
Usually, this indicates a true bug in the application (like a server 500 error preventing loading), or a very slow test environment. 
What we typically do is check the application logs to see if the server was slow. If it's purely network latency, we might increase the wait duration. We never handle it by blindly catching the exception, because a Timeout usually means the test *should* fail.

**Q: 39. What is InvalidSelectorException?**
**A:** This is purely a syntax error on the engineer's part. It means the XPath or CSS Selector string provided is syntactically malformed.
For example, missing a closing bracket in XPath: `//div[@class='header'` or using an invalid character in a CSS selector. 
Whenever I see this, I copy the locator, open Chrome DevTools, and test it in the console using `$x("locator")` for XPath or `$$("locator")` for CSS to fix the syntax.

**Q: 40. What is NoSuchFrameException / NoSuchWindowException?**
**A:** `NoSuchFrameException` occurs when we do `driver.switchTo().frame("name")` but the frame doesn't exist. Usually, this means the page hasn't fully loaded the iframe yet. I fix it by using `wait.until(ExpectedConditions.frameToBeAvailableAndSwitchToIt(locator))`.
`NoSuchWindowException` happens when we try to switch to a window handle that has already been closed or never existed. Keeping strict track of parent and child window handles in local variables prevents this.

---

## Section 6: Advanced Selenium

**Q: 41. 🔥 What is JavaScriptExecutor? When do you use it?**
**A:** `JavascriptExecutor` is an interface in Selenium that allows us to execute native JavaScript code directly within the context of the browser. 
It is extremely powerful. In my framework, I use it when standard WebDriver commands fail due to UI complexities.
My top 3 use cases:
1. **Clicking intercepted elements:** `js.executeScript("arguments[0].click();", element);`
2. **Scrolling:** `js.executeScript("arguments[0].scrollIntoView(true);", element);`
3. **Changing element attributes:** Sometimes I need to unhide an element or remove a `readonly` attribute from a date picker to force data injection. `js.executeScript("arguments[0].removeAttribute('readonly')", element);`
While it's a great fallback, I try not to overuse the JS click because it bypasses the actual user interaction layer, which could mask real CSS bugs.

**Q: 42. 🔥 How do you take screenshots in Selenium?**
**A:** We use the `TakesScreenshot` interface. In our CI/CD pipeline, taking screenshots of failed tests is mandatory for debugging.
Here is the standard code we integrate into our TestNG Listeners:
```java
// Cast the driver to TakesScreenshot
TakesScreenshot ts = (TakesScreenshot) driver;
// Capture the screenshot as a FILE object
File source = ts.getScreenshotAs(OutputType.FILE);
// Define destination path
File destination = new File("./screenshots/failure_" + System.currentTimeMillis() + ".png");
// Copy the file
FileUtils.copyFile(source, destination);
```
In Selenium 4, we also have the capability to take a screenshot of a *specific element*, not just the whole page. We do `element.getScreenshotAs(OutputType.FILE)`. We use this for visual regression testing of specific charts or components.

**Q: 43. How do you handle hidden elements?**
**A:** Hidden elements are in the DOM but have CSS properties like `display: none;` or `visibility: hidden;`. Selenium is designed to act like a human, so it refuses to interact with elements a human cannot see.
If I try to click a hidden element, it throws `ElementNotInteractableException`.
If it's a bug, the test should fail. But sometimes, like with hidden file inputs or custom UI toggles, we need a workaround. I use `JavascriptExecutor` to change the CSS property to make it visible, or I use JS to click it directly, completely ignoring the visibility state.

**Q: 44. How do you handle shadow DOM elements?**
**A:** This is a modern challenge. Shadow DOM encapsulates elements so standard XPath and CSS Selectors from the main document cannot find them. 
In Selenium 4, we have the `getShadowRoot()` method which makes this much easier.
```java
// First, locate the shadow host element
WebElement shadowHost = driver.findElement(By.cssSelector("#host"));
// Get the shadow root context
SearchContext shadowRoot = shadowHost.getShadowRoot();
// Find the element inside the shadow DOM (Must use CSS, XPath is not supported in Shadow Root)
WebElement targetElement = shadowRoot.findElement(By.cssSelector(".target-btn"));
targetElement.click();
```
The key restriction we always have to remember is that XPath absolutely does not work inside a Shadow DOM; we must use CSS Selectors.

**Q: 45. How do you handle broken links?**
**A:** We actually have a dedicated utility script for this. We don't click every link because it's too slow.
Instead, we extract all `<a>` tags, get their `href` attributes, and send backend HTTP requests to verify the status codes.
```java
List<WebElement> links = driver.findElements(By.tagName("a"));
for(WebElement link : links) {
    String url = link.getAttribute("href");
    HttpURLConnection conn = (HttpURLConnection) new URL(url).openConnection();
    conn.setRequestMethod("HEAD");
    conn.connect();
    int responseCode = conn.getResponseCode();
    if(responseCode >= 400) {
        System.out.println("Broken Link: " + url);
    }
}
```
This runs incredibly fast and we can validate hundreds of links on a page in seconds.

**Q: 46. How do you handle web tables? Extract data from dynamic tables?**
**A:** Handling dynamic web tables relies heavily on custom XPath.
Let's say I want to find the "Delete" button for a user named "Alice". I don't know which row Alice is in.
I write a dynamic XPath using the `following-sibling` axis:
`//td[text()='Alice']/following-sibling::td/button[text()='Delete']`
If I need to extract all data into a Java data structure:
1. I get all rows: `List<WebElement> rows = driver.findElements(By.xpath("//table[@id='data']/tbody/tr"));`
2. I iterate through the rows, and within the loop, I do `.findElements(By.tagName("td"))` to get the columns for that row.
We use this to verify sorting functionality or pagination data.

**Q: 47. How do you handle CAPTCHA and OTP in automation?**
**A:** This is an architectural discussion we have with developers. Selenium *cannot* and *should not* automate CAPTCHA—the whole point of CAPTCHA is to block automation.
My approach:
1. **In QA/Staging environments:** We ask developers to disable CAPTCHA entirely, or set a static dummy OTP (like `123456`) so we can bypass it.
2. **If we must test OTP:** We ask the devs to expose an internal API endpoint that returns the generated OTP for a specific test user. Our script triggers the login, makes a REST API call to fetch the OTP, and then enters it via Selenium.
Trying to automate reading SMS or Emails is extremely flaky and anti-pattern.

**Q: 48. 🔥 What is headless browser testing? How to run Chrome in headless mode?**
**A:** Headless testing means running the browser without a Graphical User Interface (GUI). The browser runs purely in memory. 
We use this exclusively in our CI/CD pipelines (Jenkins/Linux servers) because servers don't have displays, and headless mode consumes 30% less RAM/CPU, making execution much faster.
```java
ChromeOptions options = new ChromeOptions();
options.addArguments("--headless=new"); // Selenium 4 recommended syntax
options.addArguments("--window-size=1920,1080"); // Crucial to prevent responsive UI bugs
WebDriver driver = new ChromeDriver(options);
```
One catch: setting the window size is critical. In headless mode, the default viewport is often small, causing mobile-views to render and locators to fail.

**Q: 49. What is the difference between close() and quit()?**
**A:** This is a fundamental resource management question.

| Feature | `driver.close()` | `driver.quit()` |
|---------|------------------|-----------------|
| Action | Closes only the current window that WebDriver is focusing on. | Closes ALL windows opened by the WebDriver session. |
| Session | The WebDriver session remains active. | The WebDriver session is gracefully destroyed. |

**Verbal explanation:** In my framework, I almost exclusively use `driver.quit()` inside the `@AfterMethod` or `@AfterClass` teardown block. If you use `close()`, background browser processes (like `chromedriver.exe`) will remain running in your Task Manager, eventually causing memory leaks and crashing the Jenkins node. `close()` is only used when I'm handling multiple tabs and want to shut down a child tab.

**Q: 50. What is the difference between get() and navigate().to()?**
**A:** 
| Feature | `driver.get(url)` | `driver.navigate().to(url)` |
|---------|-------------------|-----------------------------|
| Loading | Waits for the page load event to complete. | Does not strictly wait for page load in older versions. |
| History | Does not maintain browser history. | Maintains history, allowing `forward()`, `back()`, and `refresh()`. |

**Verbal explanation:** Under the hood, they basically do the same thing now in W3C WebDriver. However, we use `get()` for the initial URL launch because it ensures the page is fully loaded before moving to the next step. I only use `navigate().to()` in specific test scenarios where I need to verify browser back/forward navigation features.

---

## Section 7: TestNG

**Q: 51. 🔥 What is TestNG? Why do we use it over JUnit?**
**A:** TestNG (Next Generation) is a testing framework inspired by JUnit but designed to cover a wider range of test categories (unit, functional, integration). As an SDET, Selenium is just an API to drive the browser—it doesn't have assertions or reporting. We need TestNG to actually build the test framework.
I prefer it over JUnit because:
1. **Data Providers:** TestNG allows incredibly easy parameterization through the `@DataProvider` annotation.
2. **Parallel Execution:** Native support for parallel threading via `testng.xml`.
3. **Dependency Testing:** Features like `dependsOnMethods` and `dependsOnGroups`.
4. **Grouping:** I can tag tests as `@Test(groups="smoke")` and run only the smoke suite.
It brings the actual structure to our automation codebase.

**Q: 52. 🔥 Explain all TestNG annotations and their execution order**
**A:** Understanding the hierarchy is critical for setup and teardown logic. The execution order is:
1. `@BeforeSuite` - Runs once before all tests. We use this to initialize ExtentReports or DB connections.
2. `@BeforeTest` - Runs before the `<test>` tag in testng.xml. Good for test-level config.
3. `@BeforeClass` - Runs before the first method of the current class. We usually initialize the WebDriver here.
4. `@BeforeMethod` - Runs before EVERY `@Test` method. We use this to login or clear cookies.
5. `@Test` - The actual test case.
6. `@AfterMethod` - Runs after EVERY `@Test`. We use this for capturing failure screenshots.
7. `@AfterClass` - Runs after all methods in the class. We do `driver.quit()` here.
8. `@AfterTest` - Runs after `<test>` execution.
9. `@AfterSuite` - Runs last, used to flush reports.

**Q: 53. 🔥 What is the difference between @BeforeMethod and @BeforeClass?**
**A:** 
| Feature | `@BeforeMethod` | `@BeforeClass` |
|---------|-----------------|----------------|
| Execution Frequency | Executes before *each* and every `@Test` method in the class. | Executes only *once* per class instantiation. |
| Use Case | Resetting state (e.g., logging in/out, navigating to home page). | Heavy initialization (e.g., launching browser, starting driver). |

**Verbal explanation:** In my framework, if I have 5 test cases in a class, `@BeforeClass` launches Chrome once. `@BeforeMethod` ensures we are on the Home page before test 1, test 2, test 3, etc. If we launched the browser in `@BeforeMethod`, it would open and close Chrome 5 times, making the suite incredibly slow.

**Q: 54. 🔥 What is the difference between Hard Assert and Soft Assert?**
**A:** 
| Feature | Hard Assert (`Assert.assertEquals`) | Soft Assert (`SoftAssert.assertEquals`) |
|---------|-------------------------------------|-----------------------------------------|
| Behavior on Failure | Throws `AssertionError` immediately and aborts the test method. | Records the failure but continues executing the rest of the test. |
| Implementation | Uses static methods. | Requires instantiating a `SoftAssert` object and calling `assertAll()`. |

**Verbal explanation:** We use Hard Asserts for critical validations. E.g., if login fails, there's no point testing the dashboard; the test should stop immediately. 
We use Soft Asserts when validating a UI page with multiple independent fields.
```java
SoftAssert softAssert = new SoftAssert();
softAssert.assertEquals(title, "Dashboard");
softAssert.assertTrue(logo.isDisplayed());
softAssert.assertAll(); // CRITICAL: This throws the accumulated exceptions
```
If the title fails, the script will still check the logo, and at the end, `assertAll()` marks the test as failed.

**Q: 55. 🔥 What is DataProvider? How do you use it for data-driven testing?**
**A:** `DataProvider` is one of the most powerful features in TestNG. It allows us to run a single test method multiple times with different sets of data. In my framework, we read Excel files (using Apache POI), map them to a 2D object array, and feed them to the test.
```java
@DataProvider(name = "loginData")
public Object[][] getData() {
    return new Object[][] {
        {"admin", "pass123"},
        {"user", "invalidpass"},
        {"lockedUser", "pass123"}
    };
}

@Test(dataProvider = "loginData")
public void loginTest(String username, String password) {
    driver.findElement(By.id("user")).sendKeys(username);
    // test logic
}
```
This keeps our code completely DRY. Instead of writing 3 test cases, we write 1 test case and separate the data entirely.

**Q: 56. 🔥 How do you run tests in parallel using TestNG?**
**A:** To drastically reduce execution time in CI/CD, we utilize TestNG's built-in threading. This is controlled entirely via `testng.xml`.
```xml
<suite name="RegressionSuite" parallel="methods" thread-count="5">
    <test name="UI Tests">
        <classes>
            <class name="tests.LoginTest"/>
            <class name="tests.CartTest"/>
        </classes>
    </test>
</suite>
```
We can set `parallel="methods"`, `classes`, or `tests`.
The biggest challenge here is **Thread Safety**. If we use a static `WebDriver` instance, multiple threads will override each other, causing chaos. We must use `ThreadLocal<WebDriver>` to ensure each thread gets its own isolated browser instance.

**Q: 57. What is priority in TestNG? What is dependsOnMethods?**
**A:** By default, TestNG executes tests in alphabetical order of method names, which is rarely what we want.
We use `priority` to define the exact sequence: `@Test(priority = 1)`. 
However, in modern automation, tests should ideally be independent. If I have a hard dependency (e.g., I cannot test checkout if the login test fails), I use `dependsOnMethods = {"loginTest"}`. 
If `loginTest` fails, TestNG will automatically skip the checkout test instead of letting it fail and creating a false alarm. It marks it as "Skipped" in the reports.

**Q: 58. What is @Parameters? How to pass parameters from testng.xml?**
**A:** While DataProvider is for large sets of test data, `@Parameters` is for passing environment-level configurations—like which browser to run, or which URL to target.
```xml
<!-- In testng.xml -->
<parameter name="browser" value="chrome"/>
<parameter name="env" value="qa"/>
```
```java
// In Test Class
@Parameters({"browser", "env"})
@BeforeClass
public void setup(String browser, String env) {
    if(browser.equalsIgnoreCase("chrome")) {
        driver = new ChromeDriver();
    }
}
```
This allows us to run the exact same compiled codebase across different browsers and environments just by altering the XML or passing Maven command-line arguments.

**Q: 59. 🔥 How do you handle test failures? Retry mechanism?**
**A:** In a large suite, network blips or minor rendering delays cause flaky tests. To stabilize the build, we implement `IRetryAnalyzer`.
```java
public class RetryAnalyzer implements IRetryAnalyzer {
    int counter = 0;
    int retryLimit = 2; // Retry a failed test up to 2 times

    @Override
    public boolean retry(ITestResult result) {
        if (counter < retryLimit) {
            counter++;
            return true;
        }
        return false;
    }
}
```
We bind this to tests using `@Test(retryAnalyzer = RetryAnalyzer.class)` or automatically via an `IAnnotationTransformer`. If a test fails, TestNG quietly reruns it. If it passes on retry, the build stays green. This significantly reduces our false-positive failure rate.

**Q: 60. What is testng.xml? Explain the structure**
**A:** `testng.xml` is the configuration heartbeat of a TestNG project. It allows us to manage execution without touching Java code.
The hierarchy is strict: `<suite>` -> `<test>` -> `<classes>` -> `<class>` -> `<methods>`.
We use it to:
- Define parallel execution strategies.
- Pass `<parameter>` values.
- Include or exclude specific groups (`<run><include name="smoke"/></run>`).
- Attach Listeners.
We usually maintain multiple XML files, like `smoke.xml`, `regression.xml`, and trigger them dynamically from Jenkins.

**Q: 61. How do you group tests? How to include/exclude groups?**
**A:** We use the `groups` attribute: `@Test(groups = {"smoke", "regression"})`.
In our CI/CD pipeline, when developers merge code, we don't want to run the 3-hour regression suite. We just want the 10-minute smoke suite.
In `testng.xml`:
```xml
<groups>
    <run>
        <include name="smoke"/>
        <exclude name="wip"/> <!-- work in progress -->
    </run>
</groups>
```
This creates incredible flexibility. We map these TestNG groups to Maven profiles, allowing Jenkins to execute `mvn test -PsmokeSuite`.

**Q: 62. What are TestNG listeners? Name the ones you've used**
**A:** Listeners "listen" to the events fired by TestNG and execute code when certain events occur (like test start, test success, test failure).
The most heavily used listener in my framework is `ITestListener`.
We implement methods like `onTestFailure(ITestResult result)`. Inside this block, we automatically trigger the WebDriver to take a screenshot, extract the stack trace, and attach both to our ExtentReport dashboard.
We also use `ISuiteListener` to set up global database connections before the entire suite starts and tear them down at the end.

---

## Section 8: Scenario-Based Questions

**Q: 63. 🔥 An element is visible on the page but Selenium throws ElementClickInterceptedException. What will you do?**
**A:** This is a classic real-world issue. Usually, it's caused by a loading spinner, a sticky header, or a cookie consent banner covering the element.
My debugging process:
1. I look at the failure screenshot. Is a modal or spinner blocking it? If so, I add an Explicit Wait: `wait.until(ExpectedConditions.invisibilityOfElementLocated(By.id("spinner")));`.
2. Is the element hidden under a sticky navbar? I use JavascriptExecutor to scroll the element into the exact center of the screen, or I scroll down by a few pixels.
3. If it's a permanent UI overlap design, I fallback to `JavascriptExecutor` to click it directly via the DOM: `js.executeScript("arguments[0].click();", element);`.

**Q: 64. 🔥 Your test passes locally but fails in Jenkins. How will you troubleshoot?**
**A:** This is one of my primary responsibilities as a Senior SDET.
1. **Headless & Resolution differences:** Jenkins runs in headless Linux. I check if `--window-size=1920,1080` is set. Without it, the UI might render in a mobile view, hiding elements behind hamburger menus.
2. **Performance/Timing:** Jenkins VMs are usually slower than my local MacBook. A hardcoded implicit wait might work locally but fail in CI. I review the wait strategies and increase explicit wait polling.
3. **Environment Issues:** Does Jenkins have network access to the QA DB? Are the test data states identical? 
4. **Screenshots & Logs:** I check the Jenkins artifact folder for the TestNG failure screenshot and the ExtentReport logs to pinpoint the exact failure line.

**Q: 65. 🔥 How do you handle a situation where the same element has dynamic IDs changing every time?**
**A:** Hardcoding IDs like `id="btn_123"` will fail on the next load when it becomes `id="btn_456"`.
My approach:
1. I inspect the HTML to find a static pattern. If the prefix is always `btn_`, I use `//button[starts-with(@id, 'btn_')]`.
2. I rely on other attributes. Does it have a unique name, class, or `data-cy` attribute?
3. I locate it via its visible text: `//button[text()='Submit']`.
4. If it has no text or unique attributes, I anchor my locator to a stable parent element and traverse down: `//form[@id='loginForm']//button[@type='submit']`.

**Q: 66. 🔥 You need to automate a page that takes 30 seconds to load. What is your approach?**
**A:** A standard implicit wait might not be the best idea here if I don't want to slow down my entire suite just for one page.
1. First, I set the `pageLoadTimeout` in driver configurations to ensure WebDriver doesn't throw a TimeoutException prematurely: `driver.manage().timeouts().pageLoadTimeout(Duration.ofSeconds(60));`
2. I write a custom JavascriptExecutor wait to poll `document.readyState === 'complete'` to ensure the browser has finished rendering.
3. I identify the *last* element that loads on that specific page (e.g., a data grid or a footer) and put an Explicit Wait targeting that specific element before I begin interacting with the page.

**Q: 67. 🔥 How do you verify that a file has been downloaded successfully?**
**A:** Since Selenium cannot interact with the OS file system via browser popups, I bypass the UI dialog entirely.
1. I configure ChromeOptions to automatically download files to a specific target directory inside my project workspace (e.g., `./target/downloads`).
2. I trigger the download click via Selenium.
3. I use Java's `File` class with an explicit wait loop to check if the file appears in that directory within a timeout period.
4. Finally, I assert the file exists, check its file size `file.length() > 0`, and then delete the file via Java code so the directory is clean for the next test run.

**Q: 68. An element is inside a nested iframe. How do you interact with it?**
**A:** Selenium has no context of elements inside an iframe; it only sees the iframe container. If it's nested, I must traverse step by step.
1. First, I switch to the parent iframe: `driver.switchTo().frame("parent_frame");`
2. Then, I switch to the child iframe inside it: `driver.switchTo().frame("child_frame");`
3. Now the WebDriver context is inside the child frame, and I can interact with the element using normal `findElement`.
4. Most importantly, when done, I must reset the context back to the main HTML document using `driver.switchTo().defaultContent();`.

**Q: 69. You have 500 test cases and 50 are failing intermittently. How do you handle flaky tests?**
**A:** Flaky tests destroy trust in the automation framework. If developers don't trust the results, the framework is useless.
My strategy:
1. I quarantine the 50 tests by adding a TestNG group `@Test(groups="quarantine")` so they don't break the main CI pipeline.
2. I implement `IRetryAnalyzer` to automatically retry failures 2 times. If a test passes on retry, it's a clear sign of synchronization (timing) issues.
3. I audit the quarantined tests. 95% of the time, the issue is relying on `Thread.sleep()`, Implicit Waits, or state bleeding (tests not cleaning up data). I replace all static waits with `WebDriverWait`.
4. Once stabilized locally and proven in CI, I move them back to the main suite.

**Q: 70. How do you test a feature that involves OTP verification?**
**A:** Since OTPs are sent to mobile phones or emails, and automating email/SMS clients is notoriously flaky and slow, we collaborate with backend developers.
1. **DB Mocking:** We request a static OTP for test environments (e.g., `111111` for any test user).
2. **API Fetching:** If a static OTP is a security violation, developers provide a secure internal REST API endpoint. My Selenium script enters the phone number, clicks Send. Then I use RestAssured (Java API library) to hit the endpoint, extract the generated OTP from the JSON response, and use `sendKeys()` to inject it back into the web UI. This keeps the test 100% automated and reliable.
