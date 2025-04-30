# Game_Interpreterの実行終了の仕組み

このドキュメントでは、RPGツクールMZにおける`Game_Interpreter`クラスの実行終了処理の流れと、分岐構造の管理について解説します。

## 実行終了の仕組み

### isRunningメソッド

```js
Game_Interpreter.prototype.isRunning = function () {
  return !!this._list;
};
```

`isRunning`メソッドは、インタープリタがコマンドリストを持っているかどうかを判定します。`this._list`が`null`でなければ`true`を返し、実行中と判断されます。

### updateメソッドの流れ

```js
Game_Interpreter.prototype.update = function () {
  while (this.isRunning()) {
    if (this.updateChild() || this.updateWait()) {
      break;
    }
    if (SceneManager.isSceneChanging()) {
      break;
    }
    if (!this.executeCommand()) {
      break;
    }
    if (this.checkFreeze()) {
      break;
    }
  }
};
```

`update`メソッドは毎フレーム呼び出され、`isRunning()`が`true`の間、つまりコマンドリストがある間は繰り返し実行されます。以下の条件でループが中断されます：

1. 子インタープリタの処理中または待機中の場合
2. シーン切り替え中の場合
3. コマンド実行が失敗または終了した場合
4. フリーズチェックに引っかかった場合（無限ループ防止）

### executeCommandメソッド

```js
Game_Interpreter.prototype.executeCommand = function () {
  const command = this.currentCommand();
  if (command) {
    this._indent = command.indent;
    const methodName = 'command' + command.code;
    if (typeof this[methodName] === 'function') {
      if (!this[methodName](command.parameters)) {
        return false;
      }
    }
    this._index++;
  } else {
    this.terminate();
  }
  return true;
};
```

このメソッドでは、現在のコマンドを取得し、対応するメソッドを実行します。コマンドがなくなった場合（`command`が`undefined`の場合）に`terminate`メソッドが呼び出されます。

### currentCommandメソッド

```js
Game_Interpreter.prototype.currentCommand = function () {
  return this._list[this._index];
};
```

`currentCommand`メソッドは、コマンドリスト（`this._list`）の現在の位置（`this._index`）にあるコマンドオブジェクトを返します。コマンドリストの最後に達した場合は`undefined`を返します。

### terminateメソッド

```js
Game_Interpreter.prototype.terminate = function () {
  this._list = null;
  this._comments = '';
};
```

`terminate`メソッドでは、`this._list`を`null`にセットすることで、次回の`isRunning()`呼び出しで`false`が返されるようになります。これにより、`update`のループが終了し、インタープリタの実行が停止します。

## まとめ

1. 毎フレーム`update`メソッドが呼び出される
2. `update`メソッドは`isRunning()`が`true`の間ループする
3. ループ内で`executeCommand`が実行され、コマンドリストにあるコマンドを1つずつ処理する
4. コマンドがなくなると`executeCommand`内で`terminate`が呼ばれる
5. `terminate`で`this._list = null`が実行される
6. 次回の`isRunning()`呼び出しで`false`が返され、`update`のループが終了する

RPGツクールMZのイベント実行システムは、このような仕組みで効率的にコマンドの実行と分岐構造の管理を行っています。
