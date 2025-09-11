# Shift Finder (Vue 3 + Vite)

A simple web app built with Vue 3 and Vite to help shift workers track
their schedules.\
It supports **A, B, C, and D shifts**, alternating between **4 days ON
(07:00--19:00)**, **4 days OFF**, **4 nights ON (19:00--07:00)**, and
**4 days OFF**.

---

## 🌐 Live Demo
Check it out here: [Shift Finder](https://programmmerjosh.github.io/shift-finder/)

---

## Features

-   **Search by Date**\
    Select a specific date to see if you're working (day/night) or off.\
    Displays the current work/off block, previous/next off blocks, and
    nearest work blocks.

-   **Search by Month**\
    View a full month calendar.

    -   Green = Day shifts\
    -   Purple = Night shifts\
    -   Off days are left unmarked\
        Below the calendar, a summary of all 4-day work blocks for that
        month is shown.

-   **Interactive Controls**

    -   Toggle between **A, B, C, D shifts**\
    -   Toggle between **Search-by-date** and **Search-by-month**\
    -   Custom inline date picker for easy date selection

-   **Responsive & Polished UI**

    -   Colored pills for Day/Night/Off\
    -   Accessible tab navigation\
    -   Works well on desktop & mobile

------------------------------------------------------------------------

## Tech Stack

-   [Vue 3](https://vuejs.org/) (Composition API, `<script setup>`)
-   [Vite](https://vitejs.dev/) (fast dev server & build tool)
-   GitHub Actions for deployment to [GitHub
    Pages](https://pages.github.com/)

------------------------------------------------------------------------

## Getting Started

### 1. Clone the repo

``` bash
git clone https://github.com/programmerjosh/shift-finder.git
cd shift-finder
```

### 2. Install dependencies

``` bash
npm install
```

### 3. Run locally

``` bash
npm run dev
```

App will be available at <http://localhost:5173>.

### 4. Build for production

``` bash
npm run build
npm run preview
```

------------------------------------------------------------------------

## Deployment

The app is configured for deployment via **GitHub Actions** to **GitHub
Pages**.\
Ensure your `vite.config.js` has:

``` js
export default defineConfig({
  base: '/shift-finder/',
})
```

------------------------------------------------------------------------

## License

MIT License © 2025 [programmerjosh](https://github.com/programmerjosh)
