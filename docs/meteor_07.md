# Day 07 [從 MeteorJS 學習網路應用開發] 存取 mongo DB 和資料驗證

如同前面提到的 Meteor 高度仰賴 mongoDB 作為資料庫，無論是作為後端的資料儲存，或是前端 minimongo 的資料暫存。因為 monogoDB NoSQL 的特性讓程式運行中可以加入新的欄位(key)，如果對於資料庫的修改沒有進行驗證，很可能程式運行的結果會讓資料庫莫名其妙多出一堆欄位和沒有意義的資料，因此在今天的前半部分會先以 [simpl-schema](https://github.com/aldeed/simple-schema-js) 這個函式庫來介紹如何在更新資料時作驗證，後半部分整理並比較 mongo 原生的 selector, modifier 指令和 meteor 包裹起來的差別。

mongoDB 是很常使用的 NoSQL 資料庫，一般來說的 SQL 資料庫就像是一張張的表格(table)，一開始就必須定義每個欄位描述的是什麼還有它的型別，例如使用者Id(String), 登入時間(String), 登入次數(Number)等等的，一旦加入新的功能必須要新增欄位，會是非常困難的事情。

相對來說，NoSQL 是用 key-value 的方式來儲存每一筆資料，也就是 JSON object 的方式，因此每一資料的 key 可以不同不同，也可以動態的新增或是刪除 key。不過在實際上，即使是 NoSQL 的資料我們還是希望每筆資料盡可能的扁平，同一個 collection 的資料也盡可能維持相同的 key，所以這個時候先根據開發的需求定義好資料庫的 schema 就會是一個好選擇。

雖然 [mongoose](https://github.com/Automattic/mongoose) 是相當受歡迎用來存取 mongoDB 的 library，但是這邊只需要 schema 驗證的功能，因此選用很常搭配 meteor 使用的 [simpl-schema](https://github.com/aldeed/simple-schema-js) ，這邊的操作會偏向手動的驗證資料，在判斷是否要執行資料庫的更新，如果需要自動在更新資料庫實作驗證，可以參考 [meteor/collections2](https://github.com/aldeed/meteor-collection2)。

整個資料驗證的流程非常簡單：
1. 事先定義 schema
2. 當出現需要寫入或更新資料時，先使用 schema.clean(data)，將沒有定義在 schema 的 key 剔除，並自動賦予部分 key 值，例如給予 updateAt 現在的時間
3. 使用 schema.validate(data) 驗證是否有錯誤
4. 更新資料

以下就先來設計 todos 的 schema
```javascript
import SimpleSchema from 'simpl-schema';

const schema = new SimpleSchema({
  id: String,
  // defaultValue: 表示在 clean 的時候會自動產生的值
  checked: { type: Boolean, defaultValue: false },
  isPublic: {
    type: Boolean,
    autoValue: function (){
      // 如果此欄位已經有值，回傳該值
      if (this.isSet) {
        return this.value;
      }
      // 如果此欄位沒有值，且是 modifier
      if (this.operator) {
        return this.unset();
      }
      return false;
    }
  }
}).newContext()

const data = { id: 'abc' };
const dataCleaned = schema.clean(data);
console.log(dataCleaned);
// { id: 'abc', checked: false, isPublic: false }
console.log(schema.validate(dataCleaned));
// true;

const modifier = { $set: { checked: 'hig', QQ: 'XD' } };
const modifierCleanded = schema.clean(modifier);
console.log(modifierCleanded);
// { '$set': { checked: 'hig' } }
console.log(schema.validate(modifierCleanded, { modifier: true }));
// false

```

所以昨天的 todo method 就可以作以下的更動
```javascript
'todos.add': function(todo) {
  check(todo, Object);
  // ========
  const { schema } = Todos;
  const dataToInsert = schema.clean(todo);
  schema.validate(dataToInsert);
  // ========
  Todos.insert(todo);
},
```

接下來整理 mongoDB 和 meteor 的 DB selector 和 modifier
