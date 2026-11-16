# フロントエンドのテストプラクティス（Doddsの定義）

*参考: Kent C. Dodds（React Testing Library作者）*

---

## テストトロフィー

Khorikovのテストピラミッド（単体テスト多め）とは異なり、**統合テストに重心**を置く。

```
          ___
         |E2E|            ← 少数
     ____|___|____
    |  統合テスト  |       ← 最多（ここに重心）
     \___________/
       | 単体  |          ← 中程度（純粋ロジックのみ）
      _|_______|_
     |  静的解析  |        ← 土台（TS/ESLint）
     |___________|
```

コンポーネントを単体で検証しても意味が薄いため、統合テストに重心を置く。

---

## 各テストの対象

| テスト | 対象 |
|---|---|
| 静的解析 | TypeScript, ESLint |
| 単体テスト | 純粋関数、カスタムフックのロジック |
| 統合テスト | 複数コンポーネントの協調、ユーザー操作 |
| E2E | 画面をまたぐ主要フロー |

---

## 基本方針

**「実装の詳細をテストしない。ユーザーの視点でテストする。」**

```tsx
// BAD: 内部状態をテスト（実装の詳細）
expect(component.state.isOpen).toBe(true);

// GOOD: ユーザーが見るものをテスト
await userEvent.click(screen.getByRole("button", { name: "開く" }));
expect(screen.getByRole("dialog")).toBeVisible();
```

> The more your tests resemble the way your software is used, the more confidence they can give you.
> — Kent C. Dodds

---

## 外部依存（API）の扱い

APIクライアントやfetchを直接モックせず、**ネットワーク層でモックする**（例: MSW）。

```ts
// MSWでAPIレスポンスを差し替える
server.use(
  http.get("/api/users/1", () => HttpResponse.json({ name: "Alice" }))
);
```

コンポーネント内部の通信方法（fetch / axios など）を変えてもテストが壊れない。

---

## Khorikovとの対応関係

| Khorikov（バックエンド） | Dodds（フロントエンド） |
|---|---|
| ドメイン/アルゴリズムコード | 純粋関数・カスタムフック |
| コントローラー | コンポーネント |
| 単体テスト重視 | 統合テスト重視 |
| 依存の管理で判断 | ユーザー視点で判断 |
