# What's new in React 19

## Actions 
A common use case in React apps is to perform a data mutation and then update state in response. For example, when a user submits a form to change their name, you will make an API request, and then handle the response. In the past, you would need to handle pending states, errors, optimistic updates, and sequential requests manually.

For example, you could handle the pending and error state in `useState`:

> React 애플리케이션에서 주로 사용하는 경우는 데이터를 변경하고 그에 따라 상태를 업데이트하는 것입니다. 예를 들어, 사용자가 이름을 변경하기 위해 폼을 제출하면, API 요청을 수행하고 응답을 처리해야 합니다. 과거에는 대기(pending), 오류, 낙관적 업데이트(optimistic update) 및 순차적 요청을 수동으로 처리해야 했습니다.
>
> 예를 들어, `useState`를 사용하여 대기 상태와 오류 상태를 처리할 수 있습니다:

```tsx
// Before Actions
function UpdateName({}) {
  const [name, setName] = useState("");
  const [error, setError] = useState(null);
  const [isPending, setIsPending] = useState(false);

  const handleSubmit = async () => {
    setIsPending(true);
    const error = await updateName(name);
    setIsPending(false);
    if (error) {
      setError(error);
      return;
    } 
    redirect("/path");
  };

  return (
    <div>
      <input value={name} onChange={(event) => setName(event.target.value)} />
      <button onClick={handleSubmit} disabled={isPending}>
        Update
      </button>
      {error && <p>{error}</p>}
    </div>
  );
}
```

In React 19, we’re adding support for using async functions in transitions to handle pending states, errors, forms, and optimistic updates automatically.

For example, you can use `useTransition` to handle the pending state for you:

> React 19에서는 비동기 함수(async func)를 트랜지션에서 사용하여 대기 상태, 오류, 폼 및 낙관적 업데이트를 자동으로 처리할 수 있도록 지원합니다.
>
> 예를 들어, `useTransition`을 사용하여 대기 상태를 자동으로 처리할 수 있습니다:

```tsx
// Using pending state from Actions
function UpdateName({}) {
  const [name, setName] = useState("");
  const [error, setError] = useState(null);
  const [isPending, startTransition] = useTransition();

  const handleSubmit = () => {
    startTransition(async () => {
      const error = await updateName(name);
      if (error) {
        setError(error);
        return;
      } 
      redirect("/path");
    })
  };

  return (
    <div>
      <input value={name} onChange={(event) => setName(event.target.value)} />
      <button onClick={handleSubmit} disabled={isPending}>
        Update
      </button>
      {error && <p>{error}</p>}
    </div>
  );
}
```

The async transition will immediately set the `isPending` state to true, make the async request(s), and switch `isPending` to false after any transitions. This allows you to keep the current UI responsive and interactive while the data is changing.

> 비동기 전환은 즉시 `isPending` 상태를 true로 설정하고, 비동기 요청을 수행한 후 전환이 완료되면 `isPending`을 false로 전환합니다. 이를 통해 데이터가 변경되는 동안 현재 UI를 응답성 있고 상호작용 가능하게 유지할 수 있습니다.

### 💡 Note

**By convention, functions that use async transitions are called “Actions”.**

Actions automatically manage submitting data for you:

- **Pending state**: Actions provide a pending state that starts at the beginning of a request and automatically resets when the final state update is committed.
- **Optimistic updates**: Actions support the new `useOptimistic` hook so you can show users instant feedback while the requests are submitting.
- **Error handling**: Actions provide error handling so you can display Error Boundaries when a request fails, and revert optimistic updates to their original value automatically.
- **Forms**: `<form>` elements now support passing functions to the `action` and `formAction` props. Passing functions to the `action` props use Actions by default and reset the form automatically after submission.

> ### 💡 참고
> **컨벤션에 따라, 비동기 트랜지션을 사용하는 함수는 "액션(Actions)"이라고 부릅니다.**
>
> 액션은 데이터를 자동으로 제출하는 과정을 관리합니다:
>
> - **대기 상태**: 액션은 요청이 시작될 때 대기 상태를 제공하고, 최종 상태 업데이트가 완료되면 자동으로 리셋됩니다.
> - **낙관적 업데이트**: 액션은 새로운 `useOptimistic` 훅을 지원하여 요청이 제출되는 동안 사용자가 즉각적인 피드백을 받을 수 있도록 합니다.
> - **오류 처리**: 액션은 오류 처리를 제공하여 요청이 실패할 때 에러 바운더리를 표시하고, 낙관적 업데이트를 원래 값으로 자동으로 되돌립니다.
> - **폼**: `<form>` 요소는 이제 `action` 및 `formAction` props에 함수를 전달하는 것을 지원합니다. `action` props에 함수를 전달하면 기본적으로 액션을 사용하며, 제출 후 자동으로 폼을 리셋합니다.

Building on top of Actions, React 19 introduces `useOptimistic` to manage optimistic updates, and a new hook `React.useActionState` to handle common cases for Actions. In `react-dom` we’re adding `<form>` Actions to manage forms automatically and `useFormStatus` to support the common cases for Actions in forms.

In React 19, the above example can be simplified to:

> 액션을 기반으로, React 19에서는 낙관적 업데이트를 관리하기 위한 `useOptimistic`과 액션의 일반적인 사례를 처리하기 위한 새로운 훅 `React.useActionStat`e를 도입합니다. `react-dom`에서는 폼을 자동으로 관리하기 위한 `<form>` 액션과 폼에서 액션의 일반적인 사례를 지원하기 위한 `useFormStatus`를 추가하고 있습니다.
>
> React 19에서는 위의 예시를 다음과 같이 단순화할 수 있습니다:

```tsx
// Using <form> Actions and useActionState
function ChangeName({ name, setName }) {
  const [error, submitAction, isPending] = useActionState(
    async (previousState, formData) => {
      const error = await updateName(formData.get("name"));
      if (error) {
        return error;
      }
      redirect("/path");
      return null;
    },
    null,
  );

  return (
    <form action={submitAction}>
      <input type="text" name="name" />
      <button type="submit" disabled={isPending}>Update</button>
      {error && <p>{error}</p>}
    </form>
  );
}
```

In the next section, we’ll break down each of the new Action features in React 19.

> 다음 장에서는, React 19 액션의 새로운 기능들을 살펴보겠습니다.
