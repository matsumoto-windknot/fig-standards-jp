[RFC 2119]: http://tools.ietf.org/html/rfc2119

このページはPSR-6 Caching Interfaceを日本語訳したものです。  
原文：https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-6-cache.md

# キャッシングインターフェース

キャッシングはどのプロジェクトでもパフォーマンスを改善するための通常の手段であり、キャッシングライブラリは多くのフレームワークやライブラリの最も一般的な機能の一つになっています。
これは、多くのライブラリが様々なレベルの機能を備えた独自のキャッシングライブラリを走らせる状況につながっています。
そしてこれらの違いは、開発者が必要な機能を提供しているかどうかにかかわらず、複数のシステムを学ばなければならない原因になっています。
加えて、キャッシングライブラリの開発者は、限られた数のフレムーワークををサポートするか、多くのアダプタクラスを作成するかの選択に直面しています。

キャッシュシステムのための共通のインターフェースはこれらの問題を解決します。
想定通りの方法でキャッシュシステムが動作することをライブラリやフレームワークの開発者に対して約束し、キャッシュシステムの開発者は様々なアダプタではなく単一のインターフェースを実装するだけですみます。

この文書に記載されているキーワード「しなければならない（MUST）」、「してはならない（MUST NOT）」、「必須（REQUIRED）」、「するものとする（SHALL）」、「しないものとする（SHALL NOT）」、「すべきである（SHOULD）」、「すべきでない（SHOULD NOT）」、「推奨する（RECOMMENDED）」、「することができる（MAY）」、「任意（OPTIONAL）」は、[RFC 2119][]に記載されている内容で解釈されます。


## 目的

このPSRの目的は、開発者がカスタム開発を必要とせずに、既存のフレームワークやシステムに統合可能なキャッシュライブラリを作成できるようにすることです。

## 定義

*    **呼出ライブラリ** - 実際にキャッシュサービスが必要なライブラリまたはコードのこと。
  このライブラリはこの規約のインターフェースを実装するキャッシングサービスを利用しますが、キャッシングサービスの実装は把握しません。

*    **実装ライブラリ** - 呼出ライブラリに対してキャッシングサービスを提供するために、ここでの規約を実装する役割を担うライブラリ。
  実装ライブラリは`Cache\CacheItemPoolInterface`と`Cache\CacheItemInterface`を実装したクラスを提供しなければないけません（MUST）。
  実装ライブラリは下記に記載されている秒単位の最小限のTTLの機能をサポートしなければいけません（MUST）。

*    **TTL** - 保存されて失効されたとみなされるまでのアイテムの存続期間(TTL = Time To Live)。
  通常、秒単位の時間を表す整数もしくはDateIntervalオブジェクトで定義されます。

*    **有効期限** - 実際にアイテムが無効になる時刻。
  コレは一般的にはオブジェクトが保尊された時からTTLを追加して計算されますが、DateTimeオブジェクトで明示的に設定することも出来ます。
  1:30:00に保存された300秒のTTLを持つアイテムの有効期限は1:35:00になります。
  実装ライブラリは要求された有効期限よりも前にアイテムを失効させることができますが（MAY）、有効期限に達すると失効させなければなりません（MUST）。
  呼出ライブラリがアイテムの保存を要求しても有効期限の指定がなかったり、有効期限もしくはTTLにnullが設定されていた場合、実装ライブラリは設定されたデフォルトの期間を使用することが出来ます（MAY）。
  デフォルトの期間が設定されていなかった場合、実装ライブラリは要求されたキャッシュが永続する、または基本の実装がサポートするまでの期間と解釈しなければいけません（MUST）。

*    **キー** - キャッシュされたアイテムを一意に識別する一文字以上の文字列。
  実装ライブラリは`A-Z`、`a-z`、`0-9`、`_`、`.`で成り立つ任意の順番のUTF-8エンコードの最大64文字のキーをサポートしなければいけません（MUST）。
  実装ライブラリは追加の文字やエンコーディングまたはより長い長さをサポートすることができますが（MAY）、少なくともその最低限はサポートしていなければいけません。
  実装ライブラリは自身の必要に応じてキー文字列をエスケープする必要がありますが、もとの変更されていない文字列を返すことが出来なければいけません（MUST）。
  次の文字は将来的な拡張で予約されているため、実装ライブラリはサポートしてはいけません（MUST NOT）。`{}()/\@:`

*    **キャッシュヒット** - 呼出ライブラリのキーの要求に対して、キーに対応する値が存在し、失効しておらず、他の理由で値が無効になっていない場合、キャッシュヒットといいます。
  呼出ライブラリはすべてのget()メソッドの呼び出しをisHit()メソッドで確認するべきです（SHOULD）。

*    **キャッシュミス** - キャッシュミスはキャッシュヒットの逆のことを指します。
  呼出ライブラリのキーの要求に対して、対応する値が見つからないか、見つかったが失効しているか、他の理由で値が無効になっている場合、キャッシュミスといいます。
  期限切れの値は常にキャッシュミスとみなさなければなりません（MUST）。

*    **遅延キャッシュセーブ** - 遅延キャッシュセーブはキャッシュアイテムがプールによってすぐに保存されない状態を指します。
  プールオブジェクトは一部のストレージエンジンでサポートしているバルクセット（一括設定）の利点を利用するために、キャッシュアイテムの保存を遅らせることが出来ます（MAY）。
  プールはどのような遅延したキャッシュアイテムも確実に失うことなく最終的に保存しなければならず（MUST）、呼出ライブラリが保存を要求する前に保存することが出来ます（MAY）。
  呼出ライブラリがcommit()メソッドを呼び出したときは、すべての未処理の遅延されたアイテムは保存されなければいけません（MUST）。
  実装ライブラリは遅延アイテムを保存するタイミングを決定するのに、オブジェクトのデストラクタ、save()メソッドでの保存、タイムアウト・時間上限チェック等の適切な、いかなるロジックを使用することができる（MAY）。
  遅延キャッシュに対する要求があった場合、まだ保存されてないアイテムを返さなければいけません（MUST）。

## データ

実装ライブラリはシリアライズされたすべてのPHPのデータタイプをサポートしなければいけません（MUST）。

*    **Strings（文字列）** - 任意のサイズのPHP互換のエンコーディングの文字列。
*    **Integers（整数）** - 最大64ビットのPHPがサポートする任意のサイズのすべての整数。
*    **Floats（浮動小数点値）** - すべての符号付き浮動小数点値。
*    **Boolean（真偽値）** - TRUEもしくはFALSE。
*    **Null（ヌル）** - 実際のNULL値。
*    **Arrays（配列）** - 添字配列、連想配列、任意の深さの多次元配列.
*    **Object（オブジェクト）** - `$o == unserialize(serialize($o))`のように、損失なくシリアライズとアンシリアライズをサポートするオブジェクト。 
  オブジェクトはシリアライズ可能な`__sleep()`か`__wakeup()`マジックメソッドもしくは同様の言語機能を、必要に応じて利用することが出来ます。

実装ライブラリは渡されたデータを渡された通り正確に返さなければいけません（MUST）。
それには可変型も含まれます。つまり (int) 5 で保存された場合に (string) 5 を返すのはエラーになります。
実装ライブラリは、PHPの内部的なserialize()/unserialize()の機能を利用することが出来ますが（MAY）、必須ではありません。
それらの互換性は、単にオブジェクト値の基準として使用されます。

何かしらの理由で、保存された値を正確に返すことが出来ない場合、実装ライブラリは破損したデータではなく、キャッシュミスを返さなければなりません（MUST）。

## 主な概念

### プール

プールはキャッシュシステム内でアイテムを収集することに相当します。すべてのアイテムの論理的な保存場所です。
すべてのキャッシュ可能なアイテムはプールからアイテムオブジェクトとして取得され、すべてのキャッシュオブジェクト領域へのやり取りは、プールを通じで発生します。

### アイテム

アイテムはプール内の単一のキーと値のペアのことを指します。
キーはアイテムの一次固有識別子であり、不変でなければいけません（MUST）。
値はいつでも変更することが出来ます（MAY）。

## エラー処理

キャッシュはアプリケーションのフォーマンスの重要な部分であることはよくありますが、アプリケーションの機能にとって重要な部分であるべきではありません。
従って、キャッシュシステムのエラーで、アプリケーションが機能停止になるべきではありません（SHOULD NOT）。
そのため、実装ライブラリはインターフェースで定義された例外以外をスローしてはならず（MUST NOT）、基幹のデータストアでトリガされたエラーや例外を捕らえるべきで（SHOULD）、それらを放置してはいけません。

実装ライブラリはそれらのエラーの記録するか、適切に管理者に報告するべきです（SHOULD）。

呼び出しライブラリが1つ以上のアイテムの削除を要求してきた場合や、プールをクリアする場合、要求されたキーが存在しなかったとしてもエラー状態とみなしてはいけません（MUST NOT）。
結果同じ状態になるため（キーは存在しない、プールは空である状態）、エラー状態ではありません。

## インターフェース

### CacheItemInterface

CacheItemInterfaceはキャッシュシステム内のアイテムを定義します。
各アイテムオブジェクトは特定するキーと関連付けられてなければならず（MUST）、それは実装システムに従って設定可能で、通常はCache\CacheItemPoolInterfaceによって渡されます。

Cache\CacheItemInterfaceオブジェクトはキャッシュアイテムの格納と取得をカプセル化します。
各Cache\CacheItemInterfaceは、Cache\CacheItemPoolInterfaceオブジェクトによって生成されます。Cache\CacheItemPoolInterfaceオブジェクトは、必要な設定を行い、オブジェクトを一意なキーに関連付ける役割を担います。
Cache\CacheItemInterfaceオブジェクトは、このドキュメントの「データ」セクションで定義されているPHPの値を格納し、取得できなければいけません（MUST）。

呼び出しライブラリはアイテムオブジェクト自体をインスタンス化してはいけません（MUST NOT）。
プールオブジェクトからgetItem()メソッドを経由してのみ取得することが出来ます。
呼出ライブラリは、ある実装ライブラリによって生成されたアイテムが他の実装ライブラリのプールと互換性があると思うべきではありません（SHOULD NOT）。

~~~php
<?php

namespace Psr\Cache;

/**
 * CacheItemInterfaceはキャッシュ内のオブジェクトとやり取りするためのインターフェースを定義します。
 */
interface CacheItemInterface
{
    /**
     * 現在のキャッシュアイテムのキーを返します。
     *
     * キーは実装ライブラリによってロードれされますが、上位の呼び出し元の要求に応じて使用できる
     * ようにするべきです。
     *
     * @return string
     *   このキャッシュのキー文字列
     */
    public function getKey();

    /**
     * このオブジェクトのキーに関連付けられたキャッシュアイテムの値を取得する。
     *
     * 返される値は、もともとset()によって格納された値と同一でなければなりません。
     *
     * isHit()がfalseになるとき、このメソッドはnullを返さなければいけません。
     * nullは正当なキャッシュの値なので、「null値が見つかりました」か「値が見つかりませんでした」
     * を区別するためにisHit()メソッドを使用するべきです（SHOULD）。
     *
     * @return mixed
     *   このアイテムのキーに対応する値、もしくは見つからなかった場合のnull
     */
    public function get();

    /**
     * キャッシュアイテムを検索してキャッシュヒットしたかどうかを確認します。
     *
     * 注意：isHit()の呼出と、get()の呼出が競合してはいけません（MUST NOT）。
     *
     * @return bool
     *   要求に対してキャッシュヒットした場合はtrue、それ以外はfalse
     */
    public function isHit();

    /**
     * このキャッシュアイテムの値を設定します。
     *
     * 引数の$valueはPHPによってシリアライズできるものですが、シリアライズの方法は
     * 実装ライブラリに委ねられます。
     *
     * @param mixed
     *   $value シリアライズ可能な格納する値
     *
     * @return static
     *   呼び出されたオブジェクト(このアイテムオブジェクト)
     */
    public function set($value);

    /**
     * このキャッシュの有効期限を設定します。
     *
     * @param \DateTimeInterface|null $expiration
     *   この時刻が過ぎるとアイテムは期限切れと見なさらなければなりません（MUST）。
     *   nullが明示的に渡された場合、デフォルト値を使用することが出ます(MAY)。
     *   何も設定されていいない場合は、値が永続的に、または実装が許す限り格納するべきです（SHOULD）。
     *
     * @return static
     *   呼び出されたオブジェクト(このアイテムオブジェクト)
     */
    public function expiresAt($expiration);

    /**
     * このキャッシュの有効期限を設定します。
     *
     * @param int|\DateInterval|null $time
     *   現在からこの期間が経過するアイテムは期限切れと見なさらなければなりません（MUST）。
     *   整数の場合は有効期限までの秒数として解釈されます。
     *   nullが明示的に渡された場合、デフォルト値を使用することが出ます(MAY)。
     *   何も設定されていいない場合は、値が永続的に、または実装が許す限り格納するべきです（SHOULD）。
     *
     * @return static
     *   呼び出されたオブジェクト(このアイテムオブジェクト)
     */
    public function expiresAfter($time);

}
~~~

### CacheItemPoolInterface

Cache\CacheItemPoolInterfaceの主な用途は、呼出ライブラリのキーを受け付けて関連するCache\CacheItemInterfaceオブジェクトを返すことです。
また、キャッシュコレクション全体とやり取りすることが主な目的です。
プールのすべての構成と初期化は、実装ライブラリが担います。

~~~php
<?php

namespace Psr\Cache;

/**
 * CacheItemPoolInterfaceはCacheItemInterfaceオブジェクトを生成します。
 */
interface CacheItemPoolInterface
{
    /**
     * 指定されたキーに対応するキャッシュアイテムを返します。
     *
     * このメソッドはキャッシュミスが発生してもCacheItemInterfaceを返さなければなりません。
     * nullを返してはいけません（MUST NOT）。
     *
     * @param string $key 
     *   対応するキャッシュアイテムを返すキー
     * 
     * @throws InvalidArgumentException
     *   $keyが不正な値だった場合、\Psr\Cache\InvalidArgumentExceptionをスローしなければ
     *   いけません（MUST）。
     *
     * @return CacheItemInterface
     *   対応するキャッシュアイテム。
     */
    public function getItem($key);

    /**
     * キャッシュアイテムセットのトラバーサブルを返します。
     *
     * @param string[] $keys
     *   取得するアイテムのキーを持った添字配列
     *
     * @throws InvalidArgumentException
     *   $keyに含まれるいずれかのキーが不正な値だった場合、\Psr\Cache\InvalidArgumentException
     *   をスローしなければいけません（MUST）。
     *
     * @return array|\Traversable
     *   それぞれのアイテムのキャシュキーをキーにしたキャッシュアイテムのトラバーサブル。
     *   キーが見つからない場合でも、各キーのキャッシュアイテムが返されます。
     *   キーが設定されていない場合は、空のトラバーサブルを返す必要があります。
     */
    public function getItems(array $keys = array());

    /**
     * キャッシュに指定されたキャッシュアイテムが含まれているか確認します。
     *
     * 注意：このメソッドはパフォーマンスの理由からキャッシュされた値の取得を避けることができます（MAY）。
     * しかしそれははCacheItemInterface::get()と競合する結果になるかもしれません。
     * このような状況を回避するためには、代わりにCacheItemInterface::isHit()を使用します。
     *
     * @param string $key
     *   存在を確認するためのキー
     *
     * @throws InvalidArgumentException
     *   $keyが不正な値だった場合、\Psr\Cache\InvalidArgumentExceptionをスローしなければ
     *   いけません（MUST）。
     *
     * @return bool
     *   キャッシュからアイテムが見つかった場合はtrue、それ以外はfalse
     */
    public function hasItem($key);

    /**
     * プールから、すべてのアイテムを削除します。
     *
     * @return bool
     *   プールのクリアに成功した場合はtrue、エラーが有った場合はfalse
     */
    public function clear();

    /**
     * アイテムをプールから削除します。
     *
     * @param string $key
     *   削除するためのキー
     *
     * @throws InvalidArgumentException
     *   $keyが不正な値だった場合、\Psr\Cache\InvalidArgumentExceptionをスローしなければ
     *   いけません（MUST）。
     *
     * @return bool
     *   アイテムの削除に成功した場合はtrue、エラーが発生した場合false
     */
    public function deleteItem($key);

    /**
     * プールから複数のアイテムを削除します。
     *
     * @param string[] $keys
     *   プールからアイテムを削除するためのキーを持った配列
     *
     * @throws InvalidArgumentException
     *   $keyに含まれるいずれかのキーが不正な値だった場合、\Psr\Cache\InvalidArgumentException
     *   をスローしなければいけません（MUST）。
     *
     * @return bool
     *   アイテムが正常に削除された場合はtrue、エラーが発生した場合はfalse
     */
    public function deleteItems(array $keys);

    /**
     * すぐにキャッシュアイテムを保存します。
     *
     * @param CacheItemInterface $item
     *   保存するキャッシュアイテム
     *
     * @return bool
     *   正常に保存された場合はtrue、エラーが発生した場合はfalse
     */
    public function save(CacheItemInterface $item);

    /**
     * 後で保存するキャッシュアイテムを設定します。
     *
     * @param CacheItemInterface $item
     *   保存するキャッシュアイテム
     *
     * @return bool
     *   アイテムがキュー出来なかった場合、またはコミットが試行され失敗した場合はfalse、
     *   それ以外はtrue
     */
    public function saveDeferred(CacheItemInterface $item);

    /**
     * 遅延キャッシュアイテムを保存する
     *
     * @return bool
     *   すべての未保存のアイテムの保存に成功した場合、または未保存のデータがなかった場合はtrue、
     *   それ以外の場合はfalse
     */
    public function commit();
}
~~~

### CacheException

この例外のインターフェースは無効な資格情報の提供や、キャッシュサーバー接続のような*キャッシュ設定*を含む重大なエラーが発生場合に使用されます。

実装ライブラリによってスローされる例外は、このインターフェースを実装しなければいけません（MUST）。

~~~php
<?php

namespace Psr\Cache;

/**
 * 実装ライブラリによって投げられるすべての例外のインターフェース
 */
interface CacheException
{
}
~~~

### InvalidArgumentException

~~~php
<?php

namespace Psr\Cache;

/**
 * 不正な引数のための例外のインターフェース
 *
 * メソッドに不正な引数が渡されると、Psr\Cache\InvalidArgumentExceptionを実装した例外を
 * 投げなければいけません。
 */
interface InvalidArgumentException extends CacheException
{
}
~~~