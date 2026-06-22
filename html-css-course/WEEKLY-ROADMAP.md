# HTML & CSS Course — 2-Week Intensive Roadmap (36 hours/week)

This roadmap turns the [HTML & CSS Course](README.md) into a fast, focused, two-week
sprint. It is built for people who can study close to full-time and want to be
building real, responsive web pages as quickly as possible.

- **Time budget:** 36 hours per week
- **Total length:** 2 weeks (~72 hours)
- **Split:** About **30% reading** and **70% building** (so ~11 hours reading and
  ~25 hours building each week).

> **How to read this plan.** Each week is broken into 6 study days. Each day lists
> the lessons to study and what to build. Type every example yourself, open it in a
> browser, do the lesson exercises, then build the day's practice. A bigger project
> closes out each week. This pace is intense — if a day runs long, push the overflow
> to your lighter days. Understanding beats speed.

**Daily rhythm (suggested):** 6 study days × ~6 hours = 36 hours. Aim for ~2 hours
reading/typing examples and ~4 hours building each day. Take one full day off to rest.

---

## Week-at-a-Glance

| Week | Focus                                  | Modules                | Weekly Project              |
| ---- | -------------------------------------- | ---------------------- | --------------------------- |
| 1    | HTML structure, forms, CSS fundamentals | Module 0–3            | Multi-Page Recipe Site      |
| 2    | Layout, responsive, advanced, pro skills | Module 4–6           | Final Responsive Portfolio  |

---

## Week 1 — HTML and CSS Fundamentals

**Goal:** Write clean, semantic HTML and style it confidently with CSS.

### Day 1 — The Web + HTML Structure (~6 hrs)

- [How the Web Works](00-getting-started/01-how-the-web-works.md)
- [Setup and Your First Page](00-getting-started/02-setup-and-your-first-page.md)
- [The HTML Document Structure](01-html-fundamentals/01-document-structure.md)
- [Text, Headings, and Paragraphs](01-html-fundamentals/02-text-and-headings.md)
- [Links and Navigation](01-html-fundamentals/03-links-and-navigation.md)

**Build:** A personal bio page — headings, paragraphs, and links to two sites you
like. Then add a second page and link the two together.

### Day 2 — Media, Lists, and Tables (~6 hrs)

- [Images and Media](01-html-fundamentals/04-images-and-media.md)
- [Lists](01-html-fundamentals/05-lists.md)
- [Tables](01-html-fundamentals/06-tables.md)

**Build:** A "favorite movies" page with an image for each film, a list of
highlights, and a comparison table (title, year, rating).

### Day 3 — Forms (~6 hrs)

- [Forms and Inputs](02-forms-and-semantics/01-forms-and-inputs.md)
- [Form Validation](02-forms-and-semantics/02-form-validation.md)

**Build:** A sign-up form with name, email, password, a dropdown, radio buttons,
and a checkbox. Make the right fields `required` and add basic validation.

### Day 4 — Semantics + Accessibility (~6 hrs)

- [Semantic HTML](02-forms-and-semantics/03-semantic-html.md)
- [Accessibility Basics](02-forms-and-semantics/04-accessibility-basics.md)

**Build:** Re-mark-up your Day 1 bio page using semantic tags (`header`, `nav`,
`main`, `article`, `footer`). Check it with keyboard-only navigation.

### Day 5 — CSS Basics (~6 hrs)

- [How CSS Works](03-css-fundamentals/01-how-css-works.md)
- [Selectors](03-css-fundamentals/02-selectors.md)
- [Colors and Units](03-css-fundamentals/03-colors-and-units.md)

**Build:** Add an external stylesheet to your bio page. Set colors, fonts, and
spacing using classes and a sensible color palette.

### Day 6 — Box Model, Type, and Backgrounds (~6 hrs)

- [The Box Model](03-css-fundamentals/04-the-box-model.md)
- [Typography](03-css-fundamentals/05-typography.md)
- [Backgrounds and Borders](03-css-fundamentals/06-backgrounds-and-borders.md)

**Weekly project — Multi-Page Recipe Site** (most of the day):

Build a small **2–3 page** recipe site, all hand-styled:

- A home page listing recipes (each in a bordered "card" with an image).
- A recipe detail page with a semantic structure, an ingredients list, and a
  steps list.
- Shared styling: a color theme, readable typography, padding/margins via the box
  model, and `box-sizing: border-box` set globally.

✅ **End-of-week check:** You can build multi-page, semantic HTML and style it with
external CSS — selectors, the box model, typography, colors, and backgrounds.

---

## Week 2 — Layout, Responsiveness, and Polish

**Goal:** Build real, modern, responsive layouts and finish with a portfolio.

### Day 1 — Display + Positioning (~6 hrs)

- [Display and the Normal Flow](04-layout-and-responsive/01-display-and-the-flow.md)
- [Positioning](04-layout-and-responsive/02-positioning.md)

**Build:** A card with a "NEW" badge pinned to its corner (absolute positioning)
and a sticky page header that stays visible while scrolling.

### Day 2 — Flexbox (~6 hrs)

- [Flexbox](04-layout-and-responsive/03-flexbox.md)

**Build:** A navigation bar (logo left, links right) and a row of equal-height
feature cards — both with Flexbox. Practice centering a box perfectly.

### Day 3 — Grid + Responsive (~6 hrs)

- [CSS Grid](04-layout-and-responsive/04-grid.md)
- [Responsive Design and Media Queries](04-layout-and-responsive/05-responsive-design-and-media-queries.md)

**Build:** A photo gallery using Grid with `auto-fit` + `minmax()`, plus media
queries so it collapses gracefully on phones. Test it in DevTools device mode.

### Day 4 — Advanced CSS, Part 1 (~6 hrs)

- [Specificity and the Cascade](05-advanced-css/01-specificity-and-the-cascade.md)
- [Pseudo-classes and Pseudo-elements](05-advanced-css/02-pseudo-classes-and-elements.md)
- [Custom Properties (CSS Variables)](05-advanced-css/03-custom-properties.md)

**Build:** Convert one of your earlier pages to use CSS variables for its colors
and spacing. Add a styled `:hover`/`:focus` state and a `::before` decorative icon.

### Day 5 — Advanced CSS, Part 2 + Pro Skills (~6 hrs)

- [Transitions and Transforms](05-advanced-css/04-transitions-and-transforms.md)
- [Animations](05-advanced-css/05-animations.md)
- [Organizing Your CSS](06-professional-skills/01-organizing-css.md)
- [DevTools and Debugging](06-professional-skills/02-devtools-and-debugging.md)

**Build:** Add hover transitions and one subtle keyframe animation (a fade-in or
pulse) to your gallery. Reorganize its CSS cleanly with BEM-style class names.

### Day 6 — Best Practices + Final Project (~6 hrs)

- [Best Practices and Performance](06-professional-skills/03-best-practices-and-performance.md)
- [From Design to Page](06-professional-skills/04-from-design-to-page.md)

**Weekly project — Final Responsive Portfolio** (most of the day):

Combine everything into a polished, responsive personal portfolio:

- A nav bar, a hero section, an about section, a project grid, and a contact form.
- **Flexbox** for the nav and hero, **Grid** for the project cards.
- **CSS variables** for your color theme and spacing scale.
- Hover **transitions** and one tasteful **animation** (respect
  `prefers-reduced-motion`).
- Fully **responsive** with media queries; cleanly organized CSS.
- Run the pre-launch checklist: validate your HTML/CSS, check color contrast, and
  test keyboard navigation.

✅ **End-of-week check:** You can build a complete, responsive, polished website
from scratch — and you're ready to learn JavaScript.

---

## After This Course

You now know how to structure and style any page. Next steps:

1. **Learn JavaScript** to make your pages interactive.
2. **Learn Git** to track your work and collaborate.
3. **Move on to a framework** like React or Next.js.

Congratulations — you can build for the web! 🎉
