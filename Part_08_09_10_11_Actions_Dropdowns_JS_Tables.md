# PART 8: MOUSE & KEYBOARD (Actions Class)

## 1. What is Actions Class?
**Context:** In Selenium, regular `click()` and `sendKeys()` methods are meant for basic interactions. But modern web applications have complex user interactions like hover menus, drag-and-drop, right-clicks, and keyboard shortcuts. 
**Usage:** The `Actions` class in Selenium provides advanced user interaction APIs. We import it from `org.openqa.selenium.interactions.Actions`.
**Initialization:**
```java
Actions actions = new Actions(driver);
```
**`build()` and `perform()`:**
- `perform()` executes the action directly. If you have a single action, you can just call `perform()`.
- `build()` compiles multiple actions into a single sequence (a `CompositeAction`). It's a good practice to use `build().perform()` when chaining multiple actions, though `perform()` internally calls `build()` anyway.

## 2. Mouse Hover
**Context:** Many websites have navigation menus where sub-menus appear only when you hover your mouse over the main menu item.
**Code Example:**
```java
WebElement electronicsMenu = driver.findElement(By.id("nav-electronics"));
Actions actions = new Actions(driver);
// Hovering over the element
actions.moveToElement(electronicsMenu).perform();

// Now click the sub-menu that appeared
WebElement laptopsSubMenu = driver.findElement(By.id("category-laptops"));
laptopsSubMenu.click();
```

## 3. Right Click (Context Click)
**Context:** Applications often have custom context menus that open upon right-click, replacing the default browser menu.
**Code Example:**
```java
WebElement rightClickBtn = driver.findElement(By.id("right-click-zone"));
Actions actions = new Actions(driver);
actions.contextClick(rightClickBtn).perform();

// Click an option in the context menu
WebElement editOption = driver.findElement(By.xpath("//li[text()='Edit']"));
editOption.click();
```

## 4. Double Click
**Context:** Some features require double-clicking to edit a file name or select text.
**Code Example:**
```java
WebElement fileElement = driver.findElement(By.id("file-123"));
Actions actions = new Actions(driver);
actions.doubleClick(fileElement).perform();
```

## 5. Drag and Drop
**Context:** Dragging an element from one container to another, common in Kanban boards like Trello.
**Code Example:**
```java
WebElement sourceElement = driver.findElement(By.id("draggable"));
WebElement targetElement = driver.findElement(By.id("droppable"));
Actions actions = new Actions(driver);

// Approach 1: Direct method
actions.dragAndDrop(sourceElement, targetElement).perform();

// Approach 2: Alternative using chain
actions.clickAndHold(sourceElement)
       .moveToElement(targetElement)
       .release()
       .build().perform();
```
**Fallback:** Sometimes HTML5 drag and drop doesn't work with Actions class. In my projects, we often inject a JavaScript helper to simulate HTML5 drag-and-drop events when Selenium fails.

## 6. Click and Hold
**Context:** Useful for drawing applications, selecting a range on a slider, or building a custom drag-and-drop.
**Code Example:**
```java
WebElement sliderKnob = driver.findElement(By.id("slider-knob"));
Actions actions = new Actions(driver);
actions.clickAndHold(sliderKnob).moveByOffset(50, 0).release().build().perform();
```

## 7. Keyboard Actions
**Context:** Simulating keyboard keys like ENTER, TAB, or combos like CTRL+A.
**Code Examples:**
```java
WebElement inputField = driver.findElement(By.id("search"));
Actions actions = new Actions(driver);

// Press Enter
actions.sendKeys(inputField, "Selenium").sendKeys(Keys.ENTER).perform();

// Select All (CTRL+A)
actions.keyDown(Keys.CONTROL).sendKeys("a").keyUp(Keys.CONTROL).perform();

// Copy and Paste (CTRL+C, CTRL+V)
actions.keyDown(Keys.CONTROL).sendKeys("c").keyUp(Keys.CONTROL).build().perform();
WebElement anotherField = driver.findElement(By.id("target"));
anotherField.click();
actions.keyDown(Keys.CONTROL).sendKeys("v").keyUp(Keys.CONTROL).build().perform();

// TAB to next field
actions.sendKeys(Keys.TAB).perform();
// SHIFT+TAB to previous
actions.keyDown(Keys.SHIFT).sendKeys(Keys.TAB).keyUp(Keys.SHIFT).perform();

// Escape to close a modal
actions.sendKeys(Keys.ESCAPE).perform();
```

## 8. Interview Questions on Actions

**Q: Explain how you handle Mouse Hover actions in your current framework.** 🔥
**A:** "In my current project, we have a complex navigation header where dropdowns only appear when a user hovers over the parent category. For instance, hovering over 'Products' reveals 'Software' and 'Hardware'. To handle this reliably, I use the `Actions` class provided by Selenium. I initialize it passing the WebDriver instance, then use the `moveToElement(element)` method followed by `.perform()`. 

What we typically do in our framework is abstract this into a utility method, say `hoverOverElement(WebElement element)`. Inside, before doing the hover, we wait for the parent element to be visible using an Explicit Wait. After executing `actions.moveToElement(element).perform()`, we add another Explicit Wait for the child element (the sub-menu) to become visible and clickable. This avoids `ElementNotInteractableException`. If the standard hover flakes out due to rendering issues, which happens rarely but does occur, we use JavaScript to forcefully change the CSS display property of the sub-menu, but standard `moveToElement` is always our primary approach."

**Q: How do you perform Drag and Drop in Selenium, and what do you do if it fails?** 🔥
**A:** "For drag and drop, the standard approach I use is the `Actions` class. I locate both the source web element and the target web element. Then I call `actions.dragAndDrop(source, target).perform()`. If that behaves inconsistently, I use the chained method: `actions.clickAndHold(source).moveToElement(target).release().build().perform()`. 

However, from my experience, especially with modern React or Angular applications using HTML5 drag-and-drop attributes, Selenium's native `dragAndDrop` sometimes totally fails to trigger the drop event. The element moves but doesn't snap into place. When I face this in my projects, the bulletproof workaround is to use JavaScript. I execute a custom JS script via `JavascriptExecutor` that simulates the `dragstart`, `dragenter`, `dragover`, `drop`, and `dragend` events directly on the DOM nodes. We actually keep this JS script as a constant in our framework utilities specifically for stubborn drag-and-drop components."

**Q: Can you explain the difference between `build()` and `perform()` in the Actions class?** 🔥

| Feature | `build()` | `perform()` |
|---------|-----------|-------------|
| **Purpose** | Compiles multiple actions into a single `CompositeAction`. | Executes the compiled action(s) on the browser. |
| **Returns** | `Action` object. | `void` |
| **Requirement** | Optional if you are just executing one action or don't need the object. | Mandatory to actually trigger the action in the UI. |

**Verbal explanation:** "When I'm explaining this to junior SDETs, I tell them to think of `build()` as 'compiling' or 'packaging' the steps, and `perform()` as 'clicking the run button'. If you are chaining multiple actions—like pressing Control, typing 'C', and releasing Control—you use `build()` to package them into one `Action` sequence, and then `perform()` to execute it. Interestingly, if you just call `perform()` at the end of a chain, it internally calls `build()` for you. So `actions.moveToElement(el).click().perform()` works perfectly. But best practice in our team is to use `.build().perform()` for composite actions to make the intent clear that multiple steps are being bundled."

**Q: Have you ever used Keyboard combinations like CTRL+A or Shift+Click using Selenium? How?** 🔥
**A:** "Absolutely. In our ERP project, users do a lot of bulk copy-pasting across grid cells, so simulating keyboard shortcuts is critical. To do this, I use the `Actions` class along with the `Keys` enum. For something like Select All (CTRL+A), I write: `actions.keyDown(Keys.CONTROL).sendKeys("a").keyUp(Keys.CONTROL).build().perform();`. 

The most important part here is the `keyUp()` method. If you forget to release the modifier key (like CONTROL or SHIFT), Selenium acts as if the key is physically stuck down for the remainder of the session, and all subsequent clicks or types will behave erratically (like opening links in new tabs unexpectedly). To ensure robustness, I usually wrap these keyboard shortcuts in a `try-finally` block, putting the `keyUp` in the `finally` block to guarantee the key is released even if the action fails halfway."

**Q: How do you handle sliders or drawing on a canvas in Selenium?**
**A:** "Handling sliders or canvas elements requires precise coordinate-based mouse interactions. In my last e-commerce project, we had a price range slider. Standard `sendKeys` or `click` doesn't work here. I used the `Actions` class with offset movements. 

First, I locate the slider knob element. Then I use `actions.clickAndHold(sliderKnob).moveByOffset(xOffset, yOffset).release().build().perform();`. The tricky part is calculating the `xOffset`. We usually get the width of the entire slider track, calculate the percentage we want to slide, and derive the pixel offset. For drawing on an HTML5 `<canvas>`, it's a similar approach but moving the mouse along a path. We use `actions.moveToElement(canvas, startX, startY).clickAndHold().moveByOffset(dx, dy).release().build().perform()`. We had a signature pad feature where we automated a 'scribble' using this exact sequence of offsets to ensure the canvas registered the drawing correctly."


# PART 9: DROPDOWNS

## 9. HTML Select Dropdown
**Context:** Standard dropdowns are created using the `<select>` tag in HTML, with child `<option>` tags. Selenium provides a specialized `Select` class for this.
**Methods:**
- `selectByVisibleText("text")` - best and most common.
- `selectByValue("value")` - uses the `value` attribute.
- `selectByIndex(index)` - 0-based, avoid if options are dynamic.
- `getOptions()` - returns all `<option>` elements.
- `getFirstSelectedOption()` - gets the currently selected item.
**Code Example:**
```java
WebElement dropdownElement = driver.findElement(By.id("country-select"));
Select countryDropdown = new Select(dropdownElement);

// Selecting
countryDropdown.selectByVisibleText("India");
countryDropdown.selectByValue("USA");
countryDropdown.selectByIndex(2);

// Getting all options
List<WebElement> allOptions = countryDropdown.getOptions();
for(WebElement option : allOptions) {
    System.out.println(option.getText());
}
```

## 10. Multi-Select Dropdown
**Context:** A `<select>` tag with the `multiple` attribute allows selecting more than one option.
**Code Example:**
```java
WebElement multiSelectEl = driver.findElement(By.id("skills"));
Select skills = new Select(multiSelectEl);

if(skills.isMultiple()) {
    skills.selectByVisibleText("Java");
    skills.selectByVisibleText("Selenium");
    
    // Deselecting
    skills.deselectByVisibleText("Java");
    skills.deselectAll();
}
```

## 11. Custom Dropdown (Non-Select)
**Context:** Modern UI frameworks (React/Angular/Bootstrap) use `<div>`, `<ul>`, `<li>` to create beautiful dropdowns. The `Select` class throws an exception if used on these.
**Strategy:** Click to open -> Wait for options -> Iterate and click the desired text.
**Code Example:**
```java
// 1. Click the dropdown to expand
driver.findElement(By.id("custom-dropdown-btn")).click();

// 2. Wait for options to be visible
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
List<WebElement> options = wait.until(ExpectedConditions.visibilityOfAllElementsLocatedBy(
    By.xpath("//ul[@class='dropdown-menu']/li")
));

// 3. Iterate and select
for(WebElement option : options) {
    if(option.getText().equals("Premium Plan")) {
        option.click();
        break;
    }
}
```

## 12. Dynamic/Auto-Suggest Dropdown
**Context:** Like Google Search. You type, it fetches suggestions dynamically.
**Strategy:** Type text -> Wait for auto-suggest list -> Iterate -> Click match.
**Code Example:**
```java
// 1. Type partial text
driver.findElement(By.id("search-box")).sendKeys("Sel");

// 2. Wait for suggestions to populate
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
List<WebElement> suggestions = wait.until(ExpectedConditions.visibilityOfAllElementsLocatedBy(
    By.xpath("//ul[@class='suggestions-list']/li")
));

// 3. Find and click the exact match
for(WebElement suggestion : suggestions) {
    if(suggestion.getText().equalsIgnoreCase("Selenium WebDriver")) {
        suggestion.click();
        break;
    }
}
```

## 13. Interview Questions on Dropdowns

**Q: How do you handle dropdowns in Selenium? What if it's not a `<select>` tag?** 🔥
**A:** "The way I handle dropdowns completely depends on the underlying HTML structure. If the developers used a standard HTML `<select>` tag, it's very straightforward. I instantiate Selenium's `Select` class, pass the WebElement to its constructor, and use methods like `selectByVisibleText()` or `selectByValue()`. I always prefer visible text because it aligns with what the user actually sees on the screen.

However, in my current project which uses React and Material UI, almost none of our dropdowns are `<select>` tags. They are custom dropdowns built with `<div>`, `<ul>`, and `<li>` elements to allow for complex styling. If I pass these to the `Select` class, Selenium throws an `UnexpectedTagNameException`. For these custom dropdowns, my strategy is manual simulation. First, I write code to click the dropdown trigger element to expand it. Then, I apply an Explicit Wait to ensure the list of options (`<li>` elements) is fully visible. Finally, I grab all the options into a `List<WebElement>`, loop through them, extract the text using `getText()`, and when it matches my target value, I click it and `break` the loop. We usually encapsulate this logic into a reusable utility method in our Base Page class."

**Q: How would you verify that a dropdown contains a specific set of options (e.g., verifying all countries are listed)?** 🔥
**A:** "Validating dropdown contents is a common test case we automate. If it's a standard `<select>` dropdown, I use the `Select` class and call the `getOptions()` method, which returns a `List<WebElement>` representing all the `<option>` tags. 

What we typically do is create an expected `List<String>` containing the country names we expect. Then, I iterate through the `List<WebElement>` from `getOptions()`, use `getText()` on each, and add them to an actual `List<String>`. Once I have both lists, I compare them using TestNG or JUnit assertions, like `Assert.assertEquals(actualList, expectedList)`. 

A critical edge case we handle here is ordering. Sometimes the business requirement dictates that the countries must be in alphabetical order. In that case, I take the actual list, create a copy, sort the copy using `Collections.sort()`, and assert that the original actual list exactly matches the sorted copy. If it's a custom dropdown, the logic is identical, except instead of `getOptions()`, I use `driver.findElements()` to fetch the `<li>` elements directly."

**Q: What is an auto-suggest dropdown and how do you automate it?** 🔥
**A:** "An auto-suggest dropdown, like the Google search bar or a flight booking origin/destination field, dynamically loads options based on the keystrokes you enter. It doesn't load all options on page load. 

In my project, we have a 'Search Employee' field that behaves this way. To automate it, I first use `sendKeys()` to type a partial string, say 'Smi'. The tricky part here is synchronization. As soon as you type, the DOM changes and an AJAX call fetches the suggestions. If you immediately try to find the suggestions, you'll get a `NoSuchElementException` or an empty list. So, I strictly implement an Explicit Wait using `visibilityOfAllElementsLocatedBy()` to wait for the suggestion `<li>` items to appear. Once they render, I capture them into a `List<WebElement>`, loop through them, and when I find the specific employee name, like 'John Smith', I click it and break out of the loop. If the network is slow, this wait is the only thing preventing a flaky test."

**Q: What happens if you use `selectByVisibleText()` but the text has leading or trailing spaces in the HTML?**
**A:** "This is a fantastic question and a very real pain point I've faced. The `selectByVisibleText()` method in Selenium looks for an EXACT match of the text. If the HTML developer accidentally left a space, like `<option>  India </option>`, and you script `selectByVisibleText("India")`, Selenium will throw a `NoSuchElementException` because 'India' does not strictly equal '  India '.

When we encounter this in my team, we have a couple of workarounds. The easiest is to use `selectByValue()` instead, because the `value` attribute usually doesn't have accidental spaces. If there is no `value` attribute, I avoid `selectByVisibleText()`. Instead, I get all options using `getOptions()`, loop through them, and use the `String.trim()` or `String.contains()` method in Java. For example: `if(option.getText().trim().equals("India")) { option.click(); }`. This makes the framework highly resilient to minor UI text formatting issues."

**Q: Explain how to check if a dropdown supports multiple selections.**
**A:** "With standard HTML `<select>` tags, you can have a single-select or multi-select dropdown. Multi-select dropdowns have the `multiple` attribute in their HTML tag. 

In Selenium, the `Select` class provides a built-in method called `isMultiple()`. It returns a boolean. In our framework, before we attempt to use methods like `deselectAll()` or select multiple items in a row, we usually wrap it in an `if(select.isMultiple())` check. If you try to call `deselectAll()` on a single-select dropdown, Selenium will throw an `UnsupportedOperationException`. Using `isMultiple()` is a best practice to ensure your helper methods don't crash when passed the wrong type of dropdown element."


# PART 10: JavaScriptExecutor ⭐⭐⭐

## 14. What is JavaScriptExecutor?
**Context:** Sometimes Selenium's native commands fail. A button might be covered by a sticky header, or an element might not be "interactable" according to WebDriver specs. `JavascriptExecutor` is an interface that allows us to execute raw JavaScript directly in the context of the browser.
**Usage:**
```java
JavascriptExecutor js = (JavascriptExecutor) driver;
// Synchronous execution
js.executeScript("script", arguments);
// Asynchronous execution
js.executeAsyncScript("script", arguments);
```

## 15. Scroll Operations
**Context:** Selenium doesn't always automatically scroll to an element before interacting with it (especially in modern apps). We use JS for precise scrolling.
**Code Examples:**
```java
JavascriptExecutor js = (JavascriptExecutor) driver;

// 1. Scroll by specific pixels (Down)
js.executeScript("window.scrollBy(0, 500)");

// 2. Scroll to the very bottom of the page
js.executeScript("window.scrollTo(0, document.body.scrollHeight)");

// 3. Scroll to the very top
js.executeScript("window.scrollTo(0, 0)");

// 4. Scroll until an element is in view (Most common!)
WebElement submitBtn = driver.findElement(By.id("submit"));
js.executeScript("arguments[0].scrollIntoView(true);", submitBtn);
```

## 16. Click using JavaScript
**Context:** When you get `ElementClickInterceptedException` (e.g., a loader or chat bot is overlapping your button), JS click bypasses the UI layer and triggers the click event directly on the DOM node.
**Code Example:**
```java
WebElement hiddenButton = driver.findElement(By.id("hidden-btn"));
JavascriptExecutor js = (JavascriptExecutor) driver;
js.executeScript("arguments[0].click();", hiddenButton);
```

## 17. Enter Text using JavaScript
**Context:** If `sendKeys()` is extremely slow (sometimes happens in IE/Edge) or the field is strictly readonly but you need to inject test data.
**Code Example:**
```java
WebElement dateField = driver.findElement(By.id("date"));
JavascriptExecutor js = (JavascriptExecutor) driver;
// Sets the value attribute directly
js.executeScript("arguments[0].value='2023-12-01';", dateField);
```

## 18. Other JS Operations
**Context:** JS can extract data or manipulate the DOM for debugging.
**Code Examples:**
```java
JavascriptExecutor js = (JavascriptExecutor) driver;

// Get Title
String title = js.executeScript("return document.title;").toString();

// Generate an alert
js.executeScript("alert('Automation in progress!');");

// Highlight an element (great for demos or debugging)
WebElement element = driver.findElement(By.id("logo"));
js.executeScript("arguments[0].style.border='3px solid red'", element);
```

## 19. When NOT to Use JavaScriptExecutor
**Important:** JSExecutor bypasses the browser's normal event pipeline. It doesn't care if an element is hidden, disabled, or overlapped. If you use JS to click a hidden button, it will work. But a REAL user cannot click a hidden button! Overusing JSExecutor hides real UI bugs. Use it ONLY as a last resort when Selenium's native methods fail due to automation anomalies, not application bugs.

## 20. Interview Questions on JSExecutor

**Q: What is JavaScriptExecutor and when do you use it in your framework?** 🔥
**A:** "`JavascriptExecutor` is an interface in Selenium that allows us to inject and execute raw JavaScript code directly into the browser from our Java code. We have to cast our WebDriver instance to `JavascriptExecutor` to use it. 

In my project, we strictly use it as a 'fallback' or 'last resort'. The most common scenario is dealing with the `ElementClickInterceptedException`. For example, we have a sticky chat widget on our website. Sometimes, when Selenium tries to click a 'Save' button near the bottom right, the chat widget overlaps it during the scroll, and native `.click()` fails. Since a real user could just scroll slightly to fix this, it's an automation timing issue, not an app bug. In this case, I catch the exception and use `js.executeScript("arguments[0].click();", element)`. I also use it heavily for scrolling elements into view using `scrollIntoView(true)` before interacting with them, especially in infinite-scroll grids. I also use it to set values in custom date pickers where normal `sendKeys()` is blocked by readonly attributes."

**Q: Can you explain the difference between `executeScript()` and `executeAsyncScript()`?** 🔥

| Feature | `executeScript()` | `executeAsyncScript()` |
|---------|-------------------|------------------------|
| **Execution Flow** | Synchronous. Blocks WebDriver until the JS finishes executing. | Asynchronous. WebDriver doesn't wait; script runs in background. |
| **Use Case** | DOM manipulation, simple clicks, scrolling, returning immediate values. | Waiting for an AJAX call to finish, handling setTimeout, long-polling. |
| **Callback** | Returns value directly via `return` statement in JS. | Requires a special callback function (provided by Selenium) to signal completion. |

**Verbal explanation:** "In 99% of my daily tasks, I use `executeScript()`. It is synchronous. When I call it, my Java execution pauses, the browser runs the JS snippet (like a click or scroll), and once done, Java continues to the next line. If I include a `return` keyword in the script, it passes that value back to Java.

`executeAsyncScript()` is different. We use it when the JavaScript operation might take a long time, like an intricate AJAX fetch request, and we don't want to freeze the browser session. With async, Selenium passes a callback function as the last argument to our JS code. Our JS code must explicitly invoke this callback when it finishes its asynchronous work to tell WebDriver, 'Hey, I'm done'. If we don't, it will hit a timeout. Honestly, with modern Explicit Waits, the need for `executeAsyncScript()` has dropped drastically, but it's crucial for complex client-side performance testing."

**Q: How do you scroll down to a specific element using Selenium?** 🔥
**A:** "Selenium natively tries to scroll an element into the viewport when you attempt to click or type into it. But often, especially with lazy-loading elements or sticky headers, it miscalculates and scrolls the element right under a floating header. 

To handle this perfectly, I rely on `JavascriptExecutor`. First, I identify the WebElement using standard locators. Then, I execute the script: `js.executeScript("arguments[0].scrollIntoView(true);", element)`. The `true` parameter aligns the top of the element with the top of the viewport. If I find that it hides behind a top sticky navigation bar, I change it to `false`, which aligns the bottom of the element with the bottom of the viewport, keeping it clear of the top header. In our framework, we built a custom wrapper method `scrollToElement()` that incorporates this logic so the team doesn't have to write JS code repeatedly."

**Q: Is it a good practice to always use JavaScript click instead of Selenium click?** 🔥
**A:** "Absolutely not. In fact, it is considered an anti-pattern to use JS click as the primary interaction method. The way I explain it to my team is this: Selenium's native `click()` is designed to mimic a real human. It checks if the element is visible, enabled, and actually reachable (not covered by another element). If a developer breaks the UI and places a giant transparent `<div>` over the whole page, a real user cannot click the buttons. Selenium's native click will correctly fail, catching the bug!

If you use `JavascriptExecutor` to click, it goes straight to the DOM layer and forces the click event. It completely ignores visibility or overlapping elements. The click will succeed, your test will pass, but in production, real users won't be able to click the button. Therefore, I only use JS click to bypass automation-specific anomalies—like a loader that vanishes slightly slower than our wait condition anticipates—but never as a default."

**Q: How can you highlight an element using Selenium? Why would you do that?**
**A:** "Highlighting an element is done using `JavascriptExecutor` to dynamically alter the CSS `style` attribute of the element. The code looks like this: `js.executeScript("arguments[0].style.border='3px solid red'", element);`.

I use this primarily for two reasons. First, during debugging. If my script is interacting with the wrong element because of a fuzzy XPath, highlighting it makes it instantly visually obvious on the screen during headed execution. Second, for reporting and client demos. In my previous project, we were capturing screenshots on failure. I added a listener that would intercept the failure, highlight the element that caused the issue in red, and *then* take the screenshot. This made the execution reports incredibly easy to read for manual testers and product owners, as their eyes were drawn exactly to where the automation failed."


# PART 11: WEB TABLES

## 21. What is a WebTable?
**Context:** Web tables display tabular data (rows and columns). Understanding HTML table structure is mandatory for XPath creation.
**Structure:**
- `<table>` - The main wrapper
- `<thead>` - Table header section
- `<tbody>` - Table body section
- `<tr>` - Table Row
- `<th>` - Table Header Cell
- `<td>` - Table Data (Column Cell)
**Static vs Dynamic:** Static tables have a fixed number of rows/cols. Dynamic tables grow/shrink (like an inbox or user directory), requiring dynamic XPaths.

## 22. Getting All Rows
**Context:** You need to know how many records are currently displayed.
**Code Example:**
```java
// Locating all rows inside the table body
List<WebElement> rows = driver.findElements(By.xpath("//table[@id='customers']/tbody/tr"));
System.out.println("Total rows: " + rows.size());
```

## 23. Getting All Columns
**Context:** Finding out how many columns are in a row.
**Code Example:**
```java
// Locating all columns (td) inside the first row
List<WebElement> cols = driver.findElements(By.xpath("//table[@id='customers']/tbody/tr[1]/td"));
System.out.println("Total columns: " + cols.size());
```

## 24. Extracting Complete Table Data
**Context:** Reading the whole table, perhaps to verify it against a database query.
**Code Example:**
```java
int rowCount = driver.findElements(By.xpath("//table/tbody/tr")).size();
int colCount = driver.findElements(By.xpath("//table/tbody/tr[1]/td")).size();

// Outer loop for rows
for (int r = 1; r <= rowCount; r++) {
    // Inner loop for columns
    for (int c = 1; c <= colCount; c++) {
        // Construct dynamic XPath
        String xpath = "//table/tbody/tr[" + r + "]/td[" + c + "]";
        String cellData = driver.findElement(By.xpath(xpath)).getText();
        System.out.print(cellData + " | ");
    }
    System.out.println(); // Next line after each row
}
```

## 25. Finding Specific Cell Value
**Context:** Checking if a specific row and column contains expected text.
**Code Example:**
```java
// Get the company name in the 3rd row, 2nd column
String company = driver.findElement(By.xpath("//table/tbody/tr[3]/td[2]")).getText();
System.out.println("Company is: " + company);
```

## 26. Click Action Based on Cell Value
**Context:** The most common interview scenario. "Find the row where the name is 'John', and click the 'Delete' button in THAT specific row."
**Code Example:**
```java
int rowCount = driver.findElements(By.xpath("//table/tbody/tr")).size();

for (int r = 1; r <= rowCount; r++) {
    // Assuming Name is in column 1
    String nameXPath = "//table/tbody/tr[" + r + "]/td[1]";
    String name = driver.findElement(By.xpath(nameXPath)).getText();
    
    if (name.equals("John")) {
        // Assuming Delete button is in column 4 of the SAME row
        String actionXPath = "//table/tbody/tr[" + r + "]/td[4]/button";
        driver.findElement(By.xpath(actionXPath)).click();
        break; // Stop searching once found
    }
}
```

## 27. Pagination in Tables
**Context:** If the table has 100 records but only shows 10 per page.
**Code Example:**
```java
boolean found = false;
while (!found) {
    // Search current page
    List<WebElement> names = driver.findElements(By.xpath("//table/tbody/tr/td[1]"));
    for(WebElement nameElement : names) {
        if(nameElement.getText().equals("TargetName")) {
            System.out.println("Found it!");
            found = true;
            break;
        }
    }
    
    if (!found) {
        // Check if Next button is enabled
        WebElement nextBtn = driver.findElement(By.id("next-page"));
        if(nextBtn.isEnabled() && !nextBtn.getAttribute("class").contains("disabled")) {
            nextBtn.click();
            // Add wait here for new page to load
        } else {
            System.out.println("Reached end of table. Name not found.");
            break;
        }
    }
}
```

## 28. Interview Questions on Tables

**Q: How do you handle dynamic WebTables in Selenium?** 🔥
**A:** "Dynamic web tables are tables where the row count and data constantly change, like a live list of recent orders. Handling them effectively relies completely on robust XPath strategies rather than hardcoded indexes. 

In my framework, what we typically do is first capture the total number of rows dynamically using `driver.findElements(By.xpath("//table/tbody/tr")).size()`. Then, we iterate through the rows using a `for` loop. Inside the loop, I construct a dynamic XPath using the loop counter variable. For example: `"//table/tbody/tr[" + i + "]/td[1]"`. I extract the text of that cell. Once the text matches my expected value, I perform the necessary action—like clicking a checkbox or reading sibling data—and then I execute a `break` statement. Breaking the loop is critical; otherwise, Selenium will continue searching, wasting execution time or throwing a `StaleElementReferenceException` if clicking the row caused a DOM refresh."

**Q: In a table, how would you click a 'Delete' button for a specific user, say 'Alex', if the row position changes every day?** 🔥
**A:** "This is a classic scenario I face often in admin portals. Since Alex's row is dynamic, I cannot use an absolute XPath like `tr[5]`. 

There are two approaches. The loop approach: I loop through all rows, check the text of column 1 (Name), and if it equals 'Alex', I construct the XPath for the button in the same row, like `tr[i]/td[5]/button`, and click it. 

However, my preferred approach as a senior SDET is using a single, powerful custom XPath using axes, completely avoiding loops. I would write: `//td[text()='Alex']/following-sibling::td//button[text()='Delete']`. Alternatively, if I want the exact row: `//tr[td[text()='Alex']]//button`. This tells WebDriver: 'Find the `<tr>` that contains a `<td>` with text Alex, and then inside that specific `<tr>`, find the button'. This is much faster and cleaner than writing a 10-line Java loop, and it executes entirely inside the browser's XPath engine."

**Q: How do you handle WebTables with Pagination?** 🔥
**A:** "In a recent project, we had an Employee directory with thousands of records, paginated to 50 per page. If I need to verify a newly added employee, they might be on page 3. 

To handle this, I use a `while(true)` loop. Inside the loop, I first search the currently visible table rows. If I find the target text, I perform my validation and `break` out of the while loop. If I finish checking the page and don't find the text, I look for the 'Next Page' pagination button. I check if it is displayed and enabled. If it is, I click it, apply an Explicit Wait for the table data to refresh (usually waiting for the table loader overlay to disappear), and let the `while` loop run again. If the 'Next Page' button is disabled (meaning I'm on the last page), I break the loop and throw a custom `DataNotFoundException` to fail the test. It's an elegant way to traverse the whole dataset."

**Q: You extracted text from a cell, but it contains extra spaces or newline characters. How do you assert it correctly?**
**A:** "This happens frequently when developers use nested `<span>` or `<br>` tags inside a table cell `<td>`. When you call `getText()`, Selenium returns everything, including the invisible formatting, which causes `Assert.assertEquals("Expected", actualText)` to fail. 

When I run into this, I never change the Expected data to include ugly spaces. Instead, I clean the Actual data. First, I use `.trim()` to remove leading and trailing whitespace. If there are internal line breaks or excessive spaces, I use Regex. `actualText.replaceAll("\\s+", " ").trim()` replaces multiple spaces or newlines with a single space. By normalizing the text extracted from the table before comparing it to my test data, the assertions become highly stable."

**Q: What is a `StaleElementReferenceException` and why is it so common with WebTables?** 🔥
**A:** "This exception is extremely common when dealing with tables. A `StaleElementReferenceException` occurs when you have located a WebElement and stored it in a variable, but before you interact with it, the DOM refreshes or updates. The element is no longer 'attached' to the current DOM structure.

With tables, this usually happens when you are iterating through rows. For instance, if you are looping through table rows to find 'John', and you click a 'Sort' header or a 'Next Page' button, the entire table body is re-rendered by the frontend framework (like React). If your loop tries to access the next element in your previously created `List<WebElement> rows`, Selenium throws this exception. To resolve it in my framework, I avoid storing lists of elements if the page might refresh. Instead, I re-fetch the list, or use dynamic XPaths constructed with strings so that `driver.findElement()` is freshly evaluated on every single iteration of the loop."
