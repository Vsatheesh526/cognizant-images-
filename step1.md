═══════════════════════════════════════════════════════════════════════════════
              REMOVE "hr-tags:" PREFIX & ADD SEARCH BOX
═══════════════════════════════════════════════════════════════════════════════

WHAT YOU'RE SEEING:
───────────────────────────────────────────────────────────────────────────────

Current display: "hr-tags:benefits"
Desired display: "benefits"

The prefix is the tag namespace. We need to:
1. Extract just the tag name (after the colon)
2. Add a search box
3. Keep tag-based filtering

═══════════════════════════════════════════════════════════════════════════════
SOLUTION: 3 FILES TO UPDATE
═══════════════════════════════════════════════════════════════════════════════

FILE 1: policy.html (UPDATED VERSION)
FILE 2: policies.css (UPDATED VERSION)
FILE 3: PolicyModel.java (Minor update to clean tag names)

═══════════════════════════════════════════════════════════════════════════════
STEP 1: UPDATE PolicyModel.java
═══════════════════════════════════════════════════════════════════════════════

Find this method in PolicyModel.java:

private void loadPolicies() {
    ...
    if (tagIds != null) {
        for (String tagId : tagIds) {
            Tag tag = tagManager.resolve(tagId);
            if (tag != null) {
                String tagTitle = tag.getTitle();
                policy.addTag(tagTitle);
                availableTags.add(tagTitle);
                ...
            }
        }
    }
    ...
}

CHANGE TO:

private void loadPolicies() {
    ...
    if (tagIds != null) {
        for (String tagId : tagIds) {
            Tag tag = tagManager.resolve(tagId);
            if (tag != null) {
                // Get tag name (remove namespace prefix like "hr-tags:")
                String tagTitle = tag.getTitle();
                String cleanTagName = tagTitle.contains(":") ? 
                    tagTitle.substring(tagTitle.lastIndexOf(":") + 1) : tagTitle;
                
                policy.addTag(cleanTagName);
                availableTags.add(cleanTagName);
                LOGGER.debug("Added tag '{}' to policy '{}'", cleanTagName, policy.getTitle());
            }
        }
    }
    ...
}

═══════════════════════════════════════════════════════════════════════════════
STEP 2: REPLACE policy.html
═══════════════════════════════════════════════════════════════════════════════

Use: UPDATED_policy.html

Changes:
✓ Added search box with clear button
✓ Updated tag display section
✓ Added search functionality with JavaScript
✓ Updated counter to show real-time results

═══════════════════════════════════════════════════════════════════════════════
STEP 3: REPLACE policies.css
═══════════════════════════════════════════════════════════════════════════════

Use: UPDATED_policies.css

Changes:
✓ Added search bar styling (.policies-search-bar)
✓ Added search input styling (.search-input)
✓ Added clear button styling (.search-button)
✓ Updated responsive design for search bar
✓ Improved overall layout

═══════════════════════════════════════════════════════════════════════════════
HOW IT WILL WORK:
═══════════════════════════════════════════════════════════════════════════════

SCENARIO 1: View All Policies
────────────────────────────

1. User sees all policies
2. Tags display: "benefits" (not "hr-tags:benefits")
3. Search box empty
4. Counter: "Showing 4 of 4 policies"

SCENARIO 2: Filter by Tag
─────────────────────────

1. User clicks "benefits" button
2. URL changes: ?tag=benefits
3. Only policies with benefits tag show
4. Search box available for further filtering
5. Counter updates: "Showing 2 of 4 policies"

SCENARIO 3: Search Policies
──────────────────────────

1. User types in search box: "work"
2. JavaScript searches in real-time
3. Shows only matching policies
4. Counter updates: "Showing 1 of 2 policies" (if already filtered by tag)
5. Click "Clear" to reset search

═══════════════════════════════════════════════════════════════════════════════
COMBINATION FEATURES:
═══════════════════════════════════════════════════════════════════════════════

Tag Filter + Search = Powerful Filtering

Example:
1. User clicks "benefits" filter (URL: ?tag=benefits)
2. Shows 5 policies with benefits tag
3. User types "remote" in search
4. Shows 1 policy: "Work From Home Policy" (has benefits tag + contains "remote")
5. Counter: "Showing 1 of 5 policies"

═══════════════════════════════════════════════════════════════════════════════
UPDATE PROCESS:
═══════════════════════════════════════════════════════════════════════════════

1. EDIT PolicyModel.java
   Find: private void loadPolicies()
   Update tag cleaning logic (shown above)
   Save

2. REPLACE policy.html
   Copy entire content from: UPDATED_policy.html
   Save

3. REPLACE policies.css
   Copy entire content from: UPDATED_policies.css
   Save

4. BUILD
   Terminal: mvn clean install
   
5. DEPLOY
   Terminal: mvn -PautoInstallBundle clean install

6. TEST
   http://localhost:4502/content/yoursite/policies.html
   
   Verify:
   ✓ Tags show without "hr-tags:" prefix
   ✓ Search box appears
   ✓ Clear button works
   ✓ Tag filters work
   ✓ Search + tag filters work together

═══════════════════════════════════════════════════════════════════════════════
EXPECTED RESULT:
═══════════════════════════════════════════════════════════════════════════════

Before:
[hr-tags:benefits] Work From Home Policy

After:
[benefits] Work From Home Policy

Plus search box:
┌─────────────────────────────────────┐
│ Search policies by name...  [Clear] │
└─────────────────────────────────────┘

Plus filter buttons:
[All Policies] [benefits] [compensation] [leave]

═══════════════════════════════════════════════════════════════════════════════
TROUBLESHOOTING:
═══════════════════════════════════════════════════════════════════════════════

ISSUE: Tags still show "hr-tags:benefits"
SOLUTION:
  1. Check PolicyModel.java has the cleaning logic
  2. Verify: tagTitle.substring(tagTitle.lastIndexOf(":") + 1)
  3. Rebuild: mvn clean install
  4. Clear browser cache
  5. Hard refresh: Ctrl+F5

ISSUE: Search box doesn't work
SOLUTION:
  1. Check policy.html was replaced completely
  2. Verify searchPolicies() JavaScript function exists
  3. Open browser console: F12
  4. Check for JavaScript errors
  5. Clear cache and reload

ISSUE: Clear button doesn't appear
SOLUTION:
  1. Check policies.css was updated
  2. Verify: .search-button styles present
  3. Clear browser cache
  4. Refresh page

═══════════════════════════════════════════════════════════════════════════════
FILES TO COPY:
═══════════════════════════════════════════════════════════════════════════════

1. UPDATED_policy.html
   → core/src/main/resources/jcr_root/apps/corporate-hr-portal/
     components/content/policy-list/policy.html

2. UPDATED_policies.css
   → core/src/main/resources/jcr_root/etc.clientlibs/corporate-hr-portal/
     site/css/policies.css

3. PolicyModel.java (manually update)
   → core/src/main/java/com/company/hr/core/models/PolicyModel.java
   → Update loadPolicies() method with cleaning logic

═══════════════════════════════════════════════════════════════════════════════
TESTING CHECKLIST:
═══════════════════════════════════════════════════════════════════════════════

After deployment, verify:

☐ Tags display: "benefits" (not "hr-tags:benefits")
☐ Search box visible with placeholder text
☐ Clear button visible next to search
☐ All policies show on initial load
☐ Click tag filter button → URL updates (?tag=benefits)
☐ Only matching policies show
☐ Type in search → results filter in real-time
☐ Click Clear → search resets
☐ Counter updates correctly
☐ Works on mobile (responsive)
☐ Works in different browsers

═══════════════════════════════════════════════════════════════════════════════
