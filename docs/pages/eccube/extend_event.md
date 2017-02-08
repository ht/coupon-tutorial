---
title: イベントで機能を拡張する
keywords:
tags: []
sidebar: mydoc_sidebar
permalink: extend_event.html
folder: eccube
---

EC-CUBE本体の処理にプラグインが介入する、フックポイントという機能を紹介します。

## フックポイントの設定

config.yml[](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/config.yml#L2){:target="_blank"}

``` yaml
# イベントを処理するクラス名(=ファイル名)
event: Event
```

event.yml[](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/event.yml){:target="_blank"}

``` yaml
# 下記の書式でイベントを記述
# イベント名:
#     - [関数名, 優先度(FIRST/NORMAL/LAST)]
#     - [同上 複数登録可能]

# 例 (受注登録編集完了時のイベント)
admin.order.edit.index.complete:
    - [onOrderEditComplete, NORMAL]
```

Event.php[](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/Event.php){:target="_blank"}

``` php
<?php
class Event
{
    private $app;

    public function __construct($app)
    {
        $this->app = $app;
    }
    
    public function onOrderEditComplete(EventArgs $event)
    {
        // イベント処理
    }
}
```

## フックポイントの種類

### 共通フックポイント

フロント画面・管理画面・ルーティング毎に、Symfony2のHttpKernelのライフサイクルの各ステップ(下記)で処理を差し込むことが可能です。

- Request
- Controller
- Response
- Exception
- Terminate

共通イベントの詳細は[EC-CUBE3 プラグイン仕様](http://downloads.ec-cube.net/manual/v3/plugin.pdf#8){:target="_blank"}を参照して下さい。

上記ドキュメントには記載されていませんが、```eccube.event.render.[route].before``` というイベントも存在します。
これは```eccube.event.route.[route].response``` の直後のフックポイントです。

### 個別フックポイント

コントローラ内部処理・テンプレート描画・メール送信等、EC-CUBE独自のタイミングで処理を差し込むことが可能です。

#### コントローラ内部処理

(本体) Controller/Admin/Order/EditController.php[](https://github.com/EC-CUBE/ec-cube/blob/3.0.13/src/Eccube/Controller/Admin/Order/EditController.php#L214-L224){:target="_blank"}

``` php
<?php
// イベント引数を作成
$event = new EventArgs(
    array(
        'form' => $form,
        'OriginOrder' => $OriginOrder,
        'TargetOrder' => $TargetOrder,
        'OriginOrderDetails' => $OriginalOrderDetails,
        'Customer' => $Customer,
    ),
    $request
);

$app['eccube.event.dispatcher']->dispatch(EccubeEvents::ADMIN_ORDER_EDIT_INDEX_COMPLETE, $event);
```

(本体) Event/EccubeEvents.php[](https://github.com/EC-CUBE/ec-cube/blob/3.0.13/src/Eccube/Event/EccubeEvents.php#L141){:target="_blank"}

``` php
<?php
// イベント名はこのクラス内にまとめて記述されている
const ADMIN_ORDER_EDIT_INDEX_COMPLETE = 'admin.order.edit.index.complete';
```

event.yml[](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/event.yml#L54-#L55){:target="_blank"}

``` yaml
admin.order.edit.index.complete:
    - [onOrderEditComplete, NORMAL]
```

Event.php[](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/Event.php#L110){:target="_blank"}

``` php
<?php
public function onOrderEditComplete(EventArgs $event)
{
    // $this->app['eccube.plugin.coupon.event']の中身は\Plugin\Coupon\Event\Eventのインスタンス
    $this->app['eccube.plugin.coupon.event']->onOrderEditComplete($event);
}
```

Event/Event.php[](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/Event/Event.php#L369){:target="_blank"}

``` php
<?php
// イベント引数にセットした値を取得可能
if ($event->hasArgument('TargetOrder')) {
    $Order = $event->getArgument('TargetOrder');
    if (is_null($Order)) {
        return;
    }
}
```

イベント引数の詳細は```EventArgs``` [](https://github.com/EC-CUBE/ec-cube/blob/3.0.13/src/Eccube/Event/EventArgs.php){:target="_blank"}クラスを参照して下さい。

#### テンプレート描画

イベント名にテンプレート名を指定すると、そのテンプレートの描画前のタイミングで処理を差し込むことが可能です。

テンプレートの拡張については[イベントでテンプレートを拡張する]({{site.baseurl}}/extend_template.html)を参照して下さい。

event.yml[](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/event.yml){:target="_blank"}

``` yaml
Shopping/index.twig:
    - [onRenderShoppingIndex, NORMAL]
    
# 管理画面のテンプレートの場合は先頭にAdmin/を付与して下さい。
Admin/Order/edit.twig:
    - [onRenderAdminOrderEdit, NORMAL]
```

Event.php[](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/Event.php#L42){:target="_blank"}

``` php
<?php
public function onRenderShoppingIndex(TemplateEvent $event)
{
    $this->app['eccube.plugin.coupon.event']->onRenderShoppingIndex($event);
}
```

Event/Event.php[](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/Event/Event.php#L62){:target="_blank"}

``` php
<?php
public function onRenderShoppingIndex(TemplateEvent $event)
{
    // 省略
}
```

イベント引数の詳細は```TemplateEvent``` [](https://github.com/EC-CUBE/ec-cube/blob/3.0.13/src/Eccube/Event/TemplateEvent.php){:target="_blank"}クラスを参照して下さい。

#### メール送信等

上記以外にも、独自のタイミングでフックポイントを設定している箇所が存在します。

(本体) Service/MailService.phpt[](https://github.com/EC-CUBE/ec-cube/blob/3.0.13/src/Eccube/Service/MailService.php#L264){:target="_blank"}

``` php
<?php
$this->app['eccube.event.dispatcher']->dispatch(EccubeEvents::MAIL_ORDER, $event);
```

(本体) Event/EccubeEvents.php[](https://github.com/EC-CUBE/ec-cube/blob/3.0.13/src/Eccube/Event/EccubeEvents.php#L592){:target="_blank"}

``` php
<?php
const MAIL_ORDER = 'mail.order';
```

event.yml[](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/event.yml#L60-L61){:target="_blank"}

``` yaml
mail.order:
    - [onSendOrderMail, NORMAL]
```
