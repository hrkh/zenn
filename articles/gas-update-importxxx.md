---
title: "GoogleスプレッドシートのIMPORT系の関数の結果を即座に更新する方法"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["gas"]
published: true
---

###  やりたかったこと

GoogleスプレッドシートでIMPORTDATA関数を使ってCSVデータを取り込んでいるが、元データが変更されているのに反映されないことがあったのでなんとかしたかった。

### 結論

GASで値を入れ直せばよかった。ただし、clear()したあと、setValue()する前にflush()する必要があった。

### 詳細

まず、普通にIMPORTDATA関数を試す。↓をシート1のA1セルに入力してみる。

```
=IMPORTDATA("https://hogehoge.csv")
```

すると、CSVのデータがシートに展開される。

ところが、CSVのデータが更新されていたとしても、シートを開き直しても即座に反映されない。

そこで、以下のように、ファイルを開くたびにGASで関数を消去し、再度入力するようにしてみるとうまくいく。

```javascript
const onOpen = () => {
    const spreadSheet = SpreadSheetApp.getActiveSpreadSheet();
    const sheet = spreadSheet.getSheetByName("シート1");
    const cell = sheet.getRange(1, 1);
    cell.clear();
    SpreadSheetApp.flush();  // これが重要
    cell.setValue(`=IMPORTDATA("https://hogehoge.csv")`);
};
```

### 試したがうまくいかなかったこと

- 「ファイル」＞「設定」＞「計算」＞「再計算」
を「変更時と毎分」にする
  - これは「NOW、TODAY、RAND、RANDBETWEENの更新頻度を設定します」と書いてあるとおり、IMPORT系関数には適用されないらしい
- clear()してからすぐsetValue()する
  - 関数がクリアされて再度セットされる様子は確認できるが、結果の値が更新されない
- clear()のあとにUtilities.sleep()を挟んでからsetValue()する
  - これも同様にうまくいかず、結果の値が更新されなかった

### 注意

今回はIMPORTDATAのURLをハードコーディングしているので特に問題はないが、他のセルを参照してURLを取得しているような場合、参照先のセルが行や列の挿入でずれたときに参照するセルがずれてしまい、正しいセルを参照できなくなることがあるので注意が必要。