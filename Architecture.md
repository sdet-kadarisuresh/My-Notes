Selenium 3 vs Selenium 4
Table of Contents
Introduction
What is Selenium?
Selenium Architecture
Selenium 3 Architecture
Selenium 3 Request and Response Flow
Selenium 4 Architecture
Selenium 4 Request and Response Flow
Selenium 3 vs Selenium 4
JSON Wire Protocol
W3C WebDriver Protocol
Browser Drivers
Selenium Grid 3 vs Grid 4
Relative Locators
New Tab and Window Handling
DevTools Integration
Element Screenshots
Complete Selenium 4 Architecture
Advantages of Selenium 4
Interview Questions
Quick Revision
Introduction
Selenium is an open-source framework used for automating web browsers.

Selenium WebDriver allows automation scripts to interact with web applications just like a real user.

It supports multiple programming languages and browsers.

Popular Programming Languages
Java
Python
C#
JavaScript
Ruby
Popular Browsers
Google Chrome
Mozilla Firefox
Microsoft Edge
Safari
What is Selenium?
Selenium is mainly used for:

Web application automation
Functional testing
Regression testing
Cross-browser testing
Parallel test execution
End-to-end testing
Selenium itself does not directly control the browser.

Instead, Selenium communicates with a browser driver, and the browser driver communicates with the browser.

Test Script
     |
     v
Selenium WebDriver
     |
     v
Browser Driver
     |
     v
Web Browser

Selenium Architecture
Selenium WebDriver follows a client-server style architecture.

The main components are:

+----------------------+
|     Test Script      |
|  Java / Python / C#  |
+----------+-----------+
           |
           v
+----------------------+
| Selenium Client      |
|      Library         |
+----------+-----------+
           |
           v
+----------------------+
|    WebDriver         |
|       API            |
+----------+-----------+
           |
           v
+----------------------+
|   Browser Driver     |
| ChromeDriver         |
| GeckoDriver          |
| EdgeDriver           |
+----------+-----------+
           |
           v
+----------------------+
|       Browser        |
| Chrome / Firefox /   |
| Edge / Safari        |
+----------------------+

Selenium 3 Architecture
Selenium 3 primarily used the JSON Wire Protocol for communication between the Selenium client and browser drivers.

Architecture Diagram
+----------------------+
|     Test Script      |
|       Java           |
+----------+-----------+
           |
           v
+----------------------+
| Selenium Client      |
|      Library         |
+----------+-----------+
           |
           v
+----------------------+
| JSON Wire Protocol   |
+----------+-----------+
           |
           v
+----------------------+
|   Browser Driver     |
|                      |
| ChromeDriver         |
| GeckoDriver          |
| IEDriverServer       |
+----------+-----------+
           |
           v
+----------------------+
|       Browser        |
|                      |
| Chrome / Firefox /   |
| Internet Explorer   |
+----------------------+

Components
1. Test Script
The tester writes automation code using a programming language.

Example:

WebDriver driver = new ChromeDriver();

driver.get("https://example.com");

2. Selenium Client Library
The Selenium client library provides WebDriver APIs.

For example:

driver.get();
driver.findElement();
driver.click();
driver.sendKeys();
driver.quit();

3. JSON Wire Protocol
Selenium 3 used the JSON Wire Protocol for communication.

The Selenium client converted commands into HTTP requests.

4. Browser Driver
The browser driver acts as an intermediary between Selenium and the browser.

Examples:

Chrome      → ChromeDriver
Firefox     → GeckoDriver
Edge        → EdgeDriver
Internet Explorer → IEDriverServer

5. Browser
The browser executes the requested operation.

For example:

Open URL
Click Button
Enter Text
Select Dropdown
Get Page Title
Close Browser

Selenium 3 Request and Response Flow
Suppose the test contains:

driver.get("https://example.com");

The communication can be understood as:

Test Script
     |
     | driver.get()
     v
Selenium Client
     |
     | JSON Wire request
     v
Browser Driver
     |
     | Browser command
     v
Browser
     |
     | Response
     v
Browser Driver
     |
     v
Selenium Client
     |
     v
Test Script

Simple Flow
Test Script
     ↓
Selenium WebDriver
     ↓
JSON Wire Protocol
     ↓
Browser Driver
     ↓
Browser
     ↓
Browser Driver
     ↓
Selenium WebDriver
     ↓
Test Script

Selenium 4 Architecture
Selenium 4 follows the W3C WebDriver Protocol.

The architecture is more standardized than Selenium 3.

Architecture Diagram
+----------------------+
|     Test Script      |
|  Java / Python / C#  |
+----------+-----------+
           |
           v
+----------------------+
| Selenium Client      |
|      Library         |
+----------+-----------+
           |
           v
+----------------------+
| W3C WebDriver        |
|      Protocol        |
+----------+-----------+
           |
           v
+----------------------+
|   Browser Driver     |
|                      |
| ChromeDriver         |
| GeckoDriver          |
| EdgeDriver           |
+----------+-----------+
           |
           v
+----------------------+
|       Browser        |
|                      |
| Chrome / Firefox /   |
| Edge / Safari        |
+----------------------+

Selenium 4 Components
1. Test Script
The test script contains automation instructions.

Example:

WebDriver driver = new ChromeDriver();

driver.get("https://example.com");

driver.findElement(By.id("username"))
      .sendKeys("admin");

2. Selenium Client Library
The client library provides APIs for the programming language being used.

Example:

driver.get();
driver.findElement();
driver.click();
driver.sendKeys();
driver.quit();

3. W3C WebDriver Protocol
Selenium 4 uses the W3C WebDriver standard for communication.

It defines standardized commands between the WebDriver client and browser automation implementation.

4. Browser Driver
The browser driver receives WebDriver commands and communicates with the browser.

Examples:

Chrome  → ChromeDriver
Firefox → GeckoDriver
Edge    → EdgeDriver
Safari  → SafariDriver

5. Browser
The browser performs the requested action.

Examples:

Navigate
Click
Type
Scroll
Select
Get Text
Get Title
Take Screenshot
Close

Selenium 4 Request and Response Flow
Example:

driver.get("https://example.com");

Flow:

Test Script
     |
     | WebDriver command
     v
Selenium Client
     |
     | W3C WebDriver request
     v
Browser Driver
     |
     | Browser operation
     v
Browser
     |
     | Response
     v
Browser Driver
     |
     v
Selenium Client
     |
     v
Test Script

Selenium 3 vs Selenium 4
Feature	Selenium 3	Selenium 4
Protocol	JSON Wire Protocol	W3C WebDriver Protocol
Standardization	Older approach	W3C standard
Browser Communication	JSON Wire based	W3C WebDriver based
Selenium Grid	Older architecture	Redesigned architecture
Relative Locators	❌ No	✅ Yes
New Window API	Limited	✅ Improved
New Tab API	Limited	✅ Supported
DevTools	Limited	✅ Better support
Element Screenshot	❌ Not directly available	✅ Supported
Browser Compatibility	Good	Improved
Grid Scalability	Limited	Better
Modern Automation	Legacy	Recommended

JSON Wire Protocol
The JSON Wire Protocol was the communication protocol historically used by Selenium 3.

The basic concept was:

Selenium Client
       |
       | HTTP + JSON
       v
Browser Driver
       |
       v
Browser

For example, when the test executes:

driver.get("https://example.com");

Selenium sends a command to the browser driver.

The driver then performs the action in the browser.

W3C WebDriver Protocol
Selenium 4 follows the W3C WebDriver standard.

The basic flow is:

Selenium Client
       |
       | W3C WebDriver Request
       v
Browser Driver
       |
       v
Browser
       |
       | Response
       v
Selenium Client

The main advantage is standardized communication between automation clients and browsers.

JSON Wire vs W3C WebDriver
Selenium 3

Test Script
     ↓
Selenium Client
     ↓
JSON Wire Protocol
     ↓
Browser Driver
     ↓
Browser

Selenium 4

Test Script
     ↓
Selenium Client
     ↓
W3C WebDriver Protocol
     ↓
Browser Driver
     ↓
Browser

Main Difference
Selenium 3 → JSON Wire Protocol

Selenium 4 → W3C WebDriver Protocol

Browser Drivers
A browser driver is a component that allows Selenium to communicate with a specific browser.

Chrome
Selenium
   ↓
ChromeDriver
   ↓
Chrome

Firefox
Selenium
   ↓
GeckoDriver
   ↓
Firefox

Edge
Selenium
   ↓
EdgeDriver
   ↓
Microsoft Edge

Safari
Selenium
   ↓
SafariDriver
   ↓
Safari

Selenium 3 Browser Driver Architecture
+-------------+
| Test Script |
+------+------+
       |
       v
+-------------+
| Selenium    |
+------+------+
       |
       v
+-------------+
| JSON Wire   |
| Protocol    |
+------+------+
       |
       v
+-------------+
| ChromeDriver|
+------+------+
       |
       v
+-------------+
|   Chrome    |
+-------------+

Selenium 4 Browser Driver Architecture
+-------------+
| Test Script |
+------+------+
       |
       v
+-------------+
| Selenium    |
+------+------+
       |
       v
+-------------+
| W3C WebDriver|
| Protocol     |
+------+-------+
       |
       v
+-------------+
| ChromeDriver|
+------+------+
       |
       v
+-------------+
|   Chrome    |
+-------------+

Selenium Grid
Selenium Grid is used when we want to run tests on multiple browsers, operating systems, or machines.

For example:

                Test Suite
                    |
                    v
             +-------------+
             | Selenium    |
             |    Grid     |
             +------+------+
                    |
       +------------+------------+
       |            |            |
       v            v            v
    Chrome       Firefox        Edge
     Node          Node         Node
       |            |            |
       v            v            v
    Test 1        Test 2       Test 3

Selenium Grid 3 vs Selenium Grid 4
Selenium Grid 3
Grid 3 commonly used a Hub and Node architecture.

                 +---------+
                 |   Hub   |
                 +----+----+
                      |
        +-------------+-------------+
        |             |             |
        v             v             v
     Node 1        Node 2        Node 3
     Chrome        Firefox         Edge

The Hub received test requests and distributed them to Nodes.

Selenium Grid 4
Selenium Grid 4 introduced a redesigned architecture with several internal components.

Conceptually:

                    Test
                     |
                     v
              +-------------+
              | Selenium    |
              |    Grid 4   |
              +------+------+
                     |
          +----------+----------+
          |          |          |
          v          v          v
       Chrome     Firefox      Edge
        Node        Node        Node

Grid 4 is designed for better scalability, distribution, and modern infrastructure.

Relative Locators
Selenium 4 introduced relative locators.

They allow an element to be located based on its relationship with another element.

Available relative locator methods include:

above()
below()
toLeftOf()
toRightOf()
near()

Example:

WebElement username =
    driver.findElement(By.id("username"));

WebElement password =
    driver.findElement(
        RelativeLocator.with(By.tagName("input"))
                       .below(username)
    );

The idea is:

Username
   |
   |
   ↓
Password

New Tab and Window Handling
Selenium 4 provides an easier API for creating a new tab or window.

New Tab
driver.switchTo().newWindow(WindowType.TAB);

New Window
driver.switchTo().newWindow(WindowType.WINDOW);

Example:

driver.get("https://example.com");

driver.switchTo().newWindow(WindowType.TAB);

driver.get("https://google.com");

DevTools Integration
Selenium 4 provides improved integration with browser developer tools.

This can be useful for advanced automation scenarios.

Examples include:

Network monitoring
Performance-related automation
Console information
Browser configuration
Geolocation
Network throttling
Conceptually:

Selenium Test
      |
      v
Selenium WebDriver
      |
      +-----------> Browser Driver
      |
      +-----------> DevTools
                         |
                         v
                     Browser

Element Screenshots
Selenium 4 supports taking a screenshot of a specific web element.

Example:

WebElement logo =
    driver.findElement(By.id("logo"));

logo.getScreenshotAs(OutputType.FILE);

Instead of taking only a screenshot of the complete page, you can capture a particular element.

Complete Selenium 4 Architecture
A more complete view can be represented as:

                     +-------------------+
                     |    Test Script    |
                     |   Java / Python   |
                     +---------+---------+
                               |
                               v
                     +-------------------+
                     | Selenium Client   |
                     |     Library       |
                     +---------+---------+
                               |
                               v
                     +-------------------+
                     | WebDriver API     |
                     +---------+---------+
                               |
                               v
                     +-------------------+
                     | W3C WebDriver     |
                     |     Protocol      |
                     +---------+---------+
                               |
                               v
                     +-------------------+
                     |   Browser Driver  |
                     +---------+---------+
                               |
              +----------------+----------------+
              |                |                |
              v                v                v
           Chrome           Firefox            Edge
              |                |                |
              v                v                v
          Web App          Web App           Web App

Selenium 3 Complete Architecture
                     +-------------------+
                     |    Test Script    |
                     |   Java / Python   |
                     +---------+---------+
                               |
                               v
                     +-------------------+
                     | Selenium Client   |
                     |     Library       |
                     +---------+---------+
                               |
                               v
                     +-------------------+
                     | JSON Wire         |
                     |     Protocol      |
                     +---------+---------+
                               |
                               v
                     +-------------------+
                     |   Browser Driver  |
                     +---------+---------+
                               |
              +----------------+----------------+
              |                |                |
              v                v                v
           Chrome           Firefox         Internet Explorer

Advantages of Selenium 3
Selenium 3 was widely used for web automation and provided:

Cross-browser testing
Multiple programming language support
WebDriver APIs
Selenium Grid
Parallel execution
Open-source automation
Integration with testing frameworks
However, Selenium 3 is now considered an older generation of Selenium.

Advantages of Selenium 4
Selenium 4 provides several improvements:

W3C WebDriver standard
Improved browser compatibility
Redesigned Selenium Grid
Relative locators
Better tab and window handling
Improved DevTools integration
Element screenshots
Better support for modern browsers
Better support for distributed testing
Selenium 3 vs Selenium 4 – Architecture in One Picture
                 SELENIUM 3
                 ----------

Test Script
     |
     v
Selenium Client
     |
     v
JSON Wire Protocol
     |
     v
Browser Driver
     |
     v
Browser


                 SELENIUM 4
                 ----------

Test Script
     |
     v
Selenium Client
     |
     v
W3C WebDriver Protocol
     |
     v
Browser Driver
     |
     v
Browser

Real Example
Suppose we write:

driver.findElement(By.id("username"))
      .sendKeys("admin");

Selenium 3
Conceptually:

Test Script
     |
     | findElement()
     v
Selenium Client
     |
     | JSON Wire
     v
Browser Driver
     |
     v
Browser
     |
     | Find username element
     v
Browser Driver
     |
     v
Selenium
     |
     v
Test Script

Then the sendKeys() operation is performed.

Selenium 4
Conceptually:

Test Script
     |
     | findElement()
     v
Selenium Client
     |
     | W3C WebDriver
     v
Browser Driver
     |
     v
Browser
     |
     | Find username element
     v
Browser Driver
     |
     v
Selenium
     |
     v
Test Script

Why Was Selenium 4 Introduced?
Selenium 4 was introduced to modernize Selenium and align WebDriver automation more closely with the W3C standard.

The major goals were:

Better Standardization
        +
Better Browser Support
        +
Better Grid
        +
Modern Web Features
        +
Better Developer Experience

Interview Questions
1. What is Selenium?
Selenium is an open-source framework used to automate web browsers and test web applications across different browsers and operating systems.

2. What is Selenium WebDriver?
Selenium WebDriver is an API that allows automation scripts to communicate with and control web browsers.

3. What is the main difference between Selenium 3 and Selenium 4?
The major difference is the communication protocol. Selenium 3 primarily used the JSON Wire Protocol, whereas Selenium 4 follows the W3C WebDriver Protocol.

4. What protocol does Selenium 4 use?
Selenium 4 uses the W3C WebDriver standard.

5. What protocol was used in Selenium 3?
Selenium 3 primarily used the JSON Wire Protocol.

6. What is a browser driver?
A browser driver acts as an intermediary between Selenium WebDriver and the browser. It receives WebDriver commands and communicates with the browser to execute them.

7. Give examples of browser drivers.
Chrome  → ChromeDriver
Firefox → GeckoDriver
Edge    → EdgeDriver
Safari  → SafariDriver

8. What is Selenium Grid?
Selenium Grid allows tests to run across multiple browsers, operating systems, and machines. It is commonly used for parallel and distributed test execution.

9. What are Relative Locators?
Relative Locators are a Selenium 4 feature that allows elements to be located based on their position relative to another element.

Examples:

above()
below()
toLeftOf()
toRightOf()
near()

10. What are the major features of Selenium 4?
Selenium 4 provides W3C WebDriver compliance, an improved Selenium Grid, relative locators, better window and tab handling, improved DevTools integration, and element screenshot support.

11. Explain Selenium 4 architecture.
Selenium 4 follows a client-server style architecture. The test script uses Selenium WebDriver APIs to send commands. These commands are handled using the W3C WebDriver protocol and sent to the browser driver. The browser driver communicates with the browser, executes the requested action, and returns the response back to the test script.

12. Explain Selenium 3 architecture.
Selenium 3 follows a client-server architecture where the test script communicates with Selenium WebDriver. Selenium 3 primarily used the JSON Wire Protocol to communicate with browser-specific drivers such as ChromeDriver and GeckoDriver. The driver then communicates with the browser and returns the result to the test script.

Quick Revision
Selenium 3
Selenium 3
    ↓
JSON Wire Protocol
    ↓
Browser Driver
    ↓
Browser

Remember
Selenium 3 = JSON Wire + Older Grid

Selenium 4
Selenium 4
    ↓
W3C WebDriver Protocol
    ↓
Browser Driver
    ↓
Browser

Remember
Selenium 4 = W3C + New Grid + Relative Locators

Most Important Differences
Selenium 3
├── JSON Wire Protocol
├── Older Grid
├── Older browser communication model
└── Legacy automation

Selenium 4
├── W3C WebDriver Protocol
├── Redesigned Grid
├── Relative Locators
├── Better DevTools support
├── Better Tab/Window APIs
└── Modern browser automation

One-Minute Interview Revision
Selenium 3 and Selenium 4 are both used for web automation. Selenium 3 primarily used the JSON Wire Protocol for communication between the Selenium client and browser drivers. Selenium 4 follows the W3C WebDriver Protocol, which provides a standardized communication mechanism.

Selenium 4 also introduced or improved several features such as Relative Locators, new tab and window APIs, a redesigned Selenium Grid, better DevTools integration, and element screenshots.

The basic architecture remains similar: Test Script → Selenium Client → Browser Driver → Browser, but the communication protocol and supporting architecture were significantly improved in Selenium 4.

Final Cheat Sheet
Topic	Selenium 3	Selenium 4
Main Protocol	JSON Wire	W3C WebDriver
Browser Driver	Yes	Yes
ChromeDriver	Yes	Yes
GeckoDriver	Yes	Yes
EdgeDriver	Yes	Yes
Selenium Grid	Grid 3	Grid 4
Relative Locators	❌	✅
New Tab API	Limited	✅
New Window API	Limited	✅
Element Screenshot	❌	✅
DevTools	Limited	Better
Standardization	Older	W3C
Recommended for New Projects	❌	✅

Final Formula
SELENIUM 3
Test Script
    ↓
Selenium WebDriver
    ↓
JSON Wire Protocol
    ↓
Browser Driver
    ↓
Browser


SELENIUM 4
Test Script
    ↓
Selenium WebDriver
    ↓
W3C WebDriver Protocol
    ↓
Browser Driver
    ↓
Browser

Easy Interview Memory Trick
3 → JSON Wire

4 → W3C + Grid + Relative Locators + DevTools
