# 📘 AEM Servlets – Simple Guide

**Easy-to-Understand Guide to Creating and Using Servlets in AEM**

---

## Table of Contents

1. [What is a Servlet?](#what-is-a-servlet)
2. [Servlet Lifecycle](#servlet-lifecycle)
3. [Required Annotations](#required-annotations)
4. [Registration Methods](#registration-methods)
5. [Sling Servlet Resolver](#sling-servlet-resolver)
6. [Default Servlet](#default-servlet)
7. [Complete Examples](#complete-examples)
8. [Interview Q&A](#interview-qa)
9. [Quick Revision](#quick-revision)

---

## What is a Servlet?

### 🔹 Definition

A **Servlet** is a Java program that handles HTTP requests and sends responses.

**Simple Analogy:**
```
User visits URL
    ↓
Request sent to server
    ↓
Servlet processes request
    ↓
Servlet sends response back
    ↓
User sees page/data
```

---

### ✅ What Servlets Do

```
Servlet:
  ├─ Receives HTTP request (GET, POST, etc.)
  ├─ Processes the request
  ├─ Returns HTTP response
  ├─ Sends JSON, HTML, or data
  └─ Mapped to specific URL path
```

---

### 📝 Real-World Examples

```
Example 1: Article Servlet
  URL: http://localhost:4502/bin/myApp/articles.json
  Request: GET article details
  Response: JSON with article data

Example 2: Search Servlet
  URL: http://localhost:4502/bin/myApp/search
  Request: GET search query
  Response: Search results HTML

Example 3: Form Servlet
  URL: http://localhost:4502/bin/myApp/submitForm
  Request: POST form data
  Response: Confirmation message
```

---

## Servlet Lifecycle

### 🔹 What is Lifecycle?

**Lifecycle** is the journey of a servlet from creation to destruction.

```
Step 1: INITIALIZATION
  ├─ Servlet loaded
  ├─ init() method called ONCE
  └─ Resources initialized

Step 2: SERVICE (Active use)
  ├─ Request comes in
  ├─ doGet() or doPost() called
  ├─ Process request
  ├─ Send response
  └─ Repeat for each request

Step 3: DESTRUCTION
  ├─ Server shutdown
  ├─ destroy() method called ONCE
  └─ Cleanup resources
```

---

### 📊 Complete Lifecycle Diagram

```
┌─────────────────────────────────────┐
│    SERVLET LIFECYCLE                │
├─────────────────────────────────────┤
│                                     │
│  1. INITIALIZATION (Once)           │
│     └─ init()                       │
│        └─ Load servlet              │
│        └─ Initialize resources      │
│        └─ Setup configuration       │
│                                     │
│  2. SERVICE (Many times)            │
│     ├─ Request 1 arrives            │
│     │  └─ doGet() / doPost()        │
│     │  └─ Process                   │
│     │  └─ Response sent             │
│     │                               │
│     ├─ Request 2 arrives            │
│     │  └─ doGet() / doPost()        │
│     │  └─ Process                   │
│     │  └─ Response sent             │
│     │                               │
│     └─ ... (many requests)          │
│                                     │
│  3. DESTRUCTION (Once)              │
│     └─ destroy()                    │
│        └─ Server stops              │
│        └─ Close resources           │
│        └─ Cleanup                   │
│                                     │
└─────────────────────────────────────┘
```

---

### 📝 Code Example: Lifecycle Methods

```java
import org.apache.sling.api.servlets.SlingAllMethodsServlet;
import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.api.SlingHttpServletResponse;
import org.osgi.service.component.annotations.Component;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import javax.servlet.ServletException;
import java.io.IOException;

@Component(service = javax.servlet.Servlet.class)
public class MyLifecycleServlet extends SlingAllMethodsServlet {
    
    private static final Logger logger = LoggerFactory.getLogger(MyLifecycleServlet.class);
    
    // STEP 1: INITIALIZATION (Called once when servlet loads)
    @Override
    public void init() throws ServletException {
        super.init();
        logger.info("INITIALIZATION: Servlet loaded");
        // Initialize resources, connections, configuration
    }
    
    // STEP 2: SERVICE - Handle GET requests
    @Override
    protected void doGet(SlingHttpServletRequest request, 
            SlingHttpServletResponse response) throws IOException {
        logger.info("SERVICE: Handling GET request");
        
        String name = request.getParameter("name");
        response.setContentType("text/plain");
        response.getWriter().write("Hello, " + name);
    }
    
    // STEP 2: SERVICE - Handle POST requests
    @Override
    protected void doPost(SlingHttpServletRequest request, 
            SlingHttpServletResponse response) throws IOException {
        logger.info("SERVICE: Handling POST request");
        
        String data = request.getParameter("data");
        response.setContentType("application/json");
        response.getWriter().write("{\"status\": \"received\"}");
    }
    
    // STEP 3: DESTRUCTION (Called once when server stops)
    @Override
    public void destroy() {
        logger.info("DESTRUCTION: Servlet destroyed");
        // Close connections, release resources
        super.destroy();
    }
}
```

---

### 🎯 Lifecycle Execution Example

```
Scenario: Server starts, users visit, server stops

TIME 0:00 - Server starts
    └─ init() called ONCE
    └─ Log: "Servlet loaded"
    └─ Resources initialized

TIME 0:05 - User 1 visits
    └─ doGet() called
    └─ Request processed
    └─ Response sent

TIME 0:10 - User 2 visits
    └─ doGet() called (again)
    └─ Request processed
    └─ Response sent

TIME 0:15 - User 3 visits
    └─ doPost() called
    └─ Request processed
    └─ Response sent

TIME 0:20 - Server stops
    └─ destroy() called ONCE
    └─ Log: "Servlet destroyed"
    └─ Resources cleaned up

Summary:
  ├─ init() called: 1 time
  ├─ doGet() called: 2 times
  ├─ doPost() called: 1 time
  └─ destroy() called: 1 time
```

---

## Required Annotations

### 🔹 Essential Annotations

To create a servlet in AEM, you need:

```java
@Component              // Register as OSGi component
service = Servlet.class // Type of service
property = {...}        // Configuration (path, methods, etc.)
```

---

### ✅ Common Servlet Annotations

| Annotation | Purpose | Example |
|-----------|---------|---------|
| **@Component** | Register as OSGi component | @Component |
| **service** | What service to register | service = Servlet.class |
| **property** | Configuration settings | property = {"sling.servlet.paths=/bin/..."} |

---

### 📝 Required Properties

```java
@Component(
    service = Servlet.class,
    property = {
        // Path registration (where servlet responds)
        "sling.servlet.paths=/bin/myApp/articles",
        
        // HTTP methods handled
        "sling.servlet.methods=GET",
        
        // File extensions
        "sling.servlet.extensions=json",
        
        // Resource type registration (alternative to paths)
        "sling.servlet.resourceTypes=my:Article",
        
        // Selectors (optional)
        "sling.servlet.selectors=export"
    }
)
public class MyServlet extends SlingAllMethodsServlet { ... }
```

---

### 🎯 Key Servlet Properties

```
sling.servlet.paths:
  └─ Direct URL path
  └─ Example: /bin/myApp/articles
  └─ Accessed directly

sling.servlet.methods:
  ├─ GET (read data)
  ├─ POST (submit data)
  ├─ PUT (update data)
  └─ DELETE (delete data)

sling.servlet.extensions:
  ├─ json
  ├─ html
  ├─ txt
  └─ xml

sling.servlet.resourceTypes:
  └─ Resource type
  └─ Example: my:Article
  └─ Responds to resource type
```

---

## Registration Methods

### 🔹 Method 1: Registration by Path

**Use when:** You want a specific URL path.

```java
@Component(
    service = Servlet.class,
    property = {
        "sling.servlet.paths=/bin/myApp/articles",
        "sling.servlet.methods=GET",
        "sling.servlet.extensions=json"
    }
)
public class ArticleServlet extends SlingAllMethodsServlet {
    
    @Override
    protected void doGet(SlingHttpServletRequest request, 
            SlingHttpServletResponse response) throws IOException {
        
        response.setContentType("application/json");
        response.getWriter().write("{\"articles\": [...]}");
    }
}
```

**How to call:**
```
http://localhost:4502/bin/myApp/articles.json
```

---

### ✅ Advantages of Path Registration

```
✓ Simple and direct
✓ Same path for all resources
✓ Easy to test
✓ Explicit URL
✓ Good for APIs
```

---

### 🔹 Method 2: Registration by Resource Type

**Use when:** You want to respond to specific content types.

```java
@Component(
    service = Servlet.class,
    property = {
        "sling.servlet.resourceTypes=my:Article",
        "sling.servlet.methods=GET",
        "sling.servlet.extensions=json",
        "sling.servlet.selectors=details"
    }
)
public class ArticleDetailsServlet extends SlingAllMethodsServlet {
    
    @Override
    protected void doGet(SlingHttpServletRequest request, 
            SlingHttpServletResponse response) throws IOException {
        
        // Get resource
        Resource resource = request.getResource();
        String articlePath = resource.getPath();
        
        response.setContentType("application/json");
        response.getWriter().write("{\"path\": \"" + articlePath + "\"}");
    }
}
```

**How to call:**
```
URL: /content/my-article.details.json

Where:
  └─ /content/my-article → Resource path
  └─ details → Selector (sling.servlet.selectors)
  └─ json → Extension
```

---

### ✅ Advantages of Resource Type Registration

```
✓ Dynamic path (works for any resource of type)
✓ Resource-specific handling
✓ Multiple resources, one servlet
✓ Flexible URLs
✓ Cleaner architecture
```

---

### 📊 Comparison: Path vs Resource Type

| Aspect | By Path | By Resource Type |
|--------|---------|------------------|
| **URL** | `/bin/myApp/articles` | `/content/article.details.json` |
| **Static/Dynamic** | Static | Dynamic |
| **Use Case** | API endpoints | Content renderers |
| **Flexibility** | Fixed paths | Flexible paths |
| **Resources** | Any | Specific type |

---

### 📝 Example: Both Methods

```java
// METHOD 1: By Path (API)
@Component(
    service = Servlet.class,
    property = {
        "sling.servlet.paths=/api/articles",
        "sling.servlet.methods=GET",
        "sling.servlet.extensions=json"
    }
)
public class ArticleAPIServlet extends SlingAllMethodsServlet {
    @Override
    protected void doGet(SlingHttpServletRequest request, 
            SlingHttpServletResponse response) throws IOException {
        // API logic: Get all articles
        response.setContentType("application/json");
        response.getWriter().write("{\"articles\": [...]}");
    }
}

// METHOD 2: By Resource Type (Renderer)
@Component(
    service = Servlet.class,
    property = {
        "sling.servlet.resourceTypes=my:Article",
        "sling.servlet.methods=GET",
        "sling.servlet.extensions=json"
    }
)
public class ArticleRendererServlet extends SlingAllMethodsServlet {
    @Override
    protected void doGet(SlingHttpServletRequest request, 
            SlingHttpServletResponse response) throws IOException {
        // Renderer logic: Render specific article
        Resource resource = request.getResource();
        response.setContentType("application/json");
        response.getWriter().write("{\"article\": \"...\"}");
    }
}
```

---

## Default Servlet

### 🔹 What is Default Servlet?

A **Default Servlet** handles requests when no specific servlet is found.

```
Request comes in:
    ↓
Sling Servlet Resolver looks for:
    ├─ Exact path match? → Use that servlet
    ├─ Resource type match? → Use that servlet
    ├─ Selector match? → Use that servlet
    └─ Nothing found? → Use Default Servlet
```

---

### ✅ Default Servlet in AEM

```
Default Servlet in AEM:
  ├─ Renders JCR content
  ├─ Handles component rendering
  ├─ Serves static files
  ├─ Fallback for all content
  └─ Always available
```

---

### 📝 When Default Servlet is Used

```
Scenario 1: Visit a page without extension
  URL: /content/my-site/hello
  └─ No specific servlet registered
  └─ Default servlet handles
  └─ Renders page using templates

Scenario 2: Visit component
  URL: /content/my-site/hello/jcr:content/par/image
  └─ No specific servlet registered
  └─ Default servlet renders component
  └─ Shows component output

Scenario 3: Custom servlet registered
  URL: /content/my-site/hello.details.json
  └─ Custom servlet registered for selector
  └─ Custom servlet handles
  └─ Default servlet NOT used
```

---

## Sling Servlet Resolver

### 🔹 What is Sling Servlet Resolver?

**Sling Servlet Resolver** finds the right servlet for a request.

```
Request:
  /content/my-article.details.json

Resolver searches:
  1. Path match? (/bin/articles)
  2. Resource type? (my:Article)
  3. Selector? (details)
  4. Extension? (.json)
  5. Method? (GET)
  6. Nothing? Use default servlet
```

---

### 📊 How Sling Resolver Works

```
User requests:
  /content/my-article.details.json

Step 1: Parse request
  ├─ Path: /content/my-article
  ├─ Selector: details
  ├─ Extension: json
  ├─ Method: GET
  └─ Resource type: my:Article

Step 2: Check for path-registered servlet
  └─ Looking for: /bin/myApp/...
  └─ Not found

Step 3: Check for resource-type servlet
  └─ Looking for: resourceType=my:Article
  └─ Found! ArticleDetailsServlet
  └─ Matches selector: details
  └─ Matches extension: json

Step 4: Execute servlet
  └─ ArticleDetailsServlet.doGet()
  └─ Returns response

Result:
  └─ Correct servlet found and executed! ✓
```

---

### 🎯 Resolution Order (Priority)

```
Sling resolves servlets in this order:

1. EXACT PATH MATCH
   └─ /bin/myApp/articles → ArticleServlet

2. RESOURCE TYPE + SELECTOR + EXTENSION
   └─ Type: my:Article, Selector: details, Ext: json

3. RESOURCE TYPE + EXTENSION
   └─ Type: my:Article, Ext: json

4. RESOURCE TYPE + SELECTOR
   └─ Type: my:Article, Selector: details

5. RESOURCE TYPE
   └─ Type: my:Article

6. DEFAULT SERVLET
   └─ Last resort, always available
```

---

## Complete Examples

### 📌 Example 1: Simple Path-Based Servlet

**What it does:** Returns article list as JSON

```java
import org.apache.sling.api.servlets.SlingAllMethodsServlet;
import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.api.SlingHttpServletResponse;
import org.osgi.service.component.annotations.Component;
import javax.servlet.Servlet;
import java.io.IOException;

@Component(
    service = Servlet.class,
    property = {
        "sling.servlet.paths=/bin/myApp/articles",
        "sling.servlet.methods=GET",
        "sling.servlet.extensions=json"
    }
)
public class ArticleListServlet extends SlingAllMethodsServlet {
    
    @Override
    protected void doGet(SlingHttpServletRequest request, 
            SlingHttpServletResponse response) throws IOException {
        
        // Get query parameter
        String category = request.getParameter("category");
        
        // Create response
        String jsonResponse = "{"
            + "\"articles\": ["
            + "  {\"id\": 1, \"title\": \"Article 1\", \"category\": \"" + category + "\"},"
            + "  {\"id\": 2, \"title\": \"Article 2\", \"category\": \"" + category + "\"}"
            + "]"
            + "}";
        
        // Send response
        response.setContentType("application/json");
        response.getWriter().write(jsonResponse);
    }
}
```

**How to call:**
```
http://localhost:4502/bin/myApp/articles.json?category=tech
```

**Response:**
```json
{
  "articles": [
    {"id": 1, "title": "Article 1", "category": "tech"},
    {"id": 2, "title": "Article 2", "category": "tech"}
  ]
}
```

---

### 📌 Example 2: Resource Type Servlet (Page Renderer)

**What it does:** Renders a page as JSON

```java
import org.apache.sling.api.servlets.SlingAllMethodsServlet;
import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.api.SlingHttpServletResponse;
import org.apache.sling.api.resource.Resource;
import org.osgi.service.component.annotations.Component;
import javax.servlet.Servlet;
import java.io.IOException;

@Component(
    service = Servlet.class,
    property = {
        "sling.servlet.resourceTypes=my:Article",
        "sling.servlet.methods=GET",
        "sling.servlet.extensions=json",
        "sling.servlet.selectors=details"
    }
)
public class ArticleDetailsServlet extends SlingAllMethodsServlet {
    
    @Override
    protected void doGet(SlingHttpServletRequest request, 
            SlingHttpServletResponse response) throws IOException {
        
        // Get the resource
        Resource resource = request.getResource();
        String title = resource.getValueMap().get("title", "Unknown");
        String content = resource.getValueMap().get("content", "");
        
        // Create response
        String jsonResponse = "{"
            + "\"path\": \"" + resource.getPath() + "\","
            + "\"title\": \"" + title + "\","
            + "\"content\": \"" + content + "\""
            + "}";
        
        response.setContentType("application/json");
        response.getWriter().write(jsonResponse);
    }
}
```

**How to call:**
```
Page: /content/my-site/article1 (type: my:Article)
URL: /content/my-site/article1.details.json
```

---

### 📌 Example 3: POST Servlet (Form Handler)

**What it does:** Receives form submission

```java
import org.apache.sling.api.servlets.SlingAllMethodsServlet;
import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.api.SlingHttpServletResponse;
import org.osgi.service.component.annotations.Component;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import javax.servlet.Servlet;
import java.io.IOException;

@Component(
    service = Servlet.class,
    property = {
        "sling.servlet.paths=/bin/myApp/submitForm",
        "sling.servlet.methods=POST",
        "sling.servlet.extensions=json"
    }
)
public class FormSubmitServlet extends SlingAllMethodsServlet {
    
    private static final Logger logger = LoggerFactory.getLogger(FormSubmitServlet.class);
    
    @Override
    protected void doPost(SlingHttpServletRequest request, 
            SlingHttpServletResponse response) throws IOException {
        
        // Get form parameters
        String name = request.getParameter("name");
        String email = request.getParameter("email");
        String message = request.getParameter("message");
        
        // Log the submission
        logger.info("Form submitted: " + name + " - " + email);
        
        // Validate
        if (name == null || email == null) {
            response.setStatus(400);
            response.setContentType("application/json");
            response.getWriter().write("{\"error\": \"Missing fields\"}");
            return;
        }
        
        // Save to database/JCR (not shown)
        // ...
        
        // Send success response
        response.setStatus(200);
        response.setContentType("application/json");
        response.getWriter().write("{\"status\": \"success\", \"message\": \"Thank you!\"}");
    }
}
```

**How to call:**
```
POST http://localhost:4502/bin/myApp/submitForm.json
Body:
  name=John
  email=john@example.com
  message=Hello
```

---

### 📌 Example 4: GET with Resource Access

**What it does:** Reads JCR properties

```java
import org.apache.sling.api.servlets.SlingAllMethodsServlet;
import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.api.SlingHttpServletResponse;
import org.apache.sling.api.resource.Resource;
import org.apache.sling.api.resource.ResourceResolver;
import org.osgi.service.component.annotations.Component;
import javax.servlet.Servlet;
import java.io.IOException;

@Component(
    service = Servlet.class,
    property = {
        "sling.servlet.paths=/bin/myApp/getPage",
        "sling.servlet.methods=GET",
        "sling.servlet.extensions=json"
    }
)
public class GetPageServlet extends SlingAllMethodsServlet {
    
    @Override
    protected void doGet(SlingHttpServletRequest request, 
            SlingHttpServletResponse response) throws IOException {
        
        // Get query parameter (page path)
        String pagePath = request.getParameter("path");
        
        if (pagePath == null) {
            response.setStatus(400);
            response.setContentType("application/json");
            response.getWriter().write("{\"error\": \"path parameter required\"}");
            return;
        }
        
        // Get resource resolver
        ResourceResolver resolver = request.getResourceResolver();
        
        // Get the page
        Resource pageResource = resolver.getResource(pagePath);
        
        if (pageResource == null) {
            response.setStatus(404);
            response.setContentType("application/json");
            response.getWriter().write("{\"error\": \"Page not found\"}");
            return;
        }
        
        // Read properties
        String title = pageResource.getValueMap().get("jcr:title", "");
        String description = pageResource.getValueMap().get("jcr:description", "");
        
        // Send response
        String jsonResponse = "{"
            + "\"path\": \"" + pagePath + "\","
            + "\"title\": \"" + title + "\","
            + "\"description\": \"" + description + "\""
            + "}";
        
        response.setContentType("application/json");
        response.getWriter().write(jsonResponse);
    }
}
```

**How to call:**
```
http://localhost:4502/bin/myApp/getPage.json?path=/content/my-page
```

---

## Interview Q&A

### ❓ Q1: What is a Servlet?

**Answer:**
"A Servlet is a Java program that handles HTTP requests and sends responses.

Key points:
- Receives requests (GET, POST, etc.)
- Processes logic
- Sends back response (JSON, HTML, etc.)
- Mapped to specific URL path or resource type
- Runs on web server

Example: User visits /bin/myApp/articles → Servlet processes → Returns JSON"

---

### ❓ Q2: What is Servlet Lifecycle?

**Answer:**
"Servlet Lifecycle has 3 stages:

1. Initialization (init)
   - Called once when servlet loads
   - Initialize resources
   
2. Service (doGet/doPost)
   - Called many times for requests
   - Handle each request
   - Send response
   
3. Destruction (destroy)
   - Called once when server stops
   - Cleanup resources

Example:
  init() → request1 → request2 → request3 → destroy()"

---

### ❓ Q3: How to register a Servlet in AEM?

**Answer:**
"Using @Component annotation with properties:

Method 1 - By Path:
@Component(service = Servlet.class,
property = {
  'sling.servlet.paths=/bin/myApp/articles',
  'sling.servlet.methods=GET',
  'sling.servlet.extensions=json'
})

Method 2 - By Resource Type:
@Component(service = Servlet.class,
property = {
  'sling.servlet.resourceTypes=my:Article',
  'sling.servlet.methods=GET',
  'sling.servlet.extensions=json'
})"

---

### ❓ Q4: Difference between Path and Resource Type registration?

**Answer:**
"Path Registration:
- URL: /bin/myApp/articles
- Static, fixed path
- Use for APIs
- Same path for all resources

Resource Type Registration:
- URL: /content/article.json
- Dynamic, depends on resource
- Use for content renderers
- Different path per resource"

---

### ❓ Q5: What is Sling Servlet Resolver?

**Answer:**
"Sling Servlet Resolver finds right servlet for request.

Process:
1. Request arrives
2. Parse path, extension, method
3. Check for matching servlet
4. Priority: Path > Type > Selector > Extension
5. If none found, use default servlet
6. Execute servlet

Example: /content/article.details.json
  → Finds servlet for type:my:Article, selector:details"

---

### ❓ Q6: What is Default Servlet?

**Answer:**
"Default Servlet handles requests when no specific servlet found.

Use cases:
- Render pages without extension
- Render components
- Serve static files
- Fallback for all content

Always available, built-in to AEM"

---

### ❓ Q7: How to get request parameters?

**Answer:**
"Using getParameter() method:

String name = request.getParameter('name');
String email = request.getParameter('email');

Multiple values:
String[] colors = request.getParameterValues('color');

Safe approach:
String name = request.getParameter('name');
if (name != null) {
  // Use name
}"

---

### ❓ Q8: How to get request resource?

**Answer:**
"Using getResource() method:

Resource resource = request.getResource();
String path = resource.getPath();

Get properties:
ValueMap props = resource.getValueMap();
String title = props.get('title', 'default');

Get children:
for (Resource child : resource.getChildren()) {
  // Process child
}"

---

### ❓ Q9: How to send JSON response?

**Answer:**
"Steps:

1. Set content type:
   response.setContentType('application/json');

2. Get writer:
   PrintWriter writer = response.getWriter();

3. Write JSON:
   writer.write('{\"name\": \"John\"}');

4. Send:
   writer.flush();

Or use StringBuilder:
StringBuilder json = new StringBuilder();
json.append('{\"status\": \"success\"}');
response.getWriter().write(json.toString());"

---

### ❓ Q10: How to handle errors in servlet?

**Answer:**
"Set HTTP status:

Not found (404):
  response.setStatus(404);
  response.getWriter().write('Not found');

Bad request (400):
  response.setStatus(400);
  response.getWriter().write('Missing parameters');

Server error (500):
  response.setStatus(500);
  response.getWriter().write('Server error');

Or use sendError:
  response.sendError(404, 'Page not found');"

---

## Quick Revision

### ✅ Servlet Basics

```
1. What? Java program handling HTTP requests
2. Purpose? Receive request → Process → Send response
3. Lifecycle? init() → service() → destroy()
4. Register? Use @Component annotation
5. Path? Fixed URL (/bin/myApp/...)
6. Type? Resource type (my:Article)
7. Methods? GET, POST, PUT, DELETE
8. Resolver? Finds right servlet for request
9. Default? Fallback servlet always available
10. Response? JSON, HTML, text, etc.
```

---

### 📝 Servlet Template

```java
@Component(
    service = Servlet.class,
    property = {
        "sling.servlet.paths=/bin/myApp/myServlet",
        "sling.servlet.methods=GET",
        "sling.servlet.extensions=json"
    }
)
public class MyServlet extends SlingAllMethodsServlet {
    
    private static final Logger logger = 
        LoggerFactory.getLogger(MyServlet.class);
    
    @Override
    protected void doGet(SlingHttpServletRequest request, 
            SlingHttpServletResponse response) throws IOException {
        
        // Get parameters
        String param = request.getParameter("param");
        
        // Process
        String result = "processed";
        
        // Send response
        response.setContentType("application/json");
        response.getWriter().write("{\"result\": \"" + result + "\"}");
    }
}
```

---

### 🎯 Memory Tricks

```
SERVLET = Server + Let
        = Server Helper
        = Handles requests

LIFECYCLE = Birth → Life → Death
          = init → service → destroy

PATH = Direct URL (/bin/...)
TYPE = Resource type (my:Article)

RESOLVER = Finds servlet (like GPS)
DEFAULT = Fallback (always available)

doGet = Read data (GET method)
doPost = Submit data (POST method)

@Component = Register servlet (OSGi)
property = Configuration (path, methods)
```

---

### ✅ Checklist: Create Servlet

```
☐ Create class extending SlingAllMethodsServlet
☐ Add @Component annotation
☐ Set service = Servlet.class
☐ Add sling.servlet.paths (or resourceTypes)
☐ Add sling.servlet.methods (GET, POST, etc.)
☐ Add sling.servlet.extensions (json, html, etc.)
☐ Implement doGet() for GET requests
☐ Implement doPost() for POST requests (if needed)
☐ Get request parameters: request.getParameter()
☐ Set response content type: response.setContentType()
☐ Write response: response.getWriter().write()
☐ Build and deploy
☐ Test servlet
☐ Check logs for errors
```

---

## Summary

| Concept | Explanation |
|---------|-------------|
| **Servlet** | Java program handling HTTP requests |
| **Lifecycle** | init() → service() → destroy() |
| **@Component** | Register as OSGi component |
| **Path** | Fixed URL like /bin/myApp |
| **Resource Type** | Respond to content type like my:Article |
| **doGet** | Handle GET requests (read) |
| **doPost** | Handle POST requests (submit) |
| **Resolver** | Finds right servlet for request |
| **Default Servlet** | Fallback, always available |
| **Response** | JSON, HTML, text, etc. |

---

## DO's & DON'Ts

### ✅ DO's

```
✓ Implement init() for setup
✓ Implement destroy() for cleanup
✓ Set correct content type
✓ Validate input parameters
✓ Handle errors properly
✓ Use logging
✓ Use correct HTTP methods
✓ Close resources
✓ Test with actual data
✓ Document servlet purpose
```

### ❌ DON'Ts

```
❌ Don't forget @Component annotation
❌ Don't mix up path and resource type
❌ Don't forget extensions
❌ Don't ignore error handling
❌ Don't skip input validation
❌ Don't leave resources open
❌ Don't hardcode credentials
❌ Don't ignore logging
❌ Don't test on production
❌ Don't forget to deploy
```

---

## Interview Tips

```
1. Explain servlet = handler for HTTP requests
2. Explain lifecycle: 3 stages
3. Know @Component annotation
4. Know path vs resource type difference
5. Know common properties
6. Explain servlet resolver
7. Know default servlet
8. Show code example
9. Explain doGet vs doPost
10. Handle errors in response
```

---

**Last Updated:** November 2024  
**Version:** 1.0  
**Type:** Simple Learning Guide

---

## Key Takeaways

✅ Servlet = Handle HTTP requests  
✅ Lifecycle = init → service → destroy  
✅ @Component = Register servlet  
✅ Path = /bin/myApp/... (fixed URL)  
✅ Type = Resource type (dynamic)  
✅ doGet = Read requests  
✅ doPost = Submit requests  
✅ Resolver = Finds servlet  
✅ Default = Fallback servlet  
✅ Response = JSON/HTML/text  

---

*Perfect for learning servlets from scratch and passing interviews! 🚀*
