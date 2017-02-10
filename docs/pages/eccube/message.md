---
title: メッセージを設定する
keywords:
tags: []
sidebar: mydoc_sidebar
permalink: message.html
folder: eccube
---

プラグインでメッセージを追加・編集する方法を紹介します。

## メッセージを記述する

まずメッセージの名前と文言をyaml形式で記述します。

Resource/locale/message.ja.yml[](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/Resource/locale/message.ja.yml){:target="_blank"}

``` yaml
admin.plugin.coupon.notfound: クーポンが存在しません。
```

## メッセージを登録する

記述したメッセージをアプリケーションに登録します。

ServiceProvider/CouponServiceProvider[](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/ServiceProvider/CouponServiceProvider.php#L113-L114){:target="_blank"}

``` php
<?php
$file = __DIR__.'/../Resource/locale/message.'.$app['locale'].'.yml';
$app['translator']->addResource('yaml', $file, $app['locale']);
```

なお、```Translator::addResource()``` [](https://github.com/symfony/symfony/blob/2.7/src/Symfony/Component/Translation/Translator.php#L121){:target="_blank"}の第四引数にはドメイン名を渡すことが可能です。デフォルトのドメイン名は```'messages'``` です。ドメイン名については後述します。

## メッセージを呼び出す

### PHPで呼び出す

Form/Type/CouponType.php[](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/Form/Type/CouponType.php#L181){:target="_blank"}

``` php
<?php
$form['coupon_type']->addError(new FormError($app->trans('admin.plugin.coupon.coupontype')));
```

```ApplicationTrait::trans()``` [](https://github.com/EC-CUBE/ec-cube/blob/3.0.13/src/Eccube/Application/ApplicationTrait.php#L190){:target="_blank"}の第一引数にメッセージ名を、第三引数にドメイン名を渡すと、そのドメイン名とメッセージ名に紐づくメッセージを取得できます。ドメイン名のデフォルトは```'messages'``` です。

### twigで呼び出す

Symfony2ではtransというtwigタグを提供しています。

(本体) Resource/template/default/Form/form_layout.twig[](https://github.com/EC-CUBE/ec-cube/blob/3.0.13/src/Eccube/Resource/template/default/Form/form_layout.twig#L59){:target="_blank"}

``` twig
{% raw %}<p class="errormsg text-danger">{{ error.message |trans }}</p>{% endraw %}
```

## 組み込みのバリデーションエラーメッセージを変更する

組み込みのバリデーションエラーメッセージは、```'validators'``` というドメイン名で登録されています。

(本体) Resource/locale/validator.ja.yml[](https://github.com/EC-CUBE/ec-cube/blob/3.0.13/src/Eccube/Resource/locale/validator.ja.yml){:target="_blank"}

(本体) Application.php[](https://github.com/EC-CUBE/ec-cube/blob/3.0.13/src/Eccube/Application.php#L218-L221){:target="_blank"}

``` php
<?php
$file = __DIR__.'/Resource/locale/validator.'.$app['locale'].'.yml';
if (file_exists($file)) {
    $translator->addResource('yaml', $file, $app['locale'], 'validators');
}
```

## メッセージに変数を渡す

メッセージに変数を含む場合、プレースホルダーで置き換えることが可能です。

```ApplicationTrait::trans()``` [](https://github.com/EC-CUBE/ec-cube/blob/3.0.13/src/Eccube/Application/ApplicationTrait.php#L190){:target="_blank"}の第二引数に、プレースホルダーをキーとした文字列の配列を渡します。

``` yaml
hello.%name%: "Hello %name%!"
```

```php
<?php
// Hello World!
$app->trans('hello.%name%', array('%name%' => 'World'));
```

## ブラウザによるポップアップのエラーメッセージ

{% include image.html file="browser_popup.png" %}

上図のメッセージはEC-CUBEではなくブラウザの実装によるもので、required属性が指定されたinput要素に表示されます。

フォームに下記のオプションを設定することで、required属性を付与することが可能です。

Form/Type/CouponType.php[](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/Form/Type/CouponType.php#L56){:target="_blank"}

``` php
<?php
        $builder
            ->add('coupon_cd', 'text', array(

                // requiredにtrueを指定することで、required属性を付与します
                'required' => true,

                // required属性はあくまで画面上の制御を行なうのみで、
                // 開発ツール等で属性を消すと送信できてしまいます。
                // NotBlankバリデーションを忘れないようにしましょう。
                'constraints' => array(
                    new Assert\NotBlank(),
                ),
            ))
```

なおrequired属性によるエラーメッセージはEC-CUBE側では変更できません。
