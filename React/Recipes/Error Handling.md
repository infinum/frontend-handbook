# Motivation

Handling errors is really important for any production ready app. By proper handling of error we can prevent degradation of user experience in production.
Luckily,  React gave use the right tool for this job, and it's called [Error Boundary](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary).

## Used modules

1. [SWR](https://github.com/vercel/swr) ([docs](https://swr.vercel.app/))
2. [@bugsnag/plugin-react](https://github.com/bugsnag/bugsnag-js/tree/next/packages/plugin-react) ([docs](https://docs.bugsnag.com/platforms/javascript/react/))

## Error handling done right!

```tsx
const Announcements = ({ eventId }) => {
  const { data, error } = useSWR(`/api/v1/announcements?filter[event_id]=${eventId}`);

  if (error) {
    // Error is handled outside, in BugsnagErrorBoundary
    throw error;
  }

  if (!data) {
    // Loading data
    return <AnnouncementsSkeleton />
  }

  return (
    <div>
      {data?.map(announcement) => <AnnouncementCard key={announcement.id} announcement={announcement} />}
    </div>
  );
}
```

Then in the parent component:

```tsx
<BugsnagErrorBoundary FallbackComponent={ErrorFallback}>
  <Announcements eventId={eventId} />
</BugsnagErrorBoundary>
```

```tsx
// ./src/components/shared/overlays

const ErrorFallback = (error, info, clearError) => {
  return <button onClick={clearError}>Refresh</button>;
};
```

## What do you get with this:

1. You are only handling a happy path in the component
2. Reusable error handling for API calls on one place
3. Error handling for any JS error that could happen in the production
4. Only this part of the app will stop working, not the whole App
5. Bugsnag reports

## NextJS App Router

Using an `error.tsx` file in your app’s directory structure is recommended for handling global and route-level errors, however, it’s not mandatory if you want more fine-grained control over error handling at the component level using custom error boundaries (like the Bugsnag example).

For a robust error-handling strategy, it’s often best to combine both approaches:

* Global Fallback with `error.tsx`: Fallback for unhandled errors across routes and layouts. This ensures users always see a helpful message rather than a blank screen if an error goes uncaught by component-level boundaries.
* Component-Level Error Boundaries: Use them around specific components or sections that might fail due to network issues, user input, or other isolated factors. This allows you to manage errors locally without impacting the whole route and offers a better experience for users.

## Useful links

1. [Bugsnag with Next.js pages router example](https://github.com/bugsnag/bugsnag-js/tree/master/examples/js/nextjs)
2. [Use react-error-boundary to handle errors in React](https://kentcdodds.com/blog/use-react-error-boundary-to-handle-errors-in-react)
3. If you don't have Bugsnag you can use this npm module [react-error-boundary](https://www.npmjs.com/package/react-error-boundary)
