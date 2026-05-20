# Comprehensive Guide: Service Users, OSGi Logging, Runmodes, and CQ Web Console

---

## Table of Contents
1. [Service User Creation](#service-user-creation)
2. [Logging in OSGi Services](#logging-in-osgi-services)
3. [Configuring Custom Loggers](#configuring-custom-loggers)
4. [Runmode Definition and Use](#runmode-definition-and-use)
5. [CQ Web Console](#cq-web-console)
6. [Practical Example: AEM Sample Project](#practical-example-aem-sample-project)

---

## Service User Creation

### What is a Service User?

A **Service User** is a special system account in AEM used by background processes, OSGi services, workflows, and schedulers to interact with the repository securely. Unlike regular users, service users:
- Are system accounts (not real people)
- Have no passwords
- Cannot log in via the UI
- Follow the principle of least privilege
- Replace deprecated `loginAdministrative()` methods

### Why Service Users Matter

**Security Benefits:**
- ✅ Prevent privilege escalation
- ✅ Grant only minimum required permissions
- ✅ Audit trail of automated actions
- ✅ Comply with security best practices
- ✅ Eliminate admin access risks

**Comparison: Old vs New Approach**

| Aspect | Old Way (Deprecated) | Service Users (Recommended) |
|--------|---------------------|---------------------------|
| Method | `loginAdministrative()` | `getServiceResourceResolver()` |
| Access | Full admin access | Limited permissions only |
| Security | High risk | Secure and auditable |
| Compliance | Non-compliant | Compliant with best practices |
| Auditability | Not traceable to service | Fully auditable |

### How to Create a Service User

#### **Method 1: Using AEM Touch UI**

**Steps:**
1. Log in as administrator
2. Navigate to **Tools → Security → Users**
3. Click **Create** button
4. In "Create User" dialog:
   - **User ID**: Enter unique ID (e.g., `my-service-user`)
   - **Password**: Leave empty (service users don't have passwords)
   - Leave "Create Home Directory" unchecked
5. Click **Create**
6. The service user is now created

#### **Method 2: Using Repository Initializer (Repo Init) - RECOMMENDED**

Repository Initializer allows you to create service users programmatically, ensuring consistency across environments.

**Configuration File:**
```
Path: /apps/myproject/osgiconfig/config/org.apache.sling.jcr.repoinit.RepositoryInitializer~myproject.cfg.json

Content:
{
  "scripts": [
    "create service user my-service-user",
    "set ACL for my-service-user\n  allow jcr:read on /content/myproject\nend"
  ]
}
```

Or in text format (`.config`):
```
Path: /apps/myproject/osgiconfig/config/org.apache.sling.jcr.repoinit.RepositoryInitializer~myproject.config

Content:
scripts=[\
  "create service user my-service-user",\
  "set ACL for my-service-user\n  allow jcr:read on /content/myproject\nend"\
]
```

---

### Service User Mapping Configuration

Once a service user is created, you need to map it to your OSGi service bundle.

#### **Configuration: Service User Mapping**

**File Location:**
```
/apps/myproject/osgiconfig/config/org.apache.sling.serviceusermapping.impl.ServiceUserMapperImpl.amended-myproject.json
```

**Configuration Content:**
```json
{
  "user.mapping": [
    "myproject.bundle:myservice=my-service-user",
    "myproject.bundle:assetsservice=my-assets-user"
  ]
}
```

**Explanation:**
- `myproject.bundle` = Your OSGi bundle symbolic name
- `myservice` = Subservice name (identifier within bundle)
- `my-service-user` = Actual service user created

#### **Alternative Format (.config)**
```
user.mapping=[\
  "myproject.bundle:myservice=my-service-user",\
  "myproject.bundle:assetsservice=my-assets-user"\
]
```

---

### Setting Permissions for Service Users

After creating a service user, you must grant appropriate permissions using ACEs (Access Control Entries).

#### **Using Repo Init to Set Permissions**

**Full Example:**
```
# Create service user
create service user my-content-reader

# Create service user for content writer
create service user my-content-writer

# Set read permissions
set ACL for my-content-reader
  allow jcr:read on /content/myproject
end

# Set read and write permissions
set ACL for my-content-writer
  allow jcr:read on /content/myproject
  allow jcr:modifyProperties on /content/myproject
  allow jcr:addChildNodes on /content/myproject
  allow jcr:removeNode on /content/myproject
end

# Set replication permissions
set ACL for my-replication-user
  allow crx:replicate on /content/myproject
end
```

---

## Logging in OSGi Services

### Why Logging in OSGi Services is Important

Logging is crucial for:
- 📊 Monitoring application behavior
- 🐛 Debugging issues
- 📈 Performance tracking
- 🔍 Security auditing
- 📝 Operational diagnostics

### SLF4J and Apache Sling Logging

AEM uses **SLF4J (Simple Logging Facade for Java)** based on the **Apache Sling Logging Framework** for all logging operations.

**Key Components:**
- **SLF4J**: Logging abstraction layer
- **Log4j/Logback**: Underlying implementation
- **Apache Sling**: Integration layer for AEM

---

### Implementing Logging in OSGi Services

#### **Step 1: Import SLF4J Classes**

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
```

#### **Step 2: Create Logger Instance**

```java
public class MyContentService {
    
    // Create static logger instance
    private static final Logger LOG = LoggerFactory.getLogger(MyContentService.class);
    
    public void processContent() {
        LOG.debug("Debug message");
        LOG.info("Info message");
        LOG.warn("Warning message");
        LOG.error("Error message");
    }
}
```

#### **Step 3: Use Appropriate Log Levels**

| Level | Method | Use Case | Output |
|-------|--------|----------|--------|
| **DEBUG** | `log.debug()` | Detailed info for developers | Most verbose |
| **INFO** | `log.info()` | General application flow | Normal operations |
| **WARN** | `log.warn()` | Potential issues | Warnings |
| **ERROR** | `log.error()` | Serious problems | Errors and exceptions |

---

### Log Levels Example

```java
public class OrderService {
    private static final Logger LOG = LoggerFactory.getLogger(OrderService.class);
    
    public void processOrder(Order order) {
        // Debug: Very detailed, not in production usually
        LOG.debug("Order ID: {}, Customer: {}", order.getId(), order.getCustomerId());
        
        // Info: General flow information
        LOG.info("Processing order {}", order.getId());
        
        // Warn: Something unexpected but not critical
        if (order.getTotal() < 0) {
            LOG.warn("Order {} has negative total: {}", order.getId(), order.getTotal());
        }
        
        // Error: Serious problems
        try {
            validateOrder(order);
        } catch (Exception e) {
            LOG.error("Failed to validate order {}: {}", order.getId(), e.getMessage(), e);
        }
    }
}
```

---

### Where Logs are Stored

**Default Log Locations:**
- `crx-quickstart/launchpad/logs/` - Launchpad logs
- `crx-quickstart/server/logs/` - Server logs
- `crx-quickstart/logs/` - Main AEM logs

**Important Log Files:**
| Log File | Purpose |
|----------|---------|
| `error.log` | System errors, warnings, exceptions |
| `access.log` | HTTP requests and responses |
| `audit.log` | User actions (login, logout, changes) |
| `request.log` | Sling request details |
| `replication.log` | Content replication activities |

---

### Using SLF4J with Parameters

**Best Practice: Use Parameterized Logging**

```java
// ❌ BAD: String concatenation (expensive)
LOG.info("Processing order " + orderId + " for customer " + customerId);

// ✅ GOOD: Parameterized (efficient)
LOG.info("Processing order {} for customer {}", orderId, customerId);

// With exceptions
LOG.error("Failed to save order {}", orderId, exception);
```

---

## Configuring Custom Loggers

### What is a Custom Logger?

A **Custom Logger** allows you to configure logging behavior for specific packages or classes. You can control:
- Log level for specific components
- Output file location
- Log format
- Rotation policies

### Why Configure Custom Loggers?

- 📊 Different log levels for different modules
- 🎯 Separate logs for different services
- 🔍 Detailed debugging for specific components
- 📁 Organize logs by function
- ⚙️ Fine-grained control over logging

---

### Creating Custom Logger Configurations

#### **Method 1: Using CQ Web Console (UI)**

1. Open **CQ Web Console** (http://localhost:4502/system/console)
2. Navigate to **Status → Log Support**
3. Click **Add New Logger**
4. Configure:
   - **Logger Name**: Package path (e.g., `com.mycompany.myproject.services`)
   - **Log Level**: DEBUG, INFO, WARN, ERROR
   - Click **OK**

#### **Method 2: Using OSGi Configuration Files (RECOMMENDED)**

**JSON Format:**
```json
{
  "loggers": [
    {
      "name": "com.mycompany.myproject",
      "level": "DEBUG"
    },
    {
      "name": "com.mycompany.myproject.services",
      "level": "INFO"
    },
    {
      "name": "com.mycompany.myproject.workflows",
      "level": "WARN"
    }
  ]
}
```

**File Location:**
```
/apps/myproject/osgiconfig/config/org.apache.sling.commons.log.LogManager.factory.config-myproject.cfg.json
```

#### **Method 3: Apache Sling Logger Configuration**

**For Dedicated Log File:**

```json
{
  "org.apache.sling.commons.log.names": [
    "com.mycompany.myproject"
  ],
  "org.apache.sling.commons.log.level": "INFO",
  "org.apache.sling.commons.log.file": "logs/myproject.log",
  "org.apache.sling.commons.log.file.size": "20",
  "org.apache.sling.commons.log.file.number": "5"
}
```

**Configuration Explanation:**
- `org.apache.sling.commons.log.names` - Package(s) to log
- `org.apache.sling.commons.log.level` - Log level
- `org.apache.sling.commons.log.file` - Output file location
- `org.apache.sling.commons.log.file.size` - Max file size (MB)
- `org.apache.sling.commons.log.file.number` - Number of backup files

---

### Custom Logger Configuration Example

**Scenario:** Create separate logs for different modules

```json
// File: org.apache.sling.commons.log.LogManager.factory.config-authoring.cfg.json
{
  "org.apache.sling.commons.log.names": ["com.mycompany.myproject.authoring"],
  "org.apache.sling.commons.log.level": "DEBUG",
  "org.apache.sling.commons.log.file": "logs/authoring.log",
  "org.apache.sling.commons.log.file.size": "50",
  "org.apache.sling.commons.log.file.number": "5"
}

// File: org.apache.sling.commons.log.LogManager.factory.config-publishing.cfg.json
{
  "org.apache.sling.commons.log.names": ["com.mycompany.myproject.publishing"],
  "org.apache.sling.commons.log.level": "INFO",
  "org.apache.sling.commons.log.file": "logs/publishing.log",
  "org.apache.sling.commons.log.file.size": "50",
  "org.apache.sling.commons.log.file.number": "5"
}

// File: org.apache.sling.commons.log.LogManager.factory.config-api.cfg.json
{
  "org.apache.sling.commons.log.names": ["com.mycompany.myproject.api"],
  "org.apache.sling.commons.log.level": "WARN",
  "org.apache.sling.commons.log.file": "logs/api.log",
  "org.apache.sling.commons.log.file.size": "100",
  "org.apache.sling.commons.log.file.number": "10"
}
```

**Result:** Three separate log files created:
- `logs/authoring.log` - DEBUG level
- `logs/publishing.log` - INFO level
- `logs/api.log` - WARN level

---

## Runmode Definition and Use

### What is a Runmode?

A **Runmode** is an identifier that specifies the environment or configuration context in which AEM is running. It allows you to deploy different configurations for different environments without changing code.

### Why Use Runmodes?

| Benefit | Example |
|---------|---------|
| **Environment Differentiation** | author, publish, production, development |
| **Configuration Management** | Different DB connections per environment |
| **Conditional Behavior** | Different logging levels per environment |
| **Deployment Flexibility** | Single artifact, multiple deployments |
| **Security** | Different security settings per environment |

---

### Default Runmodes in AEM

| Runmode | Purpose | Typical Use |
|---------|---------|-------------|
| **author** | Author instance | Content creation and management |
| **publish** | Publish instance | Content delivery |
| **dev** | Development mode | Development environment |
| **stage** | Staging mode | Pre-production testing |
| **prod** | Production mode | Live environment |

**Combined Runmodes Example:**
- `author,prod` - Production author instance
- `publish,dev` - Development publish instance
- `publish,prod` - Production publish instance

---

### How to Set Runmodes

#### **Method 1: Using Start Script**

```bash
# Start with specific runmode
./crx-quickstart/bin/start -r author

# Multiple runmodes
./crx-quickstart/bin/start -r author,prod

# Using JVM parameter
java -Dsling.run.modes=author,prod -jar cq-quickstart.jar
```

#### **Method 2: Using Properties File**

**File:** `sling.properties` in quickstart directory

```properties
sling.run.modes=author,prod
```

#### **Method 3: Environment Variable**

```bash
export SLING_RUN_MODES=author,prod
./crx-quickstart/bin/start
```

---

### Using Runmodes in Code

#### **Checking Runmode in OSGi Service**

```java
import org.apache.sling.api.SlingSettingsService;
import org.osgi.service.component.annotations.Component;
import org.osgi.service.component.annotations.Reference;

@Component(service = MyService.class)
public class MyService {
    
    @Reference
    private SlingSettingsService slingSettingsService;
    
    public void init() {
        String[] runModes = slingSettingsService.getRunModes();
        
        // Check if specific runmode is active
        boolean isAuthor = slingSettingsService.getRunModes().contains("author");
        boolean isProduction = slingSettingsService.getRunModes().contains("prod");
        
        if (isAuthor) {
            // Author-specific initialization
        }
        
        if (isProduction) {
            // Production-specific initialization
        }
    }
}
```

#### **Conditional OSGi Configuration Using Runmodes**

Different configurations for different runmodes using folder structure:

```
/apps/myproject/osgiconfig/
├── config/                          # Default configuration
│   └── my.service.Component.cfg.json
├── config.author/                   # Author-specific
│   └── my.service.Component.cfg.json
├── config.publish/                  # Publish-specific
│   └── my.service.Component.cfg.json
└── config.prod/                     # Production-specific
    └── my.service.Component.cfg.json
```

**File Structure Example:**

```
config/my.service.Component.cfg.json (DEV defaults)
{
  "cache.size": "100",
  "log.level": "DEBUG"
}

config.prod/my.service.Component.cfg.json (PROD overrides)
{
  "cache.size": "1000",
  "log.level": "INFO"
}
```

**Behavior:**
- On DEV: Uses `config/` settings
- On PROD: Merges `config/` + overrides with `config.prod/`

---

### Runmode-Based Configuration Example

**Scenario:** Different database connections for different environments

```
/apps/myproject/osgiconfig/
├── config/
│   └── com.mycompany.db.DataSourceFactory.cfg.json
├── config.dev/
│   └── com.mycompany.db.DataSourceFactory.cfg.json
├── config.stage/
│   └── com.mycompany.db.DataSourceFactory.cfg.json
└── config.prod/
    └── com.mycompany.db.DataSourceFactory.cfg.json
```

**config/com.mycompany.db.DataSourceFactory.cfg.json** (Default)
```json
{
  "jdbc.url": "jdbc:mysql://localhost:3306/aem_dev",
  "jdbc.username": "devuser",
  "jdbc.password": "devpass",
  "connection.pool.size": "10"
}
```

**config.prod/com.mycompany.db.DataSourceFactory.cfg.json** (Production Override)
```json
{
  "jdbc.url": "jdbc:mysql://prod-db-server:3306/aem_prod",
  "jdbc.username": "produser",
  "jdbc.password": "${env:DB_PASSWORD}",
  "connection.pool.size": "100"
}
```

---

## CQ Web Console

### What is CQ Web Console?

The **CQ Web Console** (also called **Felix Console**) is a web-based administration interface for managing OSGi bundles, configurations, and system settings in AEM.

### Accessing CQ Web Console

**URL:** 
```
http://localhost:4502/system/console
http://localhost:4502/system/console/bundles
http://localhost:4502/system/console/configMgr
```

**Default Credentials:**
- Username: `admin`
- Password: `admin` (default, change in production)

---

### Main Sections of CQ Web Console

#### **1. Bundles**
- **URL:** `/system/console/bundles`
- **Purpose:** View and manage OSGi bundles
- **Functions:**
  - Start/Stop bundles
  - View bundle details and dependencies
  - See bundle status

#### **2. Configuration Manager**
- **URL:** `/system/console/configMgr`
- **Purpose:** View and edit OSGi configurations
- **Functions:**
  - Create new configurations
  - Edit existing configurations
  - Search configurations
  - Export/Import configurations

#### **3. Log Support**
- **URL:** `/system/console/slinglog`
- **Purpose:** Configure loggers
- **Functions:**
  - Add custom loggers
  - Set log levels per package
  - View log files

#### **4. System Information**
- **URL:** `/system/console/status`
- **Purpose:** View system and environment info
- **Functions:**
  - Memory usage
  - JVM properties
  - Thread information
  - System properties

#### **5. Installed Bundles**
- **URL:** `/system/console/bundles`
- **Purpose:** Manage installed bundles
- **Functions:**
  - Install new bundles
  - Update bundles
  - Remove bundles

#### **6. OSGi Installer**
- **URL:** `/system/console/osgi-installer`
- **Purpose:** Manage artifact installation
- **Functions:**
  - View installed artifacts
  - Upload new artifacts
  - Track installation status

---

### Working with Configuration Manager

#### **Viewing Configurations**

1. Open `/system/console/configMgr`
2. Use search to find configurations
3. Click on configuration to edit
4. View and modify properties
5. Click **Save** to apply changes

#### **Common Configurations to Manage**

| Configuration | Purpose | Location |
|---------------|---------|----------|
| Apache Sling Logging | Logger configuration | Search "Logger" |
| Day CQ Mail Service | Email configuration | Search "Mail" |
| Apache Sling Servlet Resolver | Servlet mapping | Search "Servlet Resolver" |
| CQ Link Externalizer | URL rewriting | Search "Externalizer" |
| Sling Service User Mapper | Service user mapping | Search "Service User Mapper" |

---

### Creating Custom OSGi Configuration via Web Console

**Steps:**

1. Go to **Configuration Manager** (`/system/console/configMgr`)
2. Search for the service/configuration class name
3. If not found, click **Create New Configuration**
4. Enter the configuration class name or PID
5. Fill in the properties
6. Click **Save**

**Example: Creating Custom Logger**

1. Go to Configuration Manager
2. Search "LogManager"
3. Click on `org.apache.sling.commons.log.LogManager`
4. Click **Create New Configuration** (usually there's a + button)
5. Set properties:
   - Names: `com.mycompany.myproject`
   - Level: `DEBUG`
   - File: `logs/myproject.log`
6. Click **Save**

---

### Exporting and Importing Configurations

#### **Export Configuration**

1. Go to Configuration Manager
2. Find your configuration
3. Click **Export** button
4. Copy the JSON/text content
5. Save to version control

#### **Import Configuration**

1. Create OSGi config file in project
2. Deploy via code repository
3. Or use Configuration Manager to paste content

---

## Practical Example: AEM Sample Project

### Project Structure

```
my-aem-project/
├── core/
│   ├── src/main/java/
│   │   └── com/mycompany/myproject/
│   │       ├── services/
│   │       │   ├── OrderService.java
│   │       │   └── ContentReaderService.java
│   │       ├── servlets/
│   │       │   └── OrderProcessingServlet.java
│   │       └── Constants.java
│   └── pom.xml
├── ui.apps/
│   ├── src/main/content/jcr_root/
│   │   └── apps/myproject/
│   │       └── osgiconfig/
│   │           ├── config/
│   │           │   ├── com.mycompany.myproject.services.cfg.json
│   │           │   └── org.apache.sling.commons.log.LogManager.factory.config.cfg.json
│   │           ├── config.author/
│   │           │   └── org.apache.sling.commons.log.LogManager.factory.config.cfg.json
│   │           └── config.prod/
│   │               └── org.apache.sling.commons.log.LogManager.factory.config.cfg.json
│   └── pom.xml
└── pom.xml
```

---

### Complete Practical Example

#### **Step 1: Create Service User (Repo Init)**

**File:** `ui.apps/src/main/content/jcr_root/apps/myproject/osgiconfig/config/org.apache.sling.jcr.repoinit.RepositoryInitializer~myproject.cfg.json`

```json
{
  "scripts": [
    "create service user myproject-content-reader",
    "create service user myproject-content-writer",
    "set ACL for myproject-content-reader\n  allow jcr:read on /content/myproject\nend",
    "set ACL for myproject-content-writer\n  allow jcr:read on /content/myproject\n  allow jcr:modifyProperties on /content/myproject\n  allow jcr:addChildNodes on /content/myproject\nend"
  ]
}
```

#### **Step 2: Configure Service User Mapping**

**File:** `ui.apps/src/main/content/jcr_root/apps/myproject/osgiconfig/config/org.apache.sling.serviceusermapping.impl.ServiceUserMapperImpl.amended-myproject.cfg.json`

```json
{
  "user.mapping": [
    "com.mycompany.myproject.core:contentreader=myproject-content-reader",
    "com.mycompany.myproject.core:contentwriter=myproject-content-writer"
  ]
}
```

#### **Step 3: Configure Custom Loggers**

**File:** `ui.apps/src/main/content/jcr_root/apps/myproject/osgiconfig/config/org.apache.sling.commons.log.LogManager.factory.config-myproject.cfg.json`

```json
{
  "org.apache.sling.commons.log.names": ["com.mycompany.myproject"],
  "org.apache.sling.commons.log.level": "INFO",
  "org.apache.sling.commons.log.file": "logs/myproject.log",
  "org.apache.sling.commons.log.file.size": "20",
  "org.apache.sling.commons.log.file.number": "5"
}
```

**File for Production Override:** `ui.apps/src/main/content/jcr_root/apps/myproject/osgiconfig/config.prod/org.apache.sling.commons.log.LogManager.factory.config-myproject.cfg.json`

```json
{
  "org.apache.sling.commons.log.names": ["com.mycompany.myproject"],
  "org.apache.sling.commons.log.level": "WARN",
  "org.apache.sling.commons.log.file": "logs/myproject.log",
  "org.apache.sling.commons.log.file.size": "50",
  "org.apache.sling.commons.log.file.number": "10"
}
```

#### **Step 4: Create OSGi Service with Service User**

**File:** `core/src/main/java/com/mycompany/myproject/services/ContentReaderService.java`

```java
package com.mycompany.myproject.services;

import org.apache.sling.api.resource.ResourceResolver;
import org.apache.sling.api.resource.ResourceResolverFactory;
import org.osgi.service.component.annotations.Activate;
import org.osgi.service.component.annotations.Component;
import org.osgi.service.component.annotations.Reference;
import org.osgi.service.component.annotations.Deactivate;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.util.HashMap;
import java.util.Map;

@Component(service = ContentReaderService.class)
public class ContentReaderService {
    
    private static final Logger LOG = LoggerFactory.getLogger(ContentReaderService.class);
    private static final String SERVICE_USER = "myproject-content-reader";
    private static final String SUBSERVICE_NAME = "contentreader";
    
    @Reference
    private ResourceResolverFactory resolverFactory;
    
    @Activate
    protected void activate() {
        LOG.info("ContentReaderService activated");
    }
    
    @Deactivate
    protected void deactivate() {
        LOG.info("ContentReaderService deactivated");
    }
    
    /**
     * Get resource using service user credentials
     */
    public String readContent(String path) {
        LOG.debug("Reading content from path: {}", path);
        
        ResourceResolver resolver = null;
        try {
            // Get resource resolver with service user permissions
            Map<String, Object> authInfo = new HashMap<>();
            authInfo.put(ResourceResolverFactory.SUBSERVICE, SUBSERVICE_NAME);
            resolver = resolverFactory.getServiceResourceResolver(authInfo);
            
            // Read content
            if (resolver.getResource(path) != null) {
                String content = resolver.getResource(path)
                    .getValueMap()
                    .get("jcr:title", "No title");
                LOG.info("Successfully read content from: {}", path);
                return content;
            } else {
                LOG.warn("Resource not found at path: {}", path);
                return null;
            }
            
        } catch (Exception e) {
            LOG.error("Error reading content from {}: {}", path, e.getMessage(), e);
            return null;
        } finally {
            if (resolver != null && resolver.isLive()) {
                resolver.close();
            }
        }
    }
}
```

#### **Step 5: Create OSGi Service with Logging**

**File:** `core/src/main/java/com/mycompany/myproject/services/OrderService.java`

```java
package com.mycompany.myproject.services;

import org.osgi.service.component.annotations.Component;
import org.osgi.service.component.annotations.Reference;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Component(service = OrderService.class)
public class OrderService {
    
    private static final Logger LOG = LoggerFactory.getLogger(OrderService.class);
    
    @Reference
    private ContentReaderService contentReaderService;
    
    public void processOrder(String orderId) {
        LOG.debug("Starting order processing for orderId: {}", orderId);
        
        try {
            // Validate order
            if (!validateOrder(orderId)) {
                LOG.warn("Order {} failed validation", orderId);
                return;
            }
            
            // Read order content
            String orderPath = "/content/orders/" + orderId;
            String orderContent = contentReaderService.readContent(orderPath);
            
            if (orderContent != null) {
                LOG.info("Processing order {} with content: {}", orderId, orderContent);
                // Process order...
                LOG.info("Order {} processed successfully", orderId);
            } else {
                LOG.error("Failed to retrieve order content for orderId: {}", orderId);
            }
            
        } catch (Exception e) {
            LOG.error("Critical error processing order {}: {}", orderId, e.getMessage(), e);
        }
    }
    
    private boolean validateOrder(String orderId) {
        LOG.debug("Validating order: {}", orderId);
        
        if (orderId == null || orderId.isEmpty()) {
            LOG.warn("Invalid orderId: null or empty");
            return false;
        }
        
        LOG.debug("Order {} validation successful", orderId);
        return true;
    }
}
```

#### **Step 6: Create Servlet Using Service**

**File:** `core/src/main/java/com/mycompany/myproject/servlets/OrderProcessingServlet.java`

```java
package com.mycompany.myproject.servlets;

import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.api.SlingHttpServletResponse;
import org.apache.sling.api.servlets.HttpConstants;
import org.apache.sling.api.servlets.SlingAllMethodsServlet;
import org.apache.sling.api.servlets.SlingSafeMethodsServlet;
import org.osgi.service.component.annotations.Component;
import org.osgi.service.component.annotations.Reference;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import com.mycompany.myproject.services.OrderService;

@Component(
    service = SlingAllMethodsServlet.class,
    property = {
        "sling.servlet.paths=/bin/process-order",
        "sling.servlet.methods=" + HttpConstants.METHOD_POST
    }
)
public class OrderProcessingServlet extends SlingAllMethodsServlet {
    
    private static final Logger LOG = LoggerFactory.getLogger(OrderProcessingServlet.class);
    
    @Reference
    private OrderService orderService;
    
    @Override
    protected void doPost(SlingHttpServletRequest request, SlingHttpServletResponse response) {
        LOG.debug("OrderProcessingServlet received POST request");
        
        try {
            String orderId = request.getParameter("orderId");
            
            if (orderId == null || orderId.isEmpty()) {
                LOG.warn("Request missing orderId parameter");
                response.setStatus(400);
                response.getWriter().write("{\"error\": \"Missing orderId\"}");
                return;
            }
            
            LOG.info("Processing order request for orderId: {}", orderId);
            orderService.processOrder(orderId);
            
            response.setStatus(200);
            response.getWriter().write("{\"status\": \"Order processed\"}");
            
        } catch (Exception e) {
            LOG.error("Error in OrderProcessingServlet: {}", e.getMessage(), e);
            response.setStatus(500);
            try {
                response.getWriter().write("{\"error\": \"Internal server error\"}");
            } catch (Exception ex) {
                LOG.error("Error writing response: {}", ex.getMessage());
            }
        }
    }
}
```

---

### Running the Example Project

#### **Build and Deploy**

```bash
# Build the project
mvn clean install -PautoInstallPackage

# Build with specific runmode
mvn clean install -PautoInstallPackage -Dsling.run.modes=author,dev

# For production deployment
mvn clean install -PautoInstallPackage -Dsling.run.modes=publish,prod
```

#### **Verify Deployment**

1. **Check Bundles:**
   - Go to `/system/console/bundles`
   - Search for your bundle `com.mycompany.myproject.core`
   - Verify status is **Active**

2. **Check Configuration:**
   - Go to `/system/console/configMgr`
   - Search for `ContentReaderService`
   - Verify it's loaded

3. **Check Service User:**
   - Go to **Tools → Security → Users**
   - Verify `myproject-content-reader` exists

4. **Check Logs:**
   - Go to `/system/console/slinglog`
   - Search for `com.mycompany.myproject`
   - Click on logger to view logs

5. **Test Endpoint:**
   ```bash
   curl -X POST "http://localhost:4502/bin/process-order?orderId=12345"
   ```

6. **View Logs in File:**
   ```bash
   tail -f crx-quickstart/logs/myproject.log
   ```

---

### Configuration Summary

| Component | Configuration File | Purpose |
|-----------|-------------------|---------|
| **Service User** | `RepositoryInitializer~myproject.cfg.json` | Create service user and permissions |
| **Service Mapping** | `ServiceUserMapperImpl.amended-myproject.cfg.json` | Map service to user |
| **Logger** | `LogManager.factory.config-myproject.cfg.json` | Configure logging |
| **Service** | `ContentReaderService.java` | OSGi service with logging |
| **Servlet** | `OrderProcessingServlet.java` | REST endpoint |

---

### Troubleshooting Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| **Service not loading** | Missing dependency | Check OSGi console for unsatisfied references |
| **Null logger** | LoggerFactory issue | Verify SLF4J import is correct |
| **Service user not found** | Configuration not deployed | Check `/apps/myproject/osgiconfig/` structure |
| **Permission denied** | Service user lacks permissions | Verify ACL configuration in Repo Init |
| **Logs not appearing** | Logger not configured | Create logger via Web Console or config file |
| **Servlet 404** | Path not registered | Check servlet properties and path mapping |

---

## Best Practices Summary

✅ **Service Users:**
- Always use service users instead of admin access
- Follow principle of least privilege
- Use Repo Init for consistent setup

✅ **Logging:**
- Use parameterized logging (avoid string concatenation)
- Use appropriate log levels (DEBUG/INFO/WARN/ERROR)
- Log errors with exceptions

✅ **Custom Loggers:**
- Create separate loggers per module
- Use different files for different components
- Adjust log levels per environment

✅ **Runmodes:**
- Use runmodes for environment-specific configuration
- Override only what's necessary
- Document runmode requirements

✅ **Web Console:**
- Use for troubleshooting and debugging
- Create configurations programmatically for deployment
- Regularly review bundle status and health

✅ **Code:**
- Inject dependencies via @Reference
- Use try-finally to close resources
- Add appropriate logging for troubleshooting

---

## Key Interview Questions

**Q1: Why use service users?**
A: Service users follow least privilege principle, provide security through limited permissions, are auditable, and replace dangerous admin access methods.

**Q2: How do you configure custom loggers?**
A: Using OSGi configuration files in `osgiconfig/config/` with `LogManager.factory.config.cfg.json` or via Web Console.

**Q3: What are runmodes and their importance?**
A: Runmodes identify environment context, allowing different configurations per environment without code changes.

**Q4: How do you log in OSGi services?**
A: Use SLF4J with `LoggerFactory.getLogger()` and `log.debug()`, `log.info()`, `log.warn()`, `log.error()`.

**Q5: Where is CQ Web Console located?**
A: At `http://localhost:4502/system/console` (configurable port).

