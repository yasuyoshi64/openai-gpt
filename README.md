# openai-gpt

OpenAI GPTのサンプルです。
以下の部分だけ自分のAPI-Keyに書き換えれば動きます。

```js
const openai = new OpenAI({
  organization: 'org-xxxxxxxxxxxxxxxxxxxxxxxx',
  apiKey: 'sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx',
  dangerouslyAllowBrowser: true   // ブラウザ上で強制実行するためのフラグ
})
```


This template should help get you started developing with Vue 3 in Vite.

## Recommended IDE Setup

[VSCode](https://code.visualstudio.com/) + [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur) + [TypeScript Vue Plugin (Volar)](https://marketplace.visualstudio.com/items?itemName=Vue.vscode-typescript-vue-plugin).

## Customize configuration

See [Vite Configuration Reference](https://vitejs.dev/config/).

## Project Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Compile and Minify for Production

```sh
npm run build
```
