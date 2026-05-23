
# AEM OSGi Console Guide

**Complete Guide to Understanding and Using the OSGi Console in Adobe Experience Manager**

---

## Table of Contents

1. [What is OSGi Console?](#what-is-osgi-console)
2. [How to Access](#how-to-access)
3. [Important Tabs Explained](#important-tabs-explained)
4. [Debugging Scenarios](#debugging-scenarios)
5. [Common Issues & Solutions](#common-issues--solutions)
6. [Interview Q&A](#interview-qa)
7. [Quick Reference](#quick-reference)

---

## What is OSGi Console?

### Definition

The **OSGi Console** is the backend administration panel in AEM used to manage:
- **Bundles** (deployed code modules/JARs)
- **Services** (components like Servlets, Filters, Schedulers)
- **Configurations** (runtime settings)
- **System Information** (instance details)

### Simple Analogy

Think of OSGi Console as the **"Control Center of AEM Backend"**

```
┌─────────────────────────────────────┐
│      OSGi Console (Control Center)  │
├─────────────────────────────────────┤
│  ✓ Start/Stop Services              │
│  ✓ Deploy/Manage Code               │
│  ✓ Configure Components             │
│  ✓ Debug Issues                     │
│  ✓ Monitor System Health            │
└─────────────────────────────────────┘
```

### Why It's Important

✅ Verify deployed code is running  
✅ Troubleshoot service issues  
✅ Update configurations without redeploying  
✅ Monitor system health  
✅ Debug component dependencies  
✅ Check instance environment (Author/Publish)  

---

## How to Access

### Main Console URL
```
http://localhost:4502/system/console
```

### Direct Tab URLs
```
Bundles         → http://localhost:4502/system/console/bundles
Components      → http://localhost:4502/system/console/components
Configurations  → http://localhost:4502/system/console/configMgr
Sling Settings  → http://localhost:4502/system/console/status-slingsettings
System Info     → http://localhost:4502/system/console/slinginfo
```

### Authentication
- **Username:** admin (or your configured admin user)
- **Password:** admin (or your configured password)

---

## Important Tabs Explained

## 🔹 1. BUNDLES

### URL
```
/system/console/bundles
```

### What It Is

A **Bundle** is an OSGi module that contains:
- Java classes (compiled code)
- Dependencies
- Metadata (manifest)
- Resources

**In simple terms:** A JAR file deployed in AEM

### Bundle Statuses Explained

| Status | Meaning | Action Required |
|--------|---------|-----------------|
| **ACTIVE** ✅ | Bundle is running | None - everything OK |
| **INSTALLED** ⚠️ | Bundle loaded but not started | Usually resolved automatically |
| **RESOLVED** ⚠️ | All dependencies satisfied | Wait, usually starts automatically |
| **STARTING** ⏳ | Bundle is initializing | Wait a moment |
| **STOPPING** ⏳ | Bundle is shutting down | Wait a moment |
| **UNINSTALLED** ❌ | Bundle removed | May need to redeploy |

### What You Can Do in Bundles Tab

#### 1. Start a Bundle
```
Step 1: Find your bundle in the list
Step 2: Click the "Start" button
Step 3: Verify status changes to "ACTIVE"
```

#### 2. Stop a Bundle
```
Step 1: Find your bundle
Step 2: Click the "Stop" button
Step 3: Status changes to "RESOLVED"
```

#### 3. Refresh a Bundle
```
Step 1: Find your bundle
Step 2: Click the "Refresh" button
Step 3: Bundle reloads dependencies
```

### Bundle Information Panel

When you click on a bundle, you see:

```
Bundle ID: 123
Symbolic Name: com.example.myapp
Version: 1.0.0
State: Active
Services: 
  - com.example.services.MyService
Imports: 
  - org.apache.sling.api
  - javax.servlet
Exports: 
  - com.example.api
```

**What it means:**
- **Symbolic Name:** Unique identifier for the bundle
- **Version:** Current version of the bundle
- **Services:** Components this bundle provides
- **Imports:** Libraries this bundle uses
- **Exports:** Classes available to other bundles

### Why It's Important

✅ Verify your code deployment was successful  
✅ Check if code is actively running  
✅ Troubleshoot deployment failures  
✅ Monitor for failed bundles  
✅ Restart stuck services  

### Common Bundle Issues

#### Issue 1: Bundle Status is "INSTALLED" (Not ACTIVE)
```
Problem: Bundle is loaded but not running
Causes: 
  - Waiting for dependencies to resolve
  - Missing required packages
  - Incorrect manifest configuration
  
Solution:
  1. Check the bundle for error messages
  2. Click "Refresh" to retry
  3. Check Required Packages section
  4. Verify all @Reference dependencies are satisfied
```

#### Issue 2: Bundle Appears Multiple Times
```
Problem: Same bundle listed twice with different versions
Cause: Old version not uninstalled before deploying new version

Solution:
  1. Click on old bundle version
  2. Click "Uninstall"
  3. Refresh the page
  4. Deploy new version
```

#### Issue 3: Bundle Shows Red X (Error)
```
Problem: Bundle failed to start
Possible Causes:
  - Missing dependency
  - Exception during activation
  - Incompatible Java version
  
Solution:
  1. Check the error message in bundle details
  2. Review logs in crx-quickstart/logs/error.log
  3. Fix the issue and redeploy
```

### Interview Line

👉 *"Bundles represent deployed code modules in AEM. The Bundles tab shows the status of all deployed JARs. We use it to verify if our code is successfully deployed and actively running. Each bundle must be in ACTIVE state for services to work properly."*

---

## 🔹 2. COMPONENTS (SERVICES)

### URL
```
/system/console/components
```

### What It Is

**Components** are OSGi services deployed in AEM. Examples:
- **Servlets** (handle HTTP requests)
- **Filters** (intercept requests/responses)
- **Listeners** (react to events)
- **Schedulers** (run background tasks)

### Component Statuses

| Status | Meaning | Action Required |
|--------|---------|-----------------|
| **SATISFIED** ✅ | Service is ready to use | None - everything OK |
| **UNSATISFIED** ❌ | Missing dependencies | Fix missing @Reference |
| **DISABLED** ⏸️ | Service turned off | Re-enable if needed |
| **FACTORY** 🏭 | Multiple instances possible | Check configuration |

### Component Information Panel

When you click on a component:

```
Component: com.example.services.AuthenticationFilter
Status: Satisfied
Bundle: com.example.myapp (v1.0.0)

Configuration:
  - sling.filter.scope = REQUEST
  - sling.filter.order = -500

References:
  - resourceResolver (SATISFIED)
  - logger (SATISFIED)

Implementation Class:
  - com.example.filters.AuthenticationFilter
```

**What it means:**
- **Status:** Is the service ready to use?
- **References:** Does it have required dependencies?
- **Configuration:** What are its settings?
- **Implementation Class:** The actual Java class

### Common Component Issues

#### Issue 1: Component Status is "UNSATISFIED"

```
Problem: Component can't start
Cause: Missing @Reference dependency

Example:
@Component
public class MyService {
    @Reference
    private ResourceResolver resolver;  // ❌ NOT SATISFIED
}

Solution:
  1. Check References section for "unsatisfied" items
  2. Verify the referenced service exists
  3. Check bundle is ACTIVE
  4. Redeploy if necessary

Example Fix:
@Reference(cardinality = ReferenceCardinality.OPTIONAL)
private ResourceResolver resolver;  // ✅ Now OPTIONAL
```

#### Issue 2: Servlet Not Working

```
Step-by-Step Debugging:

1. Verify Bundle Status:
   /system/console/bundles
   → Find your bundle
   → Status must be ACTIVE
   
2. Verify Service Status:
   /system/console/components
   → Find your servlet component
   → Status must be SATISFIED
   
3. Check Configuration:
   /system/console/configMgr
   → Search for your service
   → Verify all required fields are filled
   
4. Check Error Logs:
   crx-quickstart/logs/error.log
   → Look for stack traces
```

### Why It's Important

✅ Verify services are active and ready  
✅ Identify missing dependencies  
✅ Troubleshoot why servlets/filters aren't working  
✅ Monitor service health  
✅ Debug component configuration issues  

### Interview Line

👉 *"The Components tab shows all deployed OSGi services like Servlets, Filters, and Schedulers. Each component must be in SATISFIED status to work properly. We use this to verify if our services have all their dependencies satisfied and are ready to handle requests."*

---

## 🔹 3. CONFIGURATIONS (ConfigMgr)

### URL
```
/system/console/configMgr
```

### What It Is

**Configurations** are runtime settings for OSGi services and AEM components. They can be modified **without redeploying code**.

### Key Features

✅ Dynamic configuration changes  
✅ No restart required  
✅ Environment-specific settings  
✅ Persist across restarts  
✅ Can override defaults  

### Types of Configurations

#### 1. Service Configurations
```
Example: Scheduler Configuration
Component: DailyCleanupScheduler
Property: scheduler.expression = "0 0 2 * * ?"
Value: Run every day at 2:00 AM
```

#### 2. Factory Configurations
```
Example: Multiple Scheduler Instances
Each can have different schedules
Instance 1: Every 5 minutes
Instance 2: Every hour
Instance 3: Every day at midnight
```

#### 3. Global Configurations
```
Example: Day CQ Mail Service
SMTP Host: smtp.example.com
SMTP Port: 587
From Address: noreply@example.com
```

### How to Create/Update Configuration

#### Step-by-Step Example: Update Scheduler Timing

```
1. Go to ConfigMgr:
   http://localhost:4502/system/console/configMgr

2. Search for your service (example: "DailyCleanupScheduler")

3. Click on the service name to open editor

4. Edit fields:
   scheduler.expression: 0 0 3 * * ?  (Change to 3 AM)
   scheduler.concurrent: false        (Don't run parallel)

5. Click "Save"

6. Configuration updated immediately!
   (No redeploy needed)
```

### Common Configurations

| Configuration | Purpose | Example Value |
|---------------|---------|----------------|
| **Scheduler Expression** | Cron timing | 0 0/5 * * * ? |
| **Email Service** | SMTP settings | smtp.gmail.com |
| **DAM Settings** | Asset configuration | Max file size |
| **Logger Levels** | Logging detail | DEBUG, INFO, ERROR |
| **Replication Agents** | Content sync | Publish agent URL |

### Real-World Example: Configure Email Service

```
Service: Day CQ Mail Service
Location: /system/console/configMgr

Configuration Fields:
┌─────────────────────────────────────┐
│ SMTP Hostname: smtp.gmail.com       │
│ SMTP Port: 587                      │
│ SMTP User: admin@example.com        │
│ SMTP Password: ••••••••             │
│ From Address: noreply@example.com   │
│ Reply To: support@example.com       │
│ Use SSL: true                       │
│ Use TLS: true                       │
└─────────────────────────────────────┘
```

### Factory Configuration Example: Multiple Replication Agents

```
/system/console/configMgr
→ Search "replication"
→ Create multiple instances:

Instance 1: Publish Agent
  - Name: Publish
  - Trigger: True
  - Target URL: http://publish:4503

Instance 2: Preview Agent
  - Name: Preview
  - Trigger: True
  - Target URL: http://preview:4503
```

### Why It's Important

✅ Update settings without redeploy  
✅ Environment-specific configurations  
✅ Quick fixes without downtime  
✅ Manage secrets (passwords, API keys)  
✅ Fine-tune service behavior  

### Common Configuration Issues

#### Issue 1: Configuration Not Taking Effect
```
Problem: Changed config but service still uses old values
Cause: Component needs restart

Solution:
  1. Go to Components tab
  2. Find the service
  3. Click "Disable" then "Enable"
  4. Service reloads configuration
```

#### Issue 2: Cannot Find Configuration
```
Problem: Service configuration not visible in ConfigMgr
Cause: Service may not support runtime configuration

Solution:
  1. Check service class for @Component annotation
  2. Verify configurationFactory or metatype declaration
  3. Ensure bundle is ACTIVE
```

#### Issue 3: Invalid Configuration Values
```
Problem: Saved configuration but service fails
Cause: Invalid values for configuration properties

Example Error:
"scheduler.expression must be valid cron format"

Solution:
  1. Review error message
  2. Fix value format
  3. Re-save configuration
```

### Interview Line

👉 *"ConfigMgr is used to manage OSGi configurations dynamically without code changes or redeploy. We can update scheduler timings, email settings, API URLs, and other runtime parameters. Changes take effect immediately after saving."*

---

## 🔹 4. SLING SETTINGS

### URL
```
/system/console/status-slingsettings
```

### What It Is

Displays critical **AEM instance information** and Sling framework settings.

### Information Provided

#### 1. Instance Details
```
Instance ID: unique-instance-id-12345
Instance Home: /path/to/crx-quickstart
Current Java Version: 11.0.12
```

#### 2. Run Modes
```
Author Mode:
  → Single user editing environment
  → Used for content creation
  → localhost:4502

Publish Mode:
  → Multi-user viewing environment
  → Content delivered to users
  → localhost:4503

Example Config by Mode:
  sling.run.modes=author,dev

  If author mode:
    - Load author-specific bundles
    - Use authoring workflows
    - Enable edit dialogs

  If publish mode:
    - Load publish-specific bundles
    - Disable edit capabilities
    - Optimize for performance
```

#### 3. Framework Details
```
OSGi Framework: Apache Felix
Sling Version: 12.0.0
Java Version: 11.0.12
System Load Average: 0.45
```

#### 4. Configured Properties
```
sling.servlet.default.extensions: [list]
sling.servlet.paths: [list]
repository.home: /path/to/repository
```

### Run Modes Explained

#### What are Run Modes?

**Run modes** are configurations that determine how AEM behaves in different environments.

```
Development Environment:
  Run Modes: author, dev
  Behavior: Verbose logging, debug enabled

Staging Environment:
  Run Modes: author, publish, stage
  Behavior: Testing mode, staged content

Production Environment:
  Run Modes: publish
  Behavior: Optimized performance, security hardened
```

#### Common Run Modes

| Mode | Environment | Purpose |
|------|-------------|---------|
| **author** | Editing environment | Content creation |
| **publish** | Delivery environment | Content consumption |
| **dev** | Development | Debug, verbose logging |
| **stage** | Staging | Pre-production testing |
| **crx3** | Repository type | Segment Node Store |
| **minimal** | Lightweight | Reduced features |

#### How to Check Run Modes

**In Browser:**
```
/system/console/status-slingsettings
→ Look for "sling.run.modes" property
```

**In Logs:**
```
AEM startup log shows:
"Startup completed. Running in modes: author, dev"
```

**Programmatically:**
```java
@Reference
private SlingSettingsService slingSettings;

public void checkMode() {
    Set<String> runModes = slingSettings.getRunModes();
    
    if (runModes.contains("author")) {
        System.out.println("This is Author instance");
    }
    if (runModes.contains("publish")) {
        System.out.println("This is Publish instance");
    }
}
```

### Real-World Use Case: Author vs Publish

```
Author Instance (localhost:4502):
  ✅ Full editing capabilities
  ✅ Workflows available
  ✅ DAM asset management
  ✅ Dialog editing
  ❌ Content NOT visible to public

Publish Instance (localhost:4503):
  ✅ Content visible to users
  ✅ Optimized for performance
  ✅ No editing capabilities
  ✅ Consumes content from Author
  ❌ Cannot create/edit pages
```

### Why It's Important

✅ Identify instance type (Author/Publish)  
✅ Check Java version compatibility  
✅ Verify Sling configuration  
✅ Debug environment-specific issues  
✅ Monitor system health indicators  

### Interview Line

👉 *"Sling Settings displays instance-level details like run modes (author/publish), Sling configuration, and system information. This helps us identify if we're on an author or publish instance and verify the environment is configured correctly."*

---

## 🔹 5. ADDITIONAL USEFUL TABS

### Logs
```
/system/console/slinglog
```
- View real-time logs
- Set log levels per package
- Debug issues in real-time

### System Information
```
/system/console/slinginfo
```
- Detailed system diagnostics
- Performance metrics
- Thread information
- Memory usage

### Memory
```
/system/console/vmstat
```
- JVM memory usage
- Heap statistics
- Garbage collection info

---

## Debugging Scenarios

## Scenario 1: Servlet Not Working

### Problem
User tries to access `/bin/myApp/articles` but gets 404 error.

### Step-by-Step Debugging

#### Step 1: Check Bundle Deployment
```
Go to: /system/console/bundles
Search for: your servlet bundle name

If ACTIVE:
  ✅ Bundle deployed successfully
  
If NOT ACTIVE:
  ❌ Bundle not running
  → Click "Start" button
  → Wait 5 seconds
  → Refresh page
  → If still fails, check for errors
```

#### Step 2: Check Service Status
```
Go to: /system/console/components
Search for: your servlet service

If SATISFIED:
  ✅ Service ready
  
If UNSATISFIED:
  ❌ Missing dependencies
  → Check References section
  → Verify all @Reference fields are satisfied
  → May need to add @Reference(cardinality = OPTIONAL)
```

#### Step 3: Verify Configuration
```
Go to: /system/console/configMgr
Search for: your servlet service

Check these properties:
  ✅ sling.servlet.paths = /bin/myApp/articles
  ✅ sling.servlet.methods = GET
  ✅ sling.servlet.extensions = json

If any are missing:
  → Add them in code as @Component property
  → Redeploy bundle
```

#### Step 4: Check Logs
```
Location: crx-quickstart/logs/error.log

Look for errors:
  - "Failed to instantiate"
  - "Missing reference"
  - "Exception during doGet"

Example error log entry:
[11/20/2024 10:23:45.123] ERROR 
[Component: ArticleServlet] 
Reference [resourceResolver] not satisfied
→ Add @Reference private ResourceResolver resolver;
```

#### Step 5: Test Servlet
```
URL: http://localhost:4502/bin/myApp/articles.json

Expected Response:
  {"articles": [...]}
  HTTP Status: 200 OK
  
If still failing:
  → Check servlet logs for exceptions
  → Verify request parameters
  → Test with curl: curl -u admin:admin \
    "http://localhost:4502/bin/myApp/articles.json"
```

---

## Scenario 2: Scheduler Not Running

### Problem
Scheduled task should run every 5 minutes but doesn't execute.

### Step-by-Step Debugging

#### Step 1: Verify Bundle Status
```
Go to: /system/console/bundles
Status: Must be ACTIVE ✅
```

#### Step 2: Verify Component Status
```
Go to: /system/console/components
Search for: Your Scheduler name
Status: Must be SATISFIED ✅
```

#### Step 3: Check Configuration
```
Go to: /system/console/configMgr
Search for: Your Scheduler name

Verify these are set:
  scheduler.expression = "0 0/5 * * * ?"
  scheduler.concurrent = false
  scheduler.name = "YourSchedulerName"

If missing:
  → Add to @Component annotation:
  @Component(
    service = Runnable.class,
    property = {
      "scheduler.expression=0 0/5 * * * ?",
      "scheduler.concurrent=false"
    }
  )
```

#### Step 4: Check Logs
```
Look in: crx-quickstart/logs/error.log

Success indicator:
  "Scheduler starting: YourSchedulerName"
  "Task completed successfully"

Error example:
  "Invalid cron expression"
  → Fix scheduler.expression format
  → Example valid: "0 0/5 * * * ?" (every 5 min)
```

#### Step 5: Manual Testing
```
1. Go to Components tab
2. Find your scheduler
3. Click "Disable" then immediately "Enable"
4. Check logs for execution
5. If runs manually but not on schedule:
   → Check expression syntax
   → Verify system clock is correct
   → Check if publishing to publish instance 
     (schedulers on publish may need Sling ID)
```

---

## Scenario 3: Filter Not Intercepting Requests

### Problem
Authentication filter should reject unauthenticated requests but doesn't work.

### Step-by-Step Debugging

#### Step 1: Bundle Check
```
/system/console/bundles → Status ACTIVE ✅
```

#### Step 2: Component Check
```
/system/console/components → Status SATISFIED ✅
```

#### Step 3: Verify Filter Configuration
```
Go to: /system/console/configMgr
Check filter properties:

Required properties:
  sling.filter.scope = REQUEST
  sling.filter.order = -500 (negative = early execution)
  
If scope is wrong:
  → REQUEST: Before processing
  → COMPONENT: During rendering
  → ERROR: On errors
```

#### Step 4: Test Request Flow
```
1. Add logging to filter:
   logger.info("Filter intercepting: " + request.getPathInfo());

2. Make request:
   curl -u admin:admin "http://localhost:4502/content/test"

3. Check logs:
   crx-quickstart/logs/error.log
   
Should see:
  "Filter intercepting: /content/test"
```

#### Step 5: Verify Filter Order
```
/system/console/components → Your filter

Check execution order:
  order = -500  → Runs very early
  order = 0     → Runs mid-execution
  order = 500   → Runs late

Tip: Negative orders execute first
```

---

## Common Issues & Solutions

## Issue 1: "Bundle Not Active"

### Symptoms
- Bundle shows "INSTALLED" or "RESOLVED" status
- Service not working
- 404 or service unavailable errors

### Causes
```
1. Waiting for dependencies
2. Missing required packages
3. Conflicting versions
4. Circular dependencies
5. Missing Maven dependency
```

### Solutions

**Solution A: Click Refresh**
```
1. Go to /system/console/bundles
2. Find your bundle
3. Click "Refresh" button
4. Wait 10 seconds
5. Check if now ACTIVE
```

**Solution B: Check Dependencies**
```
1. Click on bundle name
2. Scroll to "Required Packages" section
3. Look for ❌ items
4. Example: "Missing package: org.apache.sling.api"
5. Add to pom.xml:
   <dependency>
       <groupId>org.apache.sling</groupId>
       <artifactId>org.apache.sling.api</artifactId>
       <version>2.22.0</version>
   </dependency>
6. Redeploy
```

**Solution C: Check Manifest**
```
File: pom.xml

Add Maven Bundle Plugin:
<plugin>
    <groupId>org.apache.felix</groupId>
    <artifactId>maven-bundle-plugin</artifactId>
    <version>5.1.2</version>
    <extensions>true</extensions>
</plugin>
```

---

## Issue 2: "Component Unsatisfied"

### Symptoms
```
- Component status: UNSATISFIED
- @Reference field showing red X
- Service not available
```

### Causes
```
1. Referenced service not active
2. Wrong interface type
3. Service not exported
4. Bundle dependencies not satisfied
```

### Solutions

**Solution A: Make Reference Optional**
```java
@Reference(cardinality = ReferenceCardinality.OPTIONAL)
private MyService service;

// Service can now start even if MyService not available
```

**Solution B: Add Target Filter**
```java
@Reference(target = "(name=primary)")
private MyService service;

// Only inject service with property name=primary
```

**Solution C: Check Referenced Service**
```
1. Go to /system/console/components
2. Search for the referenced service (e.g., "MyService")
3. If not found → service not deployed
4. If UNSATISFIED → it has missing dependencies
5. Fix dependencies first, then recheck dependent service
```

---

## Issue 3: "Configuration Not Applied"

### Symptoms
```
- Changed configuration in ConfigMgr
- Service still uses old values
- No errors in logs
```

### Causes
```
1. Service not restarted
2. Configuration not persisted
3. Wrong configuration name
4. Multiple instances with same PID
```

### Solutions

**Solution A: Restart Component**
```
1. Go to /system/console/components
2. Find your service
3. Click "Disable"
4. Wait 3 seconds
5. Click "Enable"
6. Configuration reloaded
```

**Solution B: Verify Configuration Persisted**
```
1. Go to /system/console/configMgr
2. Find your configuration
3. Look for "Location" field
   Example: "file:crx-quickstart/install/..."
   Shows where it's saved
4. If location is "Runtime", it's temporary
5. Create config file instead:
   crx-quickstart/install/com.example.MyService.config
```

**Solution C: Clear AEM Cache**
```
1. Stop AEM
2. Delete: crx-quickstart/launchpad/
3. Start AEM
4. Redeploy configurations
```

---

## Issue 4: "Logger showing DEBUG but want INFO"

### Symptoms
```
- Too many log messages
- Performance impact
- Hard to find relevant errors
```

### Solutions

**In ConfigMgr:**
```
1. /system/console/configMgr
2. Search "Apache Sling Logging"
3. Find package: org.apache.sling (or your package)
4. Change level from DEBUG to INFO or WARN
5. Save - takes effect immediately
```

**In Code:**
```java
@Reference
private SlingSettingsService settings;

// Log only in dev mode
if (settings.getRunModes().contains("dev")) {
    logger.debug("Verbose debug info");
} else {
    logger.info("Production info");
}
```

---

## Interview Q&A

### Q1: What is OSGi Console and why is it important?

**Answer:**
"OSGi Console is the backend administration panel in AEM for managing bundles, services, and configurations. It's important because:
- It verifies deployed code is running and active
- It helps troubleshoot service deployment issues
- It allows dynamic configuration changes without redeploying
- It shows if all service dependencies are satisfied
- It monitors the health of all components and services"

---

### Q2: What's the difference between Bundles and Components?

**Answer:**
"Bundles are JAR files containing code and classes. Components are services deployed within those bundles. Think of it this way:
- Bundle = Container (like a box)
- Component = Item inside the box (like a service)

One bundle can contain multiple components/services."

---

### Q3: What does "UNSATISFIED" status mean?

**Answer:**
"UNSATISFIED means a component/service cannot start because it's missing required dependencies. For example, if a servlet has @Reference private ResourceResolver resolver; but ResourceResolver service is not available, the servlet stays UNSATISFIED.

Solutions:
1. Make the reference optional: @Reference(cardinality = OPTIONAL)
2. Ensure the referenced service's bundle is ACTIVE
3. Check the error message for which dependency is missing
4. Verify the referenced service is exported from its bundle"

---

### Q4: How do you debug a servlet that's not working?

**Answer:**
"Step-by-step debugging process:
1. Check bundle status in /system/console/bundles (must be ACTIVE)
2. Check component status in /system/console/components (must be SATISFIED)
3. Verify configuration in /system/console/configMgr (correct path, methods, extensions)
4. Test the URL with curl or browser
5. Check error logs in crx-quickstart/logs/error.log for exceptions
6. Verify all @Reference dependencies are satisfied
7. If still failing, restart the component (Disable/Enable)"

---

### Q5: What's the difference between Author and Publish modes?

**Answer:**
"Author mode (port 4502):
- Editing environment for content creators
- Has authoring features, workflows, dialogs
- Content created here but NOT visible to public

Publish mode (port 4503):
- Delivery environment for end users
- Content visible to public
- No editing capabilities
- Receives content from Author via replication

You can check which mode you're on using /system/console/status-slingsettings"

---

### Q6: Can you change scheduler timing without redeploying?

**Answer:**
"Yes! Through ConfigMgr:
1. Go to /system/console/configMgr
2. Search for your scheduler name
3. Find 'scheduler.expression' property
4. Change cron expression (e.g., from '0 0/5' to '0 0/10')
5. Click Save
6. Changes take effect immediately without redeploy

Example cron expressions:
- '0 0/5 * * * ?' = Every 5 minutes
- '0 0 2 * * ?' = Every day at 2 AM
- '0 0 0 ? * MON' = Every Monday at midnight"

---

### Q7: How do you know if a configuration change took effect?

**Answer:**
"Methods to verify:
1. Check logs to see if service logged new configuration values
2. Disable and re-enable the component to force reload
3. For schedulers, manually trigger and check logs
4. For servlets, make test request and verify new behavior
5. Check ConfigMgr again to confirm value persisted

If changes don't take effect:
- Restart the component (Disable/Enable)
- Check Location field shows config is persisted
- Look for errors in error.log"

---

### Q8: What does "sling.filter.order" mean?

**Answer:**
"The order property controls when filters execute:
- Negative numbers = Execute early (e.g., -500)
- Zero = Execute mid-flow
- Positive numbers = Execute late (e.g., 500)

Example:
Order -500 → runs first
Order 0 → runs second  
Order 500 → runs last

For authentication filter, use negative order so it runs before other filters can access the request."

---

### Q9: How do you find which bundle contains a specific component?

**Answer:**
"In Components tab:
1. /system/console/components
2. Search for the component name
3. Look at the search results
4. Click on component name
5. In details panel, you'll see:
   'Bundle: com.example.myapp (v1.0.0)'

Alternatively:
1. Go to Bundles tab
2. Search for bundle name (e.g., 'com.example')
3. Click on bundle
4. Scroll to 'Services' section
5. See all components provided by that bundle"

---

### Q10: What's the difference between @Reference and @ValueMapValue in Sling Models?

**Answer:**
"@Inject and @ValueMapValue are annotations for dependency injection:

@Inject:
- Generic injection from AEM registry
- Can get values from anywhere (components, services, configurations)
- Used for Sling objects and services
- Example: @Inject private ResourceResolver resolver;

@ValueMapValue:
- Specific to JCR properties of current resource
- Gets values only from component properties
- Used for component configuration
- Example: @ValueMapValue private String title;

Analogy:
- @Inject = 'Get me water from anywhere'
- @ValueMapValue = 'Get me water from the kitchen only'"

---

## Quick Reference

### OSGi Console URLs

```
Main Console:     http://localhost:4502/system/console
Bundles:          http://localhost:4502/system/console/bundles
Components:       http://localhost:4502/system/console/components
Configurations:   http://localhost:4502/system/console/configMgr
Sling Settings:   http://localhost:4502/system/console/status-slingsettings
Logs:             http://localhost:4502/system/console/slinglog
System Info:      http://localhost:4502/system/console/slinginfo
```

### Common Cron Expressions for Schedulers

```
Every 5 minutes:     0 0/5 * * * ?
Every hour:          0 0 * * * ?
Every day at 2 AM:   0 0 2 * * ?
Every day at noon:   0 0 12 * * ?
Every Monday:        0 0 0 ? * MON
First of month:      0 0 0 1 * ?
Every 30 seconds:    0/30 * * * * ?
```

### Debugging Checklist

```
For any issue, check in this order:

☐ Bundle Status
  → Go to /system/console/bundles
  → Find your bundle
  → Status must be ACTIVE

☐ Component Status
  → Go to /system/console/components
  → Find your service
  → Status must be SATISFIED

☐ Configuration
  → Go to /system/console/configMgr
  → Verify all properties are correct
  → Check if needs restart

☐ Dependencies
  → Check all @Reference fields
  → Verify referenced services are ACTIVE
  → Look for "unsatisfied" references

☐ Logs
  → Check crx-quickstart/logs/error.log
  → Look for stack traces and errors
  → Match errors to issues

☐ Test
  → Make test request
  → Verify expected behavior
  → Monitor logs during test
```

### Status Meanings

```
ACTIVE ✅
  → Service is running and ready
  → Everything is fine

UNSATISFIED ❌
  → Service has missing dependencies
  → Fix: Make @Reference optional or ensure dependency is available

INSTALLED ⚠️
  → Bundle loaded but not started
  → Usually temporary, waits for dependencies

RESOLVED ⚠️
  → Dependencies met but not started
  → Usually starts automatically after refresh

DISABLED ⏸️
  → Service turned off intentionally
  → Enable to restart
```

---

## Summary Table

| Component | Check | Expected Status | If Not | Action |
|-----------|-------|-----------------|--------|--------|
| **Bundle** | /bundles | ACTIVE | INSTALLED | Click Refresh or Start |
| **Service** | /components | SATISFIED | UNSATISFIED | Check dependencies, make optional |
| **Configuration** | /configMgr | Exists & Correct | Missing/Wrong | Create or update, restart component |
| **Logger** | /slinglog | INFO+ | DEBUG | Adjust log level in ConfigMgr |
| **Scheduler** | /configMgr | Has expression | Missing | Add scheduler.expression property |

---

## Key Takeaways

✅ OSGi Console is the control center for AEM backend  
✅ Always check Bundle → Component → Configuration in that order  
✅ Use ConfigMgr for dynamic changes without redeploy  
✅ Logs are your best friend for debugging  
✅ Understand Author vs Publish modes  
✅ Know when to make @Reference optional  
✅ Check Sling Settings to identify environment  
✅ Component must be SATISFIED for service to work  
✅ Bundle must be ACTIVE for components to deploy  
✅ Restart components (Disable/Enable) after configuration changes  

---

## Additional Resources

- [Apache Sling Documentation](https://sling.apache.org/documentation.html)
- [OSGi Alliance Specifications](https://osgi.org)
- [AEM Documentation](https://experienceleague.adobe.com/docs/experience-manager-cloud-service)
- [Felix Documentation](https://felix.apache.org/documentation/)

---

**Last Updated:** November 2024  
**Version:** 1.0  
**Author:** AEM Development Team

---

*This guide serves as a comprehensive reference for using and understanding the OSGi Console in Adobe Experience Manager for debugging, configuration management, and system administration.*
