---
title: 暗黙的インタフェースの実装
tags: [API design]
---

http://martinfowler.com/bliki/ImplicitInterfaceImplementation.html



JavaもC#も純粋なインタフェース型というものを用意している。
純粋なインタフェースは``interface Mailable``のように宣言し、
Javaの場合だと``class Customer implements Mailable``のようにして実装する。
ひとつのクラスは複数のインタフェースを実装することができる。



このモデルが考慮していないのは、
クラスには必ず''暗黙的な''インタフェース(implicit interface)があるという点である。
Customerの暗黙的なインタフェースは、Customerで宣言されたすべてのpublicなメンバである（この暗黙的なインタフェースは、今まで私が見てきたどのOO言語にも存在する）。
JavaでもC#でも、暗黙的なインタフェースをimplementすることはできない——つまり、``class ValuedCustomer implements Customer``のように書くことはできない。



暗黙的なインタフェースをimplementするとはどういうことだろうか？
基本的には、型システムにValuedCustomerクラスがCustomerのpublicなインタフェースで宣言されたすべてのメソッドをimplementしていることを知らせるが、実装の中身については考慮しないということだ。実装の中身とは、publicメソッドの中身とpublicではないメソッドとデータのことを指す。
つまり、我々が手にしているのはインタフェース継承であり、実装継承ではないのだ。



これができるなら、すべてのpublicメソッドを保ったまま、Customer をインターフェースに変更し、その上で、このインターフェースをimplementするCustomerImplクラスを作るのと、同等の効果がある。



暗黙的なインタフェースの実装は、どのようなときに便利なのだろうか？
Javaの初期、今のようにフレームワークが乱立する以前、
Vectorクラスを我々の実装に置き換えたいと思ったことがあった。
しかし、それはできなかった。
なぜなら、Vectorはクラスであり、我々はサブクラスを作ることしかできなかったからだ。
ライブラリが自由に代替できるようなインタフェースを提供していないと、
何度もこのような場面に遭遇するだろう。
暗黙的なインタフェースの実装機能がないために、我々はハマるわけだ。



最近は、特にテストのときに困るようになってきた。
スタブ化は時間のかかる作業だが、そもそもインタフェースがないとスタブ化するのは難しい。不可能な場合さえある。
テスト用の代替をサポートするためだけに、
純粋なインタフェース型を定義するということになっている。
この場合、[InterfaceImplementationPair](/InterfaceImplementationPair)を使用することが多いが、
これは我々の好みではない。
暗黙的インタフェース実装の方が明快な手法である。



では、どうして言語がこれを許していないのだろうか？
言語設計者ではないので私はよく知らないのだが、
Anders Heljsberg(訳注：Turbo Pascal、Delphi、C#などの作者)にこの点について聞いたことがある。
彼の返答は、明示的にvirtualと宣言した場合のみ、オーバーライドを許可する方が好ましいという考えと同じであるというものだった。
これはつまり、サブクラス（あるいはその実装者）がスーパークラスを破壊しないかという懸念であり、継承をどのように使うかというもっと広範囲な主題に関わってくる話だ。
ただ、夕食のときに軽く話しだけなので、詳細については話を聞けていない。



この話は動的型言語では問題にならない。
他のクラスのインタフェースを実装した場合、
必要なオブジェクトに同じメソッドを実装すればよいだけだからだ。
Javaでは、動的プロキシを使ってやる手もあるが、
暗黙的インタフェースの実装の方がより伝わりやすいだろう。




### 追記
Mike Rettigがこの手法の問題点——すべてのクライアントがpublicインタフェースだけを使っているとは限らない点——を指摘してくれた。
PaymentPlanクラスがCustomerと同じパッケージに入っている場合を考えてみよう——PaymentPlanクラスはCustomerのパッケージ可視性メソッドを呼び出すことができる。
そこでCustomerを暗黙的な実装に置き換えてしまうと、ハマる。



ここでの問題は、クラスには複数の暗黙的インタフェースがあるという点だ——アクセス制御ごとに1つのインタフェースがあることになる。
これには、同じクラスの他のインスタンスからアクセスできるという広範囲なオープンアクセスも含まれる。

## コメント

* 2006-01-29 (日) 11:43:37 keis : ★部分とそれに続く一文の訳を考えて見ました。「返答は、明示的に virtual と宣言されたメンバのみ、オーバーライドを許す方が好ましいという彼の考えと軌を一にするものだった。つきつめればこれは、サブクラスが（あるいはその実装者が）スーパークラスを破壊しやしないかという懸念であり、継承をどのように使うかというもっと広範な主題に関わってくる話だ」。Heljsbergのスタンスは、クラス設計者によるコントロールを重視する、といったところなんでしょうね。（Effective Java の著者である）Joshua Bloch的でもありますね。
* 2006-01-29 (日) 11:53:02 keis : It would be equivalent... で始まるパラグラフが抜けていたので追加しました。
* 2006-01-30 (月) 10:18:03 kdmsnr : ありがとうございます。反映しました。

