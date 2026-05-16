Alright — I’ll walk you through this **step-by-step in VS Code** so you can actually build the component without confusion.

We’ll create:
👉 `welcome-container` component
👉 Multifield (icon, title, description)
👉 Show in HTL

---

# ✅ STEP 1: Open Your AEM Project in VS Code

Open your AEM project folder (usually Maven project)

Example structure:

```
ui.apps
ui.content
core
```

👉 Go inside:

```
ui.apps/src/main/content/jcr_root/apps/your-project/components/
```

---

# ✅ STEP 2: Create Component Folder

Create a new folder:

```
welcome-container
```

Inside it create:

```
welcome-container/
 ├── .content.xml
 ├── welcome-container.html
 └── _cq_dialog/.content.xml
```

---

# ✅ STEP 3: Create Component Definition

👉 File: `.content.xml`

```xml
<jcr:root
    xmlns:sling="http://sling.apache.org/jcr/sling/1.0"
    xmlns:jcr="http://www.jcp.org/jcr/1.0"
    jcr:primaryType="cq:Component"
    jcr:title="Welcome Container"
    sling:resourceSuperType="core/wcm/components/container/v1/container"
    componentGroup="Your Project - Components"/>
```

---

# ✅ STEP 4: Create Dialog Folder

Create folder:

```
_cq_dialog
```

Inside create `.content.xml`

---

# ✅ STEP 5: Add Multifield Dialog Code

👉 File: `_cq_dialog/.content.xml`

```xml
<jcr:root
    xmlns:sling="http://sling.apache.org/jcr/sling/1.0"
    xmlns:jcr="http://www.jcp.org/jcr/1.0"
    xmlns:cq="http://www.day.com/jcr/cq/1.0"
    jcr:primaryType="nt:unstructured"
    sling:resourceType="cq/gui/components/authoring/dialog">

    <content jcr:primaryType="nt:unstructured">
        <items jcr:primaryType="nt:unstructured">

            <multifield
                sling:resourceType="granite/ui/components/coral/foundation/form/multifield"
                composite="{Boolean}true"
                fieldLabel="Welcome Items">

                <field
                    sling:resourceType="granite/ui/components/coral/foundation/container"
                    name="./items">

                    <items jcr:primaryType="nt:unstructured">

                        <icon
                            sling:resourceType="granite/ui/components/coral/foundation/form/pathfield"
                            fieldLabel="Icon"
                            name="./icon"/>

                        <title
                            sling:resourceType="granite/ui/components/coral/foundation/form/textfield"
                            fieldLabel="Title"
                            name="./title"/>

                        <description
                            sling:resourceType="granite/ui/components/coral/foundation/form/textarea"
                            fieldLabel="Description"
                            name="./description"/>

                    </items>

                </field>
            </multifield>

        </items>
    </content>
</jcr:root>
```

---

# ✅ STEP 6: Create HTL File

👉 File: `welcome-container.html`

```html
<div class="welcome-container">

    <div data-sly-list.item="${resource.getChild('items').listChildren}">
        
        <div class="welcome-card">

            <img src="${item.icon}" alt="${item.title}" />

            <h3>${item.title}</h3>

            <p>${item.description}</p>

        </div>

    </div>

</div>
```

---

# ✅ STEP 7: Build & Deploy

Open terminal in VS Code:

```bash
mvn clean install -PautoInstallSinglePackage
```

👉 This will deploy to AEM

---

# ✅ STEP 8: Add Component in AEM

1. Open AEM → Sites
2. Edit your page
3. Drag **Welcome Container** component
4. Click configure (wrench icon)

---

# ✅ STEP 9: Test Multifield

Now you will see:

👉 “Add” button
👉 Click → fill:

* icon
* title
* description

👉 Click “Add” again → new item appears

---

# ✅ STEP 10: Final Output

Frontend will show:

```
[icon] Title 1
Description 1

[icon] Title 2
Description 2
```

---

# 🔥 Bonus (Optional Styling)

Add inside your CSS:

```css
.welcome-container {
    display: flex;
    gap: 20px;
}

.welcome-card {
    padding: 15px;
    border: 1px solid #ddd;
    border-radius: 10px;
    width: 200px;
    text-align: center;
}
```

---

# 🚀 Done!

You now have:
✔ Multifield working
✔ Add button dynamic
✔ Data rendering in HTL

---

If you want next level:

* drag & drop reorder
* icon picker (font awesome)
* animations

Just tell 👍


<img width="1347" height="684" alt="image" src="https://github.com/user-attachments/assets/027cdc90-5445-4232-9f3e-3cc5ffa10d57" />

