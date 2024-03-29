# JavaScriptの基礎知識
主にES6で登場した文法の紹介
- const,let
- テンプレート文字列
- アロー関数とスコープ
```
var tahoe = {
    mountains: ["Freel", "Rose", "Tallac", "Rubicon", "Silver"],
    print: function(delay=1000) {

    setTimeout(function() {
        console.log(this.mountains.join(","))
    }, delay);

    }
};
```
この場合のthisはtahoeオブジェクトではなく、windowオブジェクトになる。
この解決としてアロー関数でコールバックを記述する。

```
var tahoe = {
    mountains: ["Freel", "Rose", "Tallac", "Rubicon", "Silver"],
    print: function(delay=1000) {

    setTimeout(
        () => console.log(this.mountains.join(",")),
        delay
    );
    }
};
```
**アロー関数は独自のスコープを持たない**
- デストラクチャリング（分割代入）
- スプレット構文
- Promise,async/await
- クラス宣言

## Javascriptにおける関数型プログラミング
関数型プログラミングとは
```
const numbers = [1, 2, 3, 4, 5];

const doubledNumbers = numbers.map(function (num) {
  return num * 2;
});
```
↑ここでいうmap関数のようなもの
for文のような役割をmap関数に切り出している。
コードが「何をするか」を記述せず、具体的な手順や手続きを指定しない、いわゆる宣言的なプログラミング手法。

特徴として
- 不変性（元のnumbersは変更されない）
- 純粋性（関数が同じ引数に対していつも同じ結果を返す）
- 再帰（ループの代わりに関数内で自分自身の関数を呼びだすことができる）
がある。

# React
React = DOM APIの呼び出しを一手に引き受けるライブラリ

Reactは以下二つのライブラリが最低必要になる。
- React(React要素を作成)
- ReactDOM(React要素をDOMに変換)

## React要素

```
React.createElement("h1", null, "Baked Salmon");
```
は
```
<h1>Baked Salmon</h1>
```
に置換される。

React要素をコンソールに出力すると以下のようになる。
```
Object
    typeof:Symbol(react.element)
    key: null
    props: 
        children: "Baked almon"
        [[Prototype]]: Object
    ref: null
    type: "h1"
    （省略）
    [[Prototype]]: Object
```
propsなどは全てオブジェクトの要素として存在している。

**描画と子要素**
```
const dish = React.createElement("h1", null, "Baked Salmon");
const dessert = React.createElement("h2", null, "Coconut Cream Pie");

ReactDOM.render(
    [dish, dessert],
    document.getElementById('react-container')
);
```
↓
```
<div id="react-container">
    <h1>Baked Salmon</h1>
    <h2>Cream Pie</h2>
</div>
```
↑このようなDOMが作られる。

**関数コンポーネント**
```
function IngredientsList({items}) {
return React.createElement(
    "ul",
    { className: "ingredients" },
    items.map((ingredient, i) =>
    React.createElement("li", { key: i }, ingredient) )
);
}

const items = [
    "1 cup unsalted butter",
    "1 cup crunchy peanut butter",
    "1 cup brown sugar",
    "1 cup white sugar",
    "2 eggs",
    "2.5 cups all purpose flour",
    "1 teaspoon baking powder",
    "0.5 teaspoon salt"
];

ReactDOM.render(
React.createElement(IngredientsList, {items}, null),
document.getElementById("react-container")
);
```
上記のコードはitems配列をli要素に変換している。

本書ではクラスコンポーネントも紹介ている。
クラスコンポーネントは将来的に廃止される予定である。

## jsx
JavascriptにXMLのようなタグベース構文を記述するための言語拡張。
ネストの深いDOMを作る必要がある際、より宣言的な記述ができるようになる。

jsx→js(es)への変換はBabelで行うことができる。

## webpackによるビルド環境構築
webpack = モジュールバンドラ : 異なる種類のファイル（js,jsx.css...）を単一のファイルに結合するためのツール

create appを使わないやり方

```
<!-- package.jsonを作成 -->
npm init -y
<!-- 必要なライブラリをインストール -->
npm install react react-dom serve

<!-- コンポーネント等を追加 -->

<!-- webpackをインストール -->
npm install --save-dev webpack webpack-cli

<!-- jsxコンパイルのためBabelをインストール -->
npm install babel - loader `babel/core --save-dev

<!-- Babelのプリセット（プラグイン）をインストール -->
npm install @babel/present-env `babel/present-react --save-dev

<!-- webpackの実行 -->
npx webpack --mode development

```

```:webpack.config.js
var path = require("path");

module.exports = {
  entry: "./src/index.js",
  output: {
    path: path.join(__dirname, "dist", "assets"),
    filename: "bundle.js",
  },
  devtool: "#source-map",
  module: {
    rules: [
      {
        <!-- 拡張子が.jsのファイルが全て対応になる -->
        test: /\.js$/,
        <!-- 除外するファイル -->
        exclude: /(node_modules)/,
        loader: "babel-loader",
      },
      {
        test: /\.css$/i,
        use: ["style-loader", "css-loader"],
      },
    ],
  },
};

```
webpack.configにビルドの設定を記述。

```:babelrc
<!-- プリセットを指定 -->
{
    "presets": ["@babel/preset-env", "@babel/preset-react"]
}
```
Babelの設定ファイル。

```:package.json
"scripts": {
    "build": "webpack --mode production"
  },
```
package.jsonに記述をしておくと楽。

## フック
フック＝コンポーネントとは独立した関数で、コンポーネントに着脱可能な機能を取り付ける。

### useRef
DOMノードへの参照を保持する。
```
import React, { useRef } from "react";

export default function AddColorForm({ onNewColor = f => f }) {
  const txtTitle = useRef();
  const hexColor = useRef();

  const submit = e => {
    e.preventDefault();
    const title = txtTitle.current.value;
    const color = hexColor.current.value;
    onNewColor(title, color);
    txtTitle.current.value = "";
    hexColor.current.value = "";
  };

<!-- refプロパティにrefオブジェクトを指定 -->
  return (
    <form onSubmit={submit}>
      <input ref={txtTitle} type="text" placeholder="color title..." required />
      <input ref={hexColor} type="color" required />
      <button>ADD</button>
    </form>
  );
}
```
ref＝{}にrefオブジェクトを指定することで、refのcurrentフィールド経由でDOMに直接アクセスできるようになる。
しかし、このコードのようにDOMに直接アクセスして書き換えるようなコンポーネントは「制御されていないコンポーネント」と呼ばれ、推奨されない。

```
import React, { useState } from "react";

export default function AddColorForm({ onNewColor = f => f }) {
  const [title, setTitle] = useState("");
  const [color, setColor] = useState("#000000");

  const submit = e => {
    e.preventDefault();
    onNewColor(title, color);
    setTitle("");
    setColor("");
  };

  return (
    <form onSubmit={submit}>
      <input
        value={title}
        onChange={event => setTitle(event.target.value)}
        type="text"
        placeholder="color title..."
        required
      />
      <input
        value={color}
        onChange={event => setColor(event.target.value)}
        type="color"
        required
      />
      <button>ADD</button>
    </form>
  );
}
```
上記はrefの代わりにstateでvalueを管理している。
inputに値を入力するたびにstateが更新される仕組み＝制御されたコンポーネント。

## カスタムフック
複数のコンポーネントで重複した関数を切り出したもの。
```
import { useState } from "react";

export const useInput = initialValue => {
  const [value, setValue] = useState(initialValue);
  return [
    { value, onChange: e => setValue(e.target.value) },
    () => setValue(initialValue)
  ];
};
```
↑stateの値とstateを更新する関数のみを切り出したもの。
stateの初期値を受け取り、配列を戻り値として返却する。

```
import React from "react";
import { useInput } from "./hooks";

export default function AddColorForm({ onNewColor = f => f }) {
  const [titleProps, resetTitle] = useInput("");
  const [colorProps, resetColor] = useInput("#000000");

  const submit = e => {
    e.preventDefault();
    onNewColor(titleProps.value, colorProps.value);
    resetTitle();
    resetColor();
  };

  return (
    <form onSubmit={submit}>
      <input
        {...titleProps}
        type="text"
        placeholder="color title..."
        required
      />
      <input {...colorProps} type="color" required />
      <button>ADD</button>
    </form>
  );
}

```
↑useStateを呼び出す代わりにuseInputを呼び出している。
{...titleProps}の箇所で、スプレッド構文によりvalueとonChange要素がinputに設定される。
onSubmitのresetTitle()により、フォーム送信後に初期化を行うこともできる。

カスタムフック内部で別のフックを呼び出した場合、呼び出し元のコンポーネントが際描画される。

### Reactコンテキスト
ステートをルートコンポーネントで一括管理するのが理想だが、コンポーネントの階層が深くなるほど上位から下位へ多くのコンポーネントを経由してデータを受け渡しする必要ができてしまう。
それを解決するのが**コンテキスト**。

```
import React, { createContext } from "react";
import colors from "./color-data";
import { render } from "react-dom";
import App from "./App";

export const ColorContext = createContext();

render(
  <ColorContext.Provider value={{ colors }}>
    <App />
  </ColorContext.Provider>,
  document.getElementById("root")
);
```
createContextを用いてコンテキストを作成する。
Context.Provider=データの提供元
Context.Consumer＝データの受け渡し先

Context.Providerで囲ったコンポーネント以下の全てのコンポーネントに直接value={{ colors }}の受け渡しを行うことができる。

```
import React, { useContext } from "react";
import Color from "./Color";
import { ColorContext } from "./";

export default function ColorList() {
  const { colors } = useContext(ColorContext);

  if (!colors.length) return <div>No Colors Listed. (Add a Color)</div>;

  return (
    <div>
      {colors.map(color => (
        <Color key={color.id} {...color} />
      ))}
    </div>
  );
}
```
const { colors } = useContext(ColorContext);
でcolorsを直接取得している。

### コンテキストとstateの併用
```
import React, { createContext, useState } from "react";
import colorData from "./color-data.json";
import { v4 } from "uuid";

export const ColorContext = createContext();

export default function ColorProvider({ children }) {
  const [colors, setColors] = useState(colorData);

  const addColor = (title, color) =>
    setColors([
      ...colors,
      {
        id: v4(),
        rating: 0,
        title,
        color
      }
    ]);

  const rateColor = (id, rating) =>
    setColors(
      colors.map(color => (color.id === id ? { ...color, rating } : color))
    );

  const removeColor = id => setColors(colors.filter(color => color.id !== id));

  return (
    <ColorContext.Provider value={{ colors, addColor, removeColor, rateColor }}>
      {children}
    </ColorContext.Provider>
  );
}
```
value={{ colors, addColor, removeColor, rateColor }}>
により、配下のコンポーネントでstateの値とその更新を行うことができる。
setColorsそのものを共有するのではなく、必要な処理（addColorなど）を提供することで、不要なバグを防ぐことができる。

### useEffect
```
function WordCount({ children = "" }) {
  useAnyKeyToRender();

  // 毎回異なるインスタンスが生成される
  const words = children.split(" ");

  useEffect(() => {
    console.log("fresh render");
  }, [words]);

  return (
    <>
      <p>{children}</p>
      <p>
        <strong>{words.length} - words</strong>
      </p>
    </>
  );
}
```
wordsはレンダリングのたびに再生成されるので、useEffectもレンダリングのたびに実行されてしまう。
これを回避するには
①関数の外で依存配列を宣言する（性的な文字列）
②useMemoを宣言する（関数など）

### useMemo
```
function WordCount({ children = "" }) {
  useAnyKeyToRender();

  // useMemoによるメモ化
  const words = useMemo(() => children.split(" "), [children]);

  useEffect(() => {
    console.log("fresh render");
  }, [words]);

  return (
    <>
      <p>{children}</p>
      <p>
        <strong>{words.length} - words</strong>
      </p>
    </>
  );
}
```
 const words = useMemo(() => children.split(" "), [children]);

 依存配列**children**に変更がない限り.splitは実行されず、wordsにはキャッシュされた値が代入される。
 wordsは同一であることが保証される。

 ### useCallback
 useMemoは関数の返り値をキャッシュするが、useCallbackは関数自体をキャッシュする。
 ```
 // useCallbackによるメモ化
  const fn = useCallback(() => {
    console.log("hello");
    console.log("world");
  }, []);

  useEffect(() => {
    console.log("fresh render");
    fn();
  }, [fn]);
 ```

 ### useLayoutEffect
 1. コンポーネントの描画関数が呼び出される
 1. useLayoutEffectが実行される
 1. ブラウザのPaint処理で描画結果が画面に反映される
 1. useEffectが実行される

 useLayoutEffect＝関数実行〜描画の間に実行できる

 ```
 function useWindowSize() {
  const [width, setWidth] = useState(0);
  const [height, setHeight] = useState(0);

  const resize = () => {
    setWidth(window.innerWidth);
    setHeight(window.innerHeight);
  };

  useLayoutEffect(() => {
    window.addEventListener("resize", resize);
    resize();
    return () => window.removeEventListener("resize", resize);
  }, []);

  return [width, height];
}
 ```
 上記はブラウザの大きさを測定する関数。
 useLayoutEffectを利用することで描画前にブラウザの大きさを返すことができる。

 ### useReducer
 ```
function Checkbox() {
  const [checked, setChecked] = useState(false);

  function toggle() {
    setChecked(checked => !checked);
  }

  return (
    <>
      <input type="checkbox" value={checked} onChange={toggle} />
      {checked ? "checked" : "not checked"}
    </>
  );
}
 ```
はuseReducerを使うと
```
function Checkbox() {
  const [checked, toggle] = useReducer(checked => !checked, false);

  return (
    <>
      <input type="checkbox" value={checked} onChange={toggle} />
      {checked ? "checked" : "not checked"}
    </>
  );
}
```
とできる。
useReducerの返り値は1つ目がステート値で二つ目がレデューサ関数。

## サスペンス

### エラーバウンダリ
エラーバウンダリは、あるコンポーネントで例外が発生してもアプリケーション全体に影響が及 ばないようにするための機構。
エラーバウンダリは特別な React コンポーネントで、子コン ポーネントで発生した例外を一手に引き受ける。

残念なことに、このエラーバウンダリを使用するにはクラスコンポーネントを実装する必要がある。
```
import React, { Component } from "react";

export default class ErrorBoundary extends Component {
  state = { error: null };

  static getDerivedStateFromError(error) {
    return { error };
  }

  render() {
    const { error } = this.state;
    const { children, fallback } = this.props;

    if (error && !fallback) return <ErrorScreen error={error} />;
    if (error) return fallback({error});
    return children;
  }
}

export const BreakThings = () => {
  throw new Error("We intentionally broke something");
}

function ErrorScreen({ error }) {
  return (
    <div className="error">
      <h3>We are sorry... something went wrong</h3>
      <p>We cannot process your request at this moment.</p>
      <p>ERROR: {error.message}</p>
    </div>
  );
}
```
getDerivedStateFromError という特別なスタティックメソッドを実装。
配下のコンポーネントchildrenで例外が発生した場合、このメソッドに通知される。

```
export const BreakThings = () => {
throw new Error("We intentionally broke something");
}
```

```
return (
    <SiteLayout
      menu={
        <ErrorBoundary>
          <p>Menu</p>
        </ErrorBoundary>
      }
    >
      <ErrorBoundary>
        <Callout>Callout</Callout>
      </ErrorBoundary>
      <ErrorBoundary>
        <h1>Contents</h1>
        <p>this is the main part of the example layout</p>
        <BreakThings />
      </ErrorBoundary>
    </SiteLayout>
  );
```
上記の例では、アプリケーションは 3 つのエラーバウンダリに分割されている。
この場合、メ ニューのカラムと Callout コンポーネントの中に BreakThings が配置されているので、Error Screen コンポーネントが 2 つ描画される。

### コードスプリッティング
コードスプリッティングはコードベースを複数のかたまりに分割することでダウンロード時間 を短縮するテクニック。
```
export default function Agreement({ onAgree = f => f }) {
  return (
    <div>
      <p>Terms...</p>
      <p>These are the terms and stuff. Do you agree?</p>
      <button onClick={onAgree}>I agree</button>
    </div>
  );
}
```

```
import React, { useState, Suspense, lazy } from "react";
import Agreement from "./Agreement";
import ClimbingBoxLoader from "react-spinners/ClimbingBoxLoader";

const Main = lazy(() => import("./Main"));

export default function App() {
  const [agree, setAgree] = useState(false);

  if (!agree) return <Agreement onAgree={() => setAgree(true)} />;

  return (
    <Suspense fallback={<ClimbingBoxLoader />}>
      <Main />
    </Suspense>
  );
}
```
初期状態では Agreement コンポーネントのみが描画され、ユーザーが利用許諾 に同意すれば、agree が更新され、Main コンポーネントと配下のコンポーネントが描画される。
React.lazy を使ってインポートすることで、Main コンポーネントが初回描画される まで、Main のダウンロードを遅らせることが可能。
React.lazy を使っているため、import 関数は即座に実行されず、Main コンポーネントの初 回描画時に実行される。それにより、Main コンポーネントは非同期に読み込まれる。

### Suspense コンポーネント
コンポーネントの非同期読み込みにはSuspense コンポーネントを用いる。
```
import React, { useState, Suspense, lazy } from "react";
import Agreement from "./Agreement";
import ClimbingBoxLoader from "react-spinners/ClimbingBoxLoader";
const Main = lazy(() => import("./Main")); 

export default function App() {
const [agree, setAgree] = useState(false); 
if (!agree)return <Agreement onAgree={() => setAgree(true)} />;
return (
<Suspense fallback={<ClimbingBoxLoader />}>
  <Main /> 
</Suspense>
); }
```
Suspense の fallback プロパティに ClimbingBoxLoader が指定されているため、Main のロードが完了するまでは ClimbingBoxLoader コンポーネントが描画される。
Main コンポーネントが無事に ロードされた時点で、Suspense は ClimbingBoxLoader をアンマウントして、代わりに Main コンポーネントを描画する。

※react-spinnersはアニメーションライブラリ。今回はClimbingBoxLoaderというコンポーネントを使用している。

** サスペンスの応用 **
```
import React, {Suspense} from "react";
import ErrorBoundary from "./ErrorBoundary";
import GridLoader from "react-spinners/GridLoader";

const loadStatus = () => {
    throw new Promise(resolves => null);
};

function Status() {
  const status = loadStatus();
  return <h1>status: {status}</h1>;
}

export default function App() {
  return (
    <Suspense fallback={<GridLoader />}>
      <ErrorBoundary>
        <Status />
      </ErrorBoundary>
    </Suspense>
  );
}
```
StatusコンポーネントをErrorBoundary内に記述することで、成功時にはStatusコンポーネント、失敗時にはエラー表示というハンドリングを行うことができる。
また、Promiseを throwした際はSuspenseによりGridLoaderのアニメーションが表示される。

# テスト

## ESLint
Create React App を使ってプロジェクトを作成した場合、ESLint はデフォルトでインストー ルされている。
React のアプリケーションで ESLint を使うには eslint-plugin-reactをインストールする。
```
npm install eslint --save-dev
```

ESLint を使う前に構文チェックのルールを記述する必要がある。
eslint --init を実行していくつかの質問に答えることで設定ファイルが生成される。

```
$ npx eslint --init
```
上記を実行すると以下の3つが実行される。

1. 必要なプラグイン(eslint-plugin-reactなど)がローカルの./node_modules フォルダ配下にインストール
2. 依存パッケージが package.json に追加
3. 設定ファイル(.eslintrc.jsonなど)がプロジェクトのルートフォルダに生成

```:.eslintrc.json
{
"env": {
"browser": true,
"es2021": true },
"extends": [ "eslint:recommended", "plugin:react/recommended"
], "parserOptions": {
"ecmaFeatures": { "jsx": true
},
"ecmaVersion": 12, "sourceType": "module"
}, "plugins": [
"react"
], "rules": { }
}
```
ここでは、extendsはeslintとreactのデフォルトルールを指定している。

```:sample.js
const gnar = "gnarly";
const info = ({
file = __filename, dir = __dirname
}) => ( <p>
{dir}: {file} </p>
);
switch (gnar) { default:
        console.log("gnarly");
break; }
```
上記のsample.jsというファイルに対してlintを実行する場合は、
```
$ npx eslint sample.js

3:7 error 'info' is assigned a value but never used no-unused-vars
4:10 error '__filename' is not defined no-undef
5:9 error '__dirname' is not defined no-undef
7:3 error 'React' must be in scope when using JSX react/react-in-jsx-scope
× 4 problems (4 errors, 0 warnings)
```
で実行でき、変数infoが使用されていないことなどが指摘されている。

また、ファイル名を指定せずに実行するとカレントディレ クトリ配下のファイルをすべてチェックする。
エスケープしたいファイルがある場合は

```:.eslintignore
dist/assets/
sample.js
```
.eslintignore に記述する。

```:package.json
{
"scripts": {
"lint": "eslint ." }
}
```
package.json ファイルの scriptsに追記しておくことでnpm run lintで実行できるようになる。

### ESLint プラグイン
ESLint には上記以外にも膨大な数のプラグインが公開されており、必要なプラグインをインス トールすることで独自の構文チェックを実行できる。
eslint-plugin-react-hooksはReact フックのコードを チェックできる。

```
$ npm install eslint-plugin-react-hooks --save-dev
```

```:.eslintrc.json
{
"plugins": [
         "react-hooks"
], "rules": {
"react-hooks/rules-of-hooks": "error",
"react-hooks/exhaustive-deps": "warn" }
}
```
npmで追加することで、rulesに追加される。

そのほか、アクセスビリティに関するルールを規定する **eslint-plugin-jsx-a11y**などもある。

Awesome ESLint(https://github.com/dustinspecker/awesome-eslint)というリポジトリに は、そのような便利なプラグインが紹介されている。

## Prettier
Prettier は JavaScript で非常によく使われているコード整形ツール。
ESLint はコードの正しさを保つのに対し、 Prettier はコードの読みやすさを保つためのツール。
Prettier をインストール
```
$ npm install prettier --save-dev
```
.prettierrc という名前のファイルを作成して、以下のルールを記述する。
```:.prettierrc
{
  "semi": true, 
  "trailingComma": none,
  "singleQuote": false,
  "printWidth": 80
}
```
記述できるオプションの詳細については Prettier のドキュメントを確認できる(https://prettier.io/docs/en/options.html)。

ESLint と Prettier を統合するのに必要なパッケージをインストール
```
$ npm install eslint-config-prettier eslint-plugin-prettier --save-dev
```
eslint-config-prettier は、ESLint のルールのうち、Prettier と相容れないものを無効化する共有設定。
eslint-plugin-prettier は、Prettier のルールを ESLint のルー ルに統合するためのプラグイン。
これによりESLint から Prettier を実行することが可能になる。
.eslintrc.json に以下の設定を追加
```:.eslintrc.json
{
"extends": [
        "plugin:prettier/recommended"
], "plugins": [
"prettier"
], "rules": {
"prettier/prettier": "error" }
}
```
これによりnpm run lintを実行することでPritterも同時に実行される。

## 型チェック
### TypeScript
reactをTypeScriptで記述する場合は
```
$ npx create-react-app my-type --template typescript
```
で作成を行う。
プロジェクトのルートには TypeScript の設定値が記述されたファイルtsconfig.json が作成される。
また、package.json を見てみると、TypeScriptに関わる依存パッケージがインストールされ ているのが分かる。

TypeScriptの型定義を提供するパッケージは @types/ のプレフィック スを持つ。つまり、 @types/ で始まるパッケージは TypeScript で型宣言されている。

## テスト駆動開発(TDD)
テスト駆動開発(Test-Driven Development:TDD)は実装よりもテストを先行させる開発手法。
1. まず初めにテストを書く
1. テストを実行して失敗(Red)することを確認する
1. アプリケーションのコードを書いてテストが成功(Green)することを確認する
1. リファクタリングによりアプリケーションのコードおよびテストを改善する(Gold)

## Jest
JavaScript のテストフレームワークであれ、ばどれを使っても Reactのテストが書けるが、オフィシャルなReactのドキュメントはJestを使うことを推奨している。

JestはJavaScript のテストランナーで、JSDOM経由でDOMをシミュレートする。

### テスト環境の構築
Create React App を使って作成されたプロジェクトにはデフォルトで Jest が含まれている。

src フォルダの中にfunctions.jsとfunctions.test.js を作成。

受け取った数値を2倍にして返す関数を作成する。
まずは、関数が意図どおり動くかテストするためのコードを書くためにJest が提供するグローバル関数 test を使う。

```:functions.test.js
test("Multiplies by two", () => {
       expect();
});
```
引数の先頭はテスト名＝"Multiplies by two"
2 番目の引数はテストのコードを含む関数。
3 番目の引数はテストが完了しなかった場合のタイムアウトを指定できる。（省略された場合デフォルトの2秒）

次にテスト対象となる関数のスタブを実装する。

```:functions.js
export function timesTwo() {/* ... */}
```

このようにスタブをエクスポートすることで、テスト側で SUT を timesTwo という名前でインポートして使うことができる。

```:functions.test.js
import { timesTwo } from "./functions";

  test("Multiplies by two", () => {
    expect(timesTwo(4)).toBe(8);
});
```
上記はtimesTwo 関数に4を渡して呼び出すことで8が返却されることを期待している。

Jest の expect 関数は値を受け取ると、その値が正しいかテストするためのマッチャー(matcher)を含んだオブジェクトを返却する。
ここではtoBe というマッチャーを使用している。
オブジェクトや配列をテストするには toEqual を使う。

## React コンポーネントのテスト
```:Star.js
import { FaStar } from "react-icons/fa";

export default function Star({ selected = false }) {
  return (
    <>
      <h1>Great Star</h1>
      <FaStar id="star" color={selected ? "red" : "grey"} />
    </>
  );
}
```
```:Star.test.js
import React from "react";
import ReactDOM from "react-dom";
import Star from "./Star";
import { toHaveAttribute } from "@testing-library/jest-dom";
import { render } from "@testing-library/react";
import "@testing-library/jest-dom/extend-expect";

expect.extend({ toHaveAttribute });

test("renders an h1", () => {
  const { getByText } = render(<Star />);
  const h1 = getByText(/Great Star/);
  expect(h1).toHaveTextContent("Great Star");
});

```

### React Testing Library
Create React App を使うと@testing-library というスコープからいくつかのパッケージがインストールされる。
中でも React Testing Library( @testing-library/react)は Reactにおけるテストのベストプラクティスを集めたプロジェクトである。

React Testing Library を使うべき理由のひとつとして、エラーメッセージの改善が挙げられる。

下記はJestでのテスト
```
expect(
  div.querySelector("notrealthing")
).toBeTruthy();
```
上記は存在しないセレクタを指定した場合、querySelector は null を返す。
これに対して toBeTruthy は以下のようなエラーを出力する。
```
expect(received).toBeTruthy()
Received: null
```

@testing-library/jest-dom から toHaveAttribute マッチャーをインポート。
これを Jest のアサーションに組み込むには、以下のように expect 関数の extend メソッドを使う。

```
expect.extend({ toHaveAttribute });
```

これを使って先ほどのJestのテストを書き直すと以下のようになる。

```
test("renders a star", () => {
  const div = document.createElement("div"); ReactDOM.render(<Star />, div);
  expect(
    div.querySelector("svg")
  ).toHaveAttribute("id", "hotdog");
});
```
上記のテストを実行すると、以下のエラーが出力される。

```
expect(element).toHaveAttribute("id", "hotdog")
// element.getAttribute("id") === "hotdog"
Expected the element to have attribute:
  id="hotdog"
Received:
  id="star"
```
上記のようにより詳細なエラー内容が出る。

カスタムマッチャーをすべて記述するのが面倒な場合は @testing-library/jest-dom/extend-expect をインポートすることで、Testing Library の提供するすべてのカスタムマッチャーが Jest から使用可能になる。

```
import "@testing-library/jest-dom/extend-expect";
```

Create React Appを使った場合、extend-expectは既にインポートされている。
src フォルダに setupTests.jsに記述がある。
```
// jest-dom adds custom jest matchers for asserting on DOM nodes.
// allows you to do things like:
// expect(element).toHaveTextContent(/react/i)
// learn more: https://github.com/testing-library/jest-dom
import "@testing-library/jest-dom/extend-expect";
```

### コードカバレッジ
**コードカバレッジ**は全体のコードのうちどれだけの行数がテストされたかを計測してパーセン テージで表すこと。
Jest もコードカバレッジのレポート機能を持っており、Istanbul を使ってこれを実現できる。
```
$ npm test -- --coverage
```
実行後、コンソールに出力され、テスト完了後にプロジェクトのルートに coverage といフォルダが生成され、その中にレポート用のHTMLがある。

# ルーティング
ルーティングとは、SPA においてユーザーのリクエストのエンドポイントを定義すること。これらのエンドポイントは、実際のファイルとは対応せず、ブラウザの提供する location や history オブジェクトと連動している。

## React Router
まず初めに、react-router と react-router-dom をインストールする。

index.jsでは ReactDOM.render で App コンポーネントを描画する代わりに、Router コンポーネントの中に App コンポーネントを描画する。

```:./src/index.js
import React from "react";
import { render } from "react-dom"; import App from "./App";
import { BrowserRouter as Router } from "react-router-dom";

render( 
  <Router>
    <App />
  </Router>,
   document.getElementById("root")
);
```
ここで Router コンポーネントの役 割は、現在ユーザーが閲覧しているページの位置を自身の子コンポーネントに伝えること。
Router コンポーネントはアプリケーション内で 1 箇所だけ、なるべくコンポーネントツリーのルート(root)に近い場所に配置する。

ルーティングの定義はApp.js 内に記述する。
```:./src/App.js
import React from "react";
import { Routes, Route } from "react-router-dom"; import {
  Home,
  About,
  Events,
  Products,
  Contact
} from "./pages";

function App() { return (
<div> 
  <Routes>
    <Route path="/" element={<Home />} /> 
    <Route
      path="/about"
      element={<About />} />
    <Route
      path="/events" element={<Events />}
    />
    <Route
      path="/products"
      element={<Products />}
     />
    <Route
      path="/contact" element={<Contact />}
    /> 
  </Routes>
</div> );
}
```

これらのルーティングの定義により、上位の Router コンポーネントは、window.location の値が変わった場合にどのコンポーネントを描画すればよいか判断する。
ブラウザの window.location が path の値と一致すれば element に指定されたコンポーネントが描画される。

 Link コンポーネントによりページ遷移が容易になる。

 ```
<nav>
  <Link to="about">About</Link>
  <Link to="events">Events</Link>
  <Link to="products">Products</Link>
  <Link to="contact">Contact Us</Link>
</nav>
 ```

### React Router プロパティ
React Router は配下のコンポーネントに対していくつかのプロパティを渡すことができる。たとえば 配下のコンポーネントは、現在の location の値をプロパティ経由で取得することが可能である。

ここでは 404 のエラー(ページが見つかりませんでし た)を表示するコンポーネント Whoops404 を実装する。

```:./src/pages.js
export function Whoops404() {
  return (
    <div>
      <h1>Resource not found</h1>
    </div>
  );
}
```

コンポーネントが実装できたら、App.js にルートを追加する。
*を指定することで存在しないルートを指定した場合、Whoops404 コンポーネントが描画される。

```./src/App.js
・
・
・
<Route path="*" element={<Whoops404 />} />
```

しかし Whoops404 コンポーネントはページが存在しないというメッセージを表示するだけで、どのページが存在しなかったのか 説明していない。
そこで、React Router の提供する useLocation フックを用いて、指定されたページのパスを表示するコンポーネントを実装する。

```:./src/pages.js
import { useLocation } from "react-router-dom";

export function Whoops404() { 
  let location = useLocation(); 
  console.log(location);
  return (
    <div> 
      <h1>
        Resource not found at {location.pathname} 
      </h1>
    </div> 
  );
}
```

ここでは useLocation で現在のページの情報を取得してコンソールに出力している。
また、プロパティからパスの値 location.pathname を取得して描画している。

### ネストしたルート
```./src/App.js
function App() { 
  return (
  <div> 
    <Routes>
      <Route path="/" element={<Home />} /> 
      <Route path="about" element={<About />}>
        <Route
          path="services" element={<Services />}
        />
        <Route
          path="history" element={<History />}
        /> 
        <Route
          path="location"
          element={<Location />}
        />
      </Route> 
    </Routes>
   </div> );
}
```
上記のように記述することでネストが深いルーティングも記述できる。

### リダイレクト
```./src/App.js
function App() { 
  return (
  <div> 
    <Routes>
      <Route path="/" element={<Home />} /> 
      <Redirect
        from="services"
        to="about/services"
      />
    </Routes>
   </div> );
}
```
このように/servicesへのアクセスをabout/servicesへリダイレクトできる。

# サーバーサイド React
## アイソモーフィックとユニバーサル
アイソモーフィックなアプローチとは、初回の描画をサーバーサイドで行うことで、シングルページアプリケーション(SPA)の苦手とする、初期表示のスピードや SEO(検 索エンジン最適化)の問題を改善すること。

アイソモーフィックなアプリケーションとは、複数のプラットフォームでレンダリング可能なアプリケーションを指す。

ユニバーサルなアプリケーションとは、コードを書き換えることなく複数のプラットフォームで実行可能なアプリケーションを指す。
ここで複数のプラットフォームと言っているのは、JavaScript の場合は、ブラウザと Node.jsを指す。
つまりブラウザで実行しているコードを、Node でサーバーや CLI として実行できれば、そのコードはユニバーサルな JavaScript と言える。

```
const userDetails = response => { 
  const login = response.login; 
  console.log(login);
};
```
例えば上記の関数 printNames は、ブラウザと Node.js の両方で実行可能であるため、ユニバーサルな JavaScript である。

ReactDOM.render を使えばtsxのコードをブラウザで描画できるが、ReactDOMServer. renderToString を使って HTML の文字列データに変換することも可能である。サーバー上でコンポーネントを HTML に変換してレスポンスとして返すことを、サーバーサイドレンダリングと 呼ぶ。

```
// Reactコンポーネントをブラウザで描画(クライアントサイドレンダリング) ReactDOM.render(<Star />);

// Reactコンポーネントをサーバーで描画してHTMLの文字列に変換(サーバーサイドレンダリング) let html = ReactDOMServer.renderToString(<Star />);
```

## React におけるサーバーサイドレンダリング
Create React App を使って作成されるアプリケーションは、描画方式としてはクライアントサイドレンダリングである。
つまり、アプリケーションはブラウザにより描画される。
この場合、アプリケーションの実体は build フォルダ内の JavaScript である。ブラウザは空の HTML をロード後に JavaScript ファイルをロードしてコンポーネントを描画する。

この方式は、ページの初期表示に時間を要するという難点がある。

### Next.js
Next を使うことで React のアプリケーションを簡単にサーバー上で実行できるようになる。また、ルーティングや静的ページの コンパイル、コード分割等、さまざまな機能を含んでいる。

**CSR**
ブラウザでアプリケーションを DOM として描画すること。Create React App でプロジェクトを作成した場合は基本 CSR となる。

**SSR**
サーバーでアプリケーションを静的な HTML として描画すること。CSR と SSR の両方に対応している場合、アイソモーフィックなアプリケーションと呼ぶ。

**ハイドレーション**
サーバーで描画された HTML をブラウザで再利用すること。アイソモーフィックなアプリケーションでは、SSR により生成された静的な HTML がブラウザでロードされて DOM ツリーが構築されると、以降は CSR によりアプリケーションが描画される。

**プリレンダリング**
アプリケーションをビルド時に静的な HTML に変換すること。静的なコンテンツとしてデプロイされるので、CDN によりキャッシュされ、非常に高速にロードされる。

### Gatsby
Gatsby はいわゆる静的サイトジェネレータで、コンテンツ主体の Web サイトを構築する手段として急速に普及しつつある。

Gatsby を使えばパフォーマンスやアクセシビリティ、画像の最適化など、 Web サイト構築に必要な多くの作業が自動化される。

以下がGatsby の提供する機能である。

**静的 Web サイトの生成**
Gatsby を使ってビルドされたアプリケーションは静的な Web ページとしてデプロイできる。

**CDN サポート**
静的な Web ページをCDN 経由でアクセスすることで、パフォーマンスと可用性を改善できる。

**プログレッシブ画像**
Gatsby は画像のプログレッシブローディングに対応している。このテクニックは Medium などのサイトで見られるが、感覚的にページの初期表示が速くなる効果がある。

**リンクのプリフェッチ**
ページが表示されると、そのページから遷移する可能性のあるページが、裏でロードされまる。そのようにユーザーがリンクをクリックする前にコンテンツを先読みしておくことで、高速なページ遷移を実現できる。
