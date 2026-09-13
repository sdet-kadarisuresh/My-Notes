# COMPLETE API Testing + Maven + Git Interview Q&A notes

## PART A: API TESTING

### Section 1: REST API Fundamentals

**Q: 1. 🔥 What is an API? What is REST API? Explain in simple terms**
**A:** "In my experience, when I explain an API to stakeholders, I simply call it a messenger or a bridge that takes a request from one system, delivers it to another, and brings the response back. For example, in my current project, our frontend UI is built on React and it needs data from our database. It can't directly query the database, so it calls our backend APIs to fetch that data. 
Now, when we say 'REST API', we are talking about an API that follows the REST (Representational State Transfer) architecture. What we typically do is expose endpoints over HTTP, and use standard HTTP methods like GET to retrieve data, POST to create, and PUT to update. A REST API is stateless, meaning every request from the client must contain all the information the server needs to understand and process it. The server doesn't remember previous requests. From an SDET perspective, testing a REST API means sending these HTTP requests and validating the response status codes, the JSON body, and headers to ensure the business logic works independently of the UI."

**Q: 2. 🔥 What are HTTP methods? Explain GET, POST, PUT, PATCH, DELETE with examples**
| Method | Description | Idempotent? | Example |
|--------|-------------|-------------|---------|
| GET | Retrieves a resource | Yes | Fetch user details |
| POST | Creates a new resource | No | Create a new user account |
| PUT | Replaces an entire resource | Yes | Update all user profile fields |
| PATCH | Partially updates a resource | No | Update only user's phone number |
| DELETE | Removes a resource | Yes | Delete a user account |

**Verbal explanation:** "In our API testing framework, we heavily rely on these 5 core HTTP methods. We use **GET** to read data. It’s idempotent, meaning if I fire the same GET request 100 times, the state on the server doesn't change. **POST** is what we use to create new records. If I send a POST payload for a new user, a new record is created. If I send it again, another record is created, so it's not idempotent. **PUT** is for full updates; if a user has 10 fields and I want to update 1, I still have to send all 10 fields in the payload. It replaces the entire resource. **PATCH**, on the other hand, is for partial updates. In my recent project, we switched many PUT endpoints to PATCH because it saves bandwidth—you just send the field that changed. Finally, **DELETE** is pretty self-explanatory, it removes a resource. Edge cases we always test for include trying to DELETE a resource that doesn't exist to ensure we get a 404, or trying to POST with missing mandatory fields to verify a 400 Bad Request."

**Q: 3. 🔥 What is the difference between PUT and PATCH?**
| Feature | PUT | PATCH |
|---------|-----|-------|
| Purpose | Complete Replacement | Partial Update |
| Bandwidth | Consumes more (full payload) | Consumes less (partial payload) |
| Idempotent | Yes | No |

**Verbal explanation:** "I get asked this a lot, and in my projects, this is a crucial distinction. We use PUT when we want to completely replace an existing resource. For example, if a customer profile has Name, Email, and Address, and I only want to change the Address, using a PUT request means I still have to pass the Name and Email in my payload. If I forget them, they might get wiped out or nullified depending on the backend logic. That's a huge risk. 
To solve this, we use PATCH. With PATCH, I only send the Address in the JSON body, and the backend only updates that specific field, leaving Name and Email untouched. From an automation standpoint, I always write negative tests for PUT to ensure omitting fields correctly throws a validation error or sets them to null, and for PATCH, I verify that only the target field is modified."

**Q: 4. 🔥 What is the difference between PUT and POST?**
| Feature | PUT | POST |
|---------|-----|------|
| Purpose | Update/Replace existing resource | Create new resource |
| Endpoint | Usually includes specific ID (`/users/1`) | Usually a collection (`/users`) |
| Idempotent | Yes | No |

**Verbal explanation:** "The way I handle this in my test automation is by looking at idempotency. POST is for creating something brand new. If I hit a POST `/users` endpoint five times with the same payload, I will create five identical users with five different IDs. It changes the state every time. PUT, however, is idempotent. It’s used for updating. If I hit PUT `/users/123` five times with the same payload, the first request updates the user, and the next four just overwrite it with the exact same data. The end state on the server is exactly the same as after the first request. In my test suites, I always include an idempotency test for PUT endpoints where I fire the request multiple times and assert that the response and DB state remain consistent."

**Q: 5. 🔥 What are HTTP status codes? Explain 200, 201, 204, 400, 401, 403, 404, 500, 502, 503**
| Code | Category | Meaning |
|------|----------|---------|
| 200 | Success | OK - GET/PUT success |
| 201 | Success | Created - POST success |
| 204 | Success | No Content - DELETE success |
| 400 | Client Error | Bad Request - Invalid syntax/validation failed |
| 401 | Client Error | Unauthorized - Missing/Invalid Token |
| 403 | Client Error | Forbidden - Has token but no permission |
| 404 | Client Error | Not Found - Endpoint/Resource missing |
| 500 | Server Error | Internal Server Error - Code exception on backend |
| 502 | Server Error | Bad Gateway - Issue with upstream server/proxy |
| 503 | Server Error | Service Unavailable - Server overloaded/down |

**Verbal explanation:** "In our REST Assured framework, status code validation is always the first assertion we make. The 200 series means success. What we typically see is 200 OK for GETs, 201 Created for POSTs, and 204 No Content for successful DELETEs—meaning it worked but there's no body to return. 
The 400 series means the client messed up. A 400 Bad Request usually means my JSON payload has a typo or failed validation. 401 Unauthorized means I forgot my bearer token, while 403 Forbidden means I have a token, but I'm trying to access an admin endpoint as a regular user. 
The 500 series means the server broke. If I see a 500 Internal Server Error during testing, it means I found a bug—likely an unhandled NullPointerException in the backend code. 502 and 503 usually point to infrastructure issues, like a load balancer failing or the server being down for deployment."

### Section 2: REST Assured

**Q: 11. 🔥 What is REST Assured? Why do we use it?**
**A:** "In my current framework, we chose REST Assured because it's a powerful Java library specifically designed for testing RESTful APIs. Why do we use it over something like Postman for automation? Because it integrates seamlessly with our existing Java ecosystem. Since our UI tests are written in Selenium with Java, using REST Assured allows us to keep everything in one repository. We can use the same TestNG test runners, the same Maven build lifecycle, and the same reporting tools like ExtentReports or Allure.
Furthermore, REST Assured uses a BDD-style Given-When-Then syntax which makes the tests highly readable. I can easily extract data from a response, pass it to a database utility class to verify backend states, and run everything in our Jenkins CI/CD pipeline. It handles JSON parsing beautifully under the hood using Jackson or Gson, saving us from writing boilerplate parsing code."

**Q: 12. 🔥 Explain the structure of a REST Assured test — given().when().then()**
**A:** "From my experience, the Given-When-Then structure is what makes REST Assured so elegant—it essentially forces you to write self-documenting code. 
- **given()**: This is where we set up our prerequisites. I put things like headers, query parameters, path parameters, request body payloads, and authentication tokens here. Basically, anything the request needs before it's sent.
- **when()**: This is the action phase. It's usually just one line where we specify the HTTP method and the endpoint URL. For example, `.when().get("/users/{id}")`.
- **then()**: This is the assertion phase. After the response comes back, this block validates it. I use it to assert the status code (like `.statusCode(200)`), check response headers, or validate specific fields in the JSON body using Hamcrest matchers. 
In my project, we wrap the `given()` part in a reusable `RequestSpecification` so we don't repeat the base URL and auth tokens in every single test method."

**Q: 13. 🔥 How do you send a GET request using REST Assured?**
**A:** "Here is a standard snippet of how I implement a GET request in our framework. We usually extract the response to validate multiple things later.
```java
Response response = given()
    .baseUri(\"https://api.example.com\")
    .header(\"Accept\", \"application/json\")
    .pathParam(\"userId\", 101)
.when()
    .get(\"/users/{userId}\")
.then()
    .statusCode(200)
    .extract().response();
    
String email = response.jsonPath().getString(\"data.email\");
Assert.assertEquals(email, \"test@example.com\");
```
What we typically do is pass the base URI from a properties file based on the environment. The `pathParam` handles dynamic endpoints. We do basic assertions in the `then()` block, but I always extract the full `Response` object. This allows me to use TestNG's `Assert` class for complex business logic validations against the database, which is a best practice to keep API and DB in sync."

**Q: 14. 🔥 How do you send a POST request with JSON body?**
**A:** "For POST requests, managing the JSON body is the key. While you can use a raw String or a Hashmap, in my projects, we strictly use POJOs (Plain Old Java Objects) for serialization. 
```java
UserPayload user = new UserPayload(\"John\", \"QA Engineer\");

Response response = given()
    .baseUri(\"https://reqres.in/api\")
    .header(\"Content-Type\", \"application/json\")
    .body(user)
.when()
    .post(\"/users\")
.then()
    .statusCode(201)
    .body(\"name\", equalTo(\"John\"))
    .extract().response();
```
By passing the `user` object into `.body()`, REST Assured automatically serializes it into a JSON string using Jackson. This makes maintenance a breeze—if the API contract changes and adds a new field, I just update the POJO class, and all my tests automatically inherit the new structure. It's much cleaner than dealing with huge concatenated strings or parsing JSON files."

**Q: 18. How do you handle authentication in API testing? (Basic Auth, Bearer Token, OAuth2)**
**A:** "In enterprise APIs, security is everything. What we typically do depends on the API architecture. For older legacy APIs, we use Basic Auth. In REST Assured, I handle this right in the `given()` block using `.auth().basic(\"username\", \"password\")`. 
However, for modern microservices, we almost exclusively use Bearer Tokens (JWT). For this, I first make a POST request to a login endpoint, extract the token from the response using JSONPath, and pass it in subsequent requests like this: `.header(\"Authorization\", \"Bearer \" + token)`. 
For OAuth2, it's a bit more complex. In my current project, we automate the authorization code flow by sending a request to the Auth server with our client ID and client secret to get an access token. REST Assured has a built-in method for this too: `.auth().oauth2(accessToken)`. I usually write a `@BeforeSuite` method to fetch these tokens once and store them in a static thread-safe variable so my tests aren't constantly bombarding the auth server."

**Q: 21. 🔥 How do you validate JSON schema?**
**A:** "Functional validation isn't always enough; we also need to ensure the structure of the JSON payload is correct. If the frontend expects an integer for an ID but the backend suddenly sends a string, the app will crash. To prevent this, I use JSON Schema validation in REST Assured.
```java
given()
    .get(\"/users/1\")
.then()
    .assertThat()
    .body(JsonSchemaValidator.matchesJsonSchemaInClasspath(\"user-schema.json\"));
```
From my experience, the best approach is to generate the schema file based on the Swagger docs and place it in the `src/test/resources` folder. The `JsonSchemaValidator` library then automatically checks if all mandatory fields are present, if the data types (string, boolean, array) match, and if the JSON object structure is completely valid. It’s an incredibly fast way to catch contract-breaking changes."

**Q: 23. What is serialization and deserialization? (POJO to JSON and back)**
**A:** "Serialization is the process of converting a Java object (POJO) into a JSON format so it can be sent over the network in a POST or PUT request. Deserialization is the exact opposite—taking the JSON response from the server and converting it back into a Java object for easy validation.
In my test automation framework, we heavily rely on Jackson for this. Instead of manually parsing huge JSON strings, I create a Java class that maps to the JSON structure.
```java
// Serialization (Java to JSON)
User user = new User(\"QA\", \"Engineer\");
given().body(user).post(\"/users\");

// Deserialization (JSON to Java)
User responseUser = given().get(\"/users/1\").as(User.class);
Assert.assertEquals(responseUser.getJob(), \"Engineer\");
```
This approach makes the code much cleaner and object-oriented. If the JSON structure has nested arrays, we just use Lists inside our POJO. It drastically reduces the maintenance effort."

**Q: 25. What is RequestSpecification and ResponseSpecification? How to reuse them?**
**A:** "When you have 500 API tests, you don't want to repeat `.baseUri()`, `.header(\"Authorization\", token)`, or `.contentType(ContentType.JSON)` in every single `given()` block. That's a maintenance nightmare. 
To solve this, I use `RequestSpecification`. It's an interface in REST Assured that allows you to bundle common request configurations together. I usually create a `TestBase` class that builds this specification. 
```java
RequestSpecification reqSpec = new RequestSpecBuilder()
    .setBaseUri(\"https://api.project.com\")
    .addHeader(\"Authorization\", \"Bearer \" + token)
    .setContentType(ContentType.JSON)
    .build();
    
given().spec(reqSpec).get(\"/dashboard\");
```
Similarly, I use `ResponseSpecification` to group common assertions, like expecting a 200 status code and verifying the response time is under 2000ms. It keeps the actual test classes incredibly short, readable, and perfectly DRY (Don't Repeat Yourself)."

### Section 3: API Testing Concepts

**Q: 26. 🔥 What is the difference between API testing and UI testing?**
| Feature | API Testing | UI Testing |
|---------|------------|------------|
| Focus | Business Logic / Data processing | Visuals / User Interaction |
| Speed | Extremely fast (milliseconds) | Very slow (seconds/minutes) |
| Stability | Highly stable | Flaky (element changes, waits) |
| Tooling | REST Assured, Postman | Selenium, Playwright, Cypress |

**Verbal explanation:** "In our test pyramid, API testing forms the thick middle layer, while UI testing is the thin top layer. The way I explain it is: API testing validates the 'brain' of the application (the business rules and database interactions), while UI testing validates the 'face' (buttons, layouts, user flows). 
From my experience, API tests are blazing fast. I can run 500 API tests in the time it takes to run 10 UI tests. They are also much more reliable. A UI test might fail because a button didn't render fast enough or a CSS class changed, but an API test only fails if the actual business logic breaks. Therefore, we try to push as much functional validation down to the API layer as possible, and reserve UI automation strictly for end-to-end user journeys and cross-browser compatibility."

### PART B: MAVEN

**Q: 31. 🔥 What is Maven? Why do we use it in Selenium projects?**
**A:** "Maven is a build automation and project management tool. In my Selenium projects, I consider Maven the backbone of the framework. Without it, setting up a project is a nightmare. 
We use it primarily for two things: Dependency Management and Build Lifecycle. Before Maven, if I needed Selenium and TestNG, I had to manually download the JAR files, add them to the build path, and hope I didn't get a `NoClassDefFoundError` due to version conflicts. With Maven, I just paste a few lines of XML into the `pom.xml`, and it automatically downloads the JARs and all their transitive dependencies from the central repository. 
Secondly, it standardizes the build process. Whether a new developer joins the team or Jenkins triggers a build in the cloud, all they have to do is run `mvn clean test`. Maven knows exactly how to compile the code, execute the TestNG suite via the Surefire plugin, and generate the results."

**Q: 32. 🔥 What is pom.xml? Explain its structure**
**A:** "The `pom.xml` (Project Object Model) is the heart of any Maven project. It's an XML file that contains information about the project and configuration details used by Maven to build the project.
When I set up a new framework, I structure the POM into a few key sections:
1. **Project Coordinates**: The `groupId`, `artifactId`, and `version` which uniquely identify my project.
2. **Properties**: I define Java compiler versions and dependency versions here (e.g., `<selenium.version>4.10.0</selenium.version>`). This ensures I only have to update a version number in one place.
3. **Dependencies**: This is where I list Selenium, REST Assured, TestNG, ExtentReports, etc.
4. **Build/Plugins**: Here, I configure plugins like the `maven-compiler-plugin` (to specify Java 11 or 17) and the `maven-surefire-plugin` (to hook up my `testng.xml` file so Maven knows which test suites to run). Without a well-organized POM, a framework becomes unmaintainable very quickly."

**Q: 33. 🔥 What is Maven lifecycle? Explain clean, compile, test, package, install, deploy**
| Phase | Action Performed |
|-------|------------------|
| clean | Deletes the `target` directory containing previous build outputs |
| compile | Compiles the source code (`src/main/java`) into `.class` files |
| test | Runs unit/integration tests using Surefire plugin |
| package | Packages compiled code into a distributable format like JAR/WAR |
| install | Installs the package into the local Maven repository |
| deploy | Copies the final package to a remote repository (like Nexus) |

**Verbal explanation:** "In Maven, the build lifecycle is sequential. From my experience, understanding this sequence is crucial for CI/CD setup. If I run `mvn test`, Maven doesn't just run the tests. It first validates the POM, then compiles the main code, compiles the test code, and ONLY then runs the tests. 
What we typically do locally or on Jenkins is run `mvn clean test`. The `clean` phase wipes out the old 'target' folder so we start fresh, ensuring no stale class files affect the results. If we are building a library for other teams to use, we use `mvn install` to put the JAR in our local `.m2` folder, or `mvn deploy` to push it to a remote Nexus repository. As SDETs, we spend 95% of our time in the `clean`, `compile`, and `test` phases."

**Q: 34. 🔥 What is the difference between compile, test, and provided scope?**
| Scope | Description | Included in build? |
|-------|-------------|--------------------|
| compile | Default. Needed for core code to run. | Yes |
| test | Only needed for test compilation/execution. | No (not in final jar) |
| provided| Needed for compilation, but container provides it at runtime. | No |

**Verbal explanation:** "Scopes in Maven dictate when a dependency is available. In a test automation framework, we mainly deal with `compile` and `test` scopes. 
By default, if you don't specify a scope, it's `compile`. This means the library is available everywhere—in your `src/main/java` and `src/test/java`. 
However, for libraries like TestNG or REST Assured, we explicitly set the scope to `<scope>test</scope>`. This tells Maven: 'Hey, I only need these for running my test scripts. Don't package them into the final production artifact.' This keeps the project lightweight. The `provided` scope is more for developers—for example, if they need the Servlet API to compile the code, but they know Tomcat will provide it at runtime, they use `provided` so it isn't bundled twice."

**Q: 36. 🔥 What is the difference between mvn clean test and mvn test?**
**A:** "This is a very common question, and in my daily routine, I almost always use `mvn clean test`. 
If you just run `mvn test`, Maven will compile any changed files and run the tests. However, it leaves the old `.class` files and generated reports from previous runs in the `target` directory. Sometimes, this causes weird caching issues or false positives in test execution, especially if you renamed or deleted a class but the old compiled version is still sitting there.
When you run `mvn clean test`, Maven first triggers the 'clean' lifecycle, which completely deletes the `target` folder. Then it moves on to the 'default' lifecycle, recompiling everything from scratch and running the tests. In our Jenkins pipelines, we strictly enforce `mvn clean test` to ensure that every build is running on a pristine, isolated environment without any garbage from the previous build."

**Q: 37. What are Maven plugins? Name the ones you've used (surefire, compiler)**
**A:** "In Maven, the core engine actually does very little on its own. All the heavy lifting is done by plugins. Every lifecycle phase is bound to a specific plugin goal. 
In my automation framework, the two most critical plugins are the `maven-compiler-plugin` and the `maven-surefire-plugin`. I configure the compiler plugin to specify which Java version we are using, for example, Java 17. Without it, Maven might default to an older version and my code won't compile. 
The `maven-surefire-plugin` is absolutely essential for us SDETs because it is the bridge between Maven and TestNG. In my `pom.xml`, I configure Surefire to point to my `testng.xml` suite file. This is how I can execute specific suites from the command line, like `mvn clean test -DsuiteXmlFile=smoke.xml`."

**Q: 39. What is Maven repository? Local vs Central vs Remote?**
| Repository Type | Location | Purpose |
|-----------------|----------|---------|
| Local | On your machine (`~/.m2/repository`) | Caches downloaded dependencies for faster builds |
| Central | Maven community servers | Global public repository containing open-source libraries |
| Remote | Company internal servers (Nexus, Artifactory) | Hosts proprietary internal libraries |

**Verbal explanation:** "A Maven repository is essentially a directory where all project jars, plugins, and dependencies are stored. 
When I add a new dependency like Selenium to my `pom.xml` and run `mvn compile`, Maven first checks my Local repository, which is a hidden `.m2` folder on my C: drive. If it finds Selenium there, it uses it, which is why subsequent builds are much faster. 
If it doesn't find it locally, Maven reaches out over the internet to the Central repository—the massive public database maintained by the Maven community—and downloads it. Finally, a Remote repository is something companies host internally. In my project, we use an internal Nexus repo to store custom proprietary JAR files that we cannot share publicly on the Central repo."

**Q: 40. What is Maven profile? How do you use it for multiple environments?**
**A:** "Maven profiles are an excellent way to customize a build for different environments. In my framework, we run tests across QA, UAT, and PROD environments. Instead of hardcoding URLs, I define multiple `<profile>` tags in the `pom.xml`.
Inside a 'QA' profile, I might set properties like `<env.url>qa.myapp.com</env.url>` and point Surefire to a `qa-testng.xml` file. Inside a 'UAT' profile, I change those properties accordingly. 
When I trigger the build, I use the `-P` flag to specify the profile, like `mvn clean test -P qa` or `mvn clean test -P uat`. What we typically do is pass this parameter dynamically from Jenkins dropdowns. It's an incredibly clean way to maintain one single codebase that can execute seamlessly across infinite environments without changing a single line of Java code."

**Q: 44. What is the difference between Maven and Gradle?**
| Feature | Maven | Gradle |
|---------|-------|--------|
| Configuration | XML (`pom.xml`) | Groovy / Kotlin DSL (`build.gradle`) |
| Performance | Slower | Faster (uses incremental builds and daemon) |
| Flexibility | Rigid, convention over configuration | Highly flexible and customizable |
| Learning Curve | Easy, standard lifecycle | Steeper due to custom scripting |

**Verbal explanation:** "In my experience, Maven is the industry standard for Java projects, primarily because its strict structure means you always know where things are. If you know one Maven project, you know them all. However, it uses XML, which can become incredibly verbose and hard to read.
Gradle, which we recently started using for our mobile automation (Appium) projects, is much more modern. It uses Groovy or Kotlin, which allows you to write actual logic, like if/else loops, right in the build file. It's also significantly faster because it has a daemon running in the background and uses an incremental build cache—it only rebuilds what has changed. If performance and high customization are the goals, Gradle wins. But for standard enterprise test automation, Maven’s simplicity makes it the default choice."

**Q: 45. How do you integrate Maven with Jenkins?**
**A:** "Integrating Maven with Jenkins is the cornerstone of our CI/CD pipeline. The way I handle this is quite straightforward. 
First, I ensure that Jenkins has Java and Maven installed via its Global Tool Configuration. Then, I create a Freestyle or Pipeline job. If it's a Jenkinsfile pipeline, I configure the build stage using a simple shell command: `sh 'mvn clean test'`. 
Before running the tests, Jenkins automatically pulls the latest code from Git. It then reads the `pom.xml`, downloads any missing dependencies, and triggers the Surefire plugin. After execution, I use the Jenkins Post-Build Actions to publish TestNG or Allure reports. If the `mvn test` command returns a non-zero exit code (meaning tests failed), Jenkins automatically marks the build as unstable or failed, and sends a Slack notification to the QA team."

### PART C: GIT

**Q: 46. 🔥 What is Git? What is the difference between Git and GitHub?**
| Feature | Git | GitHub |
|---------|-----|--------|
| Nature | Version Control System (Software) | Hosting Service (Website) |
| Location | Local machine | Cloud / Web |
| Purpose | Tracks code history and branches | Hosts Git repositories for collaboration |

**Verbal explanation:** "In interviews, I always clarify that Git and GitHub are completely different things. Git is the actual underlying engine—it's a distributed Version Control System installed locally on my machine. It tracks every single line of code I change, allows me to create branches, and commits my work. 
GitHub, on the other hand, is just a cloud-based hosting service for Git repositories. In my team, I use Git on my laptop to write my Selenium scripts and commit them. Then, I use the `git push` command to upload my local Git history to GitHub, so my teammates can see my code, review it, and pull it down to their machines. You can use Git without GitHub, but you can't use GitHub without Git."

**Q: 47. 🔥 Explain the Git workflow — working directory, staging area, local repo, remote repo**
**A:** "The Git architecture essentially has four main areas, and moving code between them is my daily workflow.
1. **Working Directory**: This is where my files currently live in my IDE. When I write a new Selenium test, it's just sitting in the working directory as 'untracked' or 'modified'.
2. **Staging Area (Index)**: This is the preparation zone. When I run `git add .`, Git takes a snapshot of my changed files and places them here. I can selectively stage files to group related changes.
3. **Local Repository**: Once I'm happy with what's staged, I run `git commit -m 'message'`. This permanently saves the changes to my local machine's Git history. The code is now safely version-controlled, but still only on my laptop.
4. **Remote Repository**: To share my work with the team, I run `git push`. This transfers my local commits to GitHub or GitLab. Understanding this flow is critical because it tells you exactly how to undo mistakes at any stage."

**Q: 48. 🔥 What are the most common Git commands you use daily?**
**A:** "In my day-to-day work as an SDET, my workflow involves a standard set of commands. 
1. `git pull origin main`: I run this first thing in the morning to get the latest framework updates from my team.
2. `git checkout -b feature/login-tests`: I use this to create a new branch and switch to it immediately so I don't write code on the main branch.
3. `git status`: I run this constantly to see which files I've modified.
4. `git add .`: Once my tests are working, this stages all my changes.
5. `git commit -m "Add valid login API test"`: This saves my changes locally with a descriptive message.
6. `git push origin feature/login-tests`: Finally, this uploads my branch to the remote repository so I can raise a Pull Request. If I make a mistake, I rely heavily on `git log` to check the history and `git stash` if I need to quickly switch branches without committing incomplete work."

**Q: 49. 🔥 What is the difference between git pull and git fetch?**
| Feature | git fetch | git pull |
|---------|-----------|----------|
| Action | Downloads new data from remote repo | Downloads AND merges data into current branch |
| Safety | Safe (doesn't modify local workspace) | Can be dangerous (might cause merge conflicts) |
| Under the hood | Updates remote-tracking branches | Runs `git fetch` followed by `git merge` |

**Verbal explanation:** "In my team, we are very careful with these two commands. `git fetch` is what I call the 'safe lookup'. When I run `git fetch`, Git goes to GitHub and downloads all the latest commits and branches, but it DOES NOT touch my actual code files. It just updates my knowledge of the remote repository. 
`git pull`, on the other hand, is aggressive. It runs `git fetch` and then immediately tries to `git merge` those new changes into my active local branch. If I have uncommitted work, `git pull` will often trigger a messy merge conflict. From my experience, if you are unsure of what changes are incoming, it's a best practice to run `git fetch` first, check the history using `git log`, and then manually merge."

**Q: 50. 🔥 What is the difference between git merge and git rebase?**
| Feature | Git Merge | Git Rebase |
|---------|-----------|------------|
| History | Creates a new merge commit | Rewrites history, linear progression |
| Traceability | Preserves exact history of branches | Cleans up history, loses branch context |
| Risk | Safe | Dangerous if used on shared branches |

**Verbal explanation:** "Both merge and rebase are used to integrate changes from one branch into another, but they do it differently. 
What we typically do is use `git merge`. If I merge 'main' into my 'feature' branch, Git takes the two endpoints, ties them together, and creates a new 'merge commit'. The history is perfectly preserved, but it can look messy and tangled with lots of branches.
`git rebase`, however, takes my feature branch commits and literally replays them on top of the tip of the 'main' branch. It creates a beautifully straight, linear history without any merge commits. But, the edge case—and danger—is that rebase rewrites commit hashes. Therefore, our team rule is: you can rebase your local, private branches to keep them clean, but NEVER rebase a public/shared branch that other developers have already pulled, otherwise you will destroy their local histories."

**Q: 51. 🔥 What is a merge conflict? How do you resolve it?**
**A:** "A merge conflict happens when Git cannot automatically figure out how to combine changes. From my experience, this usually occurs when two people modify the exact same line in the exact same file, or one person deletes a file while another modifies it. 
When I encounter a conflict during a `git pull` or `git merge`, Git stops and says 'Conflict in TestBase.java'. Here is how I step-by-step resolve it:
1. I open the conflicting file in my IDE (like IntelliJ).
2. I look for the conflict markers: `<<<<<<< HEAD` (my local changes), the `=======` separator, and `>>>>>>> branch-name` (the incoming changes).
3. I analyze the code, talk to the other SDET if necessary, and manually edit the file to keep the correct lines. Often, I need a combination of both our changes.
4. I delete all the `<<<<<<<` and `=======` markers.
5. Once fixed, I run `git add TestBase.java` to mark it resolved, and then `git commit` to finalize the merge."

**Q: 52. 🔥 What is the difference between git reset and git revert?**
| Feature | git reset | git revert |
|---------|-----------|------------|
| Mechanism | Rewrites commit history backwards | Creates a new commit moving forwards |
| History | Destroys/removes previous commits | Preserves history, adds an undo commit |
| Usage | Safe for local, private branches | Safe for public, shared branches |

**Verbal explanation:** "This is a classic disaster recovery question. If I make a bad commit and I need to undo it, I have to choose between reset and revert.
If the commit is only on my local machine and I haven't pushed it to GitHub yet, I use `git reset --hard HEAD~1`. This physically erases the last commit from my history as if it never existed.
However, if I already pushed that bad commit to a shared `main` branch, I cannot use reset because it will rewrite history and break my colleagues' local environments. Instead, I use `git revert <commit-hash>`. Revert creates a brand new, forward-moving commit that applies the exact opposite changes of the bad commit. The mistake stays in the history, but its effects are neutralized, keeping the repository completely safe and stable."

**Q: 54. 🔥 What is branching strategy? Explain GitFlow**
**A:** "A branching strategy is a set of rules a team follows for creating, naming, and merging branches to avoid chaos. In my current project, we use a variation of GitFlow, which is very popular in enterprise environments.
Here is how it works:
- `main` branch is strictly production-ready code. We never commit here directly.
- `develop` branch is the integration branch for the next release.
- When I pick up a Jira ticket to automate tests, I create a `feature` branch off `develop` (e.g., `feature/login-tests`). Once done, I raise a PR to merge it back to `develop`.
- When `develop` is stable enough for a release, we branch off a `release` branch. Only bug fixes go here.
- If a critical bug is found in production, we create a `hotfix` branch directly from `main`, fix it, and merge it back into both `main` and `develop`. This strategy ensures that ongoing QA work never interferes with production deployments."

**Q: 62. 🔥 What is pull request (PR)? What is code review process?**
**A:** "A Pull Request, or PR, is a formal request to merge my feature branch into the main repository branch. In my project, we have a strict rule: nobody pushes directly to `main`. 
When I finish automating a test, I push my branch to GitHub and open a PR. This acts as a gateway for Code Review. I assign at least one senior SDET to review my code. During the code review process, they check for several things: Is the code following our Page Object Model standards? Are the assertions correct? Are there hardcoded waits? Did I miss any edge cases? 
They leave comments directly on the lines of code. I address those comments, push fixes, and once they approve it, my PR can be merged. Furthermore, opening a PR automatically triggers our Jenkins pipeline to run a sanity suite on my branch to ensure my new code didn't break existing tests before it even gets merged."
