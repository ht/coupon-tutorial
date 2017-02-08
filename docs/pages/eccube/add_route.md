---
title: 画面を追加する
keywords:
tags: []
sidebar: mydoc_sidebar
permalink: add_route.html
folder: eccube
---

## コントローラを作成する

コントローラを作成します。

Controller/CouponController.php[](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/Controller/CouponController.php){:target="_blank"}

``` php
<?php
class CouponController extends AbstractController
{
    public function edit(Application $app, Request $request, $id = null)
    {
        // 省略
    }
}
```

## ルーティングを追加する

クーポン編集画面のルーティングを追加する処理を追っていきます。

ServiceProvider/CouponServiceProvider.php[](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/ServiceProvider/CouponServiceProvider.php#L38-L71){:target="_blank"}

``` php
<?php
// 管理画面定義
$admin = $app['controllers_factory'];

// 強制SSL
if ($app['config']['force_ssl'] == Constant::ENABLED) {
    $admin->requireHttps();
}
```

ルーティングを追加するには、```$app['controllers_factory']```を用います。
(中身は```ControllerCollection``` [](https://github.com/silexphp/Silex/blob/1.3/src/Silex/ControllerCollection.php#L39){:target="_blank"}クラスのインスタンスです)

SSLを強制する場合は、```Route::requireHttps()```[](https://github.com/silexphp/Silex/blob/1.3/src/Silex/Route.php#L150){:target="_blank"}を実行して下さい。

``` php
<?php
// クーポンの編集
$admin->match('/plugin/coupon/{id}/edit', 'Plugin\Coupon\Controller\CouponController::edit')->value('id', null)->assert('id', '\d+|')->bind('plugin_coupon_edit');
```

次は実際にルーティングを追加する処理です。```ControllerCollection::match()``` [](https://github.com/silexphp/Silex/blob/1.3/src/Silex/ControllerCollection.php#L77){:target="_blank"}の第一引数にルートのパスを、第二引数に呼び出すコントローラと関数名を渡します。
{}で囲んだパラメータ名を渡すことで、パスにパラメータ名を含めるよう設定できます。

```Route::value()``` [](https://github.com/silexphp/Silex/blob/1.3/src/Silex/Route.php#L81){:target="_blank"}は、パラメータのデフォルト値を設定する関数です。

```Route::assert()``` [](https://github.com/silexphp/Silex/blob/1.3/src/Silex/Route.php#L66){:target="_blank"}は、パラメータが受け取る値を正規表現で設定する関数です。ここでは\d+(=1文字以上の数字)をセットしています。

```Controller::bind()``` [](https://github.com/silexphp/Silex/blob/1.3/src/Silex/Controller.php#L76){:target="_blank"}は、ルーティングに名前を設定する関数です。ここではplugin_coupon_editをセットしています。

これによって、```$app->url('plugin_coupon_edit', array('id' => 10)``` とすることでクーポン編集画面のURLが取得できるようになります。

``` php
<?php
$app->mount('/'.trim($app['config']['admin_route'], '/').'/', $admin);
```

最後に```Application::mount()``` [](https://github.com/silexphp/Silex/blob/1.3/src/Silex/Application.php#L533){:target="_blank"}でルートを登録します。第一引数に文字列を渡すと、登録したルートの先頭に接頭詞を付与します。

例えば```$app['config']['admin_route']``` がadminだった場合、/admin/plugin/coupon/{id}/edit へのアクセスが合致します。

EC-CUBE3では管理画面の接頭詞がadminとは限りません。管理画面のルーティングには必ず```$app['config']['admin_route']``` を設定して下さい。
