# 既存のNuxt.jsにjestを導入した際の手順

## まずjestをいれみて、通るかどうか
```
npm install jest --save-dev
```

上記コマンドを入力しjestを入れ、testフォルダの作成し、その中にjestファイル(spec.jsファイル)を作成  
今回だと簡単な下記が良いかと！
```
describe('Sum' ()=> {
    test("1+1",()=> {
        expect(1+1).toBe(2)
    })
})
```
そして、`package.jsonn`に下記を追加
```
"scripts": {
    "test":"jest"
}
```

`npm run test`を実行し、通ればOK

## 次に外部ファイルを読み込むことができるか

[参考記事](https://dev.appswingby.com/nuxt/nuxt-js%E3%83%97%E3%83%AD%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E3%81%AB%E3%81%82%E3%81%A8%E3%81%8B%E3%82%89jest%E3%82%92%E5%88%A9%E7%94%A8%E3%81%A7%E3%81%8D%E3%82%8B%E3%82%88%E3%81%86%E3%81%AB%E3%81%99/#Babel%E3%81%AB%E5%AF%BE%E5%BF%9C%E3%81%97%E3%81%9F%E7%92%B0%E5%A2%83%E3%82%92%E4%BD%9C%E3%82%8B)

ただこの記事の通りだとエラーになるので別途下記対応
- `collectConverageFrom`にpluginsディレクトリの追加
- `moduleFileExtensions`にvueを追加
- `transform`にvueファイルを追加し、jestとvueのバージョンに合わせたvue-jestの追加が必要[(参考記事)](https://github.com/vuejs/vue-jest)

### 他つまいずいたポイント
- `@vue/test-utils`がバージョンをvueのバージョンに合わせてpeerなバージョンをインストールする必要あり。(nuxtの場合は@vue-appで定義されてることがあるのでそこでvueのバージョンを確認)
- DOM生成ができないエラー`window is undefind`となる場合  
 1. jest > ver.27の場合 `js-dom`を別途インストール
 2. jest > ver.28の場合 `jest-environment-jsdom`を別途インストール
 3. `jest.config.js`の`testEnvironment`プロパティに追加。
- `import vue`を怒られたjest.configに`moduleDirectories:["node_modules",__dirname]`を入れて解決(おそらくnode_modulesの位置定義をしている？)

## 記述で躓いた部分
- nuxtのauto importを使用している場合はComponentをスタブしないといけない。
- nuxtのpluginをテストする場合
  [参考](https://blog.nightonly.com/2021/12/26/jest%E3%81%A7nuxtvuetify%E3%81%AE%E3%83%86%E3%82%B9%E3%83%88%E3%82%92%E6%9B%B8%E3%81%84%E3%81%A6%E3%81%BF%E3%82%8B/#inject)
  - PluginをVueの記述に分解して、injectではなく、prototypeにセットしておく
  - plugin用のComponentをjestファイル内で作成(Templateは空)
  - 上記のComponentをmount時に、prototypeでセットした関数をはmounted時に実行させる。