# \_indentプロパティと分岐構造の管理

## \_indentプロパティの役割

`_indent`プロパティはイベントの分岐構造（ネスト）を管理するために使用されます。

```js
// 初期化時
this._indent = 0;

// コマンド実行時に更新
this._indent = command.indent;
```

### 分岐のスキップ処理

```js
Game_Interpreter.prototype.skipBranch = function () {
  while (this._list[this._index + 1].indent > this._indent) {
    this._index++;
  }
};
```

`skipBranch`メソッドは、現在の`_indent`より大きいインデント値を持つコマンド（現在の分岐ブロック内のコマンド）をすべてスキップします。

## 分岐結果の管理

```js
// 選択肢の結果を保存する例
$gameMessage.setChoiceCallback(n => {
  this._branch[this._indent] = n;
});

// 条件分岐処理の例
if (this._branch[this._indent] !== params[0]) {
  this.skipBranch();
}
```

`_branch`オブジェクトには、各インデントレベルでの分岐結果が保存されます。これによりネストした分岐でも正しく処理することができます。