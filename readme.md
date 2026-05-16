Nice 👍 your HTL is correct now — let’s make it look **modern + interactive**.

---

# 🎨 Interactive CSS (Clean + Attractive)

```css
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
```

---

# ✨ What You Get

✔ Smooth hover lift
✔ Icon animation
✔ Glow border effect
✔ Clean modern UI
✔ Fully responsive

---

# 🔥 Optional (Extra Animation)

If you want **entry animation when page loads**, add:

```css
.welcome-card {
    opacity: 0;
    transform: translateY(30px);
    animation: fadeUp 0.6s ease forwards;
}

.welcome-card:nth-child(2) { animation-delay: 0.2s; }
.welcome-card:nth-child(3) { animation-delay: 0.4s; }

@keyframes fadeUp {
    to {
        opacity: 1;
        transform: translateY(0);
    }
}
```

---

If you want next level:

* 🔥 click ripple effect
* 🔥 background gradient animation
* 🔥 glassmorphism UI

Just tell 👍







<div class="row-cards">

<div data-sly-list.parent="${resource.listChildren}">
    <div data-sly-test="${parent.name == 'items'}">

        <div class="welcome-container">

            <div data-sly-list.item="${parent.listChildren}">
                
                <div class="welcome-card">

                   <img src="${item.valueMap.icon @ default='/content/dam/default-icon.png'}"
     alt="${item.valueMap.title @ default='Card'}" />

<h3>${item.valueMap.title @ default='Default Title'}</h3>

<p>${item.valueMap.description @ default='Default description goes here.'}</p>

                </div>

            </div>

        </div>

    </div>
</div>

</div>



<sly data-sly-use.clientlib="/libs/granite/sightly/templates/clientlib.html"/>

<sly data-sly-call="${clientlib.css @ categories='corporate-hr-portal.site'}"/>
<sly data-sly-call="${clientlib.js @ categories='corporate-hr-portal.site'}"/>

