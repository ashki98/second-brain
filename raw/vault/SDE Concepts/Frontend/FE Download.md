# FE Download

Here’s a comprehensive summary of the **Frontend (FE) systems concepts** discussed in this conversation—formatted as point-wise notes for easy review and memory refresh:

---

## React, Vanilla JS, and DOM

- **Key features of React vs Vanilla JS:**
    - React uses component-based architecture, declarative UI, Virtual DOM, unidirectional data flow, and JSX syntax.
    - Vanilla JS requires direct, manual DOM manipulation for UI changes.
- **DOM in JS and React:**
    - DOM: Tree structure representing HTML elements in a page.
    - Vanilla JS: Direct manipulation (e.g., `getElementById`, `innerHTML`)—can be error-prone and inefficient for complex UIs.
    - React: Updates a *Virtual DOM*, diffing it with the real DOM for efficient, minimal updates.

| Vanilla JS DOM | React Virtual DOM |
| --- | --- |
| Direct manual updates | Declarative updates |
| Frequent reflows/repaints | Batched minimal updates |
| Hard to maintain | More manageable, less code |
| Slow for complex UIs | Fast via diffing |

---

## Components in React

- **Functional components:**
    - Regular JS functions returning JSX.
    - Modern React uses Hooks (`useState`, `useEffect`) to handle state and side effects.
    - Example:
        
        `javascriptfunction Greeting({ name }) {
          return <h1>Hello, {name}</h1>;
        }`
        
- **Class components:**
    - ES6 classes extending `React.Component`.
    - Use `this.state`, `this.setState`, lifecycle methods (`componentDidMount`, `componentDidUpdate`).
    - Example:
        
        `javascriptclass Greeting extends React.Component {
          render() {
            return <h1>Hello, {this.props.name}</h1>;
          }
        }`
        

| Feature | Functional Component | Class Component |
| --- | --- | --- |
| Syntax | Function | ES6 Class |
| State | Hooks (`useState`) | `this.state` |
| Side effects | Hooks (`useEffect`) | Lifecycle methods |

---

## Props & Parent-Child Communication

- **Props:**
    - “Properties” passed from parent to child; read-only for the child.
    - Used for component customization and data sharing.
- **To update the parent from the child:**
    - Parent passes a callback function (as prop).
    - Child invokes the function on some action, which runs parent logic (usually updates parent state).

Example:

`javascript*// Parent*
function Parent() {
  function updateParent() { ... }
  return <Child onAction={updateParent} />;
}

*// Child*
function Child({ onAction }) {
  return <button onClick={onAction}>Act</button>
}`

---

## Core JS Concepts

- **Callbacks:** Functions passed to other functions to be executed later.
- **Arrow functions:** Concise ES6 syntax for function expressions; retain lexical `this`.
    
    `javascriptconst add = (a, b) => a + b;`
    

---

## ES6 and ECMAScript Versions

- **ES6 (ECMAScript 2015):** Major JS upgrade—added arrow functions, classes, let/const, template literals, destructuring, modules, promises, etc.
- **Latest version: ES2025** (as of 2025)—has additional features (advanced Set operations, RegExp flags, etc.).
- “ES<number>” is like Python3.11/3.12—labels for the JavaScript language version.

---

## State Management

- **Local state:** In individual components (`useState`).
- **Global state:** Shared across components; handled by:
    - `useContext` (lightweight)
    - Redux (large-scale)
    - Zustand, Recoil, Jotai (modern alternatives)

---

## Next.js vs React (SSR, SSG, Routing, Edge Functions)

- **React:** Pure frontend; only client-side rendering; must use extra libraries for routing.
- **Next.js:** Fullstack framework on React—adds server-side rendering, static site generation, file-based routing.

| Feature | Plain React | Next.js |
| --- | --- | --- |
| SSR | Not built-in | Yes, built-in (`getServerSideProps`) |
| SSG | Not built-in | Yes, built-in (`getStaticProps`) |
| Routing | Custom needed | File-based, automatic |
- **SSG:** Pre-renders pages to HTML at build time with `getStaticProps` and `getStaticPaths`.
- **SSR:** Renders HTML at request time with `getServerSideProps`—requires a Node.js server in production; enables live data/SEO.
- **Edge functions:** Next.js can run logic close to the user (CDN edge), e.g., authentication, fast responses.

**Example SSG:**

`javascriptexport async function getStaticProps() { ... }
export async function getStaticPaths() { ... }`

**Example SSR:**

`javascriptexport async function getServerSideProps(context) { ... }`

---

## Next.js Production Server

- Build: `npm run build`
- Serve: `npm start` (runs Node.js server for SSR/API)
- Host on Vercel, custom VPS, Docker, or platforms like AWS.

---

## Next.js API Routes & DB Access

- `/pages/api/` directory exposes serverless endpoints—handle requests, connect to databases, return JSON, etc.
- Can build full CRUD logic with DB (MongoDB, Postgres, etc.) inside your Next.js app.

`javascript*// pages/api/user.js*
export default async function handler(req, res) {
  *// connect to DB, respond with data*
}`

---

## TypeScript with React

- Adds **static typing**—reducing bugs, improving dev experience, enabling safe refactoring.
- Scales well for large teams and codebases.
- Example:

`typescripttype Props = { name: string }
const Welcome: React.FC<Props> = ({ name }) => <h1>{name}</h1>;`

| Feature | JavaScript | TypeScript + React |
| --- | --- | --- |
| Type safety | No | Yes |
| Error checking | Runtime | Compile time |
| Refactoring | Risky | Safe, maintainable |

---

**Use this checklist as your fast-refresh guide whenever you want to revisit frontend system fundamentals and modern best practices!**

1. [https://www.instahyre.com/job-388212-full-stack-engineer-at-butter-money-bangalore/](https://www.instahyre.com/job-388212-full-stack-engineer-at-butter-money-bangalore/)
