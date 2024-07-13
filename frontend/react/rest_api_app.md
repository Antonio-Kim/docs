## Creating React App with REST API capability

There are many ways to interact with REST API, and most common usage is through Axios library. For now, we'll simply use Fetch but Axios will be added later. For dev, we'll setup a simple json server to interact with data. To setup json-server, add the following to the react library:

```bash
npm i -D json-server
```

Then create a .json file with data set. Each key of the json file is considered a subroute, and the array inside will be the file that will be sent to the client. Inside the package.json file we can set up easier script rather than typing out:

```json
"scripts": {
	"server: "json-server --watch db.json --port 3001"
	}
}
```

Setup .env file so that react could fetch the data easier than hard-coding the route. Now create a helper file to fetch data:

```ts
export async function getPosts() {
  const resonse = await fetch(process.env.REACT_APP_API_URL);
  const body = (await response.json()) as unknown;
  // should have validation here for body keys and types
  return body;
}
```

After deciphering the body data, we can create a list of component from fetch using `<ul>` and map it to a list. Ensure that each list has key assigned as an id. Say we created a list component called <PostsList posts:PostData[]> where it takes props of posts, with type PostData[], and subsequently PostData consisting `{id:number; title: string; description: string}`. We'll create a page of Posts called _PostsPage.tsx_, that includes the PostList. Since we're using a simple app, we'll stick with useState from react library, but as the app gets bigger, this should be transition to Redux instead.

```tsx
import { useEffect, useState } from 'react';
import { getPosts } from './getPosts';
import { PostData } from './types';
import { PostsLists } from './PostsLists';

export function PostsPage() {
  const [isLoading, setIsLoading] = useState(true);
  const [posts, setPosts] = useState<PostData[]>([]);

  useEffect(() => {
    let cancel = false;
    getPosts().then((data) => {
      if (!cancel) {
        setPosts(data);
        setIsLoading(false);
      }
    })
    return () => {
      cancel = true;
    };
  }, []);

  if (isLoading) {
    return (
      <div>Loading... </div>
    )
  }
  return (
    <div>
      <h2>Posts</h2>
      <PostsList post={posts}>
    </div>
  )
}
```

### Posting data with fetch

Let's create a helper function first. Start with types

```ts
export type NewPostDat = {
  title: string;
  description: string;
};
export type SavedPostData = {
  id: number;
};
```

create a file named _savePost.ts_ that will save the data in the json-server

```ts
import { NewPostData, SavedPostData } from './types';
export async function savePost(newPostData: NewPostData) {
  const response = await fetch(process.env.REACT_APP_API_URL!, {
    {
      method: 'POST',
      body: JSON.stringify(newPostData),
      headers: {
        'Content-Type': 'application/json',
      },
    }
  });
  const body = (await response.json()) as unknown;
  // validation here
  return { ...newPostData, ...body };
}
```

Now let's create a submit form component. To use this, make sure you have `react-hook-form` installed in your repo to simplify form submission. Create a new component called _NewPostForm.tsx_ and add the following codes:

```tsx
import { FieldError, useForm } from 'react-hook-form';
import { NewPostData } from './types';
type Props = {
  onSave: (newPost: NewPostData) => void;
};
export function NewPostFor({ onSave }: props) {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting, isSubmitSuccessful },
  } = useeForm<NewPostData>();
  return (
    <form onSubmit={handleSubmit(onSave)}>
      <div>
        <label>Title</label>
        <input type="text" id="title" {...register('title')} />
      </div>
      <div>
        <label>Description</label>
        <textarea id="description" {...register('description')} />
      </div>
      <div>
        <button type="submit" disabled={isSubmitting}>
          Save
        </button>
      </div>
    </form>
  );
}
```

Let's put the form on top of the list of posts. Inside the _PostsPage.tsx_, add the component on top of the list:

```tsx
<div>
  <h2>Posts</h2>
  <NewPostForm onSave={handleSave} />
  <PostsList posts={posts} />
</div>
```

We'll need to handle the onSave, so let's create a function right below the useEffect:

```tsx
async function handleSave(newPostData: NewPostData) {
  const newPost = await savePost(newPostData);
  setPosts([newPost, ...posts]);
}
```

### Refactoring with React Router

We now want to set a page for the app, but also allow lazy loading to the app, since it will need to fetch data from the server. The lazy loading can be enabled using async/await and **loader** key-value inside _createBrowserRouter_ array. Inside the App.tsx add the following:

```tsx
import { createBrowserRouter, RouterProvider, defer } from 'react-router-dom';
import { getPosts } from './posts/getPosts';

const router = createBrowserRouter([
  {
    path: '/',
    element: <PostsPage />,
    loader: async () => defer({ posts: getPosts() }),
  },
]);

function App() {
  return <RouterProvider router={router} />;
}
```

Now that it has been configured at the top level, we'll need to modify the _PostsPage.tsx_ since this is where list of posts are being loaded. We'll need to wrap the list component with `<Suspense>` and `<Await>` component from React library and React Router library, respectively.

```tsx
import { Suspense } from 'react';
import { useLoaderData, Await } from 'react-router-dom';
// useLoaderData is used to... load data
const data = useLoaderData();
// ... codes here
return (
  <div>
    <h2>Posts</h2>
    <NewPostForm onSave={handleSave} />
    <Suspense fallback={<div>Fetching...</div>}>
      <Await resolve={data.posts}>
        {(posts) => {
          return <PostsList posts={posts} />;
        }}
      </Await>
    </Suspense>
  </div>
);
```

### Using React Query

We can do better with caching using React Query than using React Router. React Router reduces number of re-renders while React Query provides client-side cache of data. Let's modify our app using React Query. Install the following library:

```bash
npm i @tanstack/react-query
```

Inside _App.tsx_ we'll wrap around our Router to be able to use Query

```tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
const queryClient = new QueryClient();
const router = createBrowser(...);
function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <RouterProvider router={router}>
    </QueryClientProvider>
  );
};
```

Now inside the _PostsPage.tsx_ we'll import useQuery hook and fetch the data from the server.

```tsx
import { useQuery } from '@transtack/react-query';
import { assertIsPosts, getPosts } from './getPosts';

export function PostsPage() {
  const {
    isLoading,
    isFetching,
    data: posts,
  } = useQuery(['postsData'], getPosts);
  const data = useLoaderData();
  if (isLoading || posts === undefined) {
    return <div>Loading...</div>;
  }
  return (
    <div>
      <h2>Posts</h2>
      <NewPostForm onSave={handleSave} />
      {isFetching ? <div>Fetching ...</div> : <PostsList posts={posts} />}; //{' '}
      <Await /> here..
    </div>
  );
}
```

The first argument passed to useQuery is a unique key for the data. This is because React Query can store many datasets and uses the key to identify each one. The key is an array containing the name given to the data in our case. There are also destructed functions after useQuery

- isLoading: check if it is first time when loading
- isFetching: check whether fetch function is being calle
- data: data that's being fetched. Alias to name posts instead.

Now onto posting new data. React Query uses **mutations** to update data using hook called **useMutation**. We'll update the same PostsPage.tsx by add Navigate from reactor dom and after submission, redirect to home.

```tsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { useLoaderData, Await, useNavigate } from 'react-router-dom';
export function PostsPage() {
  const {
    isLoading,
    isFetching,
    data: posts,
  } = useQuery(['postsData'], getPosts);
  const data = useLoaderData();
  // mutation is added here
  const { mutate } = useMutation(savePosts, {
    onSuccess: (savedPost) => {
      queryClient.setQueryData<PostData[]>(
        ['postsData'],
        (oldPosts) => {
          if (oldPosts === undefined) {
            return [savedPost];
          } else {
            return [savedPost, ...oldPosts];
          })
        }
      );
      navigate('/');
    }
  });
  if (isLoading || posts === undefined) {
    return <div>Loading...</div>;
  }
  return (
    <div>
      <h2>Posts</h2>
      <NewPostForm onSave={mutate} />
      {isFetching ? <div>Fetching ...</div> : <PostsList posts={posts} />}
      <Suspense fallback={<div>Fetching...</div>}>
        <Await resolve={data.posts}>
          {(posts) => {
          return <PostsList posts={posts} />;
          }}
        </Await>
      </Suspense>
    </div>
  );
}
```

We then need to modify the _App.tsx_ so that both React Router and React Query can work together

```tsx
const router = createBrowserRouter([
  {
    path: '/',
    element: ...,
    loader: async () => {
      const existingData = queryClient.getQueryData([
        'postsData',
      ]);
      if (existingData) {
        return defer({ posts: existingData });
      }
      return defer( {
        posts: queryClient.fetchQuery(['postsData'], getPosts);
      })
    }
  }
])
```
