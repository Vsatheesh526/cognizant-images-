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



<img width="1914" height="972" alt="image" src="https://github.com/user-attachments/assets/e11a26fb-a909-4e1c-98a1-d6304e43721f" />

<img width="1819" height="1072" alt="image" src="https://github.com/user-attachments/assets/6a47078d-1fb9-4e03-b511-33c72695b7cb" />






<img width="1896" height="831" alt="image" src="https://github.com/user-attachments/assets/c743f95e-69ea-4db6-b60c-9b0ba608efe6" />


<div class="welcome-container">

    <!-- First: get 'items' node -->
    <div data-sly-list.parent="${resource.listChildren}">
        
        <!-- Check only items node -->
        <div data-sly-test="${parent.name == 'items'}">

            <!-- Now loop item0, item1 -->
            <div data-sly-list.item="${parent.listChildren}">
                
                <div class="welcome-card">

                    <img src="${item.valueMap.icon}" alt="${item.valueMap.title}" />

                    <h3>${item.valueMap.title}</h3>

                    <p>${item.valueMap.description}</p>

                </div>








            </div>

        </div>

    </div>







    /* Container Layout */
.welcome-container {
    display: flex;
    flex-wrap: wrap;
    gap: 25px;
    justify-content: center;
    padding: 20px;
}

/* Card Style */
.welcome-card {
    width: 240px;
    padding: 20px;
    border-radius: 15px;
    background: #ffffff;
    box-shadow: 0 8px 20px rgba(0, 0, 0, 0.08);
    text-align: center;
    transition: all 0.3s ease;
    cursor: pointer;
    position: relative;
    overflow: hidden;
}

/* Hover Effect */
.welcome-card:hover {
    transform: translateY(-10px) scale(1.03);
    box-shadow: 0 15px 30px rgba(0, 0, 0, 0.15);
}

/* Icon Style */
.welcome-card img {
    width: 60px;
    height: 60px;
    margin-bottom: 15px;
    transition: transform 0.3s ease;
}

/* Icon Animation */
.welcome-card:hover img {
    transform: rotate(10deg) scale(1.1);
}

/* Title */
.welcome-card h3 {
    font-size: 18px;
    margin-bottom: 10px;
    color: #333;
}

/* Description */
.welcome-card p {
    font-size: 14px;
    color: #666;
    line-height: 1.5;
}

/* Glow Border Animation */
.welcome-card::before {
    content: "";
    position: absolute;
    inset: 0;
    border-radius: 15px;
    padding: 2px;
    background: linear-gradient(45deg, #00c6ff, #0072ff, #00c6ff);
    -webkit-mask: 
        linear-gradient(#fff 0 0) content-box, 
        linear-gradient(#fff 0 0);
    -webkit-mask-composite: xor;
    mask-composite: exclude;
    opacity: 0;
    transition: 0.3s;
}

/* Show border on hover */
.welcome-card:hover::before {
    opacity: 1;
}

/* Responsive */
@media (max-width: 768px) {
    .welcome-card {
        width: 100%;
        max-width: 300px;
    }
}

</div>
