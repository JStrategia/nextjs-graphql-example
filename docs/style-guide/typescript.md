# TypeScript

## No. Damn. Classes.

Supported by linter: **NO**

We keep this codebase FP-oriented, following the KISS principle. No classes should be present to avoid dealing with
state induced problems.

This codebase should only include functions and types. Nothing else.

This is especially true for static class methods. Never do that. Simply create functions and use them as you would.

Incorrect:

```ts
class AccountService {
  constructor(private readonly repository: AccountRepository) {}

  getAccounts() {
    return repository.getAccounts();
  }
}

const accountService = new AccountService(accountRepository);
const accounts = await accountService.getAccounts();
```

Correct:

```ts
function makeAccountService(repository: AccountRepository) {
  function getAccounts() {
    return repository.getAccounts();
  }

  // "private" functions = not returned
  return { getAccounts };
}

const accountService = makeAccountService(accountRepository);
const accounts = await accountService.getAccounts();
```

## Prefer types to interfaces

Supported by linter: **NO**

Interfaces are useless when types exist. They both do the same job, but types are more strict and are more FP-oriented.

[More info](https://fettblog.eu/tidy-typescript-prefer-type-aliases/)

Incorrect:

```ts
interface AccountRepository {
  getAccounts(): Promise<Account[]>;
}
```

Correct:

```ts
type AccountRepository = {
  getAccounts(): Promise<Account[]>;
};
```

Even better, when you don't need multiple implementations:

```ts
function makeAccountRepository() {
  function getAccounts() {
    return [];
  }
}

type AccountRepository = ReturnType<typeof makeAccountRepository>;
```

## No arrow functions

Supported by linter: **NO** ([might help](
https://mysticatea.github.io/eslint-plugin-es/rules/no-arrow-functions.html))

Arrow functions can bring confusion, it's hard to see what is a value and what is a function. To avoid this situation,
always use `function`.

Incorrect:

```ts
const double = (a: number) => a * 2;
```

Correct:

```ts
function double(a: number) {
  return a * 2;
}
```

## No function return types

Supported by linter: **NO**

Specifying function return types often leads to imports only used for this exact reason. To avoid this, we can let
our IDEs tell us when a function returns an unwanted type. This will also be caught be the CI when building the app.

Incorrect:

```ts
import { Account } from 'types/accounts';

function getAccounts(): Promise<Account[]> {
  return [{ firstName: 'John', lastName: 'Doe' }];
}
```

Correct:

```ts
function getAccounts() {
  return [{ firstName: 'John', lastName: 'Doe' }];
}
```

## Wrap multiple params in a type

Supported by linter: **NO**

For readability and usability, we prefer using a single type as function param **when the amount of params is more than one**.
This reduces ripple effect when we want to switch around params placement and lets the function user decide in which order
params are sent (defined in type). This also reduces the amount of param type definition from n (number of values) to 1
(params type).

Incorrect:

```ts
// Definition
function getAccountUri(baseUrl: string, accountId: string) {
  return `${baseUrl}/accounts/${accountId}`;
}

// usage
const accountUri = getAccountUri('https://example.com', '123');
```

Correct:

```ts
// Definition
type GetAccountUriParams = {
  baseUrl: string;
  accountId: string;
}

function getAccountUri({ baseUrl, accountId }: GetAccountUriParams) {
  return `${baseUrl}/accounts/${accountId}`;
}

// usage
const accountUri = getAccountUri({ baseUrl: 'https://example.com', accountId: '123' });
```

## Imports first, exports last

Supported by linter: **NO** ([might help](
https://github.com/import-js/eslint-plugin-import/blob/main/docs/rules/exports-last.md))

We should read script files (including JS/TS) as functions, with params (imports) and returned values (exports). This
means files should always before with imports and end with exports.

Separate declarations from exports.

Incorrect:

```tsx
import { useState } from 'react';

export default function AccountPage() {
  const [account, setAccount] = useState<Account>(null);

  return <Container>Something<Container/>;
}

const Container = styled.div`
  color: black;
`;
```

Correct:

```tsx
import { useState } from 'react';

function AccountPage() {
  const [account, setAccount] = useState<Account>(null);

  return <Container>Something<Container/>;
}

const Container = styled.div`
  color: black;
`;

export default AccountPage;
```

## Avoid state mutation

Supported by linter: **NO**

Always make sure not to mutate the state without re-creating it. Best example is mutating function params. Remember to
always return the new value.

Incorrect:

```ts
function updateFirstName({ account, firstName }: UpdateFirstNameParams) {
  account.firstName = firstName;
  return account;
}
```

Correct:

```ts
function updateFirstName({ account, firstName }: UpdateFirstNameParams) {
  return {
    ...account, // Shallow copy, used as an example
    firstName,
  };
}
```