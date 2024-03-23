---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://cover.sli.dev
# some information about your slides, markdown enabled
title: Welcome to Slidev
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# apply any unocss classes to the current slide
class: text-center
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# https://sli.dev/guide/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/guide/syntax#mdc-syntax
mdc: true
---

# 과거로의 회귀? Hypermedia 기반 개발 Fastapi HTMX 체험하기

<p>Pyweb Simposium 2024</p>
<p>발표자: 김순</p>

<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    다음 <carbon:arrow-right class="inline"/>
  </span>
</div>

<div class="abs-br m-6 flex gap-2">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon:edit />
  </button>
  <a href="http://localhost:8000" target="_blank" alt="Localhost" title="Open in Localhost"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon:application-web />
  </a>
  <a href="https://github.com/szto/foxy-reminder" target="_blank" alt="GitHub" title="Open in GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---
transition: fade-out
layout: center
---

<h2 class="text-center mt-10">Foxy-Reminder</h2>

<p v-after class="opacity-30 transform text-center">Reminders App</p>
<img src="https://foxy-reminder-slidev.vercel.app/images/foxy-raw.png" class="m-5 h-70 rounded shadow">
<p class="italic text-blue-600">https://github.com/szto/foxy-reminder/</p>



---
transition: fade-out
level: 2
---

# Architecture

<img src="https://foxy-reminder-slidev.vercel.app/images/htmx-architecture.png" class="m-5 h-100 rounded shadow">


---
transition: fade-out
level: 2
---

# Architecture for Python

<img src="https://foxy-reminder-slidev.vercel.app/images/full-stack-python-architecture.png" class="m-5 h-100 rounded shadow" v-after>



---
transition: fade-out
---

# Folder Tree

````md magic-move
```html {*|4-7,11,16,18}
.
├── LICENSE
├── README.md
├── app
│   ├── main.py
│   ├── routers
│   └── utils
├── config.json
├── inputs.json
├── pyproject.toml
├── reminder_db.json
├── requirements.txt
├── static
├── styles
├── tailwind.config.js
├── templates
├── testlib
└── tests
```

```html
├── app
│   ├── __init__.py
│   ├── main.py
│   ├── routers
│   │   ├── __init__.py
│   │   ├── api.py
│   │   ├── login.py
│   │   ├── reminders.py
│   │   └── root.py
│   └── utils
│       ├── __init__.py
│       ├── auth.py
│       ├── exceptions.py
│       └── storage.py
```

```html
├── templates
│   ├── pages
│   │   ├── login.html
│   │   ├── not-found.html
│   │   └── reminders.html
│   └── partials
│       └── reminders
│           ├── content.html
│           ├── item-row-edit.html
│           ├── item-row.html
│           ├── list-row-edit.html
│           ├── list-row.html
│           ├── new-item-row-edit.html
│           ├── new-item-row.html
│           ├── new-list-row-edit.html
│           └── new-list-row.html
```

```html
└── tests
    ├── __init__.py
    ├── conftest.py
    ├── test_ui.py
    └── test_unit.py
```


````


---
level: 2
---

# Content

컨텐츠 HTML에서는 리마인더 리스트, 리만인더 상세를 구분함.
````md magic-move
```html {*|5-8|5-7|8|12-20|*}
<div class="grid grid-cols-1 md:grid-cols-2 gap-8 m-8 reminders-content p-8">
  <div class="bg-white rounded-lg shadow overflow-hidden m-4">
    <h3 class="text-xl font-semibold text-gray-700 p-4 mb-6 border-b">Reminder Lists</h3>
    <div class="divide-y m-4 reminder-list-list">
      {% for reminder_list in reminder_lists %}
        {% include "partials/reminders/list-row.html" %}
      {% endfor %}
      {% include "partials/reminders/new-list-row.html" %}
    </div>
  </div>
  <div class="bg-white rounded-lg shadow overflow-hidden m-4 reminders_content-items">
      {% if selected_list %}
      <h3 class="text-xl font-semibold text-gray-700 p-4 mb-6 border-b">{{ selected_list.name }}</h3>
      <div class="divide-y m-4">
      {% for reminder_item in selected_list.items %}
        {% include "partials/reminders/item-row.html" %}
      {% endfor %}
      {% include "partials/reminders/new-item-row.html" %}
      </div>
      {% endif %}
  </div>
</div>
```

````

---

# New List Row

리스트 하단에서 새로운 리마인드 리스트를 생성할 수 있도록 함.
````md magic-move
```html {*}
<div
    class="reminder-row flex items-center justify-between p-3 hover:bg-gray-100 cursor-pointer text-gray-400"
    data-id="new-reminder-row"
    hx-get="/reminders/new-list-row-edit"
    hx-trigger="click"
    hx-swap="outerHTML"
>
    <p>New list</p>
    <img src="/static/img/icons/icon-add.svg" class="h-6 w-6" />
</div>
```

```html {*}
  <input
    type="text"
    placeholder="New list"
    name="reminder_list_name"
    autofocus
    size="40"
  />
  <div class="flex items-center">
    <img
      class="h-6 w-6 mr-2"
      src="/static/img/icons/icon-check-circle.svg"
      hx-post="/reminders/new-list-row"
      hx-include="[name='reminder_list_name']"
      hx-target=".reminders-content"
      hx-trigger="click, keyup[key=='Enter'] from:[name='reminder_list_name']"
      hx-swap="outerHTML"
    />
  </div>
</div>
```

```html {12-16}
  <input
    type="text"
    placeholder="New list"
    name="reminder_list_name"
    autofocus
    size="40"
  />
  <div class="flex items-center">
    <img
      class="h-6 w-6 mr-2"
      src="/static/img/icons/icon-check-circle.svg"
      hx-post="/reminders/new-list-row" -- POST 메소드
      hx-include="[name='reminder_list_name']" -- 인자
      hx-target=".reminders-content" -- 타겟 클래스
      hx-trigger="click, keyup[key=='Enter'] from:[name='reminder_list_name']" -- 클릭 또는 수정
      hx-swap="outerHTML" -- HTML div 를 교체
    />
  </div>
</div>
```

````

<div class="grid" grid-cols-2>
  <img src="https://foxy-reminder-slidev.vercel.app/images/reminder-list.png" class="m-10 h-30 rounded shadow">
  <img src="https://foxy-reminder-slidev.vercel.app/images/reminder-list-new.png" class="m-10 h-30 rounded shadow">
</div>

---

# Edit List Row

리마인더 리스트를 수정

```html
<div
  class="reminder-row-with-input{{ ' selected-list.id' if reminder_list.id == selected_list }} flex items-center justify-between p-3 hover:bg-gray-100 cursor-pointer text-gray-400"
  data-id="reminder-row-{{ reminder_list.id }}"
>
  <input
    type="text" value="{{ reminder_list.name }}"
    placeholder="{{ reminder_list.name }}"
    name="new_name" onfocus="this.select();" autofocus
  />
  <div class="flex items-center">
    <img
            class="h-6 w-6 mr-2"
            src="/static/img/icons/icon-check-circle.svg"
            hx-patch="/reminders/list-row-name/{{ reminder_list.id }}"
            hx-include="[name='new_name']"
            hx-target=".reminders-content"
            hx-trigger="click, keyup[key=='Enter'] from:[name='new_name']"
            hx-swap="outerHTML"
    />
    <img/>
  </div>
</div>
```
