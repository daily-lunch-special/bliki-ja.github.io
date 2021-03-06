---
title: Junit新インスタンス
tags: [testing]
---

http://martinfowler.com/bliki/JunitNewInstance.html

[JUnit](http://junit.org/) testing framework のあるデザインについて、よく質問を受ける。
テストメソッドを走らせるたびに、新しいオブジェクトができる点についてだ。
blikiへ投稿するに値する内容だと思ったのでここに記す。
（
念のために言っておくが、JUnitについて何か書くからといって、
その他のテストのやり方が重要じゃないと思っているわけじゃないですから。
有益なテスト方法はたくさんあるわけで、
JUnit やその親戚（xUnit）がいくら便利だからって、
すべてを解決してくれるわけじゃない。
テストについて言及してるblogがいくつかあるから、
そちらを読んでみることをお勧めする
（
[Brett Pettichord](http://www.io.com/~wazmo/blog/), 
[Brian Marick](http://www.testing.com/cgi-bin/blog), 
[James Bach](http://blackbox.cs.fit.edu/blog/james/)
）。
あと、これも念のために言っておくが、
xUnitについて何か書くからといって、
リファクタリングやユースケース、flossing（★）が重要じゃないと言ってるわけじゃないですから。
）

下の Java の簡単な Test クラスを見て欲しい。

 import junit.framework.*;
 import java.util.*;
 
 public class Tester extends TestCase {
       public Tester(String name) {super(name);}
       private List list = new ArrayList();
       public void testFirst() {
             list.add("one");
             assertEquals(1, list.size());
       }
       public void testSecond() {
             assertEquals(0, list.size());
       }
 }

気付かないひともいるかもしれないが、このテストメソッドはどちらもパスする
——ついでに言っておくと、実行順序はどちらが先でも構わない。
なぜなら、JUnit が test メソッドごとに Tester クラスのインスタンスを作っているからだ。
list は、テストが実行されるたびに新規にインスタンス化されている。
[JUnitのバグ](http://beust.com/weblog/archives/000082.html)じゃねーの？そう思ったひともいるはずだ。
でも、バグじゃない。意図的にこうデザインされているのだ
（詳しくはKentの新刊を読むべし）。


JUnit の基本的なデザインは、Kent が Smalltalk で作ったテスティングフレームワークから来ている
（
これをフレームワークと呼ぶのは間違いかもしれない
——Kent はフレームワークとして発表したわけじゃないし。
1，2時間で出来てしまうんだから、彼はむしろ、
みんなに自分たちで作ってもらいたかったんだと思う。
そのほうが違ったものが欲しいと思ったときに、
恐がらずに変更することが出来るしね。
）
JUnitの大切な原則は何かというと、それは「分離(isolation)」である。
つまり、テストは、他のテストを失敗させるようなことをしてはならないということだ。

分離によるメリットはいくつかある。

* どんな組み合わせをしても、どの順で実行しても、テストの結果は変わらない。
* テストを書く→他のテストが失敗→なぜだー！というシチュエーションがなくなる。
* テストが失敗したときに、ゴミを残してしまって他のテストを失敗させたりはしないだろうかと心配しなくても済む。エラーのカスケードは本当のバグを隠してしまうが、これを避けることも出来る。


JUnitには分離をサポートするためのメカニズムが他にも備わっている。
setUp と tearDown メソッドである。
それぞれ、各テストメソッドの最初と最後に実行される。
上記の例で使用するには、以下のようにすればいい。


       public void setUp() {
             list = new ArrayList();
       }


tearDownを使うことはほとんどない。
setUp メソッドが、必要な初期化処理をやってくれる。

JUnitを批判するひとは、setUp と tearDown があるなら毎回新しいオブジェクトは必要ないんじゃないかと言う。
JUnitの支持者は、それはもっともだが、フィールドで初期化する人は多いし、分離させといたほうが良いと反論する。
結局のところ、コストはどうなっているのか？

コストに関する議論の中心となるのは、
JUnitのテストケースや初期化における余分なオブジェクトの生成についてである。
だがこれは、ほとんどの場合においてなんら問題にならない。
オブジェクトを生成しすぎることを心配するかもしれないが、
それには[何ら根拠がない](http://www-106.ibm.com/developerworks/java/library/j-jtp01274.html#author1)——そんな考えは時代遅れなのだ。
たしかにオブジェクトの生成が問題になることもあった。
初期のJavaも、そんな問題を抱えていた。
だが、現在のJavaではオブジェクトの生成コストはほとんどかからない。
まったく問題にならないのだ。
（
Smalltalkでは長年そんなことは問題にならなかった。
Kent と Erich がそんなことに悩まなくて済んだのはそのためだ。
）
というわけで、オブジェクトの生成についてあれこれ悩まないこと。

先ほど「ほとんど」問題ないとは言ったが、「常に」問題ないとは言ってない。
頻繁に生成したくないオブジェクトだってある。データベースコネクションがいい例だ。
こいつを共有するのは当然として、
テストメソッド間での共有だけでは、十分共有できてるとは言えない。
もっと広い範囲で共有したいと思うだろう。
手っ取り早いのは static 変数を使うことだ。
一般に、static 変数の使用は避けたほうが賢明だが、
テストなら別に構わないだろう——でもまあ、私はやらないけども。
JUnitでは、テスト用のオブジェクトを共有するための柔軟なメカニズムが用意されている。
[TestSetup](http://junit.sourceforge.net/javadoc/junit/extensions/TestSetup.html) デコレータだ。
これはテスト用にある状態をセットアップし、
テスト間で柔軟に状態を共有できるようにしてくれるものだ。
クラス内のメソッド間共有だけではなく、もっと広範囲で共有可能だ。

おそらく、TestSetup で最も問題となるのは、
TestSetup に関する情報を見つけるのが非常に困難だという点だ。
私も、ドキュメントの中に「豹に注意(訳注：beware of the leopard - 『銀河ヒッチハイクガイド』より)）」とでも書いてあるのかと思ったくらいだ。
で、やはり、豹はいた。
TestSetupを使うと分離が効かなくなる。
分離されてないと、バグの発見が厄介になる。
本当に本当に本当に本当に必要になるまで、TestSetupは使わないようにすること。
（
それでも使うっていうんだったら、
[このスレ](http://www.testdriven.com/modules/newbb/viewtopic.php?viewmode=flat&order=ASC&topic_id=1115&forum=7&move=prev&topic_time=1089140252)を読むとヒントを得られるかもしれない。
J.B. Rainsberger の新刊も役に立つだろう。
）

（
テストメソッドをクラスにしちゃえばいいじゃんと思った人もいるかもしれない。
実は、JUnitもかつてはそうだった。
インナークラスを TestCase のサブクラスにして使っていたのだ。
これは分かりやすいデザインのようだが、
テストを書くのがかなり大変だった。
というわけで、ちょっと分かりにくい [pluggable selector](http://junit.sourceforge.net/doc/cookstour/cookstour.htm) パターン を使うようになっていったわけだ。
）

2番目の反論は「直感的ではない」というものだ。
引っ張ってくるメカニズムが分かりにくいそうだ（★）。
まあ、そうだね。
Pluggable Selector パターンはあまり知られていないし、
馴染みのないパターンを使うのは心地悪いのだろう。
だが、私は JUnit のアプローチが好きだ。
きちんと分離ができて、テストを書くのが簡単なら、
実装が難解でも別に構わないと思う。

だが、これに賛同してくれない仲間もいる。
Cedric Beust の [TestNG](http://www.beust.com/testng/) は分離していない。
たぶん驚くと思うが、あの [NUnit](http://nunit.org/) も分離していないのだ。（★分離でいいのかな？）。
以下の NUnit のテストケースは失敗する。

    [TestFixture]
    public class ServerTester
    {
       private IList list = new ArrayList();
       [Test]
       public void first() {
          list.Add(1);
          Assert.AreEqual(1, list.Count);
       }
       [Test]
       public void second() {
          Assert.AreEqual(0, list.Count);
       }
    }

こういった類のフレームワークを使っているのであれば、
setupメソッドを使ってすべてのインスタンス変数を初期化しておくことを強くお奨めする。
こうしてテストを分離することが出来る。
ひいては、デバッグのやり杉で毛が抜けるなんてこともなくなるのだ。

テストケースのインスタンスは再利用したほうがいいとはちょっと考えられないが、
だからといって、再利用してるひとたちの
IQが一桁しかないとか、
財政的に圧迫しているとか、
宇宙服を着ておかしな行動をしてるんじゃねーかとか
そんなことはぜんっぜん思ってませんから（★）。
これは、デザインのトレードオフの仕方が違うだけなのである。
ソフトウェアデザインの流動性に関して、
敬意をもって賛同できないと言えるということは、
よいことだと思うのだなあ。

## comment
* 2004-08-27 (金) 11:14:12 ''[holic]]'' : TestNG や NUnit はデザインとして分離しないことを選択しているので、「分離できていない」じゃなくて「分離していない」の方が訳として適切な気がします。
* 2004-08-30 (月) 12:33:12 ''[[kdmsnr]]'' : [[Otaku, Cedric's weblog: More on JUnit and multiple instantiations](http://beust.com/weblog/archives/000175.html)
* 2005-01-12 (水) 00:19:49 ''[[babie]]'' : flossing はfloss(free libre open sourse software)化することかなぁ。文脈から「フリーソフトウェア化」というよりは日本で言う「オープン化」に近い物だと思います。
