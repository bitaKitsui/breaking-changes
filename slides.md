---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false
---

# Vue2からの破壊的変更

---

# Agenda

- 新しい変更点
- 破壊的変更点と新要素
- まとめ

---

# 今日のひとこと

- 古谷実の漫画を読んでみた
  - 『ヒミズ』
  - 『ヒメアノ〜ル』
  - 『わにとかげぎす』
- 『Sonny Boy』を見てみたい

---

# 新しい変更点

- Composition API
- `<script setup>`構文
- Teleport
- Fragments
- Emits Component Option
- `createRenderer` API from @vue/runtime-core
- `v-bind` in `<style>`
- `<style scoped>`のルール変更
- Suspense(experimental)

---

# 破壊的変更と新要素

結構変更点はありそう。🤨‼️

https://v3-migration.vuejs.org/breaking-changes/

---

# v-model

フォーム入力バインディング

v-modelのpropsとeventのデフォルト名が変更

```ts
prop: value => modelValue
event: input => update:modelValue
```

Vue 3 Syntax

```ts
<ChildComponent v-model="pageTitle" />

<ChildComponent 
  :modelValue="pageTitle"
  @update:modelValue="pageTitle = $event"
/>
```

---

# v-model

フォーム入力バインディング

modelValueの名前を変更することも可能

<img src="/v-bind-instead-of-sync.png" alt="">

---

# v-model

フォーム入力バインディング

実際の使い方

```ts
<ChildComponent v-model:title="pageTitle" v-model:content="pageContent" />

<!-- would be shorthand for: -->

<ChildComponent
  :title="pageTitle"
  @update:title="pageTitle = $event"
  :content="pageContent"
  @update:content="pageContent = $event"
/>
```

---

# v-model

フォーム入力バインディング

`v-bind.sync`の削除 => v-modelの引数に与えられるように

子のイベントを検知して、親で値を書き換えられる（双方向データバインディング）

```ts
this.$emit('update:title', newValue)
``` 

```ts
<ChildComponent :title="pageTitle" @update:title="pageTitle = $event" />

or 
        
<ChildComponent :title.sync="pageTitle" />
```

---

# v-model

フォーム入力バインディング

- 複数のv-modelが使えるように
  - Vue2では1つのコンポーネントに1つだけ
- カスタムmodifierの作成が可能に(`v-model.trim`, `v-model.number`的な)
  - `modelModifiers`プロパティを介してカスタム機能を追加できる

```ts
<script setup >
const props = defineProps({
  modelValue: string,
  modelModifiers: { default: () => ({}) }
})
< /script> 
        
<ChildComponent v-model.capitalize="pageTitle" />
```

---

# key属性

templateに渡す属性

- Vue3では自動的にユニークなキーを生成するように
  - `v-if`などの文脈でキーの指定が不要になった
- 手動でキーを指定する場合はユニークな値が必須
- `<template v-for>`の場合は`template`タグにキーを指定する

```ts
<!-- Vue 2.x -->
<template v-for="item in list">
  <div :key="'heading-' + item.id">...</div>
  <span :key="'content-' + item.id">...</span>
</template>

<!-- Vue 3.x -->
<template v-for="item in list" :key="item.id">
        <div>...</div>
<span>...</span>
</template>
```

---

# v-if vs. v-for

どっちが優先する？

- Vue2では`v-for`が優先された
- Vue3では`v-if`が優先される
- 同じ要素で両方を使用することは非推奨
  - `computed`や`<script>`タグ内で制御して、計算されたプロパティを作成して回避すべき
  - template内でやることではない

---

# Functional Components

関数型コンポーネント

- 関数型コンポーネントとは
  - propsとcontextを受け取れるプレーンな関数を使ってのみ作るコンポーネント
    - `slot`, `emit`, `attrs`など
  - パフォーマンスの最適化に使われた
    - コンポーネントの初期化が速かったため

```ts
<template functional>
  <component
    :is="`h${props.level}`"
    v-bind="attrs"
    v-on="listeners"
  />
</template>

<script>
export default {
  props: ['level']
}
</script>
```

---

# Functional Components

関数型コンポーネント

- Vue3ではステートフルなコンポーネントのパフォーマンスが劇的に向上
- 関数型コンポーネントを使う必要はなくなった
- SFCの`template`の`functional`属性が削除

---

# Async Components(new)

非同期コンポーネント用のヘルパーメソッド

- 明示的に定義する`defineAsyncComponent`が登場
- `component`オプションが`loader`に名称変更

---

# Async Components(new)

非同期コンポーネント用のヘルパーメソッド

Vue2時代（特定のコンポーネントだけロードしたい！など）

```ts
const asyncModal = () => import('./Modal.vue')

const asyncModal = {
  component: () => import('./Modal.vue'),
  delay: 200,
  timeout: 3000,
  error: ErrorComponent,
  loading: LoadingComponent
}
```

---

# Async Components(new)

非同期コンポーネント用のヘルパーメソッド

Vue3 syntax

```ts
import { defineAsyncComponent } from 'vue'
import ErrorComponent from './components/ErrorComponent.vue'
import LoadingComponent from './components/LoadingComponent.vue'

// Async component without options
const asyncModal = defineAsyncComponent(() => import('./Modal.vue'))

// Async component with options
const asyncModalWithOptions = defineAsyncComponent({
  loader: () => import('./Modal.vue'),
  delay: 200,
  timeout: 3000,
  errorComponent: ErrorComponent,
  loadingComponent: LoadingComponent
})
```

---

# Async Components(new)

非同期コンポーネント用のヘルパーメソッド

ちなみに。。。

Vue Routerにも`Lazy Loading Routes`という機能があるが、<br />
Vueの`Async Components`とは似ていても別物<br />
併用は不可とのこと。

https://router.vuejs.org/guide/advanced/lazy-loading.html

---

# `$listeners` removed

イベントリスナー

- Vue3では`$attrs`に統合された
- Vue2ではコンポーネントに渡される属性に`this.$attrs`でアクセスでき、<br/>イベントリスナーに`this.$listeners`でアクセスできた
- `inheritAttrs: false`と組み合わせると、これらの属性やリスナーをルート要素ではなく他の要素に適用できる

```ts
<template>
  <label>
    <input type="text" v-bind="$attrs" v-on="$listeners" />
  </label>
</template>
<script>
  export default {
    inheritAttrs: false
  }
</script>
```

---

# `$listeners` removed

イベントリスナー

Vue3の仮想DOMではイベントリスナーは単なる属性になり、接頭辞に`on`がつき、<br/>
`$attrs`オブジェクトの一部となり、`$listeners`が削除された

```ts
<template>
  <label>
    <input type="text" v-bind="$attrs" />
  </label>
</template>
<script>
export default {
  inheritAttrs: false
}
</script>
```

`$attrs`の中身（`id`属性と`v-on:click`を受け取った場合）

```json
{
  id: 'my-input',
  onClose: () => console.log('close Event triggered')
}
```

---

# `$attrs` includes `class` & `style`

- クラスやスタイルなどコンポーネントに渡される全ての属性が`$attrs`に含まれるように
- Vue2ではクラスとスタイル属性は、仮想DOMの実装で特別な扱いを受けていたので、<br/>`$attrs`に含まれてはいなかった
- `inheritAttrs: false`を指定したときに問題が生じる
  - `attrs`に含まれる属性は、ルート要素に自動的に追加されなくなり、<br/>どこに追加するかは開発者の判断に委ねられる
  - しかし、`$attrs`の一部でないクラスとスタイルは依然としてコンポーネントのルート要素に適用されたままになる
- Vue3の`$attrs`にはすべての属性が含まれているので、すべての属性を別の要素に適用することが簡単になる

---

# Events API

- `$on`, `$off`, `$once`のインスタンスメソッドが削除された
- Vue2ではこれらのイベントエミッターAPIを介して、強制的にアタッチされたハンドラーをトリガーできた
  - アプリケーション全体で使用されるグローバルなイベントリスナーを作成するための、<br/>イベントバス作成ようとして使われていた
- Vue3では削除され、emitは親コンポーネントによって宣言的にアタッチされた<br/>イベントハンドラーをトリガーするのみとなった

https://v3-migration.vuejs.org/breaking-changes/events-api.html#events-api

---

# Global API Treeshaking

- ツリーシェイキングとは
  - webpackやrollupのようなモジュールバンドラーでサポートされている機能
  - 「デッドコードの除去」をして、クライアントサイドのコードをなるべく軽くする
  - `nextTick()`などはグローバルAPIとして公開されていて、便宜上現在のインスタンスに<br/>自動的にバインドされている
  - これらのグローバルなAPIは使う必要がないものまでバンドルに含まれていた

---

# Global API Treeshaking

- Vue3ではこれらの問題を考慮して、グローバルAPIと内部APIが再構築された
- その結果、グローバルAPIはESモジュールのビルドでは、名前付きエクスポートとして<br/>のみアクセスできるようになった
  - Vue.nextTick
  - Vue.observable など
- ちなみに、
  - 内部コンポーネント/ヘルパーの多くも名前付きエクスポートされるようになった
  - コンパイラは機能が使用される時だけインポートするコードを出力できる

---

# まとめ

<div style="display: table; width: 100%; height: 20rem; text-align: center">
  <p style="display: table-cell; vertical-align: middle">人生色々、Vueの破壊的変更も色々</p
></div>