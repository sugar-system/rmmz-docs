# 戦闘シーンでのメッセージ表示位置について

## 問題点

RPGツクールMZの戦闘シーン（Scene_Battle）では、メッセージウィンドウ（Window_Message）の表示位置（\_positionType）が常に画面下部（\_positionType === 2）に固定されてしまう問題があります。

通常のシーンでは、イベントの「メッセージを表示」コマンドで指定した位置（上/中/下）にメッセージウィンドウが表示されますが、戦闘シーンではその設定が反映されにくい状況があります。

## 原因

コードを調査した結果、以下のような仕組みになっていることがわかりました：

1. イベントコマンド「メッセージを表示」が実行されると、Game_Interpreter.prototype.command101でパラメータに応じて`$gameMessage.setPositionType(params[3])`が呼ばれ、表示位置が設定されます
2. しかし、$gameMessage.clear()が実行されると、\_positionTypeは2（下部）にリセットされます
3. メッセージ表示時にWindow_Message.prototype.updatePlacementが実行され、`this._positionType = $gameMessage.positionType()`で$gameMessageの値を取得して位置を設定します

つまり、戦闘シーンで何らかの理由でメッセージのクリアが発生すると、次のメッセージ表示は必ず下部になってしまいます。これは特に戦闘中のスキル使用メッセージなどで顕著に現れます。

また、Window_Message.prototype.terminateMessageにおいてメッセージ終了時に`$gameMessage.clear()`が呼ばれるため、一度表示したメッセージが終了すると、次のメッセージはデフォルトの下部位置にリセットされてしまいます。

## 解決策

戦闘中に異なる位置にメッセージを表示したい場合は、**メッセージ表示の直前**に以下のコードを実行する必要があります：

```javascript
// メッセージを上部に表示
$gameMessage.setPositionType(0);

// または中央に表示
$gameMessage.setPositionType(1);

// デフォルト（下部）に表示
$gameMessage.setPositionType(2);
```

これをメッセージ表示のたびに実行する必要があります。プラグインを作成して、特定の戦闘メッセージの直前に位置タイプを設定するように実装するといいでしょう。

## 位置タイプの値

positionTypeの値は以下の通りです：

- 0: 上部
- 1: 中央
- 2: 下部（デフォルト）

## コードの詳細

### イベントコマンドでのメッセージ表示位置の設定

```javascript
// Show Text
Game_Interpreter.prototype.command101 = function (params) {
  if ($gameMessage.isBusy()) {
    return false;
  }
  $gameMessage.setFaceImage(params[0], params[1]);
  $gameMessage.setBackground(params[2]);
  $gameMessage.setPositionType(params[3]); // ここでメッセージの位置タイプを設定
  $gameMessage.setSpeakerName(params[4]);
  // ... 残りのコード ...
};
```

### Window_Message.prototype.updatePlacement

```javascript
Window_Message.prototype.updatePlacement = function () {
  const goldWindow = this._goldWindow;
  this._positionType = $gameMessage.positionType();
  this.y = (this._positionType * (Graphics.boxHeight - this.height)) / 2;
  if (goldWindow) {
    goldWindow.y = this.y > 0 ? 0 : Graphics.boxHeight - goldWindow.height;
  }
};
```

### Game_Message.prototype.clear

```javascript
Game_Message.prototype.clear = function () {
  this._texts = [];
  this._choices = [];
  this._speakerName = '';
  this._faceName = '';
  this._faceIndex = 0;
  this._background = 0;
  this._positionType = 2; // ここでデフォルトの位置タイプ（下部）に設定
  // ... 残りのコード ...
};
```

### メッセージ終了時のクリア処理

```javascript
Window_Message.prototype.terminateMessage = function () {
  this.close();
  this._goldWindow.close();
  $gameMessage.clear(); // ここでメッセージ情報がクリアされる
};
```

## まとめ

戦闘シーンでメッセージ表示位置を変更したい場合は、メッセージ表示の直前に明示的に`$gameMessage.setPositionType()`を呼び出して位置を設定する必要があります。これはエディタの設定だけでは解決できない仕様であり、プラグインやスクリプトを使って対応する必要があります。
