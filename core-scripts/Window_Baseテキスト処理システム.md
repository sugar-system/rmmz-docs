# Window_Baseのテキスト処理システム

MZのテキスト処理システムは、`Window_Base`クラスの`processCharacter`メソッドを中心に構築されています。このドキュメントでは、テキスト処理の仕組みと独自の制御文字を追加する方法について説明します。

## textStateオブジェクト

textStateは、テキスト描画時に使用される状態オブジェクトです。TypeScriptの型定義で表すと以下のようになります：

```typescript
interface TextState {
  text: string; // 描画するテキスト
  index: number; // 現在処理中の文字インデックス
  x: number; // 現在のX座標
  y: number; // 現在のY座標
  width: number; // 描画領域の幅
  height: number; // 1行の高さ
  startX: number; // 行の開始X座標
  startY: number; // 開始Y座標
  rtl: boolean; // 右から左への表示かどうか
  buffer: string; // 文字バッファ
  drawing: boolean; // 描画中かどうか
  outputWidth: number; // 出力された幅
  outputHeight: number; // 出力された高さ
}
```

textStateの初期化と使用法は以下のとおりです：

```javascript
const textState = this.createTextState(text, x, y, width);
this.processAllText(textState);
```

## processCharacterメソッド

`processCharacter`メソッドは各文字を処理する中心的なメソッドです：

```javascript
Window_Base.prototype.processCharacter = function (textState) {
  const c = textState.text[textState.index++];
  if (c.charCodeAt(0) < 0x20) {
    this.flushTextState(textState);
    this.processControlCharacter(textState, c);
  } else {
    textState.buffer += c;
  }
};
```

このメソッドは：

1. 現在の文字を取得してインデックスを進める
2. 文字コードが0x20未満（制御文字）の場合：
   - `flushTextState`でバッファの内容を描画
   - `processControlCharacter`で制御文字を処理
3. 通常の文字の場合はバッファに追加

## 制御文字の処理

`processControlCharacter`メソッドは制御文字を処理します：

```javascript
Window_Base.prototype.processControlCharacter = function (textState, c) {
  if (c === '\n') {
    this.processNewLine(textState);
  }
  if (c === '\x1b') {
    const code = this.obtainEscapeCode(textState);
    this.processEscapeCharacter(code, textState);
  }
};
```

このメソッドは：

1. 改行文字（`\n`）の場合は改行処理
2. エスケープ文字（`\x1b`）の場合は後続のエスケープコードを取得して処理

## 独自のコントロール文字を追加する方法

独自の制御文字を追加するには、以下の方法があります：

### 1. processControlCharacterをオーバーライドする

`Window_Message`クラスの実装を参考に、独自の制御文字を追加できます：

```javascript
// 既存の処理を継承しつつ拡張する例
MyWindow.prototype.processControlCharacter = function (textState, c) {
  // 親クラスの処理を呼び出す
  Window_Base.prototype.processControlCharacter.call(this, textState, c);

  // 独自の制御文字処理を追加
  if (c === '\f') {
    // フォームフィード文字
    this.processMySpecialCharacter(textState);
  }
};

MyWindow.prototype.processMySpecialCharacter = function (textState) {
  // 独自の処理をここに実装
};
```

### 2. processEscapeCharacterをオーバーライドする

エスケープ文字（`\X[param]`のような形式）を追加するには：

```javascript
MyWindow.prototype.processEscapeCharacter = function (code, textState) {
  switch (code) {
    case 'X': // 独自コード「\X」
      this.processMyEscapeCode(this.obtainEscapeParam(textState));
      break;
    default:
      // 親クラスの処理を呼び出す
      Window_Base.prototype.processEscapeCharacter.call(this, code, textState);
      break;
  }
};

MyWindow.prototype.processMyEscapeCode = function (param) {
  // 独自エスケープコードの処理
  // 例：特殊効果の適用、スタイル変更など
};
```

## 実際の使用例

以下は、アイコンを表示する制御文字の例です：

```javascript
// \I[n] でアイコンを表示する処理
Window_Base.prototype.processDrawIcon = function (iconIndex, textState) {
  if (textState.drawing) {
    const x = textState.x + 2;
    const y = textState.y + 2;
    this.drawIcon(iconIndex, x, y);
  }
  textState.x += ImageManager.iconWidth + 4;
};
```

これにより、テキスト中に `\I[42]` と記述することでアイコン（ID: 42）を表示できます。
