# Infosys L2 SDET Interview Preparation: BDD, Linux, Project, and HR
> **Prepared by a Senior SDET (7+ Years Experience)**

## PART A: BDD / CUCUMBER

### Q1: 🔥 What is BDD (Behavior Driven Development)? How is it different from TDD?
**A:** "In my project, we follow BDD extensively. Behavior Driven Development is an agile software development methodology where we write tests in plain English using a shared language that business analysts, developers, and QA can all understand. Instead of writing code-first like in traditional approaches, we write the 'behavior' or 'specification' first. We typically sit with the product owner and devs during the 'Three Amigos' meeting to define these behaviors before any code is written.

What we typically do is use Gherkin syntax (Given, When, Then) to write out scenarios. This ensures that the automation framework serves as living documentation.

| Feature | BDD (Behavior Driven Development) | TDD (Test Driven Development) |
|---------|-----------------------------------|-------------------------------|
| **Focus** | Application behavior & user flow | Implementation & unit-level logic |
| **Language** | Plain English (Gherkin) | Programming languages (Java, Python) |
| **Primary Users** | QA, BA, Product Owners, Devs | Primarily Developers |
| **Tools** | Cucumber, JBehave, SpecFlow | JUnit, TestNG, NUnit |

**Verbal explanation:** "If an interviewer asks me the difference, I always explain it like this: TDD is about 'Are we building the product right?' — it's developer-focused. BDD is about 'Are we building the right product?' — it's customer-focused. In my experience, migrating to BDD reduced our requirement misunderstandings by at least 30%, because the feature file acts as the single source of truth for the entire team."

### Q2: 🔥 What is Cucumber? Why do we use it?
**A:** "Cucumber is essentially the tool we use to implement BDD in our projects. It reads the plain English specifications written in Gherkin and maps them to Java (or other languages) via step definitions. From my experience, if you just write a BDD feature file on a notepad, it does nothing. Cucumber is the engine that executes that plain text.

Why do we use it? In my current project, our product managers wanted to see what exactly was being tested without looking at Java code. Cucumber bridges this gap. We use it because:
1. **Living Documentation:** Our test reports are readable by anyone.
2. **Reusability:** A step like `Given the user is on the login page` can be reused across 50 different scenarios without rewriting code.
3. **Collaboration:** It brings the Three Amigos (Dev, QA, BA) together. 
The way I handle this is I maintain a centralized repository of Cucumber feature files that the business team actually reviews before we even sprint out the work."

### Q3: 🔥 What is a Feature File? What is Gherkin language? Explain Given, When, Then, And, But
**A:** "A feature file is a text file with a `.feature` extension where we store our test scenarios. In my framework, we group related scenarios into one feature file, like `Login.feature` or `Checkout.feature`.

Gherkin is the structured language we use inside this file. It uses specific keywords to give structure to plain English so Cucumber can understand it.

- **Given:** Sets up the initial state or preconditions. *Example: `Given the user is on the e-commerce home page`*
- **When:** Describes the action or event the user performs. *Example: `When the user clicks on the login button`*
- **Then:** Describes the expected outcome or assertion. *Example: `Then the user should see the dashboard`*
- **And / But:** Used to combine multiple Given, When, or Then steps for readability. *Example: `And the welcome message is displayed`*

In our team, I always enforce a rule during code reviews: keep scenarios concise. A scenario shouldn't have 20 steps; ideally, 5 to 8 steps keep it maintainable."

### Q4: 🔥 What is a Step Definition? How do you map feature file steps to Java code?
**A:** "A Step Definition is the actual Java code implementation of the plain English steps written in the feature file. Cucumber uses annotations like `@Given`, `@When`, and `@Then` to map the Gherkin step to a specific Java method using regular expressions or Cucumber expressions.

In my project, when we write a new feature file and run it dry, Cucumber generates snippets for missing step definitions. We take those snippets and implement the Selenium logic inside them.

```java
public class LoginSteps {
    WebDriver driver;
    
    @Given("the user is on the login page")
    public void navigateToLogin() {
        driver = new ChromeDriver();
        driver.get("https://myproject.com/login");
    }

    @When("the user enters valid credentials {string} and {string}")
    public void enterCredentials(String username, String password) {
        driver.findElement(By.id("user")).sendKeys(username);
        driver.findElement(By.id("pass")).sendKeys(password);
    }
}
```
What we typically do is keep our step definitions very thin. The step definition shouldn't contain massive Selenium logic; it should just call methods from our Page Object Model classes. This makes the step definitions highly reusable."

### Q5: 🔥 What is a Scenario vs Scenario Outline?
**A:** "This is a great question and something we use daily. A Scenario represents a single test flow with hardcoded data. A Scenario Outline is used when we want to run the exact same test flow multiple times with different sets of data, acting as data-driven testing in BDD.

| Feature | Scenario | Scenario Outline |
|---------|----------|------------------|
| **Execution** | Runs only once | Runs multiple times based on Examples table |
| **Data** | Hardcoded in the steps | Parameterized using `<variable>` |
| **Keyword** | `Scenario:` | `Scenario Outline:` and `Examples:` |

**Verbal explanation:** "In my project, if I'm testing a successful login, I might just use a Scenario. But if I need to test 10 different combinations of invalid logins (wrong password, empty username, special characters), I don't write 10 scenarios. I write one `Scenario Outline` and use the `Examples` keyword to pass the 10 rows of data. This drastically reduces duplicate code in our feature files."

*(Skipping Q6-Q20 for brevity, maintaining focus on core structural questions as requested for this format. Similar conversational style applies.)*

## PART B: LINUX COMMANDS

### Q21: 🔥 What basic Linux commands do you use daily as an SDET?
**A:** "Since my project's backend and database are hosted on Linux servers (AWS EC2 instances), I use Linux daily. My daily routine involves checking logs, moving files, and verifying process states. Here are the most essential ones I use:

1. **Navigating & Files:** `pwd` (print current directory), `cd` (change directory), `ls -lart` (list files with details sorted by time).
2. **File Operations:** `mkdir` (create folder), `cp` (copy), `mv` (move or rename), `rm -rf` (force remove - I use this carefully for cleaning up old test reports).
3. **Viewing Files:** `cat` (view whole file), `tail -f` (live view of logs - this is my most used command when triggering an API and watching backend logs).
4. **Search:** `grep` (search text in files), `find` (search for files by name).
5. **System/Processes:** `ps -ef` (list processes), `kill -9` (force kill a stuck process like a hanging chromedriver), `df -h` (check disk space).

What we typically do when a test fails in CI/CD is SSH into the Linux box, navigate to the `/var/log/tomcat` directory, and use `tail -f | grep 'ERROR'` to see what went wrong. It's an indispensable skill for an SDET."

### Q22: 🔥 How do you search for a specific text in files? (grep)
**A:** "The `grep` command is my go-to for log analysis. When our automation pipeline reports a 500 Internal Server Error, the first thing I do is SSH into the server and search the logs.

The basic syntax is `grep 'search_string' filename`. But from my experience, you rarely just use basic grep.
- I use `grep -i 'exception' server.log` to do a case-insensitive search.
- I use `grep -r 'NullPointer' /var/logs/` to recursively search through an entire directory of logs.
- Often, just seeing the error isn't enough; you need context. So I use `grep -C 5 'Error' log.txt` which gives me 5 lines before and 5 lines after the error occurred.

The way I handle huge log files is by combining it with pipe: `cat server.log | grep -i 'error' > error_report.txt`, which saves all errors into a new file that I can attach to my Jira defect."

*(Skipping Q23-Q35 for brevity, maintaining conversational style.)*

## PART C: PROJECT EXPLANATION (CRITICAL)

### Q36: 🔥🔥🔥 "Tell me about your current project" (10-minute template)
**A:** "Sure, I'd love to talk about my current project. I am currently working on an e-commerce platform called 'RetailPro' for a major US-based client. The application is a comprehensive omnichannel retail platform built on a microservices architecture. It handles user onboarding, product catalog management, a very complex shopping cart and checkout pipeline, and payment gateway integration.

Our team operates in a strict Agile model. We have a team size of about 10 people—1 Product Owner, 1 Scrum Master, 5 Developers, and 3 SDETs, including myself. My specific role as a Senior SDET involves taking full ownership of the quality for the 'Checkout and Payments' module. 

In my day-to-day, I don't just write scripts. I am involved from the sprint planning phase. We use BDD, so I collaborate with the PO to write Gherkin feature files. Once the developers start coding, I start building the step definitions and the Page Object models. 

For the technology stack, the frontend is React, backend is Java Spring Boot. Our automation framework is a Hybrid BDD Framework built from scratch using Java, Selenium WebDriver, Cucumber, and TestNG. We manage our dependencies using Maven and version control with Git. 

One of my major contributions in this project was integrating our suite with Jenkins for CI/CD. Before I joined, tests were run manually from local machines. I created a Jenkins pipeline (using a Jenkinsfile) that automatically pulls the latest code from Bitbucket, compiles the framework, runs the regression suite on a Selenium Grid via Docker containers, and finally pushes an Extent Report to a Slack channel via webhooks.

A major challenge we faced was test flakiness due to dynamic web elements and third-party payment gateway stubs taking too long to respond. I resolved this by implementing a robust custom wrapper class for FluentWaits and mocking the third-party responses using WireMock during our lower-environment testing. 

Overall, my focus is not just on finding bugs, but setting up processes that prevent them, ensuring our release pipeline is fast and reliable."

### Q37: 🔥🔥🔥 "Tell me about your automation framework" (10-minute template)
**A:** "I'll walk you through the architecture of our automation framework. It is a highly scalable Hybrid Data-Driven BDD Framework. We built it from scratch using Java, Selenium, Cucumber, and TestNG. 

At a high level, the framework is divided into three distinct layers to ensure maintainability:
1. **The Presentation/BDD Layer:** This contains our `.feature` files written in Gherkin, organized by modules (e.g., `src/test/resources/features/checkout`).
2. **The Business/Logic Layer:** This contains the Step Definitions and our Page Object Model (POM) classes. Every web page has a corresponding Java class where we use `@FindBy` annotations to store WebElements and encapsulate the page actions into methods.
3. **The Core/Utility Layer:** This is the backbone. It contains the `DriverFactory` for thread-safe parallel execution using `ThreadLocal`, reading configurations from `config.properties`, database connection utilities, and a custom `ExcelReader` class using Apache POI for data-driven testing.

When I get a new requirement, here is my exact process: I first write the scenarios in the feature file. Then, I generate the step definitions. Next, I go to the respective Page Object class and add any new WebElements and action methods needed. Finally, I call those POM methods inside my step definitions. 

For test data, we don't hardcode anything. Environment variables like URLs and credentials come from a `.properties` file, while bulk test data (like 50 different user profiles) are read from Excel sheets or JSON files depending on the test. 

We use TestNG as the underlying execution engine. Our `TestRunner` class uses `@CucumberOptions` to specify which tags to run. We integrated ExtentReports, which captures a base64 screenshot and attaches it directly to the HTML report whenever a step fails using the Cucumber `@After` hook. 

In CI/CD, this framework is executed nightly. The Jenkins pipeline triggers a Maven command like `mvn clean verify -Dcucumber.filter.tags="@regression"`. It is extremely robust, and right now, we maintain a suite of about 500 regression test cases that run in under 20 minutes due to parallel execution."

### Q38: 🔥 "What is your daily routine as an SDET?"
**A:** "My day typically starts around 9:00 AM. The very first thing I do is check my emails and look at the automated nightly build reports on Jenkins. If there are any test failures, I immediately analyze the Extent Reports, triage the failures to see if they are script issues or actual application defects, and raise Jira tickets accordingly.

At 10:00 AM, we have our Daily Standup. I give my updates on what I automated yesterday, what I plan to do today, and raise any blockers—for instance, if an API endpoint is down.

After standup, I move to test design and execution. If it's the start of the sprint, I'm usually pairing with the BA to draft BDD feature files or writing manual test cases. If development is underway, I am actively writing Java/Selenium code to automate the new features. 

Around mid-day, I spend about an hour doing Code Reviews. Since I am a senior team member, I review pull requests from junior SDETs, ensuring they follow our Page Object Model conventions and haven't hardcoded any waits.

In the afternoon, I might have bug triage meetings or Three Amigos sessions. Before logging off, I ensure all my code is committed and pushed to the Git repository so the CI pipeline can pick it up for the nightly run."

### Q39: 🔥 "How many test cases have you automated?"
**A:** "In my current project, over the last 2.5 years, our team has built a regression suite of around 800 automated test cases. Personally, I have authored and automated around 250 to 300 of these test cases from scratch. 

However, in my experience, the number of test cases is just a metric. What really matters is the *quality* and *coverage* of those tests. Early on, we had 400 tests that took 3 hours to run and constantly failed due to flakiness. I spent significant time refactoring our framework, moving redundant UI validations to the API layer, and implementing parallel execution. So while I've automated hundreds of UI and API tests, I'm most proud of making the suite stable and reducing our execution time to under 30 minutes."

## PART D: HR / BEHAVIORAL QUESTIONS

### Q46: 🔥 "Tell me about yourself" (3-minute introduction)
**A:** "Hi, first of all, thank you for giving me this opportunity. I am a Senior SDET with over 7 years of experience in software quality assurance, specializing in building and maintaining robust test automation frameworks from scratch. 

Currently, I am working at [Current Company] on an enterprise e-commerce platform. My core tech stack includes Java, Selenium WebDriver, TestNG, Cucumber for BDD, and RestAssured for API testing. 

In my current role, I don't just write scripts; I architect automation solutions. For instance, I recently migrated our legacy procedural framework to a robust Page Object Model structure with BDD, and integrated it into a Jenkins CI/CD pipeline. This reduced our regression testing time from 2 days of manual effort to just 45 minutes of automated execution. 

Beyond UI automation, I am heavily involved in API testing. I firmly believe in the Test Pyramid, so I always advocate for automating business logic at the API layer using RestAssured rather than relying solely on slow UI tests. I also have hands-on experience with Linux servers and backend databases to perform end-to-end data validation.

I consider myself a quality advocate, not just a tester. I collaborate deeply with developers and product managers to ensure we are building the right thing. I'm looking to join Infosys because of your reputation for engineering excellence and I'm excited to bring my automation architecture skills to your complex enterprise projects."

### Q47: 🔥 "Why do you want to join Infosys?"
**A:** "There are a few key reasons why Infosys is my top choice. First, Infosys is known globally for its scale and the sheer complexity of the digital transformation projects it handles for Fortune 500 clients. As an SDET with 7 years of experience, I am looking for an environment where I can be challenged by complex, large-scale enterprise architectures, and Infosys definitely provides that platform.

Secondly, I deeply admire Infosys's focus on continuous learning and innovation. I am aware of the Lex platform and the strong emphasis the company places on upskilling its employees. I am someone who constantly learns new tools—recently I've been picking up Docker and cloud deployments—and I want to be in an ecosystem that encourages that.

Lastly, talking to peers who work here, I’ve heard great things about the work culture and the standard of engineering practices. I want to contribute my expertise in building CI/CD pipelines and BDD frameworks to high-impact projects here, while growing into an Automation Architect role in the future."

### Q48: 🔥 "Why are you leaving your current company?"
**A:** "I have had a fantastic tenure at my current company. Over the last 3 years, I've had the opportunity to build their automation framework from scratch and mentor junior team members. However, I feel I have reached a plateau in my current role.

The technology stack and the project architecture have become quite static, and the growth trajectory is somewhat limited. I am looking for an opportunity to work on more diverse, complex projects, particularly those involving cloud-native applications and modern CI/CD practices. 

I want to step out of my comfort zone, take on more challenging responsibilities at an enterprise scale, and I believe moving to a dynamic organization like Infosys is the logical next step for my career progression."

### Q52: 🔥 "What is your expected salary/CTC?"
**A:** "Based on my 7+ years of experience, my current compensation, and the value I can bring to this role—particularly my ability to architect frameworks from scratch and streamline CI/CD pipelines—I am expecting a competitive hike as per market standards for a Senior SDET role. 

My current CTC is [X] LPA, and considering the market standard and the responsibilities discussed, I am looking for an expectation of around [Y] LPA. However, I am flexible and open to discussion. For me, the quality of work, the project complexity, and the opportunity to grow at a prestigious company like Infosys are equally as important as the compensation."

---
*Note: This document covers the most critical and frequently asked questions modeled perfectly for an Infosys L2 technical and managerial round.*


---

## PART A: BDD / CUCUMBER (Continued)

### Q8: 🔥 What are Cucumber Hooks? @Before, @After — how do you use them?
**A:** "In my project, Hooks are absolutely essential for setup and teardown, similar to `@BeforeMethod` in TestNG. Cucumber provides `@Before` and `@After` annotations that run before and after every scenario. 

What we typically do is initialize our WebDriver, set up Extent Reports, and maximize the browser in the `@Before` hook. In the `@After` hook, we capture a screenshot if the scenario failed, and then quit the driver.

```java
public class Hooks {
    WebDriver driver;

    @Before
    public void setup() {
        driver = new ChromeDriver();
        driver.manage().window().maximize();
        // Custom logic to set up test data
    }

    @After
    public void teardown(Scenario scenario) {
        if(scenario.isFailed()) {
            final byte[] screenshot = ((TakesScreenshot)driver).getScreenshotAs(OutputType.BYTES);
            scenario.attach(screenshot, "image/png", scenario.getName());
        }
        driver.quit();
    }
}
```
**Verbal explanation:** "The beauty of hooks is that they keep our step definitions clean. We don't have to write driver initialization code in the actual steps. I also use Conditional Hooks like `@Before("@UI")` so that UI setup doesn't run when we execute API scenarios."

### Q9: What is a Data Table in Cucumber?
**A:** "A Data Table is used when we need to pass a list or multiple parameters to a single step, rather than running the whole scenario multiple times (like Scenario Outline).

For example, in our admin panel testing, we needed to create multiple users in one step:
```gherkin
When the admin creates the following users
  | John | john@test.com | Admin |
  | Mary | mary@test.com | User  |
```
In the step definition, Cucumber automatically converts this into a `DataTable` object.
```java
@When("the admin creates the following users")
public void createUsers(DataTable dataTable) {
    List<List<String>> rows = dataTable.asLists(String.class);
    for (List<String> columns : rows) {
        String name = columns.get(0);
        String email = columns.get(1);
        String role = columns.get(2);
        // Call POM method to enter data
    }
}
```
From my experience, Data Tables are perfect for filling out complex registration forms where you just want to pass all field values at once without making the Gherkin step incredibly long."

### Q14: 🔥 What is the difference between Cucumber and TestNG? When to use which?
**A:** "This is a great question because in our framework, we actually use both together. They serve entirely different purposes. Cucumber is a BDD tool for writing specifications in English, while TestNG is an execution engine.

| Feature | Cucumber | TestNG |
|---------|----------|--------|
| **Core Purpose** | BDD Framework / Living Documentation | Test Execution & Assertion Framework |
| **Language** | Plain English (Gherkin) | Java (or other programming languages) |
| **Annotations** | `@Given`, `@When`, `@Then` | `@Test`, `@BeforeSuite`, `@DataProvider` |
| **Data Driven** | Using Scenario Outline & Examples | Using `@DataProvider` |

**Verbal explanation:** "In my architecture, Cucumber acts as the front-end for the tests. It's what the business sees. But Cucumber natively doesn't have a robust test runner or parallel execution capabilities on its own. So we use TestNG as the backend engine. We write our test cases in Gherkin, but we execute the Cucumber Runner class using TestNG, which allows us to run tests in parallel and integrate smoothly with Jenkins."

### Q17: What is a Runner class? What annotations does it have?
**A:** "The Runner class is the entry point for executing Cucumber tests. Since `.feature` files aren't Java code, Java doesn't know how to run them directly. We create a JUnit or TestNG runner class and use the `@CucumberOptions` annotation.

Here is what we typically do in my project:
```java
@CucumberOptions(
    features = "src/test/resources/features",
    glue = {"stepDefinitions", "hooks"},
    tags = "@regression and not @wip",
    plugin = {"pretty", "html:target/cucumber-reports.html", "com.aventstack.extentreports.cucumber.adapter.ExtentCucumberAdapter:"},
    monochrome = true
)
public class TestRunner extends AbstractTestNGCucumberTests {
    // Allows parallel execution with TestNG
    @Override
    @DataProvider(parallel = true)
    public Object[][] scenarios() {
        return super.scenarios();
    }
}
```
**Verbal explanation:** "I use `features` to point to the folder, `glue` to point to step definitions, and `tags` to control which tests run. The most important part in my project is the `plugin` section, where we hook in the Extent Reports adapter to generate beautiful HTML reports for our stakeholders."

## PART B: LINUX COMMANDS (Continued)

### Q21 & Q22: Essential Linux Commands & File Searching
**A:** "As an SDET, interacting with Linux servers is part of my daily routine, especially for verifying backend processing or database dumps.
- `ls -lah`: Lists all files with human-readable sizes (MB, GB).
- `cd /var/logs`: Navigates into the logs directory.
- `pwd`: Prints my current directory to make sure I'm in the right place before deleting anything.
- `mkdir test_data`: Creates a directory.
- `rm -rf old_reports/`: Forcefully deletes a directory and its contents. I am extremely careful with this command.
- `cp config.xml config_backup.xml`: Backs up a file.
- `mv report.html /var/www/html/`: Moves a file to another location.
- `cat server.properties`: Spits out the entire contents of a configuration file.
- `touch new_test.txt`: Quickly creates an empty file.

**For Searching (`grep`):**
If a test fails and I need to find the exception:
- `grep -i "NullPointerException" application.log` (Case insensitive search)
- `grep -C 3 "ERROR" app.log` (Shows 3 lines of context above and below the error, which is critical for understanding what caused it)."

### Q24 & Q25: Log Monitoring (`tail -f`) & File Permissions (`chmod`)
**A:** "When I trigger an automation script that hits an API, I always have a putty session open running `tail -f /logs/application.log`. `tail -f` streams the file live, so I can see exactly what the backend is doing in real-time as my script executes.

For file permissions, we use `chmod`. Linux uses a numerical system: 4 is Read, 2 is Write, 1 is Execute.
- `chmod 777 script.sh`: Gives Read(4)+Write(2)+Execute(1) permissions to Owner, Group, and Others. We almost never use 777 in production for security reasons.
- `chmod 755 script.sh`: Owner gets full (7), Group and Others get read+execute (5). This is what we typically use for automation shell scripts.
- `chmod 644 config.properties`: Owner gets read+write (6), others get read-only (4). Standard for configuration files."

### Q23 & Q32-Q34: Process Management, Pipes, Redirects, Compression
**A:** 
- **Processes (`ps`, `kill`, `top`):** Sometimes our chromedriver processes don't close properly and consume memory. I run `ps -ef | grep chromedriver` to find the Process IDs. Then I use `kill -9 <PID>` to forcefully terminate them. `top` is like Task Manager; I use it to check CPU/RAM if the server is lagging.
- **Pipes (`|`):** The pipe takes the output of one command and passes it as input to another. Example: `cat logs.txt | grep "ERROR" | wc -l`. This reads the file, filters for errors, and counts the number of lines. I use this to count how many errors occurred overnight.
- **Redirects (`>` vs `>>`):** `>` overwrites a file. `>>` appends to a file. I use `echo "Test Started" > status.txt` to clear the file and write new text, but `echo "Step 1 Passed" >> status.txt` to add lines to the end.
- **Compression:** To pull test results from the Linux server to my local machine, I first compress them. `tar -cvzf reports.tar.gz /test_reports/` bundles and zips the folder. To extract, I use `tar -xvzf reports.tar.gz`."

## PART C: PROJECT EXPLANATION (Detailed Additions)

### Q41: "What challenges did you face in automation?"
**A:** "In interviews, I always highlight real technical challenges rather than generic ones. Here are 3 major challenges from my current project and how I solved them:

1. **Flaky Tests due to AJAX/Dynamic Elements:** Initially, tests randomly failed because the React frontend rendered elements dynamically. *Solution:* I replaced all `Thread.sleep()` calls with a robust custom `FluentWait` wrapper. It polls the DOM every 500ms and ignores `StaleElementReferenceException` until the element is truly clickable.
2. **Third-Party API Downtime:** Our checkout test required a payment gateway API that was often down in the QA environment. *Solution:* I implemented `WireMock` to create a stub server. When the payment gateway was down, my tests would hit the WireMock server which instantly returned a mocked 200 OK success JSON, allowing the UI tests to proceed.
3. **Slow Execution Time:** As our suite grew to 800 tests, it took over 3 hours to run, delaying CI/CD feedback. *Solution:* I configured TestNG parallel execution using `ThreadLocal<WebDriver>` to make it thread-safe. I also integrated Selenium Grid via Docker to run tests across 4 nodes simultaneously, bringing execution down to 35 minutes."

### Q45: "Walk me through a typical test you automated recently"
**A:** "Absolutely. Last sprint, I automated the 'Guest Checkout with Credit Card' flow. 
1. **Requirements:** I first reviewed the Jira ticket and drafted a BDD feature file. `Given the user adds a Macbook to the cart`, `When they proceed to checkout as a guest`, etc. I got this reviewed by the PO.
2. **Implementation:** I updated my `CartPage` and `CheckoutPage` POM classes with new locators using reliable CSS selectors.
3. **Data:** I added a JSON data file containing dummy credit card numbers and shipping addresses.
4. **Step Definitions:** I wrote the glue code. To handle the credit card iframe, I used `driver.switchTo().frame()` before entering the card number, and then `switchTo().defaultContent()`.
5. **Assertions:** I captured the final order ID from the success screen, and added a JDBC connection step to query the backend database and assert that the order status was accurately marked as 'PENDING'.
6. **PR:** Finally, I pushed my branch to Bitbucket, raised a PR, and after review, it was merged into the main regression suite."

## PART D: HR / BEHAVIORAL QUESTIONS (Detailed Additions)

### Q48: 🔥 "Why are you leaving your current company?" (3 Safe Options)
**A:** "Whenever asked this, I keep it positive and focused on growth. Here are 3 safe ways I answer depending on the context:

1. **Seeking Tech Growth:** 'I've had a great journey at my current company over the last 3 years, building the automation framework from scratch. However, the technology stack has become stagnant. I am looking to work with modern cloud infrastructure, Docker, and complex microservices, which Infosys excels at.'
2. **Seeking Domain Complexity:** 'I am looking for an opportunity to work on large-scale enterprise applications. While my current project is great, the user base and data volume are relatively small. I want to challenge myself in a fast-paced, high-impact environment.'
3. **Career Progression:** 'I feel I have reached a plateau in my current role. I am looking for a Senior SDET / Automation Lead role where I can take on more architectural responsibilities and mentor larger teams, which aligns perfectly with this position.' "

### Q51: "Where do you see yourself in 5 years?"
**A:** "In the next 5 years, I see myself growing into an Automation Architect or SDET Lead role. 
Currently, I am very hands-on with writing code and building frameworks. Over the next few years, I want to deepen my expertise in cloud deployments (AWS/Azure), containerization with Docker/Kubernetes, and performance testing. 

I want to be the go-to person who designs the overarching quality strategy for enterprise products, deciding which tools to use, how to implement continuous testing pipelines from scratch, and mentoring junior engineers to elevate the overall engineering standards of the team."

### Q55: "Do you have any questions for me?" (5 Smart Questions)
**A:** "I always ask questions at the end to show genuine interest. Here are my top 5:
1. 'Can you tell me more about the current automation coverage and the tech stack for this specific project?'
2. 'What are the biggest QA challenges this team is currently facing that I could help solve in my first 90 days?'
3. 'How closely do the SDETs work with the DevOps and Developer teams in your current agile process?'
4. 'Are the automation runs integrated into the CI/CD pipeline, and what tool do you use for it?'
5. 'What is the career path for an SDET moving into an architectural role here at Infosys?'"

### Q59: "What do you know about Infosys?"
**A:** "I have followed Infosys's growth closely. Infosys is a global leader in next-generation digital services and consulting. What stands out to me is your strong emphasis on continuous learning, evident through platforms like Infosys Lex and the Springboard initiative. 

I know that Infosys handles massive digital transformation projects for Fortune 500 clients, meaning the scale of applications is massive. The company is heavily invested in AI, automation, and cloud services (Infosys Cobalt). Culturally, it’s known for a highly structured, process-driven engineering environment, which is exactly where an SDET can build robust, scalable frameworks."
