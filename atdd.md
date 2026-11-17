# ATDD（受け入れテスト駆動開発）

ATDD（Acceptance Test Driven Development）とは、**実装の前に受け入れ条件をテストとして書き、それを満たすように開発する**手法。

---

## TDDとの違い

| | TDD | ATDD |
|---|---|---|
| 視点 | 開発者 | 顧客・ビジネス |
| 粒度 | クラス・関数 | ユーザーストーリー・機能 |
| 問い | 正しく作っているか | 正しいものを作っているか |
| 書く人 | 開発者 | PO・QA・開発者の三者で合意 |

ATDDは外側のループ、TDDは内側のループとして組み合わせる（二重ループ）。

```
[受け入れテストを書く（失敗）]
        ↓
   ┌→ [単体テストを書く（失敗）] → [実装] → [リファクタリング] ─┐
   └──────────────── 受け入れテストが通るまで繰り返す ←────────┘
        ↓
[受け入れテストが通る → 完了]
```

---

## サイクル

1. **Discuss（議論）** — PO・QA・開発者（スリーアミーゴス）でストーリーの受け入れ条件を具体例で話し合う
2. **Distill（抽出）** — 具体例を実行可能なテストの形に落とし込む
3. **Develop（開発）** — テストが通るように実装する（内側はTDD）
4. **Demo（デモ）** — 受け入れテストが通った状態でステークホルダーに見せる

---

## 受け入れ条件の書き方

**具体例（Specification by Example）で書く。** 抽象的なルールではなく、入力と期待結果の組で表す。

```gherkin
Feature: 送料計算

  Scenario: 購入金額が5000円以上なら送料無料
    Given カートに 5000 円の商品がある
    When 注文を確定する
    Then 送料は 0 円である

  Scenario: 購入金額が5000円未満なら送料500円
    Given カートに 4999 円の商品がある
    When 注文を確定する
    Then 送料は 500 円である
```

**ポイント:**
- ビジネスの言葉で書く（UIやDBなどの実装の詳細を書かない）
- 境界値を具体例に含める
- 1シナリオ1振る舞い

---

## 実装例（Cucumber + JAX-RS）

```java
public class ShippingSteps {

    @Inject
    CartService cartService;

    @Inject
    OrderService orderService;

    private Order order;

    @Given("カートに {int} 円の商品がある")
    public void カートに商品がある(int price) {
        cartService.add(new Item("商品A", price));
    }

    @When("注文を確定する")
    public void 注文を確定する() {
        order = orderService.checkout(cartService.getCart());
    }

    @Then("送料は {int} 円である")
    public void 送料は(int expected) {
        assertEquals(expected, order.getShippingFee());
    }
}
```

ステップ定義はアプリケーションサービス層（ユースケース）を呼ぶ。UI経由にすると遅く壊れやすくなるため、UIの検証が必要なものだけE2Eで行う。

---

## よくある失敗

- **開発者だけで書く** → 「正しいものを作っているか」の確認にならない。合意形成こそが本質
- **UI操作の手順を書く**（「ボタンをクリックし…」） → 画面変更でテストが壊れる。振る舞いを書く
- **実装後に書く** → ただの回帰テスト。仕様の曖昧さを実装前に潰すのが目的
- **すべてのケースを受け入れテストで網羅する** → 遅くなる。代表例のみ書き、網羅は単体テストで行う
