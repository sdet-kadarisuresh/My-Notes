# PART 3: WebDriver Methods

### 1. `get(url)` 🔥

**Q: Can you explain the `get(url)` method and how it handles page loads?**
**A:** Whenever I am asked this, I always start by emphasizing that `get(url)` is the absolute foundation of any Selenium script. In my 7 years of building automation frameworks, `driver.get(url)` is the very first action we perform after instantiating the browser. What it essentially does is command the browser to load a specific web page. 
But the critical part you need to mention in an interview is its synchronization behavior. `get()` is a blocking call. It inherently waits until the page has fully loaded—specifically, it waits for the `document.readyState` to reach the `complete` state before it hands control back to the next line of your code. 
However, there's a catch we often face in real-time projects: if the page is a modern Single Page Application (SPA) like Angular or React, `get()` might return control while AJAX calls are still rendering elements dynamically. That's why we pair it with Explicit Waits later on.

**Syntax:**
```java
WebDriver driver = new ChromeDriver();
driver.get("https://www.example.com");
```

**Real-time usage:** We use this method purely for the initial entry point of our test cases. For example, navigating to the login page of our e-commerce portal before we start executing any test steps.
**Important points:** It does not retain browser history in a way that allows you to easily jump back and forth.
**Common mistakes:** A very common mistake juniors make is forgetting the protocol (`http://` or `https://`). If you just write `driver.get("www.google.com")`, Selenium will throw an `InvalidArgumentException`.

**Q: What is the exact difference between `get()` and `navigate().to()`?** 🔥

| Feature | `get(String url)` | `navigate().to(String url)` |
|---------|-------------------|-----------------------------|
| **Page Load Sync** | Waits for the page to completely load before returning control. | Also waits for the page to load (internally calls `get()`). |
| **Browser History** | Does not actively manage or traverse browser history. | Maintains browser history, allowing `back()` and `forward()` methods. |
| **Method Signature** | Direct method of the `WebDriver` interface. | Belongs to the `Navigation` interface. |
| **Overloading** | Accepts only a String parameter. | Overloaded to accept a `String` or a `java.net.URL` object. |

**Verbal explanation:** Interviewers love this question. The way I explain it is to first bust the biggest myth: many people think `navigate().to()` doesn't wait for the page to load, but that's completely false. Internally, `navigate().to(String)` actually just delegates to `get()`. The real difference comes down to history management. In my projects, if a test scenario requires me to simulate a user hitting the browser's back or forward button, I absolutely have to use the `navigate()` interface. If I just need to launch the app and start clicking, `get()` is perfectly fine and slightly cleaner to read.


### 2. `navigate()`

**Q: How do you use the `navigate()` methods in your framework, and when are they necessary?**
**A:** In my experience, we don't use `navigate()` for everyday element interactions, but it becomes crucial for very specific edge-case testing. The `navigate()` interface provides methods to simulate the browser's built-in navigation buttons. 
For instance, in our banking application, we had a strict security requirement: if a user logs out and then clicks the browser's "Back" button, they should not be able to see their account dashboard. To automate this exact scenario, we couldn't use `get()`. We had to use `driver.navigate().back()`. 
Similarly, `refresh()` is incredibly useful when we are polling for a status update on a web page—like waiting for an order status to change from 'Processing' to 'Shipped'.

**Syntax & Code Examples:**
```java
// Navigating to a URL
driver.navigate().to("https://www.amazon.com");

// Simulating the browser Back button
driver.navigate().back();

// Simulating the browser Forward button
driver.navigate().forward();

// Refreshing the current page
driver.navigate().refresh();
```

**Real-time usage:** Testing session management (login/logout back button behavior) and polling pages for updated data (using refresh).
**Important points:** If you haven't navigated anywhere yet, calling `back()` or `forward()` won't do anything or might throw an exception depending on the browser state.
**Common mistakes:** Using `refresh()` inside a loop without an explicit wait. If the page is heavy, continuous refreshing can crash the session or cause `StaleElementReferenceException` if you try to interact with old elements.


### 3. `getTitle()` 🔥

**Q: Explain how you use `getTitle()` for validation in your automation scripts.**
**A:** `getTitle()` is one of the simplest yet most effective verification checkpoints we use. Every time we navigate to a new page, the first assertion we typically do in our framework is verifying the page title. It fetches the text inside the `<title>` tag in the HTML head.
From my project experience, titles are highly dynamic in modern apps. For example, if you search for "Laptop" on an e-commerce site, the title might change to "Search Results for Laptop". We use `getTitle()` to capture this runtime string and `Assert.assertEquals()` to ensure the user actually landed on the correct page. It’s a very fast validation because it doesn't require finding elements in the DOM.

**Syntax:**
```java
String actualTitle = driver.getTitle();
Assert.assertEquals(actualTitle, "Expected Page Title");
```

**Real-time usage:** Validating successful navigation. E.g., clicking "Login" and verifying the next page title is "User Dashboard".
**Important points:** Sometimes the title takes a fraction of a second to update after a click. If you call `getTitle()` too fast, you might get the title of the previous page.
**Common mistakes:** Doing exact string matches when titles have dynamic trailing spaces or unpredictable capitalization. I usually recommend `assertTrue(title.contains("Expected"))` for more robust tests.


### 4. `getCurrentUrl()`

**Q: Why do we need `getCurrentUrl()` and how is it used practically?**
**A:** The way I handle navigation validation in my automation framework relies heavily on `getCurrentUrl()`. While `getTitle()` is good, URLs are unique identifiers. When testing a secure application, URLs change drastically—for instance, moving from HTTP to HTTPS, or passing authentication tokens in the query parameters.
In a recent project, we integrated a payment gateway. When the user clicked "Pay", they were redirected to a third-party vendor site. We couldn't inspect elements there, but we *could* use `getCurrentUrl()` to assert that the URL contained "paypal.com/checkout". It's the best way to verify redirects without needing complex DOM element locators.

**Syntax:**
```java
String currentUrl = driver.getCurrentUrl();
System.out.println("Landed on: " + currentUrl);
if(currentUrl.contains("dashboard")) {
    System.out.println("Login Successful");
}
```

**Real-time usage:** Validating third-party redirects (SSO logins, payment gateways) and ensuring HTTPS is enforced.
**Important points:** Like the title, the URL might not update instantaneously during AJAX navigations.
**Common mistakes:** Hardcoding full URLs in assertions. Environments change (dev, QA, staging), so it's always better to assert on the URL path or endpoints rather than the full base URL.


### 5. `getPageSource()`

**Q: Have you ever used `getPageSource()`? In what scenarios is it helpful?**
**A:** To be honest, in my 7+ years of automation, we don't use `getPageSource()` very often for assertions because it is quite heavy and slow. What it does is return the entire DOM structure of the current page as a massive String. 
However, it is a lifesaver in very specific scenarios. What we typically do is use it to verify the presence of text on a page when the text doesn't belong to a specific, easily locatable WebElement. For instance, if a generic server error like "500 Internal Error" is thrown directly on the page, checking `driver.getPageSource().contains("500 Internal Error")` is much faster than trying to write an XPath for a completely broken page layout.

**Syntax:**
```java
String pageSource = driver.getPageSource();
if(pageSource.contains("Transaction Successful")) {
    System.out.println("Text found in DOM");
}
```

**Real-time usage:** Quick text validation on static pages or capturing the DOM state right before a test fails for debugging purposes (saving the source to an HTML file).
**Important points:** It returns the source of the page *as it is right now*, including any DOM changes made by JavaScript since the initial load.
**Common mistakes:** Using this for everyday element validation instead of proper `findElement` locators. It's brittle and bad practice for standard verifications.


### 6. `findElement()` vs `findElements()` 🔥

**Q: What is the difference between `findElement` and `findElements`?** 🔥

| Feature | `findElement(By by)` | `findElements(By by)` |
|---------|----------------------|-----------------------|
| **Return Type** | Returns a single `WebElement`. | Returns a `List<WebElement>`. |
| **Match Behavior**| Returns the *first* matching element in the DOM. | Returns *all* matching elements in the DOM. |
| **Failure State** | Throws `NoSuchElementException` if no element is found. | Returns an empty list (`size() == 0`). It does NOT throw an exception. |
| **Use Case** | When you want to interact with a specific, unique element (e.g., login button). | When you want to interact with a collection (e.g., all links on a page, rows in a table). |

**Verbal explanation:** This is a guaranteed interview question. The way I explain it is through error handling. If I use `findElement()` and the locator is wrong or the element isn't there, my test script instantly crashes with a `NoSuchElementException`. However, if I use `findElements()`, and nothing is found, it gracefully returns a List of size zero. 
In my framework, we actually exploit this behavior. If I want to check if an element is *not* present (like verifying a warning message disappeared), using `findElement` requires a try-catch block. Instead, I use `driver.findElements().size() == 0`. It’s much cleaner and avoids exception handling overhead.

**Code Examples:**
```java
// findElement
try {
    WebElement submitBtn = driver.findElement(By.id("submit"));
    submitBtn.click();
} catch(NoSuchElementException e) {
    System.out.println("Element not present");
}

// findElements
List<WebElement> allLinks = driver.findElements(By.tagName("a"));
System.out.println("Total links on page: " + allLinks.size());
for(WebElement link : allLinks) {
    System.out.println(link.getText());
}
```

**Real-time usage:** `findElement` is for 90% of interactions (buttons, textboxes). `findElements` is for reading data from web tables, drop-downs, or checking element non-existence.


### 7. `close()` vs `quit()` 🔥

**Q: When do you use `close()` and when do you use `quit()`?** 🔥

| Feature | `close()` | `quit()` |
|---------|-----------|----------|
| **Action** | Closes only the current window/tab that the WebDriver has focus on. | Closes all open windows/tabs and safely ends the WebDriver session. |
| **Driver State**| The driver instance is still alive if other windows are open. | The driver instance is completely destroyed (`Session ID is null`). |
| **Use Case** | When you opened a child window, finished work there, and want to close just that tab. | Used in the `@AfterSuite` or `@AfterMethod` tear-down to completely clean up. |
| **Background Process**| Does not terminate the browser executable process in the background. | Kills the underlying browser driver process (like chromedriver.exe). |

**Verbal explanation:** From my experience, confusing these two leads to memory leaks on CI/CD pipelines. If you use `close()` at the end of your test, it might close the browser UI, but the `chromedriver.exe` process keeps running in the background. Do that 100 times, and your Jenkins server runs out of RAM. 
That’s why in our framework's teardown method, we *always* use `driver.quit()`. We strictly use `close()` only during window handling scenarios—like clicking a link that opens a new tab, verifying the new tab, and then calling `close()` to shut just that tab before switching back to the parent window.

**Code Example:**
```java
// Scenario for close()
String parent = driver.getWindowHandle();
driver.findElement(By.id("newTabLink")).click();
// Switch to new tab, do work
driver.close(); // Closes only the new tab
driver.switchTo().window(parent);

// Scenario for quit()
@AfterClass
public void tearDown() {
    if(driver != null) {
        driver.quit(); // Kills everything, frees memory
    }
}
```


### 8. `manage()` methods

**Q: What are the different `manage()` methods you use frequently in your framework?**
**A:** The `driver.manage()` interface is like the control panel for the browser window and session. In our framework's initialization setup, we heavily rely on it. 
First, we immediately call `manage().window().maximize()`. Testing in a minimized or random window size is a recipe for disaster because elements might get hidden under responsive UI changes, causing `ElementNotInteractable` exceptions. 
We also use `manage().timeouts().implicitlyWait()`, though we are moving towards explicit waits now. Another crucial one is `manage().deleteAllCookies()`, which we call before every test to ensure a completely clean state, avoiding session carryover between tests.

**Syntax & Code:**
```java
// Window manipulation
driver.manage().window().maximize();
driver.manage().window().minimize(); // Added in newer Selenium versions
driver.manage().window().fullscreen();

// Getting and Setting Size
Dimension size = driver.manage().window().getSize();
driver.manage().window().setSize(new Dimension(1024, 768)); // Useful for responsive testing

// Timeouts
driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));

// Cookies
driver.manage().deleteAllCookies();
```

**Real-time usage:** Setting up the test environment preconditions. We often use `setSize()` to simulate iPad or mobile resolution testing within a desktop browser.
**Important points:** `implicitlyWait` applies globally to the driver for its entire lifetime.
**Common mistakes:** Mixing implicit and explicit waits, which can cause unpredictable wait times and sluggish test execution.

---

## PART 4: WebElements

### 9. WebElement Interface

**Q: What exactly is a WebElement in Selenium?**
**A:** When interviewers ask this, they are looking for the technical definition. `WebElement` is an interface in Selenium WebDriver. It represents a single HTML node on the web page. Everything you see—buttons, text boxes, links, images—are treated as WebElements. 
In my project, we never hardcode interactions. We always locate the HTML node, store it in a `WebElement` reference, and then perform actions on it using methods declared in the `WebElement` interface like `click()`, `sendKeys()`, and `getText()`.

**Syntax:**
```java
// WebElement is the interface, findElement returns an implementation of it
WebElement loginButton = driver.findElement(By.id("loginBtn"));
```


### 10. `click()`

**Q: Explain the `click()` method and the challenges you face with it.**
**A:** The `click()` method simulates a left mouse click on an element. It sounds simple, but in real-time projects, it's the method that fails the most. 
What we typically see are two major exceptions: `ElementNotInteractableException` and `ElementClickInterceptedException`. This happens when the button is loaded in the DOM but is covered by a loading spinner, a sticky header, or a pop-up modal. 
The way I handle this is by first ensuring the element is clickable using an Explicit Wait (`ExpectedConditions.elementToBeClickable`). If standard click still fails because of an overlay, my fallback mechanism is to use a JavaScript click.

**Syntax & Code:**
```java
WebElement submitBtn = driver.findElement(By.id("submit"));
submitBtn.click();

// Real-time fallback for ElementClickInterceptedException
JavascriptExecutor js = (JavascriptExecutor) driver;
js.executeScript("arguments[0].click();", submitBtn);
```

**Real-time usage:** Clicking buttons, links, checkboxes, and radio buttons.
**Common mistakes:** Trying to click an element that is outside the current viewport. Although Selenium usually scrolls to it automatically, sometimes you have to use JS to scroll it into view first.


### 11. `sendKeys()`

**Q: How does `sendKeys()` work, and what else can it do besides typing text?**
**A:** We use `sendKeys()` primarily to simulate keyboard typing into text fields, like usernames or search bars. But a powerful feature that many people miss is that it can also simulate keyboard key presses using the `Keys` enum.
In our e-commerce project, the search bar didn't have a "Search" button. You had to type and press Enter. So, we chained it: `element.sendKeys("Laptop", Keys.ENTER)`. It’s also incredibly useful for testing keyboard accessibility, like using `Keys.TAB` to move through a form.

**Syntax & Code:**
```java
WebElement username = driver.findElement(By.name("user"));
// Standard text input
username.sendKeys("admin_user");

// Simulating Keyboard actions
WebElement searchBox = driver.findElement(By.id("search"));
searchBox.sendKeys("iPhone 15", Keys.ENTER);

// Selecting all text and deleting (Ctrl+A, Backspace)
searchBox.sendKeys(Keys.chord(Keys.CONTROL, "a"), Keys.BACK_SPACE);
```

**Real-time usage:** Filling out forms, file uploads (by sending the file path), and simulating keyboard navigation.
**Common mistakes:** Not clearing the field before sending keys. If the field already has text, `sendKeys()` will append to it, ruining your test data.


### 12. `clear()`

**Q: Why do we use `clear()`, and what is a best practice regarding its usage?**
**A:** `clear()` simply deletes the existing text inside an input box or textarea. From my experience, using `clear()` before `sendKeys()` is an absolute mandatory rule in framework design. 
Why? Because many applications have default placeholder text or pre-filled data (like an update profile page). If you just use `sendKeys("John")` on a field containing "Smith", the result will be "SmithJohn". Therefore, our custom wrapper methods in our Page Object Model always call `clear()` internally before sending text.

**Syntax & Code:**
```java
WebElement emailField = driver.findElement(By.id("email"));
emailField.clear(); // Always clear first
emailField.sendKeys("test@domain.com");
```

**Common mistakes:** Expecting `clear()` to work on non-input elements. If you use it on a `<div>` or `<span>`, it throws an `InvalidElementStateException`.


### 13. `getText()` 🔥

**Q: What is the difference between `getText()` and `getAttribute("value")`?** 🔥

| Feature | `getText()` | `getAttribute("value")` | `getAttribute("innerText")` |
|---------|-------------|-------------------------|-----------------------------|
| **Target** | Retrieves inner text between opening and closing HTML tags. | Retrieves text present inside an input field or text box. | Retrieves inner text using JavaScript, even if hidden. |
| **Visibility**| Returns empty string if the text is hidden (CSS `display:none`). | Returns the value regardless of visibility. | Can return hidden text depending on browser implementation. |
| **Tags used on**| `<p>`, `<h1>`, `<div>`, `<span>`, `<a>` | `<input>`, `<textarea>` | Any tag. |

**Verbal explanation:** I see junior automation engineers struggle with this constantly. If you want to read the text inside a paragraph or a heading, you use `getText()`. However, if you type something into a login box and want to verify what you just typed, `getText()` will return nothing. Why? Because text in input boxes is stored in the `value` attribute, not between HTML tags. In that case, you must use `getAttribute("value")`.

**Code Example:**
```java
// Scenario 1: Reading a label
WebElement welcomeMsg = driver.findElement(By.tagName("h1"));
System.out.println(welcomeMsg.getText()); // Prints "Welcome User"

// Scenario 2: Reading text typed into an input field
WebElement nameBox = driver.findElement(By.id("name"));
nameBox.sendKeys("John Doe");
System.out.println(nameBox.getText()); // Prints empty string ""
System.out.println(nameBox.getAttribute("value")); // Prints "John Doe"
```


### 14. `getAttribute()`

**Q: Explain how you use `getAttribute()` in real-time scenarios.**
**A:** `getAttribute()` allows us to read any HTML attribute of a web element. In my daily tasks, this is incredibly versatile. 
For instance, if I want to verify that an image loaded correctly, I fetch the `src` attribute. If I want to verify a hyperlink URL before clicking it, I fetch the `href` attribute. A very common test case is verifying placeholder text in an empty text box, for which I use `getAttribute("placeholder")`. Another advanced use case is checking the state of UI toggles by reading the `class` attribute to see if it contains "active" or "disabled".

**Syntax & Code:**
```java
WebElement link = driver.findElement(By.id("forgotPassword"));
String url = link.getAttribute("href");
System.out.println("Link points to: " + url);

WebElement tooltip = driver.findElement(By.id("info"));
String toolTipText = tooltip.getAttribute("title");
```

**Important points:** If the attribute does not exist, it returns `null`.


### 15. `isDisplayed()`, `isEnabled()`, `isSelected()` 🔥

**Q: Can you explain the difference between `isDisplayed()`, `isEnabled()`, and `isSelected()`?** 🔥

| Method | What it checks | When to use it | Return Type |
|--------|----------------|----------------|-------------|
| `isDisplayed()` | Checks if the element is visible on the UI. | To verify an element is not hidden by CSS (`display:none` or `visibility:hidden`). | boolean |
| `isEnabled()` | Checks if the element is enabled (can be interacted with). | To verify a button is not greyed out/disabled. | boolean |
| `isSelected()` | Checks if a radio button, checkbox, or option is ticked. | To verify the state of checkboxes or drop-down options. | boolean |

**Verbal explanation:** These three boolean methods are the backbone of our UI state validations. 
In a typical user registration flow, the "Submit" button might be visible (`isDisplayed` is true), but greyed out until the user checks the Terms and Conditions box (`isEnabled` is false). We use `isSelected()` to verify the user successfully clicked the T&C checkbox, and then we assert that `isEnabled()` on the Submit button has now become true. It's the perfect way to test dynamic UI logic.

**Code Examples:**
```java
WebElement checkbox = driver.findElement(By.id("terms"));
// Check if it's already selected before clicking
if(!checkbox.isSelected()) {
    checkbox.click();
}

WebElement submitBtn = driver.findElement(By.id("submit"));
// Verify visibility and interactivity
Assert.assertTrue(submitBtn.isDisplayed(), "Button is not visible");
Assert.assertTrue(submitBtn.isEnabled(), "Button is still disabled");
```


### 16. `submit()`

**Q: What is `submit()` and how is it different from `click()`?**

| Feature | `submit()` | `click()` |
|---------|------------|-----------|
| **Usage** | Can be called on *any* element inside a `<form>` tag. | Must be called exactly on the target button element. |
| **HTML constraint** | Only works if elements are wrapped in a `<form>`. | Works on any clickable element anywhere. |
| **Action** | Triggers the form submission event directly. | Simulates physical mouse click. |

**Verbal explanation:** To be honest, in modern automation, we rarely use `submit()`. It was very popular in older HTML architectures where forms were strictly defined. If your login fields are inside a `<form>`, you can use `submit()` on the password field itself, and it will automatically find the submit button and trigger the login. 
However, modern apps use React or Angular where forms are managed by JavaScript, and standard `<form>` tags are often missing. Therefore, we almost exclusively use `click()` on the actual button to simulate real user behavior.

**Code Example:**
```java
WebElement passwordField = driver.findElement(By.id("pass"));
passwordField.sendKeys("Secret123");
// This will submit the whole form without needing to find the login button
passwordField.submit(); 
```

---

## PART 5: Browser Handling

### 17. Browser Launch (Chrome, Firefox, Edge)

**Q: How do you handle cross-browser execution in your framework?**
**A:** In a robust framework, we don't hardcode the browser launch. What we typically do is implement a Factory Design Pattern or a simple switch-case block that reads the browser name from a properties file or a Maven command-line argument. 
With Selenium 4, browser launching has become much easier because of Selenium Manager. We no longer need to download `chromedriver.exe` manually or set `System.setProperty()`. We just instantiate the driver directly.

**Syntax & Code:**
```java
public WebDriver launchBrowser(String browserName) {
    WebDriver driver = null;
    switch (browserName.toLowerCase()) {
        case "chrome":
            driver = new ChromeDriver(); // Selenium 4 handles binary
            break;
        case "firefox":
            driver = new FirefoxDriver();
            break;
        case "edge":
            driver = new EdgeDriver();
            break;
        default:
            throw new IllegalArgumentException("Invalid browser: " + browserName);
    }
    return driver;
}
```


### 18. ChromeOptions / FirefoxOptions / EdgeOptions 🔥

**Q: What is the `ChromeOptions` class, and what configurations do you use?**
**A:** The `Options` classes are extremely powerful. They allow us to manipulate the browser's behavior before it even opens. In my project, we never launch a naked Chrome browser; we always pass a `ChromeOptions` object.
We use arguments to disable infobars (like "Chrome is being controlled by automated software"), accept insecure certificates for our QA environments, and most importantly, run tests in headless mode for our Jenkins pipelines. It's also how we handle annoying browser popups like geolocation or notification prompts by disabling them entirely.

**Code Examples:**
```java
ChromeOptions options = new ChromeOptions();

// Maximize on start
options.addArguments("--start-maximized");

// Disable annoying popups and notifications
options.addArguments("--disable-notifications");

// Run in incognito mode (great for clean sessions)
options.addArguments("--incognito");

// Accept untrusted SSL certificates
options.setAcceptInsecureCerts(true);

WebDriver driver = new ChromeDriver(options);
```

**Real-time usage:** Customizing the browser profile to ensure a stable, distraction-free environment for automated tests.


### 19. Headless Execution

**Q: What is Headless mode, and why do we use it?**
**A:** Headless mode means running the browser in the background without a graphical user interface (GUI). The browser renders the DOM and executes JavaScript just like normal, but you can't see it on your screen.
From my experience, headless execution is mandatory for CI/CD integrations. When we run our regression suite on a Linux Jenkins server, there is no display monitor attached, so GUI browsers will crash. Headless mode bypasses this. Plus, without the overhead of painting the UI, tests run significantly faster and consume less RAM.

**Syntax & Code:**
```java
ChromeOptions options = new ChromeOptions();
// The new, better headless mode in modern Chrome
options.addArguments("--headless=new"); 
// Older syntax: options.addArguments("--headless");

WebDriver driver = new ChromeDriver(options);
driver.get("https://www.google.com");
System.out.println("Title in headless: " + driver.getTitle());
```

**Important points:** Sometimes, elements that are visible in normal UI mode might get hidden in headless mode due to default responsive sizing. Always set the window size explicitly (`--window-size=1920,1080`) when running headless.


### 20. File Download Handling

**Q: How do you verify file downloads in Selenium?**
**A:** Selenium itself cannot verify a file downloaded to your Windows file system. So, what we do is a two-step process. First, we use `ChromeOptions` to change the default download directory to a specific project folder so we know exactly where the file will go. Second, after clicking the download link, we use standard Java `File` I/O operations to check if the file exists in that folder.

**Code Example:**
```java
// 1. Setup download directory via Options
String downloadPath = System.getProperty("user.dir") + "\\downloads";
HashMap<String, Object> chromePrefs = new HashMap<>();
chromePrefs.put("profile.default_content_settings.popups", 0);
chromePrefs.put("download.default_directory", downloadPath);

ChromeOptions options = new ChromeOptions();
options.setExperimentalOption("prefs", chromePrefs);
WebDriver driver = new ChromeDriver(options);

// 2. Perform download
driver.get("https://example.com/download");
driver.findElement(By.id("downloadBtn")).click();

// 3. Verify using Java (with a slight wait for download to finish)
File file = new File(downloadPath + "\\report.pdf");
if(file.exists()) {
    System.out.println("Download Successful!");
}
```


### 21. File Upload Handling 🔥

**Q: How do you automate file uploads in Selenium?**
**A:** This is a classic interview question. The answer depends entirely on the HTML structure. 
If the upload button is an `<input type="file">` tag, it's incredibly easy. We completely ignore clicking the button (which would open a Windows dialog that Selenium can't control). Instead, we directly use `sendKeys()` on the input element and pass the absolute path of the file.
However, if the element is not an input tag, or if the developers built a custom drag-and-drop widget, Selenium fails. In those cases, in my project, we integrate third-party tools like the Java `Robot` class to simulate keyboard strokes (CTRL+V the path, press Enter) or use AutoIT scripts.

**Code Example (The easy way):**
```java
// MUST be an <input type="file"> element
WebElement uploadElement = driver.findElement(By.id("uploadFile"));

// Pass absolute path directly
String filePath = "C:\\Users\\admin\\Documents\\testData.csv";
uploadElement.sendKeys(filePath);

driver.findElement(By.id("submitUpload")).click();
```


### 22. Cookies Handling

**Q: How do you handle Cookies, and why is it important in testing?**
**A:** Handling cookies is an advanced technique we use to manipulate session states. Every time you log into a website, a session token is stored in a cookie. 
In my framework, instead of going through the slow UI login process for every single test case, we log in once via API, grab the session cookie, and inject it directly into the WebDriver using `addCookie()`. Then we refresh the page, and boom—we are logged in instantly. It saves minutes of execution time. Conversely, we use `deleteAllCookies()` during teardown to ensure the next test starts clean.

**Code Example:**
```java
// Deleting all cookies
driver.manage().deleteAllCookies();

// Getting cookies
Set<Cookie> allCookies = driver.manage().getCookies();
for(Cookie c : allCookies) {
    System.out.println(c.getName() + " : " + c.getValue());
}

// Adding a custom cookie to bypass login
Cookie sessionCookie = new Cookie("session_id", "12345XYZ");
driver.manage().addCookie(sessionCookie);
driver.navigate().refresh(); // Refresh to apply cookie
```

---

## 23. Top Interview Questions Recap 🔥

Here is a rapid-fire summary of how to answer the most critical questions:

1. **`get()` vs `navigate().to()`?** Both wait for page load. `navigate` allows history manipulation (back/forward). `get` is simpler for initial launch.
2. **`close()` vs `quit()`?** `close` shuts the current focused tab. `quit` kills the entire browser session and driver executable to prevent memory leaks. Always use `quit` in teardown.
3. **`findElement()` vs `findElements()`?** `findElement` returns one element and crashes (`NoSuchElementException`) if not found. `findElements` returns a list and gracefully returns empty size 0 if not found.
4. **`getText()` vs `getAttribute("value")`?** `getText` reads inner HTML text (like paragraphs). `getAttribute("value")` reads text typed inside an input box.
5. **How to handle file upload?** Use `sendKeys("path")` if it's an `<input type='file'>`. Otherwise, use Robot class or AutoIT.
6. **How to run headless?** Use `ChromeOptions` and `addArguments("--headless=new")`. Add window size argument to avoid element hidden issues.
7. **What ChromeOptions do you use?** `--start-maximized`, `--incognito`, `--disable-notifications`, and setting experimental options for default download paths.
8. **isDisplayed vs isEnabled vs isSelected?** `isDisplayed` (visibility check), `isEnabled` (disabled/greyed out check), `isSelected` (checkbox/radio button check).
