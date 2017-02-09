---
title: バリデーションを設定する
keywords:
tags: []
sidebar: mydoc_sidebar
permalink: validation.html
folder: eccube
---

EC-CUBE3でフォームの入力チェックを行なう方法を紹介します。

バリデーションには、組み込みのバリデーションとカスタムバリデーションの2種類があります。

## 組み込みのバリデーション

### 設定方法

組み込みのバリデーションは、Symfony2で用意されています。

```FormBuilder::add()``` [](https://github.com/symfony/symfony/blob/2.7/src/Symfony/Component/Form/FormBuilder.php#L59){:target="_blank"}の第三引数の配列に、constraintsというキーでバリデーションのインスタンスの配列を渡します。

Form/Type/CouponType.php[](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/Form/Type/CouponType.php#L58-L61){:target="_blank"}

``` php
<?php
use Symfony\Component\Validator\Constraints as Assert;
// 中略
$builder
    ->add('coupon_cd', 'text', array(
        'constraints' => array(
            new Assert\NotBlank(),
            new Assert\Regex(array('pattern' => '/^[a-zA-Z0-9]+$/i')),
        ),
    ))
```

### バリデーションの種類

組み込みバリデーションは[Symfony2のドキュメント](https://symfony.com/doc/2.7/reference/constraints.html){:target="_blank"}で確認できます。

バリデーションのコンストラクタに配列を渡して、チェック基準やエラーメッセージをカスタマイズできます。

ここで全てを紹介することはしませんが、例えばLength[](https://symfony.com/doc/2.7/reference/constraints/Length.html){:target="_blank"}では下記の設定が可能です。

- ```min``` 最小長
- ```max``` 最大長
- ```charset``` 比較に用いる文字コード
- ```minMessage``` 文字列長が最小長に達しなかった時のエラーメッセージ
- ```maxMessage``` 文字列長が最大長よりも長かった時のエラーメッセージ

## カスタムバリデーション

組み込みのバリデーションだけでは実現できない、複雑なバリデーションも設定可能です。

Form/Type/CouponType.php[](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/Form/Type/CouponType.php#L177-L182){:target="_blank"}

``` php
<?php
$builder

    // Submitの後のイベント
    ->addEventListener(FormEvents::POST_SUBMIT, function (FormEvent $event) use ($app) {
    
        // フォームを取得
        $form = $event->getForm();
        // フォームの値を取得
        $data = $form->getData();
        
        // CoupontDetailsの配列サイズが0で、かつcoupon_typeが3以外なら
        if (count($data['CouponDetails']) == 0 && $data['coupon_type'] != 3) {
        
            // coupon_typeにバリデーションエラーを追加
            $form['coupon_type']->addError(new FormError($app->trans('admin.plugin.coupon.coupontype')));
        }
```

```FormConfigBuilder::addEventListener()``` [](https://github.com/symfony/symfony/blob/2.7/src/Symfony/Component/Form/FormConfigBuilder.php#L208){:target="_blank"}で、フォームイベントに処理を差し込んでバリデーションを行ないます。

第一引数に渡す値によって、タイミングが変わります。バリデーションは値が送信された後に行なうものですから、```FormEvents::POST_SUBMIT``` [](https://github.com/symfony/symfony/blob/2.7/src/Symfony/Component/Form/FormEvents.php#L59){:target="_blank"}を指定します。

POST_SUBMIT以外にも介入できるタイミングがあります。詳しくは[Symfony2のドキュメント](https://symfony.com/doc/2.7/form/events.html){:target="_blank"}を参照して下さい。

*なお、メッセージの設定は[メッセージを設定する](message.html)にて解説します。*

## 入力チェックを行なう

設定したバリデーションで入力チェックを行なうには、```Form::isValid()``` [](https://github.com/symfony/symfony/blob/2.7/src/Symfony/Component/Form/Form.php#L759){:target="_blank"}を呼び出して下さい。

Controller/CouponController.php[](https://github.com/izayoi256/coupon-tutorial/blob/2.0.0/Controller/CouponController.php#L99){:target="_blank"}

``` php
<?php
if ($form->isSubmitted() && $form->isValid()) {
    // 中略
}
```
