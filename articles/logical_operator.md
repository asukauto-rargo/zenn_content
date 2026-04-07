---
title: "論理演算子と短絡評価"
emoji: "💻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["TypeScript"]
published: true
published_at: 2026-04-07
---

## はじめに

TypeScriptのコードを読んでいると、`&&` や `||` はもちろん、`??` や `!!` といった記号に出くわすことがある。
条件分岐でしか使わないと思っていたこれらの演算子には、**短絡評価（short-circuit evaluation）** という仕組みを活かした便利な使い方が存在する。

本記事では、それぞれの演算子の「条件分岐以外」の振る舞いと実践的な使い所を整理していく。

---

## `&&`（論理AND）— 左辺がfalsyなら即座に返す

### おなじみの使い方

`&&` は「かつ」を意味する論理演算子で、両辺がともに `true` のときだけ式全体が `true` になる。

```typescript
const num: number = 59;
if (0 <= num && num <= 100) {
  console.log("0から100の数です。");
}
```

### 短絡評価としての使い方

実は `&&` は、条件分岐の外でも値を返す式として使うことができる。

```typescript
const result = 0 && 100;
console.log(result); // 0
```

ルールはシンプルで、**左辺をBooleanに変換した結果がfalsyなら左辺を、truthyなら右辺を返す**。

上の例では左辺 `0` はfalsyなので、右辺 `100` は評価されず、`0` がそのまま返される。これが「短絡（ショートサーキット）」と呼ばれる所以である。

:::message
**truthyとfalsy**
JavaScriptでは、`0`・`""`（空文字）・`null`・`undefined`・`NaN`・`false` がfalsyとして扱われ、それ以外はすべてtruthyである。
:::

### 使い所

Reactなどで条件付きレンダリングに使われるパターンが代表的。

```tsx
// itemsが空配列でなければリストを表示する
{
  items.length > 0 && <ItemList items={items} />;
}
```

左辺がfalsyのときは右辺を評価しないため、**不要な処理をスキップできる**というメリットがある。

---

## `||`（論理OR）— 左辺がfalsyなら右辺へフォールバック

### おなじみの使い方

`||` は「または」を意味し、いずれか一方が `true` であれば式全体が `true` になる。

### 短絡評価としての使い方

`&&` の逆で、**左辺がtruthyならそのまま返し、falsyなら右辺を返す**。

```typescript
const result = 0 || 100;
console.log(result); // 100
```

左辺の `0` はfalsyであるため、右辺の `100` が返されている。

### 使い所

デフォルト値の設定に便利。

```typescript
function greet(name?: string) {
  const displayName = name || "ゲスト";
  console.log(`こんにちは、${displayName}さん！`);
}

greet(); // こんにちは、ゲストさん！
greet("太郎"); // こんにちは、太郎さん！
```

ただし `||` には落とし穴がある。`0` や `""` もfalsyであるため、それらを有効な値として扱いたい場面では意図しない挙動になる。

```typescript
const count = 0 || 10;
console.log(count); // 10 — 0を有効な値として使いたかったのに上書きされてしまう
```

この問題を解決するのが、次に紹介する `??` である。

---

## `??`（Null合体演算子）— nullとundefinedだけを区別する

### 使い方

`??` は **左辺が `null` または `undefined` のときだけ右辺を返す**。

```typescript
const input: string | null = null;
const value: string = input ?? "default";
console.log(value); // "default"
```

### `||` との違い

`||` と `??` の違いは、**何を「値がない」とみなすか**にある。

| 左辺の値       | `\|\|` の結果 | `??` の結果    |
| -------------- | ------------- | -------------- |
| `null`         | 右辺を返す    | 右辺を返す     |
| `undefined`    | 右辺を返す    | 右辺を返す     |
| `0`            | 右辺を返す    | **左辺を返す** |
| `""`（空文字） | 右辺を返す    | **左辺を返す** |
| `false`        | 右辺を返す    | **左辺を返す** |

`??` は `null` と `undefined` だけをフォールバックの対象とするため、「データが存在しない」場合にだけデフォルト値を適用したいときに最適である。

### 使い所

API応答やオプション設定など、`0` や `false` が有効な値として使われる場面で活躍する。

```typescript
// APIから取得したスコアが0でも有効な値として扱いたい
const score: number | null = fetchScore(); // 0が返ってくる可能性がある
const displayScore = score ?? "未取得";
// || を使うと、score が 0 のとき "未取得" になってしまう
```

---

## `!!`（二重否定）— 任意の値をBooleanに変換する

### 使い方

`!` は真偽値を反転させる演算子。これを2回適用すると、元の値のtruthy/falsyをそのまま `boolean` 型に変換できる。

```typescript
const num: number = 0;
console.log(!num); // true  — 反転
console.log(!!num); // false — さらに反転 → 元のfalsy状態をbooleanで取得
```

`Boolean(num)` と同じ結果になるが、`!!` のほうが短く書けるため好んで使われることがある。

### 使い所

値の有無をboolean型で明示的に扱いたいときに使う。

```typescript
const username: string | undefined = getUser();
const isLoggedIn: boolean = !!username;
// undefinedや空文字ならfalse、それ以外ならtrue
```

:::message
**`!!` を使うべきか？**
`!!` は短く書ける反面、慣れていないと読みづらいという意見もある。チームの規約や可読性を優先するなら `Boolean()` を使うのも良い選択肢である。
:::

---

## まとめ

| 演算子 | 振る舞い                                        | 主な用途                   |
| ------ | ----------------------------------------------- | -------------------------- |
| `&&`   | 左辺がfalsyなら左辺を返す、truthyなら右辺を返す | 条件付き実行・レンダリング |
| `\|\|` | 左辺がtruthyなら左辺を返す、falsyなら右辺を返す | デフォルト値の設定         |
| `??`   | 左辺がnull/undefinedなら右辺を返す              | null安全なデフォルト値     |
| `!!`   | 値をbooleanに変換する                           | truthy/falsyの明示的な判定 |

短絡評価を理解すると、冗長な `if` 文を簡潔に書き換えられる場面が増える。ただし、過度に省略すると可読性が下がるため、チームの方針や読み手のレベルに合わせてバランスよく使い分けることが大切である。
