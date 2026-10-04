# HTML Notes

HTML stands for **HyperText Markup Language**. It is used to define the **structure and content of a webpage**.

---

## 1. Essential HTML Tags

Some common HTML tags are:

```html
<head>
<body>
<title>

<h1> to <h6>
<p>
<button>

<a>
<img>

<ul>
<ol>
<li>

<div>
```

### Headings

HTML provides six heading levels:

```html
<h1>Main Heading</h1>
<h2>Subheading</h2>
<h3>Smaller Heading</h3>
...
<h6>Smallest Heading</h6>
```

`<h1>` is the highest-level heading and `<h6>` is the lowest-level heading.

### Paragraph

```html
<p>This is a paragraph.</p>
```

### Button

```html
<button>Click Me</button>
```

---

# 2. Basic HTML Page Structure

A basic HTML document looks like this:

```html
<!DOCTYPE html>
<html>
<head>
    <title>My Website</title>
</head>

<body>
    <h1>Hello World</h1>
    <p>Welcome to my website.</p>
</body>
</html>
```

Two important parts are:

```html
<head>
</head>

<body>
</body>
```

## `<head>`

The `<head>` contains **information about the webpage** that is generally not displayed as the main page content.

It can contain:

- Page title
- Metadata
- CSS links
- Fonts
- SEO-related information

Example:

```html
<head>
    <title>My Portfolio</title>
    <meta charset="UTF-8">
    <link rel="stylesheet" href="style.css">
</head>
```

## `<body>`

The `<body>` contains the **actual content displayed on the webpage**.

Example:

```html
<body>
    <h1>Kundan Sai</h1>
    <p>Welcome to my portfolio.</p>
    <button>Contact Me</button>
</body>
```

---

# 3. HTML Elements

Consider:

```html
<h1>Hello World</h1>
```

It has:

```text
<h1>          Hello World          </h1>
 ↓                 ↓                 ↓
Opening tag      Content          Closing tag
```

Together, they form an **HTML element**.

```text
Opening Tag + Content + Closing Tag = HTML Element
```

Another example:

```html
<p>I am learning HTML.</p>
```

---

# 4. HTML Attributes

Attributes provide **additional information about an HTML element**.

General syntax:

```html
<tag attribute="value">
```

Example:

```html
<p class="para">Hello World</p>
```

Here:

```text
p       → HTML element
class   → attribute
"para"  → attribute value
```

Attributes are written inside the **opening tag**.

---

# 5. Links — `<a>`

HTML uses the `<a>` element to create hyperlinks.

`a` means **anchor**.

Example:

```html
<a href="https://github.com">GitHub</a>
```

The `href` attribute specifies **where the link should go**.

```text
<a href="https://github.com">GitHub</a>
   ↑
   destination
```

## Opening a Link in a New Tab

```html
<a href="https://github.com" target="_blank">
    My GitHub
</a>
```

`target="_blank"` tells the browser to open the destination in a **new browsing context**, which is usually a new tab.

### Quick Revision

```text
<a>             → creates a link
href            → destination of the link
target="_blank" → usually opens link in a new tab
```

---

# 6. Images — `<img>`

HTML uses `<img>` to display images.

```html
<img src="profile.png" alt="My profile picture">
```

Important attributes:

```text
src → location/path of the image
alt → alternative text describing the image
```

### `src`

`src` tells the browser **which image to load**.

```html
<img src="github.png" alt="GitHub logo">
```

### `alt`

`alt` provides a **text alternative for the image**.

It is important for accessibility, especially for screen readers, and can also be displayed when the image cannot be loaded.

```html
<img src="profile.jpg" alt="Kundan Sai">
```

`<img>` is a **void element**, meaning it does not have a closing tag.

```html
<img src="image.jpg" alt="Description">
```

Not:

```html
<img>...</img>
```

---

# 7. Making an Image Clickable

We can put an `<img>` inside an `<a>` element.

```html
<a href="https://github.com">
    <img src="github.png" alt="Visit my GitHub">
</a>
```

Now clicking the **image** opens the link.

Think of it as:

```text
<a>                    ← link
    <img>              ← clickable content
</a>
```

---

# 8. HTML Lists

There are two common types of lists.

## Unordered List — `<ul>`

Used when the order of items **doesn't matter**.

```html
<ul>
    <li>Python</li>
    <li>PyTorch</li>
    <li>Transformers</li>
</ul>
```

Output conceptually:

```text
• Python
• PyTorch
• Transformers
```

`<li>` means **list item**.

## Ordered List — `<ol>`

Used when the **order matters**.

```html
<ol>
    <li>Learn Python</li>
    <li>Learn PyTorch</li>
    <li>Learn Transformers</li>
</ol>
```

Output conceptually:

```text
1. Learn Python
2. Learn PyTorch
3. Learn Transformers
```

### Quick Revision

```text
<ul> → unordered list
<ol> → ordered list
<li> → list item
```

---

# 9. `<div>`

`<div>` is a **generic container** used to group elements.

Example:

```html
<div>
    <h2>My Projects</h2>
    <p>These are my projects.</p>
</div>
```

A `<div>` does not tell us what the content **means**.

Because of this, when an HTML element with meaningful semantics exists, we usually prefer it.

Instead of using `<div>` everywhere:

```html
<div>
<div>
<div>
<div>
```

we can use semantic elements such as:

```html
<header>
<nav>
<main>
<section>
<article>
<footer>
```

---

# 10. Semantic HTML

**Semantic HTML** means using elements that describe the **meaning or purpose of their content**.

For example:

```html
<header>
<nav>
<main>
<section>
<article>
<footer>
```

This makes the HTML structure easier to understand and can improve accessibility.

---

# 11. `<header>`

`<header>` represents introductory content for a page or section.

It commonly contains:

- Logo
- Site name
- Heading
- Navigation

Example:

```html
<header>
    <h1>Kundan Sai</h1>
</header>
```

---

# 12. `<nav>`

`<nav>` represents a section containing **navigation links**.

Example:

```html
<nav>
    <a href="#">Home</a>
    <a href="#">About</a>
    <a href="#">Projects</a>
    <a href="#">Contact</a>
</nav>
```

Think:

```text
nav = navigation
```

---

# 13. `<header>` + `<nav>`

Navigation is commonly placed inside the page header.

```html
<header>

    <h1>Kundan Sai</h1>

    <nav>
        <a href="#">Home</a>
        <a href="#">About</a>
        <a href="#">Projects</a>
        <a href="#">Contact</a>
    </nav>

</header>
```

Structure:

```text
HEADER
│
├── Website Name / Logo
│
└── NAV
    ├── Home
    ├── About
    ├── Projects
    └── Contact
```

---

# 14. `<main>`

`<main>` contains the **dominant/main content of the page**.

Example:

```html
<main>
    <h1>My Portfolio</h1>
    <p>Welcome to my portfolio.</p>
</main>
```

Repeated site-wide content such as navigation or copyright information normally sits outside `<main>`.

A page normally has one main content area.

---

# 15. `<section>`

`<section>` groups **related content into a meaningful section**.

Imagine a portfolio:

```text
MAIN
│
├── Hero
├── About
├── Skills
├── Projects
└── Contact
```

These can be represented using sections:

```html
<main>

    <section>
        <h2>About Me</h2>
    </section>

    <section>
        <h2>Skills</h2>
    </section>

    <section>
        <h2>Projects</h2>
    </section>

    <section>
        <h2>Contact</h2>
    </section>

</main>
```

A useful rule:

> Use `<section>` when the group of content has a meaningful purpose or topic.

Sections will often have a heading.

---

# 16. Then Why Do We Need `<div>`?

Semantic elements describe **meaning**.

Sometimes we need a container that has **no special semantic meaning**, usually for styling or layout.

That's where `<div>` is useful.

Example:

```html
<section>
    <h2>Projects</h2>

    <div class="project-container">

        <div class="project-card">
            Project 1
        </div>

        <div class="project-card">
            Project 2
        </div>

    </div>
</section>
```

Here:

```text
<section> → meaningful Projects section

<div> → containers mainly used for layout/styling
```

### Rule of Thumb

```text
Does the container have a meaningful semantic purpose?
        │
        ├── YES → use an appropriate semantic element
        │
        └── NO  → <div> may be appropriate
```

---

# 17. `<article>`

`<article>` represents **self-contained content**.

Ask yourself:

> Could this piece of content make sense on its own or potentially be reused/distributed independently?

If yes, `<article>` may be appropriate.

Common examples include:

- Blog posts
- News articles
- User comments
- Forum posts
- Product cards
- Independent content items

Example:

```html
<article>
    <h2>Learning HTML</h2>
    <p>Today I learned about semantic HTML.</p>
</article>
```

Another example:

```html
<section>
    <h2>Blog Posts</h2>

    <article>
        <h3>Learning HTML</h3>
        <p>...</p>
    </article>

    <article>
        <h3>Learning CSS</h3>
        <p>...</p>
    </article>

</section>
```

Think:

```text
<section> → groups related content

<article> → self-contained piece of content
```

---

# 18. `<footer>`

`<footer>` represents footer information for a page or section.

It commonly contains:

- Copyright information
- Contact information
- Privacy links
- Social links
- Other related information

Example:

```html
<footer>
    <p>© 2026 Kundan Sai</p>
    <a href="#">GitHub</a>
    <a href="#">LinkedIn</a>
</footer>
```

---

# 19. Putting Semantic HTML Together

A portfolio page might look like:

```html
<!DOCTYPE html>
<html>

<head>
    <title>Kundan Sai | Portfolio</title>
</head>

<body>

    <header>
        <h1>Kundan Sai</h1>

        <nav>
            <a href="#about">About</a>
            <a href="#skills">Skills</a>
            <a href="#projects">Projects</a>
            <a href="#contact">Contact</a>
        </nav>
    </header>

    <main>

        <section id="about">
            <h2>About Me</h2>
            <p>...</p>
        </section>

        <section id="skills">
            <h2>Skills</h2>
            <p>...</p>
        </section>

        <section id="projects">
            <h2>Projects</h2>

            <article>
                <h3>Project One</h3>
                <p>...</p>
            </article>

            <article>
                <h3>Project Two</h3>
                <p>...</p>
            </article>
        </section>

        <section id="contact">
            <h2>Contact</h2>
            <p>...</p>
        </section>

    </main>

    <footer>
        <p>© 2026 Kundan Sai</p>
    </footer>

</body>

</html>
```

The structure is:

```text
HTML
│
├── HEAD
│   ├── Title
│   ├── Metadata
│   └── CSS / other resources
│
└── BODY
    │
    ├── HEADER
    │   ├── Site heading / logo
    │   └── NAV
    │
    ├── MAIN
    │   ├── SECTION → About
    │   ├── SECTION → Skills
    │   ├── SECTION → Projects
    │   │   ├── ARTICLE → Project 1
    │   │   └── ARTICLE → Project 2
    │   └── SECTION → Contact
    │
    └── FOOTER
```

---

# Quick Revision

| HTML | Purpose |
|---|---|
| `<head>` | Information/metadata about the page |
| `<body>` | Visible page content |
| `<title>` | Browser/tab title |
| `<h1>`–`<h6>` | Headings |
| `<p>` | Paragraph |
| `<button>` | Button |
| `<a>` | Hyperlink |
| `href` | Link destination |
| `<img>` | Image |
| `src` | Image source/path |
| `alt` | Text alternative for an image |
| `<ul>` | Unordered list |
| `<ol>` | Ordered list |
| `<li>` | List item |
| `<div>` | Generic container |
| `<header>` | Introductory/header content |
| `<nav>` | Navigation links |
| `<main>` | Main/dominant page content |
| `<section>` | Sematic group of related content |
| `<article>` | Self-contained content |
| `<footer>` | Footer information |

## Remember

```text
HTML = Structure

<head>    → information ABOUT the page
<body>    → content INSIDE the page

<a>       → link
<img>     → image

<ul>      → unordered list
<ol>      → ordered list
<li>      → list item

<header>  → introductory content
<nav>     → navigation
<main>    → main content
<section> → related/thematic content
<article> → self-contained content
<footer>  → footer content

<div>     → generic container when no semantic
            element is appropriate
```