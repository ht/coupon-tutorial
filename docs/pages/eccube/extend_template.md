---
title: イベントでテンプレートを拡張する
keywords:
tags: []
sidebar: mydoc_sidebar
permalink: extend_template.html
folder: eccube
---

*イベントについては[イベントで機能を拡張する](/extend_event.html)を参照して下さい。*

EC-CUBE3でテンプレートをイベントで拡張する主な方法は2つあります。

## テンプレート描画イベントで拡張する

[テンプレート描画イベント](/extend_event.html#section-5)では、twig描画前のソースに対して拡張が可能です。

### 実装方法

Event/Event.php[](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/Event/Event.php#L62){:target="_blank"}

``` php
<?php
// 追加するtwigソースを取得
$snipet = $app['twig']->getLoader()->getSource('Coupon/Resource/template/default/coupon_shopping_item.twig');

// 拡張元のtwigソースを取得
$source = $event->getSource();

// $searchは検索対象
// $replaceは置換後の文字列
$replace = $snipet.$search;

// twigソースを置換
$source = str_replace($search, $replace, $source);

// 置換したtwigソースをイベント引数にセット
$event->setSource($source);

// 置換後のtwigソースで使う変数をイベント引数にセット
$parameters['CouponOrder'] = $CouponOrder;
$event->setParameters($parameters);
```

### 特徴

- twig描画前のソースを操作でき、また描画時のパラメータの参照や追加もできるため比較的手間が少ない。
- DOM操作が使えない。例えばid属性がhogeのdiv要素の前に追加したいケースで、twigに```<div id="hoge"``` と記述されていれば置換は簡単だが、```<div class="fuga" id="hoge"``` となっている可能性も考慮すると複雑な正規表現を用いる必要がある。
- ユーザーが好きな位置に記述した[特定の文言](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/Event/Event.php#L40){:target="_blank"}を置換するような用途に向いている。

## renderイベントで拡張する

```eccube.event.render.[route].before``` イベントでは、twig描画後のソースに対して拡張が可能です。

### 実装方法

event.yml[](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/event.yml#L5-L6){:target="_blank"}

``` yaml
eccube.event.render.shopping.before:
    - [onRenderShoppingBefore, NORMAL]
```

Event.php[](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/Event.php#L120){:target="_blank"}

``` php
<?php
public function onRenderShoppingBefore(FilterResponseEvent $event)
{
    // $this->app['eccube.plugin.coupon.event.legacy']の中身は\Plugin\Coupon\Event\EventLegacyのインスタンス
    $this->app['eccube.plugin.coupon.event.legacy']->onRenderShoppingBefore($event);
}
```

Event/EventLegacy.php[](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/Event/EventLegacy.php#L58){:target="_blank"}

``` php
<?php
// レスポンスに、操作後のContentをセットする
$response->setContent($this->getHtmlShopping($response, $Order));

// イベント変数にレスポンスをセットする
$event->setResponse($response);
```

Event/EventLegacy.php[](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/Event/EventLegacy.php#L438){:target="_blank"}

``` php
<?php
private function getHtmlShopping(Response $response, Order $Order)
{
    // Crawler等でDOM操作を行ない、操作後のhtmlを返す
    return $html;
}
```

### 特徴

- 描画時のパラメータが参照できないため、DOMを解析する等して値を取得する必要がありコード量が増えがち。
- DOM操作の知識が必要。また描画後のソースが構文的に正しくないと、正常にパースできない可能性がある。
- 拡張する際に複雑な条件を指定することが可能。フォームの拡張等に向いている。
