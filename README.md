# React Micro-Query

Bare bones [query](#query) and [mutation](#mutation) hooks for React.

Inspired by (and API compatible with) React Query's [useQuery](https://tanstack.com/query/v4/docs/reference/useQuery) and [useMutation](https://tanstack.com/query/v4/docs/reference/useMutation) hooks.

**It does:**

- Perform asynchronous reads, writes, and other affects.
- Support polling on an interval.

**It does not:**

- Cache or support [SWR](https://www.toptal.com/react-hooks/stale-while-revalidate).
- Retry failed requests.
- Use a `<QueryClientProvider>` 

By removing caching and retries, the API is simplified, the library size is reduced, there is no need for a `<QueryClientProvider>` wrapper to provide a shared `QueryClient`. There are other solutions and libraries with caching and retrying as their core capability, which can be composed with this library. [Small is beautiful, and each library does one thing well.](https://opensource.com/business/15/2/how-linux-philosophy-affects-you)

## Query

The `useQuery` hook is for asynchronously reading data.

This hook is suitable for [safe](https://developer.mozilla.org/en-US/docs/Glossary/Safe/HTTP) operations which should not have any side effects (eg. HTTP GET, HEAD, OPTIONS, and TRACE requests)

**It includes:**

- Results
  - `data`
  - `error`
  - `isFetching`
  - `refetch()`
- Options
  - `enabled`
  - `refetchInterval`
  - `refetchOnReconnect`
  - `refetchOnWindowFocus`
- Query Context
  - `queryKey`
  - `signal`

The `useQuery` hook can be used directly in components, but generally you should wrap it in a custom hook.

```tsx
const useResource = (id: string): QueryResult<Resource> => {
  const result = useQuery(
    // Query is refetched when the query key (serializable) changes.
    [id],
      const response = await fetch('https://...', { signal: context.signal });

      if (!response.ok) {
        throw new Error(`Request failed (status: ${response.status}, key: ${JSON.stringify(context.key)})`);
      }

      return response.json();
    },
    {
      // Only query when `result.refetch()` is called if false.
      enabled: true, // Default
      // Refetch automatically when positive non-zero.
      refetchInterval: 0, // Default
      // Refetch when connectivity is regained if true.
      refetchOnReconnect: true, // Default
      // Refetch when the window regains focus if true.
      refetchOnWindowFocus: true, // Default
    },
  );

  return result;
};
```

Use your custom query hook in a component.

```tsx
const Component = (props: Props): JSX.Element => {
  const { data, error, isFetching, refetch } = useResource(props.id);

  if (isFetching) {
    return <Loading />;
  }

  if (error != null) {
    return <Error error={error} />;
  }

  return <Resource data={data} onRefresh={refetch} />;
};
```

## Mutation

The `useMutation` hook is for asynchronously creating, updating, and deleting data.

This hook is suitable for operations which may be [unsafe](https://developer.mozilla.org/en-US/docs/Glossary/Safe/HTTP) due to side effects (eg. POST, PUT, PATCH, and DELETE requests).

**It includes:**

- Results
  - `data`
  - `error`
  - `isLoading`
  - `mutate()`
- Options
  - `onMutate`
  - `onSettled`

The `useMutation` hook can be used directly in components, but generally you should wrap it in a custom hook.

```tsx
const useCreateResource = (): MutationResult<Resource> => {
  const result = useMutation(
    async (resource: Resource): Resource => {
      const response = await fetch('https://...', {
        method: 'POST',
        headers: { 'content-type': 'application/json' },
        body: JSON.stringify(resource),
      });

      if (!response.ok) {
        throw new Error(`Request failed (status: ${response.status})`);
      }

      return response.json();
    },
  );

  return result;
};
```

Use your custom mutation hook in a component.

```tsx
const Component = (props: Props): JSX.Element => {
  const { data, error, isLoading, mutate } = useCreateResource();
  const onSave = useCallback((resource: Resource) => {
    mutate(resource)
  }, [mutate]);

  return (
    <div>
      {isLoading && <Saving />}
      {error && <Error error={error} />}
      {data && <Success data={data} />}
      <CreateResourceForm enabled={!isLoading} onSave={onSave} />
    </div>
  )
};
```
