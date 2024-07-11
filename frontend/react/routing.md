## Routing in React using React Router

- Installing React Router

```bash
npm i react-router-dom
```

### Setting up React router

Create Routes.tsx file somewhere inside the main src directory, same directory as index.tsx. Create the following code to set up

```jsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom';
import { ProductsPage } from './pages/ProductsPage';

const router = createBrowserRouter([
  {
    path: 'products',
    elements: <ProductsPage />,
  },
]);

export function Routes() {
  return <RouterProvider router={router} />;
}
```

This will in turn create `/products` route. The Routes should be inside the root.render on index.tsx

```jsx
root.render(
  <React.StrictMode>
    <Routes />
  </React.StrictMode>
);
```

### Nested Routes

We would want an app that handles multiple pages within the same component, or even having main page with different components. Nested Routes can handle those situations, using **Outlet**. Going back to previous example, assume we have some header component created already. Inside App.tsx, add the following:

```jsx
import { Outlet } from 'react-router-dom';
import { Header } from './Header';

export default function App() {
  return (
    <>
      <Header />
      <Outlet />
    </>
  );
}
```

We then modify the routes by adding `children` property inside of the router element:

```jsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom';
import { ProductsPage } from './pages/ProductsPage';

const router = createBrowserRouter([
  {
    path: '/',
    element: <App />,
    children: [
      {
        path: 'products',
        element: <ProductsPage />,
      },
    ],
  },
]);

export function Routes() {
  return <RouterProvider router={router} />;
}
```

notice product page has been down to children component instead, compared to the last example where it was the main page. Now the page includes both the root page using App, and sub-page with Product. The `<Outlet />` in App.tsx is where children pages are shown. We can even go further by creating pages using route parameters. Let's create a single page dedicated for one product inside products:

```jsx
import { useParams } from 'react-router-dom';
import { products } from '../data/products';

type Params = {
  id: string;
};
export function ProductPage() {
  const params = useParams<Params>();
  const id = params.id === undefined ? undefined : parseInt(params.id);
  const product = product.find((product) => product.id === id);
  return (
    <div>
      {product === undefined ? (<h1>Unknown Product</h1>) : (
        <h1>{product.name}</h1>
        <p>{product.description}</p>
        <p>{product.price}</p>
      )}
    </div>
  );
}
```

going back to the routes.tsx, we then can add individual product pages

```jsx
const router = createBrowserRouter([
  {
    path: '/',
    element: <App />,
    children: [
      {
        path: 'products',
        element: <ProductsPage />,
      },
      {
        path: 'product/:id',
        element: <ProductPage />,
      },
    ],
  },
]);
```

### Navigating to different pages.

Use <Link> attribute inside the react file and set one of the parameter to the page you would want the link to take you to for example, continuing on from example above:

```jsx
<nav>
  <Link to="product">Product</Link>
</nav>
```

to navigate to main root, no need for anything inside the "to" parameter:

```jsx
<nav>
  <Link to="">Home</Link>
</nav>
```

### Error pages

Error can be included on each of the element in the createBrowserRouter, using **errorElement**. Say you have a page component that's named `<ErrorPage />`

```jsx
const router = createBrowserRouter([
  {
    path: '/',
    element: <App />,
    errorElement: <ErrorPage />,
    children: [
      {
        path: 'products',
        element: <ProductsPage />,
      },
      {
        path: 'product/:id',
        element: <ProductPage />,
      },
    ],
  },
]);
```

### Index Route

Notice that there's no main page displayed in the App.tsx: only the Header and Outlet for child pages. To set a main page, we can set one of the children outlet to index by inserting `index: true`.

```jsx
const router = createBrowserRouter([
  {
    path: '/',
    element: <App />,
    errorElement: <ErrorPage />,
    children: [
      {
        index: true,
        element: <HomePage />,
      },
      {
        path: 'products',
        element: <ProductsPage />,
      },
      {
        path: 'product/:id',
        element: <ProductPage />,
      },
    ],
  },
]);
```

### Lazy Loading

Sometimes you do not want the client to load all the data from a page. To do so you have to set your page to act in lazy loading format. Say I have an admin page (`<AdminPage />`) that should be loaded when it's required to do so. Import the page like this: `const AdminPage = lazy(() => import('./pages/AdminPage'));` and then inside the children we'll modify the elements using Suspense.

```jsx
import { lazy, Suspense } from 'react';
// ... other import codes
const AdminPage = lazy(() => import('./pages/AdminPage'));

const router = createBrowserRouter([
  {
    path: '/',
    element: <App />,
    errorElement: <ErrorPage />,
    children: [
      // .. Other routes are here
      {
        path: 'admin',
        element: (
          <Suspense fallback={<div>Loading...</div>}>
            <AdminPage />
          </Suspense>
        ),
      },
    ],
  },
]);
```

The suspense will wait until the page is required and while it's loading it will set the output to be div inside the fallback parameter of Suspense. Once the data is loaded, AdminPage will show up.
