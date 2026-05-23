# 📘 JUnit Testing in AEM – Simple Guide

**Easy-to-Understand Unit Testing for AEM Developers**

---

## Table of Contents

1. [Introduction to Unit Testing](#introduction-to-unit-testing)
2. [JUnit Basics](#junit-basics)
3. [Annotations in JUnit](#annotations-in-junit)
4. [Mocks and Mockito](#mocks-and-mockito)
5. [Testing in AEM](#testing-in-aem)
6. [Simple Examples](#simple-examples)
7. [Common Interview Questions](#common-interview-questions)
8. [Quick Revision](#quick-revision)

---

## Introduction to Unit Testing

### 🔹 What is Unit Testing?

**Unit Testing** is testing small pieces of code (functions/methods) in isolation.

**Simple Analogy:**
```
Testing a car:
  ❌ Bad way: Drive the entire car on road (tests everything together)
  ✅ Good way: Test each part separately
      - Test brake pedal alone
      - Test steering wheel alone
      - Test lights alone
      - Then test all together

Same with code!
  ❌ Bad: Test entire application at once
  ✅ Good: Test each method separately
```

---

### ✅ Why Unit Testing?

| Reason | Benefit |
|--------|---------|
| **Find bugs early** | Catch issues before production |
| **Code quality** | Forces you to write better code |
| **Confidence** | Know your code works |
| **Easy refactoring** | Change code without fear |
| **Documentation** | Tests show how to use code |
| **Time-saving** | Prevents manual testing every time |

---

### 📊 Testing Pyramid

```
        /\
       /  \
      / E2E \          Integration Tests
     /  Tests \        (Slow, expensive)
    /________\
    
    /        \
   /  Service \         Unit Tests
  /   Tests    \        (Fast, cheap)
 /____________\
 
 /          \
/Unit Tests  \           Unit Tests
/__________\           (Fast, isolated)
```

---

## JUnit Basics

### 🔹 What is JUnit?

**JUnit** is a framework for writing and running automated tests in Java.

**Think of it as:**
```
JUnit = Testing framework
       = Way to write test code
       = Automation for testing
```

---

### ✅ Basic JUnit Syntax

```java
import org.junit.Before;      // Setup before each test
import org.junit.Test;        // Mark as test method
import org.junit.After;       // Cleanup after each test
import static org.junit.Assert.*; // Assertions

public class CalculatorTest {
    
    private Calculator calculator;  // What we're testing
    
    @Before
    public void setUp() {
        // Run before EACH test
        calculator = new Calculator();
    }
    
    @Test
    public void testAddition() {
        // Actual test
        int result = calculator.add(2, 3);
        
        // Assert (check result)
        assertEquals(5, result);  // Pass if 5 == 5
    }
    
    @Test
    public void testSubtraction() {
        int result = calculator.subtract(5, 3);
        assertEquals(2, result);
    }
    
    @After
    public void tearDown() {
        // Run after EACH test
        calculator = null;
    }
}
```

---

### 🎯 Test Structure (AAA Pattern)

```
@Test
public void testSomething() {
    
    // ARRANGE - Setup
    String input = "Hello";
    String expected = "HELLO";
    
    // ACT - Execute
    String result = input.toUpperCase();
    
    // ASSERT - Verify
    assertEquals(expected, result);
}
```

---

### ✅ Common Assertions

```java
// Test if two values are equal
assertEquals(expected, actual);
assertEquals(5, calculator.add(2, 3));

// Test if condition is true
assertTrue(list.isEmpty());
assertTrue(value > 0);

// Test if condition is false
assertFalse(list.isEmpty());
assertFalse(value < 0);

// Test if object is null
assertNull(obj);

// Test if object is not null
assertNotNull(obj);
```

---

### 📝 Simple Example: Calculator Test

**Code to Test (Calculator.java):**
```java
public class Calculator {
    
    public int add(int a, int b) {
        return a + b;
    }
    
    public int subtract(int a, int b) {
        return a - b;
    }
    
    public int multiply(int a, int b) {
        return a * b;
    }
}
```

**Test Code (CalculatorTest.java):**
```java
import org.junit.Before;
import org.junit.Test;
import static org.junit.Assert.*;

public class CalculatorTest {
    
    private Calculator calc;
    
    @Before
    public void setUp() {
        calc = new Calculator();
    }
    
    @Test
    public void testAdd() {
        int result = calc.add(2, 3);
        assertEquals(5, result);
    }
    
    @Test
    public void testSubtract() {
        int result = calc.subtract(5, 2);
        assertEquals(3, result);
    }
    
    @Test
    public void testMultiply() {
        int result = calc.multiply(4, 5);
        assertEquals(20, result);
    }
    
    @Test
    public void testNegativeNumbers() {
        int result = calc.add(-5, 10);
        assertEquals(5, result);
    }
}
```

**Run Tests:**
```
In IDE: Right-click → Run As → JUnit Test
Result: 4 tests passed ✅
```

---

## Annotations in JUnit

### 🔹 Important Annotations

| Annotation | Purpose | Runs |
|-----------|---------|------|
| **@Before** | Setup before EACH test | Before every @Test |
| **@After** | Cleanup after EACH test | After every @Test |
| **@BeforeClass** | Setup once before ALL tests | Once at start |
| **@AfterClass** | Cleanup once after ALL tests | Once at end |
| **@Test** | Mark as test method | By test runner |
| **@Ignore** | Skip this test | Not run |
| **@Test(expected=...)** | Expect exception | Validate throws |
| **@Test(timeout=...)** | Test must finish in time | Kill if too slow |

---

### 📊 Execution Order

```
@BeforeClass (static)
    ↓
@Before
  ↓
@Test 1
  ↓
@After
    ↓
@Before
  ↓
@Test 2
  ↓
@After
    ↓
@Before
  ↓
@Test 3
  ↓
@After
    ↓
@AfterClass (static)

Result: All tests passed or failed
```

---

### 📝 Example with All Annotations

```java
public class TestLifecycle {
    
    @BeforeClass
    public static void setUpOnce() {
        System.out.println("Setup once for all tests");
    }
    
    @Before
    public void beforeEachTest() {
        System.out.println("Before each test");
    }
    
    @Test
    public void test1() {
        System.out.println("Running test 1");
        assertTrue(true);
    }
    
    @Test
    public void test2() {
        System.out.println("Running test 2");
        assertTrue(true);
    }
    
    @After
    public void afterEachTest() {
        System.out.println("After each test");
    }
    
    @AfterClass
    public static void tearDownOnce() {
        System.out.println("Cleanup once after all tests");
    }
}
```

**Output:**
```
Setup once for all tests
Before each test
Running test 1
After each test
Before each test
Running test 2
After each test
Cleanup once after all tests
```

---

## Mocks and Mockito

### 🔹 What is a Mock?

A **Mock** is a fake object that simulates real behavior for testing.

**Why needed?**
```
Real object problems:
  ❌ Calls database (slow)
  ❌ Calls external API (slow, unreliable)
  ❌ Has side effects
  ❌ Hard to test edge cases

Solution:
  ✅ Use Mock instead
  ✅ Mock acts like real object
  ✅ Returns what we want
  ✅ Fast and predictable
```

---

### ✅ Mockito Framework

**Mockito** is a library to create mocks easily.

```xml
<!-- Add to pom.xml -->
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>3.12.4</version>
    <scope>test</scope>
</dependency>
```

---

### 📝 Simple Mock Example

**Real Code (with database):**
```java
public class UserService {
    
    private UserRepository userRepository; // Database dependency
    
    public UserService(UserRepository repo) {
        this.userRepository = repo;
    }
    
    public User getUserById(int id) {
        return userRepository.findById(id);
    }
    
    public boolean userExists(int id) {
        User user = userRepository.findById(id);
        return user != null;
    }
}
```

**Test Code (with Mock):**
```java
import org.junit.Before;
import org.junit.Test;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;
import static org.junit.Assert.*;
import static org.mockito.Mockito.*;

public class UserServiceTest {
    
    @Mock
    private UserRepository mockRepository; // Fake database
    
    private UserService userService;
    
    @Before
    public void setUp() {
        MockitoAnnotations.initMocks(this);
        userService = new UserService(mockRepository);
    }
    
    @Test
    public void testGetUserById() {
        // ARRANGE - Setup mock
        User fakeUser = new User(1, "John");
        when(mockRepository.findById(1)).thenReturn(fakeUser);
        
        // ACT - Call method
        User result = userService.getUserById(1);
        
        // ASSERT - Verify result
        assertEquals("John", result.getName());
        
        // Verify mock was called
        verify(mockRepository).findById(1);
    }
    
    @Test
    public void testUserExists() {
        // ARRANGE
        User fakeUser = new User(5, "Jane");
        when(mockRepository.findById(5)).thenReturn(fakeUser);
        
        // ACT
        boolean exists = userService.userExists(5);
        
        // ASSERT
        assertTrue(exists);
    }
    
    @Test
    public void testUserNotExists() {
        // ARRANGE
        when(mockRepository.findById(999)).thenReturn(null);
        
        // ACT
        boolean exists = userService.userExists(999);
        
        // ASSERT
        assertFalse(exists);
    }
}
```

---

### 🎯 Mockito Syntax

#### 1. Create Mock
```java
// Option 1: Using annotation
@Mock
private UserRepository mockRepository;

// Option 2: Using static method
UserRepository mockRepository = mock(UserRepository.class);
```

---

#### 2. Setup Mock Behavior (when-then)

```java
// Simple return
when(mockRepository.findById(1)).thenReturn(new User(1, "John"));

// Return null
when(mockRepository.findById(999)).thenReturn(null);

// Throw exception
when(mockRepository.delete(1)).thenThrow(new RuntimeException("DB error"));

// Multiple return values
when(mockRepository.getAll())
    .thenReturn(new ArrayList<>())
    .thenReturn(getUserList());
```

---

#### 3. Verify Calls (verify-times)

```java
// Verify method was called
verify(mockRepository).findById(1);

// Verify called exactly once
verify(mockRepository, times(1)).findById(1);

// Verify called at least once
verify(mockRepository, atLeastOnce()).findById(1);

// Verify never called
verify(mockRepository, never()).delete(1);

// Verify called exactly 3 times
verify(mockRepository, times(3)).save(any());
```

---

### 📊 Real vs Mock Comparison

```
WITHOUT Mock (Bad):
@Test
public void testUserService() {
    UserService service = new UserService(realDatabase);
    User user = service.getUser(1); // Hits real database!
    // Test slow, depends on database, test data
}

WITH Mock (Good):
@Test
public void testUserService() {
    UserRepository mockDb = mock(UserRepository.class);
    when(mockDb.getUser(1)).thenReturn(new User(1, "John"));
    UserService service = new UserService(mockDb);
    User user = service.getUser(1); // Fast, no database!
    // Test fast, isolated, predictable
}
```

---

## Testing in AEM

### 🔹 AEM-Specific Testing

**Challenges in AEM:**
```
❌ Sling API (complex)
❌ JCR Repository (database)
❌ Resource Resolver (needs setup)
❌ Service references (@Reference)
❌ OSGi Components (needs container)
```

**Solution:**
```
✅ Use AEM Test Utils
✅ Mock ResourceResolver
✅ Mock Resource
✅ Use Sling Models Test Utils
```

---

### ✅ Testing a Sling Model

**Real Code (Sling Model):**
```java
@Model(adaptables = Resource.class)
public class ArticleModel {
    
    @ValueMapValue
    private String title;
    
    @ValueMapValue
    private String author;
    
    public String getTitle() {
        return title;
    }
    
    public String getAuthor() {
        return author;
    }
}
```

**Test Code:**
```java
import org.apache.sling.api.resource.Resource;
import org.apache.sling.models.slingmodels.injectors.InjectWith;
import org.apache.sling.testing.mock.sling.ResourceResolverType;
import org.apache.sling.testing.mock.sling.junit.SlingContext;
import org.junit.Rule;
import org.junit.Test;

import static org.junit.Assert.*;

public class ArticleModelTest {
    
    @Rule
    public final SlingContext context = 
        new SlingContext(ResourceResolverType.JCR_MOCK);
    
    @Test
    public void testArticleModel() {
        // ARRANGE - Create test resource with properties
        context.load().json("/com/example/article.json", "/content/article");
        
        Resource resource = context.resourceResolver()
            .getResource("/content/article");
        
        // ACT - Adapt to model
        ArticleModel model = resource.adaptTo(ArticleModel.class);
        
        // ASSERT - Verify
        assertEquals("Test Article", model.getTitle());
        assertEquals("John Doe", model.getAuthor());
    }
    
    @Test
    public void testEmptyArticle() {
        // ARRANGE - Create empty resource
        Resource resource = context.create()
            .resource("/content/empty");
        
        // ACT
        ArticleModel model = resource.adaptTo(ArticleModel.class);
        
        // ASSERT
        assertNull(model.getTitle());
        assertNull(model.getAuthor());
    }
}
```

**Test JSON Data (article.json):**
```json
{
    "jcr:primaryType": "cq:Page",
    "jcr:content": {
        "title": "Test Article",
        "author": "John Doe"
    }
}
```

---

### ✅ Testing a Servlet

**Real Code (Servlet):**
```java
@Component(
    service = Servlet.class,
    property = {
        "sling.servlet.paths=/bin/myApp/articles",
        "sling.servlet.methods=GET"
    }
)
public class ArticleServlet extends SlingAllMethodsServlet {
    
    @Override
    protected void doGet(SlingHttpServletRequest request, 
            SlingHttpServletResponse response) throws IOException {
        
        String articlePath = request.getParameter("path");
        
        if (articlePath == null) {
            response.sendError(400, "Missing path parameter");
            return;
        }
        
        response.setContentType("application/json");
        response.getWriter().write("{\"path\": \"" + articlePath + "\"}");
    }
}
```

**Test Code:**
```java
import org.apache.sling.testing.mock.sling.junit.SlingContext;
import org.junit.Rule;
import org.junit.Test;
import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.api.SlingHttpServletResponse;

import static org.junit.Assert.*;
import static org.mockito.Mockito.*;

public class ArticleServletTest {
    
    @Rule
    public final SlingContext context = 
        new SlingContext(ResourceResolverType.JCR_MOCK);
    
    private ArticleServlet servlet;
    
    @Before
    public void setUp() {
        servlet = new ArticleServlet();
    }
    
    @Test
    public void testGetWithValidPath() throws IOException {
        // ARRANGE
        SlingHttpServletRequest request = mock(SlingHttpServletRequest.class);
        SlingHttpServletResponse response = mock(SlingHttpServletResponse.class);
        
        when(request.getParameter("path")).thenReturn("/content/article");
        
        StringWriter writer = new StringWriter();
        when(response.getWriter()).thenReturn(new PrintWriter(writer));
        
        // ACT
        servlet.doGet(request, response);
        
        // ASSERT
        verify(response).setContentType("application/json");
        assertThat(writer.toString(), containsString("/content/article"));
    }
    
    @Test
    public void testGetWithoutPath() throws IOException {
        // ARRANGE
        SlingHttpServletRequest request = mock(SlingHttpServletRequest.class);
        SlingHttpServletResponse response = mock(SlingHttpServletResponse.class);
        
        when(request.getParameter("path")).thenReturn(null);
        
        // ACT
        servlet.doGet(request, response);
        
        // ASSERT
        verify(response).sendError(400, "Missing path parameter");
    }
}
```

---

## Simple Examples

### 📌 Example 1: Simple String Service

**Code to Test:**
```java
public class StringService {
    
    public String capitalize(String input) {
        if (input == null || input.isEmpty()) {
            return input;
        }
        return input.substring(0, 1).toUpperCase() + input.substring(1);
    }
    
    public int countWords(String sentence) {
        if (sentence == null || sentence.trim().isEmpty()) {
            return 0;
        }
        return sentence.trim().split(" ").length;
    }
}
```

**Test Code:**
```java
import org.junit.Before;
import org.junit.Test;
import static org.junit.Assert.*;

public class StringServiceTest {
    
    private StringService service;
    
    @Before
    public void setUp() {
        service = new StringService();
    }
    
    @Test
    public void testCapitalize() {
        assertEquals("Hello", service.capitalize("hello"));
        assertEquals("W", service.capitalize("w"));
        assertEquals("", service.capitalize(""));
        assertNull(service.capitalize(null));
    }
    
    @Test
    public void testCountWords() {
        assertEquals(3, service.countWords("one two three"));
        assertEquals(1, service.countWords("single"));
        assertEquals(0, service.countWords(""));
        assertEquals(0, service.countWords(null));
    }
}
```

---

### 📌 Example 2: Service with Mock Dependency

**Code to Test:**
```java
public class PaymentService {
    
    private BankAPI bankAPI;
    
    public PaymentService(BankAPI api) {
        this.bankAPI = api;
    }
    
    public boolean processPayment(int amount) {
        if (amount <= 0) {
            return false;
        }
        return bankAPI.transfer(amount);
    }
}
```

**Test Code:**
```java
import org.junit.Before;
import org.junit.Test;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;

import static org.junit.Assert.*;
import static org.mockito.Mockito.*;

public class PaymentServiceTest {
    
    @Mock
    private BankAPI mockBankAPI;
    
    private PaymentService paymentService;
    
    @Before
    public void setUp() {
        MockitoAnnotations.initMocks(this);
        paymentService = new PaymentService(mockBankAPI);
    }
    
    @Test
    public void testSuccessfulPayment() {
        // ARRANGE
        when(mockBankAPI.transfer(100)).thenReturn(true);
        
        // ACT
        boolean result = paymentService.processPayment(100);
        
        // ASSERT
        assertTrue(result);
        verify(mockBankAPI).transfer(100);
    }
    
    @Test
    public void testFailedPayment() {
        // ARRANGE
        when(mockBankAPI.transfer(100)).thenReturn(false);
        
        // ACT
        boolean result = paymentService.processPayment(100);
        
        // ASSERT
        assertFalse(result);
    }
    
    @Test
    public void testInvalidAmount() {
        // ACT
        boolean result = paymentService.processPayment(-50);
        
        // ASSERT
        assertFalse(result);
        verify(mockBankAPI, never()).transfer(any());
    }
}
```

---

### 📌 Example 3: AEM Sling Model

**Code to Test:**
```java
import org.apache.sling.api.resource.Resource;
import org.apache.sling.models.annotations.Model;
import org.apache.sling.models.annotations.injectorspecific.ValueMapValue;

@Model(adaptables = Resource.class)
public class PageModel {
    
    @ValueMapValue
    private String title;
    
    @ValueMapValue
    private String description;
    
    public String getTitle() {
        return title;
    }
    
    public String getDescription() {
        return description;
    }
}
```

**Test Code:**
```java
import org.apache.sling.testing.mock.sling.ResourceResolverType;
import org.apache.sling.testing.mock.sling.junit.SlingContext;
import org.junit.Rule;
import org.junit.Test;

import static org.junit.Assert.*;

public class PageModelTest {
    
    @Rule
    public final SlingContext context = 
        new SlingContext(ResourceResolverType.JCR_MOCK);
    
    @Test
    public void testPageModel() {
        // ARRANGE - Create resource with properties
        context.create().resource("/content/page",
            "title", "My Page",
            "description", "This is my page"
        );
        
        // ACT - Get resource and adapt
        PageModel model = context.resourceResolver()
            .getResource("/content/page")
            .adaptTo(PageModel.class);
        
        // ASSERT
        assertEquals("My Page", model.getTitle());
        assertEquals("This is my page", model.getDescription());
    }
}
```

---

## Common Interview Questions

### ❓ Q1: What is Unit Testing?

**Answer:**
"Unit Testing is testing small pieces of code (methods/functions) in isolation to ensure they work correctly.

Benefits:
- Find bugs early
- Improve code quality
- Make refactoring safe
- Serve as documentation

Example: Test an add() method separately, not the whole calculator app."

---

### ❓ Q2: What's difference between Unit Test and Integration Test?

**Answer:**
"Unit Test: Tests one method in isolation
- Fast
- Uses mocks
- Tests logic only

Integration Test: Tests multiple components together
- Slow
- Uses real objects
- Tests interaction between parts

Example:
Unit Test: Test add() method with mocks
Integration Test: Test calculator + database together"

---

### ❓ Q3: What is a Mock?

**Answer:**
"A Mock is a fake object that simulates real behavior for testing.

Why use?
- Real object might call database (slow)
- Real object might call external API (unreliable)
- Need to test error scenarios

Example:
Real: UserRepository calls actual database
Mock: UserRepository returns pre-set fake data"

---

### ❓ Q4: What is Mockito?

**Answer:**
"Mockito is a framework for creating mocks in Java tests.

Key features:
- Create mocks: @Mock or mock()
- Setup behavior: when-then
- Verify calls: verify()

Example:
when(mockRepository.getUser(1)).thenReturn(user);
verify(mockRepository).getUser(1);"

---

### ❓ Q5: What are common JUnit Annotations?

**Answer:**
"@Before: Run before EACH test (setup)
@After: Run after EACH test (cleanup)
@BeforeClass: Run once before ALL tests
@AfterClass: Run once after ALL tests
@Test: Mark as test method
@Ignore: Skip this test"

---

### ❓ Q6: What is assertEquals?

**Answer:**
"assertEquals is a JUnit method to verify expected vs actual value.

Syntax: assertEquals(expected, actual);

Example:
assertEquals(5, calculator.add(2, 3));
// Passes if 5 == 5"

---

### ❓ Q7: How to test exception throwing?

**Answer:**
"Using @Test annotation with expected parameter:

@Test(expected = IllegalArgumentException.class)
public void testException() {
    service.process(null); // Should throw exception
}"

---

### ❓ Q8: What's difference between verify and assertion?

**Answer:**
"Assertion: Checks the result
assertEquals(5, result);

Verify: Checks if method was called
verify(mockObject).method();"

---

### ❓ Q9: How to test AEM Sling Models?

**Answer:**
"Use AEM testing libraries:

1. Add dependency: aem-core-wcm-components-testing
2. Use SlingContext rule
3. Create test resource
4. Adapt to model
5. Assert properties

Example:
context.create().resource('/content/page', ...);
PageModel model = resource.adaptTo(PageModel.class);
assertEquals('Title', model.getTitle());"

---

### ❓ Q10: When to use @Mock vs Mock?

**Answer:**
"@Mock: Using Mockito annotations
- Cleaner code
- Need MockitoAnnotations.initMocks(this);

Mock: Using static method
- More explicit
- No annotation needed

Both do same thing, just different syntax."

---

## Quick Revision

### ✅ Unit Testing Basics

```
1. What? Test small code pieces in isolation
2. Why? Find bugs, improve quality, confidence
3. Tools? JUnit, Mockito, AssertJ
4. Pattern? AAA (Arrange, Act, Assert)
5. Mock? Fake object for testing
6. Assert? Verify expected result
7. Annotation? @Test, @Before, @After
8. AEM? Use SlingContext for testing
```

---

### 📝 JUnit Test Template

```java
import org.junit.Before;
import org.junit.Test;
import static org.junit.Assert.*;

public class YourServiceTest {
    
    private YourService service;
    
    @Before
    public void setUp() {
        // Setup
        service = new YourService();
    }
    
    @Test
    public void testSomething() {
        // ARRANGE
        String input = "test";
        
        // ACT
        String result = service.doSomething(input);
        
        // ASSERT
        assertEquals("expected", result);
    }
}
```

---

### 📝 Mockito Template

```java
import org.junit.Before;
import org.junit.Test;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;
import static org.mockito.Mockito.*;

public class YourServiceTest {
    
    @Mock
    private Dependency mockDependency;
    
    private YourService service;
    
    @Before
    public void setUp() {
        MockitoAnnotations.initMocks(this);
        service = new YourService(mockDependency);
    }
    
    @Test
    public void testWithMock() {
        // Setup mock behavior
        when(mockDependency.getData()).thenReturn("data");
        
        // Call method
        String result = service.process();
        
        // Verify
        verify(mockDependency).getData();
        assertEquals("data", result);
    }
}
```

---

### 🎯 Memory Tricks

```
AAA = Arrange, Act, Assert
     (Setup, Execute, Verify)

@Before = Before EACH test (setup)
@After = After EACH test (cleanup)

@Test = This is a test

Mock = Fake object
when-then = Setup mock behavior
verify = Check if method called

assertEquals = Are they equal?
assertTrue = Is it true?
assertNull = Is it null?
```

---

## Final Tips

### ✅ DO's

```
✓ Test one thing per test
✓ Use clear test names
✓ Follow AAA pattern
✓ Mock external dependencies
✓ Use assertions to verify
✓ Test edge cases (null, empty, negative)
✓ Keep tests simple and readable
```

### ❌ DON'Ts

```
❌ Don't test implementation details
❌ Don't test too many things in one test
❌ Don't mock everything
❌ Don't ignore test failures
❌ Don't write tests after code
❌ Don't make tests dependent on each other
❌ Don't test private methods
```

---

## Test Coverage Checklist

```
☐ Happy path (normal scenario)
☐ Null input
☐ Empty input
☐ Invalid input
☐ Boundary conditions
☐ Exception cases
☐ Multiple calls
☐ State changes
```

---

## Summary

| Concept | Explanation |
|---------|-------------|
| **Unit Test** | Test small code piece in isolation |
| **JUnit** | Framework for writing tests |
| **Mock** | Fake object for testing |
| **Mockito** | Library to create mocks |
| **@Test** | Mark method as test |
| **@Before** | Run before each test |
| **@After** | Run after each test |
| **assertEquals** | Verify expected == actual |
| **when-then** | Setup mock behavior |
| **verify** | Check if method called |
| **AAA** | Arrange, Act, Assert |
| **Sling Context** | Test Sling Models in AEM |

---

## Quick Command Reference

```bash
# Run all tests
mvn test

# Run specific test
mvn test -Dtest=CalculatorTest

# Run specific test method
mvn test -Dtest=CalculatorTest#testAdd

# Run with coverage
mvn test jacoco:report
```

---

## Interview Tips

```
1. Explain AAA pattern clearly
2. Give real example (add method, user service)
3. Explain why mocks are needed
4. Show difference between @Mock and mock()
5. Know common assertions
6. Mention AEM testing libraries
7. Talk about test coverage
8. Show you test edge cases
```

---

**Last Updated:** November 2024  
**Version:** 1.0  
**Type:** Simple Learning Guide

---

## Key Takeaways

✅ Unit Test = Test small piece in isolation  
✅ JUnit = Framework for tests  
✅ Mock = Fake object for testing  
✅ Mockito = Library to create mocks  
✅ AAA = Arrange, Act, Assert pattern  
✅ @Before = Setup before each test  
✅ @Test = Mark as test  
✅ assertEquals = Verify result  
✅ verify = Check if method called  
✅ when-then = Setup mock behavior  

---

*Perfect for quick revision before interviews and exams. Practice the examples and you're ready!* 🚀
