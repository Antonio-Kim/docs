## Working with forms in React

There are various methods of handling forms in react: controlled via useState, uncontrolled via native html form, and React Router Form. Here we'll focus on using React Hook Form.

## React Hook Form

React Hook Form has the following structure:

```tsx
const {
  register,
  handleSubmit,
  formState: { errors, isSubmitting, isSubmitSuccessful },
} = useForm<FormData>();
```

the register function is important in useForm. It takes in unique field name and returns the following properties and functions:

- onChange
- onBlur
- reference to field editor element
- field name

To implement React Hook Form, the basic setup is as follow:

```tsx
<input {...register('name')}>
```

as mentioned above, it returns four different functions and properties.

```tsx
<input
  ref={someVariableInRHF}
  name="name"
  onChange={someVaribleInRHF}
  onBlur={anotherHandlerInRHF}
>
```

optionally, we can set required like native form

```tsx
<input {...register('name', { required: 'You must enter a name' })} />
```

You do not need to write the message. This field can be set to simply true.

### Example

We'll use generic cases instead of specific implementation. On top of the component function, add the following:

```tsx
import { useForm } from 'react-hook-form';
import { useNavigate } from 'react-router-dom';

type Contact = {
  name: string;
  email: string;
};

export function Contact() {
  const { register, handleSubmit } = useForm<Contact>();
  const navigate = UseNavigate;
  function onSubmit(contact: Contact) {
    console.log('submitted details:', contact);
    navigate(`/thank-you/${contact.name}`);
  }
  <form onSubmit={handleSubmit(onSubmit)}>
    <div>
      <label htmlFor="name">Your Name</label>
      <input type="text" id="name" {...register('name')}>
    </div>
    <div>
    <label htmlFor="email">Your Email</label>
    <input type="email" id="email" {...register('email')}>
  </form>;
}
```

We're using useNavigate from React Router so that after submission the page gets redirected to Home page.

### Validation

We can add validation mode on React Hook Form after register name:

```tsx
<form onSubmit={handleSubmit(onSubmit)}>
  <div>
    <label htmlFor="name">Your Name</label>
    <input type="text" id="name" {...register('name', { required:'Please enter your name' })}>
  </div>
  <div>
  <label htmlFor="email">Your Email</label>
  <input type="email" id="email" {...register('email', { required:'Please enter your email' })}>
</form>;
```

You should add various components inside the form to display error messages but we won't go in here. We can also set our form to re-evaluate the inputs when it loses focus on the input field:

```tsx
import { useForm } from 'react-hook-form';
import { useNavigate } from 'react-router-dom';

type Contact = {
  name: string;
  email: string;
};

export function Contact() {
  const { register, handleSubmit } = useForm<Contact>({
    mode: 'onBlur',
    reValidateMode: 'onBlur',
  });
  const navigate = UseNavigate;
  function onSubmit(contact: Contact) {
    console.log('submitted details:', contact);
    navigate(`/thank-you/${contact.name}`);
  }
  <form onSubmit={handleSubmit(onSubmit)}>
    <div>
      <label htmlFor="name">Your Name</label>
      <input type="text" id="name" {...register('name')}>
    </div>
    <div>
    <label htmlFor="email">Your Email</label>
    <input type="email" id="email" {...register('email')}>
  </form>;
}
```

The complete code for simple validation is shown above
