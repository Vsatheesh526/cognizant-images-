Here is the CSS and the improved HTML (HTL) code to make the cards horizontally scrollable or wrap in a responsive grid.

### 1. The CSS (styles.css)

This CSS ensures the cards are displayed horizontally and look clean on all screen sizes.

```css
/* Container that holds all the cards */
.welcome-container {
    display: flex;
    flex-wrap: nowrap;      /* Prevents wrapping to next line */
    overflow-x: auto;       /* Adds horizontal scrollbar if cards exceed width */
    gap: 24px;              /* Space between cards */
    padding: 20px;
    scroll-snap-type: x mandatory; /* Smooth snapping (optional) */
    -webkit-overflow-scrolling: touch; /* Smooth scrolling on iOS */
}

/* Individual Card Styling */
.welcome-card {
    flex: 0 0 auto;         /* Prevents cards from shrinking or growing */
    width: 280px;           /* Fixed width for each card */
    background: #ffffff;
    border-radius: 16px;
    box-shadow: 0 10px 20px rgba(0, 0, 0, 0.1);
    overflow: hidden;
    transition: transform 0.3s ease, box-shadow 0.3s ease;
    scroll-snap-align: start; /* Snaps to start of card */
    
    /* Text styling inside card */
    display: flex;
    flex-direction: column;
    text-align: center;
    padding: 20px 16px;
}

/* Hover effect for cards */
.welcome-card:hover {
    transform: translateY(-8px);
    box-shadow: 0 20px 30px rgba(0, 0, 0, 0.15);
}

/* Image styling inside card */
.welcome-card img {
    width: 80px;
    height: 80px;
    object-fit: contain;
    margin: 0 auto 16px auto;
    display: block;
}

/* Title styling */
.welcome-card h3 {
    font-size: 1.25rem;
    font-weight: 600;
    color: #1e293b;
    margin: 12px 0 8px 0;
}

/* Description styling */
.welcome-card p {
    font-size: 0.9rem;
    color: #475569;
    line-height: 1.5;
    margin: 0;
}

/* Optional: Hide scrollbar for cleaner look (Chrome/Safari) */
.welcome-container::-webkit-scrollbar {
    height: 6px;
}

.welcome-container::-webkit-scrollbar-track {
    background: #f1f1f1;
    border-radius: 10px;
}

.welcome-container::-webkit-scrollbar-thumb {
    background: #cbd5e1;
    border-radius: 10px;
}

/* Responsive: On smaller screens, cards take up more width */
@media (max-width: 768px) {
    .welcome-card {
        width: 240px;
        padding: 16px 12px;
    }
    
    .welcome-container {
        gap: 16px;
        padding: 16px;
    }
}
```

### 2. The Improved HTL (HTML)

I fixed the syntax errors (closing div tags) and added proper HTML elements (`<h3>` for title, `<p>` for description) so the CSS works correctly.

```html
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
```

### How to make it responsive (Alternative: Grid Layout)

If you prefer the cards to **wrap naturally** without horizontal scrolling (for desktop screens), replace the `.welcome-container` CSS with this:

```css
.welcome-container {
    display: flex;
    flex-wrap: wrap;        /* Allows wrapping to next line */
    justify-content: center; /* Centers cards on the screen */
    gap: 24px;
    padding: 20px;
}

/* Remove overflow and scrollbar styles if using wrap */
```

### What changed in the HTL:
1. Added `<h3>` tags around the title.
2. Added `<p>` tags around the description.
3. Fixed the double `</div>` at the end.
4. Kept the exact HTL expressions you used (`item.valueMap.title`) so AEM/Sling will still resolve properties correctly.

This will give you modern, horizontally scrollable cards that work perfectly on mobile and desktop.



<img width="1598" height="1199" alt="image" src="https://github.com/user-attachments/assets/835b59ce-2159-40a7-b531-e2ed761aef74" />

