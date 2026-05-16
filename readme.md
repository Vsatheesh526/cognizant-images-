Got it 👍 — here is a **clean, correct, production-ready AEM multifield component** (dialog + HTL). No confusion, no extra things.

---

# ✅ 1. Component Structure

Create this inside your project:

```bash
ui.apps/src/main/content/jcr_root/apps/your-project/components/welcome-container/
```

Files:

```bash
welcome-container/
 ├── .content.xml
 ├── welcome-container.html
 └── _cq_dialog/.content.xml
```

---

# ✅ 2. Component Definition (`.content.xml`)

```xml
<jcr:root
    xmlns:sling="http://sling.apache.org/jcr/sling/1.0"
    xmlns:jcr="http://www.jcp.org/jcr/1.0"
    jcr:primaryType="cq:Component"
    jcr:title="Welcome Container"
    componentGroup="Your Project - Components"/>
```

---

# ✅ 3. Dialog (Multifield) — CLEAN VERSION

👉 `_cq_dialog/.content.xml`

```xml
<jcr:root
    xmlns:sling="http://sling.apache.org/jcr/sling/1.0"
    xmlns:jcr="http://www.jcp.org/jcr/1.0"
    jcr:primaryType="nt:unstructured"
    sling:resourceType="cq/gui/components/authoring/dialog">

    <content
        jcr:primaryType="nt:unstructured"
        sling:resourceType="granite/ui/components/coral/foundation/container">

        <items jcr:primaryType="nt:unstructured">

            <welcomeItems
                jcr:primaryType="nt:unstructured"
                sling:resourceType="granite/ui/components/coral/foundation/form/multifield"
                fieldLabel="Welcome Items"
                composite="{Boolean}true">

                <field
                    jcr:primaryType="nt:unstructured"
                    sling:resourceType="granite/ui/components/coral/foundation/container"
                    name="./items">

                    <items jcr:primaryType="nt:unstructured">

                        <!-- ICON -->
                        <icon
                            jcr:primaryType="nt:unstructured"
                            sling:resourceType="granite/ui/components/coral/foundation/form/pathfield"
                            fieldLabel="Icon"
                            name="./icon"/>

                        <!-- TITLE -->
                        <title
                            jcr:primaryType="nt:unstructured"
                            sling:resourceType="granite/ui/components/coral/foundation/form/textfield"
                            fieldLabel="Title"
                            name="./title"/>

                        <!-- DESCRIPTION -->
                        <description
                            jcr:primaryType="nt:unstructured"
                            sling:resourceType="granite/ui/components/coral/foundation/form/textarea"
                            fieldLabel="Description"
                            name="./description"/>

                    </items>

                </field>

            </welcomeItems>

        </items>
    </content>
</jcr:root>
```

---

# ✅ 4. HTL (Sightly) — CLEAN & CORRECT

👉 `welcome-container.html`

```html
<div class="welcome-container"
     data-sly-test="${resource.getChild('items')}">

    <div data-sly-list.item="${resource.getChild('items').children}">

        <div class="welcome-card">

            <!-- ICON -->
            <img src="${item.icon}" alt="${item.title}" />

            <!-- TITLE -->
            <h3>${item.title}</h3>

            <!-- DESCRIPTION -->
            <p>${item.description}</p>

        </div>

    </div>

</div>
```

---

# ✅ 5. (Optional) CSS

```css
.welcome-container {
    display: flex;
    gap: 20px;
    flex-wrap: wrap;
}

.welcome-card {
    width: 220px;
    padding: 15px;
    border-radius: 10px;
    border: 1px solid #ddd;
    text-align: center;
}

.welcome-card img {
    width: 50px;
    height: 50px;
}
```

---

# ✅ 6. Build & Deploy

```bash
mvn clean install -PautoInstallSinglePackage
```

---

# ✅ 7. What You Get

In AEM author:

✔ Click **Add**
✔ Fill:

* Icon
* Title
* Description

✔ Click **Add again** → new set appears

---

# ✅ 8. Final Output (Frontend)

```
[icon] Title 1
Description 1

[icon] Title 2
Description 2
```

---

# 🚀 Important Notes (Don’t Miss)

* ✔ Use `composite=true` (mandatory for multifield objects)
* ✔ Use `.children` in HTL (NOT `listChildren()`)
* ✔ Use `./items` as parent node name
* ✔ Always check `data-sly-test` to avoid null error

---

If you want next upgrade:

* 🔥 Drag reorder
* 🔥 Font Awesome icon picker instead of pathfield
* 🔥 Sling Model (interview-ready clean code)

Just tell 👍


<div class="welcome-container"
     data-sly-test="${resource.getChild('items')}">

    <div data-sly-list.item="${resource.getChild('items').children}">
        
        <div class="welcome-card">

            <img src="${item.icon}" alt="${item.title}" />

            <h3>${item.title}</h3>

            <p>${item.description}</p>

        </div>

    </div>

</div>
