---
title: フォームを拡張する
keywords:
tags: []
sidebar: mydoc_sidebar
permalink: extend_form.html
folder: eccube
---

EC-CUBE3でフォームを拡張する方法を紹介します。
クーポンプラグインではフォーム拡張をあまり利用していないため、商品登録画面のフォームを拡張するケースを記述します。

## フォームを拡張するタイミング

EC-CUBE3でフォームを拡張する主なタイミングは2つあります。

### Extensionで拡張する

#### 拡張を作成する

Form/Extension/ProductTypeExtension.php

``` php
<?php
class ProductTypeExtension extends AbstractTypeExtension
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        // 拡張処理
    }

    public function getExtendedType()
    {
        return 'admin_product';
    }
}
```

```AbstractTypeExtension``` [](https://github.com/symfony/symfony/blob/2.7/src/Symfony/Component/Form/AbstractTypeExtension.php){:target="_blank"}を継承したクラスを作成します。

```getExtendedType()``` [](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/Form/Type/CouponType.php#L257){:target="_blank"}メソッドで、拡張対象のフォームの名前を文字列でreturnして下さい。
商品登録画面のフォームの場合、```ProductType::getName()``` [](https://github.com/EC-CUBE/ec-cube/blob/3.0.13/src/Eccube/Form/Type/Admin/ProductType.php#L170){:target="_blank"}の戻り値を指定して下さい。

```buildForm()``` メソッドで、フォームの中身を拡張します。

#### 拡張を登録する

ServiceProvider/CouponServiceProvider.php

``` php
<?php
$app['form.type.extensions'] = $app->share($app->extend('form.type.extensions', function ($extensions) {
    $extensions[] = new ProductTypeExtension();
    return $extensions;
}));
```

### イベントで拡張する

EC-CUBE3で用意されているイベントで拡張できる場合があります。
イベントについては[イベントで機能を拡張する]({{site.baseurl}}/extend_event.html)を参照して下さい。

config.yml

``` 
event: PluginEvent
```

event.yml

``` yaml
admin.product.edit.initialize:
    - [onAdminProductEditInitialize, NORMAL]
```

PluginEvent.php

``` php
<?php
class PluginEvent
{
    public function onAdminProductEditInitialize($event)
    {
        $builder = $event->getArgument('builder');
        // 拡張処理
    }
}
```

## フィールドを追加する

``` php
<?php
$builder->add('field_name', 'text');
```

フィールドを追加しただけでは画面に表示されません。
テンプレートを拡張する方法は[イベントでテンプレートを拡張する]({{site.baseurl}}/extend_template.html)を参照して下さい。

``` php
<?php
$builder->add('plg_field_name', 'text');
```

但しフィールド名がplgで始まるフィールドは、[ほとんどの画面](https://github.com/EC-CUBE/ec-cube/blob/3.0.13/src/Eccube/Resource/template/admin/Product/product.twig#L309-L315){:target="_blank"}で自動的に挿入されます。

*という建前ですが、実際にはplで始まるフィールドが対象です。[参考URL](https://github.com/EC-CUBE/ec-cube/issues/1822){:target="_blank"}*

## フィールドを変更する

フォームの拡張では、既存のフィールドを変更することも可能です。

``` php
<?php
// 商品名フィールドを取得
$name = $builder->get('name');

// フィールドのタイプを文字列で取得
$type = $name->getType()->getName();

// フィールドのオプションを取得
$options = $name->getOptions();

// labelを変更
$options['label'] = '商品の名前';

// constraintsからNotBlankを削除
$constraints = isset($options['constraints']) ? $options['constraints'] : array();
$options['constraints'] = array_filter($constraints, function ($constraint) {
    return $constraint instanceof Assert\NotBlank;
});

// nameフィールドを上書き
$builder->add('name', $type, $options);
```
