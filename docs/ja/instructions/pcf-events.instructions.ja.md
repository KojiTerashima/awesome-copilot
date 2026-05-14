---
description: 'PCF コンポーネントでカスタム イベントを定義して処理する'
applyTo: '**/*.{ts,tsx,js,json,xml,pcfproj,csproj}'
---

# イベントの定義 (Preview)

[このトピックはプレリリース ドキュメントであり、変更される可能性があります。]

Power Apps Component Framework で custom component を構築するときによくある要件の 1 つは、control 内で生成された event に反応できることです。これらの event は、ユーザー操作によって発火することも、code によってプログラム的に発火することもあります。たとえば、アプリケーションにはユーザーが product bundle を構成できる code component を含めることができます。この component は event を発生させ、アプリケーションの別の領域に product 情報を表示させることもできます。

## コンポーネントのデータ フロー

code component の一般的なデータ フローは、hosting application から control に入力としてデータが流れ込み、更新されたデータが control から hosting form または page に流れ出る形です。次の図は、典型的な PCF component の標準的なデータ フロー パターンを示しています。

![コード コンポーネントからバインド フィールドへのデータ更新が OnChange イベントをトリガーすることを示す図](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/media/component-events-onchange-example.png)

code component から bound field へのデータ更新は `OnChange` event をトリガーします。ほとんどの component シナリオではこれで十分であり、maker は後続アクションを起動する handler を追加するだけです。ただし、より複雑な control では、field 更新ではない event を発火させる必要がある場合があります。event 機構により、code component は個別の event handler を持つ event を定義できます。

## イベントの使用

PCF における event 機構は、JavaScript の標準 event model に基づいています。component は manifest file 内で event を定義し、code 内でそれらの event を発火できます。hosting application はこれらの event を監視し、それに反応できます。

### Manifest でイベントを定義する

component は、manifest file の [event element](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/manifest-schema-reference/event) を使って event を定義します。このデータにより、それぞれの hosting application はさまざまな方法で event に反応できます。

```xml
<property
  name="sampleProperty"
  display-name-key="Property_Display_Key"
  description-key="Property_Desc_Key"
  of-type="SingleLine.Text"
  usage="bound"
  required="true"
/>
<event
  name="customEvent1"
  display-name-key="customEvent1"
  description-key="customEvent1"
/>
<event
  name="customEvent2"
  display-name-key="customEvent2"
  description-key="customEvent2"
/>
```

### Canvas Apps でのイベント処理

Canvas apps では、Power Fx 式を使って event に反応します。

![canvas apps designer 内の custom event を示す図](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/media/custom-events-in-canvas-designer.png)

### Model-Driven Apps でのイベント処理

Model Driven Apps では、[addEventHandler method](https://learn.microsoft.com/en-us/power-apps/developer/model-driven-apps/clientapi/reference/controls/addeventhandler) を使って、component の custom event に event handler を関連付けます。

```javascript
const controlName1 = "cr116_personid";

this.onLoad = function (executionContext) {
  const formContext = executionContext.getFormContext();

  const sampleControl1 = formContext.getControl(controlName1);
  sampleControl1.addEventHandler("customEvent1", this.onSampleControl1CustomEvent1);
  sampleControl1.addEventHandler("customEvent2", this.onSampleControl1CustomEvent2);
}
```

> **注**: これらの event は、app 内の code component の各 instance ごとに個別に発生します。

## モデル駆動型アプリ向けイベントの定義

model-driven apps では、より複雑なシナリオに対応するため event とともに payload を渡せます。たとえば次の図では、component が event 内で callback function を渡し、script handling 側から component へ callback できるようにしています。

![この例では、component が event 内で callback function を渡し、script handling 側から component に callback できるようにしている](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/media/passing-payload-in-events.png)

```javascript
this.onSampleControl1CustomEvent1 = function (params) {
   //alert(`SampleControl1 Custom Event 1: ${params}`);
   alert(`SampleControl1 Custom Event 1`);
}.bind(this);

this.onSampleControl2CustomEvent2 = function (params) {
  alert(`SampleControl2 Custom Event 2: ${params.message}`);
  // prevent the default action for the event
  params.callBackFunction();
}
```

## Canvas Apps 向けイベントの定義

maker は、properties pane 上の PCF control に対して Power Fx を使って event を構成します。

## イベントを呼び出す

event の呼び出し方法については、[Events](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/reference/events) を参照してください。

## 次のステップ

[Tutorial: Define a custom event in a component](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/tutorial-define-event)
