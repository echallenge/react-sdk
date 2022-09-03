# fractal-sdk

## Before Integrating

You will need to provide us with a list of allowed origins (e.g. http://localhost,
https://your.game.com). Please reach out to us at developers@fractal.is and let
us know the origins you would like to allow when you are ready to start
integrating.

## Installation

```sh
npm install @fractalwagmi/fractal-sdk
```

## Example

[SDK Demo Preview](https://sdk-demo.fractalpreview.com/)

## Setup and Authentication

### 1. Import the global stylesheet

If using the provided default button rather than your own UI, import our global
stylesheet to ensure that the component renders correctly:

```
import '@fractalwagmi/fractal-sdk/styles.css';
```

### 2. Set up the provider

Render the provider above any components that need access to data from the SDK.

```tsx
import { FractalProvider } from '@fractalwagmi/fractal-sdk';

const App = () => {
  return <FractalProvider clientId="YOUR_CLIENT_ID">...</FractalProvider>;
};
```

### 3A. Render the `SignInWithFractal` component

This will display a button for logging in.

```tsx
import {
  Scope,
  SignInWithFractal,
  User,
  FractalSDKError,
} from '@fractalwagmi/fractal-sdk';

export function YourSignInComponent() {
  return (
    <SignInWithFractal
      // `scopes` defaults to [Scope.IDENTIFY].
      scopes={[Scope.IDENTIFY, Scope.ITEMS_READ, Scope.COINS_READ]}
      onError={(err: FractalSDKError) => {
        console.log(err.getUserFacingErrorMessage());
      }}
      onSuccess={(user: User) => {
        console.log('user = ', user);
      }}
    />
  );
}
```

#### SignInWithFractal Props

| Prop          | Type / Description                                                                                                                   | Default            |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------ | ------------------ |
| `buttonProps` | `HTMLAttributes<HTMLButtonElement>`<br/>Any additional props for `<button>` that should be passed to the default sign-in button.     | `{}`               |
| `onError`     | `(e: FractalSDKError) => void`<br/>A callback function to call when an error occurs.                                                 | `undefined`        |
| `onSuccess`   | `(user: User) => void`<br/>A callback function to call when a user successfully signs in.                                            | `undefined`        |
| `onSignOut`   | `() => void`<br/>A callback function to call when a sign out occurs.                                                                 | `undefined`        |
| `scopes`      | `Scope[]`<br/>The scope to assign to the access token. See [src/types/scope.ts](/src/types/scope.ts) for a list of available scopes. | `[Scope.IDENTIFY]` |
| `variant`     | `"light" \| "dark"`<br/>The button style variant to use.                                                                             | `"light"`          |

#### Customizations

By default, there are 2 button variants that we support:

```tsx
const YourComponent = () => {
  // There is a "light" (default) and "dark" variant:
  return <SignInWithFractal variant="dark">;
}
```

You can make minor adjustments with your own class name to override any styles:

```tsx
const YourComponent = () => {
  // Using your own class name to override any default styles:
  return <SignInWithFractal className="foobar">;
};
```

### 3B: Render Your Own Button

You can go headless and render your own button with the help of the
`useAuthButtonProps` hook.

This option offers full control over your mark up and styles:

```tsx
import { useAuthButtonProps, Scope } from '@fractalwagmi/fractal-sdk';

const YourButtonComponent = () => {
  const { loading, signedIn, onClick } = useAuthButtonProps({
    scopes: [Scope.IDENTIFY, Scope.ITEMS_READ, Scope.COINS_READ],
  });

  if (loading) {
    return '...';
  }

  return <button onClick={onClick}>{signedIn ? 'Sign Out' : 'Sign In'}</button>;
};
```

`useAuthButtonProps` supports the same props as `SignInWithFractal` except for
the `variant` and `buttonProps`.

**Be sure to add support for both signed in and signed out states** (like in the
example above with the alternating button text,) because the `onClick` prop will
invoke different logic based on the `signedIn` boolean.

## General Error Handling

All error classes are exported from the SDK directly and extend the
`FractalSDKError` class. The `FractalSDKError` class extends the native JS
`Error` class.

All exported error classes have a `getUserFacingErrorMessage` method that
returns a fallback a UI-friendly message, although we encourage you to handle
each error case individually and render UI text that is appropriate for your
application.

You may handle different error cases using an `instanceof` checks to infer the
meaning of the error state:

```tsx
import {
  useSignTransaction,
  FractalSDKSignTransactionDeniedError,
  FractalSDKApprovalOccurringError,
} from '@fractalwagmi/fractal-sdk';

const MyComponent = () => {
  const { data: signature, error } = useSignTransaction({
    unsignedTransactionB58: 'some-string',
  });

  if (error instanceof FractalSDKApprovalOccurringError) {
    return <div>Approving...</div>;
  }
  if (error instanceof FractalSDKSignTransactionDeniedError) {
    return <div>The transaction was denied.</div>;
  }

  return <div>...</div>;
};
```

## Data Hooks

There are a wide variety of hooks that wrap our API functions to give you access
to user data.

```tsx
import {
  useCoins,
  useItems,
  useUser,
  useUserWallet,
} from '@fractalwagmi/fractal-sdk';

export function YourWalletComponent() {
  // Returns user information like email, username, and id.
  const { data: user } = useUser();

  // Returns the user's wallet information like solana public keys.
  const { data: userWallet } = useUserWallet();

  // Returns the items in the user's wallet.
  const { data: items } = useItems();

  // Returns the coins in the user's wallet.
  const { data: coins } = useCoins();

  return <div>...</div>;
}
```

## Functional Hooks

### Signing Out

If you need to programmatically sign the user out, you can use the `useSignOut`
hook:

```tsx
import { useSignOut } from '@fractalwagmi/fractal-sdk';

export function YourWalletComponent() {
  const { signOut } = useSignOut();

  return <button onClick={signOut}>Your Sign Out Button Text</button>;
}
```

### Approving a Generic Transaction

If you need the user to approve a generic transaction, you can create an
unsigned transaction and initialize an approval popup flow for the user to
approve the transaction:

```tsx
import {
  useSignTransaction,
  FractalSDKSignTransactionDeniedError,
} from '@fractalwagmi/fractal-sdk';

interface YourComponentProps {
  someTransactionB58: string | undefined;
}

export function YourComponent({ someTransactionB58 }: YourComponentProps) {
  const {
    data: signature,
    error,
    approving,
    refetch,
  } = useSignTransaction({
    // One thing to keep in mind is that the `unsignedTransactionB58` input
    // parameter is what controls the popup URL. Given the same
    // `unsignedTransactionB58` input, the popup will only be opened once.
    //
    // If you need to re-request approval (because the user denied the
    // transaction,) you can call the `refetch` function that is returned from
    // the hook.
    unsignedTransactionB58: someTransactionB58,
  });

  const denied = e instanceof FractalSDKSignTransactionDeniedError;

  return (
    <div>
      <p>Transaction Signature: {signature}</p>
      <p>An error occurred: {error.getUserFacingErrorMessage()}</p>
      <p>Approval popup is currently {approving ? 'open' : 'closed'}</p>
      {denied ? (
        <button onClick={refetch}>Re-request an approval</button>
      ) : null}
    </div>
  );
}
```

Keep in mind that a signed transaction does not mean that it has been posted to
the chain yet. As of now, this hook only returns a signed transaction signature.

If you need to know when a transaction completes, use the returned transaction signature and
[Solana's JSON RPC API](https://docs.solana.com/developing/clients/jsonrpc-api#gettransaction)
to accomplish this.

#### Error Handling

The `useSignTransaction` hook has multiple error states that is provided in the
`error` property that is returned from the hook:

| Error class                           | meaning                                                                                             |
| ------------------------------------- | --------------------------------------------------------------------------------------------------- |
| FractalSDKAuthenticationError         | An authentication error occurred. This typically means that the user is not properly authenticated. |
| FractalSDKApprovalOccurringError      | An approval flow popup is already open for this hook instance.                                      |
| FractalSDKInvalidTransactionError     | The transaction input was invalid.                                                                  |
| FractalSDKSignTransactionDeniedError  | The transaction was denied.                                                                         |
| FractalSDKSignTransactionUnknownError | An unknown error occurred (catch-all).                                                              |
