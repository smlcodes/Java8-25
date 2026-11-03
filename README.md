# Java 8 to Java 25: A Practical Guide

A beginner-friendly tour of the Java language and platform features added between Java 8 (2014) and Java 25 (2025), with a focus on what matters in real-world enterprise and Spring Boot applications.

## How to Read This Guide

- Each version lists its **major and widely used** features. Minor internal changes are skipped on purpose.
- Every feature has a **Description**, **Why it matters**, an **Example**, and a **Before vs After** comparison where it helps.
- **Preview** features need `--enable-preview` and can change. Avoid them in production code. **Incubator** features live in `jdk.incubator.*` modules and are even less stable.
- **LTS** (Long-Term Support) releases are the ones most companies run in production: **8, 11, 17, 21, 25**.
- Examples are small and self-contained. Snippets assume the usual imports (`java.util.*`, `java.util.stream.*`).

## Table of Contents

- [Java 8](#java-8) · [Java 9](#java-9) · [Java 10](#java-10) · [Java 11](#java-11) · [Java 12](#java-12) · [Java 13](#java-13)
- [Java 14](#java-14) · [Java 15](#java-15) · [Java 16](#java-16) · [Java 17](#java-17) · [Java 18](#java-18) · [Java 19](#java-19)
- [Java 20](#java-20) · [Java 21](#java-21) · [Java 22](#java-22) · [Java 23](#java-23) · [Java 24](#java-24) · [Java 25](#java-25)
- [Summary Table](#summary-table)

---

# Java 8

<sub>Released: March 2014 · **LTS** · Theme: functional programming comes to Java</sub>

Java 8 is the biggest language upgrade in Java's history. Almost all modern Java code builds on it.

### 1. Lambda Expressions

**Description:** A lambda is a short, anonymous function. Instead of writing a whole class to pass a bit of behavior, you write `(parameters) -> expression`.

**Why it matters:** It removes the boilerplate of anonymous inner classes and makes callbacks, comparators, and stream operations much easier to read.

**Example:**

```java
List<String> names = Arrays.asList("Charlie", "alice", "Bob");
names.sort((a, b) -> a.compareToIgnoreCase(b));
names.forEach(n -> System.out.println(n));
```

**Before vs After:**

```java
// Before (Java 7)
Collections.sort(names, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return a.compareToIgnoreCase(b);
    }
});

// After (Java 8)
names.sort((a, b) -> a.compareToIgnoreCase(b));
```

### 2. Functional Interfaces and Method References

**Description:** A functional interface has exactly one abstract method, so a lambda can implement it. Java 8 ships ready-made ones in `java.util.function` (`Function`, `Predicate`, `Consumer`, `Supplier`). A method reference (`Class::method`) is a shortcut for a lambda that only calls one method.

**Why it matters:** You rarely have to define your own single-method interfaces. Method references make code read almost like English.

**Example:**

```java
Function<String, Integer> length = String::length;
Predicate<String> isEmpty = String::isEmpty;
Supplier<List<String>> listMaker = ArrayList::new;

System.out.println(length.apply("Java"));   // 4
```

**Before vs After:**

```java
// Before: lambda
names.forEach(n -> System.out.println(n));

// After: method reference
names.forEach(System.out::println);
```

### 3. Streams API

**Description:** A stream is a pipeline for processing a collection of data: filter, transform, then collect a result. Streams do not change the original collection.

**Why it matters:** You describe *what* you want instead of writing loops that describe *how*. It is the standard way to filter, group, and aggregate data in service and repository code.

**Example:**

```java
record Employee(String name, String dept, double salary) {} // record needs Java 16; use a class on Java 8

Map<String, Double> avgByDept = employees.stream()
    .filter(e -> e.salary() > 30_000)
    .collect(Collectors.groupingBy(Employee::dept,
             Collectors.averagingDouble(Employee::salary)));
```

**Before vs After:**

```java
// Before: loop + temporary list
List<String> result = new ArrayList<>();
for (String n : names) {
    if (n.length() > 3) {
        result.add(n.toUpperCase());
    }
}

// After: stream pipeline
List<String> result = names.stream()
    .filter(n -> n.length() > 3)
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

### 4. Optional

**Description:** `Optional<T>` is a container that either holds a value or is empty. It makes "this might be missing" explicit in the method signature.

**Why it matters:** It reduces `NullPointerException`s and makes null-handling intent clear. Spring Data repositories return `Optional` from methods like `findById`.

**Example:**

```java
Optional<String> email = Optional.ofNullable(user.getEmail());
String shown = email.map(String::toLowerCase).orElse("no-email");
```

**Before vs After:**

```java
// Before
String shown;
if (user != null && user.getEmail() != null) {
    shown = user.getEmail().toLowerCase();
} else {
    shown = "no-email";
}

// After
String shown = Optional.ofNullable(user)
    .map(User::getEmail)
    .map(String::toLowerCase)
    .orElse("no-email");
```

### 5. Default and Static Methods in Interfaces

**Description:** Interfaces can now contain methods with a body, marked `default` (inherited by implementers) or `static`.

**Why it matters:** Library authors can add new methods to an interface without breaking every existing implementation. This is how `List.sort()` and `Collection.stream()` were added.

**Example:**

```java
interface Greeter {
    String name();

    default String greet() {            // has a body
        return "Hello, " + name();
    }

    static Greeter of(String n) {       // static factory
        return () -> n;
    }
}

System.out.println(Greeter.of("Maneesha").greet());
```

### 6. New Date and Time API (`java.time`)

**Description:** A modern, immutable, thread-safe replacement for `java.util.Date` and `Calendar`. Key types: `LocalDate`, `LocalDateTime`, `ZonedDateTime`, `Instant`, `Duration`, `Period`, and `DateTimeFormatter`.

**Why it matters:** The old API was mutable, confusing (months started at 0), and not thread-safe. `java.time` is clear and is the default choice in JPA entities and REST DTOs today.

**Example:**

```java
LocalDate today = LocalDate.now();
LocalDate due = today.plusDays(30);
long days = ChronoUnit.DAYS.between(today, due);   // 30
String text = due.format(DateTimeFormatter.ofPattern("dd MMM yyyy"));
```

**Before vs After:**

```java
// Before
Calendar cal = Calendar.getInstance();
cal.add(Calendar.DAY_OF_MONTH, 30);
Date due = cal.getTime();
SimpleDateFormat f = new SimpleDateFormat("dd MMM yyyy"); // not thread-safe!

// After
LocalDate due = LocalDate.now().plusDays(30);
```

### 7. CompletableFuture

**Description:** A `Future` you can chain, combine, and handle errors on, without blocking.

**Why it matters:** It enables non-blocking, composed asynchronous code, such as calling two services in parallel and combining the results.

**Example:**

```java
CompletableFuture<String> user  = CompletableFuture.supplyAsync(() -> "Maneesha");
CompletableFuture<Integer> orders = CompletableFuture.supplyAsync(() -> 5);

String summary = user.thenCombine(orders, (u, o) -> u + " has " + o + " orders")
                     .exceptionally(ex -> "unavailable")
                     .join();
```

### Other Notable Java 8 Additions

- `String.join(", ", list)` for joining strings.
- `Base64` encoder/decoder in the standard library.
- `Map.getOrDefault`, `computeIfAbsent`, and `merge`.
- Repeating annotations and type annotations.

---

# Java 9

<sub>Released: September 2017 · Theme: modularity and convenience APIs</sub>

### 1. Module System (Project Jigsaw)

**Description:** Code can be grouped into **modules** described in a `module-info.java` file. A module declares what it *requires* and what it *exports*. Everything else stays hidden.

**Why it matters:** It gives strong encapsulation (internal packages can't be accessed by accident) and made the JDK itself modular, so smaller runtimes are possible. Most Spring Boot apps still run on the classpath, but the module system explains many "illegal access" warnings and errors during upgrades.

**Example:**

```java
// module-info.java
module com.example.orders {
    requires java.sql;
    exports com.example.orders.api;   // only this package is visible to others
}
```

### 2. Collection Factory Methods

**Description:** `List.of(...)`, `Set.of(...)`, and `Map.of(...)` create small **immutable** collections in one line.

**Why it matters:** Less ceremony for constants and test data, and immutability makes code safer.

**Example:**

```java
List<String> roles = List.of("ADMIN", "USER");
Map<String, Integer> ports = Map.of("http", 80, "https", 443);
```

**Before vs After:**

```java
// Before
List<String> roles = new ArrayList<>();
roles.add("ADMIN");
roles.add("USER");
roles = Collections.unmodifiableList(roles);

// After
List<String> roles = List.of("ADMIN", "USER");
```

> Note: these collections reject `null` and throw `UnsupportedOperationException` if you try to modify them.

### 3. Private Interface Methods

**Description:** Interfaces can have `private` methods to share code between their `default` methods.

**Why it matters:** It avoids duplicate logic in default methods without exposing helper methods publicly.

**Example:**

```java
interface Logger {
    default void info(String m)  { log("INFO", m); }
    default void error(String m) { log("ERROR", m); }

    private void log(String level, String m) {
        System.out.println("[" + level + "] " + m);
    }
}
```

### 4. Stream and Optional Improvements

**Description:** New stream methods: `takeWhile`, `dropWhile`, `ofNullable`, and `iterate` with a stop condition. `Optional` gained `ifPresentOrElse`, `or`, and `stream`.

**Why it matters:** Common patterns now need less code.

**Example:**

```java
Stream.of(1, 2, 3, 10, 4).takeWhile(n -> n < 5).toList();   // toList() needs Java 16; [1, 2, 3]

Stream.iterate(1, n -> n < 100, n -> n * 2).forEach(System.out::println);

optionalUser.ifPresentOrElse(
    u -> System.out.println("Found " + u),
    () -> System.out.println("Not found"));
```

**Before vs After:**

```java
// Before: no clean way to write an if/else on an Optional
if (optionalUser.isPresent()) {
    System.out.println("Found " + optionalUser.get());
} else {
    System.out.println("Not found");
}

// After
optionalUser.ifPresentOrElse(u -> System.out.println("Found " + u),
                             () -> System.out.println("Not found"));
```

### 5. JShell (Interactive REPL)

**Description:** A command-line tool (`jshell`) where you type Java code and see results immediately, with no class or `main` method needed.

**Why it matters:** It is great for learning and for quickly trying an API.

**Example:**

```text
$ jshell
jshell> "hello".toUpperCase()
$1 ==> "HELLO"
```

### 6. Try-with-Resources on Existing Variables

**Description:** You can use an existing `final` or effectively final variable in `try (...)` instead of declaring a new one.

**Example:**

```java
BufferedReader reader = new BufferedReader(new FileReader("data.txt"));
try (reader) {
    System.out.println(reader.readLine());
}
```

---

# Java 10

<sub>Released: March 2018 · Theme: less typing</sub>

### 1. Local Variable Type Inference (`var`)

**Description:** For local variables, you can write `var` and let the compiler infer the type from the right-hand side. The variable is still strongly typed.

**Why it matters:** It cuts repeated type names, especially with generics. Use it when the type is obvious from the right side.

**Example:**

```java
var names = new ArrayList<String>();   // ArrayList<String>
var count = 10;                        // int
for (var name : names) { /* ... */ }
```

**Before vs After:**

```java
// Before
Map<String, List<Employee>> byDept = new HashMap<String, List<Employee>>();

// After
var byDept = new HashMap<String, List<Employee>>();
```

> `var` only works for local variables, not fields, method parameters, or return types. Avoid it when it hides the type, such as `var x = getData();`.

### 2. Unmodifiable Collection Helpers

**Description:** `List.copyOf`, `Set.copyOf`, `Map.copyOf`, and `Collectors.toUnmodifiableList/Set/Map`.

**Why it matters:** Safely return or store immutable snapshots of data.

**Example:**

```java
List<String> safe = List.copyOf(mutableList);
```

### 3. Container Awareness

**Description:** The JVM now reads CPU and memory limits from Docker/Kubernetes containers (cgroups) instead of looking at the whole host machine.

**Why it matters:** Heap size and thread-pool sizing are now correct in containers. This is a big deal for Spring Boot services on Kubernetes.

### 4. `Optional.orElseThrow()`

**Description:** A no-argument `orElseThrow()` that throws `NoSuchElementException`. It is a clearer name than `get()`.

**Example:**

```java
String value = optional.orElseThrow();
```

---

# Java 11

<sub>Released: September 2018 · **LTS** · Theme: a stable, cleaned-up baseline</sub>

Java 11 was the first LTS after Java 8 and the version where many companies finally moved off Java 8.

### 1. New `String` Methods

**Description:** `isBlank()`, `strip()`, `stripLeading()`, `stripTrailing()`, `lines()`, and `repeat(n)`.

**Why it matters:** These handle common text tasks without helper libraries, and `strip()` understands Unicode whitespace (unlike `trim()`).

**Example:**

```java
"   ".isBlank();                        // true
"  hello  ".strip();                    // "hello"
"a\nb\nc".lines().count();              // 3
"ab".repeat(3);                         // "ababab"
```

**Before vs After:**

```java
// Before
if (text == null || text.trim().isEmpty()) { /* ... */ }

// After
if (text == null || text.isBlank()) { /* ... */ }
```

### 2. Standard HTTP Client

**Description:** A modern `java.net.http.HttpClient` with HTTP/2, synchronous and asynchronous calls, and a clean builder API.

**Why it matters:** You can make HTTP calls without `HttpURLConnection` or an extra library.

**Example:**

```java
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/users/1"))
    .header("Accept", "application/json")
    .GET()
    .build();

HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
System.out.println(response.statusCode() + " " + response.body());
```

**Before vs After:**

```java
// Before: verbose HttpURLConnection
HttpURLConnection con = (HttpURLConnection) new URL("https://api.example.com/users/1").openConnection();
con.setRequestMethod("GET");
BufferedReader in = new BufferedReader(new InputStreamReader(con.getInputStream()));
// ...read lines, close streams...
```

### 3. `var` in Lambda Parameters

**Description:** You may use `var` for lambda parameters, which allows annotations on them.

**Example:**

```java
BiFunction<Integer, Integer, Integer> add = (@NonNull var a, @NonNull var b) -> a + b;
```

### 4. Simple File Read/Write

**Description:** `Files.readString(path)` and `Files.writeString(path, text)`.

**Example:**

```java
Path p = Path.of("notes.txt");
Files.writeString(p, "Hello");
String content = Files.readString(p);
```

**Before vs After:**

```java
// Before
String content = new String(Files.readAllBytes(Paths.get("notes.txt")), StandardCharsets.UTF_8);

// After
String content = Files.readString(Path.of("notes.txt"));
```

### 5. Run a Single Source File Directly

**Description:** `java Hello.java` compiles and runs a single file in one step.

**Example:**

```text
$ java Hello.java
```

### 6. `Predicate.not`

**Example:**

```java
lines.stream().filter(Predicate.not(String::isBlank)).toList();   // toList() needs Java 16
```

### Migration Note: Java EE and CORBA Modules Removed

Java 11 removed `javax.xml.bind` (JAXB), `javax.activation`, `javax.annotation`, and the CORBA modules from the JDK. If you upgrade from Java 8, add them as explicit dependencies (for example, `jakarta.xml.bind-api`) or you will see `ClassNotFoundException`.

---

# Java 12

<sub>Released: March 2019 · Theme: small API additions</sub>

### 1. `Collectors.teeing`

**Description:** Runs two collectors on the same stream in a single pass, then merges their results.

**Why it matters:** Compute two things (such as a count and a sum) without streaming the data twice.

**Example:**

```java
double average = Stream.of(10, 20, 30).collect(
    Collectors.teeing(
        Collectors.summingInt(i -> i),
        Collectors.counting(),
        (sum, count) -> (double) sum / count));   // 20.0
```

### 2. `String.indent` and `String.transform`

**Example:**

```java
String indented = "line1\nline2".indent(4);
int length = "java".transform(String::length);   // 4
```

### 3. Compact Number Formatting

**Description:** Format numbers as "1K", "2M", and so on.

**Example:**

```java
NumberFormat nf = NumberFormat.getCompactNumberInstance(Locale.US, NumberFormat.Style.SHORT);
nf.setMaximumFractionDigits(1);
nf.format(1_200_000);   // "1.2M"
```

### 4. Switch Expressions (Preview)

First preview of the feature that became final in [Java 14](#java-14).

---

# Java 13

<sub>Released: September 2019 · Theme: previews of future language features</sub>

Java 13 added mostly previews. Two of them became permanent features later:

- **Text Blocks (preview):** multi-line strings with `"""`. Final in [Java 15](#java-15).
- **Switch Expressions (second preview):** adds the `yield` keyword. Final in [Java 14](#java-14).

Other change: **Dynamic CDS Archives** let you create a class-data-sharing archive when the app exits, which improves startup time with little effort.

---

# Java 14

<sub>Released: March 2020 · Theme: switch expressions and better error messages</sub>

### 1. Switch Expressions (Standard)

**Description:** `switch` can now return a value, uses `->` arrows, supports multiple labels per case, and has no accidental fall-through. Use `yield` to return a value from a block.

**Why it matters:** It is shorter, safer (no forgotten `break`), and the compiler can check that every case is covered.

**Example:**

```java
enum Day { MON, TUE, SAT, SUN }

String type = switch (day) {
    case SAT, SUN -> "Weekend";
    case MON, TUE -> {
        String s = "Week";
        yield s + "day";            // yield returns from a block
    }
};
```

**Before vs After:**

```java
// Before
String type;
switch (day) {
    case SAT:
    case SUN:
        type = "Weekend";
        break;
    default:
        type = "Weekday";
        break;
}

// After
String type = switch (day) {
    case SAT, SUN -> "Weekend";
    default       -> "Weekday";
};
```

### 2. Helpful NullPointerException Messages

**Description:** The JVM now tells you exactly what was null.

**Why it matters:** It saves a lot of debugging time on chained calls.

**Example:**

```java
String city = user.getAddress().getCity().toUpperCase();
```

**Before vs After:**

```text
// Before
Exception in thread "main" java.lang.NullPointerException

// After
Exception in thread "main" java.lang.NullPointerException:
  Cannot invoke "Address.getCity()" because the return value of "User.getAddress()" is null
```

### 3. Previews That Became Final in Java 16

Records and pattern matching for `instanceof` were first previewed here. See [Java 16](#java-16).

---

# Java 15

<sub>Released: September 2020 · Theme: text blocks and sealed-class preview</sub>

### 1. Text Blocks (Standard)

**Description:** A multi-line string literal written between triple quotes `"""`. Indentation is trimmed automatically and you don't need `\n` or escaped quotes.

**Why it matters:** It makes embedded JSON, SQL, HTML, and XML readable. It is very helpful in tests and in `@Query` annotations.

**Example:**

```java
String json = """
    {
      "name": "Maneesha",
      "role": "Engineer"
    }
    """;
```

**Before vs After:**

```java
// Before
String json = "{\n" +
              "  \"name\": \"Maneesha\",\n" +
              "  \"role\": \"Engineer\"\n" +
              "}\n";

// After: see the text block above
```

### 2. `String.formatted`

**Description:** An instance-method version of `String.format`.

**Example:**

```java
String msg = "Hello %s, you have %d items".formatted(name, count);
```

### 3. Sealed Classes (Preview)

First preview of the feature that became final in [Java 17](#java-17).

### 4. ZGC and Shenandoah Become Production-Ready

Two low-pause garbage collectors are no longer experimental. They are useful for latency-sensitive services.

---

# Java 16

<sub>Released: March 2021 · Theme: records and pattern matching arrive</sub>

### 1. Records

**Description:** A record is a short way to declare an immutable data class. Java generates the constructor, getters (named like the fields, without `get`), `equals`, `hashCode`, and `toString`.

**Why it matters:** It removes pages of boilerplate for DTOs, API responses, and value objects. It is a good fit for Spring Boot request/response models and projections.

**Example:**

```java
public record User(String name, int age) {

    // Optional compact constructor for validation
    public User {
        if (age < 0) throw new IllegalArgumentException("age must be >= 0");
    }
}

var u = new User("Maneesha", 30);
System.out.println(u.name());   // Maneesha
System.out.println(u);          // User[name=Maneesha, age=30]
```

**Before vs After:**

```java
// Before: ~30 lines
public final class User {
    private final String name;
    private final int age;

    public User(String name, int age) { this.name = name; this.age = age; }
    public String getName() { return name; }
    public int getAge() { return age; }
    @Override public boolean equals(Object o) { /* ... */ }
    @Override public int hashCode() { /* ... */ }
    @Override public String toString() { /* ... */ }
}

// After: 1 line
public record User(String name, int age) {}
```

### 2. Pattern Matching for `instanceof`

**Description:** Check a type and get a typed variable in one step.

**Why it matters:** It removes the repeated cast after every `instanceof`.

**Example:**

```java
if (obj instanceof String s && s.length() > 3) {
    System.out.println(s.toUpperCase());
}
```

**Before vs After:**

```java
// Before
if (obj instanceof String) {
    String s = (String) obj;
    System.out.println(s.toUpperCase());
}

// After
if (obj instanceof String s) {
    System.out.println(s.toUpperCase());
}
```

### 3. `Stream.toList()`

**Description:** A shortcut for collecting a stream into an unmodifiable list.

**Example and Before vs After:**

```java
// Before
List<String> upper = names.stream().map(String::toUpperCase).collect(Collectors.toList());

// After
List<String> upper = names.stream().map(String::toUpperCase).toList();
```

> The returned list is unmodifiable, while `Collectors.toList()` returns a normal modifiable list.

---

# Java 17

<sub>Released: September 2021 · **LTS** · Theme: sealed hierarchies and a modern baseline</sub>

Java 17 is the **minimum version required by Spring Framework 6 and Spring Boot 3**, so it is the baseline for most new enterprise projects.

### 1. Sealed Classes and Interfaces

**Description:** A sealed type lists exactly which classes may extend or implement it, using `permits`. Subclasses must be `final`, `sealed`, or `non-sealed`.

**Why it matters:** You can model a closed set of options, such as payment types, and the compiler knows all of them. This pairs very well with `switch` (see [Java 21](#java-21)).

**Example:**

```java
public sealed interface Shape permits Circle, Rectangle {}

public record Circle(double radius) implements Shape {}
public record Rectangle(double width, double height) implements Shape {}
```

**Before vs After:**

```java
// Before: anyone can implement Shape, so code can never be sure it has handled every type
public interface Shape {}

// After: only Circle and Rectangle are allowed; adding a new subtype forces a compile-time review
public sealed interface Shape permits Circle, Rectangle {}
```

### 2. Strong Encapsulation of JDK Internals

**Description:** Reflection into internal JDK packages (such as `sun.misc` helpers and `java.lang` internals) is now blocked by default. The old `--illegal-access` flag is gone.

**Why it matters:** This is a common cause of upgrade failures with old libraries (older Lombok, Mockito, Hibernate, and serialization libraries). Update dependencies, or as a last resort use `--add-opens`.

**Example:**

```text
java --add-opens java.base/java.lang=ALL-UNNAMED -jar app.jar
```

### 3. Enhanced Pseudo-Random Number Generators

**Description:** A common `RandomGenerator` interface and new algorithms.

**Example:**

```java
RandomGenerator rng = RandomGenerator.getDefault();
int dice = rng.nextInt(1, 7);   // 1 to 6
```

### 4. Pattern Matching for `switch` (Preview)

First previewed here. Final in [Java 21](#java-21).

---

# Java 18

<sub>Released: March 2022 · Theme: sensible defaults</sub>

### 1. UTF-8 by Default

**Description:** The default charset for file and text APIs is now UTF-8 on every operating system.

**Why it matters:** Code that used to behave differently on Windows, Linux, and macOS (because of the platform default charset) now behaves the same.

**Before vs After:**

```java
// Before: depends on the OS default encoding
new FileReader("data.txt");

// After: UTF-8 everywhere; you can still be explicit
new FileReader("data.txt", StandardCharsets.UTF_8);
```

### 2. Simple Web Server

**Description:** A minimal static file server started with one command.

**Example:**

```text
$ jwebserver -p 8000
```

**Why it matters:** Handy for quick demos, serving test files, or prototyping.

### 3. `@snippet` in Javadoc

Lets you embed real, compilable code samples in Javadoc.

---

# Java 19

<sub>Released: September 2022 · Theme: previews for concurrency and patterns</sub>

This release mainly previewed features that arrived in [Java 21](#java-21):

- **Virtual Threads (preview):** lightweight threads managed by the JVM.
- **Record Patterns (preview):** take apart a record directly in a pattern.
- **Structured Concurrency (incubator):** treat related tasks as one unit.

Nothing here is meant for production code. Read the Java 21 section instead.

---

# Java 20

<sub>Released: March 2023 · Theme: previews, round two</sub>

Another preview-heavy release:

- **Scoped Values (incubator):** a safer alternative to `ThreadLocal`. Final in [Java 25](#java-25).
- Second previews of **Virtual Threads**, **Record Patterns**, and **Pattern Matching for switch**.

---

# Java 21

<sub>Released: September 2023 · **LTS** · Theme: virtual threads and data-oriented programming</sub>

The most important release since Java 17. Pattern matching, records, and sealed types now work together, and virtual threads change how server applications handle concurrency.

### 1. Virtual Threads

**Description:** Virtual threads are very cheap threads managed by the JVM instead of the operating system. You can run hundreds of thousands at once. When one waits for I/O (a database or HTTP call), the JVM parks it and reuses the real OS thread.

**Why it matters:** Traditional "one thread per request" servers run out of OS threads under heavy load. With virtual threads you keep the simple, blocking code style and still scale, with no reactive programming needed.

**Example:**

```java
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 10_000; i++) {
        executor.submit(() -> {
            Thread.sleep(Duration.ofSeconds(1));   // blocking is cheap now
            return "done";
        });
    }
}   // closing the executor waits for all tasks

Thread.startVirtualThread(() -> System.out.println("Hello from a virtual thread"));
```

**Before vs After:**

```java
// Before: a limited pool of heavy platform threads
ExecutorService pool = Executors.newFixedThreadPool(200);

// After: one cheap virtual thread per task
ExecutorService pool = Executors.newVirtualThreadPerTaskExecutor();
```

**Spring Boot tip:** From Spring Boot 3.2, enable virtual threads for Tomcat and `@Async` with one property:

```properties
spring.threads.virtual.enabled=true
```

> Use virtual threads for I/O-bound work (web requests, database calls). They do not speed up CPU-heavy work. Don't pool them; create one per task.

### 2. Pattern Matching for `switch`

**Description:** `switch` can now match on **types** (and add conditions with `when`), not just constants. It also handles `null` and can be checked for completeness.

**Why it matters:** It replaces long `if / else if (x instanceof ...)` chains. Combined with sealed types, the compiler warns you if a case is missing.

**Example:**

```java
static String describe(Object obj) {
    return switch (obj) {
        case null                    -> "null value";
        case Integer i when i > 100  -> "big number " + i;
        case Integer i               -> "number " + i;
        case String s                -> "text of length " + s.length();
        default                      -> "something else";
    };
}
```

**Before vs After:**

```java
// Before
static String describe(Object obj) {
    if (obj == null) return "null value";
    if (obj instanceof Integer) {
        Integer i = (Integer) obj;
        return i > 100 ? "big number " + i : "number " + i;
    }
    if (obj instanceof String) {
        return "text of length " + ((String) obj).length();
    }
    return "something else";
}
```

### 3. Record Patterns

**Description:** Break a record into its components right inside a pattern. Patterns can be nested.

**Why it matters:** You get the data you need in one step, without calling accessor methods.

**Example (with a sealed interface):**

```java
sealed interface Shape permits Circle, Rectangle {}
record Circle(double radius) implements Shape {}
record Rectangle(double width, double height) implements Shape {}

static double area(Shape shape) {
    return switch (shape) {
        case Circle(double r)               -> Math.PI * r * r;
        case Rectangle(double w, double h)  -> w * h;
        // no default needed: the compiler knows these are all the Shape types
    };
}
```

**Before vs After:**

```java
// Before
if (shape instanceof Circle) {
    Circle c = (Circle) shape;
    double r = c.radius();
    return Math.PI * r * r;
}

// After
if (shape instanceof Circle(double r)) {
    return Math.PI * r * r;
}
```

### 4. Sequenced Collections

**Description:** New interfaces (`SequencedCollection`, `SequencedSet`, `SequencedMap`) add a standard way to get the first and last element and to reverse. They are implemented by `List`, `Deque`, `LinkedHashSet`, `LinkedHashMap`, `SortedSet`, and others.

**Why it matters:** Before this, getting the "last element" was different for every collection type.

**Example:**

```java
List<String> list = new ArrayList<>(List.of("a", "b", "c"));
list.getFirst();      // "a"
list.getLast();       // "c"
list.addFirst("z");   // [z, a, b, c]
list.reversed();      // view in reverse order
```

**Before vs After:**

```java
// Before
String last = list.get(list.size() - 1);              // List
String lastInSet = new ArrayList<>(linkedSet).get(linkedSet.size() - 1);  // LinkedHashSet: awkward

// After
String last = list.getLast();
String lastInSet = linkedSet.getLast();
```

### 5. Generational ZGC

**Description:** The low-pause ZGC collector now separates young and old objects.

**Why it matters:** Lower CPU use and better throughput while keeping very short pauses. Enable it with:

```text
-XX:+UseZGC -XX:+ZGenerational
```

### 6. Small API Additions

```java
Math.clamp(value, 0, 100);                 // keep a value inside a range
new StringBuilder().repeat("ab", 3);       // "ababab"
"hello world".indexOf('o', 0, 5);          // search inside a range
```

### Previews in Java 21 (Do Not Use in Production)

String Templates (preview; **removed again in Java 23**), Unnamed Patterns and Variables (final in [Java 22](#java-22)), and Unnamed Classes and Instance Main Methods (final in [Java 25](#java-25)).

---

# Java 22

<sub>Released: March 2024 · Theme: cleaner syntax and native interop</sub>

### 1. Unnamed Variables and Patterns (`_`)

**Description:** Use `_` for a variable you must declare but never use.

**Why it matters:** It shows clearly that a value is intentionally ignored, which removes "unused variable" noise.

**Example:**

```java
try {
    Integer.parseInt(text);
} catch (NumberFormatException _) {
    System.out.println("Not a number");
}

int count = 0;
for (var _ : list) {
    count++;
}

if (shape instanceof Rectangle(double w, _)) {   // only width is needed
    System.out.println(w);
}
```

**Before vs After:**

```java
// Before
catch (NumberFormatException ignored) { ... }
for (var item : list) { count++; }          // item unused

// After
catch (NumberFormatException _) { ... }
for (var _ : list) { count++; }
```

### 2. Foreign Function and Memory API (Standard)

**Description:** A safe, pure-Java way to call native (C) libraries and work with off-heap memory.

**Why it matters:** It is the modern replacement for JNI, which was verbose and error-prone. Mainly useful for libraries and performance-critical integrations.

**Example (calling C's `strlen`):**

```java
Linker linker = Linker.nativeLinker();
MethodHandle strlen = linker.downcallHandle(
    linker.defaultLookup().find("strlen").orElseThrow(),
    FunctionDescriptor.of(ValueLayout.JAVA_LONG, ValueLayout.ADDRESS));

try (Arena arena = Arena.ofConfined()) {
    MemorySegment str = arena.allocateFrom("Hello");
    long len = (long) strlen.invokeExact(str);   // 5
}
```

### 3. Launch Multi-File Source Programs

**Description:** `java Main.java` can now compile and run a program split into several `.java` files in the same folder.

**Why it matters:** Good for scripts and small tools without a build setup.

### 4. Previews in Java 22

Statements before `super(...)` (final in [Java 25](#java-25)) and Stream Gatherers (final in [Java 24](#java-24)).

---

# Java 23

<sub>Released: September 2024 · Theme: documentation and GC defaults</sub>

### 1. Markdown Documentation Comments

**Description:** Javadoc comments can be written in Markdown using `///`.

**Why it matters:** No more `<p>`, `<pre>`, or `{@code ...}` noise in documentation.

**Example:**

```java
/// Returns the **total price** including tax.
///
/// Example:
/// ```java
/// double total = calc.totalWithTax(100);
/// ```
///
/// @param amount the base price
/// @return the price including tax
double totalWithTax(double amount) { ... }
```

### 2. Generational ZGC Becomes the Default ZGC Mode

When you enable `-XX:+UseZGC`, you now get the generational version automatically.

### 3. Changes to Preview Features

- **String Templates** were removed (the design is being reworked).
- **Primitive Types in Patterns** and **Module Import Declarations** appeared as previews (see [Java 25](#java-25)).

---

# Java 24

<sub>Released: March 2025 · Theme: stream gatherers, virtual-thread fixes, and security</sub>

### 1. Stream Gatherers (Standard)

**Description:** `Stream.gather(...)` lets you add custom intermediate operations to a stream. The built-in `Gatherers` class includes ready-made ones such as `windowFixed`, `windowSliding`, `fold`, `scan`, and `mapConcurrent`.

**Why it matters:** Streams used to be limited to the built-in operations (`map`, `filter`, and so on). Now you can batch, take sliding windows, or keep running totals directly in a pipeline.

**Example:**

```java
Stream.of(1, 2, 3, 4, 5, 6, 7)
      .gather(Gatherers.windowFixed(3))
      .toList();                     // [[1, 2, 3], [4, 5, 6], [7]]

Stream.of(1, 2, 3, 4)
      .gather(Gatherers.windowSliding(2))
      .toList();                     // [[1, 2], [2, 3], [3, 4]]
```

**Before vs After:**

```java
// Before: manual batching with a loop and index math
List<List<Integer>> batches = new ArrayList<>();
for (int i = 0; i < data.size(); i += 3) {
    batches.add(data.subList(i, Math.min(i + 3, data.size())));
}

// After
List<List<Integer>> batches = data.stream().gather(Gatherers.windowFixed(3)).toList();
```

### 2. Virtual Threads No Longer Get Pinned by `synchronized`

**Description:** In Java 21, a virtual thread that blocked inside a `synchronized` block held on to its OS thread ("pinning"). That is fixed in Java 24.

**Why it matters:** Virtual threads now scale well even with older libraries that use `synchronized`. This was the main caveat of virtual threads in Java 21.

### 3. Class-File API (Standard)

**Description:** A standard API to read, write, and transform `.class` files.

**Why it matters:** Mostly for framework and tool authors. It removes the need for libraries such as ASM that must be updated for every new Java release. As an app developer you benefit indirectly.

### 4. Ahead-of-Time Class Loading and Linking

**Description:** The JVM can store loaded and linked classes from a training run and reuse them at startup.

**Why it matters:** Faster startup for large applications, including Spring Boot services. Spring Boot's AOT cache support builds on this.

### 5. Security Updates

- **Security Manager permanently disabled.** Old code that depends on it must be changed.
- **Quantum-resistant algorithms** (ML-KEM for key exchange and ML-DSA for signatures) are available in the standard security API.

### 6. Experimental and Preview

Compact object headers (experimental), plus further previews of Scoped Values, Structured Concurrency, Flexible Constructor Bodies, Module Imports, Primitive Patterns, and Simple Source Files. All of these are covered in [Java 25](#java-25).

---

# Java 25

<sub>Released: September 2025 · **LTS** · Theme: simpler Java, scoped values, and lower memory use</sub>

Java 25 is the current LTS release. Several long-running previews are now final.

### 1. Scoped Values (Standard)

**Description:** A `ScopedValue` holds data that is shared with a method and everything it calls, but only for the duration of one block of code. The value is immutable inside that block.

**Why it matters:** It is a safer, cheaper alternative to `ThreadLocal`, especially with virtual threads. Typical uses: the current user, a request id, or a tenant id, passed down the call chain without adding a parameter to every method.

**Example:**

```java
private static final ScopedValue<String> REQUEST_ID = ScopedValue.newInstance();

void handleRequest(String id) {
    ScopedValue.where(REQUEST_ID, id).run(() -> service());
}

void service() {
    System.out.println("Handling " + REQUEST_ID.get());   // visible here...
    repository();
}

void repository() {
    System.out.println("Query for " + REQUEST_ID.get());  // ...and here
}
```

**Before vs After:**

```java
// Before: ThreadLocal must be cleaned up manually, can leak, and is mutable
private static final ThreadLocal<String> REQUEST_ID = new ThreadLocal<>();

REQUEST_ID.set(id);
try {
    service();
} finally {
    REQUEST_ID.remove();     // easy to forget
}

// After: the value disappears automatically when the block ends
ScopedValue.where(REQUEST_ID, id).run(() -> service());
```

### 2. Flexible Constructor Bodies

**Description:** You may now run statements (validation, calculations) **before** calling `super(...)` or `this(...)`. You just can't use the object being built (`this`) yet.

**Why it matters:** A subclass can validate or prepare arguments before the parent constructor runs, without tricks.

**Example:**

```java
class Person {
    Person(String name) { /* ... */ }
}

class Employee extends Person {
    Employee(String name, int age) {
        if (age < 18) {                                   // runs before super(...)
            throw new IllegalArgumentException("Must be an adult");
        }
        super(name);
    }
}
```

**Before vs After:**

```java
// Before: validation had to be hidden in a static helper
Employee(String name, int age) {
    super(validateAndGetName(name, age));
}
private static String validateAndGetName(String name, int age) {
    if (age < 18) throw new IllegalArgumentException("Must be an adult");
    return name;
}
```

### 3. Compact Source Files and Instance Main Methods

**Description:** A beginner can write a program with just a `main` method. No `class`, no `public static`, no `String[] args` are needed. The new `IO` class gives simple console input and output.

**Why it matters:** Java becomes much easier to learn and good for small scripts. Real applications still use normal classes.

**Example:**

```java
// Hello.java
void main() {
    IO.println("Hello, Java 25!");
}
```

**Before vs After:**

```java
// Before
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello!");
    }
}

// After
void main() {
    IO.println("Hello!");
}
```

### 4. Module Import Declarations

**Description:** `import module java.base;` imports every public class from all the packages that module exports.

**Why it matters:** Quick scripts and teaching code don't need a long import list.

**Example:**

```java
import module java.base;

void main() {
    List<String> names = List.of("a", "b");                 // java.util
    Map<String, Integer> m = Map.of("x", 1);                // java.util
    String joined = names.stream().collect(Collectors.joining(","));  // java.util.stream
    IO.println(joined);
}
```

**Before vs After:**

```java
// Before
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

// After
import module java.base;
```

### 5. Compact Object Headers

**Description:** Every Java object has a small hidden header. This feature shrinks it from 12 bytes to 8 bytes on 64-bit JVMs.

**Why it matters:** Applications with many small objects use less heap and can run faster, often with no code change. Enable it with a flag:

```text
-XX:+UseCompactObjectHeaders
```

> It is a product feature in Java 25 but **off by default**. Test your application before turning it on in production.

### 6. Other Final Features Worth Knowing

- **Generational Shenandoah:** a generational mode for the Shenandoah low-pause collector, enabled with `-XX:+UseShenandoahGC -XX:ShenandoahGCMode=generational`.
- **Key Derivation Function (KDF) API:** a standard API for algorithms such as HKDF, used in modern cryptography.

```java
KDF hkdf = KDF.getInstance("HKDF-SHA256");
AlgorithmParameterSpec params = HKDFParameterSpec.ofExtract()
    .addIKM(inputKeyMaterial)
    .addSalt(salt)
    .thenExpand(info, 32);
SecretKey key = hkdf.deriveKey("AES", params);
```

- **Faster startup and warm-up (AOT):** easier command-line setup for the AOT cache and AOT method profiling, which help Spring Boot startup.
- **JFR improvements:** better, more accurate CPU profiling in Java Flight Recorder.
- **32-bit x86 support removed.**

### 7. Still in Preview or Incubator in Java 25

| Feature | Status | What it does |
|---|---|---|
| Structured Concurrency | Preview (5th) | Run related tasks as one unit that succeeds, fails, or cancels together |
| Primitive Types in Patterns | Preview (3rd) | Allow `int`, `long`, and so on in `instanceof` and `switch` patterns |
| Stable Values | Preview | Lazily initialized values that the JVM treats as constants |
| PEM Encodings of Cryptographic Objects | Preview | Read and write PEM-format keys and certificates |
| Vector API | Incubator (10th) | SIMD math for data-parallel computation |

Structured Concurrency example (preview; the API is still changing):

```java
// Requires: java --enable-preview
try (var scope = StructuredTaskScope.open()) {
    var user   = scope.fork(() -> loadUser());
    var orders = scope.fork(() -> loadOrders());
    scope.join();                                   // wait for both
    return new Dashboard(user.get(), orders.get()); // if one failed, join() throws
}
```

---

# Summary Table

## Release Overview

| Version | Released | LTS | Headline Features |
|---|---|---|---|
| **8** | Mar 2014 | Yes | Lambdas, Streams, Optional, `java.time`, default methods, CompletableFuture |
| **9** | Sep 2017 | No | Module system, JShell, `List.of`/`Map.of`, private interface methods, Stream/Optional additions |
| **10** | Mar 2018 | No | `var`, `List.copyOf`, container awareness |
| **11** | Sep 2018 | Yes | `HttpClient`, new `String` methods, `Files.readString`, `var` in lambdas, Java EE modules removed |
| **12** | Mar 2019 | No | `Collectors.teeing`, compact number format, switch expressions (preview) |
| **13** | Sep 2019 | No | Text blocks (preview), switch `yield` (preview), dynamic CDS |
| **14** | Mar 2020 | No | Switch expressions (final), helpful NPE messages, records and `instanceof` patterns (preview) |
| **15** | Sep 2020 | No | Text blocks (final), `String.formatted`, sealed classes (preview), ZGC/Shenandoah production |
| **16** | Mar 2021 | No | Records (final), pattern matching for `instanceof` (final), `Stream.toList()` |
| **17** | Sep 2021 | Yes | Sealed classes (final), strong JDK encapsulation, `RandomGenerator`, Spring Boot 3 baseline |
| **18** | Mar 2022 | No | UTF-8 by default, Simple Web Server, `@snippet` |
| **19** | Sep 2022 | No | Virtual threads, record patterns (previews) |
| **20** | Mar 2023 | No | Scoped values (incubator), second previews |
| **21** | Sep 2023 | Yes | Virtual threads, pattern matching for `switch`, record patterns, sequenced collections, generational ZGC |
| **22** | Mar 2024 | No | Unnamed variables `_`, Foreign Function and Memory API, multi-file source launch |
| **23** | Sep 2024 | No | Markdown Javadoc, generational ZGC default; String Templates removed |
| **24** | Mar 2025 | No | Stream Gatherers, no virtual-thread pinning, Class-File API, AOT class loading, post-quantum crypto |
| **25** | Sep 2025 | Yes | Scoped values, flexible constructor bodies, compact source files, module imports, compact object headers |

## Language Feature Timeline (Preview to Final)

| Feature | First Preview | Final | Why Developers Care |
|---|---|---|---|
| Lambdas and Streams | n/a | 8 | Functional-style collections processing |
| `var` | n/a | 10 | Less boilerplate |
| Switch expressions | 12 | 14 | Safer, shorter `switch` |
| Text blocks | 13 | 15 | Readable JSON/SQL/HTML strings |
| Records | 14 | 16 | Boilerplate-free DTOs |
| Pattern matching for `instanceof` | 14 | 16 | No manual casts |
| Sealed classes | 15 | 17 | Closed type hierarchies |
| Pattern matching for `switch` | 17 | 21 | Type-based branching |
| Record patterns | 19 | 21 | Deconstruct data in one step |
| Virtual threads | 19 | 21 | Scalable blocking code |
| Sequenced collections | n/a | 21 | `getFirst()`, `getLast()`, `reversed()` |
| Unnamed variables and patterns | 21 | 22 | Mark unused values with `_` |
| Foreign Function and Memory API | 19 | 22 | Modern JNI replacement |
| Stream Gatherers | 22 | 24 | Custom stream operations |
| Class-File API | 22 | 24 | Bytecode tooling without ASM |
| Scoped values | 20 (incubator) | 25 | Safer `ThreadLocal` replacement |
| Flexible constructor bodies | 22 | 25 | Validate before `super(...)` |
| Compact source files | 21 | 25 | Easier learning and scripts |
| Module import declarations | 23 | 25 | Fewer import lines |
| Structured concurrency | 19 (incubator) | Still preview in 25 | Task groups that cancel together |
| Primitive types in patterns | 23 | Still preview in 25 | Primitives in `switch` and `instanceof` |

## Which Version Should You Use?

| Situation | Recommendation |
|---|---|
| Legacy app on Java 8 | Plan a move to **17 or 21**. Check removed Java EE modules (Java 11) and strong encapsulation (Java 17). |
| New Spring Boot 3.x project | **Java 17 minimum**. **Java 21 or 25** is better, especially for virtual threads and pattern matching. |
| High-concurrency, I/O-heavy service | **Java 21+** for virtual threads. **Java 24+** avoids `synchronized` pinning. |
| Want the newest LTS | **Java 25** (check your framework's supported-version list first). |
| Experimenting with new ideas | Any non-LTS release, using `--enable-preview` only in non-production code. |

> Always check the "System Requirements" page of your Spring Boot version for the exact supported Java range before upgrading.
