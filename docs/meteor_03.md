# Day 03 [從 MeteorJS 學習網路應用開發] Meteor + React

在上一篇文章中我們創建 Meteor 的專案，並設定好資料夾結構，這篇文章會開始安裝 React 和 Redux，並同時設定令人又愛又恨的 eslint 藉此檢查 code style，並預防錯誤發生。那麼就開始吧！

首先，一樣按照[官網的指示](https://guide.meteor.com/react.html)安裝 react，這裡一定要使用 `meteor npm install`
```bash
# meteor yarn add react react-dom 也行
meteor npm install --save react react-dom
```

接著將官方的 *client/main.js*、*client/main.html* 修改，並仿造原先的計數器畫面加到 *imports/ui/components/App.jsx* 如下：詳細的原始碼請參考 [github](https://github.com/pomelyu/meteor-message)
```javascript
// client/main.jsx
import { Meteor } from 'meteor/meteor';
import React from 'react';
import { render } from 'react-dom';

import App from '../imports/ui/components/App';

Meteor.startup(() => {
  render(<App />, document.getElementById('app'));
});
```
```javascript
// imports/ui/components/App.jsx
class App extends React.Component {
  constructor(props) {
    this.state = { counter: 0 }
    // ...
  }
  render() {
    // ...
  }
}
```

確認執行成功後，可以將 Meteor 內附的 Blaze 移除
```bash
meteor remove blaze-html-templates
meteor add static-html
```

接著我們可以開始設定 eslint，由於 Meteor 會增加某些的全域變數，導致在開發上可能不小心引用到錯誤的變數，例如 'meteor-base' 這個套件就將 [underscore](http://underscorejs.org) 這個小工具包的常用變數 `_` 設為全域，如果像筆者常用的是另一個小工具包 [loadash](https://lodash.com/docs/)，開發時一旦忘記載入，就會發生各種亂七八糟的錯誤訊息。事實上利用 `meteor add` 安裝套件平台(atmosphere)上的套件有部分就會出現這樣的問題，因此還是盡可能的利用 eslint 來防止直接引用這些全域變數，如果真的必須使用，可以將`/* global var-name */`放置在文件開頭，來暫時允許特定的全域變數。

所以首先先安裝 eslint
```bash
meteor npm install --save-dev babel-eslint eslint-config-airbnb eslint-plugin-import eslint-plugin-meteor eslint-plugin-react eslint-plugin-jsx-a11y eslint-import-resolver-meteor eslint @meteorjs/eslint-config-meteor
```
接著設定 *.eslintrc*，這邊筆者是參考 stackoverflow 上網友的回答，可以根據需求自行修改，只列出與 meteor 特別相關的，配合 VSCode 的 eslint 套件可以自動檢查。
```javascript
{
  "plugins": [
    "meteor"
  ],
  "extends": [
    "airbnb",
    "plugin:meteor/recommended",
    "eslint:recommended"
  ],
  "rules": {
    "import/no-unresolved": [
      2, { "ignore": ["^meteor/"] }
    ],
    // add some message to meteor global variable
    "no-restricted-globals": [
      "error", {
        "name": "Meteor",
        "message": "import { Meteor } from 'meteor/meteor' instead"
      },
      {
        "name": "_",
        "message": "do not use the global variable _"
      }
    ]
  },
}
```

React 和 eslint 的環境設定就到這裡，在下一篇會以一個簡單、不具文字留存功能的聊天視窗當作範例，並使用 Redux 切換視窗的佈景，接著再進入 Meteor 的 data flow 部分，並和 Redux 的 data flow 作比較。
