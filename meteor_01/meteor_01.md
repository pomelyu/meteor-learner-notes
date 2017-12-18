# Meteor 學習筆記 - (1) Meteor 環境安裝與設置

## 安裝與資料夾配置

這部份會說明如何安裝並設置 Meteor 開發環境，包含安裝 React、使用 yarn 取代內建的 npm、安裝 css-module 和 scss，安裝並設置 eslint，以及設置專案的資料夾結構。

首先可以先到官網[安裝 Meteor](https://www.meteor.com/install)，在 Linux 或 Mac 上只要執行 `curl https://install.meteor.com/ | sh` 即可。安裝後，可以使用 Meteor 提供的 `meteor create`，來產生新的專案。


```bash
# 1. creat an empty meteor project
meteor create myApp && cd myApp

# 2.1 install node modules：這邊必須要用 meteor 內建的 npm 安裝，
# 如果直接用 npm 安裝會有問題，未來安裝其他 node module 也是一樣
meteor npm install

# 2.2 或者可以改用 yarn 來安裝，會是更好的選擇，但可能在某些系統會有問題
# 未來更新 meteor 之後，還要重新安裝一次
meteor npm i -g yarn
meteor yarn install

# 3. 執行預設的專案
meteor run
```
PS：
- meteor 安裝 yarn 可以參考[這個討論串](https://forums.meteor.com/t/yarn-and-meteor/30254/13)
- `meteor create --bare` 和 `meteor create --full`，也能用來產生新的專案，但 `--bare` 產生的是空的專案，`--full` 產生的專案比較適合用 meteor template 開發

在預設的專案和官方的教學中，`api` 底下的檔案並沒有特別區分，但我認為至少需要拆開成 `collections`, `publications` 和 `methods`，分別用來描述：資料庫的 schema、server 發佈的資料和 sever 提供的 api。另外還拆出 interface 資料夾用來儲存直接資料庫的函式。調整專案產生後的資料夾結構如下（只列出重要的）：
```bash
.meteor/
  packages              # meteor packages used by this project

client/                 # [automatically import in client side]
  main.js               # (client) The entrace point of client

server/                 # [automatically import in server side]
  main.js               # (server) The entrace point of server

imports/                # [mutually import by client or server]
  api/
    collections/        # (both) The mongo & minimogo collections and their schema
    methods/            # (both) The api to insert, remove, update or query data from database
    publications/       # (server) The publisher to publish reactive data
    interface/          # (server) The database operation in server side
  startup/
    client/             # (client) client startup configuration
    server/             # (server) server startup configuration
  ui/                   # (client) Front-end (react or etc)

private/                # (server) Asset only for server

public/                 # Public asset, such as favicon.ico

...                     # [other folder would be automatically import by both server and client side]
```

此外

Meteor 在建制專案時，會根據資料夾的名稱和順序，將檔案載入前端或是者是後端，所以資料夾的配置不可以錯誤，它依循以下規則：
1. 根目錄下 client 資料夾會被 client 載入、server 資料夾會被 server 載入，imports 資料夾會根據呼叫 import 的位置決定，其他的資料夾會同時被 server 和 client 載入。
2. 在任何位置的 `client/` 底下的檔案都不會被 server 載入，在任何位置的 `server/` 底下的檔案都不會被 client 載入
3. 如果是同時被 client, server 載入的檔案，可以用`if(Meteor.isServer)` 或 `if(Meteor.isClient)` 限制執行的位置，這一點在實作 method api 時很重要。也就是
```javascript
// This file is imported by both client and server,
// for example the definition of collections and meteor method

console.log('Execute in both');

if (Meteor.isServer) {
  console.log('Execute in server');
}

if (Metoer.isClient) {
  console.log('Execute in client');
}
```

詳細的規則可以看[官方的說明](https://guide.meteor.com/structure.html)。雖然看起來這些規則很麻煩，但簡單來說，只要記得所有的檔案都應該放在 imports 底下，利用程式碼來從 `client/main.js` 和 `server/main.js` 載入（也就是 lazy loading）就好，盡可能保持根目錄`client/`、`server/` 底下沒有多餘的檔案，避免讓 meteor 自己載入（eager loading）。
