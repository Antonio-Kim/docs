## Introduction to React

**NOTE! Lots of gibberish here in this React folders. These are self-note, not full documentation so there might not be some sort of logical explanation. Treat it like scratch pad of notes**

Front-End JavaScript library to help with development of websites. Very much like HTML but with JavaScript. Uses Babel to make it look like HTML but reality JavaScript code. React uses In-Memory to map the DOM-like structure and updates the actual DOM when it needs to.

It begins with HTML file at public->index.html and the JavaScript code retrieves the `<div id="root">` tag and then the React library kicks in.

```jsx
import { createRoot } from 'react-dom/client';

const rootElement = document.getElementById('root');
const root = createRoot(rootElement);
```

The code above looks very similar to regular JavaScript. It retrieves the root id and then the React library creates its own roots based on the tag. React focuses on components when building a website. Every part, even buttons, can be broken down to React components. The re-usability is the focus here. Components have to be capitalized - cannot be lowercased.

- Props: You can pass down parameters from parent component to the child component(s) using props.
- Modules: using export you can have classes and functions available to other components. Using `export` on function will export those functions outside. `export default` will set that function to be main function being exported.
- States: the useState returns a tuple: first returns a parameter of the state, and the second returns the function that modifies that state. Naming does not matter but the convention is usually `const [<state>, set<State>] = useState(<default value>)

### Styling with Tailwind CSS

We'll start with setting up Tailwind with Vite

```bash
npm create vite@latest my-project -- --template react
cd my-project
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss -init -p
```

Open tailwind.config.js and we'll need to specify path for react

```js
module.epxorts = {
  content: ['./src/**/*.{js,jsx,ts,tsx}'],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

now inside index.css, add the following:

```css
@tailwind base;
@tailwind components;
@tailwind utilites;
```

If you're using form, `npm i -D @tailwindcss/forms` and then add the following inside the plugins of tailwind.config.js:

```js
module.epxorts = {
  content: ['./src/**/*.{js,jsx,ts,tsx}'],
  theme: {
    extend: {},
  },
  plugins: [require('@tailwindcss/forms')],
};
```
