<!--- Copyright (C) 2009-2013 Typesafe Inc. <http://www.typesafe.com> -->
<!--
# Adding support for a custom format to the template engine
-->
# テンプレートエンジンに対するカスタムフォーマットのサポートの追加

<!--
The built-in template engine supports common template formats (HTML, XML, etc.) but you can easily add support for your own formats, if needed. This page summarizes the steps to follow to support a custom format.
-->
組み込みのテンプレートエンジンは一般的なテンプレートフォーマット (HTML、XML、等) をサポートしていますが、必要であれば独自のフォーマットのサポートを簡単に追加する事ができます。このページではカスタムフォーマットをサポートする為に必要なステップをまとめています。

<!--
## Overview of the the templating process
-->
## テンプレートプロセスの概要

<!--
The template engine builds its result by appending static and dynamic content parts of a template. Consider for instance the following template:
-->
テンプレートエンジンはテンプレートの静的および動的な部品を結合する事でビルドします。例えば以下のテンプレートを考えてみて下さい:

```
foo @bar baz
```

<!--
It consists in two static parts (`foo ` and ` baz`) around one dynamic part (`bar`). The template engine concatenates these parts together to build its result. Actually, in order to prevent cross-site scripting attacks, the value of `bar` can be escaped before being concatenated to the rest of the result. This escaping process is specific to each format: e.g. in the case of HTML you want to transform “<” into “&amp;lt;”.
-->
上記のテンプレートは一つの動的な部品 (`bar`) およびその周囲の二つの静的な部品 (`foo ` および ` baz`) から構成されています。テンプレートエンジンはこれらの部品を結合する事で結果をビルドします。実際には、クロスサイトスクリプティング攻撃を防ぐ為、他の部品と結合される前に `bar` の値をエスケープすることができます。このエスケーププロセスはフォーマット毎に特有となります: 例えば、 HTML の場合は「<」を「&amp;lt;」に変換したくなるでしょう。

<!--
How does the template engine know which format correspond to a template file? It looks at its extension: e.g. if it ends with `.scala.html` it associates the HTML format to the file.
-->
テンプレートエンジンはどのようにしてテンプレートファイルに対応するフォーマットを知る事が出来るのでしょうか? 拡張子を見ています: 例えば、 `.scala.html` で終わる場合は、HTML フォーマットとファイルを紐付けます。

<!--
In summary, to support your own template format you need to perform the following steps:
-->
まとめると、独自のテンプレートフォーマットをサポートするには、以下のステップを行う必要があります:

<!--
* Implement the text integration process for the format ;
* Associate a file extension to the format.
-->
* フォーマットに対するテキスト統合のプロセスを実装する ;
* ファイル拡張子をフォーマットと関連づける.

<!--
## Implement a format
-->
## フォーマットを実装する

Implement the `play.twirl.api.Format<A>` interface that has the methods `A raw(String text)` and `A escape(String text)` that will be used to integrate static and dynamic template parts, respectively.

The type parameter `A` of the format defines the result type of the template rendering, e.g. `Html` for a HTML template. This type must be a subtype of the `play.twirl.api.Appendable<A>` trait that defines how to concatenates parts together.

For convenience, Play provides a `play.twirl.api.BufferedContent<A>` abstract class that implements `play.twirl.api.Appendable<A>` using a `StringBuilder` to build its result and that implements the `play.twirl.api.Content` interface so Play knows how to serialize it as an HTTP response body.

In short, you need to write two classes: one defining the result (implementing `play.twirl.api.Appendable<A>`) and one defining the text integration process (implementing `play.twirl.api.Format<A>`). For instance, here is how the HTML format could be defined:

<!--
```java
public class Html extends BufferedContent<Html> {
  public Html(StringBuilder buffer) {
    super(buffer);
  }
  String contentType() {
    return "text/html";
  }
}

public class HtmlFormat implements Format<Html> {
  Html raw(String text: String) { … }
  Html escape(String text) { … }
  public static final HtmlFormat instance = new HtmlFormat(); // The build process needs a static reference to the format (see the next section)
}
```
-->
```java
public class Html extends BufferedContent<Html> {
  public Html(StringBuilder buffer) {
    super(buffer);
  }
  String contentType() {
    return "text/html";
  }
}

<!--
## Associate a file extension to the format
-->
## ファイル拡張子をフォーマットと関連づける

The templates are compiled into a `.scala` files by the build process just before compiling the whole application sources. The `TwirlKeys.templateFormats` key is a sbt setting of type `Map[String, String]` defining the mapping between file extensions and template formats. For instance, if you want Play to use your own HTML format implementation you have to write the following in your build file to associate the `.scala.html` files to your custom `my.HtmlFormat` format:

```scala
TwirlKeys.templateFormats += ("html" -> "my.HtmlFormat.instance")
```

Note that the right side of the arrow contains the fully qualified name of a static value of type `play.twirl.api.Format<?>`.

<!--
> **Next:** [[HTTP form submission and validation | JavaForms]]
-->
> **次:** [[HTTP フォーム送信とバリデーション | JavaForms]]
