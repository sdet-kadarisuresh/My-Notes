# Infosys L2 SDET Interview Preparation: Core Java

## Section 1: OOPs Concepts (MOST IMPORTANT)

**Q: 1. What are the 4 pillars of OOPs? Explain each with real-time examples** 🔥
**A:** In my experience, explaining the 4 pillars is all about connecting them to what we do daily. 
First is **Encapsulation** — which is basically data hiding and bundling data and methods together. In our automation framework, we use POJO classes for API payloads where variables are private and accessed via public getters/setters. 
Second is **Inheritance** — acquiring properties of a parent class. We typically have a `BaseTest` class containing driver initialization and teardown methods, and all our test classes extend this `BaseTest` so we don't rewrite that logic. 
Third is **Polymorphism** — taking multiple forms. Method overloading (compile-time) is heavily used in our utility classes, like `clickElement(By locator)` vs `clickElement(WebElement element)`. Method overriding (runtime) is used when we implement interface methods differently. 
Fourth is **Abstraction** — hiding implementation details. We define a `WebDriver` interface, and we don't care how `ChromeDriver` or `FirefoxDriver` implements the `get()` method, we just use it. This keeps our code clean and maintainable.

**Q: 2. What is Encapsulation? Real example from your project** 🔥
**A:** Encapsulation is the mechanism of wrapping the data (variables) and code acting on the data (methods) together as a single unit. From my project experience, the best example is the Page Object Model (POM) design pattern. In a Page Class, say `LoginPage`, we declare the WebElements (locators) as `private`. This prevents test classes from directly accessing and modifying the locators. We then provide `public` methods like `enterUsername()` or `clickLogin()`. By doing this, we control how the data is accessed or modified. If the UI changes tomorrow, I only update the locator in the Page Class, and none of the test classes are affected. It perfectly demonstrates data hiding and security within our framework.

**Q: 3. What is Inheritance? Types of inheritance in Java** 🔥
**A:** Inheritance is when one class acquires the properties and behaviors of another class. It's the foundation for code reusability. In Java, we have Single, Multilevel, and Hierarchical inheritance. We don't have Multiple inheritance with classes to avoid the Diamond problem. In my automation projects, we use **Hierarchical inheritance** extensively. We have a parent `BaseTest` class handling `@BeforeSuite`, `@AfterSuite`, driver setups, and reporting initialization. Then we have multiple child classes like `LoginTests`, `CheckoutTests` extending `BaseTest`. This means I don't have to write driver initialization in every test class. What we typically do is keep all common utilities in the parent, making the child classes focused strictly on test logic.

**Q: 4. What is Polymorphism? Compile-time vs Runtime polymorphism with code** 🔥
**A:** Polymorphism means "many forms". It allows us to perform a single action in different ways. 
Compile-time polymorphism is achieved through **Method Overloading**. The compiler decides which method to call based on method signature. 
Runtime polymorphism is achieved through **Method Overriding**. The JVM decides which method to call at runtime based on the actual object type.
In my framework, overloading looks like this:
```java
public void waitAndClick(WebElement element) { ... }
public void waitAndClick(By locator, int timeout) { ... }
```
Overriding is when we implement an interface. For instance, if we have a `ReportListener` implementing `ITestListener`, we override `onTestFailure()` to capture screenshots. The framework calls our specific implementation at runtime.

**Q: 5. What is Abstraction? How did you use it in your framework?** 🔥
**A:** Abstraction is the process of hiding the implementation details and showing only the functionality to the user. You only know *what* it does, not *how* it does it. In our Selenium framework, we use abstraction constantly without even thinking about it. The `WebDriver` itself is an interface. When I call `driver.findElement()`, I don't know the internal HTTP calls it makes to the browser driver; I just know it returns a WebElement. 
In my custom code, we created an interface `DatabaseOperations` with methods like `connect()`, `executeQuery()`, and `disconnect()`. We then had concrete classes like `MySQLOperations` and `OracleOperations` implementing it. The test scripts only interacted with the interface, completely abstracted from the SQL specifics of each database.

**Q: 6. What is the difference between Abstract class and Interface?** 🔥
| Feature | Abstract Class | Interface |
|---------|----------|----------|
| **Methods** | Can have both abstract and concrete methods. | Prior to Java 8, only abstract methods. Now can have default/static methods. |
| **Variables** | Can have final, non-final, static, non-static variables. | Variables are implicitly `public static final`. |
| **Inheritance** | A class can extend only one abstract class. | A class can implement multiple interfaces. |
| **Constructors** | Can have constructors. | Cannot have constructors. |

**Verbal explanation:** The way I explain this in interviews is to look at the relationship. Use an abstract class when classes share a strong "IS-A" relationship and common state/behavior. For instance, `BasePage` abstract class can have a constructor to initialize the driver and `PageFactory`, plus abstract methods like `waitForPageLoad()`. Use an interface when you want to define a contract for classes that might be completely unrelated, like a `RetryAnalyzer` interface in TestNG. An interface provides complete abstraction, while an abstract class provides partial abstraction.

**Q: 7. When to use abstract class vs interface? Give real project scenario**
**A:** From my experience, we use an **Interface** when we want to establish a strict contract across unrelated classes. For example, in our API framework, we created an `IAuthenticator` interface with a `getToken()` method. Both `OAuthAuthenticator` and `BasicAuthenticator` implemented it, though they shared no common code. 
We use an **Abstract Class** when we want to share common state and default behavior among closely related classes. For example, our `BaseAPIClient` abstract class holds the RestAssured `RequestSpecification` object and a concrete method `initRequest()`, but leaves the `executeRequest()` as an abstract method for child classes (like `UserAPIClient` or `OrderAPIClient`) to implement based on their specific endpoints.

**Q: 8. What is method overloading vs method overriding?** 🔥
| Feature | Method Overloading | Method Overriding |
|---------|----------|----------|
| **Location** | Happens within the same class. | Happens between superclass and subclass. |
| **Signature** | Method name must be same, but arguments MUST be different. | Method name and arguments MUST be exactly the same. |
| **Binding** | Static/Compile-time binding. | Dynamic/Runtime binding. |
| **Return Type**| Can be different. | Must be same or covariant type. |

**Verbal explanation:** What we typically do in our utility classes is Method Overloading — writing multiple `waitForElement` methods that take different parameters (one with just the element, another with element and custom timeout). It improves readability. Method Overriding is entirely about inheritance. If the parent class has a generic `login()` method, but the admin module needs an extra step, the `AdminLoginPage` extends the parent and overrides the `login()` method to provide the admin-specific implementation.

**Q: 9. Can we override static methods? Can we override private methods? Why?**
**A:** The short answer is No to both. 
For **static methods**, they belong to the class, not the object. Overriding relies on dynamic method dispatch at runtime based on the object type. Since static methods are resolved at compile time using the class reference, they cannot be overridden. If you declare the same static method in a subclass, it's called Method Hiding, not overriding.
For **private methods**, they are simply not visible outside the class they are defined in. Since a subclass cannot see the private methods of its parent, it obviously cannot override them. In our framework, we keep helper methods private so they aren't accidentally exposed or overridden by child classes.

**Q: 10. What is constructor? Types of constructors**
**A:** A constructor is a special block of code that is called when an object is instantiated. Its main purpose is to initialize the newly created object. The name must perfectly match the class name, and it has no return type. 
There are three types: 
1. **Default Constructor**: Provided by the JVM if we don't write any. It initializes variables to default values.
2. **No-arg Constructor**: Explicitly written by us but takes no arguments. 
3. **Parameterized Constructor**: Takes arguments to initialize object with specific values.
In our framework, we heavily use parameterized constructors in Page Object classes. When we create an object of `HomePage`, we pass the `WebDriver` instance to its constructor:
```java
public HomePage(WebDriver driver) {
    this.driver = driver;
    PageFactory.initElements(driver, this);
}
```

**Q: 11. Can constructor be inherited? Can we have constructor in abstract class?**
**A:** No, **constructors cannot be inherited** in Java. Subclasses inherit properties and methods, but they must define their own constructors. However, a subclass constructor always invokes the parent class constructor (implicitly or explicitly via `super()`) to ensure the inherited properties are initialized correctly.
Yes, **we CAN have a constructor in an abstract class**. Even though we cannot instantiate an abstract class directly, when we create an object of its concrete subclass, the subclass constructor will call the abstract class's constructor. In my project, our abstract `BasePage` class has a constructor that initializes the `WebDriverWait` object, so every child page class automatically gets a configured wait object.

**Q: 12. What is 'this' keyword vs 'super' keyword?** 🔥
| Feature | `this` keyword | `super` keyword |
|---------|----------|----------|
| **Reference** | Refers to the current class instance variable/method. | Refers to the immediate parent class instance variable/method. |
| **Constructor**| `this()` invokes current class constructor. | `super()` invokes parent class constructor. |
| **Usage** | Resolves naming collisions between instance variables and local variables. | Accesses overridden methods or hidden fields of the superclass. |

**Verbal explanation:** In my day-to-day coding, I use `this` all the time in setters or constructors. For example, `this.driver = driver;` ensures the class-level driver gets the value of the local driver parameter. I use `super` when dealing with inheritance. If I override a `setup()` method in a child class but still want the parent's setup to run first, I'll call `super.setup()` inside the child's method. 

**Q: 13. What is IS-A vs HAS-A relationship?**
**A:** These are fundamental ways objects relate to each other. 
An **IS-A** relationship is inheritance. It's implemented using `extends` or `implements`. For instance, `ChromeDriver IS-A WebDriver`. We use this when a class is a specialized version of another.
A **HAS-A** relationship is composition or aggregation. It means one class has a reference to another. For example, a `TestClass HAS-A WebDriver`. The test class doesn't extend the driver; it instantiates it and uses it. In our framework design, favor HAS-A over IS-A when possible because it provides better flexibility and avoids deep inheritance trees.

**Q: 14. Why does Java not support multiple inheritance? How does interface solve it?** 🔥
**A:** Java avoids multiple inheritance with classes to prevent the **Diamond Problem**. If Class C extends both Class A and Class B, and both A and B have a method called `display()`, the compiler gets confused about which `display()` to inherit. To keep the language simple and avoid this ambiguity, Java restricts classes from extending more than one class.
However, an interface solves this because interface methods (prior to Java 8) are purely abstract—they have no body. If Class C implements Interface A and Interface B, and both have a `display()` method signature, there is no conflict because the implementation only exists in Class C. The ambiguity is completely removed.


## Section 2: String Handling

**Q: 15. What is the difference between String, StringBuilder, and StringBuffer?** 🔥
| Feature | String | StringBuffer | StringBuilder |
|---------|----------|----------|----------|
| **Mutability**| Immutable | Mutable | Mutable |
| **Thread-Safety**| Yes (due to immutability)| Yes (methods are synchronized) | No (not synchronized) |
| **Performance**| Slow for concatenations | Slower than StringBuilder | Fastest for concatenations |

**Verbal explanation:** From my experience, if the value is not going to change, I always use `String`. If I need to do a lot of string manipulations — like building an API request payload dynamically or generating random test data — I use `StringBuilder` because it's fast. I rarely use `StringBuffer` unless I'm explicitly dealing with a multi-threaded scenario where multiple threads are appending to the same string object simultaneously, which is uncommon in standard test scripts.

**Q: 16. Why is String immutable in Java? What is String pool?** 🔥
**A:** Immutability means once a String object is created, its value cannot be changed. If you try to modify it, a new object is created. Java designed it this way primarily for memory optimization and security. 
The **String Constant Pool (SCP)** is a special area in Heap memory. When we create a string like `String s1 = "Infosys";`, Java checks the pool. If "Infosys" exists, `s1` just points to the existing reference. If it doesn't, it creates a new one. Because strings are immutable, multiple references can safely point to the same object without worrying that one reference will change the value and break the others. This saves massive amounts of memory. It's also critical for security, as strings are used for database URLs, usernames, and passwords.

**Q: 17. What is the difference between == and .equals()?** 🔥
| Feature | `==` Operator | `.equals()` Method |
|---------|----------|----------|
| **Functionality**| Compares object references (memory addresses). | Compares the actual content/value of the objects. |
| **Usage** | Primitives and object references. | Only objects (classes that override it). |

**Verbal explanation:** This is a classic trap. If I have `String s1 = new String("Test");` and `String s2 = new String("Test");`, using `s1 == s2` will return `false` because the `new` keyword creates two distinct objects in the heap memory with different addresses. However, `s1.equals(s2)` will return `true` because the characters inside the strings are identical. In automation, when we assert UI text against expected text, we MUST always use `.equals()` or `.equalsIgnoreCase()`, never `==`.

**Q: 18. How to reverse a String in Java? (3 different ways)**
**A:** In interviews, I usually outline the built-in way first, then the logical ways.
**Way 1: Using StringBuilder (Easiest)**
```java
String str = "automation";
String reversed = new StringBuilder(str).reverse().toString();
```
**Way 2: Using Character Array**
```java
String str = "automation";
char[] chars = str.toCharArray();
for (int i = chars.length - 1; i >= 0; i--) {
    System.out.print(chars[i]);
}
```
**Way 3: Using charAt()**
```java
String str = "automation";
String rev = "";
for (int i = str.length() - 1; i >= 0; i--) {
    rev += str.charAt(i);
}
```
In real projects, I stick to `StringBuilder` for readability and performance.

**Q: 19. How to check if a String is palindrome?**
**A:** A palindrome reads the same forwards and backwards, like "madam". The most efficient way is to reverse the string and compare it to the original using `.equals()`.
```java
public boolean isPalindrome(String str) {
    if (str == null) return false;
    String reversed = new StringBuilder(str).reverse().toString();
    return str.equalsIgnoreCase(reversed);
}
```
Alternatively, for interviews, you can use the two-pointer approach which is more memory efficient because it doesn't create a new string:
```java
public boolean isPalindromeOpt(String str) {
    int left = 0, right = str.length() - 1;
    while (left < right) {
        if (str.charAt(left) != str.charAt(right)) return false;
        left++; right--;
    }
    return true;
}
```

**Q: 20. How to count characters/words in a String?**
**A:** To count words, we generally use the `split()` method based on spaces.
```java
String text = "Welcome to Infosys interview";
String[] words = text.trim().split("\\s+"); // Handles multiple spaces
System.out.println("Word count: " + words.length);
```
To count specific character occurrences, say 'e':
```java
int count = 0;
for (char c : text.toCharArray()) {
    if (c == 'e') count++;
}
```
Alternatively, using Java 8 streams (which interviewers love):
```java
long count = text.chars().filter(ch -> ch == 'e').count();
```

**Q: 21. What is the difference between String.valueOf() and toString()?**
**A:** Both are used to convert objects or primitives into a String, but they handle `null` differently. 
If you call `.toString()` on a null reference, it will immediately throw a `NullPointerException`. This is dangerous in automation when fetching dynamic data that might be null.
On the other hand, `String.valueOf()` is null-safe. If you pass a null object to it, it simply returns the literal string `"null"` instead of crashing the program. What we typically do in our data-driven frameworks is use `String.valueOf(excelCellData)` to safely convert cell values regardless of whether they are numeric, boolean, or unexpectedly null.


## Section 3: Collections Framework (MOST ASKED)

**Q: 22. What is Collections Framework? Explain the hierarchy** 🔥
**A:** The Collections Framework provides a unified architecture to store and manipulate groups of objects. It offers interfaces, implementations (classes), and algorithms.
The root interface is `Iterable`, extended by `Collection`. From `Collection`, we have three main branches:
1. **List**: Ordered collection, allows duplicates. Implemented by `ArrayList`, `LinkedList`, `Vector`.
2. **Set**: Unordered collection, NO duplicates. Implemented by `HashSet`, `LinkedHashSet`. (Note: `TreeSet` implements `SortedSet`).
3. **Queue**: Ordered, follows FIFO. Implemented by `PriorityQueue`, `LinkedList`.
*Note*: `Map` is part of the framework but does NOT extend `Collection`. It stores key-value pairs. Implemented by `HashMap`, `LinkedHashMap`, `TreeMap`, `HashTable`.

**Q: 23. ArrayList vs LinkedList — when to use which?** 🔥
| Feature | ArrayList | LinkedList |
|---------|----------|----------|
| **Internal Data Structure**| Resizable Array | Doubly Linked List |
| **Search/Retrieval**| Very fast (O(1)) as it uses index. | Slow (O(n)) as it traverses nodes. |
| **Insert/Delete**| Slow, as elements must be shifted. | Fast, only node pointers are updated. |

**Verbal explanation:** In test automation, 95% of the time I use `ArrayList`. For example, when `driver.findElements()` returns a list of web elements, we just need to iterate through them or get an element by index. `ArrayList` is perfect for this. I would only choose `LinkedList` if I had a scenario requiring frequent insertions and deletions in the middle of a massive list, which is extremely rare in UI or API testing.

**Q: 24. ArrayList vs Vector** 🔥
| Feature | ArrayList | Vector |
|---------|----------|----------|
| **Thread Safety**| Not synchronized (not thread-safe). | Synchronized (thread-safe). |
| **Performance**| Fast because no locks are acquired. | Slow due to thread synchronization overhead. |
| **Growth Rate**| Increases capacity by 50% when full. | Doubles capacity (100%) when full. |

**Verbal explanation:** `Vector` is a legacy class. We almost never use it in modern Java development. If we need a thread-safe list in a multithreaded framework (like parallel test execution in TestNG), we prefer using `CopyOnWriteArrayList` or `Collections.synchronizedList()` instead of `Vector` because they offer better concurrency performance.

**Q: 25. HashMap vs Hashtable** 🔥
| Feature | HashMap | Hashtable |
|---------|----------|----------|
| **Thread Safety**| Non-synchronized (Not thread-safe). | Synchronized (Thread-safe). |
| **Null Keys/Values**| Allows ONE null key and multiple null values. | Does NOT allow any null key or value (Throws NPE). |
| **Performance**| Faster. | Slower due to locking. |

**Verbal explanation:** Just like `Vector`, `Hashtable` is considered legacy. In my projects, I use `HashMap` extensively. For instance, passing a set of query parameters to a RestAssured API request or reading config properties. If I am running parallel tests and multiple threads need to update the map simultaneously, I don't use `Hashtable`; instead, I use `ConcurrentHashMap`, which provides thread safety without locking the entire map.

**Q: 26. HashMap vs LinkedHashMap vs TreeMap** 🔥
| Map Implementation | Ordering | Performance | Internal Structure |
|---------|----------|----------|----------|
| **HashMap** | No guarantee of insertion order. | Fastest (O(1)) | Hashtable |
| **LinkedHashMap** | Maintains insertion order. | Slightly slower than HashMap | Hashtable + Doubly Linked List |
| **TreeMap** | Sorts keys in ascending order (Natural ordering). | Slower (O(log n)) | Red-Black Tree |

**Verbal explanation:** If I just need key-value storage and don't care about order, `HashMap` is my go-to. However, if I am reading Excel columns into a Map and I need the headers to stay in the exact order they were read, I use `LinkedHashMap`. If I need the keys sorted alphabetically—for example, sorting employee names retrieved from a database before validating them—I use `TreeMap`.

**Q: 27. HashSet vs LinkedHashSet vs TreeSet** 🔥
| Set Implementation | Ordering | Null Elements | Internal Structure |
|---------|----------|----------|----------|
| **HashSet** | Unordered. | Allows one null. | HashMap |
| **LinkedHashSet** | Maintains insertion order. | Allows one null. | LinkedHashMap |
| **TreeSet** | Sorted in ascending order. | Does NOT allow null. | TreeMap |

**Verbal explanation:** In my automation framework, if I want to collect all window handles using `driver.getWindowHandles()`, Selenium returns a `Set<String>` (specifically a `LinkedHashSet` to maintain order). If I extract a list of unique product prices from a web page and need to verify they are sorted low to high, throwing them into a `TreeSet` is a great trick because it automatically sorts them.

**Q: 28. List vs Set vs Map — differences** 🔥
| Feature | List | Set | Map |
|---------|----------|----------|----------|
| **Duplicates** | Allows duplicate elements. | Cannot contain duplicates. | Keys must be unique, values can duplicate. |
| **Ordering** | Maintains insertion order. | Generally unordered (except Linked/Tree variants). | Unordered (except Linked/Tree variants). |
| **Data Storage** | Stores single elements. | Stores single elements. | Stores Key-Value pairs. |

**Verbal explanation:** This is the core of Collections. Use a `List` when order matters and duplicates are fine (like a list of WebElements). Use a `Set` when you need uniqueness (like collecting unique user IDs from a table). Use a `Map` when data has a relationship (like matching an expected JSON key to its value).

**Q: 30. How does HashMap work internally?** 🔥
**A:** This is a crucial concept. Internally, `HashMap` uses an array of Nodes (or buckets). Each Node contains a Key, Value, Hash, and a pointer to the next Node (LinkedList). 
When we call `put(key, value)`, Java calculates the `hashCode()` of the key to determine which bucket (array index) to store the data in. 
If the bucket is empty, the node is placed there. If there is already a node there (a **Hash Collision**), Java checks the `.equals()` method. If `.equals()` is true, it overrides the existing value. If false, it attaches the new node to the existing node, forming a LinkedList.
In Java 8, there's a performance improvement: if the LinkedList in a single bucket grows beyond 8 elements, it converts into a **Red-Black Tree** to reduce worst-case retrieval time from O(n) to O(log n).

**Q: 34. What is fail-fast vs fail-safe?** 🔥
| Feature | Fail-Fast Iterator | Fail-Safe Iterator |
|---------|----------|----------|
| **Modification**| Throws `ConcurrentModificationException` if collection is modified while iterating. | Does NOT throw exception if modified during iteration. |
| **Operation**| Works on the original collection directly. | Works on a clone/copy of the collection. |
| **Examples** | `ArrayList`, `HashMap`, `HashSet` | `ConcurrentHashMap`, `CopyOnWriteArrayList` |

**Verbal explanation:** In standard `ArrayList`s, if I am using an Iterator in a loop and I try to do `list.remove()` outside of the iterator's own remove method, the program crashes with a `ConcurrentModificationException`. This is fail-fast behavior. If I am dealing with multi-threading where one thread is iterating and another might add elements, I use `CopyOnWriteArrayList`. It creates a snapshot of the array for the iterator, so modifications don't crash it. This is fail-safe.


## Section 4: Exception Handling

**Q: 35. What is Exception Handling? Explain try-catch-finally with real example** 🔥
**A:** Exception handling is a mechanism to handle runtime errors so the normal flow of the application is not disrupted. In automation, if an element isn't found, we don't want the whole test suite to crash; we want to catch the error, log it, take a screenshot, and move to the next test.
```java
try {
    // Risky code
    WebElement btn = driver.findElement(By.id("submit"));
    btn.click();
} catch (NoSuchElementException e) {
    // Handling code
    logger.error("Submit button not found");
    takeScreenshot();
} finally {
    // Always executes
    driver.quit();
}
```
The `finally` block is crucial. Even if the test fails or throws an exception, the `finally` block executes, ensuring we always close the browser and release resources.

**Q: 36. Checked vs Unchecked exceptions** 🔥
| Feature | Checked Exceptions | Unchecked Exceptions |
|---------|----------|----------|
| **Checking**| Checked by compiler at compile-time. | Occur at runtime, not checked by compiler. |
| **Inheritance**| Extend `Exception` class directly. | Extend `RuntimeException` class. |
| **Handling**| MUST be handled via try-catch or `throws` keyword. | Handling is optional (best practice is to fix the logic). |
| **Examples**| `IOException`, `SQLException`, `InterruptedException` | `NullPointerException`, `ArithmeticException`, `NoSuchElementException` |

**Verbal explanation:** If I write `Thread.sleep(1000)`, Java immediately forces me to handle `InterruptedException`—that's a Checked Exception. It anticipates external issues. Conversely, if I have an array of size 5 and try to access index 10, I get `ArrayIndexOutOfBoundsException` at runtime. The compiler didn't warn me because it's an Unchecked Exception—it represents a programming flaw on my end.

**Q: 37. throw vs throws** 🔥
| Feature | `throw` | `throws` |
|---------|----------|----------|
| **Purpose**| Used to explicitly throw an exception object. | Used to declare that a method might throw an exception. |
| **Location**| Inside the method body. | In the method signature. |
| **Quantity**| Can throw only one exception at a time. | Can declare multiple exceptions separated by commas. |

**Verbal explanation:** What we typically do is use `throws` to push the responsibility to the caller. For example, `public void readFile() throws IOException`. 
We use `throw` when we want to trigger an exception deliberately. In my framework, if an API response returns status 500, I manually throw an exception: 
`if(response.statusCode() != 200) { throw new RuntimeException("API failed"); }`

**Q: 38. final vs finally vs finalize** 🔥
| Keyword | Type | Purpose |
|---------|----------|----------|
| **final** | Modifier | Applies to variables (constant), methods (prevent overriding), classes (prevent inheritance). |
| **finally** | Block | Used with try-catch. Code here executes regardless of whether exception occurs. Used for cleanup. |
| **finalize**| Method | Called by Garbage Collector just before an object is destroyed to release non-Java resources. (Deprecated in Java 9). |

**Verbal explanation:** These sound similar but are entirely different. I use `final` to declare constants like timeouts: `public static final int TIMEOUT = 10;`. I use `finally` to ensure `driver.quit()` or database connection closures are guaranteed to execute. I never use `finalize()` because modern Java handles memory efficiently, and its execution is unpredictable.

**Q: 40. What is custom exception? How to create one?**
**A:** Custom exceptions are user-defined exception classes to represent specific errors in our application. They make debugging much easier. Instead of generic `RuntimeExceptions`, we create meaningful ones. 
To create a checked exception, extend `Exception`. For unchecked, extend `RuntimeException`.
```java
// Creation
public class FrameworkSetupException extends RuntimeException {
    public FrameworkSetupException(String message) {
        super(message);
    }
}

// Usage in code
if (driver == null) {
    throw new FrameworkSetupException("WebDriver failed to initialize check browser properties");
}
```
This clearly tells the team exactly what went wrong when viewing the logs.


## Section 5: Java 8 Features

**Q: 42. What are the new features in Java 8?** 🔥
**A:** Java 8 was a massive update. In interviews, I highlight the top 5 we actually use in automation:
1. **Lambda Expressions**: Introduced functional programming, reducing boilerplate code.
2. **Stream API**: Revolutionized how we process Collections with operations like filter, map, and collect.
3. **Functional Interfaces**: Interfaces with exactly one abstract method (like `@FunctionalInterface`), enabling Lambdas.
4. **Default and Static Methods in Interfaces**: Allowed us to add new methods to interfaces without breaking implementing classes.
5. **Optional Class**: A great way to handle nulls and avoid `NullPointerExceptions`.

**Q: 43. What is Lambda expression? Explain with example** 🔥
**A:** A Lambda expression is essentially an anonymous function—a method without a name, return type, or access modifier. It provides a clear and concise way to implement a Functional Interface.
Before Java 8, if we wanted to create a thread using `Runnable`, we had to use an anonymous inner class:
```java
Runnable r = new Runnable() {
    public void run() { System.out.println("Thread running"); }
};
```
With Lambda, we write exactly the same logic in one clean line:
```java
Runnable r = () -> System.out.println("Thread running");
```
The syntax is `(parameters) -> { body }`. It dramatically reduces code bloat.

**Q: 45. What is Stream API? How do you use it?** 🔥
**A:** The Stream API is used to process sequences of elements (like Lists or Sets) in a declarative way. Instead of writing `for` loops and `if` conditions, we pipeline operations. It does not modify the original data structure.
In automation, if I extract a list of product names and want to find those starting with "Apple":
```java
List<String> products = Arrays.asList("Apple Mac", "Samsung TV", "Apple iPad");
List<String> appleProducts = products.stream()
    .filter(p -> p.startsWith("Apple"))
    .collect(Collectors.toList());
```
This replaces 5-6 lines of traditional loop logic with a single, highly readable statement.

**Q: 46. filter(), map(), collect(), forEach(), reduce()**
**A:** These are core Stream methods. 
`filter()` takes a Predicate (condition). It keeps elements that match and discards others. (e.g., filter prices > 100).
`map()` takes a Function. It transforms each element. (e.g., converting a list of String IDs to Integer IDs).
`collect()` is a terminal operation that gathers the stream results back into a List or Set.
`forEach()` iterates over each element to perform an action, like printing.
`reduce()` combines all elements into a single result, like calculating the sum of all prices in a list.


## Section 7: Miscellaneous Java

**Q: 55. What is the difference between JDK, JRE, and JVM?** 🔥
| Component | Full Form | Role | Contents |
|---------|----------|----------|----------|
| **JVM** | Java Virtual Machine | Executes the byte code line by line. | Just the execution engine, platform-dependent. |
| **JRE** | Java Runtime Environment | Provides the environment to RUN Java applications. | JVM + Core Libraries + Other runtime files. |
| **JDK** | Java Development Kit | Provides the environment to DEVELOP and RUN Java applications. | JRE + Development Tools (compiler `javac`, debugger, etc.). |

**Verbal explanation:** As an SDET, I install the **JDK** on my machine because I write and compile code. The CI/CD server (like Jenkins) technically only needs the **JRE** to run the compiled tests, but usually, we put JDK there too for maven compilation. The **JVM** is the heart of it all; it's what makes Java "Write Once, Run Anywhere" by translating the compiled `.class` bytecode into machine-specific instructions.

**Q: 57. What is static keyword?** 🔥
**A:** The `static` keyword in Java implies that a member belongs to the **class itself**, rather than instances (objects) of the class. This means memory is allocated only once.
1. **Static Variables**: Used for properties common to all objects. In my framework, `public static WebDriver driver;` is sometimes used (though threadLocal is better for parallel) so the same driver instance is shared.
2. **Static Methods**: Can be called without creating an object, like `Math.max()`. Utility methods like `ConfigReader.getProperty("url")` are made static so we can call them directly via class name.
3. **Static Blocks**: Executed exactly once when the class is loaded into memory, even before constructors. Useful for one-time setup tasks.

**Q: 65. What is the difference between Comparable and Comparator?** 🔥
| Feature | Comparable | Comparator |
|---------|----------|----------|
| **Package**| `java.lang` | `java.util` |
| **Method** | `compareTo(Object obj)` | `compare(Object obj1, Object obj2)` |
| **Logic** | Defines a single "natural" sorting sequence (e.g., sorting IDs). | Can define multiple custom sorting sequences. |
| **Modification**| Modifies the actual class whose objects are being sorted. | Does not modify the class; logic is written separately. |

**Verbal explanation:** If I have an `Employee` class and I implement `Comparable`, I have to override `compareTo` inside the Employee class itself. I can only sort by one logic, say Employee ID. But what if I want to sort by Name, and later by Salary? I shouldn't keep altering the class. Instead, I write multiple `Comparator` classes (`NameComparator`, `SalaryComparator`). I can then pass the specific comparator to `Collections.sort(list, new SalaryComparator())`. It's highly flexible.


## Section 8: Java Coding Questions

**Q: 66. Write a program to find duplicate elements in an array**
**A:** The most efficient way in an interview is using a `HashSet` because its `add()` method returns false if the element already exists.
```java
public class FindDuplicates {
    public static void main(String[] args) {
        String[] arr = {"Java", "Python", "C#", "Java", "Ruby", "Python"};
        Set<String> uniqueElements = new HashSet<>();
        
        for (String lang : arr) {
            // If add() returns false, it's a duplicate
            if (!uniqueElements.add(lang)) {
                System.out.println("Duplicate found: " + lang);
            }
        }
    }
}
```

**Q: 74. Write a program to find the first non-repeated character in a string**
**A:** This is a very common question. We use a `LinkedHashMap` to maintain insertion order and count occurrences.
```java
import java.util.LinkedHashMap;
import java.util.Map;

public class FirstNonRepeatedChar {
    public static void main(String[] args) {
        String str = "swiss";
        Map<Character, Integer> map = new LinkedHashMap<>();
        
        // Count characters
        for (char c : str.toCharArray()) {
            map.put(c, map.getOrDefault(c, 0) + 1);
        }
        
        // Find first with count 1
        for (Map.Entry<Character, Integer> entry : map.entrySet()) {
            if (entry.getValue() == 1) {
                System.out.println("First non-repeated char is: " + entry.getKey());
                break;
            }
        }
    }
}
```
**Verbal explanation:** "In an interview, I prefer this over nested for-loops. Using a LinkedHashMap guarantees O(N) time complexity, and it preserves the order of characters as they appear in the string. We just iterate once to populate counts, then iterate the map to find the first entry with a count of 1."
