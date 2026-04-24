# Java — Fundamentals to Expert for Backend and Test Automation

> Subject #6 of the Master Learning Plan.
> Owner: Govind Divekar. Prerequisites: any prior programming exposure helpful. Estimated time: ~75 hours.
> Target outcome: Write idiomatic modern Java (21 LTS), build a Spring Boot service, and write professional JUnit 5 + REST-assured test suites.

---

## 1. Overview & Learning Outcomes

This plan covers Java from core language features through professional enterprise testing frameworks. You'll master the Java platform's ecosystem for backend services—understanding both how to write production code and how to validate it rigorously. By the end, you'll be proficient in Spring Boot, JUnit 5, REST-assured, and the patterns that define Java testing at scale.

**Key outcomes:**
- Write clean, idiomatic modern Java (21 LTS) with proper use of records, sealed classes, and pattern matching.
- Build Spring Boot microservices with dependency injection, JPA, and validation.
- Test at all levels: unit (JUnit 5 + Mockito + AssertJ), integration (Testcontainers, MockMvc), and API (REST-assured).
- Understand concurrency primitives, virtual threads, and performance tuning for the JVM.
- Recognize and avoid common pitfalls: ConcurrentModificationException, equals/hashCode bugs, string interning, memory model issues.
- Implement CI/CD pipelines with GitHub Actions and Docker.

---

## 2. Detailed Syllabus (modules with hours)

| Module | Hours | Topics |
|--------|-------|--------|
| **Part 1: Language Fundamentals** | | |
| 1.1 Setup & Build Tools | 3 | Java 21 LTS installation, JAVA_HOME, Maven/Gradle, IntelliJ IDEA/VS Code |
| 1.2 Primitives & Types | 4 | Primitives, wrappers, autoboxing, String, text blocks, String.format |
| 1.3 Operators & Control Flow | 3 | Operators, conditionals, switch expressions, pattern matching |
| 1.4 Arrays & Loops | 3 | Arrays, enhanced for, Arrays utility, iteration patterns |
| 1.5 Methods & Scope | 3 | Overloading, varargs, static vs instance, method references |
| **Part 2: Object-Oriented Fundamentals** | | |
| 2.1 Classes & Visibility | 4 | Classes, fields, constructors, this/super, encapsulation |
| 2.2 Inheritance & Polymorphism | 4 | Inheritance, method override, abstract classes, final keyword |
| 2.3 Interfaces & Modern Java | 4 | Interfaces, default/static/private methods, records, sealed classes |
| 2.4 Generics & Collections | 6 | Generic types, bounded types, wildcards (PECS), collections API |
| 2.5 Comparable & Comparator | 2 | Ordering, natural vs custom comparisons |
| **Part 3: Functional & Concurrent Java** | | |
| 3.1 Lambdas & Functional Interfaces | 3 | Lambdas, Function/Predicate/Supplier/Consumer, SAM interfaces |
| 3.2 Streams API | 5 | Sources, intermediate ops, terminal ops, collectors, parallel streams |
| 3.3 Optional | 2 | Correct usage, avoiding anti-patterns, chaining |
| 3.4 Exceptions & IO | 4 | Checked vs unchecked, try-with-resources, java.nio, Files API |
| 3.5 Concurrency Basics | 4 | Thread, Runnable, synchronized, volatile, memory model |
| 3.6 java.util.concurrent | 4 | ExecutorService, Future, CompletableFuture, concurrent collections |
| 3.7 Virtual Threads & Structured Concurrency | 3 | JEP 444, Thread.ofVirtual, scoped values |
| **Part 4: JVM & Performance** | | |
| 4.1 JVM Internals | 3 | Class loading, JIT (C1/C2), GC (G1, ZGC overview), heap/metaspace |
| 4.2 Memory Model & Happens-Before | 2 | Visibility, ordering, data races (high level) |
| **Part 5: Testing & Frameworks** | | |
| 5.1 JUnit 5 | 3 | @Test, assertions, lifecycle, parameterized tests, nested/dynamic tests |
| 5.2 Mockito | 3 | Mocks, stubs, spies, argument captors, BDD style |
| 5.3 AssertJ & Testcontainers | 3 | Fluent assertions, soft assertions, spinning up real services |
| 5.4 REST-assured & Contract Testing | 3 | Sending/receiving, JSON Path, schema validation, Pact intro |
| **Part 6: Spring & Enterprise Testing** | | |
| 6.1 Spring Boot Fundamentals | 4 | Starters, autoconfiguration, @SpringBootApplication, embedded servers |
| 6.2 Spring Core & Dependency Injection | 3 | @Component, @Autowired, constructor injection, profiles, properties |
| 6.3 Spring Data JPA | 3 | Entities, repositories, derived queries, @Query, pagination, lazy loading |
| 6.4 Spring Validation & Web | 2 | jakarta.validation, @Valid, @RestController, content negotiation |
| 6.5 Spring Test & Integration Testing | 3 | @SpringBootTest, @WebMvcTest, @DataJpaTest, MockMvc, test slicing |
| 6.6 Spring Security Basics | 2 | Authentication, authorization, JWT pointers |
| **Part 7: DevOps & Production** | | |
| 7.1 Logging & Observability | 2 | SLF4J + Logback, structured logging, log levels |
| 7.2 Build Pipelines | 3 | GitHub Actions, Maven/Gradle caching, test reports, artifacts |
| 7.3 Packaging & Deployment | 2 | jar vs fat-jar, Spring Boot plugin, Docker image, compose |
| 7.4 Review & Micro-Interview Patterns | 3 | Common pitfalls, edge cases, architecture discussion |
| | **Total: ~75 hours** | |

---

## 3. Topics — Explanations with Examples

### Part 1: Language Fundamentals

#### 1.1 Setup & Build Tools

**Concept:** Java development requires the JDK (Java Development Kit), which includes the compiler and runtime. Modern Java evolves quickly; Java 21 LTS (Long Term Support) is the production standard. Build tools like Maven and Gradle manage dependencies and compilation.

**Installation (SDKMAN recommended):**
```bash
# macOS/Linux: install SDKMAN
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"

# Install Java 21 LTS
sdk install java 21.0.1-tem

# Verify
java -version
# openjdk version "21.0.1" 2023-10-17
```

**JAVA_HOME environment variable:**
```bash
# Verify (should point to Java 21 installation)
echo $JAVA_HOME
# Output: /home/user/.sdkman/candidates/java/21.0.1-tem

# Set permanently in ~/.bashrc or ~/.zshrc
export JAVA_HOME="$(sdk home java 21.0.1-tem)"
export PATH="$JAVA_HOME/bin:$PATH"
```

**Maven (pom.xml):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>my-app</artifactId>
  <version>1.0.0</version>

  <properties>
    <maven.compiler.source>21</maven.compiler.source>
    <maven.compiler.target>21</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit-jupiter</artifactId>
      <version>5.9.3</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.11.0</version>
        <configuration>
          <source>21</source>
          <target>21</target>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```

**Gradle (build.gradle.kts):**
```kotlin
plugins {
    java
    id("org.springframework.boot") version "3.2.0"
    id("io.spring.dependency-management") version "1.1.4"
}

java {
    sourceCompatibility = JavaVersion.VERSION_21
    targetCompatibility = JavaVersion.VERSION_21
}

repositories {
    mavenCentral()
}

dependencies {
    testImplementation("org.junit.jupiter:junit-jupiter:5.9.3")
    implementation("org.springframework.boot:spring-boot-starter-web")
}

tasks.test {
    useJUnitPlatform()
}
```

**IntelliJ IDEA setup:**
- Install Java 21 SDK (Preferences → Project Structure → SDKs → Add → select Java 21).
- Enable annotation processing (Preferences → Build → Compiler → Annotation Processors → Enable).
- Configure code inspections (Editor → Inspections, enable "Java → Probable bugs").

**Pitfalls:**
- Mixing JDK versions in classpath; always verify `java -version` before running tests.
- Not setting JAVA_HOME leads to subtle runtime version mismatches.
- Maven/Gradle cache corruption; clean with `mvn clean` or `./gradlew clean`.

---

#### 1.2 Primitives & Types

**Concept:** Java has 8 primitive types and wrapper classes. Modern Java provides autoboxing (automatic conversion), but understanding when and why it happens is critical for performance and correctness.

**Primitive types:**
```java
// 8 primitive types
byte b = 1;           // 8-bit signed integer
short s = 100;        // 16-bit signed integer
int i = 1000;         // 32-bit signed integer (default for numeric literals)
long l = 1000L;       // 64-bit signed integer (literal requires 'L' suffix)
float f = 3.14f;      // 32-bit floating point (literal requires 'f' suffix)
double d = 3.14;      // 64-bit floating point (default for decimal literals)
boolean flag = true;  // true or false
char c = 'A';         // 16-bit Unicode character
```

**Wrapper classes & autoboxing:**
```java
// Manual boxing/unboxing (pre-Java 5 style)
Integer wrapped = Integer.valueOf(42);
int unwrapped = wrapped.intValue();

// Autoboxing (automatic conversion)
Integer boxed = 42;                    // automatic Integer.valueOf(42)
int unboxed = boxed;                   // automatic boxed.intValue()
Integer sum = 10 + 20;                 // both int literals auto-boxed to Integer

// Useful wrapper utilities
int max = Integer.max(10, 20);         // 20
String binary = Integer.toBinaryString(15); // "1111"
int parsed = Integer.parseInt("42");   // 42
```

**String and text blocks:**
```java
// String concatenation
String name = "Alice";
String greeting = "Hello, " + name;

// String.format (like printf)
String formatted = String.format("User %s has %d messages", name, 42);
// "User Alice has 42 messages"

// Text blocks (Java 15+, for multiline strings)
String json = """
    {
      "name": "Alice",
      "age": 30
    }
    """;

// String methods
String text = "  Hello World  ";
text.trim();                           // "Hello World"
text.toLowerCase();                    // "  hello world  "
text.split(" ");                       // ["", "", "Hello", "World", "", ""]
text.contains("Hello");                // true
text.replaceAll("\\s+", "");           // "HelloWorld"
```

**Pitfalls:**
- Autoboxing on null causes NullPointerException: `Integer x = null; int y = x;` throws NPE.
- Integer cache: `Integer.valueOf(127) == Integer.valueOf(127)` is true (cached), but `Integer.valueOf(128) == Integer.valueOf(128)` is false. Always use `.equals()`.
- String immutability: `String s = "hello"; s.toUpperCase();` does not modify s; you must reassign: `s = s.toUpperCase();`

---

#### 1.3 Operators & Control Flow

**Concept:** Modern Java supports switch expressions (Java 12+) and pattern matching for switch (Java 21), which are more powerful than traditional if-else chains.

**Switch expressions (Java 12+):**
```java
// Traditional switch statement
String level = "WARN";
String message;
switch (level) {
    case "ERROR":
        message = "An error occurred";
        break;
    case "WARN":
        message = "Warning detected";
        break;
    default:
        message = "Unknown level";
}

// Switch expression (value-producing)
String message = switch (level) {
    case "ERROR" -> "An error occurred";
    case "WARN" -> "Warning detected";
    default -> "Unknown level";
};

// Multi-case arms
String classification = switch (code) {
    case 200, 201, 202, 204 -> "Success";
    case 301, 302 -> "Redirect";
    case 400, 401, 403, 404 -> "Client error";
    case 500, 502, 503 -> "Server error";
    default -> "Unknown";
};
```

**Pattern matching for switch (Java 21):**
```java
record Point(int x, int y) {}
record Circle(Point center, int radius) {}
record Rectangle(Point p1, Point p2) {}

Object shape = new Circle(new Point(0, 0), 5);

// Pattern matching eliminates casting and instanceof chains
String describe = switch (shape) {
    case Circle(Point(int x, int y), int r) when r > 10 ->
        "Large circle at (" + x + ", " + y + ")";
    case Circle(Point center, int r) ->
        "Circle with radius " + r;
    case Rectangle(Point p1, Point p2) ->
        "Rectangle from " + p1 + " to " + p2;
    case null ->
        "Null shape";
    default ->
        "Unknown shape";
};
```

**Ternary operator:**
```java
int age = 20;
String status = age >= 18 ? "adult" : "minor"; // "adult"
```

**Pitfalls:**
- Fall-through in traditional switch: forgetting `break;` causes unintended execution of next case.
- Pattern matching with `when` guards: guards are evaluated only if the pattern matches.

---

#### 1.4 Arrays & Loops

**Concept:** Arrays are fixed-size collections of primitives or objects. Enhanced for-loops simplify iteration; the Arrays utility class provides useful operations.

**Array basics:**
```java
// Declare and initialize
int[] nums = new int[5];                    // [0, 0, 0, 0, 0]
int[] nums2 = {10, 20, 30};                 // [10, 20, 30]
String[][] matrix = new String[2][3];      // 2D array

// Access and modify
nums[0] = 42;
int first = nums[0];                        // 42

// Length
nums.length;                                // 3
```

**Enhanced for-loop:**
```java
for (int num : nums) {
    System.out.println(num);
}

// With objects
String[] names = {"Alice", "Bob", "Charlie"};
for (String name : names) {
    System.out.println("Hello, " + name);
}
```

**Arrays utility:**
```java
int[] arr = {3, 1, 4, 1, 5, 9};

Arrays.sort(arr);                           // [1, 1, 3, 4, 5, 9]
Arrays.binarySearch(arr, 4);                // index 3
Arrays.fill(arr, 0);                        // [0, 0, 0, 0, 0, 0]
Arrays.equals(arr, arr);                    // true
String joined = Arrays.toString(arr);      // "[0, 0, 0, 0, 0, 0]"
```

**Pitfalls:**
- ArrayIndexOutOfBoundsException: always check array bounds.
- Arrays passed by reference: modifying an array in a method affects the original.
- `.equals()` on arrays compares reference, not contents; use `Arrays.equals(arr1, arr2)`.

---

#### 1.5 Methods & Scope

**Concept:** Methods encapsulate behavior. Overloading allows multiple methods with the same name but different signatures. Varargs simplify passing variable numbers of arguments.

**Method overloading:**
```java
class Calculator {
    // Overload: different parameter types
    public int add(int a, int b) {
        return a + b;
    }

    public double add(double a, double b) {
        return a + b;
    }

    // Overload: different number of parameters
    public int add(int a, int b, int c) {
        return a + b + c;
    }

    // Overload: different order
    public void process(String s, int i) {}
    public void process(int i, String s) {}
}

// Compiler resolves based on argument types at compile time
Calculator calc = new Calculator();
calc.add(1, 2);           // calls int version
calc.add(1.0, 2.0);       // calls double version
```

**Varargs (variable-length arguments):**
```java
public class Logger {
    // varargs parameter (must be last)
    public void log(String format, Object... args) {
        String message = String.format(format, args);
        System.out.println(message);
    }
}

Logger logger = new Logger();
logger.log("User %s logged in", "Alice");
logger.log("Processing %d items in %s", 42, "batch");
logger.log("Done");
```

**Static vs instance:**
```java
class Account {
    static int nextId = 1;               // shared across all instances
    private int id;                      // per-instance

    Account() {
        this.id = nextId++;              // use static field
    }

    // Static method (no access to instance fields)
    static Account create() {
        return new Account();
    }

    // Instance method
    int getId() {
        return id;
    }
}

Account a1 = Account.create();
Account a2 = Account.create();
a1.getId();                              // 1
a2.getId();                              // 2
Account.nextId;                          // 3
```

**Method references:**
```java
// Traditional lambda
List<String> names = List.of("alice", "bob", "charlie");
names.forEach(name -> System.out.println(name));

// Method reference (cleaner)
names.forEach(System.out::println);

// Reference to a static method
names.stream()
    .map(String::toUpperCase)             // reference to instance method
    .forEach(System.out::println);
```

**Pitfalls:**
- Confusing static and instance: static methods cannot access `this` or instance fields.
- Varargs and method resolution: `method(String... args)` and `method(Object... args)` are both varargs; the compiler picks the most specific.

---

### Part 2: Object-Oriented Fundamentals

#### 2.1 Classes & Visibility

**Concept:** Classes are blueprints. Visibility modifiers control access; encapsulation hides implementation details.

**Class structure:**
```java
public class BankAccount {
    // Fields (package-private, visible to same package)
    private String accountNumber;
    private double balance;
    private LocalDate createdAt;

    // Constructor
    public BankAccount(String accountNumber, double initialBalance) {
        this.accountNumber = accountNumber;
        this.balance = initialBalance;
        this.createdAt = LocalDate.now();
    }

    // Getter (read access)
    public double getBalance() {
        return balance;
    }

    // Setter (write access with validation)
    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }
        balance += amount;
    }

    public boolean withdraw(double amount) {
        if (amount <= 0 || amount > balance) {
            return false;
        }
        balance -= amount;
        return true;
    }
}
```

**Visibility modifiers:**
```
| Modifier    | Class | Package | Subclass | World |
|-------------|-------|---------|----------|-------|
| public      |  yes  |  yes    |   yes    |  yes  |
| protected   |  yes  |  yes    |   yes    |  no   |
| (package)   |  yes  |  yes    |   no     |  no   |
| private     |  yes  |  no     |   no     |  no   |
```

```java
class Example {
    public int pub = 1;          // visible everywhere
    protected int prot = 2;      // visible in subclasses and package
    int pkg = 3;                 // package-private, default
    private int priv = 4;        // visible only in this class
}
```

**Constructor overloading:**
```java
class Person {
    private String name;
    private int age;
    private String email;

    // Primary constructor
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
        this.email = "unknown@example.com";
    }

    // Overloaded constructor (calls primary via this())
    public Person(String name, int age, String email) {
        this(name, age);
        this.email = email;
    }
}
```

**Pitfalls:**
- Exposing mutable fields (even public) breaks encapsulation. Use getters/setters.
- Not initializing fields in constructor leads to NullPointerException.

---

#### 2.2 Inheritance & Polymorphism

**Concept:** Inheritance allows code reuse and establishes "is-a" relationships. Polymorphism lets you treat objects by their supertype.

**Inheritance and super:**
```java
// Parent class
class Animal {
    protected String name;

    public Animal(String name) {
        this.name = name;
    }

    public void speak() {
        System.out.println(name + " makes a sound");
    }
}

// Child class
class Dog extends Animal {
    private String breed;

    public Dog(String name, String breed) {
        super(name);                       // call parent constructor
        this.breed = breed;
    }

    @Override
    public void speak() {
        System.out.println(name + " barks");
    }

    public String getBreed() {
        return breed;
    }
}

// Usage
Animal dog = new Dog("Buddy", "Golden Retriever");
dog.speak();                               // "Buddy barks"
((Dog) dog).getBreed();                    // must cast to call Dog-specific method
```

**Abstract classes:**
```java
abstract class Shape {
    abstract double area();

    // Concrete method in abstract class
    public void printArea() {
        System.out.println("Area: " + area());
    }
}

class Circle extends Shape {
    private double radius;

    Circle(double radius) {
        this.radius = radius;
    }

    @Override
    double area() {
        return Math.PI * radius * radius;
    }
}

class Rectangle extends Shape {
    private double width, height;

    Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }

    @Override
    double area() {
        return width * height;
    }
}
```

**final keyword:**
```java
// Prevent subclassing
final class ImmutableData {
    // ...
}

class Parent {
    // Prevent overriding
    final void criticalOperation() {
        // ...
    }
}

// Immutable field (must be initialized)
class Config {
    final String apiKey;

    Config(String apiKey) {
        this.apiKey = apiKey;
    }
}
```

**Pitfalls:**
- Forgetting `@Override` annotation; typos in method signatures silently create overloads instead of overrides.
- Deep inheritance hierarchies are hard to maintain; prefer composition.

---

#### 2.3 Interfaces & Modern Java

**Concept:** Interfaces define contracts. Modern Java (8+) interfaces support default methods, static methods, and private methods, blurring the distinction with abstract classes.

**Basic interface:**
```java
interface PaymentProcessor {
    void process(double amount);
    boolean verify();                     // abstract by default
}

class CreditCardProcessor implements PaymentProcessor {
    @Override
    public void process(double amount) {
        System.out.println("Processing $" + amount);
    }

    @Override
    public boolean verify() {
        return true;
    }
}
```

**Default and static methods:**
```java
interface Logger {
    // Abstract method
    void log(String message);

    // Default method (concrete, can be overridden)
    default void info(String message) {
        log("[INFO] " + message);
    }

    // Static method (utility, not overrideable)
    static Logger create() {
        return message -> System.out.println(message);
    }
}

class ConsoleLogger implements Logger {
    @Override
    public void log(String message) {
        System.out.println(message);
    }

    // Can override default method, but not required
}

Logger logger = Logger.create();
logger.log("Hello");                       // "Hello"
logger.info("Info");                       // "[INFO] Info"
```

**Private methods (Java 9+):**
```java
interface DataValidator {
    boolean validate(String data);

    default String validateAndFormat(String data) {
        if (validate(data)) {
            return format(data);
        }
        return null;
    }

    // Private method shared by default methods
    private String format(String data) {
        return data.trim().toUpperCase();
    }
}
```

**Records (Java 16+):**
```java
// Immutable data carrier class
record User(String name, int age, String email) {
    // Automatically generates:
    // - constructor
    // - getters (name(), age(), email())
    // - equals(), hashCode(), toString()

    // Can add custom methods
    public boolean isAdult() {
        return age >= 18;
    }

    // Compact constructor for validation
    public User {
        if (age < 0) throw new IllegalArgumentException("Age cannot be negative");
    }
}

// Usage
User user = new User("Alice", 30, "alice@example.com");
user.name();                               // "Alice"
user.isAdult();                            // true
```

**Sealed classes (Java 17+):**
```java
// Restrict which classes can extend/implement
sealed interface Payment permits CreditCardPayment, PayPalPayment {
    void execute(double amount);
}

final class CreditCardPayment implements Payment {
    @Override
    public void execute(double amount) {
        System.out.println("Card payment: $" + amount);
    }
}

final class PayPalPayment implements Payment {
    @Override
    public void execute(double amount) {
        System.out.println("PayPal payment: $" + amount);
    }
}

// Trying to implement Payment elsewhere results in compile error
```

**Pitfalls:**
- Overriding default methods without clear intent can confuse readers.
- Sealed classes and records are still relatively new; IDE support is improving.

---

#### 2.4 Generics & Collections

**Concept:** Generics allow type-safe collections and methods. Understanding bounded types and wildcards prevents runtime errors and enables flexible APIs.

**Generic classes:**
```java
public class Container<T> {
    private T value;

    public void set(T value) {
        this.value = value;
    }

    public T get() {
        return value;
    }
}

// Usage
Container<String> strContainer = new Container<>();
strContainer.set("hello");
String str = strContainer.get();           // no cast needed

Container<Integer> intContainer = new Container<>();
intContainer.set(42);
int num = intContainer.get();              // int value, no cast
```

**Generic methods:**
```java
public class Util {
    // Generic method (can be called on non-generic class)
    public static <T> void print(T value) {
        System.out.println(value);
    }

    public static <T> T first(List<T> list) {
        return list.isEmpty() ? null : list.get(0);
    }
}

// Usage
Util.print("hello");                       // String
Util.print(42);                            // Integer
String first = Util.first(List.of("a", "b"));
```

**Bounded types:**
```java
// Upper bound: T must be Number or a subclass
public <T extends Number> double sum(T a, T b) {
    return a.doubleValue() + b.doubleValue();
}

// Multiple bounds
public <T extends Comparable<T> & Serializable> void sort(List<T> list) {
    // T must be Comparable and Serializable
}

sum(5, 10);                                // 15.0
sum(3.5, 2.5);                             // 6.0
```

**Wildcards:**
```java
// ? extends T (upper bound): read-only
public void processList(List<? extends Number> numbers) {
    for (Number num : numbers) {
        System.out.println(num.doubleValue());
    }
    // numbers.add(5);  // ERROR: cannot add (might be List<Double>, etc.)
}

// ? super T (lower bound): write-friendly
public void fillList(List<? super Integer> list) {
    list.add(1);
    list.add(2);
    // Object obj = list.get(0);  // retrieves Object, not Integer
}

// PECS: Producer Extends, Consumer Super
List<Integer> ints = new ArrayList<>();
processList(ints);                         // ? extends Number matches List<Integer>

List<Number> numbers = new ArrayList<>();
fillList(numbers);                         // ? super Integer matches List<Number>
```

**Collections Framework:**
```java
// List: ordered, allows duplicates
List<String> names = new ArrayList<>();
names.add("Alice");
names.add("Bob");
names.get(0);                              // "Alice"
names.remove(0);

LinkedList<String> linkedNames = new LinkedList<>();  // better for removals

// Set: unique elements, no order guarantee
Set<Integer> ids = new HashSet<>();
ids.add(1);
ids.add(2);
ids.add(1);                                // duplicate ignored
ids.size();                                // 2
ids.contains(1);                           // true

TreeSet<Integer> sortedIds = new TreeSet<>();  // sorted

// Map: key-value pairs
Map<String, Integer> ages = new HashMap<>();
ages.put("Alice", 30);
ages.put("Bob", 25);
ages.get("Alice");                         // 30
ages.getOrDefault("Charlie", 0);           // 0

TreeMap<String, Integer> sortedAges = new TreeMap<>();  // sorted by key
```

**Equals and hashCode contract:**
```java
public class User {
    private String email;
    private int age;

    public User(String email, int age) {
        this.email = email;
        this.age = age;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        User user = (User) obj;
        return age == user.age && Objects.equals(email, user.email);
    }

    @Override
    public int hashCode() {
        return Objects.hash(email, age);
    }
}

// Contract: if a.equals(b), then a.hashCode() == b.hashCode()
Set<User> users = new HashSet<>();
users.add(new User("alice@example.com", 30));
users.add(new User("alice@example.com", 30));  // recognized as duplicate
users.size();                              // 1
```

**Pitfalls:**
- Raw types (e.g., `List` instead of `List<String>`) bypass compile-time safety.
- Forgetting equals/hashCode or implementing them inconsistently causes silent bugs.
- Mutable keys in HashMap break the contract.

---

#### 2.5 Comparable & Comparator

**Concept:** Comparable provides natural ordering; Comparator allows custom orderings.

**Comparable (natural ordering):**
```java
class Person implements Comparable<Person> {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public int compareTo(Person other) {
        // Order by age (ascending)
        return Integer.compare(this.age, other.age);
    }

    @Override
    public String toString() {
        return name + " (" + age + ")";
    }
}

List<Person> people = new ArrayList<>();
people.add(new Person("Charlie", 35));
people.add(new Person("Alice", 30));
people.add(new Person("Bob", 32));

Collections.sort(people);                  // sorts by age
// [Alice (30), Bob (32), Charlie (35)]
```

**Comparator (custom ordering):**
```java
// Sort by name
Comparator<Person> byName = Comparator.comparing(p -> p.getName());
people.sort(byName);
// [Alice (30), Bob (32), Charlie (35)]

// Sort by age descending
Comparator<Person> byAgeDesc = Comparator.comparingInt(Person::getAge).reversed();
people.sort(byAgeDesc);
// [Charlie (35), Bob (32), Alice (30)]

// Chain comparators: sort by age, then by name
Comparator<Person> byAgeAndName = Comparator.comparingInt(Person::getAge)
    .thenComparing(Person::getName);
people.sort(byAgeAndName);
```

**Pitfalls:**
- Inconsistent Comparable/Comparator (e.g., equals disagrees with compareTo) causes issues in TreeSet/TreeMap.
- Mutable objects that change after insertion into a TreeSet become unsortable.

---

### Part 3: Functional & Concurrent Java

#### 3.1 Lambdas & Functional Interfaces

**Concept:** Lambdas are anonymous functions. Functional interfaces (single abstract method) enable concise code via lambda expressions.

**Lambda basics:**
```java
// Functional interface: single abstract method
@FunctionalInterface
interface Predicate<T> {
    boolean test(T t);
}

// Traditional anonymous class
Predicate<Integer> isPositive = new Predicate<Integer>() {
    @Override
    public boolean test(Integer value) {
        return value > 0;
    }
};

// Lambda expression (much shorter)
Predicate<Integer> isPositiveL = value -> value > 0;

// Multi-line lambda (requires braces and return)
Predicate<Integer> isEvenL = value -> {
    int result = value % 2;
    return result == 0;
};

isPositive.test(5);                        // true
isPositiveL.test(5);                       // true
```

**Common functional interfaces:**
```java
// Function<T, R>: T -> R
Function<Integer, String> toHex = n -> Integer.toHexString(n);
String hex = toHex.apply(255);             // "ff"

// Predicate<T>: T -> boolean
Predicate<String> isLong = s -> s.length() > 5;
isLong.test("hello");                      // false
isLong.test("hello world");                // true

// Supplier<T>: () -> T
Supplier<String> greeting = () -> "Hello";
String msg = greeting.get();               // "Hello"

// Consumer<T>: T -> void
Consumer<String> printer = System.out::println;
printer.accept("Hello");                   // prints "Hello"

// BiFunction<T, U, R>: (T, U) -> R
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
int result = add.apply(5, 3);              // 8
```

**Method references (syntactic sugar for lambdas):**
```java
// Static method reference
Function<String, Integer> length = String::length;
// equivalent to: s -> s.length()

// Instance method reference
Consumer<String> printer = System.out::println;
// equivalent to: msg -> System.out.println(msg)

// Constructor reference
Supplier<ArrayList> createList = ArrayList::new;
// equivalent to: () -> new ArrayList()

List<String> names = List.of("alice", "bob", "charlie");
names.stream()
    .map(String::toUpperCase)              // reference to instance method
    .forEach(System.out::println);
```

**Pitfalls:**
- Lambdas capture variables (must be effectively final or final).
- Long lambdas reduce readability; extract to named methods.

---

#### 3.2 Streams API

**Concept:** Streams provide functional transformations on collections. Think of pipelines: source -> intermediate operations -> terminal operation.

**Stream basics:**
```java
List<Integer> nums = List.of(1, 2, 3, 4, 5);

// Terminal operation: collect
List<Integer> doubled = nums.stream()
    .map(n -> n * 2)
    .collect(Collectors.toList());
// [2, 4, 6, 8, 10]

// Terminal operation: forEach
nums.stream()
    .filter(n -> n % 2 == 0)               // keep evens
    .forEach(System.out::println);
// 2, 4

// Terminal operation: reduce
int sum = nums.stream()
    .reduce(0, Integer::sum);              // combine into single value
// 15

int product = nums.stream()
    .reduce(1, (a, b) -> a * b);
// 120
```

**Intermediate operations:**
```java
List<String> words = List.of("apple", "banana", "apricot", "blueberry");

// filter: keep elements matching predicate
List<String> startsWithA = words.stream()
    .filter(w -> w.startsWith("a"))
    .collect(Collectors.toList());
// ["apple", "apricot"]

// map: transform each element
List<Integer> lengths = words.stream()
    .map(String::length)
    .collect(Collectors.toList());
// [5, 6, 7, 9]

// flatMap: flatten nested streams
List<List<Integer>> nested = List.of(
    List.of(1, 2),
    List.of(3, 4),
    List.of(5)
);
List<Integer> flattened = nested.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList());
// [1, 2, 3, 4, 5]

// distinct: remove duplicates
List<Integer> nums = List.of(1, 2, 2, 3, 3, 3);
List<Integer> unique = nums.stream()
    .distinct()
    .collect(Collectors.toList());
// [1, 2, 3]

// sorted: order elements
List<String> sorted = words.stream()
    .sorted(Comparator.reverseOrder())
    .collect(Collectors.toList());
// ["blueberry", "banana", "apricot", "apple"]
```

**Collectors (terminal operations):**
```java
List<String> items = List.of("apple", "banana", "apple", "cherry");

// toList, toSet
List<String> list = items.stream().collect(Collectors.toList());
Set<String> set = items.stream().collect(Collectors.toSet());

// toMap
Map<String, Integer> wordLengths = items.stream()
    .collect(Collectors.toMap(
        w -> w,                            // key: the word itself
        String::length                     // value: its length
    ));
// {"apple": 5, "banana": 6, "cherry": 6}

// groupingBy: partition into groups
Map<Integer, List<String>> byLength = items.stream()
    .collect(Collectors.groupingBy(String::length));
// {5: ["apple"], 6: ["banana", "cherry"]}

// joining: concatenate strings
String joined = items.stream()
    .collect(Collectors.joining(", "));
// "apple, banana, apple, cherry"

// counting: count elements
long count = items.stream()
    .collect(Collectors.counting());
// 4
```

**Parallel streams:**
```java
List<Integer> largeList = IntStream.rangeClosed(1, 1_000_000)
    .boxed()
    .collect(Collectors.toList());

// Sequential: slower on large lists
long sumSeq = largeList.stream()
    .map(n -> n * n)
    .collect(Collectors.summingLong(n -> n));

// Parallel: faster on large lists (uses fork/join pool)
long sumPar = largeList.parallelStream()
    .map(n -> n * n)
    .collect(Collectors.summingLong(n -> n));
```

**Pitfalls:**
- Modifying source collection while streaming causes ConcurrentModificationException.
- Parallel streams have overhead; only beneficial on large datasets.
- Terminal operations consume the stream; reuse requires creating a new stream.

---

#### 3.3 Optional

**Concept:** Optional<T> represents a value that may or may not be present. It's a container, not a replacement for null checking; misuse leads to anti-patterns.

**Correct usage:**
```java
Optional<String> name = Optional.of("Alice");
Optional<String> empty = Optional.empty();

// ifPresent: execute action if present
name.ifPresent(System.out::println);       // prints "Alice"
empty.ifPresent(System.out::println);      // does nothing

// ifPresentOrElse: action for present and absent
name.ifPresentOrElse(
    n -> System.out.println("Hello, " + n),
    () -> System.out.println("No name")
);

// get: returns value or throws
String value = name.get();                 // "Alice"
// String none = empty.get();  // throws NoSuchElementException

// orElse: default value
String nameOrDefault = empty.orElse("Unknown");  // "Unknown"

// orElseGet: lazy default (supplier)
String nameOrLazy = empty.orElseGet(() -> computeDefault());

// map: transform if present
Optional<Integer> length = name.map(String::length);  // Optional[5]
Optional<Integer> emptyLength = empty.map(String::length);  // Optional.empty

// flatMap: chain optional operations
Optional<User> user = findUser("alice");
Optional<String> email = user.flatMap(u -> findEmail(u));  // Optional<String>
```

**Anti-patterns to avoid:**
```java
// WRONG: Optional<String> null = null;  // defeats the purpose
// WRONG: if (opt.isPresent()) { use(opt.get()); }  // verbose, old style

// RIGHT: opt.ifPresent(this::use);

// WRONG: return opt.orElse(null);  // defeats Optional's purpose
// RIGHT: return opt.orElseThrow(() -> new CustomException());

// WRONG: opt.map(s -> s.toUpperCase()).get();  // can throw
// RIGHT: opt.map(String::toUpperCase).orElse("DEFAULT");
```

**Pitfalls:**
- Overusing Optional as a field or method return when nullability is rare.
- Chaining operations that might all return empty; use `ifPresentOrElse`.

---

#### 3.4 Exceptions & IO

**Concept:** Checked exceptions must be declared/caught; unchecked exceptions (RuntimeException) need not. Try-with-resources ensures proper cleanup.

**Exception hierarchy:**
```
Throwable
  ├── Error (JVM errors; don't catch)
  └── Exception
      ├── Checked (must handle): IOException, SQLException, FileNotFoundException
      └── RuntimeException (unchecked): NullPointerException, IllegalArgumentException, IndexOutOfBoundsException
```

**Checked vs unchecked:**
```java
// Checked exception: must declare in throws or catch
public void readFile(String path) throws IOException {
    FileReader reader = new FileReader(path);  // throws FileNotFoundException
    reader.read();                             // throws IOException
    reader.close();
}

// Unchecked exception: no declaration required
public int divide(int a, int b) {
    return a / b;  // throws ArithmeticException if b == 0 (unchecked)
}

// Catching and re-throwing
try {
    readFile("data.txt");
} catch (FileNotFoundException e) {
    System.err.println("File not found: " + e.getMessage());
} catch (IOException e) {
    System.err.println("IO error: " + e.getMessage());
    throw new RuntimeException("Failed to read file", e);  // wrap and rethrow
}
```

**Try-with-resources (Java 7+, auto-close):**
```java
// Traditional: manual close
FileReader reader = null;
try {
    reader = new FileReader("data.txt");
    int ch = reader.read();
    System.out.println((char) ch);
} finally {
    if (reader != null) reader.close();
}

// Try-with-resources: auto-close
try (FileReader reader = new FileReader("data.txt")) {
    int ch = reader.read();
    System.out.println((char) ch);
}  // reader.close() called automatically
```

**Java NIO (modern IO):**
```java
import java.nio.file.*;
import java.nio.charset.*;

// Read entire file
String content = Files.readString(Paths.get("data.txt"), StandardCharsets.UTF_8);

// Read lines
List<String> lines = Files.readAllLines(Paths.get("data.txt"));

// Write file
Files.writeString(Paths.get("output.txt"), "Hello, World!");

// Walk directory
Files.walk(Paths.get("src"))
    .filter(Files::isRegularFile)
    .filter(p -> p.toString().endsWith(".java"))
    .forEach(System.out::println);

// Copy, delete, exists
Files.copy(Paths.get("src.txt"), Paths.get("dst.txt"));
Files.delete(Paths.get("tmp.txt"));
Files.exists(Paths.get("data.txt"));
```

**Pitfalls:**
- Ignoring checked exceptions (don't swallow them in catch blocks).
- Not using try-with-resources for streams/readers.

---

#### 3.5 Concurrency Basics

**Concept:** Multiple threads execute simultaneously on multi-core systems. Synchronization prevents data races; volatile ensures visibility.

**Threads:**
```java
// Implement Runnable
class Worker implements Runnable {
    private String name;

    Worker(String name) {
        this.name = name;
    }

    @Override
    public void run() {
        for (int i = 0; i < 3; i++) {
            System.out.println(name + ": " + i);
            Thread.sleep(100);             // sleep
        }
    }
}

// Create and start threads
Thread t1 = new Thread(new Worker("Thread-1"));
Thread t2 = new Thread(new Worker("Thread-2"));

t1.start();                                // NOT t1.run()
t2.start();
```

**Synchronized (mutual exclusion):**
```java
public class Counter {
    private int count = 0;

    // Synchronized method: lock on this
    public synchronized void increment() {
        count++;
    }

    // Synchronized block: lock on specific object
    public void doSomething() {
        synchronized (this) {
            count++;
        }
    }

    public synchronized int getCount() {
        return count;
    }
}

Counter counter = new Counter();
for (int i = 0; i < 10; i++) {
    new Thread(counter::increment).start();
}
// count is safely incremented without race conditions
```

**Volatile (visibility):**
```java
class Flag {
    private volatile boolean running = true;  // changes are immediately visible

    public void stop() {
        running = false;
    }

    public void work() {
        while (running) {
            // check running frequently
        }
    }
}
```

**Memory model (simplified):**
- Atomicity: compound operations (e.g., count++) are NOT atomic; must synchronize.
- Visibility: changes in one thread are not automatically visible to others; use `volatile` or `synchronized`.
- Ordering: compiler/CPU reordering can break assumptions; establish happens-before relationships.

**Pitfalls:**
- Unprotected shared state causes data races and non-deterministic bugs.
- Over-synchronization hurts performance; minimize critical sections.
- volatile is NOT a substitute for synchronization; it only ensures visibility.

---

#### 3.6 java.util.concurrent

**Concept:** High-level concurrency utilities (ExecutorService, Future, CompletableFuture) simplify multi-threading without manual Thread creation.

**ExecutorService:**
```java
ExecutorService executor = Executors.newFixedThreadPool(4);  // 4 threads

// Submit tasks
Future<Integer> future1 = executor.submit(() -> {
    Thread.sleep(1000);
    return 42;
});

Future<Integer> future2 = executor.submit(() -> {
    Thread.sleep(500);
    return 100;
});

// Wait for results
try {
    int result1 = future1.get();           // blocks until done
    int result2 = future2.get(2, TimeUnit.SECONDS);  // timeout
    System.out.println(result1 + result2); // 142
} catch (ExecutionException e) {
    // Task threw exception
} catch (TimeoutException e) {
    // Timeout occurred
}

executor.shutdown();                       // no new tasks, wait for existing
// executor.shutdownNow();  // cancel remaining tasks
```

**CompletableFuture (modern, Java 8+):**
```java
// Create a completed future
CompletableFuture<Integer> cf = CompletableFuture.completedFuture(42);

// Create a future that completes asynchronously
CompletableFuture<Integer> async = CompletableFuture.supplyAsync(() -> {
    return expensiveComputation();
});

// Chain operations
async
    .thenApply(n -> n * 2)                 // transform
    .thenAccept(n -> System.out.println(n)) // consume result
    .thenRun(() -> System.out.println("Done"));

// Combine multiple futures
CompletableFuture<Integer> cf1 = CompletableFuture.supplyAsync(() -> 10);
CompletableFuture<Integer> cf2 = CompletableFuture.supplyAsync(() -> 20);

cf1.thenCombine(cf2, (a, b) -> a + b)
    .thenAccept(System.out::println);      // 30

// Handle exceptions
async
    .exceptionally(ex -> {
        System.err.println("Error: " + ex.getMessage());
        return 0;
    });

// Timeout (Java 9+)
async.orTimeout(5, TimeUnit.SECONDS);
```

**Concurrent collections:**
```java
// ConcurrentHashMap: thread-safe without locking entire map
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
map.put("a", 1);
map.putIfAbsent("b", 2);                   // atomic if-absent-then-put
map.compute("a", (k, v) -> v == null ? 1 : v + 1);

// CopyOnWriteArrayList: thread-safe, good for reads
List<String> list = new CopyOnWriteArrayList<>();
list.add("item1");
list.add("item2");
for (String item : list) {
    // safe even if other threads modify list
}

// BlockingQueue: thread-safe queue with blocking operations
BlockingQueue<Integer> queue = new LinkedBlockingQueue<>(10);
queue.put(42);                             // blocks if full
Integer value = queue.take();              // blocks if empty
```

**Pitfalls:**
- Not shutting down ExecutorService leads to resource leaks.
- Calling `.get()` on a Future without timeout can block forever.
- CompletableFuture exception handling requires `.exceptionally()` or `.handle()`.

---

#### 3.7 Virtual Threads & Structured Concurrency

**Concept:** Virtual threads (JEP 444, Java 21) are lightweight, enabling millions of concurrent tasks. Structured concurrency (preview) provides scope-based cancellation.

**Virtual threads:**
```java
// Traditional platform thread (heavy, ~1 MB stack)
Thread t = new Thread(() -> {
    System.out.println("Hello from platform thread");
});
t.start();

// Virtual thread (lightweight, on-demand stacks)
Thread vt = Thread.ofVirtual().start(() -> {
    System.out.println("Hello from virtual thread");
});

// ExecutorService with virtual threads
ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();
for (int i = 0; i < 1_000_000; i++) {
    executor.submit(() -> {
        // Each task runs on a separate virtual thread
        System.out.println("Task " + Thread.currentThread().getName());
    });
}
executor.shutdown();

// Benchmark: I/O-bound tasks
long start = System.nanoTime();
ExecutorService platformPool = Executors.newFixedThreadPool(100);
for (int i = 0; i < 10_000; i++) {
    platformPool.submit(this::ioTask);
}
platformPool.shutdown();
platformPool.awaitTermination(10, TimeUnit.SECONDS);
long platformTime = System.nanoTime() - start;

start = System.nanoTime();
ExecutorService virtualPool = Executors.newVirtualThreadPerTaskExecutor();
for (int i = 0; i < 10_000; i++) {
    virtualPool.submit(this::ioTask);
}
virtualPool.shutdown();
virtualPool.awaitTermination(10, TimeUnit.SECONDS);
long virtualTime = System.nanoTime() - start;

System.out.println("Platform threads: " + platformTime / 1_000_000 + "ms");
System.out.println("Virtual threads: " + virtualTime / 1_000_000 + "ms");
// Virtual threads typically 10-100x faster for I/O tasks
```

**Structured concurrency (preview in 21, finalized in later versions):**
```java
// Group related tasks and enforce scope-based cancellation
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Future<String> userData = scope.fork(() -> fetchUserData("alice"));
    Future<String> preferences = scope.fork(() -> fetchPreferences("alice"));
    
    scope.join();                          // wait for all tasks
    
    System.out.println("User: " + userData.resultNow());
    System.out.println("Prefs: " + preferences.resultNow());
} catch (ExecutionException | InterruptedException e) {
    System.err.println("Task failed: " + e.getMessage());
}
// All tasks cancelled if any fails
```

**Pitfalls:**
- Virtual threads still require proper exception handling (not a silver bullet).
- Structured concurrency is preview; expect API changes.

---

### Part 4: JVM & Performance

#### 4.1 JVM Internals

**Concept:** The JVM translates bytecode to machine instructions. Understanding class loading, JIT compilation, and garbage collection helps diagnose performance issues.

**Class loading:**
```
Bootstrap ClassLoader (JDK classes)
    ↓
Extension ClassLoader (ext/)
    ↓
Application ClassLoader (classpath)
```

**JIT compilation (Just-In-Time):**
- Java bytecode is initially interpreted (slow).
- JVM profiles execution and recompiles frequently-executed code to native instructions (fast).
- Two JIT compilers: C1 (quick, warm-up) and C2 (aggressive, latency).

**GC (Garbage Collection):**
```java
// Generational GC: young generation (GC often) -> old generation (GC rarely)
// Heap regions: Eden (new objects) -> Survivor S0/S1 -> Old generation

Object obj = new Object();
obj = null;                                // eligible for GC

// G1GC (default in Java 11+): divides heap into regions
// ZGC: ultra-low latency, concurrent, not stop-the-world

// Monitor GC
// JVM flags: -XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xmx4G -Xms4G
```

**Heap vs Metaspace:**
```
Heap: where objects live (resizes dynamically, GC managed)
Metaspace: where class metadata lives (Java 8+, replaces PermGen, not fixed-size)
```

**Common tuning flags:**
```bash
# Heap size
java -Xms2G -Xmx4G MyApp  # min 2GB, max 4GB

# GC selection
java -XX:+UseG1GC MyApp
java -XX:+UseZGC MyApp

# GC logging
java -Xlog:gc*:file=gc.log MyApp
```

**Pitfalls:**
- Setting -Xmx too high causes long GC pauses.
- Not tuning for your workload (latency-sensitive vs throughput-focused).

---

#### 4.2 Memory Model & Happens-Before

**Concept:** Java Memory Model defines when changes become visible across threads.

**Happens-before rules (simplified):**
1. Thread start: `thread.start()` happens-before actions in the thread.
2. Volatile: write to volatile happens-before subsequent read.
3. Synchronized: release happens-before next acquire.
4. Final: initialization happens-before other threads see the object.

```java
class Example {
    private volatile int flag = 0;
    private int data = 0;

    public void writer() {
        data = 42;                         // write
        flag = 1;                          // volatile write (happens-after)
    }

    public void reader() {
        if (flag == 1) {                   // volatile read
            System.out.println(data);      // guaranteed to see 42
        }
    }
}
```

**Pitfalls:**
- Race conditions happen silently and are hard to debug.
- Hoping synchronization isn't necessary (it usually is).

---

### Part 5: Testing & Frameworks

#### 5.1 JUnit 5

**Concept:** JUnit 5 (Jupiter) is the modern testing framework. Parameterized tests reduce boilerplate; nested tests organize complex scenarios.

**Basic test:**
```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class CalculatorTest {
    private Calculator calc = new Calculator();

    @Test
    void shouldAddTwoNumbers() {
        assertEquals(5, calc.add(2, 3));
    }

    @Test
    void shouldHandleNegatives() {
        assertEquals(-1, calc.add(-5, 4));
    }
}
```

**Lifecycle:**
```java
class LifecycleTest {
    @BeforeAll
    static void setUpAll() {
        System.out.println("Before all tests");
    }

    @BeforeEach
    void setUp() {
        System.out.println("Before each test");
    }

    @Test
    void test1() {}

    @Test
    void test2() {}

    @AfterEach
    void tearDown() {
        System.out.println("After each test");
    }

    @AfterAll
    static void tearDownAll() {
        System.out.println("After all tests");
    }
}
```

**Parameterized tests:**
```java
class MathTest {
    @ParameterizedTest
    @ValueSource(ints = {1, 2, 3, 4, 5})
    void testEven(int num) {
        assertTrue(num % 2 == 0 || num % 2 == 1);
    }

    @ParameterizedTest
    @CsvSource({
        "1, 1, 2",
        "2, 3, 5",
        "3, 4, 7"
    })
    void testAdd(int a, int b, int expected) {
        assertEquals(expected, a + b);
    }

    @ParameterizedTest
    @MethodSource("provideStrings")
    void testString(String input, int expected) {
        assertEquals(expected, input.length());
    }

    static Stream<Arguments> provideStrings() {
        return Stream.of(
            Arguments.of("hello", 5),
            Arguments.of("world", 5)
        );
    }
}
```

**Nested tests:**
```java
class BankAccountTest {
    class Deposit {
        @Test
        void shouldIncreaseBalance() {
            // test deposit
        }
    }

    class Withdraw {
        @Test
        void shouldDecreaseBalance() {
            // test withdrawal
        }

        @Test
        void shouldRejectNegativeAmount() {
            // test invalid withdrawal
        }
    }
}
```

**Assertions:**
```java
// Basic assertions
assertEquals(expected, actual);
assertNotEquals(unexpected, actual);
assertTrue(condition);
assertFalse(!condition);
assertNull(obj);
assertNotNull(obj);

// Grouped assertions
assertAll(
    () -> assertEquals(1, 1),
    () -> assertTrue(true),
    () -> assertNotNull("not null")
);

// Exception assertions
assertThrows(IllegalArgumentException.class, () -> {
    calc.divide(10, 0);
});
```

**Pitfalls:**
- Not naming tests clearly; use `shouldDoXWhenYCondition`.
- Over-testing trivial getters.

---

#### 5.2 Mockito

**Concept:** Mocks replace real dependencies; stubs return fixed values. Verification checks that methods were called.

**Basic mocking:**
```java
import static org.mockito.Mockito.*;

class PaymentServiceTest {
    private PaymentProcessor processor = mock(PaymentProcessor.class);
    private PaymentService service = new PaymentService(processor);

    @Test
    void shouldProcessPayment() {
        // Stub: specify return value
        when(processor.charge(100.0))
            .thenReturn(true);

        // Act
        boolean result = service.processPayment(100.0);

        // Verify
        assertTrue(result);
        verify(processor).charge(100.0);    // method called once
    }

    @Test
    void shouldHandleFailure() {
        when(processor.charge(anyDouble()))
            .thenThrow(new ProcessingException("Error"));

        assertThrows(ProcessingException.class, () -> {
            service.processPayment(100.0);
        });
    }
}
```

**Argument captors:**
```java
@Test
void shouldCaptureArguments() {
    ArgumentCaptor<String> captor = ArgumentCaptor.forClass(String.class);
    
    service.doSomething();
    
    verify(logger).log(captor.capture());
    String logged = captor.getValue();
    assertEquals("expected message", logged);
}
```

**BDD style (Given-When-Then):**
```java
import static org.mockito.BDDMockito.*;

class OrderServiceTest {
    @Test
    void shouldReturnOrderDetails() {
        // Given
        OrderRepository repo = mock(OrderRepository.class);
        Order order = new Order("123", 50.0);
        given(repo.findById("123")).willReturn(Optional.of(order));
        OrderService service = new OrderService(repo);

        // When
        Order result = service.getOrder("123").orElse(null);

        // Then
        assertNotNull(result);
        assertEquals(50.0, result.getAmount());
        then(repo).should().findById("123");
    }
}
```

**Pitfalls:**
- Over-mocking (mock everything, test nothing real).
- Mocking internal implementation details; mock boundaries (interfaces).

---

#### 5.3 AssertJ & Testcontainers

**Concept:** AssertJ provides fluent, readable assertions. Testcontainers spins up real services (Postgres, Redis, etc.) for integration tests.

**AssertJ:**
```java
import static org.assertj.core.api.Assertions.*;

@Test
void assertJExample() {
    List<String> names = List.of("Alice", "Bob", "Charlie");

    // Fluent assertions
    assertThat(names)
        .isNotEmpty()
        .hasSize(3)
        .contains("Alice", "Bob")
        .doesNotContain("David")
        .startsWith("Alice")
        .endsWith("Charlie");

    // Soft assertions (don't stop on first failure)
    SoftAssertions soft = new SoftAssertions();
    soft.assertThat(names).hasSize(2);    // fails but continues
    soft.assertThat(names).contains("David"); // fails but continues
    soft.assertAll();                      // report all failures
}
```

**Testcontainers:**
```java
import org.testcontainers.containers.PostgreSQLContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

@Testcontainers
class UserRepositoryTest {
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test");

    @Test
    void shouldSaveAndRetrieveUser() {
        // Create connection to container
        String url = postgres.getJdbcUrl();
        String username = postgres.getUsername();
        String password = postgres.getPassword();

        // Create schema and insert data
        Connection conn = DriverManager.getConnection(url, username, password);
        conn.createStatement().execute("CREATE TABLE users (id SERIAL, name VARCHAR)");
        conn.createStatement().execute("INSERT INTO users (name) VALUES ('Alice')");

        // Test queries against real database
        ResultSet rs = conn.createStatement().executeQuery("SELECT * FROM users");
        assertThat(rs.next()).isTrue();
        assertThat(rs.getString("name")).isEqualTo("Alice");
    }
}
```

**Pitfalls:**
- Testcontainers slow down tests; use sparingly and in separate integration test suite.
- Not cleaning up containers (memory leaks).

---

#### 5.4 REST-assured & Contract Testing

**Concept:** REST-assured makes it easy to test HTTP APIs. Pact ensures consumer-driven contracts.

**REST-assured:**
```java
import static io.restassured.RestAssured.*;
import static io.restassured.matcher.RestAssuredMatchers.*;

class UserApiTest {
    @Test
    void shouldGetUserById() {
        given()
            .baseUri("https://api.example.com")
            .basePath("/users")
            .pathParam("id", 123)
        .when()
            .get("/{id}")
        .then()
            .statusCode(200)
            .body("id", equalTo(123))
            .body("name", notNullValue())
            .body("email", matchesPattern(".+@.+"));
    }

    @Test
    void shouldPostNewUser() {
        given()
            .baseUri("https://api.example.com")
            .basePath("/users")
            .contentType("application/json")
            .body("""
                {
                  "name": "Alice",
                  "email": "alice@example.com"
                }
                """)
        .when()
            .post()
        .then()
            .statusCode(201)
            .header("Location", notNullValue());
    }

    @Test
    void shouldExtractValues() {
        String email = given()
            .baseUri("https://api.example.com")
            .pathParam("id", 123)
        .when()
            .get("/users/{id}")
        .then()
            .extract()
            .path("email");

        assertThat(email).contains("@");
    }
}
```

**JSON Path assertions:**
```java
@Test
void jsonPathExample() {
    given()
        .baseUri("https://api.example.com")
    .when()
        .get("/posts")
    .then()
        .body("$", hasSize(greaterThan(0)))  // root is array with size > 0
        .body("$.id", everyItem(notNullValue()))  // every id is not null
        .body("findAll { it.published }.title", hasItems("Published Post"));
}
```

**Pact (Consumer-Driven Contracts):**
```java
import au.com.dius.pact.consumer.dsl.PactBuilder;
import au.com.dius.pact.core.model.RequestResponsePact;

class UserServicePactTest {
    @Test
    void testUserEndpointContract() throws Exception {
        RequestResponsePact pact = new PactBuilder()
            .given("a user with ID 123 exists")
            .uponReceiving("a request for user 123")
            .path("/users/123")
            .method("GET")
            .willRespondWith()
            .status(200)
            .body(new PactDslJsonBody()
                .integerType("id", 123)
                .stringType("name", "Alice")
                .stringType("email", "alice@example.com"))
            .toPact();

        // Test against mock server matching the contract
        // Pact ensures provider (API) and consumer (client) agree on format
    }
}
```

**Pitfalls:**
- REST-assured verbosity; extract common setup to base class.
- Brittle tests that check implementation details instead of behavior.

---

### Part 6: Spring & Enterprise Testing

#### 6.1 Spring Boot Fundamentals

**Concept:** Spring Boot provides autoconfiguration and embedded servers, eliminating boilerplate. Starters bundle common dependencies.

**Spring Boot application:**
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication  // combines @Configuration, @ComponentScan, @EnableAutoConfiguration
public class LibraryApplication {
    public static void main(String[] args) {
        SpringApplication.run(LibraryApplication.class, args);
    }
}
```

**REST controller:**
```java
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/books")
public class BookController {
    private final BookService service;

    public BookController(BookService service) {  // constructor injection
        this.service = service;
    }

    @GetMapping("/{id}")
    public ResponseEntity<BookDto> getBook(@PathVariable Long id) {
        return service.findById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }

    @PostMapping
    public ResponseEntity<BookDto> createBook(@RequestBody CreateBookRequest req) {
        BookDto book = service.create(req);
        return ResponseEntity.created(URI.create("/books/" + book.id()))
            .body(book);
    }
}
```

**POM for Spring Boot (Maven):**
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.0</version>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <version>42.7.1</version>
    </dependency>
</dependencies>
```

**application.properties:**
```properties
spring.application.name=library-api
server.port=8080

spring.datasource.url=jdbc:postgresql://localhost:5432/library
spring.datasource.username=user
spring.datasource.password=password
spring.jpa.hibernate.ddl-auto=validate

spring.jpa.show-sql=false
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQL95Dialect

logging.level.root=INFO
logging.level.com.example=DEBUG
```

**Pitfalls:**
- Over-relying on autoconfiguration; understand what it does.
- Not using profiles for different environments (dev, test, prod).

---

#### 6.2 Spring Core & Dependency Injection

**Concept:** Dependency injection decouples classes. Spring manages object creation and wiring.

**@Component and @Autowired:**
```java
interface BookRepository {
    Optional<Book> findById(Long id);
}

@Repository
class PostgresBookRepository implements BookRepository {
    @Override
    public Optional<Book> findById(Long id) {
        // query database
        return Optional.of(new Book(id, "Title", "Author"));
    }
}

@Service
class BookService {
    private final BookRepository repository;

    public BookService(BookRepository repository) {  // constructor injection (preferred)
        this.repository = repository;
    }

    public Optional<Book> getBook(Long id) {
        return repository.findById(id);
    }
}

@RestController
@RequestMapping("/books")
class BookController {
    private final BookService service;

    public BookController(BookService service) {
        this.service = service;
    }

    @GetMapping("/{id}")
    public ResponseEntity<Book> get(@PathVariable Long id) {
        return service.getBook(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
}
```

**Profiles and properties:**
```java
@Configuration
@Profile("production")
class ProductionConfig {
    @Bean
    public DataSource dataSource() {
        return new ProductionDataSource();
    }
}

@Configuration
@Profile("test")
class TestConfig {
    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .build();
    }
}

// In application.properties or application-prod.properties
spring.profiles.active=production
```

**Pitfalls:**
- Circular dependencies (A depends on B, B depends on A); refactor to remove cycle.
- @Autowired on fields (non-final, mutable); use constructor injection instead.

---

#### 6.3 Spring Data JPA

**Concept:** JPA simplifies database access. Spring Data generates repository implementations automatically.

**Entity and repository:**
```java
import jakarta.persistence.*;

@Entity
@Table(name = "books")
public class Book {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String title;

    @Column(nullable = false)
    private String author;

    @Column(name = "published_year")
    private Integer publishedYear;

    // Getters, setters, constructors omitted for brevity
}

@Repository
interface BookRepository extends JpaRepository<Book, Long> {
    // Derived query methods
    List<Book> findByAuthor(String author);
    Optional<Book> findByTitle(String title);
    List<Book> findByAuthorAndPublishedYearGreaterThan(String author, Integer year);
    
    // Custom query
    @Query("SELECT b FROM Book b WHERE b.publishedYear > :year")
    List<Book> publishedAfter(@Param("year") Integer year);
    
    // Pagination
    Page<Book> findByAuthor(String author, Pageable pageable);
}

@Service
class BookService {
    private final BookRepository repository;

    public BookService(BookRepository repository) {
        this.repository = repository;
    }

    public List<Book> getBooksByAuthor(String author) {
        return repository.findByAuthor(author);
    }

    public Page<Book> listBooks(Pageable pageable) {
        return repository.findAll(pageable);
    }
}
```

**Usage in controller:**
```java
@RestController
@RequestMapping("/books")
class BookController {
    private final BookService service;

    public BookController(BookService service) {
        this.service = service;
    }

    @GetMapping
    public Page<BookDto> listBooks(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size) {
        Pageable pageable = PageRequest.of(page, size, Sort.by("title").ascending());
        return service.listBooks(pageable)
            .map(this::toDto);
    }

    @GetMapping("/author/{author}")
    public List<BookDto> getByAuthor(@PathVariable String author) {
        return service.getBooksByAuthor(author)
            .stream()
            .map(this::toDto)
            .toList();
    }

    private BookDto toDto(Book book) {
        return new BookDto(book.getId(), book.getTitle(), book.getAuthor());
    }
}
```

**Pitfalls:**
- N+1 query problem: use `@EntityGraph` or `fetch = FetchType.EAGER` to eager-load relationships.
- Lazy loading outside transaction throws LazyInitializationException.

---

#### 6.4 Spring Validation & Web

**Concept:** Validation annotations (jakarta.validation) ensure data integrity. Spring WebMvc handles content negotiation.

**Validation:**
```java
import jakarta.validation.constraints.*;

record CreateBookRequest(
    @NotBlank(message = "Title is required")
    @Size(min = 1, max = 255)
    String title,

    @NotBlank(message = "Author is required")
    String author,

    @Positive(message = "Year must be positive")
    @Max(value = 2025, message = "Year cannot be in the future")
    Integer publishedYear
) {}

@RestController
@RequestMapping("/books")
class BookController {
    private final BookService service;

    public BookController(BookService service) {
        this.service = service;
    }

    @PostMapping
    public ResponseEntity<BookDto> create(@Valid @RequestBody CreateBookRequest req) {
        BookDto book = service.create(req);
        return ResponseEntity.created(URI.create("/books/" + book.id()))
            .body(book);
    }
}

// Global exception handler
@ControllerAdvice
class GlobalExceptionHandler {
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        List<String> errors = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .map(FieldError::getDefaultMessage)
            .toList();
        return ResponseEntity.badRequest()
            .body(new ErrorResponse("Validation failed", errors));
    }
}
```

**Content negotiation (JSON/XML):**
```java
@RestController
@RequestMapping("/books")
class BookController {
    @GetMapping(produces = {MediaType.APPLICATION_JSON_VALUE, MediaType.APPLICATION_XML_VALUE})
    public List<BookDto> list() {
        // Spring automatically serializes to JSON or XML based on Accept header
        return List.of(new BookDto(1L, "Title", "Author"));
    }
}

// application.properties
spring.mvc.contentnegotiation.favor-parameter=false
```

**Pitfalls:**
- Validation annotations without `@Valid` are silently ignored.
- Not handling validation errors; results in 400 Bad Request without details.

---

#### 6.5 Spring Test & Integration Testing

**Concept:** Spring Test annotations (@SpringBootTest, @WebMvcTest, etc.) simplify testing Spring applications.

**Integration test (@SpringBootTest):**
```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class BookControllerIntegrationTest {
    @Autowired
    TestRestTemplate restTemplate;

    @Autowired
    BookRepository repository;

    @BeforeEach
    void setUp() {
        repository.deleteAll();
        repository.save(new Book("The Great Gatsby", "F. Scott Fitzgerald", 1925));
    }

    @Test
    void shouldListBooks() {
        ResponseEntity<String> response = restTemplate.getForEntity(
            "/books", String.class
        );
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(response.getBody()).contains("The Great Gatsby");
    }
}
```

**Unit test with MockMvc:**
```java
@WebMvcTest(BookController.class)
class BookControllerTest {
    @Autowired
    MockMvc mockMvc;

    @MockBean
    BookService service;

    @Test
    void shouldGetBookById() throws Exception {
        Book book = new Book("Title", "Author", 2000);
        when(service.getBook(1L)).thenReturn(Optional.of(book));

        mockMvc.perform(get("/books/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.title").value("Title"))
            .andExpect(jsonPath("$.author").value("Author"));

        verify(service).getBook(1L);
    }

    @Test
    void shouldReturn404WhenNotFound() throws Exception {
        when(service.getBook(999L)).thenReturn(Optional.empty());

        mockMvc.perform(get("/books/999"))
            .andExpect(status().isNotFound());
    }
}
```

**Slice testing (@DataJpaTest):**
```java
@DataJpaTest
class BookRepositoryTest {
    @Autowired
    BookRepository repository;

    @Autowired
    TestEntityManager entityManager;

    @Test
    void shouldFindByAuthor() {
        Book book = new Book("Title", "Author", 2000);
        entityManager.persistAndFlush(book);

        List<Book> found = repository.findByAuthor("Author");

        assertThat(found).hasSize(1)
            .extracting(Book::getTitle)
            .contains("Title");
    }
}
```

**Testcontainers with Spring:**
```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
@Testcontainers
class BookRepositoryContainerTest {
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
        .withDatabaseName("testdb")
        .withUsername("user")
        .withPassword("password");

    @DynamicPropertySource
    static void postgresProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    BookRepository repository;

    @Test
    void shouldSaveAndRetrieve() {
        Book book = new Book("Title", "Author", 2000);
        repository.save(book);

        Optional<Book> found = repository.findByTitle("Title");
        assertThat(found).isPresent()
            .get()
            .extracting(Book::getAuthor)
            .isEqualTo("Author");
    }
}
```

**Pitfalls:**
- Using @SpringBootTest for unit tests (slow); use @WebMvcTest or @DataJpaTest instead.
- Not cleaning up test data between tests (state leakage).

---

#### 6.6 Spring Security Basics

**Concept:** Spring Security handles authentication (who are you) and authorization (what can you do).

**Basic authentication and authorization:**
```java
@Configuration
@EnableWebSecurity
class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/public/**").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .formLogin(login -> login
                .loginPage("/login")
                .defaultSuccessUrl("/home")
            )
            .logout(logout -> logout
                .logoutUrl("/logout")
                .logoutSuccessUrl("/login")
            );
        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}

@RestController
@RequestMapping("/books")
class BookController {
    @GetMapping
    @PreAuthorize("hasRole('USER')")
    public List<BookDto> list() {
        return List.of();
    }

    @PostMapping
    @PreAuthorize("hasRole('ADMIN')")
    public BookDto create(@Valid @RequestBody CreateBookRequest req) {
        return new BookDto(1L, req.title(), req.author());
    }
}
```

**JWT (JSON Web Token) basics (pointers):**
```java
// Use libraries: jjwt or spring-security-oauth2-jose
// JWT structure: header.payload.signature
// Token stored in Authorization header: "Bearer <token>"

// On login, generate token
String token = Jwts.builder()
    .subject("user@example.com")
    .issuedAt(new Date())
    .expiration(new Date(System.currentTimeMillis() + 3600000))
    .signWith(SignatureAlgorithm.HS256, "secret-key")
    .compact();

// Filter extracts and validates token from request
// Spring Security sets SecurityContext with authenticated principal
```

**Pitfalls:**
- Storing passwords in plain text (always hash with bcrypt).
- JWT secret exposed in code (use environment variables).

---

### Part 7: DevOps & Production

#### 7.1 Logging & Observability

**Concept:** SLF4J + Logback provide structured logging. Proper levels (DEBUG, INFO, WARN, ERROR) aid troubleshooting.

**SLF4J + Logback:**
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Service
class BookService {
    private static final Logger logger = LoggerFactory.getLogger(BookService.class);

    public Optional<Book> getBook(Long id) {
        logger.debug("Fetching book with id: {}", id);
        // query
        if (found) {
            logger.info("Book found: {}", book.getTitle());
            return Optional.of(book);
        }
        logger.warn("Book not found for id: {}", id);
        return Optional.empty();
    }

    public void deleteBook(Long id) {
        try {
            // delete logic
        } catch (Exception e) {
            logger.error("Failed to delete book with id: {}", id, e);
            throw new RuntimeException("Deletion failed", e);
        }
    }
}
```

**logback.xml (configuration):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/app.log</file>
        <encoder>
            <pattern>%d{ISO8601} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
    </root>

    <logger name="com.example" level="DEBUG"/>
</configuration>
```

**Structured logging (JSON):**
```java
// Use logstash-logback-encoder for JSON output
logger.info("User logged in", "userId", "123", "timestamp", System.currentTimeMillis());
// Output: {"@timestamp":"...", "message":"User logged in", "userId":"123", ...}
```

**Pitfalls:**
- Logging at wrong level (debug in loops, info for every transaction).
- Not sanitizing sensitive data (passwords, tokens) in logs.

---

#### 7.2 Build Pipelines

**Concept:** GitHub Actions automate testing and deployment. Caching Maven/Gradle artifacts speeds up builds.

**GitHub Actions workflow (.github/workflows/test.yml):**
```yaml
name: Java Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        java-version: [21, 22]

    steps:
      - uses: actions/checkout@v4

      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          java-version: ${{ matrix.java-version }}
          distribution: temurin
          cache: maven

      - name: Run tests
        run: mvn clean test

      - name: Generate coverage report
        run: mvn jacoco:report

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          files: ./target/site/jacoco/jacoco.xml

      - name: Build jar
        run: mvn clean package -DskipTests

      - name: Upload artifact
        uses: actions/upload-artifact@v3
        with:
          name: library-api.jar
          path: target/library-api-*.jar
```

**Maven pom.xml (test and coverage):**
```xml
<plugins>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>3.0.0</version>
        <configuration>
            <includes>
                <include>**/*Test.java</include>
                <include>**/*Tests.java</include>
            </includes>
        </configuration>
    </plugin>

    <plugin>
        <groupId>org.jacoco</groupId>
        <artifactId>jacoco-maven-plugin</artifactId>
        <version>0.8.10</version>
        <executions>
            <execution>
                <goals>
                    <goal>prepare-agent</goal>
                </goals>
            </execution>
            <execution>
                <id>report</id>
                <phase>test</phase>
                <goals>
                    <goal>report</goal>
                </goals>
            </execution>
        </executions>
    </plugin>
</plugins>
```

**Pitfalls:**
- Not caching dependencies (slow CI).
- Running integration tests in CI without containers (flaky, slow).

---

#### 7.3 Packaging & Deployment

**Concept:** Spring Boot produces fat JAR files (all dependencies included). Docker images containerize applications.

**Spring Boot Maven plugin (fat JAR):**
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <mainClass>com.example.LibraryApplication</mainClass>
            </configuration>
        </plugin>
    </plugins>
</build>

<!-- Build: mvn clean package
     Run: java -jar target/library-api-1.0.0.jar
-->
```

**Docker (Dockerfile):**
```dockerfile
FROM eclipse-temurin:21-jdk-alpine
WORKDIR /app

COPY target/library-api-1.0.0.jar app.jar
EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Docker Compose (local development):**
```yaml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/library
      SPRING_DATASOURCE_USERNAME: user
      SPRING_DATASOURCE_PASSWORD: password
    depends_on:
      - postgres

  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: library
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

**Pitfalls:**
- Not setting resource limits in Docker (OOMKilled).
- Hardcoding secrets in Dockerfile or environment (use Secrets management).

---

#### 7.4 Review & Micro-Interview Patterns

**Concept:** Understand common pitfalls and how to discuss them in interviews.

**ConcurrentModificationException:**
```java
// WRONG: modifying list while iterating
List<String> items = new ArrayList<>(List.of("a", "b", "c"));
for (String item : items) {
    if (item.equals("b")) {
        items.remove(item);  // ConcurrentModificationException!
    }
}

// RIGHT: use iterator or removeIf
List<String> items = new ArrayList<>(List.of("a", "b", "c"));
items.removeIf(item -> item.equals("b"));
```

**Equals and hashCode bugs:**
```java
// WRONG: inconsistent equals and hashCode
class User {
    String email;
    int age;

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (!(obj instanceof User u)) return false;
        return Objects.equals(email, u.email) && age == u.age;
    }

    @Override
    public int hashCode() {
        return Objects.hash(email);  // hashCode ignores age!
    }
}
// Two users with same email but different ages have same hash but aren't equal.
// Breaks TreeMap, HashSet contract.

// RIGHT
@Override
public int hashCode() {
    return Objects.hash(email, age);
}
```

**String interning:**
```java
String a = "hello";
String b = "hello";
String c = new String("hello");

a == b;           // true (interned, same reference)
a.equals(b);      // true (content comparison)
a == c;           // false (different objects)
a.equals(c);      // true (content comparison)

// Always use .equals() for string comparison, never ==
```

**Lazy initialization and null reference:**
```java
// WRONG: lazy field access without synchronization
class Config {
    private Database db;

    public Database getDatabase() {
        if (db == null) {
            db = new Database();  // race condition: two threads might initialize
        }
        return db;
    }
}

// RIGHT: eager initialization or synchronized
class Config {
    private final Database db = new Database();  // thread-safe, always available

    public Database getDatabase() {
        return db;
    }
}

// Or: double-checked locking (complex, error-prone)
```

**N+1 query problem:**
```java
// WRONG: SELECT author -> for each author SELECT books
List<Author> authors = repository.findAll();  // 1 query
for (Author author : authors) {
    List<Book> books = author.getBooks();     // N queries (N = number of authors)
}

// RIGHT: eager load with @EntityGraph or JPQL fetch
@Repository
interface AuthorRepository extends JpaRepository<Author, Long> {
    @EntityGraph(attributePaths = {"books"})
    List<Author> findAll();  // 1 query with JOIN
}
```

---

## 4. Exercises

### Beginner

**(a) Hello, Java + Maven**
Create a Maven project with a simple `HelloWorld` class. Use `mvn clean compile` and `mvn exec:java` to run. Add JUnit 5 dependency and write a test for a utility method.

**(b) Bank Account OOP Kata**
Implement a `BankAccount` class with deposit, withdraw, and balance methods. Use private fields, public accessors, and validation. Write at least 5 unit tests covering normal and edge cases.

**(c) FizzBuzz with Streams**
Implement FizzBuzz using streams: given a range 1 to N, return a list where multiples of 3 are "Fizz", multiples of 5 are "Buzz", and multiples of both are "FizzBuzz". Write parameterized JUnit 5 tests.

**(d) Unit-test a PasswordValidator**
Create a `PasswordValidator` class that enforces rules (min length 8, at least one number, one uppercase). Write at least 10 test cases using JUnit 5 + AssertJ. Include edge cases (null, empty, exact boundaries).

### Intermediate

**(a) Small In-Memory CRUD Service (Spring Boot + H2)**
Build a REST API for a simple resource (e.g., `Task`). Include GET (list, by ID), POST (create), PUT (update), DELETE. Use Spring Data JPA with H2 in-memory database. Validate input with `@Valid`.

**(b) Integration Tests (MockMvc + Testcontainers)**
Write integration tests for the CRUD service above using MockMvc. Add a second test suite using Testcontainers with Postgres. Include error cases (validation, not found).

**(c) REST-assured API Test Suite**
Choose a public API (e.g., JSONPlaceholder). Write 5+ tests covering GET, POST, error cases, and JSON validation using REST-assured. Include parameterized tests.

**(d) Mockito Kata**
Create a `PaymentService` that depends on a `PaymentProcessor` (interface). Write tests using Mockito to mock the processor, verify calls, and test success/failure paths. Include argument captors.

### Advanced

**(a) CompletableFuture Pipeline**
Write a function that fans out to 3 services (simulated with delays) and aggregates results. Implement timeout fallbacks. Test with real delays and verify async behavior. Benchmark virtual threads vs platform threads.

**(b) Virtual Threads Benchmark**
Create an I/O-bound task (e.g., HTTP requests). Run with ExecutorService (platform threads) and `newVirtualThreadPerTaskExecutor()`. Measure and compare latency for 10,000 tasks.

**(c) Pact Consumer Test**
Write a consumer test using Pact that verifies a contract with an API endpoint. Run the provider API in the test and validate the Pact works.

**(d) Multi-Module Gradle Project**
Create a Gradle project with 3 modules: (1) core (domain + repo), (2) api (Spring Boot controller), (3) test-utils (shared test fixtures). Use composite build or dependency management.

### Capstone: Library API

**Scope:** Full-stack Spring Boot microservice with professional testing and CI/CD.

**Requirements:**
- **Service:** Spring Boot 3 REST API for managing books and authors. Endpoints: CRUD for books, search by author, list by genre.
- **Database:** Postgres, Flyway migrations, JPA entities with relationships.
- **Testing:** JUnit 5 + Mockito (unit), MockMvc (integration), Testcontainers (Postgres), REST-assured (API contract).
- **Coverage:** >= 80% line coverage (JaCoCo).
- **CI/CD:** GitHub Actions: test on push, build jar, upload artifact, optionally push Docker image.
- **Packaging:** Spring Boot fat JAR, Dockerfile, docker-compose.yml for local dev.
- **Documentation:** OpenAPI/Swagger, architecture diagram (README.md), API spec (generated or hand-written).

**Estimated effort:** 30-40 hours.

---

## 5. Additional Reading & Resources

- **Effective Java** (Joshua Bloch, 3rd ed, 2018): Best practices, idioms, pitfalls.
- **Modern Java in Action** (Raoul-Gabriel Urma et al., 2nd ed, 2021): Lambdas, streams, concurrency.
- **Spring in Action** (Craig Walls, 6th ed, 2022): Spring Boot, web, security, testing.
- Baeldung (baeldung.com): tutorials, examples, Spring guides.
- Oracle Java Docs: java.lang, java.util, java.util.concurrent, java.nio.file.
- JUnit 5 User Guide (junit.org/junit5).
- Mockito Documentation (javadoc, site).
- AssertJ Documentation (assertj.github.io/doc).
- REST-assured GitHub (rest-assured.io).
- Testcontainers (testcontainers.org): examples, supported services.
- Spring Framework Reference (spring.io/projects/spring-framework).
- Spring Security Reference (spring.io/projects/spring-security).
- Java Champions' blogs: Aleksey Shipilev, Nitsan Wakart, Holly Cummins.
- JetBrains Academy Java Track (freemium, hands-on).
- Marco Codes YouTube (Java/Spring content, clear explanations).

---

## 6. Index

| Topic | Section |
|-------|---------|
| Setup & Build Tools (Maven, Gradle) | 1.1 |
| Primitives, Wrappers, String, Text Blocks | 1.2 |
| Operators, Switch Expressions, Pattern Matching | 1.3 |
| Arrays, Enhanced For, Arrays Utility | 1.4 |
| Methods, Overloading, Varargs, Static | 1.5 |
| Classes, Constructors, Visibility | 2.1 |
| Inheritance, Polymorphism, Abstract, Final | 2.2 |
| Interfaces, Default/Static Methods, Records, Sealed | 2.3 |
| Generics, Collections, Equals/HashCode | 2.4 |
| Comparable, Comparator | 2.5 |
| Lambdas, Functional Interfaces, Method References | 3.1 |
| Streams: filter, map, flatMap, collect | 3.2 |
| Optional: correct usage, chaining | 3.3 |
| Exceptions: checked/unchecked, try-with-resources, NIO | 3.4 |
| Threads, Synchronized, Volatile | 3.5 |
| ExecutorService, Future, CompletableFuture, Concurrent Collections | 3.6 |
| Virtual Threads, Structured Concurrency | 3.7 |
| JVM: Class Loading, JIT, GC, Memory Model | 4.1-4.2 |
| JUnit 5: @Test, Lifecycle, Parameterized, Nested | 5.1 |
| Mockito: Mocks, Stubs, Spies, Verification, BDD | 5.2 |
| AssertJ, Testcontainers | 5.3 |
| REST-assured, Pact | 5.4 |
| Spring Boot: Starters, Autoconfiguration, @RestController | 6.1 |
| Spring DI: @Component, @Autowired, Profiles, Properties | 6.2 |
| Spring Data JPA: Entities, Repositories, Pagination | 6.3 |
| Validation, Content Negotiation | 6.4 |
| Spring Test: @SpringBootTest, @WebMvcTest, @DataJpaTest, Testcontainers | 6.5 |
| Spring Security: Authentication, Authorization, JWT | 6.6 |
| Logging: SLF4J, Logback, Structured Logging | 7.1 |
| Build Pipelines: GitHub Actions, Maven/Gradle Caching | 7.2 |
| Packaging: Fat JAR, Docker, docker-compose | 7.3 |
| Micro-Interview Patterns: ConcurrentModification, equals/hashCode, N+1 | 7.4 |

---

## 7. References

1. Bloch, J. (2018). *Effective Java* (3rd ed.). Addison-Wesley.
2. Urma, R., Fusco, A., & Mycroft, A. (2021). *Modern Java in Action* (2nd ed.). Manning.
3. Walls, C. (2022). *Spring in Action* (6th ed.). Manning.
4. JUnit 5 User Guide. Retrieved from https://junit.org/junit5/docs/current/user-guide/
5. Mockito Documentation. Retrieved from https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html
6. AssertJ. Retrieved from https://assertj.github.io/
7. REST-assured. Retrieved from https://rest-assured.io/
8. Testcontainers. Retrieved from https://www.testcontainers.org/
9. Spring Framework Reference. Retrieved from https://spring.io/projects/spring-framework
10. Spring Security Reference. Retrieved from https://spring.io/projects/spring-security
11. Spring Data JPA Reference. Retrieved from https://spring.io/projects/spring-data-jpa
12. Shipilev, A. (2014). "The Java Memory Model." Retrieved from https://shipilev.net/blog/
13. Gorelick, M., & Verbruggen, P. (2020). "Java Performance Tuning." Retrieved from https://www.oreilly.com/
