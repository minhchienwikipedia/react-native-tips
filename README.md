# React Native Tips 

This repo will show you the tips in React Native, this is my experience when I'm working on it.
Feel free to create the PR to share your tips!

<br><p><a href="https://www.buymeacoffee.com/minhchien"> <img align="left" src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" height="50" width="210" alt="fateh999" /></a></p><br><br><br>

### Here is the tips

- [React Navigation](#react-navigation)
    - [Navigate from anywhere](#navigate-from-anywhere)
    - [Auto go back when navigate to same screen, same params](#auto-go-back-when-navigate-to-same-screen-same-params)
- [React Redux](#react-redux)
    - [Dispatch the function from anywhere](#dispatch-the-function-from-anywhere)
    - [Use shallow compare for `useSelector`](#use-shallow-compare-for-useSelector)
    - [Only defined the values we want to use](#only-defined-the-values-we-want-to-use)



# React Navigation
1. #### Navigate from anywhere

- Step 1: Create the `RootNavigation.js` file like this
```javascript
import React from 'react';

export const navigationRef = React.createRef();

export const NavigationAction = navigationRef.current;


export function navigate(name, params) {
    navigationRef.current?.navigate?.(name, params);
}

export function goBack() {
    if (navigationRef.current?.canGoBack?.()) {
        navigationRef.current?.goBack?.();
    }
}
```

- Step 2: Add `navigationRef` to your `NavigationContainer`
```javascript
import { navigationRef } from './routes/RootNavigation';

<NavigationContainer ref={navigationRef}>
    <AppContainer />
</NavigationContainer>
```

- Step 3: Enjoy it
```javascript
import { navigate, goBack } from './routes/RootNavigation';

navigate('Home', { params });

goBack()
```

**[⬆ Back to Top](#here-is-the-tips)**

2. #### Auto go back when navigate to same screen, same params
- Step 1: Setup [Navigate from anywhere](#navigate-from-anywhere)
- Step 2: Update navigate function
```javascript
import { StackActions } from '@react-navigation/routers';
import isEqual from 'lodash/isEqual';

export function navigate(name, params) {
    const pushAction = StackActions.push(name, params);
    const { routes = [] } = navigationRef.current?.getRootState?.() || {};
    const isExist =
        routes.findIndex((item) => {
            if (item.name === name && isEqual(item.params, params)) {
                return true;
            }
            return false;
        }) >= 0;
    if (isExist) {
        navigationRef.current?.navigate?.(name, params);
        return;
    }
    navigationRef.current?.dispatch?.(pushAction);
}
```
Explain: in [React Navigation docs](https://reactnavigation.org/docs/navigating/#summary) `navigate` will go back to that screen if it's already in stack, `push` will push anyway it's already or not. So we will check if target screen already in stack or not, if it's already and same params we will use navigate for react navigation handle go back, if it's not we will use `push` to have same screen but different params.

- Step 3: Enjoy it

**[⬆ Back to Top](#here-is-the-tips)**

# React Redux

1. #### Dispatch the function from anywhere

- Step 1: Create the `ReduxDispatcher.js` file like this
```javascript
import React from 'react';

export const dispatchRef = React.createRef();

const ReduxDispatcher = (action) => {
    dispatchRef.current?.(action);
};

export default ReduxDispatcher;
```

- Step 2: Open the file where you had setup the your `Provider` 
```javascript
import { store } from './store';

useEffect(() => {
    dispatchRef.current = store.dispatch;
    return () => {};
}, []);

<Provider store={store}>
// Your App
</Provider>
```

- Step 3: Enjoy it
```javascript
import ReduxDispatcher from '@/store/ReduxDispatcher';

ReduxDispatcher(updateUserInfo(params));
```

**[⬆ Back to Top](#here-is-the-tips)**

2. #### Use shallow compare for `useSelector`
When we defined the value in useSelector it didn't check the object as well so we will use shallow compare to make it re-render only when the object had changed.

- Step 1: Create `useShallowEqualSelector` function
```javascript
import { shallowEqual, useSelector } from 'react-redux';

export function useShallowEqualSelector(selector) {
    return useSelector(selector, shallowEqual);
}
```

- Step 2: Enjoy it

```javascript
const { uid } = useShallowEqualSelector((state) => ({
        uid: state.userInfo?.uid,
}));
```

**[⬆ Back to Top](#here-is-the-tips)**

3. #### Only defined the values we want to use

❌ Wrong
```javascript
const { userInfo } = useShallowEqualSelector((state) => ({
        userInfo: state.userInfo,
}));

render(){
    return <Text>{userInfo.userName}</Text>
}
```
----
✅ Correct
```javascript
const { userName } = useShallowEqualSelector((state) => ({
        userName: state.userInfo.userName,
}));

render(){
    return <Text>{userName}</Text>
}
```
Why❓
Because when you defined the object but you only want to use some fields in there it will re-render when you dont want to. Example, you just want to use `userName` but you defined the `userInfo` object so when `userInfo.address` changed your component will re-render.

**[⬆ Back to Top](#here-is-the-tips)**

#### [Welcome to contribute](https://github.com/minhchienwikipedia/react-native-tips/pulls)
