# Building a Corporate HR Portal with Adobe Experience Manager (AEM)

This guide walks you through building a real-world **HR Portal** on AEM, covering project setup in VS Code, editable templates, reusable components, Content Fragments, Experience Fragments, QueryBuilder, Sling Models, OSGi services, tagging, and user permissions. Everything is explained step by step, with code snippets and best practices.

---

## 1. Prerequisites

- AEM 6.5 or AEM as a Cloud Service SDK running locally
- Java 11 (or 8 for AEM 6.5)
- Apache Maven 3.6+
- VS Code with **AEM Developer Tools** extension (optional but recommended)
- Basic understanding of AEM, Maven, HTL, and Sling

---

## 2. Project Setup Using the AEM Maven Archetype

Open a terminal and generate the project:

```bash
mvn -B archetype:generate \
 -D archetypeGroupId=com.adobe.aem \
 -D archetypeArtifactId=aem-project-archetype \
 -D archetypeVersion=35 \
 -D appTitle="Corporate HR Portal" \
 -D appId="hrportal" \
 -D groupId="com.corp.hr" \
 -D artifactId="aem-hr-portal" \
 -D package="com.corp.hr.portal" \
 -D aemVersion="cloud"   # or "6.5.0" for on‑prem
```

After generation, open the `aem-hr-portal` folder in VS Code.

---

## 3. Project Folder Structure Explained

The archetype creates a multi‑module Maven project. The key modules we’ll work with:

| Module         | Purpose |
|----------------|---------|
| **core**       | Java bundles – Sling Models, OSGi services, servlets |
| **ui.apps**    | Component definitions (HTL scripts, dialogs, clientlibs), editable templates |
| **ui.content** | Initial content packages – sample pages, Content Fragment models, configurations |
| **ui.config**  | OSGi configurations (optional) |
| **ui.tests**   | Integration tests |

**Where to put code:**
- Java classes: `core/src/main/java/com/corp/hr/portal/core/…`
- HTL scripts and `cq:dialog`: `ui.apps/src/main/content/jcr_root/apps/hrportal/components/…`
- Editable templates: `ui.apps/src/main/content/jcr_root/conf/hrportal/settings/wcm/templates/…`
- Content Fragment Models: `ui.content/src/main/content/jcr_root/conf/hrportal/settings/dam/cfm/models/…`
- Sample content: `ui.content/src/main/content/jcr_root/content/hrportal/…`

---

## 4. Module 1: Reusable Header and Footer with Experience Fragments

Experience Fragments (XF) allow you to create a piece of content once and reuse it across many pages. We’ll build header and footer components, make them available inside an XF template, then include the XF in the page structure.

### 4.1. Create Header and Footer Components

**Header Component**  
Path: `/apps/hrportal/components/content/header`

- `header.html` (HTL):
```html
<sly data-sly-use.header="com.corp.hr.portal.core.models.HeaderModel">
    <header class="hr-header">
        <div class="logo">
            <img src="${header.logoPath}" alt="HR Portal"/>
        </div>
        <nav>
            <ul>
                <li data-sly-repeat.nav="${header.navItems}">
                    <a href="${nav.url}">${nav.title}</a>
                </li>
            </ul>
        </nav>
    </header>
</sly>
```

- Dialog (`_cq_dialog/.content.xml`):
```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:jcr="http://www.jcp.org/jcr/1.0"
          xmlns:nt="http://www.jcp.org/jcr/nt/1.0"
          xmlns:sling="http://sling.apache.org/jcr/sling/1.0"
          jcr:primaryType="nt:unstructured"
          sling:resourceType="cq/gui/components/authoring/dialog"
          title="Header">
    <content jcr:primaryType="nt:unstructured"
             sling:resourceType="granite/ui/components/coral/foundation/container">
        <items jcr:primaryType="nt:unstructured">
            <logo jcr:primaryType="nt:unstructured"
                  sling:resourceType="granite/ui/components/coral/foundation/form/pathfield"
                  fieldDescription="Select logo image"
                  fieldLabel="Logo"
                  name="./logo"
                  rootPath="/content/dam"/>
            <navItems jcr:primaryType="nt:unstructured"
                      sling:resourceType="granite/ui/components/coral/foundation/form/multifield"
                      fieldLabel="Navigation Items">
                <field jcr:primaryType="nt:unstructured"
                       sling:resourceType="granite/ui/components/coral/foundation/form/textfield"
                       name="title"
                       fieldLabel="Link Title"/>
                <field jcr:primaryType="nt:unstructured"
                       sling:resourceType="granite/ui/components/coral/foundation/form/pathfield"
                       name="url"
                       fieldLabel="Link URL"
                       rootPath="/content"/>
            </navItems>
        </items>
    </content>
</jcr:root>
```

- Sling Model (`HeaderModel.java` in `core`):
```java
package com.corp.hr.portal.core.models;

import com.adobe.cq.wcm.core.components.models.NavigationItem;
import org.apache.sling.api.resource.Resource;
import org.apache.sling.models.annotations.DefaultInjectionStrategy;
import org.apache.sling.models.annotations.Model;
import org.apache.sling.models.annotations.injectorspecific.ValueMapValue;

import java.util.List;

@Model(adaptables = Resource.class, defaultInjectionStrategy = DefaultInjectionStrategy.OPTIONAL)
public class HeaderModel {
    @ValueMapValue
    private String logo;

    @ValueMapValue
    private List<NavigationItem> navItems;  // multifield items

    public String getLogoPath() { return logo; }
    public List<NavigationItem> getNavItems() { return navItems; }
}
```

**Footer Component** – similar, with fields like copyright text, social links, etc.

### 4.2. Create an Editable Template for Experience Fragments

Experience Fragments need a special template that only allows the XF container.  
Create at `/conf/hrportal/settings/wcm/templates/experience-fragment-web`:

- `structure/jcr:content` includes an XF root component (resource type `cq/experience-fragments/components/xfpage`).  
*Note: The archetype already provides a basic XF template; you can extend it.*

### 4.3. Create Header and Footer Experience Fragments

1. Go to AEM **Experience Fragments** console.
2. Create folder `/content/experience-fragments/hrportal`.
3. Create a new XF “Header” based on the template.
4. Add the **Header component** to the XF and fill in the dialog.
5. Repeat for “Footer”.

### 4.4. Use Experience Fragments in the Page Template

Your HR Portal page template (`/conf/hrportal/settings/wcm/templates/hr-page`) should include the XF components in its structure:

```html
<!-- In the template’s initial jcr:content -->
<headerinclude
    jcr:primaryType="nt:unstructured"
    sling:resourceType="cq/experience-fragments/components/content/xfpage"
    fragmentVariationPath="/content/experience-fragments/hrportal/header/master"/>
<footerinclude
    jcr:primaryType="nt:unstructured"
    sling:resourceType="cq/experience-fragments/components/content/xfpage"
    fragmentVariationPath="/content/experience-fragments/hrportal/footer/master"/>
```

Now every page created from this template will automatically have the header and footer. Authors can still edit the XF content independently; changes propagate everywhere.

---

## 5. Module 2: HR Policy Management Using Content Fragments

We’ll model an HR Policy with a Content Fragment Model, then build a component to display a single policy. The policy listing with filtering comes later.

### 5.1. Define the Content Fragment Model

Create the model definition at:  
`/conf/hrportal/settings/dam/cfm/models/hr-policy`

`model.xml`:
```xml
<jcr:root xmlns:cq="http://www.day.com/jcr/cq/1.0"
          xmlns:jcr="http://www.jcp.org/jcr/1.0"
          xmlns:nt="http://www.jcp.org/jcr/nt/1.0"
          jcr:primaryType="cq:Page">
    <jcr:content
        jcr:primaryType="nt:unstructured"
        jcr:title="HR Policy"
        cq:allowedTemplates="[/conf/hrportal/settings/dam/cfm/models/hr-policy]">
        <model jcr:primaryType="nt:unstructured">
            <elements jcr:primaryType="nt:unstructured">
                <title
                    jcr:primaryType="nt:unstructured"
                    name="title"
                    dataType="String"
                    title="Policy Title"
                    required="true"/>
                <summary
                    jcr:primaryType="nt:unstructured"
                    name="summary"
                    dataType="String"
                    title="Short Summary"
                    multiLine="true"/>
                <details
                    jcr:primaryType="nt:unstructured"
                    name="details"
                    dataType="Rich Text"
                    title="Full Policy Details"/>
                <policyTags
                    jcr:primaryType="nt:unstructured"
                    name="policyTags"
                    dataType="String[]"
                    title="Policy Tags"
                    metaType="tags"
                    rootPath="/content/cq:tags/hr-portal/policies"/>
            </elements>
        </model>
    </jcr:content>
</jcr:root>
```

Deploy this model via `ui.content`. Then authors can create Content Fragments of type “HR Policy” in the DAM (e.g., `/content/dam/hrportal/policies/leave-policy`).

### 5.2. Sling Model to Read a Policy Content Fragment

We need a model that adapts from the fragment’s resource.

```java
package com.corp.hr.portal.core.models;

import com.adobe.cq.dam.cfm.ContentFragment;
import org.apache.sling.api.resource.Resource;
import org.apache.sling.models.annotations.Model;
import org.apache.sling.models.annotations.injectorspecific.Self;

@Model(adaptables = Resource.class)
public class PolicyModel {
    @Self
    private Resource resource;

    private ContentFragment fragment;

    public ContentFragment getFragment() {
        if (fragment == null) {
            fragment = resource.adaptTo(ContentFragment.class);
        }
        return fragment;
    }

    public String getTitle() {
        return getFragment().getElement("title").getContent();
    }

    public String getSummary() {
        return getFragment().getElement("summary").getContent();
    }

    public String getDetails() {
        return getFragment().getElement("details").getContent();
    }

    public String[] getTags() {
        return getFragment().getElement("policyTags").getValue().getValue(String[].class);
    }
}
```

### 5.3. Policy Detail Component

Create component `/apps/hrportal/components/content/policydetail`:

- `policydetail.html`:
```html
<sly data-sly-use.policy="com.corp.hr.portal.core.models.PolicyModel">
    <article class="policy-detail">
        <h1>${policy.title}</h1>
        <p class="summary">${policy.summary}</p>
        <div class="details">${policy.details @ context='html'}</div>
        <ul class="tags">
            <li data-sly-repeat.tag="${policy.tags}">${tag}</li>
        </ul>
    </article>
</sly>
```

- Dialog: a simple pathfield to select the Content Fragment (name `./fragmentPath`), then the Sling Model can use that path to resolve the CF resource. A more advanced approach is to use a reference component or a sling:resourceType that points to the CF directly. For simplicity, we’ll use a pathfield and adapt the resource from the given path. In the HTL you’d use `<sly data-sly-resource="${policy.fragmentPath @ resourceType='...'}">` or better, handle everything in the model.

*Best practice:* use **Core Components Content Fragment Component** as a base and extend it. But for learning we’ll custom-build.

---

## 6. Module 3: Policy Listing Page with Tag‑Based Filtering

We need a component that lists all HR policies and allows users to filter by tags (e.g., “Benefits”, “Code of Conduct”). This involves QueryBuilder and tag picker UI.

### 6.1. OSGi Service to Encapsulate QueryBuilder Queries

Create `PolicySearchService` in the core module:

```java
package com.corp.hr.portal.core.services;

import com.day.cq.search.PredicateGroup;
import com.day.cq.search.Query;
import com.day.cq.search.QueryBuilder;
import com.day.cq.search.result.Hit;
import com.day.cq.search.result.SearchResult;
import org.apache.sling.api.resource.ResourceResolver;
import org.osgi.service.component.annotations.Component;
import org.osgi.service.component.annotations.Reference;

import javax.jcr.Session;
import java.util.*;

@Component(service = PolicySearchService.class)
public class PolicySearchServiceImpl implements PolicySearchService {

    @Reference
    private QueryBuilder queryBuilder;

    @Override
    public List<PolicySearchResult> searchPolicies(ResourceResolver resolver, String[] tags, int offset, int limit) {
        Map<String, String> map = new HashMap<>();
        map.put("path", "/content/dam/hrportal/policies");
        map.put("type", "dam:Asset");
        map.put("1_property", "jcr:content/data/master/*/policyTags");
        // If tags provided, add property.value for each
        if (tags != null && tags.length > 0) {
            int i = 1;
            for (String tag : tags) {
                map.put(i + "_property.value", tag);
                i++;
            }
            map.put("property.operation", "like"); // or "equals" depending on tag storage
        }
        map.put("orderby", "@jcr:content/jcr:title");
        map.put("p.limit", String.valueOf(limit));
        map.put("p.offset", String.valueOf(offset));

        Query query = queryBuilder.createQuery(PredicateGroup.create(map), resolver.adaptTo(Session.class));
        SearchResult result = query.getResult();

        List<PolicySearchResult> policies = new ArrayList<>();
        for (Hit hit : result.getHits()) {
            policies.add(new PolicySearchResult(
                hit.getPath(),
                hit.getProperties().get("jcr:content/jcr:title", String.class),
                hit.getProperties().get("jcr:content/data/master/summary", String.class)
            ));
        }
        return policies;
    }
}
```

### 6.2. Sling Model for the Policy Listing Component

```java
@Model(adaptables = {Resource.class, SlingHttpServletRequest.class},
       defaultInjectionStrategy = DefaultInjectionStrategy.OPTIONAL)
public class PolicyListModel {

    @SlingObject
    private ResourceResolver resourceResolver;

    @SlingObject
    private SlingHttpServletRequest request;

    @OSGiService
    private PolicySearchService searchService;

    private List<PolicySearchResult> policies;

    @PostConstruct
    protected void init() {
        String[] selectedTags = request.getParameterValues("tag"); // from user selection
        int page = 1; // parse from request
        int offset = (page - 1) * 10;
        policies = searchService.searchPolicies(resourceResolver, selectedTags, offset, 10);
    }

    public List<PolicySearchResult> getPolicies() { return policies; }

    // Also expose available tags (e.g., from TagManager) for filter UI
    public List<Tag> getAllPolicyTags() {
        // Use TagManager to get tags under /etc/tags/hr-portal/policies
    }
}
```

### 6.3. HTL and Dialog for Policy List Component

- `policylist.html`:
```html
<sly data-sly-use.model="com.corp.hr.portal.core.models.PolicyListModel">
    <div class="policy-filter">
        <ul data-sly-repeat.tag="${model.allPolicyTags}">
            <li><a href="${currentPage.path @ extension = 'html', query = 'tag=' + tag.tagID}">${tag.title}</a></li>
        </ul>
    </div>
    <div class="policy-list">
        <div data-sly-repeat.policy="${model.policies}" class="policy-item">
            <h3><a href="${policy.path @ extension = 'html'}">${policy.title}</a></h3>
            <p>${policy.summary}</p>
        </div>
    </div>
</sly>
```

- Dialog: author can optionally restrict the root path or set a default tag, but not mandatory. The component mainly relies on request parameters. A good practice is to add a “no content” state when no policies match.

### 6.4. How Tag Filtering Works End‑to‑End

1. Author tags a Content Fragment using the `policyTags` field (tag picker in CF editor).
2. When a user clicks a tag link, the page reloads with `?tag=hr-portal:policies/benefits`.
3. The Sling Model reads the tag parameter, passes it to `PolicySearchService`.
4. QueryBuilder searches for CFs whose `policyTags` contain that tag.
5. Results are displayed.

---

## 7. Module 4: Careers Section with Dynamic Job Listings

Jobs can also be Content Fragments (model “Job Posting” with fields: title, department, location, description, closingDate). A job listing component uses QueryBuilder to fetch all jobs ordered by posting date, with optional department filter.

### 7.1. Job Search Service

Similar to policies, but no tagging; filter by department or location.

```java
public List<JobResult> getJobs(String department, int limit) {
    Map<String, String> map = new HashMap<>();
    map.put("path", "/content/dam/hrportal/jobs");
    map.put("type", "dam:Asset");
    if (department != null) {
        map.put("1_property", "jcr:content/data/master/department");
        map.put("1_property.value", department);
    }
    map.put("orderby", "@jcr:content/data/master/closingDate");
    map.put("orderby.sort", "desc");
    // ...
}
```

### 7.2. Component

HTL loops over `JobListModel.jobs` and renders each job with title, department, link to a detail page (which is another component reading the full CF). Department filter can be a dropdown populated from all distinct departments (another QueryBuilder facet or a separate service).

---

## 8. Module 5: Announcements Section with Priority Handling

Announcements can be simple pages or Content Fragments. We’ll use Content Fragments with fields: `message` (rich text), `priority` (dropdown: high, medium, low), `startDate`, `endDate`. The listing component shows active announcements sorted by priority (high first) and then by startDate.

**Priority Enum Handling in QueryBuilder**

Since priority is stored as a string, we can order by a custom property. We can map priorities to numeric values using a Sling Model to sort after fetching, or use a `CASE`‑like expression if the underlying database supports it (not recommended). Simpler: fetch all active announcements, then sort them in Java using a comparator.

```java
List<AnnouncementResult> announcements = searchService.getActiveAnnouncements(resolver);
announcements.sort(Comparator.comparing(AnnouncementResult::getPriorityValue).reversed()
        .thenComparing(AnnouncementResult::getStartDate));
```

Where `priorityValue` maps "high"->3, "medium"->2, "low"->1.

### 8.1. OSGi Service for Announcements

```java
public List<AnnouncementResult> getActiveAnnouncements() {
    map.put("path", "/content/dam/hrportal/announcements");
    map.put("type", "dam:Asset");
    // property startDate <= now, endDate >= now
    // QueryBuilder date range is tricky; you can fetch all and filter in code.
}
```

### 8.2. Announcement List Component

HTL:
```html
<div data-sly-repeat.a="${model.announcements}" class="announcement ${a.priority}">
    <span class="badge">${a.priority}</span>
    <div>${a.message @ context='html'}</div>
</div>
```

---

## 9. Permissions & User Access Control

Set up closed user groups (CUG) or basic ACLs:

- HR Managers group: can edit all HR content and CFs.
- HR Authors group: can create/edit policies and announcements.
- Employees: read‑only access to the published portal.

In the `ui.content` module, include a file `META-INF/vault/filter.xml` with an ACL entry, or better, use repository initializer scripts (Apache Sling RepoInit) to set permissions.

Example RepoInit script (`org.apache.sling.jcr.repoinit.RepositoryInitializer-hrportal.config`):

```ini
create path /content/hrportal(nt:unstructured)
create group hr-authors
set ACL for hr-authors
    allow jcr:read, jcr:write on /content/hrportal
    allow jcr:read, jcr:write on /content/dam/hrportal
end
create group hr-admins
set ACL for hr-admins
    allow jcr:read, jcr:write on /content/hrportal
    allow jcr:read, jcr:write on /content/dam/hrportal
    allow jcr:all on /conf/hrportal
end
```

Place this file under `ui.config/src/main/content/jcr_root/apps/hrportal/osgiconfig/config.author/` (or use `repoinit` folder).

Users are then assigned to these groups in the AEM User Admin.

---

## 10. Deployment Steps Using Maven

1. Make sure local AEM author is running on `localhost:4502`.
2. In the project root, run:
   ```bash
   mvn clean install -PautoInstallPackage
   ```
   This builds all modules, installs the OSGi bundle (core) and deploys the content packages to the author instance.
3. For publish, use `-PautoInstallPackagePublish`.
4. After deployment, go to **Sites**, create a page from the HR template, and you’ll see the header/footer appear. Add the policy list component and start creating content fragments.

---

## 11. How Everything Connects Together

- **Editable Templates** control the page structure and allowed components. The header/footer are placed statically as XF includes, while the main area (`parsys`) allows adding the policy list, job list, announcement list, or any other component.
- **Content Fragments** are stored in DAM and hold structured policy/job/announcement data. Components reference these fragments either directly (pathfield) or via QueryBuilder searches.
- **Sling Models** bridge AEM resources and the HTL view, fetching data from CFs or services, performing business logic.
- **OSGi Services** keep reusable backend logic (search, filtering) outside of models, making them testable and shareable.
- **Tagging** tags are applied to content fragments; QueryBuilder searches those tags to filter listings.
- **Permissions** ensure only HR staff can create/edit content.

---

## 12. Best Practices & Real‑Time Tips

- **Use Core Components** where possible (e.g., Content Fragment Component, List Component). Extend or overlay them instead of writing from scratch.
- **Client Libraries**: Place CSS/JS in `ui.apps/src/main/content/jcr_root/apps/hrportal/clientlibs`. Use `categories` and embed dependencies. Always minify via AEM’s built‑in processor.
- **HTL Context**: Always use `@ context='html'` or `@ context='text'` to prevent XSS.
- **Resource Resolver**: Do not close the resolver you get from `@SlingObject` in a Sling Model. But in OSGi services, if you use `resourceResolverFactory.getServiceResourceResolver()`, always close it in a finally block.
- **Error Handling**: Wrap QueryBuilder execution in try‑catch and return empty lists on failure. Use `log.error` to aid debugging.
- **Caching**: Output from Sling Models can be cached (e.g., using `@RequestAttribute`). For dynamic listings that depend on request parameters, do not cache the model at a scope higher than request.
- **Testing**: Use AEM Mocks and `wcm.io Testing` for unit‑testing Sling Models and services.
- **Content Structure**: Keep content fragments in dedicated folders under `/content/dam/hrportal/{policies,jobs,announcements}` to simplify path‑based queries.
- **Dialogs**: Always use Coral UI 3 (Granite) dialogs. Use `granite/ui/components/coral/foundation/form/pathfield` for internal links.

---

## 13. Common Mistakes & How to Fix Them

1. **Forgot to add `sling:resourceSuperType` for templates** → editable template not showing up. Fix: verify your template type is `/libs/wcm/core/templates/editable`.
2. **Experience Fragment not rendering** → Check that the `fragmentVariationPath` is correct and the XF page has a valid variation (`master`).
3. **QueryBuilder does not return results** → Escape special characters in paths. Always verify your predicate properties match the actual JCR structure. Use `/crx/de` to inspect.
4. **Tag filter returns nothing** → Tags are stored as an array of strings. Use `property.operation=like` with proper escaping (`%tag%`). For exact match, store tags as a multi‑value property and use `property.value` with `property.operation=equals`; QueryBuilder will match if the array contains the value.
5. **Sling Model injection null** → Ensure the model is adaptable from `Resource.class` or `SlingHttpServletRequest.class` as needed. Check `@Model` annotation.
6. **Dialog fields not saving** → The `name` attribute must match a JCR property (prefixed with `./`) or you must handle it in the Sling Model’s post construct.
7. **Permissions not applied** → RepoInit scripts run only once at bundle activation. If you modify them, delete the corresponding config node under `/apps/system/config` and restart the bundle.

---

## 14. Conclusion

You now have a complete, modular HR Portal architecture built on AEM best practices. The key takeaways are:

- **Reusable structure** via Experience Fragments for header/footer.
- **Structured content** via Content Fragment Models, making policies, jobs, and announcements author‑friendly.
- **Dynamic listings** using QueryBuilder and OSGi services, exposed through Sling Models.
- **Tag‑based filtering** that ties AEM’s tagging system directly to content discovery.
- **Fine‑grained access control** using groups and ACLs.

Expand upon this foundation with pagination, search functionality, and integration with external HR systems via custom OSGi services. All components remain testable and maintainable, following real‑world AEM project standards.
