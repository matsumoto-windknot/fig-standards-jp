

このページはPSR-0 Autoloading Standardを日本語訳したものです。  
原文：https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md

[PSR-4]: http://www.php-fig.org/psr/psr-4/

<!--
Autoloading Standard
====================
-->
オートローディング規格
====================

<!--
> **Deprecated** - As of 2014-10-21 PSR-0 has been marked as deprecated. [PSR-4] is now recommended as an alternative.
-->
> **廃止予定** - 2014年10月21日現在、PSR-0は廃止予定となりました、 代わりに現在は[PSR-4][]が推奨されいています。

<!--
The following describes the mandatory requirements that must be adhered to for autoloader interoperability.
-->
次にオートローダーの相互運用性を保つために守らなければならない必須要件について説明します。

<!--
Mandatory
---------
-->
必須要件
---------

<!--
* A fully-qualified namespace and class must have the following structure `\<Vendor Name>\(<Namespace>\)*<Class Name>`
* Each namespace must have a top-level namespace ("Vendor Name").
* Each namespace can have as many sub-namespaces as it wishes.
* Each namespace separator is converted to a `DIRECTORY_SEPARATOR` when loading from the file system.
* Each `_` character in the CLASS NAME is converted to a `DIRECTORY_SEPARATOR`. The `_` character has no special meaning in the namespace.
* The fully-qualified namespace and class are suffixed with `.php` when loading from the file system.
* Alphabetic characters in vendor names, namespaces, and class names may be of any combination of lower case and upper case.
-->
* 完全修飾名前空間とクラス名は次の構造を持たなければなりません。`\<Vendor Name>\(<Namespace>\)*<Class Name>`
* どの名前空間もトップレベルに「ベンダー名」を持たなければなりません。
* どの名前空間も好きな数だけサブ名前空間を持つことが出来ます。
* 名前空間の区切り文字はファイルシステムからロードする際に`DIRECTORY_SEPARATOR`(ディレクトリの区切り文字)に変換されます。
* クラス名に含まれる`_`(アンダースコア)は、`DIRECTORY_SEPARATOR`に変換されます。名前空間に含まれる`_`は特別な意味を持ちません。
* 完全修飾名前空間とクラス名はファイルシステムからロードする際に`.php`が最後に付け加えられます。
* ベンダー名・名前空間・クラス名は、大文字・小文字の組み合わせのアルファベットが使用できます。

<!--
Examples
--------
-->
例
--------

* `\Doctrine\Common\IsolatedClassLoader` => `/path/to/project/lib/vendor/Doctrine/Common/IsolatedClassLoader.php`
* `\Symfony\Core\Request` => `/path/to/project/lib/vendor/Symfony/Core/Request.php`
* `\Zend\Acl` => `/path/to/project/lib/vendor/Zend/Acl.php`
* `\Zend\Mail\Message` => `/path/to/project/lib/vendor/Zend/Mail/Message.php`

<!--
Underscores in Namespaces and Class Names
-----------------------------------------
-->
名前空間とクラス名のアンダースコアの例
-----------------------------------------

* `\namespace\package\Class_Name` => `/path/to/project/lib/vendor/namespace/package/Class/Name.php`
* `\namespace\package_name\Class_Name` => `/path/to/project/lib/vendor/namespace/package_name/Class/Name.php`

<!--
The standards we set here should be the lowest common denominator for painless autoloader interoperability.
following these standards by utilizing this sample SplClassLoader implementation which is able to load PHP 5.3 classes.
-->
ここで定めた規格は、オートローダーの相互運用性を実現する最も簡単な方法になるはずです。
下記にPHP5.3クラスがロードできる、この規格に準拠したクラスローダーの実装サンプルを紹介します。

<!--
Example Implementation
----------------------
-->
実装例
----------------------

<!--
Below is an example function to simply demonstrate how the above proposed standards are autoloaded.
-->
下記は、上記で提案した規格のオートロード方法を示した簡単なデモ関数です。

~~~php
<?php

function autoload($className)
{
    $className = ltrim($className, '\\');
    $fileName  = '';
    $namespace = '';
    if ($lastNsPos = strrpos($className, '\\')) {
        $namespace = substr($className, 0, $lastNsPos);
        $className = substr($className, $lastNsPos + 1);
        $fileName  = str_replace('\\', DIRECTORY_SEPARATOR, $namespace) . DIRECTORY_SEPARATOR;
    }
    $fileName .= str_replace('_', DIRECTORY_SEPARATOR, $className) . '.php';

    require $fileName;
}
spl_autoload_register('autoload');
~~~

<!--
SplClassLoader Implementation
-----------------------------
-->
SplClassLoaderの実装
-----------------------------

<!--
The following gist is a sample SplClassLoader implementation that can load your classes if you follow the autoloader interoperability standards proposed above.
It is the current recommended way to load PHP 5.3 classes that follow these standards.
-->
次のgistはこのオートローディング規格に準拠したクラスがロードできるクラスローダー(SplClassLoader)の実装例です。
規格に準拠するPHP5.3のクラスをロードする現時点での推奨方法です。

* [http://gist.github.com/221634](http://gist.github.com/221634)
