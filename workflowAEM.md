# 📘 AEM Workflows – Complete Simple Guide

**Quick Revision Guide for AEM Workflows - Perfect for Interviews & Exams**

---

## Table of Contents

1. [Introduction](#introduction)
2. [Workflow Console](#workflow-console)
3. [Workflow Types](#workflow-types)
4. [Workflow Terms](#workflow-terms)
5. [Workflow Execution](#workflow-execution)
6. [Custom Workflow Process](#custom-workflow-process)
7. [Workflow Categorization](#workflow-categorization)
8. [Real-Time Examples](#real-time-examples)
9. [Interview Q&A](#interview-qa)
10. [Quick Revision Checklist](#quick-revision-checklist)

---

## Introduction

### 🔹 What is an AEM Workflow?

A **workflow** is a process that **automates tasks in AEM**.

It moves content through different steps automatically or manually.

**Simple Example:**
```
Create page → Review → Approve → Publish
```

### ✅ Purpose of Workflows

| Purpose | Benefit |
|---------|---------|
| **Save time** | No manual action needed |
| **Reduce manual work** | Automation handles repetitive tasks |
| **Ensure proper approval** | Every page/asset follows same process |
| **Track history** | Know who did what and when |
| **Maintain quality** | Multiple reviews ensure quality |
| **Enforce governance** | Follow company policies |

### 👉 Real-Life Analogy

```
Without Workflow (Manual):
Author writes → Manually sends email to manager 
→ Manager reads email → Finds page in system 
→ Reviews → Manually approves → Notifies author 
→ Author publishes manually
TIME: 2-3 hours ⏱️

With Workflow (Automated):
Author submits → Auto-notification to manager 
→ Manager opens task → Reviews → Approves 
→ Auto-published
TIME: 30 minutes ⚡
```

---

## Workflow Console

### 📍 How to Access

**Path in AEM:**
```
Tools → Workflow
```

**Direct URL:**
```
http://localhost:4502/aem/workflow
```

### 🔹 Workflow Console Sections

#### 1️⃣ MODELS

**What it is:** Create/edit workflow templates (blueprints)

```
Models → List of all workflows
         ↓
Create → Design new workflow
         ↓
Edit → Modify existing workflow
```

**What you do here:**
- Design workflow steps
- Add participant steps (user actions)
- Add process steps (automated actions)
- Set up conditions and splits
- Configure notifications

**Example Models:**
```
✓ Request for Activation (Page approval)
✓ Publish (Auto-publish)
✓ DAM Asset Update (Image processing)
✓ Translation (Multi-language)
```

---

#### 2️⃣ INSTANCES

**What it is:** Running/completed workflows

```
Instance = One execution of a model

Example:
  Model: "Request for Activation"
  Instance 1: John's page approval
  Instance 2: Sarah's page approval
  Instance 3: Mike's page approval
```

**What you see:**
```
List shows:
  ✓ Status (Running, Completed, Aborted)
  ✓ Page/Asset being processed
  ✓ Started by (who started it)
  ✓ Current step (where is it now)
  ✓ Created date
```

**Status Meanings:**
| Status | Meaning |
|--------|---------|
| **RUNNING** | Workflow in progress |
| **COMPLETED** | Workflow finished |
| **ABORTED** | Workflow stopped |
| **SUSPENDED** | Workflow paused |
| **STALE** | Stuck (timeout reached) |

---

#### 3️⃣ LAUNCHERS

**What it is:** Auto-trigger workflows on events

```
Launcher = Rule that starts workflow automatically

Example:
  Event: Page created under /content/my-site
  Action: Start "Approval Workflow" automatically
```

**How it works:**
```
Event occurs (e.g., page created)
         ↓
Launcher detects event
         ↓
Checks conditions
         ↓
If matched → Workflow starts automatically
         ↓
No manual action needed!
```

**Common Launchers:**
- Start workflow on page creation
- Start workflow on asset upload
- Start workflow on content modification
- Start workflow on schedule

---

## Workflow Types

### 1️⃣ PARTICIPANT WORKFLOW

**What it is:** Needs human action

```
Step 1: Author submits page
         ↓
Step 2: Manager reviews (HUMAN ACTION NEEDED)
         ↓
Step 3: Manager approves or rejects
         ↓
Step 4: Publish (automatic)
```

**Use Case:** Content approval, reviews, sign-offs

**Example Code:**
```java
// This is just configuration, not Java code
Workflow: "Page Approval"
Steps:
  1. Submit (Author)
  2. Review (Manager) ← Human action
  3. Approve (Director) ← Human action
  4. Publish (Auto)
```

---

### 2️⃣ PROCESS WORKFLOW

**What it is:** Fully automated (Java code)

```
Triggered
   ↓
Step 1: Generate thumbnail (auto)
   ↓
Step 2: Extract metadata (auto)
   ↓
Step 3: Create renditions (auto)
   ↓
Completed
```

**Use Case:** Image processing, data transformation, automatic validations

**Example Code:**
```java
@Component(service = WorkflowProcess.class,
property = {"process.label=Generate Thumbnail"})
public class GenerateThumbnailProcess implements WorkflowProcess {
    
    @Override
    public void execute(WorkItem item, WorkflowSession session, MetaDataMap args) {
        // Get the page/asset path
        String path = (String) item.getWorkflowData().getPayload();
        
        // Your automation logic
        System.out.println("Generating thumbnail for: " + path);
        
        // Generate thumbnail
        // Save it
        // Complete step
    }
}
```

---

### 3️⃣ HYBRID WORKFLOW

**What it is:** Combination of both participant and process workflows

```
Step 1: Auto-validate content (process)
   ↓
Step 2: Manager reviews (participant) ← Human action
   ↓
Step 3: Auto-publish (process)
   ↓
Step 4: Auto-notify users (process)
```

**Use Case:** Complex workflows with both automatic and manual steps

**Example:**
```
1. Asset uploaded (trigger)
2. Auto-generate metadata (process)
3. Auto-generate thumbnails (process)
4. Send to reviewer (participant) ← Human action
5. Auto-publish if approved (process)
```

---

## Workflow Terms

### 📖 Key Terminology

| Term | Meaning | Example |
|------|---------|---------|
| **Workflow Model** | Design/template of workflow | "Page Approval Workflow" |
| **Workflow Instance** | One running workflow | "John's page approval #123" |
| **Step** | Single task in workflow | "Manager Review" |
| **Payload** | Content being processed | "/content/my-page" |
| **Work Item** | Task assigned to user | "Review page" assigned to Manager |
| **Transition** | Path between steps | From Review → Approve → Publish |
| **Participant** | Person doing the task | Manager, Editor, Director |
| **Inbox** | User task list | "You have 5 tasks to do" |
| **Handler Advance** | Auto move to next step | After step completes, go to next |
| **Transient** | Workflow without history | Fast, no database logging |

---

### 📊 Visual: Workflow Structure

```
┌──────────────────────────────────┐
│     WORKFLOW MODEL               │
│   (Design/Template)              │
└──────────────────────────────────┘

        │
        ├─ Step 1: Author Submits
        │
        ├─ Step 2: Manager Reviews ← Participant
        │         (Human action needed)
        │
        ├─ Step 3: Approve/Reject
        │
        ├─ Step 4: Auto-Publish ← Process
        │         (Automated)
        │
        └─ Step 5: Complete

Each time this model runs = New Instance
```

---

## Workflow Execution

### ✅ Ways to Start a Workflow

#### 1️⃣ MANUAL START

**From UI (Page Editor):**
```
1. Open page in editor
2. Click "Page" menu
3. Select "Start Workflow"
4. Choose workflow model
5. Click "Start"

→ Workflow instance created!
```

**From Workflow Console:**
```
1. Go to /aem/workflow/instances
2. Click "Create"
3. Select model
4. Select page/asset
5. Click "Start"

→ Workflow instance created!
```

---

#### 2️⃣ AUTOMATIC START (Launcher)

**How it works:**
```
Event Triggers:
  ✓ Page created
  ✓ Page modified
  ✓ Asset uploaded
  ✓ Schedule (cron)
  
Launcher detects
   ↓
Checks conditions
   ↓
Workflow starts automatically
```

**Configuration Example:**
```
Launcher: "Auto-approve on upload"
Trigger: Asset uploaded to /content/dam
Workflow: "Quick Approval"
Conditions: File type = image only

Result: Every image uploaded auto-starts workflow!
```

---

#### 3️⃣ PROGRAMMATICALLY (Java Code)

**Code Example:**
```java
@Component
public class WorkflowStarter {
    
    @Reference
    private WorkflowSession wfSession;
    
    public void startWorkflow(String pagePath) {
        try {
            // Get the workflow model
            WorkflowModel model = wfSession.getModel(
                "/etc/workflow/models/page-approval"
            );
            
            // Create workflow data
            WorkflowData wfData = wfSession.newWorkflowData(
                "JCR_PATH", 
                pagePath
            );
            
            // Start the workflow
            wfSession.startWorkflow(model, wfData);
            
            System.out.println("Workflow started!");
            
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

---

### 🔄 Execution Flow

```
TRIGGER (Manual / Automatic / Code)
   ↓
INITIALIZE (Load content)
   ↓
STEP 1 Executes
   ├─ If Participant: Wait for user action
   └─ If Process: Auto-execute
   ↓
STEP 2 Executes
   ├─ Conditional? (If approved go to 3, else go to 1)
   └─ Parallel? (Run multiple at same time)
   ↓
STEP N Executes
   ↓
COMPLETED
   ├─ Archive instance
   ├─ Send notification
   └─ History saved
```

---

## Custom Workflow Process

### 🔹 What is it?

A custom workflow process is **Java code** that handles automated tasks in a workflow.

**When to use:**
- Default workflow steps not enough
- Need complex business logic
- Integration with external systems
- Data transformation
- Custom validation

---

### ✅ How to Create Custom Workflow Process

#### Step 1: Create Java Class

```java
import org.osgi.service.component.annotations.Component;
import com.adobe.granite.workflow.exec.WorkflowProcess;
import com.adobe.granite.workflow.exec.WorkItem;
import com.adobe.granite.workflow.WorkflowSession;
import com.adobe.granite.workflow.metadata.MetaDataMap;
import com.adobe.granite.workflow.WorkflowException;

@Component(
    service = WorkflowProcess.class,
    property = {
        "process.label=Auto-generate Page Thumbnail",
        "process.description=Create thumbnail for pages automatically"
    }
)
public class GenerateThumbnailProcess implements WorkflowProcess {
    
    @Override
    public void execute(
            WorkItem workItem, 
            WorkflowSession workflowSession, 
            MetaDataMap args) throws WorkflowException {
        
        try {
            // Get the page path
            String pagePath = (String) workItem.getWorkflowData().getPayload();
            
            System.out.println("Processing: " + pagePath);
            
            // Your logic here
            generateThumbnail(pagePath);
            
            System.out.println("Thumbnail generated!");
            
        } catch (Exception e) {
            throw new WorkflowException("Process failed: " + e.getMessage());
        }
    }
    
    private void generateThumbnail(String pagePath) {
        // Your implementation
        // 1. Get page
        // 2. Take screenshot/create image
        // 3. Save as asset
        // 4. Link to page
    }
}
```

---

#### Step 2: Important Annotations

```java
@Component
  → Registers as OSGi component
  
service = WorkflowProcess.class
  → Type of service
  
property = {"process.label=..."}
  → Name shown in workflow editor
  → Important for identification
```

---

#### Step 3: Deploy & Use

```
1. Build & Package:
   mvn clean install
   
2. Deploy to AEM:
   mvn clean install -Paem-publish
   
3. Verify in OSGi Console:
   /system/console/components
   → Search for your process name
   → Status should be SATISFIED ✓
   
4. Use in Workflow Model:
   → Workflow Models
   → Edit model
   → Add your custom process
   → Save
```

---

### 📝 Process Parameters

```
You can pass parameters from workflow step:

In Workflow Model Configuration:
  Arguments: minLength=100, requireImage=true

In Java Code:
  String minLength = args.get("minLength", "100");
  boolean requireImage = args.get("requireImage", false);
```

---

## Workflow Categorization

### 🔹 What is it?

Organizing workflows into **groups/categories** for easy management.

### ✅ Categories in AEM

| Category | Purpose | Examples |
|----------|---------|----------|
| **DAM** | Digital asset management | Image processing, Video conversion |
| **WCM** | Page/content management | Page approval, Publishing |
| **CUSTOM** | Business specific | Company workflows |
| **PROJECTS** | Team collaboration | Campaign, Product launch |
| **SYSTEM** | AEM internal | Email notifications |

---

### 📂 How to Categorize

#### Method 1: Set in Workflow Properties

```
Open Workflow Model → Properties
Set: cq:category = "DAM"

Or in code:
@Component(property = {
    "process.label=...",
    "cq:category=DAM"
})
```

---

#### Method 2: Repository Location

```
Organize by folder:

/etc/workflow/models/
  ├─ dam/
  │   ├─ asset-update-wf
  │   └─ image-processing-wf
  ├─ wcm/
  │   ├─ page-approval-wf
  │   └─ publish-wf
  └─ custom/
      ├─ integration-wf
      └─ reporting-wf
```

---

#### Method 3: Naming Convention

```
Pattern: category-description-wf

Examples:
  ✓ dam-image-optimization-wf
  ✓ wcm-page-approval-wf
  ✓ custom-external-sync-wf
  ✓ projects-campaign-wf
```

---

## Real-Time Examples

### 📌 Example 1: Page Publishing Workflow

**Scenario:** Every new page must be reviewed before publishing

```
Step 1: Author creates page
        ↓
        (Workflow triggered by launcher)
        ↓
Step 2: Auto-validate (Process)
        ├─ Check spelling
        ├─ Check links
        └─ Check metadata
        ↓
Step 3: Manager review (Participant)
        ├─ Assigned to: Managers group
        ├─ Notification: Email sent
        ├─ Timeout: 2 days
        └─ Actions: Approve / Reject
        ↓
        If Approved:
        ├─ Step 4: Auto-publish (Process)
        ├─ Step 5: Notify author (Auto)
        └─ Workflow Completed ✓
        
        If Rejected:
        ├─ Step 4: Send back to author (Auto)
        ├─ Step 5: Wait for revision
        └─ Loop back to Step 2
```

**Configuration:**
```
Model Name: Page Publishing Workflow
Category: WCM
Triggers: Page created under /content/my-site
```

---

### 📌 Example 2: DAM Asset Processing

**Scenario:** When image uploaded, auto-process then review

```
Step 1: Asset uploaded
        ↓
        (Launcher detects upload)
        ↓
Step 2: Extract metadata (Process)
        ├─ Read EXIF data
        ├─ Auto-tag content
        └─ Extract dimensions
        ↓
Step 3: Generate renditions (Process)
        ├─ Create web size (500x500)
        ├─ Create thumbnail (100x100)
        └─ Create mobile size (300x300)
        ↓
Step 4: Quality review (Participant)
        ├─ Assigned to: DAM Team
        ├─ Review quality
        └─ Approve or reject
        ↓
        If Approved:
        ├─ Step 5: Publish to CDN (Process)
        └─ Workflow Completed ✓
        
        If Rejected:
        ├─ Step 5: Notify uploader
        └─ Workflow Aborted
```

---

### 📌 Example 3: Multi-Language Publishing

**Scenario:** Publish page in multiple languages with translations

```
Step 1: Author submits page
        ↓
Step 2: Auto-validate (Process)
        ↓
Step 3: Manager approves (Participant)
        ↓
Step 4: Send to translation (Process)
        ├─ Copy content to /content/de/
        ├─ Copy content to /content/fr/
        └─ Copy content to /content/es/
        ↓
Step 5: Translator review (Participant)
        ├─ German translator
        ├─ French translator  
        ├─ Spanish translator
        └─ Parallel reviews (all at same time!)
        ↓
Step 6: Auto-publish (Process)
        ├─ Publish English
        ├─ Publish German
        ├─ Publish French
        └─ Publish Spanish
        ↓
Workflow Completed ✓
```

---

## Interview Q&A

### ❓ Q1: What is an AEM Workflow?

**Answer:**
"An AEM Workflow is a process that automates tasks in AEM. It moves content through different steps, which can be manual (requiring user action) or automatic (handled by code).

Example: A page goes through Create → Review → Approval → Publish steps.

Benefits:
- Saves time
- Reduces manual work
- Ensures consistent quality
- Maintains audit trail
- Enforces governance"

---

### ❓ Q2: What's the difference between Workflow Model and Instance?

**Answer:**
"Workflow Model is a template (like a blueprint) that defines all the steps.

Workflow Instance is one actual execution of that model (like a project built from that blueprint).

Analogy:
- Model = Recipe
- Instance = One cooked meal from that recipe

One model can have many instances."

---

### ❓ Q3: What are the 3 types of workflows?

**Answer:**
"1. **Participant Workflow** - Needs human action
   Example: Manager reviews and approves

2. **Process Workflow** - Fully automated Java code
   Example: Auto-generate thumbnails

3. **Hybrid Workflow** - Mix of both
   Example: Auto-validate + Manual approval"

---

### ❓ Q4: How do you start a workflow?

**Answer:**
"3 ways:

1. **Manual** - User clicks 'Start Workflow' button

2. **Automatic (Launcher)** - Triggered by event
   Example: Page created → Auto-start workflow

3. **Programmatically** - Using Java code
   WorkflowSession.startWorkflow(model, data)"

---

### ❓ Q5: What is a Launcher?

**Answer:**
"A Launcher automatically starts workflows based on events.

Example:
- Event: Image uploaded to /content/dam
- Action: Auto-start 'Image Review Workflow'

Benefits: No manual action needed!"

---

### ❓ Q6: What is a custom workflow process?

**Answer:**
"Custom workflow process is Java code that handles automated tasks.

Used when default steps aren't enough.

Example code:
```java
@Component(service = WorkflowProcess.class,
property = {"process.label=My Process"})
public class MyProcess implements WorkflowProcess {
    execute() { // Your logic }
}
```

Steps:
1. Create Java class
2. Implement WorkflowProcess
3. Deploy as bundle
4. Use in workflow model"

---

### ❓ Q7: What is workflow categorization?

**Answer:**
"Organizing workflows into groups for easy management.

Common categories:
- DAM (Asset management)
- WCM (Page/content)
- CUSTOM (Your workflows)
- PROJECTS (Team projects)

Set using:
- Property: cq:category = 'DAM'
- Folder location
- Naming convention"

---

### ❓ Q8: What's the difference between Transient and Regular workflows?

**Answer:**
"Transient Workflow:
- Doesn't save history
- Runs fast
- Good for background tasks
- No audit trail

Regular Workflow:
- Saves complete history
- Slower (more database operations)
- Required for compliance
- Full audit trail

Use transient for speed, regular for accountability."

---

### ❓ Q9: How do you pass data between workflow steps?

**Answer:**
"Methods:

1. **Using Metadata:**
```java
MetaDataMap wfMetadata = workItem.getMetaDataMap();
wfMetadata.put('imageSize', '1024x768');
```

2. **Using Workflow Data:**
```java
workItem.getWorkflowData().setPayload('JCR_PATH', newPath);
```

3. **Using JCR Properties:**
- Store in page properties
- Read in next step

4. **Using Comments:**
- Add notes in participant steps
- Manual communication"

---

### ❓ Q10: What happens if workflow times out?

**Answer:**
"If workflow exceeds timeout:

1. Timeout handler is triggered
2. Options:
   - ABORT: Stop workflow
   - CONTINUE: Move to next step
   - RAISE_EXCEPTION: Throw error

3. Notifications sent:
   - Email alert
   - Status marked 'STALE'

4. Prevention:
   - Set realistic timeouts
   - Monitor for frequent timeouts
   - Adjust if needed"

---

## Quick Revision Checklist

### ✅ Before Interview/Exam

- [ ] **Understand basics:** Workflow = automation process
- [ ] **Know 3 types:** Participant, Process, Hybrid
- [ ] **Know 3 ways to start:** Manual, Launcher, Code
- [ ] **Know key terms:** Model, Instance, Payload, Step, Work Item
- [ ] **Understand custom processes:** Java + @Component + WorkflowProcess
- [ ] **Know console sections:** Models, Instances, Launchers
- [ ] **Can explain categorization:** DAM, WCM, CUSTOM
- [ ] **Real example ready:** Page publishing or asset processing workflow
- [ ] **Code example ready:** Custom workflow process implementation
- [ ] **Interview lines memorized:** Check Q&A section

---

### 📝 One-Line Summary for Each Topic

```
1. Workflow → Automation process that moves content through steps

2. Console → Place to manage Models, Instances, Launchers

3. Types → Participant (human), Process (auto), Hybrid (both)

4. Terms → Model (template), Instance (execution), Payload (content)

5. Execution → Manual / Launcher / Code

6. Custom Process → Java class implementing WorkflowProcess

7. Categorization → DAM, WCM, CUSTOM (organized by purpose)

8. Participant Step → Human action needed (review, approve)

9. Process Step → Automated code (generate, validate)

10. Launcher → Auto-trigger workflow on event
```

---

### 🎯 Memory Tricks

```
MODELS = Templates
  → Like recipe books

INSTANCES = Executions
  → Like cooked meals from recipes

LAUNCHER = Auto-start
  → Like alarm clock that rings automatically

CUSTOM PROCESS = Java code
  → Extends default workflow features

CATEGORIZATION = Organization
  → Group workflows by type (DAM, WCM, etc.)

PARTICIPANT STEP = Human action
  → Someone must do something

PROCESS STEP = Automatic action
  → Code handles it automatically

PAYLOAD = What being processed
  → The page, asset, or content
```

---

## Quick Reference Table

| Aspect | Details |
|--------|---------|
| **What** | Automated process for content |
| **Where** | Tools → Workflow |
| **Types** | Participant, Process, Hybrid |
| **Start Methods** | Manual, Launcher, Programmatic |
| **Key Components** | Models, Instances, Launchers |
| **Important Terms** | Model, Instance, Payload, Work Item, Step |
| **Custom Process** | Java class + @Component + WorkflowProcess |
| **Categories** | DAM, WCM, CUSTOM, PROJECTS |
| **Participant Step** | Needs human action |
| **Process Step** | Fully automated |

---

## Interview Tips

### ✅ DO's

```
✓ Always mention "Workflow = Model + Instance"
✓ Provide real examples (page approval, asset processing)
✓ Explain why workflow is useful (saves time, consistency)
✓ Show you understand the 3 types
✓ Can write basic custom process code
✓ Mention launchers for automation
✓ Talk about categorization/organization
✓ Show understanding of participant vs process steps
```

### ❌ DON'Ts

```
❌ Don't confuse Model with Instance
❌ Don't say "workflow is just for publishing"
❌ Don't forget about Launchers (auto-trigger)
❌ Don't overcomplicate your examples
❌ Don't skip mentioning audit trail benefit
❌ Don't forget about custom process creation
❌ Don't mix up Participant and Process steps
❌ Don't say transient and regular are the same
```

---

## Summary

### 🎯 Key Points to Remember

```
✓ Workflow = Automation process

✓ 3 Types:
  - Participant (human action)
  - Process (automated)
  - Hybrid (both)

✓ 3 Ways to Start:
  - Manual (click button)
  - Launcher (auto-trigger)
  - Code (Java)

✓ 3 Console Sections:
  - Models (design)
  - Instances (execution)
  - Launchers (auto-trigger)

✓ Custom Process = Java class implementing WorkflowProcess

✓ Categories = DAM, WCM, CUSTOM (organize by type)

✓ Participant Step = Human action needed

✓ Process Step = Automated code

✓ Payload = Content being processed

✓ Benefits = Save time, quality, consistency, audit trail
```

---

## Resources for Further Learning

```
Official Documentation:
  → Adobe AEM Workflows Guide
  → WorkflowProcess API
  → OSGi Component Documentation

Code Examples:
  → Adobe AEM Sample Projects
  → Open Source AEM Workflows
  → GitHub AEM Samples

Practice:
  → Create simple workflow
  → Build custom process
  → Test launchers
  → Review real workflows in your AEM instance
```

---

**Last Updated:** November 2024  
**Version:** 1.0  
**Type:** Quick Revision Guide

---

## Final Tips for Exam/Interview

```
1. Practice explaining "What is workflow?" in 30 seconds
2. Have a real example ready (page approval or asset processing)
3. Be able to write custom process code from memory
4. Understand when to use each workflow type
5. Know the difference between Model and Instance
6. Mention launchers when talking about automation
7. Explain categorization with examples
8. Show knowledge of participant vs process steps
9. Talk about benefits (time-saving, consistency)
10. Ask interviewer follow-up questions to show interest

Good luck! 🚀
```

---

*This guide is designed for quick revision before interviews and exams. Read through once, memorize key points, and you're ready to ace any AEM Workflow question!*
