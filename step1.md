# Module 1: Reusable Header & Footer with Experience Fragments – Detailed Implementation

This section provides a complete, step‑by‑step guide to building a corporate HR Portal header and footer as reusable Experience Fragments. You will learn how to create the required editable templates, components (with full HTL, dialog, and Sling Model), the XF pages themselves, and finally how to embed them into the site template so they appear automatically on every page.

---

## 1. Architecture Overview

- **Header & Footer Components** – Custom AEM components with author‑friendly dialogs to manage logo, navigation, legal links, etc.
- **Experience Fragment (XF) Template** – An editable template that only allows the XF root component and our custom components. Authors use it to create dedicated XF pages.
- **Experience Fragments (content)** – Two XF pages (Header and Footer) created under `/content/experience-fragments/hrportal`. Authors place the respective component into the XF and configure it once.
- **Page Template (HR Page)** – The main editable template for HR portal pages. Its structure includes the Experience Fragment components (via `fragmentVariationPath`) so every page inherits the header and footer automatically.

---

## 2. Prerequisites

- AEM project generated from the Adobe archetype (as described earlier).
- Your code modules: `core` (Java), `ui.apps` (HTL, dialogs, clientlibs, templates), `ui.content` (initial XF pages).
- Local AEM author instance running.

We’ll work under the base path `/apps/hrportal`.

---

## 3. Step‑by‑Step Implementation

### Step 1: Create an Editable Template for Experience Fragments

We need a template that authors can use when creating new Experience Fragments for the HR portal. The template defines the initial structure and allowed components. The archetype may already contain a generic XF template (`/conf/<project>/settings/wcm/templates/experience-fragment`), but we’ll create a specific one for HR.

Create the template definition in `ui.apps`:

**Path:**  
`ui.apps/src/main/content/jcr_root/conf/hrportal/settings/wcm/templates/hr-xf-template`

You need the following files:

#### a) `settings.xml` (template metadata)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:cq="http://www.day.com/jcr/cq/1.0" xmlns:jcr="http://www.jcp.org/jcr/1.0"
    jcr:primaryType="cq:Template">
    <jcr:content
        jcr:primaryType="nt:unstructured"
        jcr:title="HR Experience Fragment Template"
        status="enabled"
        allowedPaths="[/content/experience-fragments/hrportal(/.*)?]"
        ranking="100"/>
</jcr:root>
```

This registers the template under `/conf/hrportal` and allows XF pages only inside the HR folder.

#### b) `initial/.content.xml` – Initial page content

When an author creates a page from this template, AEM copies this subtree.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:sling="http://sling.apache.org/jcr/sling/1.0" xmlns:cq="http://www.day.com/jcr/cq/1.0"
    xmlns:jcr="http://www.jcp.org/jcr/1.0" jcr:primaryType="cq:Page">
    <jcr:content
        jcr:primaryType="cq:PageContent"
        sling:resourceType="cq/experience-fragments/components/content/xfpage">
        <root
            jcr:primaryType="nt:unstructured"
            sling:resourceType="wcm/foundation/components/responsivegrid">
            <responsivegrid
                jcr:primaryType="nt:unstructured"
                sling:resourceType="wcm/foundation/components/responsivegrid"/>
        </root>
    </jcr:content>
</jcr:root>
```

This sets up the standard XF page component with a responsive grid so authors can add components.

#### c) `structure/.content.xml` – Structure definition (optional but recommended)

To lock down certain areas, we can add non‑removable components. For a simple XF template, we can leave it empty or simply include the same `sling:resourceType` as the initial content but with different policies. However, for XFs, the initial content is often sufficient. We’ll skip the structure layer to keep it simple.

#### d) `policies/.content.xml` – Component policies (optional)

Define policies to limit what components can be added to the responsive grid. For now, we’ll allow our custom header and footer components.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:sling="http://sling.apache.org/jcr/sling/1.0" xmlns:jcr="http://www.jcp.org/jcr/1.0"
    jcr:primaryType="cq:Page">
    <jcr:content
        jcr:primaryType="nt:unstructured"
        sling:resourceType="wcm/core/components/policy/policy"
        cq:policy="wcm/default">
        <content
            jcr:primaryType="nt:unstructured"
            sling:resourceType="wcm/core/components/policies/mappings"
            components="[/apps/hrportal/components/content/header,/apps/hrportal/components/content/footer]"/>
    </jcr:content>
</jcr:root>
```

Now the template is ready. After deployment, authors will see “HR Experience Fragment Template” when creating an XF inside `/content/experience-fragments/hrportal`.

---

### Step 2: Create the Header Component (with Full Code)

The header component will have fields for a logo image, navigation links (multifield), and optional search bar and language switcher.

**Component path in `ui.apps`:**  
`/apps/hrportal/components/content/header`

#### a) `header.html` – HTL Script

```html
<sly data-sly-use.header="com.corp.hr.portal.core.models.HeaderModel"
     data-sly-use.templates="core/wcm/components/commons/v1/templates.html">
    <header class="hr-header">
        <div class="header-top">
            <div class="logo">
                <!-- Logo from model -->
                <img src="${header.logoPath}" alt="HR Portal" data-sly-test="${header.logoPath}"/>
                <span class="logo-text" data-sly-test="${!header.logoPath}">Corporate HR</span>
            </div>
            <div class="header-utilities" data-sly-test="${header.showSearch || header.showLanguage}">
                <div class="search" data-sly-test="${header.showSearch}">
                    <form action="/search">
                        <input type="text" name="q" placeholder="Search"/>
                        <button type="submit">🔍</button>
                    </form>
                </div>
                <div class="language-switcher" data-sly-test="${header.showLanguage}">
                    <!-- implement language switcher logic -->
                </div>
            </div>
        </div>
        <nav class="main-nav" role="navigation" data-sly-test="${header.navItems.size > 0}">
            <ul>
                <li data-sly-repeat.nav="${header.navItems}">
                    <a href="${nav.url @ extension='html'}"
                       data-sly-attribute.target="${nav.linkTarget}">${nav.title}</a>
                </li>
            </ul>
        </nav>
    </header>
</sly>
```

#### b) Sling Model – `HeaderModel.java` (in `core`)

```java
package com.corp.hr.portal.core.models;

import com.day.cq.wcm.api.Page;
import com.day.cq.wcm.api.PageManager;
import org.apache.sling.api.resource.Resource;
import org.apache.sling.api.resource.ResourceResolver;
import org.apache.sling.models.annotations.DefaultInjectionStrategy;
import org.apache.sling.models.annotations.Model;
import org.apache.sling.models.annotations.injectorspecific.SlingObject;
import org.apache.sling.models.annotations.injectorspecific.ValueMapValue;
import org.apache.commons.lang3.StringUtils;

import java.util.ArrayList;
import java.util.List;

@Model(adaptables = Resource.class, defaultInjectionStrategy = DefaultInjectionStrategy.OPTIONAL)
public class HeaderModel {

    @ValueMapValue
    private String logo;          // Path to logo image (DAM)

    @ValueMapValue
    private List<NavItem> navItems;  // Multifield items

    @ValueMapValue
    private boolean showSearch = false;

    @ValueMapValue
    private boolean showLanguage = false;

    @SlingObject
    private ResourceResolver resourceResolver;

    @SlingObject
    private Resource resource;

    public String getLogoPath() {
        return logo;
    }

    public List<NavItem> getNavItems() {
        if (navItems == null) {
            // Fallback: build navigation from current site structure
            return getDefaultNavItems();
        }
        // Ensure target attribute is set
        for (NavItem item : navItems) {
            if (item.getLinkTarget() == null) {
                item.setLinkTarget("_self");
            }
        }
        return navItems;
    }

    public boolean isShowSearch() {
        return showSearch;
    }

    public boolean isShowLanguage() {
        return showLanguage;
    }

    /**
     * Fallback navigation: list child pages of the site root.
     */
    private List<NavItem> getDefaultNavItems() {
        List<NavItem> items = new ArrayList<>();
        PageManager pageManager = resourceResolver.adaptTo(PageManager.class);
        if (pageManager != null) {
            // Assume the site root is two levels up from current component
            Page currentPage = pageManager.getContainingPage(resource);
            if (currentPage != null) {
                Page rootPage = currentPage.getAbsoluteParent(2); // /content/hrportal
                if (rootPage != null) {
                    for (Page child : rootPage.listChildren()) {
                        NavItem item = new NavItem();
                        item.setTitle(child.getTitle());
                        item.setUrl(child.getPath() + ".html");
                        item.setLinkTarget("_self");
                        items.add(item);
                    }
                }
            }
        }
        return items;
    }

    // Inner class for multifield data (must match field names)
    public static class NavItem {
        private String title;
        private String url;
        private String linkTarget;

        public String getTitle() { return title; }
        public void setTitle(String title) { this.title = title; }
        public String getUrl() { return url; }
        public void setUrl(String url) { this.url = url; }
        public String getLinkTarget() { return linkTarget; }
        public void setLinkTarget(String linkTarget) { this.linkTarget = linkTarget; }
    }
}
```

> **Note:** The multifield binding works automatically if the multifield item fields are named `title`, `url`, etc., and the model class has a `List<NavItem>` and `NavItem` is a simple POJO with matching field names. Ensure you register the adapter (Sling Model) with the correct package export in `core`’s `pom.xml`.

#### c) Dialog Definition (`_cq_dialog/.content.xml`)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:jcr="http://www.jcp.org/jcr/1.0"
          xmlns:nt="http://www.jcp.org/jcr/nt/1.0"
          xmlns:sling="http://sling.apache.org/jcr/sling/1.0"
          jcr:primaryType="nt:unstructured"
          sling:resourceType="cq/gui/components/authoring/dialog"
          title="Header Configuration">
    <content jcr:primaryType="nt:unstructured"
             sling:resourceType="granite/ui/components/coral/foundation/container">
        <items jcr:primaryType="nt:unstructured">
            <!-- Tab: Logo & Navigation -->
            <tabs jcr:primaryType="nt:unstructured"
                  sling:resourceType="granite/ui/components/coral/foundation/tabs">
                <items jcr:primaryType="nt:unstructured">
                    <logo-nav jcr:primaryType="nt:unstructured"
                              jcr:title="Logo &amp; Navigation"
                              sling:resourceType="granite/ui/components/coral/foundation/container">
                        <items jcr:primaryType="nt:unstructured">
                            <logo jcr:primaryType="nt:unstructured"
                                  sling:resourceType="granite/ui/components/coral/foundation/form/pathfield"
                                  fieldLabel="Logo Image"
                                  name="./logo"
                                  rootPath="/content/dam"
                                  predicate="nosystem"/>
                            <navItems jcr:primaryType="nt:unstructured"
                                      sling:resourceType="granite/ui/components/coral/foundation/form/multifield"
                                      fieldLabel="Navigation Links">
                                <field jcr:primaryType="nt:unstructured"
                                       sling:resourceType="granite/ui/components/coral/foundation/container"
                                       name="./navItems">
                                    <items jcr:primaryType="nt:unstructured">
                                        <title jcr:primaryType="nt:unstructured"
                                               sling:resourceType="granite/ui/components/coral/foundation/form/textfield"
                                               fieldLabel="Title"
                                               name="title"
                                               required="true"/>
                                        <url jcr:primaryType="nt:unstructured"
                                             sling:resourceType="granite/ui/components/coral/foundation/form/pathfield"
                                             fieldLabel="URL"
                                             name="url"
                                             rootPath="/content"
                                             required="true"/>
                                        <linkTarget jcr:primaryType="nt:unstructured"
                                                    sling:resourceType="granite/ui/components/coral/foundation/form/select"
                                                    fieldLabel="Target"
                                                    name="linkTarget">
                                            <items jcr:primaryType="nt:unstructured">
                                                <self jcr:primaryType="nt:unstructured" text="Same Tab" value="_self"/>
                                                <blank jcr:primaryType="nt:unstructured" text="New Tab" value="_blank"/>
                                            </items>
                                        </linkTarget>
                                    </items>
                                </field>
                            </navItems>
                        </items>
                    </logo-nav>
                    <options jcr:primaryType="nt:unstructured"
                             jcr:title="Options"
                             sling:resourceType="granite/ui/components/coral/foundation/container">
                        <items jcr:primaryType="nt:unstructured">
                            <showSearch jcr:primaryType="nt:unstructured"
                                        sling:resourceType="granite/ui/components/coral/foundation/form/checkbox"
                                        fieldLabel="Show Search Bar"
                                        name="./showSearch"
                                        value="true"/>
                            <showLanguage jcr:primaryType="nt:unstructured"
                                          sling:resourceType="granite/ui/components/coral/foundation/form/checkbox"
                                          fieldLabel="Show Language Switcher"
                                          name="./showLanguage"
                                          value="true"/>
                        </items>
                    </options>
                </items>
            </tabs>
        </items>
    </content>
</jcr:root>
```

Now the header component is complete.

---

### Step 3: Create the Footer Component

Similar to header, but with footer‑specific fields: copyright text, social media links, legal links.

**Path:** `/apps/hrportal/components/content/footer`

#### a) `footer.html`

```html
<sly data-sly-use.footer="com.corp.hr.portal.core.models.FooterModel">
    <footer class="hr-footer">
        <div class="footer-content">
            <div class="footer-links">
                <ul data-sly-repeat.link="${footer.legalLinks}">
                    <li><a href="${link.url}">${link.title}</a></li>
                </ul>
            </div>
            <div class="social-links" data-sly-test="${footer.socialLinks.size > 0}">
                <a data-sly-repeat.social="${footer.socialLinks}" href="${social.url}" target="_blank"
                   class="social-icon ${social.platform}">${social.icon}</a>
            </div>
            <div class="copyright">
                <p>${footer.copyrightText}</p>
            </div>
        </div>
    </footer>
</sly>
```

#### b) `FooterModel.java`

```java
package com.corp.hr.portal.core.models;

import org.apache.sling.api.resource.Resource;
import org.apache.sling.models.annotations.DefaultInjectionStrategy;
import org.apache.sling.models.annotations.Model;
import org.apache.sling.models.annotations.injectorspecific.ValueMapValue;
import java.util.List;

@Model(adaptables = Resource.class, defaultInjectionStrategy = DefaultInjectionStrategy.OPTIONAL)
public class FooterModel {

    @ValueMapValue
    private String copyrightText;

    @ValueMapValue
    private List<LinkItem> legalLinks;

    @ValueMapValue
    private List<SocialLink> socialLinks;

    public String getCopyrightText() { return copyrightText; }
    public List<LinkItem> getLegalLinks() { return legalLinks; }
    public List<SocialLink> getSocialLinks() { return socialLinks; }

    public static class LinkItem {
        private String title;
        private String url;
        public String getTitle() { return title; }
        public void setTitle(String title) { this.title = title; }
        public String getUrl() { return url; }
        public void setUrl(String url) { this.url = url; }
    }

    public static class SocialLink {
        private String platform;
        private String url;
        private String icon;
        public String getPlatform() { return platform; }
        public void setPlatform(String platform) { this.platform = platform; }
        public String getUrl() { return url; }
        public void setUrl(String url) { this.url = url; }
        public String getIcon() { return icon; }
        public void setIcon(String icon) { this.icon = icon; }
    }
}
```

#### c) Dialog XML (similar structure as header, with multifields for legal links and social links, plus a text field for copyright)

Just the relevant parts:

```xml
<legalLinks jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/multifield"
    fieldLabel="Legal Links">
    <field jcr:primaryType="nt:unstructured"
           sling:resourceType="granite/ui/components/coral/foundation/container">
        <items jcr:primaryType="nt:unstructured">
            <title .../>
            <url .../>
        </items>
    </field>
</legalLinks>
<socialLinks jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/multifield"
    fieldLabel="Social Media">
    <field jcr:primaryType="nt:unstructured"
           sling:resourceType="granite/ui/components/coral/foundation/container">
        <items jcr:primaryType="nt:unstructured">
            <platform jcr:primaryType="nt:unstructured"
                      sling:resourceType="granite/ui/components/coral/foundation/form/select"
                      fieldLabel="Platform">
                <items jcr:primaryType="nt:unstructured">
                    <facebook text="Facebook" value="facebook"/>
                    <twitter text="Twitter" value="twitter"/>
                    <linkedin text="LinkedIn" value="linkedin"/>
                </items>
            </platform>
            <url .../>
            <icon jcr:primaryType="nt:unstructured"
                  sling:resourceType="granite/ui/components/coral/foundation/form/textfield"
                  fieldLabel="Icon HTML (e.g. &amp;#xf39e;)"
                  name="icon"/>
        </items>
    </field>
</socialLinks>
<copyrightText jcr:primaryType="nt:unstructured"
    sling:resourceType="granite/ui/components/coral/foundation/form/textfield"
    fieldLabel="Copyright Text"
    name="./copyrightText"/>
```

---

### Step 4: Create Client Libraries for Header & Footer (Optional but Best Practice)

To style the header and footer, create a client library under `/apps/hrportal/clientlibs/clientlib-base`.

`ui.apps/src/main/content/jcr_root/apps/hrportal/clientlibs/clientlib-base/.content.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:cq="http://www.day.com/jcr/cq/1.0" xmlns:jcr="http://www.jcp.org/jcr/1.0"
    jcr:primaryType="cq:ClientLibraryFolder"
    categories="[hrportal.base]"
    dependencies="[cq.jquery]"/>
```

Add `css.txt` and `js.txt` and style files. Then include this clientlib in your header HTL using `<sly data-sly-call="${clientLib.css @ categories='hrportal.base'}"/>` (requires extra setup). Alternatively, embed it in the page template.

---

### Step 5: Create the Experience Fragments for Header and Footer

We need two XF pages under `/content/experience-fragments/hrportal`. You can create them manually after deployment, or include them as initial content via `ui.content`.

Add the following files to `ui.content/src/main/content/jcr_root/content/experience-fragments/hrportal/header/.content.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:sling="http://sling.apache.org/jcr/sling/1.0" xmlns:cq="http://www.day.com/jcr/cq/1.0"
          xmlns:jcr="http://www.jcp.org/jcr/1.0" jcr:primaryType="cq:Page">
    <jcr:content
        jcr:primaryType="cq:PageContent"
        sling:resourceType="cq/experience-fragments/components/content/xfpage"
        cq:xfVariantType="web"
        cq:tags="[]">
        <root
            jcr:primaryType="nt:unstructured"
            sling:resourceType="wcm/foundation/components/responsivegrid">
            <header
                jcr:primaryType="nt:unstructured"
                sling:resourceType="hrportal/components/content/header"
                logo="/content/dam/hrportal/logo.png"
                showSearch="true">
                <navItems jcr:primaryType="nt:unstructured">
                    <item0 jcr:primaryType="nt:unstructured"
                           title="Home" url="/content/hrportal/home" linkTarget="_self"/>
                    <item1 jcr:primaryType="nt:unstructured"
                           title="Policies" url="/content/hrportal/policies" linkTarget="_self"/>
                    <item2 jcr:primaryType="nt:unstructured"
                           title="Careers" url="/content/hrportal/careers" linkTarget="_self"/>
                </navItems>
            </header>
        </root>
    </jcr:content>
</jcr:root>
```

Similarly, create the Footer XF with the footer component preconfigured. Create the `footer` folder and an analogous `.content.xml`.

> **Important:** The XF must have a variation (usually “master” or “web”). The XF page component automatically creates a variation when authors publish or edit. The variation is referenced via the path `/content/experience-fragments/hrportal/header/master`. For initial content, we need to also include the variation node. Add:

`header/jcr:content/variations/master/.content.xml` (or similar) – but it's simpler to let authors create the variation manually after first deployment. In production, you can pre-create the variation node.

For now, after deploying the content, you’ll open the XF editor, which will create the variation automatically when you publish. Then the variation path becomes active.

---

### Step 6: Integrate XFs into the HR Page Editable Template

Now we need the HR Portal page template to include the header and footer XFs in its structure so every page inherits them.

Create the editable template for HR pages at:

`ui.apps/src/main/content/jcr_root/conf/hrportal/settings/wcm/templates/hr-page`

#### a) `settings.xml` (template properties)

```xml
<jcr:root xmlns:cq="http://www.day.com/jcr/cq/1.0" xmlns:jcr="http://www.jcp.org/jcr/1.0"
    jcr:primaryType="cq:Template">
    <jcr:content
        jcr:primaryType="nt:unstructured"
        jcr:title="HR Portal Page"
        status="enabled"
        allowedPaths="[/content/hrportal(/.*)?]"
        ranking="0"/>
</jcr:root>
```

#### b) `initial/.content.xml` – page initial content

```xml
<jcr:root xmlns:sling="http://sling.apache.org/jcr/sling/1.0" xmlns:cq="http://www.day.com/jcr/cq/1.0"
    xmlns:jcr="http://www.jcp.org/jcr/1.0" jcr:primaryType="cq:Page">
    <jcr:content
        jcr:primaryType="cq:PageContent"
        sling:resourceType="hrportal/components/structure/page"
        cq:template="/conf/hrportal/settings/wcm/templates/hr-page"
        jcr:title="HR Home">
        <root
            jcr:primaryType="nt:unstructured"
            sling:resourceType="wcm/foundation/components/responsivegrid">
            <headerinclude
                jcr:primaryType="nt:unstructured"
                sling:resourceType="cq/experience-fragments/components/content/xfpage"
                fragmentVariationPath="/content/experience-fragments/hrportal/header/master"/>
            <main-content
                jcr:primaryType="nt:unstructured"
                sling:resourceType="wcm/foundation/components/parsys"/>
            <footerinclude
                jcr:primaryType="nt:unstructured"
                sling:resourceType="cq/experience-fragments/components/content/xfpage"
                fragmentVariationPath="/content/experience-fragments/hrportal/footer/master"/>
        </root>
    </jcr:content>
</jcr:root>
```

Notice we used a custom page component `hrportal/components/structure/page`. We need to create that component to provide the basic HTML skeleton (head, body, inclusion of clientlibs). If you don’t, you can use `wcm/foundation/components/basicpage` (but it's deprecated). Better to create a custom page component.

#### c) Custom Page Component (minimal)

**Path:** `/apps/hrportal/components/structure/page`  
`page.html`:

```html
<!DOCTYPE html>
<html lang="en" data-sly-use.page="com.adobe.cq.wcm.core.components.models.Page">
<head>
    <meta charset="UTF-8">
    <title>${page.title}</title>
    <sly data-sly-include="head.html" />
</head>
<body class="hr-portal">
    <sly data-sly-include="body.html" />
</body>
</html>
```

`body.html`:
```html
<div data-sly-resource="${@path='root', resourceType='wcm/foundation/components/responsivegrid'}"></div>
```

`head.html`:
```html
<sly data-sly-call="${clientlib.css @ categories='hrportal.base'}" />
<sly data-sly-call="${clientlib.js @ categories='hrportal.base'}" />
```

This simple page component renders the responsivegrid, which contains the XF includes and the `parsys`. The header and footer will appear on every page without authors needing to add them.

---

### Step 7: Test and Deploy

1. Build and deploy the project:
   ```bash
   mvn clean install -PautoInstallPackage
   ```
2. Navigate to **Experience Fragments** → `hrportal` folder. You should see the Header and Footer XFs (if initial content was deployed) or create them manually.
3. Open the Header XF, edit the header component (set logo, links). Publish the XF. This creates the `master` variation.
4. Go to **Sites**, create a page from the “HR Portal Page” template. You’ll see the header and footer automatically rendered.
5. If you need to change header/navigation, edit the Header XF, publish, and all pages update instantly.

---

## 6. End‑to‑End Flow Summary

1. **Author:** Creates/edits an XF page (Header) using the dedicated template.
2. **Author:** Drops the **Header component** onto the XF and configures it (logo, nav items).
3. **Author:** Publishes the XF → variation `master` is created.
4. **Page Template:** The HR page template’s initial content includes the XF include node with `fragmentVariationPath=/.../header/master`.
5. **Visitor:** Requests an HR page. AEM resolves the page component, which includes the responsive grid. The grid’s XF include component dynamically pulls the rendered HTML of the Header Experience Fragment variation and inserts it.
6. **Footer** works identically.

---

## 7. Best Practices & Real‑Time Tips

- **Use dedicated XF folder:** Keep HR XFs under `/content/experience-fragments/hrportal` to maintain separation.
- **Always create a variation:** The component includes use the variation path (`.../master`). Without it, the include will not render.
- **Publish XFs after editing:** XF changes must be published to reflect on published pages.
- **Secure the XF template:** Only allow trusted authors to create XFs; restrict via permissions on the template.
- **Client libraries in XFs:** If your header/footer require CSS, load them in the page component, not inside the XF, to avoid duplication.
- **Fallback navigation:** Our Sling Model provides a fallback navigation if no items are configured. This prevents blank menus.

---

## 8. Common Mistakes & Troubleshooting

| Problem | Solution |
|---------|----------|
| XF include shows nothing on page | Verify `fragmentVariationPath` points to a valid XF variation (e.g., `/content/experience-fragments/hrportal/header/master`). Check that variation exists. |
| Header component dialog not showing | Ensure the component `sling:resourceType` in the XF matches the component path exactly, and the component has `_cq_dialog`. |
| Navigation links not working | In multifield, the pathfield name must be `url` and the model must have `setUrl()`. Also, links are rendered with `.html` extension automatically. |
| Logo not displaying | `rootPath` of pathfield should be `/content/dam`. Check that the image is published if on publish instance. |
| XF template not visible | Make sure the template is under `/conf/hrportal/settings/wcm/templates` and has `status="enabled"`. |

---

This completes the full implementation of reusable header and footer with Experience Fragments. You now have a production‑ready, author‑friendly solution that can be expanded with additional features like mega menus, breadcrumbs, or search.
