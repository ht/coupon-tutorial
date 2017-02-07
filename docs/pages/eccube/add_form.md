---
title: フォームを追加する
keywords:
tags: []
sidebar: mydoc_sidebar
permalink: add_form.html
folder: eccube
---

## フォームを作成する

クーポン編集画面のフォームを作成します。

Form/Type/CouponType.php[](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/Form/Type/CouponType.php){:target="_blank"}

``` php
<?php
class CouponType extends AbstractType
{
    public function getName()
    {
        return 'admin_plugin_coupon';
    }
    
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('coupon_cd', 'text', array(
                'label' => 'クーポンコード',
                'required' => true,
                'trim' => true,
                'constraints' => array(
                    new Assert\NotBlank(),
                    new Assert\Regex(array('pattern' => '/^[a-zA-Z0-9]+$/i')),
                ),
            ));
    }
}
```

```AbstractType``` [](https://github.com/symfony/symfony/blob/2.7/src/Symfony/Component/Form/AbstractType.php){:target="_blank"}を継承したクラスを作成します。

```getName()``` [](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/Form/Type/CouponType.php#L257){:target="_blank"}メソッドで、このフォームの名前を文字列でreturnして下さい。

```buildForm()``` [](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/Form/Type/CouponType.php#L50){:target="_blank"}メソッドで、フォームの中身を組み立てます。

フォームの組み立て方の詳細はSymfony2のドキュメントを参照して下さい。

- [フォーム](http://symfony.com/doc/2.7/forms.html){:target="_blank"}
- [バリデーション](http://symfony.com/doc/2.7/validation.html){:target="_blank"}

## フォームを登録する

作成したフォームを利用できるよう登録します。

ServiceProvider/CouponServiceProvider.php[](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/ServiceProvider/CouponServiceProvider.php#L82-L90){:target="_blank"}

``` php
<?php
// Formの登録
$app['form.types'] = $app->share($app->extend('form.types', function ($types) use ($app) {
    $types[] = new CouponType($app);
    return $types;
}));
```

## フォームを呼び出す

登録したフォームを呼び出します。

Controller/CouponController.php[](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/Controller/CouponController.php#L86){:target="_blank"}

``` php
<?php
$form = $app['form.factory']->createBuilder('admin_plugin_coupon', $Coupon)->getForm();
```

```FormFactory::createBuilder()``` [](https://github.com/symfony/symfony/blob/2.7/src/Symfony/Component/Form/FormFactory.php#L61){:target="_blank"}の第一引数にフォーム名の先述の文字列を渡すことで、```CouponType::buildForm()``` [](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/Form/Type/CouponType.php#L50){:target="_blank"}で初期化した```FormBuilderInterface```[](https://github.com/symfony/symfony/blob/2.7/src/Symfony/Component/Form/FormBuilderInterface.php){:target="_blank"} のサブクラスのインスタンスを取得できます。
