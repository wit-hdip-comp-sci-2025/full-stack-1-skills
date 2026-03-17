---
name: ux-bulma-css
description: Bulma CSS framework for layouts, forms, navigation, and components in Handlebars views
metadata:
  tags: bulma, css, frontend, layout, forms, navbar, components
---

# Bulma CSS

## Setup

Include Bulma in layout head:

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bulma@0.9.4/css/bulma.min.css">
```

## Layout structure

```html
<body>
  {{> nav}}
  <section class="section">
    <div class="container">
      {{{content}}}
    </div>
  </section>
</body>
```

- `section.section`: vertical padding
- `container`: max-width, centered content

## Navbar

```html
<nav class="navbar is-primary" role="navigation">
  <div class="navbar-brand">
    <a class="navbar-item" href="/"><strong>App</strong></a>
  </div>
  <div class="navbar-menu">
    <div class="navbar-start">
      <a class="navbar-item {{#if (eq currentPath '/dashboard')}}is-active{{/if}}" href="/dashboard">Dashboard</a>
    </div>
    <div class="navbar-end">
      {{#if user}}
        <span class="navbar-item">Logged in as {{user.firstName}}</span>
        <a class="button is-light navbar-item" href="/logout">Logout</a>
      {{else}}
        <a class="button is-light" href="/signup">Sign up</a>
        <a class="button is-white" href="/">Log in</a>
      {{/if}}
    </div>
  </div>
</nav>
```

## Forms

```html
<div class="field">
  <label class="label">First Name</label>
  <div class="control">
    <input class="input" type="text" name="firstName" required>
  </div>
</div>
<div class="field">
  <div class="control">
    <button class="button is-primary">Submit</button>
  </div>
</div>
```

## Components

- **Box**: `class="box"` – card-like container
- **Table**: `class="table is-fullwidth is-striped"`
- **Buttons**: `button is-primary`, `button is-light`, `button is-small is-info`
- **Notification**: `class="notification is-info is-light"` for empty states
- **Columns**: `columns is-centered`, `column is-half` for centered forms

## Form with addons

```html
<div class="field has-addons">
  <div class="control is-expanded">
    <input class="input" type="text" name="title" placeholder="Enter title">
  </div>
  <div class="control">
    <button class="button is-primary">Add</button>
  </div>
</div>
```

## Typography

- `title`, `title is-5` for headings
- `content` for prose blocks
- `has-text-right` for alignment
