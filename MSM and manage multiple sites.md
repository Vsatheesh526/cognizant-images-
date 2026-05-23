# 📘 AEM Multi Site Management (MSM) – Simple Guide

**Easy-to-Understand Guide to Managing Multiple Websites in AEM**

---

## Table of Contents

1. [Introduction to MSM](#introduction-to-msm)
2. [Blueprint](#blueprint)
3. [Live Copy](#live-copy)
4. [Language Copy](#language-copy)
5. [Rollout Configuration](#rollout-configuration)
6. [How MSM Works](#how-msm-works)
7. [Common Use Cases](#common-use-cases)
8. [Interview Q&A](#interview-qa)
9. [Quick Revision](#quick-revision)

---

## Introduction to MSM

### 🔹 What is MSM?

**MSM (Multi Site Management)** is a feature that lets you manage multiple websites from one central location.

**Simple Analogy:**
```
Without MSM:
  Website 1 → Editor 1 → Separate content
  Website 2 → Editor 2 → Separate content
  Website 3 → Editor 3 → Separate content
  Problem: Changes need to be made 3 times!

With MSM:
  Master Site (Blueprint) → Central content
       ↓
  Website 1 (Live Copy) → Auto-updated
  Website 2 (Live Copy) → Auto-updated
  Website 3 (Live Copy) → Auto-updated
  Solution: Update once, applies to all!
```

---

### ✅ Why Use MSM?

| Reason | Benefit |
|--------|---------|
| **One master source** | Edit once, update everywhere |
| **Consistency** | All sites have same content |
| **Easy updates** | Changes roll out automatically |
| **Time-saving** | No manual copy-paste |
| **Brand control** | Maintain brand standards |
| **Multi-language** | Manage different languages |
| **Multi-region** | Different sites for different countries |

---

### 📊 Real-World Scenario

```
Scenario: Global Company Website

Master Site (Headquarters):
  ├─ Company overview
  ├─ Product information
  ├─ Global policies
  └─ Brand guidelines

Live Copies (Regional Sites):
  ├─ USA Site (local content + master)
  ├─ Europe Site (local content + master)
  ├─ Asia Site (local content + master)
  └─ Automatically sync'd ✓

When master updates:
  └─ All regions auto-update
  └─ Regional info stays independent
  └─ No manual sync needed!
```

---

## Blueprint

### 🔹 What is a Blueprint?

A **Blueprint** is the master/source content that other sites inherit from.

**Think of it as:**
```
Blueprint = Template
          = Original
          = Master source
          = What others copy from
```

---

### ✅ Blueprint Characteristics

```
Blueprint:
  ├─ Master content
  ├─ Source of truth
  ├─ Can create Live Copies
  ├─ Central management
  └─ Content is "pushed" to others
```

---

### 📝 Example Blueprint Structure

```
Blueprint (Master Site)
  └─ /content/blueprint/
     ├─ /products/
     │  ├─ product-1 (Master version)
     │  ├─ product-2 (Master version)
     │  └─ product-3 (Master version)
     │
     ├─ /news/
     │  ├─ news-1 (Master version)
     │  └─ news-2 (Master version)
     │
     └─ /about/
        ├─ company-info (Master version)
        └─ contact (Master version)
```

---

### 🔍 Blueprint Properties

```
In AEM UI:
  Page → Properties → Blueprint

Settings:
  ├─ This is a Blueprint page ✓
  ├─ Allowed blueprint configs
  │  └─ Used for rollout
  └─ Rollout triggers
     ├─ On Publish
     ├─ On Modification
     └─ Manual
```

---

### 📋 Step-by-Step: Create Blueprint

```
1. Create a site:
   /content/my-blueprint/
   ├─ /products/
   ├─ /news/
   └─ /about/

2. Mark as Blueprint:
   Navigate to root page (/content/my-blueprint)
   └─ Right-click → Properties
   └─ Check "This is a Blueprint page"
   └─ Click "Save"

3. Create Live Copies:
   (Will explain in next section)
```

---

## Live Copy

### 🔹 What is a Live Copy?

A **Live Copy** is a copy of blueprint content that auto-syncs with the blueprint.

**Key difference:**
```
Regular Copy:
  ├─ Independent
  ├─ No connection to original
  ├─ Changes don't sync
  └─ Manual updates needed

Live Copy:
  ├─ Connected to blueprint
  ├─ Auto-syncs on changes
  ├─ Can have local edits
  └─ Rollout updates changes
```

---

### ✅ Live Copy Features

```
Live Copy:
  ├─ Inherits from Blueprint
  ├─ Auto-updated via rollout
  ├─ Can have local changes
  ├─ Shows inherited content
  ├─ Can be edited locally
  ├─ Relationship to Blueprint
  └─ Status indicators
```

---

### 📝 Example: Live Copy Structure

```
Blueprint (Master)
  /content/my-blueprint/
    └─ /products/
       └─ product-1 (Master)

Live Copy 1 (USA)
  /content/usa-site/
    └─ /products/
       └─ product-1 (Synced from Blueprint)

Live Copy 2 (Europe)
  /content/europe-site/
    └─ /products/
       └─ product-1 (Synced from Blueprint)

When Blueprint updates:
  └─ Rollout triggers
  └─ Syncs to all Live Copies ✓
```

---

### 📊 Visual: Blueprint to Live Copy

```
BLUEPRINT (Master)
┌─────────────────────┐
│ /content/blueprint/ │
│                     │
│ Product Info:       │
│ - Name: XYZ Product │
│ - Price: $100       │
│ - Features: 5       │
└─────────────────────┘

Create Live Copy
        ↓
        ↓
LIVE COPY (USA)              LIVE COPY (EU)
┌─────────────────────┐     ┌─────────────────────┐
│ /content/usa-site/  │     │ /content/eu-site/   │
│                     │     │                     │
│ Product Info:       │     │ Product Info:       │
│ - Name: XYZ Product │     │ - Name: XYZ Product │
│ - Price: $100       │     │ - Price: €90        │
│ + Local: "50% OFF"  │     │ - Features: 5       │
└─────────────────────┘     └─────────────────────┘

When Blueprint updates:
  └─ Rollout triggers
  └─ Syncs to all Live Copies
```

---

### 🔍 Live Copy Status Indicators

```
In AEM Page:

SYNCHRONIZED:
  ├─ Green indicator ✓
  ├─ Content matches blueprint
  └─ Ready for rollout

OUT OF SYNC:
  ├─ Orange indicator ⚠️
  ├─ Blueprint changed
  ├─ Live copy hasn't updated
  └─ Rollout pending

CONFLICT:
  ├─ Red indicator ❌
  ├─ Local edits conflict with blueprint
  ├─ Manual resolution needed
  └─ Rollout blocked
```

---

### 📋 Step-by-Step: Create Live Copy

```
Method 1: From Blueprint Page

1. Go to Blueprint page:
   /content/my-blueprint/products/product-1

2. Right-click page → Create → Create Live Copy

3. Configure:
   ├─ Live Copy path: /content/usa-site/products/
   ├─ Title: "Product 1"
   ├─ Rollout config: Select one
   └─ Click "Create"

4. Live Copy created!
   └─ /content/usa-site/products/product-1
   └─ Now synced with blueprint

Method 2: From Sites Console

1. Tools → Sites → Live Copy Overview

2. Select Blueprint

3. Create → Live Copy
   └─ Enter path
   └─ Configure
   └─ Create
```

---

### ✅ Live Copy Behavior

```
After creating Live Copy:

Content inherited:
  ├─ Page structure
  ├─ Content blocks
  ├─ Images
  ├─ Text
  └─ Properties

Local changes allowed:
  ├─ Edit text (local)
  ├─ Add components
  ├─ Change images (local)
  └─ Add local content

On rollout:
  ├─ Blueprint changes applied
  ├─ Local edits preserved
  ├─ Conflicts highlighted
  └─ Manual resolution if needed
```

---

## Language Copy

### 🔹 What is a Language Copy?

A **Language Copy** is a Live Copy created for a different language.

**Scenario:**
```
Blueprint (English):
  /content/my-site/en/

Language Copies:
  ├─ /content/my-site/de/ (German)
  ├─ /content/my-site/fr/ (French)
  ├─ /content/my-site/es/ (Spanish)
  └─ /content/my-site/ja/ (Japanese)

Structure inherited from English blueprint
Content translated to local language
```

---

### ✅ Language Copy Use Case

```
Company Website:

Blueprint (English):
  /content/company/en/
    ├─ /products/
    ├─ /news/
    └─ /about/

Language Copies:
  
  German Site:
    /content/company/de/
      ├─ /products/ (German content)
      ├─ /news/ (German content)
      └─ /about/ (German content)
  
  French Site:
    /content/company/fr/
      ├─ /products/ (French content)
      ├─ /news/ (French content)
      └─ /about/ (French content)

When English blueprint updates:
  └─ Structure syncs to all languages
  └─ German keeps German content
  └─ French keeps French content
  └─ No need to re-translate!
```

---

### 📝 Step-by-Step: Create Language Copy

```
1. Blueprint ready:
   /content/company/en/

2. Create Language Copy:
   Page → Create → Language Copy

3. Configure:
   ├─ Target language: Select (e.g., German)
   ├─ Target path: /content/company/de/
   ├─ Rollout config: Select
   └─ Click "Create"

4. German site created!
   └─ /content/company/de/
   └─ Same structure as English
   └─ Ready for German content

5. Translate content:
   └─ Open German pages
   └─ Translate text
   └─ Keep images same
   └─ Local edits (won't be overwritten)
```

---

### 🔍 Language Copy Characteristics

```
Language Copy:
  ├─ Inherits structure from blueprint
  ├─ Different language content
  ├─ Separate translation
  ├─ Maintains relationships
  ├─ Can have local changes
  ├─ Updates sync structure only
  └─ Content remains independent
```

---

## Rollout Configuration

### 🔹 What is Rollout Configuration?

**Rollout Configuration** defines how changes from blueprint sync to live copies.

**Think of it as:**
```
Rollout Config = Rules for syncing
               = What to update
               = When to update
               = How to handle conflicts
```

---

### ✅ Rollout Configuration Settings

```
Standard Rollout Config:

Name: "Standard Rollout Config"

Triggers:
  ├─ On Publish
  │  └─ Update when blueprint page published
  ├─ On Modification
  │  └─ Update when blueprint page modified
  └─ On Scheduled Activation
     └─ Update on schedule

Actions:
  ├─ Update
  │  └─ Replace live copy content with blueprint
  ├─ Create
  │  └─ Create new pages in live copy
  ├─ Delete
  │  └─ Remove deleted blueprint pages
  └─ Sync
     └─ Synchronize properties
```

---

### 📊 Types of Rollout Configurations

| Config | Trigger | Behavior |
|--------|---------|----------|
| **Standard** | On Publish | Updates on blueprint publish |
| **Standard (Advanced)** | On Modification | Updates on any change |
| **Custom** | Your choice | User-defined rules |

---

### 🔍 How Rollout Works

```
Step 1: Blueprint changes
        └─ Author edits page
        └─ Publishes page

Step 2: Rollout config triggered
        └─ "On Publish" detected
        └─ Rollout process starts

Step 3: Changes analyzed
        ├─ What changed?
        ├─ New content?
        ├─ Deleted?
        ├─ Conflicts?

Step 4: Apply to Live Copies
        ├─ Live Copy 1: Updated ✓
        ├─ Live Copy 2: Updated ✓
        ├─ Live Copy 3: Conflict ⚠️

Step 5: Report status
        └─ Summary of updates
```

---

### 📝 Step-by-Step: Configure Rollout

```
1. Go to Blueprint page:
   /content/my-blueprint/

2. Right-click → Properties

3. Go to "Blueprint" tab

4. Configure Rollout:
   ├─ Select "Rollout Config"
   ├─ Choose:
   │  ├─ Standard
   │  ├─ Standard (Advanced)
   │  └─ Custom
   ├─ Configure triggers
   └─ Click "Save"

5. Now when you publish:
   └─ Changes auto-rollout to Live Copies
```

---

### ✅ Common Rollout Triggers

```
ON PUBLISH:
  └─ Updates when blueprint published
  └─ Most common
  └─ Controlled publishing

ON MODIFICATION:
  └─ Updates on any change
  └─ Immediate sync
  └─ No publish needed

MANUAL:
  └─ Manual rollout only
  └─ Click "Rollout" button
  └─ Full control

SCHEDULED:
  └─ Rollout at specific time
  └─ Batch updates
  └─ After hours updates
```

---

### 🎯 Rollout Actions

```
UPDATE:
  └─ Replace content
  └─ Keep local edits if not conflicting
  └─ Sync structure

CREATE:
  └─ Create new pages
  └─ Add components
  └─ Copy from blueprint

DELETE:
  └─ Remove deleted pages
  └─ Clean up
  └─ Maintain consistency

SYNC:
  └─ Synchronize properties
  └─ Metadata sync
  └─ Settings sync
```

---

## How MSM Works

### 🔹 Complete MSM Flow

```
Step 1: Create Blueprint
        └─ /content/my-blueprint/
        └─ Mark as Blueprint page
        └─ Create content

Step 2: Create Live Copies
        └─ Select Blueprint page
        └─ Create → Create Live Copy
        └─ Choose destinations
        └─ Select Rollout Config

Step 3: Live Copies Created
        ├─ /content/usa-site/
        ├─ /content/europe-site/
        └─ /content/asia-site/

Step 4: Edit Blueprint
        └─ Update content in blueprint
        └─ Publish blueprint

Step 5: Rollout Triggered
        └─ Rollout config detects change
        └─ Auto or manual trigger
        └─ Changes propagate

Step 6: Live Copies Updated
        ├─ USA site: Updated ✓
        ├─ Europe site: Updated ✓
        └─ Asia site: Updated ✓

Result: All sites in sync!
```

---

### 📊 MSM Relationship

```
BLUEPRINT
  └─ Source of truth
  └─ Content authored here
  └─ Push changes out

         ↓ Rollout

LIVE COPY 1              LIVE COPY 2              LIVE COPY 3
  └─ Inherits            └─ Inherits              └─ Inherits
  └─ Syncs on rollout    └─ Syncs on rollout      └─ Syncs on rollout
  └─ Can edit locally    └─ Can edit locally      └─ Can edit locally
```

---

### 🔄 Edit Options

```
In Live Copy:

INHERITED CONTENT:
  ├─ From blueprint
  ├─ Shows indicator
  ├─ Can cancel inheritance
  ├─ Edit locally if needed
  └─ Rollout will update

LOCAL CONTENT:
  ├─ Added locally
  ├─ Not from blueprint
  ├─ Won't be affected by rollout
  ├─ Always editable
  └─ Independent

HANDLING CONFLICTS:
  └─ Resolve manually
  └─ Choose: Keep local or use blueprint
  └─ Then proceed with rollout
```

---

## Common Use Cases

### 📌 Use Case 1: Global Company Website

```
Scenario: Company with offices worldwide

Blueprint (Headquarters):
  /content/headquarters/
    ├─ /products/ (Global)
    ├─ /news/ (Global)
    ├─ /company/ (Global)
    └─ /resources/ (Global)

Live Copies (Regional):
  ├─ /content/usa/
  │  └─ Structure from blueprint
  │  └─ Local phone number
  │  └─ Local address
  │  └─ Local language/pricing
  │
  ├─ /content/europe/
  │  └─ Structure from blueprint
  │  └─ Local phone number
  │  └─ Local address
  │  └─ Local language/pricing
  │
  └─ /content/asia/
     └─ Structure from blueprint
     └─ Local phone number
     └─ Local address
     └─ Local language/pricing

Benefit:
  ✓ Product info same globally
  ✓ Company news syncs everywhere
  ✓ Local contact info independent
  ✓ One blueprint update = Global sync
```

---

### 📌 Use Case 2: Multi-Language Website

```
Scenario: Company website in 5 languages

Blueprint (English):
  /content/mysite/en/
    ├─ /products/
    ├─ /blog/
    └─ /support/

Language Copies:
  ├─ /content/mysite/de/ (German)
  ├─ /content/mysite/fr/ (French)
  ├─ /content/mysite/es/ (Spanish)
  ├─ /content/mysite/ja/ (Japanese)
  └─ /content/mysite/zh/ (Chinese)

When English blueprint updates:
  └─ All language sites get same structure
  └─ Content stays in each language
  └─ No re-translation of structure

Benefit:
  ✓ Consistent structure in all languages
  ✓ Update once, applies to all
  ✓ Content independent (translate locally)
  ✓ Translation teams work independently
```

---

### 📌 Use Case 3: Multi-Brand Website

```
Scenario: Company with multiple brands

Master Blueprint:
  /content/master/
    ├─ Brand guidelines
    ├─ Global policies
    ├─ Company info
    └─ Contact info

Brand Live Copies:
  ├─ /content/brand-a/ (Premium brand)
  │  ├─ Same structure
  │  ├─ Brand A styling
  │  ├─ Brand A products
  │  └─ Brand A messaging
  │
  ├─ /content/brand-b/ (Budget brand)
  │  ├─ Same structure
  │  ├─ Brand B styling
  │  ├─ Brand B products
  │  └─ Brand B messaging
  │
  └─ /content/brand-c/ (Eco brand)
     ├─ Same structure
     ├─ Brand C styling
     ├─ Brand C products
     └─ Brand C messaging

Benefit:
  ✓ Consistent branding across brands
  ✓ Global policies enforce everywhere
  ✓ Each brand independent in details
  ✓ Policy updates apply to all brands
```

---

## Interview Q&A

### ❓ Q1: What is MSM?

**Answer:**
"MSM (Multi Site Management) is a feature that lets you manage multiple websites from one central location.

Key points:
- One master (Blueprint)
- Multiple copies (Live Copies)
- Auto-sync of changes
- Reduce manual work
- Maintain consistency

Example: Company website with USA, Europe, and Asia sites. Update master once, all sites update automatically."

---

### ❓ Q2: What is a Blueprint?

**Answer:**
"A Blueprint is the master/source content in MSM.

Characteristics:
- Master version
- Source of truth
- Content is pushed to others
- Can create Live Copies
- Central management point

When Blueprint updates → Changes rollout to all Live Copies"

---

### ❓ Q3: What is a Live Copy?

**Answer:**
"A Live Copy is a copy of Blueprint content that auto-syncs.

Key differences:
- Connected to blueprint (not independent)
- Inherits structure/content
- Can have local edits
- Auto-updates via rollout
- Shows sync status

Example: USA site is Live Copy of master blueprint. When master updates, USA site auto-updates."

---

### ❓ Q4: Difference between Live Copy and regular copy?

**Answer:**
"Regular Copy:
- Independent copy
- No connection to original
- Manual updates only
- Completely separate

Live Copy:
- Linked to blueprint
- Auto-syncs on changes
- Can have local edits
- Maintains relationship"

---

### ❓ Q5: What is Language Copy?

**Answer:**
"Language Copy is a Live Copy for different languages.

Use case:
- Company in multiple countries
- Each country has own language site
- All inherit structure from blueprint
- Content translated independently

Example:
- Blueprint (English): /en/
- Language Copy (German): /de/
- Language Copy (French): /fr/
- When English updates, all get structure"

---

### ❓ Q6: What is Rollout Configuration?

**Answer:**
"Rollout Configuration defines how changes from Blueprint sync to Live Copies.

Settings:
- When to rollout (trigger)
- What to sync (actions)
- How to handle conflicts

Triggers:
- On Publish
- On Modification
- Manual
- Scheduled

When Blueprint changes → Rollout applies changes to Live Copies"

---

### ❓ Q7: What are Rollout Triggers?

**Answer:**
"Rollout Triggers decide when to sync Blueprint changes to Live Copies.

Options:
1. On Publish - When you click 'Publish'
2. On Modification - Any change detected
3. Manual - You click 'Rollout' button
4. Scheduled - At specific time

Most common: On Publish (controlled updates)"

---

### ❓ Q8: How to create a Live Copy?

**Answer:**
"Steps:
1. Navigate to Blueprint page
2. Right-click → Create → Create Live Copy
3. Choose destination path
4. Select Rollout Config
5. Click 'Create'

Result: New Live Copy created at destination
- Inherits blueprint structure/content
- Auto-syncs on rollout
- Can be edited locally"

---

### ❓ Q9: What happens when Blueprint updates?

**Answer:**
"When Blueprint updates:

1. Trigger fires (On Publish, etc.)
2. Rollout process starts
3. Changes analyzed
4. Applied to Live Copies
5. Conflicts highlighted
6. Status reported

Result:
- Live Copies updated ✓
- Local edits preserved
- Structure in sync
- Content same (if no conflicts)"

---

### ❓ Q10: What are Rollout Actions?

**Answer:**
"Rollout Actions define what to update:

1. Update - Replace content
2. Create - Add new pages
3. Delete - Remove pages
4. Sync - Update properties

These actions apply Blueprint changes to Live Copies during rollout."

---

## Quick Revision

### ✅ MSM Basics

```
1. What? Manage multiple sites from one place
2. How? Blueprint + Live Copies + Rollout
3. Blueprint? Master/source content
4. Live Copy? Copy that syncs with blueprint
5. Language Copy? Live Copy for different language
6. Rollout? Process of syncing changes
7. Trigger? What causes rollout
8. Action? What gets synced
9. Sync? All copies updated at once
10. Local? Edits that don't get overwritten
```

---

### 📝 Key Concepts

```
BLUEPRINT:
  ├─ Master site
  ├─ Source of truth
  ├─ Content authored here
  └─ Changes pushed out

LIVE COPY:
  ├─ Copy of blueprint
  ├─ Auto-updates via rollout
  ├─ Can edit locally
  └─ Inherits structure

LANGUAGE COPY:
  ├─ Live Copy for language
  ├─ Inherits structure
  ├─ Different language content
  └─ Independent translations

ROLLOUT:
  ├─ Sync process
  ├─ Blueprint → Live Copies
  ├─ Triggered automatically/manually
  └─ Updates all at once
```

---

### 🎯 Memory Tricks

```
MSM = Multiple Sites from one Master
BLUEPRINT = Master (source)
LIVE COPY = Synced copy (not independent)
LANGUAGE COPY = Multi-language support
ROLLOUT = Push changes to copies
TRIGGER = What causes rollout
ACTION = What gets synced
INHERITED = From blueprint (syncs)
LOCAL = Edited locally (doesn't sync)
CONFLICT = Blueprint vs Local edit (manual resolution)
```

---

### 📊 Visual: MSM Structure

```
              BLUEPRINT
              (Master)
                  │
        ┌─────────┼─────────┐
        │         │         │
    Live Copy1  Live Copy2  Live Copy3
    (USA)       (Europe)    (Asia)
    
    Synced ←──→ Synced ←──→ Synced
```

---

### ✅ Checklist: Setting Up MSM

```
☐ Create Blueprint site
  ☐ /content/my-blueprint/
  ☐ Create content
  ☐ Mark as Blueprint

☐ Configure Rollout Config
  ☐ Choose trigger (On Publish)
  ☐ Choose actions (Update, Create)
  ☐ Save config

☐ Create Live Copies
  ☐ Select Blueprint page
  ☐ Create → Live Copy
  ☐ Choose destination
  ☐ Select Rollout Config
  ☐ Create

☐ Test Rollout
  ☐ Edit Blueprint
  ☐ Publish
  ☐ Check Live Copies updated
  ☐ Verify sync worked

☐ Add Local Content
  ☐ Edit Live Copy locally
  ☐ Add local content
  ☐ Test: Rollout doesn't overwrite
```

---

## Summary

| Concept | Explanation |
|---------|-------------|
| **MSM** | Manage multiple sites from one master |
| **Blueprint** | Master site (source of truth) |
| **Live Copy** | Copy that syncs with blueprint |
| **Language Copy** | Live Copy for different language |
| **Rollout** | Process of syncing changes |
| **Trigger** | What causes rollout (On Publish, etc.) |
| **Action** | What gets synced (Update, Create, etc.) |
| **Inherited** | From blueprint (syncs automatically) |
| **Local** | Edited locally (independent) |
| **Sync** | All copies updated at once |

---

## DO's & DON'Ts

### ✅ DO's

```
✓ Use blueprint for common content
✓ Use local edits for regional content
✓ Test rollout before going live
✓ Plan blueprint structure well
✓ Use appropriate rollout config
✓ Monitor sync status
✓ Document blueprint relationships
✓ Backup before major rollouts
```

### ❌ DON'Ts

```
❌ Don't edit blueprint and live copy same page
❌ Don't change rollout config without testing
❌ Don't ignore sync conflicts
❌ Don't replicate if using MSM
❌ Don't create circular relationships
❌ Don't delete blueprint without warning
❌ Don't ignore inheritance indicators
❌ Don't skip rollout config selection
```

---

## Interview Tips

```
1. Explain flow: Blueprint → Live Copy → Rollout
2. Know difference: Blueprint (master) vs Live Copy (synced)
3. Understand triggers: When does rollout happen?
4. Explain actions: What gets synced?
5. Know use cases: Multi-site, multi-language, multi-brand
6. Talk about local edits: How to customize
7. Mention conflicts: Manual resolution
8. Show you understand inheritance
9. Know where to access: Tools → Sites → Live Copy Overview
10. Give real example
```

---

## Common Use Cases Summary

```
Multi-Language:
  └─ English blueprint
  └─ German, French, Spanish language copies
  └─ Same structure, different content

Multi-Regional:
  └─ Global blueprint
  └─ USA, Europe, Asia live copies
  └─ Same products, local contact info

Multi-Brand:
  └─ Master blueprint
  └─ Brand A, Brand B, Brand C live copies
  └─ Same policies, different branding
```

---

**Last Updated:** November 2024  
**Version:** 1.0  
**Type:** Simple Learning Guide

---

## Key Takeaways

✅ MSM = Manage multiple sites from one master  
✅ Blueprint = Master/source site  
✅ Live Copy = Synced copy (auto-updates)  
✅ Language Copy = Multi-language support  
✅ Rollout = Push changes to copies  
✅ Trigger = What causes rollout  
✅ Action = What gets synced  
✅ Local = Independent edits  
✅ Inherited = From blueprint  
✅ Conflict = Manual resolution needed  

---

## Command Reference

```bash
# Access Live Copy Overview
http://localhost:4502/sites.html/content/dam

# View blueprint relationships
Tools → Sites → Live Copy Overview

# Check rollout config
Tools → Deployment → Rollout Configs

# Trigger manual rollout
Page → Rollout
```

---

## Final Tips

```
1. Blueprint = Think "master template"
2. Live Copy = Think "synced copy"
3. Rollout = Think "push to all"
4. Local = Think "unique to this site"
5. Language Copy = Think "same + translated"
6. Use MSM for consistency
7. Reduce manual sync work
8. Automate updates
9. Maintain quality
10. Scale efficiently
```

---

*Perfect for quick learning and interview prep! Master MSM and manage multiple sites like a pro!* 🚀
