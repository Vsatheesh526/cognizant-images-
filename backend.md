# AEM Backend Development Guide

A comprehensive guide covering core AEM backend concepts with detailed code examples and easy-to-understand explanations.

---

## Table of Contents
1. [Filters in AEM](#filters-in-aem)
2. [Listeners in AEM](#listeners-in-aem)
3. [Validators in AEM](#validators-in-aem)
4. [Conditional Section Display](#conditional-section-display)
5. [Sling Models](#sling-models)
6. [Servlets](#servlets)
7. [Schedulers](#schedulers)

---

## Filters in AEM

### What are Filters?

Filters in AEM are **Sling-based components** that intercept and process HTTP requests and responses. They act like a middleman between the user and your application.

**Simple Analogy**: Like a security guard at a building entrance who checks everyone before letting them in.

### Purpose of Filters

✅ Modify incoming requests  
✅ Modify outgoing responses  
✅ Add authentication / authorization logic  
✅ Perform logging, security checks, redirects  
✅ Control request processing flow

### Types of Filters

| Type | When Executed | Use Case |
|------|---------------|----------|
| **REQUEST** | Before request processing | Authentication, validation |
| **COMPONENT** | During component rendering | Data processing during render |
| **ERROR** | When error occurs | Error handling, logging |

### Filter Lifecycle Methods

#### 1. **init()** - Initialization
Called **once** when the filter is first loaded.

```java
@Override
public void init(FilterConfig config) throws ServletException {
    // Load configuration
    // Initialize resources
    // Set up database connections
    System.out.println("Filter initialized");
}
```

**Use Cases:**
- Load configuration from files
- Initialize database connections
- Set up logging

**Interview Answer**: *"init() is used to initialize the filter when it is first loaded. It's called only once and is used for setup activities."*

---

#### 2. **doFilter()** - Core Logic
The **main method** where request/response processing happens. Called for **every request**.

```java
@Override
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) 
        throws IOException, ServletException {
    
    HttpServletRequest httpRequest = (HttpServletRequest) request;
    HttpServletResponse httpResponse = (HttpServletResponse) response;
    
    // BEFORE request processing
    System.out.println("Request received: " + httpRequest.getRequestURI());
    
    // Add custom header to request
    httpRequest.setAttribute("customUser", "Admin");
    
    // Continue the filter chain (important!)
    chain.doFilter(request, response);
    
    // AFTER request processing (response modification)
    System.out.println("Response status: " + httpResponse.getStatus());
}
```

**What happens here:**
1. Receive the request
2. Do some logic (validation, logging, etc.)
3. Call `chain.doFilter()` to pass to next filter
4. Modify response if needed

**Interview Answer**: *"doFilter() contains the core logic and is executed for every request passing through the filter. It's where we intercept and modify requests/responses."*

---

#### 3. **destroy()** - Cleanup
Called **once** when the filter is removed or server stops.

```java
@Override
public void destroy() {
    // Close database connections
    // Release resources
    // Stop any background processes
    System.out.println("Filter destroyed, cleanup complete");
}
```

**Use Cases:**
- Close database connections
- Release file handles
- Stop scheduled tasks

**Interview Answer**: *"destroy() is used to clean up resources when the filter is taken out of service. It's called when the server shuts down or the filter is undeployed."*

---

### Complete Filter Example

```java
import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.api.SlingHttpServletResponse;
import org.apache.sling.engine.EngineConstants;
import org.osgi.service.component.annotations.Component;
import org.osgi.service.component.annotations.Reference;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import java.io.IOException;

@Component(
    service = Filter.class,
    property = {
        EngineConstants.SLING_FILTER_SCOPE + "=" + EngineConstants.REQUEST_SCOPE,
        EngineConstants.SLING_FILTER_ORDER + "=-500"  // Execute early
    }
)
public class AuthenticationFilter implements Filter {
    
    private static final Logger logger = LoggerFactory.getLogger(AuthenticationFilter.class);
    
    @Override
    public void init(FilterConfig config) throws ServletException {
        logger.info("Authentication Filter initialized");
    }
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) 
            throws IOException, ServletException {
        
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        String token = httpRequest.getHeader("Authorization");
        
        // Check if user is authenticated
        if (token != null && validateToken(token)) {
            logger.info("User authenticated successfully");
            chain.doFilter(request, response);
        } else {
            logger.warn("Unauthorized access attempt");
            ((HttpServletResponse) response).sendError(401, "Unauthorized");
        }
    }
    
    @Override
    public void destroy() {
        logger.info("Authentication Filter destroyed");
    }
    
    private boolean validateToken(String token) {
        // Your validation logic here
        return token.startsWith("Bearer ");
    }
}
```

---

## Listeners in AEM

### What are Listeners?

Listeners are components that **observe repository or system events** and trigger logic automatically when changes occur.

**Simple Analogy**: Like a smoke detector that automatically triggers an alarm when smoke is detected.

### How Listeners Work

```
Event happens (e.g., page created)
       ↓
Listener detects it
       ↓
Automatically runs your code
       ↓
Task completes (notify, start workflow, etc.)
```

### Purpose of Listeners

✅ React when content changes  
✅ Run tasks automatically in background  
✅ Reduce manual work  
✅ Keep systems in sync  

### Real-World Examples

| Event | What Happens | Listener Action |
|-------|--------------|-----------------|
| Page Created | New page added | Start workflow, log activity |
| Asset Uploaded | New image uploaded | Generate thumbnail, index |
| Page Modified | Content changed | Clear cache, notify team |
| Page Deleted | Page removed | Archive data, update links |

### Event Listener Implementation

```java
import org.apache.sling.api.resource.observation.ResourceChange;
import org.apache.sling.api.resource.observation.ResourceChangeListener;
import org.osgi.service.component.annotations.Component;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Component(
    service = ResourceChangeListener.class,
    property = {
        ResourceChangeListener.PATHS + "=/content/dam/my-site",
        ResourceChangeListener.CHANGES + "=ADDED,MODIFIED"
    }
)
public class AssetUploadListener implements ResourceChangeListener {
    
    private static final Logger logger = LoggerFactory.getLogger(AssetUploadListener.class);
    
    @Override
    public void onChange(java.util.Collection<ResourceChange> changes) {
        for (ResourceChange change : changes) {
            if (change.getType() == ResourceChange.ChangeType.ADDED) {
                logger.info("New asset uploaded: " + change.getPath());
                // Generate thumbnail
                // Index asset
                // Notify user
            } else if (change.getType() == ResourceChange.ChangeType.MODIFIED) {
                logger.info("Asset modified: " + change.getPath());
                // Clear cache
                // Update references
            }
        }
    }
}
```

---

## Validators in AEM

### What are Validators?

Validators ensure that **data entered by authors in dialog fields is correct** and meets predefined rules.

**Simple Analogy**: Like a form validator on a website that checks if email is valid before submission.

### Purpose of Validators

✅ Check if input is valid  
✅ Prevent wrong or incomplete data  
✅ Improve data quality  
✅ Show error messages if invalid

### Common Validators

| Validator | Purpose | Example |
|-----------|---------|---------|
| Email Validator | Validates email format | test@example.com |
| Number Validator | Validates numeric input | 123, -45.67 |
| Length Validator | Validates string length | Min 5, Max 20 chars |
| Regex Validator | Custom pattern matching | Phone: ^\d{10}$ |

### Validator Example

```java
import com.adobe.granite.ui.components.ds.ValueMapResource;
import com.adobe.granite.validation.spi.ValidationMeta;
import com.adobe.granite.validation.spi.Validator;
import org.apache.sling.api.resource.Resource;
import org.osgi.service.component.annotations.Component;

@Component(
    service = {Validator.class}
)
public class EmailValidator implements Validator {
    
    private static final String VALIDATOR_TYPE = "email";
    private static final String EMAIL_REGEX = "^[A-Za-z0-9+_.-]+@(.+)$";
    
    @Override
    public String getValidatorId() {
        return VALIDATOR_TYPE;
    }
    
    @Override
    public void validate(Object data, ValidationMeta metadata, ValidationReport report) {
        if (data == null) {
            return; // Optional field
        }
        
        String email = data.toString();
        
        if (!email.matches(EMAIL_REGEX)) {
            report.fail("Invalid email format. Please enter a valid email address.");
        }
    }
}
```

### Using Validator in Dialog

```xml
<field
    jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/textfield"
    fieldLabel="Email Address"
    name="./email"
    validation="email">
    <granite:data
        validation="email"
        validationText="Please enter a valid email address"/>
</field>
```

---

## Conditional Section Display

### What is Conditional Section Display?

A feature that **shows or hides dialog fields based on user input** (dropdown selection, checkbox state, etc.).

**Simple Analogy**: Like a form where selecting "Other" reveals a text field to specify what "Other" means.

### How It Works

```
User selects value in dropdown
       ↓
Condition checks the value
       ↓
Show/hide relevant fields
```


---

## Sling Models

### What is a Sling Model?

A **Java class** that maps AEM resources (JCR data) into Java objects to handle business logic.

**Simple Analogy**: Instead of mixing HTML, logic, and data together, Sling Models separate concerns:
- **HTL** = UI Display only
- **Sling Model** = Business logic in Java

### Why Use Sling Models?

✅ Separate business logic from UI  
✅ Replace old JSP/Use-API  
✅ Clean, maintainable code  
✅ Easy integration with HTL  
✅ Testable Java code  

### Key Annotations

| Annotation | Purpose | Example |
|-----------|---------|---------|
| `@Model` | Creates the Sling Model | Marks class as model |
| `@ValueMapValue` | Gets data from JCR property | Get component properties |
| `@Inject` | Auto-injects values | Gets from AEM registry |
| `@SlingObject` | Injects Sling objects | Request, Resource, Resolver |
| `@PostConstruct` | Runs after initialization | Initialize computed fields |
| `@Optional` | Makes field optional | Field not required |

### @Inject vs @ValueMapValue

```java
// @Inject: Generic injection (from anywhere in AEM)
@Inject
private String anyValue;
// Gets value from component properties, configurations, etc.

// @ValueMapValue: Specific to component properties (JCR)
@ValueMapValue
private String componentProperty;
// Gets value only from current component's JCR properties
```

**Analogy:**
- `@Inject` = "Bring me water from anywhere"
- `@ValueMapValue` = "Bring me water from the kitchen only"

```
@Model(adaptables = Resource.class,
    defaultInjectionStrategy = DefaultInjectionStrategy.OPTIONAL
)
```

-` @Model → define model 

-`  adaptables → source of data 

-` OPTIONAL → no error if value missing






### Basic Sling Model Example

```java
import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.api.resource.Resource;
import org.apache.sling.models.annotations.DefaultInjectionStrategy;
import org.apache.sling.models.annotations.Model;
import org.apache.sling.models.annotations.injectorspecific.ValueMapValue;
import org.apache.sling.models.annotations.injectorspecific.SlingObject;
import javax.annotation.PostConstruct;

@Model(
    adaptables = Resource.class,
    defaultInjectionStrategy = DefaultInjectionStrategy.OPTIONAL
)
public class ArticleModel {
    
    // Get from JCR properties
    @ValueMapValue
    private String title;
    
    @ValueMapValue
    private String description;
    
    @ValueMapValue
    private String author;
    
    // Inject Sling objects
    @SlingObject
    private Resource resource;
    
    @SlingObject
    private SlingHttpServletRequest request;
    
    // Initialize computed values
    @PostConstruct
    public void init() {
        // Add logic after all properties are loaded
        if (title != null && title.length() > 50) {
            title = title.substring(0, 50) + "...";
        }
    }
    
    // Getters
    public String getTitle() {
        return title;
    }
    
    public String getDescription() {
        return description;
    }
    
    public String getAuthor() {
        return author;
    }
    
    public String getResourcePath() {
        return resource.getPath();
    }
}
```

### Using Sling Model in HTL

```html
<!-- article.html -->
<sly data-sly-use.article="com.example.models.ArticleModel">
    <div class="article">
        <h1>${article.title}</h1>
        <p>${article.description}</p>
        <p>By: ${article.author}</p>
        <p>Resource Path: ${article.resourcePath}</p>
    </div>
</sly>
```

### Advanced Sling Model with List and Collections

```java
import org.apache.sling.api.resource.Resource;
import org.apache.sling.models.annotations.Model;
import org.apache.sling.models.annotations.injectorspecific.ValueMapValue;
import org.apache.sling.models.annotations.injectorspecific.ChildResource;
import java.util.List;

@Model(adaptables = Resource.class)
public class CarouselModel {
    
    @ValueMapValue
    private String title;
    
    // Get child resources (list of items)
    @ChildResource
    private List<SlideModel> slides;
    
    public String getTitle() {
        return title;
    }
    
    public List<SlideModel> getSlides() {
        return slides;
    }
}

// Model for individual slide
@Model(adaptables = Resource.class)
public class SlideModel {
    
    @ValueMapValue
    private String slideTitle;
    
    @ValueMapValue
    private String imagePath;
    
    public String getSlideTitle() {
        return slideTitle;
    }
    
    public String getImagePath() {
        return imagePath;
    }
}
```


## ✅ @SlingObject

### What is @SlingObject?
`@SlingObject` is used to inject AEM/Sling-specific objects like **request, resource, and resource resolver** into a Sling Model.

👉 It gives access to **AEM internal objects**.

---

### ✅ What It Can Inject

Using `@SlingObject`, you can get:

- ✅ SlingHttpServletRequest → for request details  
- ✅ Resource → current component data  
- ✅ ResourceResolver → to access repository  

---

### ✅ Simple Understanding

👉 `@SlingObject` = Access AEM objects directly inside Sling Model  

---

### ✅ Example

```java
@SlingObject
private SlingHttpServletRequest request;

@SlingObject
private Resource resource;

```





























---

## Servlets

### What is a Servlet?

A component that **handles HTTP requests and sends responses** in AEM.

**Simple Analogy**: When user visits a URL → Servlet processes it → Returns response (JSON, HTML, etc.)

```
User requests URL
       ↓
Servlet intercepts request
       ↓
Processes logic
       ↓
Sends back response
```

### Types of Servlets

| Type | HTTP Methods | Purpose | Data Changes |
|------|--------------|---------|--------------|
| **SlingSafeMethodsServlet** | GET, HEAD | Fetch data (read-only) | ❌ No |
| **SlingAllMethodsServlet** | GET, POST, PUT, DELETE | Create, read, update, delete | ✅ Yes |

### GET Servlet Example (Read-only)

```java
import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.api.SlingHttpServletResponse;
import org.apache.sling.api.servlets.SlingSafeMethodsServlet;
import org.osgi.service.component.annotations.Component;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

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
public class GetArticlesServlet extends SlingSafeMethodsServlet {
    
    private static final Logger logger = LoggerFactory.getLogger(GetArticlesServlet.class);
    
    @Override
    protected void doGet(SlingHttpServletRequest request, SlingHttpServletResponse response) 
            throws IOException {
        
        logger.info("Fetching articles...");
        
        // Get query parameter
        String category = request.getParameter("category");
        
        // Create JSON response
        String jsonResponse = "{"
            + "\"articles\": ["
            + "{\"id\": 1, \"title\": \"Article 1\", \"category\": \"" + category + "\"},"
            + "{\"id\": 2, \"title\": \"Article 2\", \"category\": \"" + category + "\"}"
            + "]"
            + "}";
        
        response.setContentType("application/json");
        response.getWriter().write(jsonResponse);
    }
}
```

**URL to call:** `/bin/myApp/articles.json?category=tech`

---

### POST Servlet Example (Create Data)

```java
import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.api.SlingHttpServletResponse;
import org.apache.sling.api.servlets.SlingAllMethodsServlet;
import org.apache.sling.api.resource.ResourceResolver;
import org.osgi.service.component.annotations.Component;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import javax.servlet.Servlet;
import java.io.IOException;

@Component(
    service = Servlet.class,
    property = {
        "sling.servlet.paths=/bin/myApp/articles",
        "sling.servlet.methods={GET,POST}",
        "sling.servlet.extensions=json"
    }
)
public class ArticleServlet extends SlingAllMethodsServlet {
    
    private static final Logger logger = LoggerFactory.getLogger(ArticleServlet.class);
    
    @Override
    protected void doPost(SlingHttpServletRequest request, SlingHttpServletResponse response) 
            throws IOException {
        
        // Get POST parameters
        String title = request.getParameter("title");
        String author = request.getParameter("author");
        String content = request.getParameter("content");
        
        logger.info("Creating article: " + title);
        
        try {
            // Get resource resolver
            ResourceResolver resolver = request.getResourceResolver();
            
            // Create article in repository
            // (simplified example - use proper JCR API in production)
            
            response.setContentType("application/json");
            response.getWriter().write("{\"status\": \"success\", \"message\": \"Article created\"}");
            
        } catch (Exception e) {
            logger.error("Error creating article", e);
            response.setStatus(500);
            response.getWriter().write("{\"status\": \"error\", \"message\": \"" + e.getMessage() + "\"}");
        }
    }
}
```

---

## Schedulers

### What is a Scheduler?

A component that **runs background tasks automatically at a specific time or interval**.

**Key Features:**
✔️ Doesn't need any user request  
✔️ Works like a cron job  
✔️ Runs automatically on schedule  

**Simple Analogy**: Like an alarm clock that automatically goes off at set times.

### Scheduler Expression Format

```
0    0/5  *    *    *    ?
│    │    │    │    │    │
│    │    │    │    │    └─ Day of week (0=Sunday, 1=Monday, etc.)
│    │    │    │    └────── Month (1-12)
│    │    │    └─────────── Day of month (1-31)
│    │    └──────────────── Hour (0-23)
│    └───────────────────── Minute (0-59)
└────────────────────────── Second (0-59)
```

### Common Scheduler Expressions

| Expression | Meaning |
|-----------|---------|
| `0 0 * * * ?` | Every hour at 0 minutes |
| `0 0/5 * * * ?` | Every 5 minutes |
| `0 0 0 * * ?` | Every day at 12:00 AM |
| `0 0 12 * * ?` | Every day at 12:00 PM (noon) |
| `0 0 0 ? * MON` | Every Monday at 12:00 AM |
| `0 0 0 1 * ?` | First day of month at 12:00 AM |
| `0 0 0 ? * 0` | Every Sunday at 12:00 AM |

### Simple Scheduler Example

```java
import org.osgi.service.component.annotations.Component;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Component(
    service = Runnable.class,
    property = {
        "scheduler.expression=0 0/5 * * * ?",  // Every 5 minutes
        "scheduler.concurrent=false",           // Don't run concurrent instances
        "scheduler.name=DemoScheduler"
    }
)
public class DemoScheduler implements Runnable {
    
    private static final Logger logger = LoggerFactory.getLogger(DemoScheduler.class);
    
    @Override
    public void run() {
        logger.info("Scheduler running at: " + System.currentTimeMillis());
        System.out.println("Background task executed!");
    }
}
```

---

### Advanced Scheduler Example with Logic

```java
import org.osgi.service.component.annotations.Component;
import org.osgi.service.component.annotations.Reference;
import org.apache.sling.api.resource.ResourceResolverFactory;
import org.apache.sling.api.resource.ResourceResolver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.text.SimpleDateFormat;
import java.util.Date;

@Component(
    service = Runnable.class,
    property = {
        "scheduler.expression=0 0 2 * * ?",  // Every day at 2:00 AM
        "scheduler.concurrent=false",
        "scheduler.name=DailyCleanupScheduler"
    }
)
public class DailyCleanupScheduler implements Runnable {
    
    private static final Logger logger = LoggerFactory.getLogger(DailyCleanupScheduler.class);
    
    @Reference
    private ResourceResolverFactory resolverFactory;
    
    @Override
    public void run() {
        logger.info("Starting daily cleanup task");
        
        try {
            // Format time
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            String runTime = sdf.format(new Date());
            
            logger.info("Cleanup started at: " + runTime);
            
            // Get resource resolver
            ResourceResolver resolver = resolverFactory.getAdministrativeResourceResolver(null);
            
            try {
                // Perform cleanup tasks
                cleanupOldAssets(resolver);
                clearCache();
                archiveOldContent(resolver);
                
                logger.info("Cleanup completed successfully");
                
            } finally {
                resolver.close();
            }
            
        } catch (Exception e) {
            logger.error("Error during cleanup", e);
        }
    }
    
    private void cleanupOldAssets(ResourceResolver resolver) {
        logger.info("Cleaning up old assets...");
        // Delete assets older than 30 days
    }
    
    private void clearCache() {
        logger.info("Clearing cache...");
        // Clear application cache
    }
    
    private void archiveOldContent(ResourceResolver resolver) {
        logger.info("Archiving old content...");
        // Move old pages to archive
    }
}
```

---

### Scheduler Properties Explained

```java
@Component(
    service = Runnable.class,
    property = {
        "scheduler.expression=0 0/5 * * * ?",
        // Cron expression for when to run
        
        "scheduler.concurrent=false",
        // false = Run one at a time
        // true = Allow concurrent executions
        
        "scheduler.name=MyScheduler",
        // Name for identification
        
        "scheduler.runOn=LEADER"
        // LEADER = Run only on cluster leader
        // ALL = Run on all instances
    }
)
public class MyScheduler implements Runnable {
    @Override
    public void run() {
        // Your task here
    }
}
```

---

## Quick Reference

### Filter Flow
```
Request → init() → doFilter() → chain.doFilter() → Response → destroy()
```

### Listener Flow
```
Event Occurs → Listener Detects → onChange() Triggered → Task Executes
```

### Scheduler Flow
```
Scheduled Time Reached → run() Method Called → Task Executes
```

### Servlet Flow
```
User Request → doGet()/doPost() → Process Logic → Send Response
```

### Sling Model Flow
```
Resource Data → @Model → @ValueMapValue → @PostConstruct → Ready to Use
```

---

## Best Practices

✅ Use **Sling Models** for business logic  
✅ Use **Filters** for cross-cutting concerns (auth, logging)  
✅ Use **Listeners** for event-driven actions  
✅ Use **Validators** to ensure data quality  
✅ Use **Servlets** for external API endpoints  
✅ Use **Schedulers** for background tasks  
✅ Always **close ResourceResolvers** properly  
✅ Use **@Optional** annotation for optional fields  
✅ Implement proper **error handling**  
✅ Use **logging** instead of System.out.println()  

---

## Summary

| Component | Purpose | When to Use |
|-----------|---------|------------|
| **Filter** | Intercept requests/responses | Authentication, logging, redirects |
| **Listener** | React to events | Content changes, automations |
| **Validator** | Validate input data | Dialog fields, form validation |
| **Conditional Display** | Show/hide fields | Dynamic dialogs based on selection |
| **Sling Model** | Map JCR to Java objects | Business logic, data processing |
| **Servlet** | Handle HTTP requests | APIs, external integrations |
| **Scheduler** | Run background tasks | Cleanup, reports, sync tasks |


✅ Real-Life Example 💡
👉 Think like an alarm clock
•	You set time ⏰
•	It runs automatically
•	No need to press anything
👉 Same in AEM:
•	Schedule time
•	Code runs automatically

---

**Happy Coding with AEM! 🚀**
