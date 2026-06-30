---
title: "OSSのデザインパターン解説シリーズ：Iteratorパターンの活用と悪いコード例"
emoji: "🐈"
type: "tech"
topics:
  - "java"
  - "配列"
  - "デザインパターン"
  - "list"
published: true
published_at: "2024-05-23 08:58"
---

# 1. はじめに
このシリーズでは、オープンソースソフトウェア（OSS）のソースコードを通じて、GoF（Gang of Four）デザインパターンの活用方法を解説します。
特に悪い例と良い例を見ることで、デザインパターンのメリットを考えていきます。

なお、OSSやデザインパターンの詳しい解説は書きません。
最小限のコード例と解説を書いていきます。

# 2. Iteratorパターンとは
Iteratorパターンとは、「"ものの集まり"の要素を全てたどること」を抽象化することです。

Javaの場合、"ものの集まり"に当たるクラスは以下が挙げられます：
* 配列
* ArrayList
* LinkedList
* HashSet
* HashMapのkeySet()やvalues()メソッド(の戻り値)

もちろん、データ構造の数だけ他にも"ものの集まり"が存在します。
Iteratorパターンでは、このデータ構造を気にせずに「要素をたどる」方法を提供してくれるパターンです。

# 3. 悪いコードの例
以下は、Iteratorパターンを使用せずに書かれたJavaのコードの例です。

```java
//配列の場合
String[] strs={"one", "two", "three"};
for(int i=0; i < strs.length; i++){
  System.out.println(strs[i]);
}
//Listの場合
List<String> list=new LinkedList(List.of("four", "five", "six"));
for(int i=0; i < list.size(); i++){
  System.out.println(list.get(i));
}
//Setの場合
Set<String> set=new HashSet(List.of("seven", "eight", "nine"));
String[] strSet=set.toArray(new String[0]);
for(int i=0; i < strSet.length; i++){
  System.out.println(strSet[i]);
}
```

この悪いコード例では、データ構造に依存してfor文を書き換える必要があります。
やりたい事は同じなのに、やり方が異なるのは少し不便ですね。

特に、Setは一度配列に変換しないと値を取り出すことができません。

# 4. デザインパターンの適用
Iteratorパターンは、Javaや他のOSSプロジェクトで広く活用されています。

以下は、Iteratorパターンの実装例です。（実装というより利用者側のコード）
```java
//配列の場合
String[] strs={"one", "two", "three"};
for(String str: strs){
  System.out.println(str);
}
//Listの場合
List<String> list=new LinkedList(List.of("four", "five", "six"));
for(String str: list){
  System.out.println(str);
}
//Setの場合
Set<String> set=new HashSet(List.of("seven", "eight", "nine"));
for(String str: set){
  System.out.println(str);
}
```

この例では、配列・List・Setに依らず、全て同じ方法で値を取り出すことができています。

拡張for構文を使用していますが、裏では共通の処理としてIteratorパターンが使われています。

# 5. 改善されたコードの比較
Iteratorパターンを適用することで、「"ものの集まり"から値を取り出す」操作を共通の処理で行うことができるようになりました。

これは利用者側からとても便利ですね。

# 6. まとめ
今回はIteratorパターンを利用する/しない場合のコードを基に、Iteratorパターンのメリットを考えました。
このデザインパターンは「利用者が使いやすくなるパターン」と考えられますね。

# 7. 参考文献
1. [Java言語で学ぶデザインパターン入門第3版](https://www.amazon.co.jp/Java%E8%A8%80%E8%AA%9E%E3%81%A7%E5%AD%A6%E3%81%B6%E3%83%87%E3%82%B6%E3%82%A4%E3%83%B3%E3%83%91%E3%82%BF%E3%83%BC%E3%83%B3%E5%85%A5%E9%96%80%E7%AC%AC3%E7%89%88-%E7%B5%90%E5%9F%8E-%E6%B5%A9/dp/4815609802/ref=pd_sbs_d_sccl_2_1/357-1375368-9621130?pd_rd_w=dVwck&content-id=amzn1.sym.e146c92a-7981-4c58-8351-e2a023395915&pf_rd_p=e146c92a-7981-4c58-8351-e2a023395915&pf_rd_r=3XCHR33Q12S3QJP842HK&pd_rd_wg=dqYdG&pd_rd_r=47c9746d-af80-49b6-98c9-5c294a0b3540&pd_rd_i=4815609802&psc=1)　結城 浩　SBクリエイティブ
2. [Oracle Help Center](https://docs.oracle.com/javase/jp/8/docs/api/java/util/List.html)
3. [paiza.io](https://paiza.io/projects/4pShyyhUcPBA_wZiFMwaxA?language=java)　※コード例の実行環境