# 📘 AEM Replication – Simple Guide

**Easy-to-Understand Guide to AEM Content Replication**

---

## Table of Contents

1. [Introduction to Replication](#introduction-to-replication)
2. [Replication Agent](#replication-agent)
3. [Author-Publisher Replication](#author-publisher-replication)
4. [Reverse Replication](#reverse-replication)
5. [Flush Agent](#flush-agent)
6. [Replication Concepts](#replication-concepts)
7. [Common Issues & Solutions](#common-issues--solutions)
8. [Interview Q&A](#interview-qa)
9. [Quick Revision](#quick-revision)

---

## Introduction to Replication

### 🔹 What is Replication?

**Replication** is copying content from one AEM instance to another.

**Simple Example:**
```
Author Instance (where you create)
    ↓ Replicate (copy)
    ↓
Publisher Instance (where users see)
```

---

### ✅ Why Replication?

| Reason | Benefit |
|--------|---------|
| **Publish content** | Move from author to users |
| **Keep in sync** | Both instances have same data |
| **Load balance** | Distribute users across instances |
| **Backup** | Copy data to another location |
| **Multi-site** | Distribute to multiple servers |

---

### 📊 Real-World Scenario

```
Scenario: Blog Publishing

Author (localhost:4502)
  ├─ Writer creates article
  ├─ Editor reviews
  ├─ Manager approves
  ├─ Click "Publish" button
  │
  └─> Replication Agent
      ├─ Captures content
      ├─ Sends to Publisher
      └─ Updates Dispatcher Cache
      
Publisher (localhost:4503)
  └─ Article visible to users
     (100,000 visitors can read)
```

---

### 🎯 Key Concept

```
WITHOUT Replication:
  Author = Editor
  Publisher = Editor
  Problem: Users see in-progress work!

WITH Replication:
  Author = Editing space (safe)
  Publisher = Public space (clean)
  Solution: Only published content visible!
```

---

## Replication Agent

### 🔹 What is a Replication Agent?

A **Replication Agent** is a component that copies content from one place to another.

**Think of it as:**
```
Replication Agent = Messenger
                  = Copies content
                  = Sends to destination
                  = Reports status
```

---

### ✅ How Replication Agent Works

```
Step 1: Content is created/modified
        ↓
Step 2: Replication Agent detects change
        ↓
Step 3: Agent packages content
        ↓
Step 4: Agent sends to destination
        ↓
Step 5: Destination receives
        ↓
Step 6: Agent confirms delivery
        ↓
Replication Complete ✓
```

---

### 📍 Where to Configure

```
Path: Tools → Deployment → Replication

Or direct URL:
http://localhost:4502/etc/replication/agents.author.html
```

---

### 🔍 Replication Agent Settings

```
Agent Configuration:

Name: "Publish"
├─ Description: "Publish to production"
├─ Enabled: ✓
├─ Serialize Type: Default
├─ Log Level: Info
│
├─ Transport:
│  ├─ URI: http://localhost:4503/bin/receive?sling:authRequestLogin=1
│  ├─ User: admin
│  ├─ Password: admin
│
├─ Triggers:
│  ├─ Ignore default (auto-detect)
│  ├─ Ignore versioning
│
└─ Advanced:
   ├─ Max retry delay: 60000
   ├─ Number of retries: 4
```

---

### 📝 Agent Types

| Agent Type | Purpose | Use Case |
|-----------|---------|----------|
| **Publish** | Send to Publisher | Main replication |
| **Reverse** | Send back to Author | Reverse replication |
| **Flush** | Clear Dispatcher cache | After publishing |
| **Dynamic Agents** | Temporary agents | On-demand replication |

---

### ✅ Agent Status

```
ENABLED ✓ = Ready to replicate
DISABLED ✗ = Not working
BLOCKED ⚠️ = Queue is full
ERROR ❌ = Connection problem

Check status:
Tools → Deployment → Replication → Agents
└─ Look at Status column
```

---

## Author-Publisher Replication

### 🔹 What is It?

**Author-Publisher Replication** is copying content from Author instance to Publisher instance.

```
Author (Creation Environment)
  ├─ Editors work here
  ├─ Content created
  ├─ Content edited
  └─ Content approved
  
  ↓ REPLICATE (copy)
  
Publisher (Delivery Environment)
  ├─ Users visit
  ├─ See published content only
  └─ Don't see draft/unpublished
```

---

### ✅ How Author-Publisher Works

#### Step 1: Author Creates Content

```
Author Instance (localhost:4502)
  └─ Editor creates page: /content/my-site/hello
     ├─ Title: "Hello World"
     ├─ Content: "Welcome to my site"
     └─ Status: Draft (not published)

At this point:
  ✓ Visible in Author
  ✗ NOT visible to users
```

---

#### Step 2: Author Publishes Content

```
In AEM Editor:
  └─ Click "Page" menu
     └─ Select "Publish Page"
        └─ Click "Publish"

What happens:
  1. Content packed
  2. Sent to Publisher
  3. Content activated (published)
```

---

#### Step 3: Replication Queue

```
Replication Queue (automatic):
  1. Get request to publish page
  2. Page sent to Publish agent
  3. Publish agent sends to Publisher
  4. Publisher confirms receipt
  5. Queue cleared

If fails:
  └─ Content stays in queue
     └─ Retry automatically
```

---

#### Step 4: Publisher Receives

```
Publisher Instance (localhost:4503)
  └─ Page received: /content/my-site/hello
     ├─ Title: "Hello World"
     ├─ Content: "Welcome to my site"
     └─ Status: Published

At this point:
  ✓ Visible in Publisher
  ✓ Visible to users (www.example.com/hello)
```

---

### 📊 Visual Flow

```
AUTHOR (Editorial)              PUBLISHER (Public)

Editor creates page
    ↓
Editor reviews
    ↓
Manager approves
    ↓
Click "Publish"
    ↓
[Replication Agent]
    ├─ Packages content
    ├─ Sends HTTP request
    ├─ Destination: /bin/receive
    └─ Confirms delivery
    
                                 ← Receives content
                                 ← Activates page
                                 ← Users see page

Users visit:
www.example.com/hello → Served from Publisher
```

---

### 📝 Step-by-Step: Publish a Page

```
1. In AEM Author:
   Navigate → Pages
   └─ Find your page
   
2. Click "Page" menu (top-left)
   
3. Select "Publish Page"
   └─ Or "Manage Publication"
   
4. Dialog appears:
   ├─ Select pages to publish
   ├─ Choose publication date (now or later)
   └─ Click "Publish" button
   
5. Status message:
   "Page published successfully"
   
6. Check Replication Queue:
   Tools → Deployment → Replication
   └─ Should be empty (successful)
```

---

### ✅ Verification

**Check if Published:**

```
On Author:
  └─ Page should have "Published" indicator
     └─ Small green checkmark or indicator
     
On Publisher:
  └─ Page should be visible
     └─ Without /author/ in URL
     └─ Without edit mode
```

---

## Reverse Replication

### 🔹 What is It?

**Reverse Replication** is sending content from Publisher BACK to Author.

```
Unusual scenario, but useful for:
  ├─ Copy user-generated content back
  ├─ Sync from Publisher to Author
  ├─ Distribute modifications from Publisher
```

---

### ✅ How Reverse Replication Works

```
Normal (Author → Publisher):
Author creates → Publishes → Publisher shows

Reverse (Publisher → Author):
Publisher has data → Sends to Author

Example:
  ├─ Users submit forms on Publisher
  ├─ Data collected on Publisher
  ├─ Reverse replicate form data to Author
  └─ Author backs up user data
```

---

### 📍 Setup Reverse Agent

```
Path: Tools → Deployment → Replication

Create Reverse Agent:
├─ Name: "Reverse"
├─ Description: "Send from Publisher to Author"
├─ Enabled: ✓
│
└─ Transport:
   ├─ URI: http://localhost:4502/bin/receive?sling:authRequestLogin=1
   ├─ User: admin
   ├─ Password: admin
   
This runs ON PUBLISHER
Points TO AUTHOR
```

---

### 📝 Example: User Form Data

```
Scenario: Contact form on Publisher

Publisher (Production):
  1. User fills contact form
  2. Data saved to /content/forms/submissions/john
  3. Reverse agent picks it up
  4. Sends to Author
  
Author (Backup):
  5. Receives form submission
  6. Stored at /content/forms/submissions/john
  7. Backup complete ✓

Author can:
  └─ Review submissions
  └─ Export for CRM
  └─ Analyze feedback
```

---

### ⚠️ Important Points

```
❌ DON'T reverse replicate published pages
   → Would overwrite author draft
   → Causes confusion

✅ DO reverse replicate:
   ├─ Form submissions
   ├─ User-generated content
   ├─ Data from Publisher
   └─ Non-editorial content
```

---

## Flush Agent

### 🔹 What is It?

A **Flush Agent** clears the Dispatcher cache after publishing.

```
What happens:
  1. Content published to Publisher
  2. Dispatcher Cache still has old content
  3. Users see OLD content (not updated!)
  
Solution:
  └─ Flush Agent
     └─ Clears Dispatcher cache
     └─ Forces new content to load
```

---

### ✅ Why Flush Agent?

```
WITHOUT Flush Agent:
  1. Content published
  2. Dispatcher still shows old version
  3. Users see old content
  4. Wait 1 hour for cache to expire
  Problem: Users see stale content!

WITH Flush Agent:
  1. Content published
  2. Flush agent immediately clears cache
  3. Dispatcher fetches new content
  4. Users see updated content
  Solution: Instant updates! ✓
```

---

### 📍 Setup Flush Agent

```
Path: Tools → Deployment → Replication

Create Flush Agent:
├─ Name: "Flush"
├─ Description: "Flush Dispatcher cache"
├─ Enabled: ✓
│
├─ Serialization Type: Flush
│
└─ Transport:
   ├─ URI: http://localhost:4505/dispatcher/invalidate.cache
   ├─ User: admin
   ├─ Password: admin
   
This clears cache at:
  http://localhost:4505/ (Dispatcher)
```

---

### 📊 Flush Process

```
Step 1: Content Published
        └─ Page: /content/my-site/hello updated
        
Step 2: Publish Agent
        └─ Sends to Publisher: ✓
        
Step 3: Flush Agent Triggered
        ├─ Type: Flush
        ├─ Sends to: Dispatcher
        └─ Command: Clear /content/my-site/hello
        
Step 4: Dispatcher Clears Cache
        └─ Old content removed
        └─ Next request fetches new content
        
Step 5: Users See Update
        └─ New content displayed ✓
```

---

### ✅ Flush Agent Configuration

```
Key Settings:

Serialization Type: 
  └─ MUST be "Flush"
  
URI:
  └─ Points to Dispatcher
  └─ Example: http://localhost:4505/dispatcher/invalidate.cache
  
Path:
  └─ What to flush
  └─ Example: /content/my-site/*
  └─ Wildcard to flush multiple paths
```

---

### 📝 Paths to Flush

```
Flush specific page:
  └─ /content/my-site/hello

Flush entire site:
  └─ /content/my-site/*

Flush specific paths:
  └─ /content/my-site/blog/*
  └─ /content/my-site/news/*

Flush static files:
  └─ /etc/designs/my-site/*
```

---

## Replication Concepts

### 🔹 Replication Queue

**Queue** = List of content waiting to be replicated.

```
Queue Status:

EMPTY:
  └─ All content successfully replicated
  └─ No pending changes
  
ITEMS IN QUEUE:
  └─ Content waiting to send
  └─ Might be failing
  └─ Check status!
  
BLOCKED:
  └─ Connection problem
  └─ Retry attempt in progress
  └─ Check Publish agent
```

**Check Queue:**
```
Tools → Deployment → Replication
└─ Click agent (e.g., "Publish")
   └─ Look at "Queue" tab
   └─ Shows pending items
```

---

### 🔹 Activation vs Deactivation

```
ACTIVATION (Publish):
  └─ Send to Publisher
  └─ Content becomes LIVE
  └─ Users can see
  └─ Action: "Publish Page"

DEACTIVATION (Unpublish):
  └─ Remove from Publisher
  └─ Content goes OFFLINE
  └─ Users can't see
  └─ Action: "Unpublish Page"
```

---

### 📝 Example: Activate/Deactivate

```
Activate (Publish):
  Author Page
    └─ Click "Publish"
    └─ Replicated to Publisher
    └─ Live on website ✓

Deactivate (Unpublish):
  Author Page
    └─ Click "Unpublish"
    └─ Removed from Publisher
    └─ Offline from website ✗
```

---

### 🔹 Replication Status

```
Status in AEM:

✓ SUCCESS:
  └─ Content replicated successfully
  └─ No errors

⏳ QUEUED:
  └─ Waiting to send
  └─ Will send soon

⚠️ PENDING:
  └─ In progress
  └─ Currently sending

❌ FAILED:
  └─ Error occurred
  └─ Check connection
  └─ Check credentials
  └─ Retry manually
```

---

## Common Issues & Solutions

### ❓ Issue 1: Replication Failed

**Symptoms:**
```
Message: "Replication failed"
Status: RED/ERROR
Content: Not appearing on Publisher
```

**Common Causes:**
```
1. Publish agent not enabled
   → Check: Tools → Deployment → Replication
   → Look for "Enabled" checkbox

2. Wrong URI
   → Check URI points to correct Publisher
   → Example: http://localhost:4503/bin/receive

3. Wrong credentials
   → Username/password incorrect
   → Check Publisher user exists

4. Network issue
   → Firewall blocking
   → Publisher not running
   → Check connectivity
```

**Solution:**
```
1. Verify agent is ENABLED
   
2. Check URI is correct:
   └─ Tools → Replication → Click agent
   └─ Check "URI" field
   
3. Verify credentials:
   └─ Click agent
   └─ Check username/password
   └─ Test user on Publisher
   
4. Check Publisher:
   └─ Is it running?
   └─ Can you access /bin/receive?
   └─ curl http://localhost:4503/bin/receive
   
5. Check logs:
   └─ crx-quickstart/logs/error.log
   └─ Look for connection errors
```

---

### ❓ Issue 2: Queue Not Empty

**Symptoms:**
```
Content published but queue shows items
Content not appearing on Publisher
```

**Solution:**
```
1. Clear queue manually:
   └─ Tools → Deployment → Replication
   └─ Click agent (e.g., "Publish")
   └─ Click "Clear Queue"
   
2. Manually trigger replication:
   └─ Click "Test Connection"
   └─ Should show "200 OK"
   
3. If still fails:
   └─ Restart agents
   └─ Or restart AEM
```

---

### ❓ Issue 3: Old Content Showing (Cache)

**Symptoms:**
```
Content published on Publisher
But users still see old version
```

**Cause:**
```
Dispatcher cache not cleared
Flush agent not working
```

**Solution:**
```
1. Check Flush agent:
   └─ Tools → Deployment → Replication
   └─ Look for "Flush" agent
   └─ Verify it's ENABLED
   
2. Manually flush:
   └─ Click Flush agent
   └─ Go to "Test" tab
   └─ Enter path: /content/my-site/*
   └─ Click "Execute Test"
   
3. Check Dispatcher:
   └─ Is it running?
   └─ Check cache directory
   └─ crx-quickstart/dispatcher/cache
```

---

### ❓ Issue 4: Credentials Not Working

**Symptoms:**
```
Error: "401 Unauthorized"
Replication blocked
```

**Solution:**
```
1. Verify user on Publisher:
   └─ Log in to Publisher
   └─ Check if user exists
   └─ admin/admin default
   
2. Check user permissions:
   └─ User needs "jcr:write" permission
   └─ User needs "crx:replicate" permission
   
3. Update credentials in agent:
   └─ Tools → Replication
   └─ Click agent
   └─ Update username/password
   └─ Click "Save"
   
4. Test connection:
   └─ Click "Test Connection"
   └─ Should show "200 OK"
```

---

## Interview Q&A

### ❓ Q1: What is Replication?

**Answer:**
"Replication is copying content from one AEM instance to another.

Example: Copy from Author to Publisher so users can see.

Key points:
- Author = Editing space
- Publisher = Public space
- Replication = Copy mechanism
- Agents = Do the copying

Benefits:
- Separate editing from viewing
- Content control
- Load distribution"

---

### ❓ Q2: What is a Replication Agent?

**Answer:**
"A Replication Agent is a component that copies content from source to destination.

Types:
1. Publish Agent (Author → Publisher)
2. Reverse Agent (Publisher → Author)
3. Flush Agent (Clear cache)

How it works:
- Detects change
- Packages content
- Sends to destination
- Confirms delivery"

---

### ❓ Q3: Difference between Author and Publisher?

**Answer:**
"Author Instance (localhost:4502):
- Where editors create and edit
- Drafts and unpublished content
- Full editing capabilities
- Not accessible to public

Publisher Instance (localhost:4503):
- Where published content lives
- Only live, approved content
- Optimized for speed
- Accessible to public users"

---

### ❓ Q4: What is Reverse Replication?

**Answer:**
"Reverse Replication sends content from Publisher BACK to Author.

Use cases:
- Copy user-generated content
- Back up form submissions
- Sync data from Publisher
- Distribute Publisher modifications

Example:
- Users submit forms on Publisher
- Form data automatically backs up to Author
- Author can review submissions"

---

### ❓ Q5: What is a Flush Agent?

**Answer:**
"A Flush Agent clears the Dispatcher cache after publishing.

Why needed:
- Content published to Publisher
- Dispatcher still has old cached version
- Users see old content

Solution:
- Flush agent clears cache
- Forces Dispatcher to fetch new content
- Users see updated content immediately"

---

### ❓ Q6: How to publish a page?

**Answer:**
"Steps:
1. In AEM Author, open page
2. Click 'Page' menu (top-left)
3. Select 'Publish Page'
4. Select pages to publish
5. Click 'Publish' button
6. Wait for confirmation message
7. Page now live on Publisher

Check:
- Tools → Deployment → Replication
- Queue should be empty
- Page visible on Publisher (localhost:4503)"

---

### ❓ Q7: What happens if Replication fails?

**Answer:**
"If replication fails:
1. Content stays in queue
2. Page not published to Publisher
3. Users can't see content

Causes:
- Publisher not running
- Wrong credentials
- Network error
- Agent disabled
- Wrong URI

Solution:
- Check agent is enabled
- Verify credentials
- Check Publisher is running
- Check logs for error
- Manually retry in queue"

---

### ❓ Q8: How to check Replication Queue?

**Answer:**
"Steps:
1. Go to Tools → Deployment → Replication
2. Click on agent (e.g., 'Publish')
3. Go to 'Queue' tab
4. See pending items

If queue empty:
- All content replicated ✓

If queue has items:
- Content waiting to send
- Check status of each item
- See why it's pending"

---

### ❓ Q9: What's the difference between Activation and Deactivation?

**Answer:**
"Activation (Publish):
- Send content to Publisher
- Make content LIVE
- Users can see it
- Action: 'Publish Page'

Deactivation (Unpublish):
- Remove from Publisher
- Make content OFFLINE
- Users can't see it
- Action: 'Unpublish Page'"

---

### ❓ Q10: How to clear cache after publishing?

**Answer:**
"Using Flush Agent:
1. Tools → Deployment → Replication
2. Look for 'Flush' agent
3. Verify it's ENABLED
4. Should auto-trigger after publish

Manual flush:
1. Click Flush agent
2. Go to 'Test' tab
3. Enter path: /content/my-site/*
4. Click 'Execute Test'

Purpose:
- Dispatcher cache updated
- Users see new content immediately"

---

## Quick Revision

### ✅ Replication Basics

```
1. What? Copy content between instances
2. Why? Separate editing from viewing
3. From? Author (creation)
4. To? Publisher (delivery)
5. How? Replication Agents
6. Queue? List of pending items
7. Activate? Publish/make live
8. Deactivate? Unpublish/make offline
9. Reverse? Publisher → Author
10. Flush? Clear cache
```

---

### 📝 Key Points

```
AUTHOR Instance:
  ├─ localhost:4502
  ├─ Creation environment
  ├─ Editors work here
  ├─ Draft content
  └─ NOT public

PUBLISHER Instance:
  ├─ localhost:4503
  ├─ Delivery environment
  ├─ Only live content
  ├─ Public facing
  └─ Optimized for speed

REPLICATION:
  ├─ Author → Publisher
  ├─ Via agents
  ├─ Via queue
  └─ Automatic or manual

FLUSH:
  ├─ Clear cache
  ├─ After publishing
  ├─ Via Flush agent
  └─ Force content update
```

---

### 🎯 Memory Tricks

```
AUTHOR = A = Authoring (creating)
PUBLISHER = P = Public (users see)
REPLICATE = Copy from A to P
ACTIVATE = Publish (make live)
DEACTIVATE = Unpublish (go offline)
REVERSE = P → A (backward)
FLUSH = Clear cache (erase old)
QUEUE = Waiting list (pending items)
AGENT = Messenger (does copying)
DISPATCHER = Cache (stores copies)
```

---

### 📊 Visual: Replication Flow

```
AUTHOR                          PUBLISHER
  
  Page created
      ↓
  Page edited
      ↓
  Click "Publish"
      ↓
  [PUBLISH AGENT]
      ├─ Package content
      ├─ Send HTTP
      └─ Confirm
                                    ← Receive
                                    ← Activate
                                    ↓
  [FLUSH AGENT]
      ├─ Send flush command
      └─ Clear cache
                                    ← Clear dispatcher cache
                                    ↓
                                Users see update ✓
```

---

### ✅ Checklist: Setting Up Replication

```
☐ Author instance running (4502)
☐ Publisher instance running (4503)
☐ Publish agent configured
  ☐ Name: "Publish"
  ☐ Enabled: ✓
  ☐ URI: http://localhost:4503/bin/receive
  ☐ User: admin
  ☐ Password: admin
☐ Flush agent configured
  ☐ Name: "Flush"
  ☐ Enabled: ✓
  ☐ URI: http://localhost:4505/dispatcher/invalidate.cache
☐ Test connection works
☐ Replication queue empty
☐ Publish a test page
☐ Verify on Publisher
☐ Check cache cleared
```

---

## Summary

| Concept | Explanation |
|---------|-------------|
| **Replication** | Copy content between instances |
| **Agent** | Component that does copying |
| **Author** | Creation environment (editors) |
| **Publisher** | Delivery environment (users) |
| **Queue** | List of pending replications |
| **Activate** | Publish/make live |
| **Deactivate** | Unpublish/make offline |
| **Reverse** | Replicate from Publisher to Author |
| **Flush** | Clear Dispatcher cache |
| **Dispatcher** | Cache system for performance |

---

## Interview Tips

```
1. Explain flow: Author → Agent → Publisher
2. Know difference: Author (edit) vs Publisher (live)
3. Talk about queue (waiting list)
4. Mention agents (how copying happens)
5. Explain activation (publish action)
6. Know reverse replication (Publisher → Author)
7. Understand flush (cache clearing)
8. Know common issues (failed replication)
9. Show you can troubleshoot
10. Mention Dispatcher caching
```

---

## DO's & DON'Ts

### ✅ DO's

```
✓ Check queue regularly
✓ Verify both instances running
✓ Test agent connections
✓ Flush after publish
✓ Monitor for failures
✓ Keep credentials secure
✓ Document agent setup
✓ Review replication logs
```

### ❌ DON'Ts

```
❌ Publish to wrong instance
❌ Ignore queue warnings
❌ Leave agents disabled
❌ Use weak passwords
❌ Forget to clear cache
❌ Replicate draft content
❌ Reverse replicate pages
❌ Ignore error messages
```

---

## Command Reference

```bash
# Check replication logs
tail -f crx-quickstart/logs/error.log

# Check queue status
curl http://localhost:4502/etc/replication.html

# Manual replication (curl)
curl -u admin:admin -X POST \
  -d "cmd=Activate&path=/content/my-page" \
  http://localhost:4502/bin/replicate.json

# Test agent connection
curl http://localhost:4503/bin/receive

# Clear cache (manual)
curl -u admin:admin \
  http://localhost:4505/dispatcher/invalidate.cache?glob=/*
```

---

## Final Tips

```
1. Replication = Just copying content!
2. Author = Safe space for editing
3. Publisher = Live space for users
4. Agents = Messengers (do the work)
5. Queue = Waiting room (pending items)
6. Flush = Refresh cache (clear old)
7. Test before go-live
8. Monitor regularly
9. Document setup
10. Know troubleshooting
```

---

**Last Updated:** November 2024  
**Version:** 1.0  
**Type:** Simple Learning Guide

---

## Key Takeaways

✅ Replication = Copy content between instances  
✅ Author = Editing space  
✅ Publisher = Public space  
✅ Agent = Does the copying  
✅ Queue = Waiting list for replications  
✅ Activate = Publish (make live)  
✅ Deactivate = Unpublish (go offline)  
✅ Reverse = Publisher → Author  
✅ Flush = Clear cache for updates  
✅ Dispatcher = Cache system  

---

*Perfect for quick revision and interviews! Keep it simple, understand the flow, and you've got replication! 🚀*
