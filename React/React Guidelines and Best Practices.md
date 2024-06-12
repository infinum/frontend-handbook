*Guides are not rules and should not be followed blindly. Use your head and think.*

### Split components in smaller ones to increase reusability
By splitting components into smaller reusable ones, you're also potentially lowering the bundle size.

```jsx
// BAD
const UserProfile = ({ user }) => (
  <div>
    <div>
      <img src={avatar} alt="User Avatar" />
      <h1>{name}</h1>
      <p>{bio}</p>
    </div>
    {user.posts.map(post => <p key={post.id}>{post.content}</p>)}
  </div>
);
```

```jsx
// GOOD
const UserProfileHeader = ({ avatar, name, bio }) => (
  <div>
    <img src={avatar} alt="User Avatar" />
    <h1>{name}</h1>
    <p>{bio}</p>
  </div>
);

const UserPosts = ({ posts }) => posts.map(post => <p key={post.id}>{post.content}</p>);

const UserProfile = ({ user }) => (
  <div>
    <UserProfileHeader
      avatar={user.avatar}
      name={user.name}
      bio={user.bio}
    />
    <UserPosts posts={user.posts} />
  </div>
);
```

### Avoid Inline Functions and Objects
Avoid defining functions and objects inside the component render methods to prevent unnecessary re-renders.

```jsx
// BAD
const MyComponent = () => {
  const handleClick = () => {
    console.log('Clicked');
  };

  return <button onClick={handleClick}>Click me</button>;
};
```

```jsx
// GOOD
const handleClick = () => {
  console.log('Clicked');
};

const MyComponent = () => (
  <button onClick={handleClick}>Click me</button>
);
```


### Memoize Expensive Calculations
Use useMemo and useCallback to memoize expensive calculations and functions, but only for truly expensive calculations to avoid unnecessary complexity.

