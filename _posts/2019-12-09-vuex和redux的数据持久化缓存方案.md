---
layout: post
title: "vuex和redux的数据持久化缓存方案"
date: 2019-12-19
tags: 随笔

color: "rgb(154,133,255)"
---

# vuex

## vuex-persist

Vuex 解决了多视图之间的数据共享问题。但是运用过程中又带来了一个新的问题是，Vuex 的状态存储并不能持久化。也就是说当你存储在 Vuex 中的 store 里的数据，只要一刷新页面，数据就丢失了。

引入[vuex-persist](https://github.com/championswimmer/vuex-persist) 插件，它就是为 Vuex 持久化存储而生的一个插件。不需要你手动存取 storage ，而是直接将状态保存至 cookie 或者 localStorage 中。具体用法如下：

安装：

```js
    npm install --save vuex-persist
    or
    yarn add vuex-persist
```

引入：

```js
import VuexPersistence from "vuex-persist";
```

先创建一个对象并进行配置：

```js
const vuexLocal = new VuexPersistence({
  storage: window.localStorage,
  reducer: state => ({
    systemInfo: state.systemInfo //这个就是存入localStorage的值
  })
});
```

引入进 vuex 插件：

```js
    const store = new Vuex.Store({
      state: { ... },
      mutations: { ... },
      actions: { ... },
      plugins: [vuexLocal.plugin]
    })
```

通过以上设置，各个页面之间跳转，如果刷新某个视图，数据并不会丢失，依然存在，并且不需要在每个 mutations 中手动存取 storage 。

vuex-persist 的详细属性：

| 属性         | 类型                                 | 描述                                                                                                                            |
| ------------ | ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------- |
| key          | string                               | 将状态存储在存储中的键。默认: 'vuex'                                                                                            |
| storage      | Storage (Web API)                    | 可传 localStorage, sessionStorage, localforage 或者你自定义的存储对象. 接口必须要有 get 和 set. **默认是: window.localStorage** |
| saveState    | function (key, state\[, storage\])   | 如果不使用存储，这个自定义函数将保存状态保存为持久性。                                                                          |
| restoreState | function (key\[, storage\]) => state | 如果不使用存储，这个自定义函数处理从存储中检索状态                                                                              |
| reducer      | function (state) => object           | 将状态减少到只需要保存的值。默认情况下，保存整个状态。                                                                          |
| filter       | function (mutation) => boolean       | 突变筛选。看 mutation.type 并返回 true，只有那些你想坚持写被触发。所有突变的默认返回值为 true。                                 |
| modules      | string\[\]                           | 要持久化的模块列表。                                                                                                            |

## 利用 storage 存储 解决防止刷新页面 vuex store 丢失的问题

```js
// 在页面加载时读取sessionStorage里的状态信息
const sessionStore = window.sessionStorage.getItem(storeKey.vuexStore);
if (sessionStore) {
  let store = {};
  Object.assign(store, this.$store.state, JSON.parse(sessionStore));
  this.$store.replaceState(store);
  window.sessionStorage.removeItem(storeKey.vuexStore);
}

// 在页面刷新时将vuex里的信息保存到sessionStorage里
// ie、谷歌、360 页面刷新执行顺序 onbeforeunload -> onunload -> onload，关闭执行顺序 onbeforeunload -> onunload
// firefox 页面刷新只执行 onunload，页面关闭只执行 onbeforeunload
let eventName = "beforeunload";
const fireFox = navigator.userAgent.indexOf("Firefox") !== -1;
if (fireFox) {
  eventName = "unload";
}
window.addEventListener(eventName, () => {
  // 根据 currentRefresh 判断是退出还是刷新
  const currentRefresh = this.$store.state.account.currentRefresh;
  if (currentRefresh) {
    window.sessionStorage.setItem(
      storeKey.vuexStore,
      JSON.stringify(this.$store.state)
    );
  }
});
```

# redux

## redux-persist

在 React 项目中，我们经常会通过**redux**以及**react-redux**来存储和管理全局数据。但是通过 redux 存储全局数据时，会有这么一个问题，如果用户刷新了网页，那么我们通过 redux 存储的全局数据就会被全部清空，比如登录信息等。

这个时候，我们就会有全局数据持久化存储的需求。首先我们想到的就是 localStorage，localStorage 是没有时间限制的数据存储，我们可以通过它来实现数据的持久化存储。

但是在我们已经使用 redux 来管理和存储全局数据的基础上，再去使用 localStorage 来读写数据，这样不仅是工作量巨大，还容易出错。那么有没有结合 redux 来达到持久数据存储功能的框架呢？当然，它就是**redux-persist**。redux-persist 会将 redux 的 store 中的数据缓存到浏览器的 localStorage 中。

> redux-persist 的使用

1、对于 reducer 和 action 的处理不变，只需修改 store 的生成代码，修改如下

```js
import { createStore } from "redux";
import reducers from "../reducers/index";
import { persistStore, persistReducer } from "redux-persist";
import storage from "redux-persist/lib/storage";
import autoMergeLevel2 from "redux-persist/lib/stateReconciler/autoMergeLevel2";

const persistConfig = {
  key: "root",
  storage: storage,
  stateReconciler: autoMergeLevel2 // 查看 'Merge Process' 部分的具体情况
};

const myPersistReducer = persistReducer(persistConfig, reducers);

const store = createStore(myPersistReducer);

export const persistor = persistStore(store);
export default store;
```

2、在 index.js 中，将 PersistGate 标签作为网页内容的父标签

```js
import React from "react";
import ReactDOM from "react-dom";
import { Provider } from "react-redux";
import store from "./redux/store/store";
import { persistor } from "./redux/store/store";
import { PersistGate } from "redux-persist/lib/integration/react";

ReactDOM.render(
  <Provider store={store}>
    <PersistGate loading={null} persistor={persistor}>
      {/*网页内容*/}
    </PersistGate>
  </Provider>,
  document.getElementById("root")
);
```

这就完成了通过 redux-persist 实现 React 持久化本地数据存储的简单应用

3、最后我们调试查看浏览器中的 localStorage 缓存数据

发现数据已经存储到了 localStorage 中，此时刷新网页，redux 中的数据也不会丢失
