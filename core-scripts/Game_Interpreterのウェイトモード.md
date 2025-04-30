# Game_Interpreter とイベントの同期（ウェイトモード）

## 概要

RPG ツクール MZ では、`Game_Interpreter`クラスがイベントコマンドの実行を制御しています。ウェイトモードは、特定の動作が完了するまで次のコマンドの実行を待機させる機能です。これにより、メッセージ表示やアニメーションなどが完了するまで次のコマンドに進まないようにしています。

## 動作の仕組み

1. 各コマンドの実行メソッド（例：`command101`）の最後で `this.setWaitMode("xxx")` を呼び出し
2. 毎フレームの `update` で `updateWaitMode()` が実行される
3. 設定された条件が true の間は待機状態が続く
4. 条件が false になると `_waitMode = ""` が設定されて次のコマンドに進む

## ウェイトモードの一覧

`Game_Interpreter.prototype.updateWaitMode` メソッドから見つかったウェイトモードは全部で 10 種類あります：

### 1. message（メッセージ）

- **目的**: メッセージ表示が終わるまで待機
- **条件**: `$gameMessage.isBusy()` が true の間待機
- **使用例**: メッセージ表示コマンド（command101）
- **詳細**: `$gameMessage.isBusy()` は、テキストの表示中、選択肢表示中、数値入力中、アイテム選択中のいずれかの状態で true を返す

### 2. transfer（場所移動）

- **目的**: 場所移動が完了するまで待機
- **条件**: `$gamePlayer.isTransferring()` が true の間待機
- **使用例**: 場所移動コマンド

### 3. scroll（スクロール）

- **目的**: マップスクロールが終わるまで待機
- **条件**: `$gameMap.isScrolling()` が true の間待機
- **使用例**: マップスクロールコマンド

### 4. route（移動ルート）

- **目的**: 強制移動ルートが完了するまで待機
- **条件**: `character.isMoveRouteForcing()` が true の間待機
- **使用例**: 移動ルート設定コマンド
- **特徴**: 特定のキャラクター (`this._characterId`) に関連付けられる

### 5. animation（アニメーション）

- **目的**: アニメーション再生が終わるまで待機
- **条件**: `character.isAnimationPlaying()` が true の間待機
- **使用例**: アニメーション表示コマンド
- **特徴**: 特定のキャラクター (`this._characterId`) に関連付けられる

### 6. balloon（フキダシ）

- **目的**: フキダシアイコンの表示が終わるまで待機
- **条件**: `character.isBalloonPlaying()` が true の間待機
- **使用例**: フキダシアイコン表示コマンド
- **特徴**: 特定のキャラクター (`this._characterId`) に関連付けられる

### 7. gather（フォロワー集合）

- **目的**: フォロワー（仲間キャラ）の集合が終わるまで待機
- **条件**: `$gamePlayer.areFollowersGathering()` が true の間待機
- **使用例**: フォロワー集合コマンド

### 8. action（戦闘アクション）

- **目的**: 戦闘アクションが完了するまで待機
- **条件**: `BattleManager.isActionForced()` が true の間待機
- **使用例**: 戦闘関連コマンド

### 9. video（ビデオ）

- **目的**: 動画再生が終わるまで待機
- **条件**: `Video.isPlaying()` が true の間待機
- **使用例**: ムービー再生コマンド

### 10. image（画像）

- **目的**: 画像の読み込みが完了するまで待機
- **条件**: `!ImageManager.isReady()` が true の間待機
- **使用例**: 大きな画像を表示する前など

## コード例

下記は `updateWaitMode` メソッドの実装例です：

```javascript
Game_Interpreter.prototype.updateWaitMode = function () {
  let character = null;
  let waiting = false;
  switch (this._waitMode) {
    case 'message':
      waiting = $gameMessage.isBusy();
      break;
    case 'transfer':
      waiting = $gamePlayer.isTransferring();
      break;
    case 'scroll':
      waiting = $gameMap.isScrolling();
      break;
    case 'route':
      character = this.character(this._characterId);
      waiting = character && character.isMoveRouteForcing();
      break;
    case 'animation':
      character = this.character(this._characterId);
      waiting = character && character.isAnimationPlaying();
      break;
    case 'balloon':
      character = this.character(this._characterId);
      waiting = character && character.isBalloonPlaying();
      break;
    case 'gather':
      waiting = $gamePlayer.areFollowersGathering();
      break;
    case 'action':
      waiting = BattleManager.isActionForced();
      break;
    case 'video':
      waiting = Video.isPlaying();
      break;
    case 'image':
      waiting = !ImageManager.isReady();
      break;
  }
  if (!waiting) {
    this._waitMode = '';
  }
  return waiting;
};
```

## まとめ

ウェイトモードは、イベントコマンドの処理と視覚的・聴覚的な演出を確実に同期させるための重要な機能です。これにより、ゲームプレイの流れを制御し、ユーザーに自然な体験を提供しています。
