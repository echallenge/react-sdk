# fractal-sdk

### Installation

```sh
npm install @fractalwagmi/fractal-sdk
```

### Example

[SDK Demo Preview](https://sdk-demo.fractalpreview.com/)

### Usage

1. Setup the provider above any components that need access to the available
   hooks.

```tsx
import { FractalProvider } from '@fractalwagmi/fractal-sdk';

const App = () => {
  return <FractalProvider>...</FractalProvider>;
};
```

2. Use the hooks.

```tsx
import {
  Scope,
  SignIn,
  useCoins,
  useItems,
  User,
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

  return (
    <>
      <SignIn
        clientId="YOUR_CLIENT_ID"
        scopes={[Scope.IDENTIFY, Scope.ITEMS_READ, Scope.COINS_READ]}
        onError={err => {
          console.log('err = ', err);
        }}
        onSuccess={(user: User) => {
          console.log('user = ', user);
        }}
      />
    </>
  );
}
```
