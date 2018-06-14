---
layout: post
title:  "フィボナッチ数列の第k項をO(log n)で計算しようとしたけど難しかった話"
date:   2018-06-09 00:00:00 +0900
categories: math
---
フィボナッチ数列の計算を最適化してO(log n)を目指したのですが、そう簡単には高速化は叶いませんでした。。  

![FibonacciSeries](https://raw.githubusercontent.com/shogonir/blog/master/assets/images/paint/FibonacciSeries.png "FibonacciSeries")

# 目次
1. フィボナッチ数列
2. ビネの公式
3. 計算誤差について
4. 計算の工夫
5. 最終的な計算時間
6. サンプルコード

&nbsp;  

# 1. フィボナッチ数列
フィボナッチ数列は、イタリアの数学者レオナルド・フィボナッチの名前をとって名付けられた数列です。  
数列の決まりは、「ある項は、1つ前の項と2つ前の項の和である」というシンプルなものです。  
数式にすると下記のようになります。  

&nbsp;  

$$
F(k) = \begin{cases}
0 & ( k = 0) \\\
1 & ( k = 1) \\\
F(k - 1) + F(k - 2) & \left( k \geq 2 \right)
\end{cases}
$$

&nbsp;  

このシンプルなルールの数列なのに、自然界の至る所で見ることができる興味深い数列です。  

&nbsp;  

[フィボナッチ数 - Wikipedia](https://ja.wikipedia.org/wiki/%E3%83%95%E3%82%A3%E3%83%9C%E3%83%8A%E3%83%83%E3%83%81%E6%95%B0 "フィボナッチ数 - Wikipedia")

&nbsp;  

この記事では自然数 `k` が与えられた時に、フィボナッチ数列の第 `k` 項目を計算するプログラムを実装します。  
下記の記事で、単純な繰り返しを用いることで `O(n)` で計算できることがわかりました。  

&nbsp;  

[http://blog.shogonir.jp/entry/2018/04/29/121201 :embed:cite]

&nbsp;  

この記事では、さらに高速に計算できないか模索します。  
結論を先に言うと、 `O(n)` よりは速くなりませんでした。  
しかしその結論に至る過程が興味深かったので、記事に残しておきます。  
また、読者として対象とするのは計算時間のオーダーの概念をなんとなく理解している方とします。  

&nbsp;  

# 2. ビネの公式
今回の高速化の鍵になるのがビネの公式です。  
フィボナッチ数列の一般項は、下記のビネの公式で一発で求められることがわかっています。  

&nbsp;  

$$
F(k)=\frac{1}{\sqrt{5}}\left( \left(\frac{1 + \sqrt{5}}{2}\right)^{k} - \left(\frac{1 - \sqrt{5}}{2}\right)^{k} \right)
$$

&nbsp;  

この公式をそのままで計算すると、 `O(log n)` で計算できます。  
塁上の計算は `O(log n)` で計算できるからです。  

よしこれで `O(log n)` で計算できた！と思ったら落とし穴があります。  

&nbsp;  

# 3. 計算誤差について
n乗の計算をする際に浮動小数の計算誤差がでてしまいます。  
√5が無理数なので、このままではどうしても計算誤差はさけられません。  

ではビネの公式を使ってフィボナッチ数列を計算するのはできないのでしょうか。  
いえ、そうとも限りません。少し工夫してみましょう。  

&nbsp;  

# 4. 計算の工夫
ビネの公式を観察して見ると、√5がでてきますがこれって不思議ではないでしょうか。  
フィボナッチ数列は全ての項が自然数なのに、公式には無理数がでてくるのです。  
実際、ビネの公式の `k` にどんな自然数を代入しても、√5は計算途中で消えるのです。  

ということで、ビネの公式を少し変形して√5を消してしまいましょう。  

&nbsp;  

$$
\begin{align}
F(k) &= \frac{1}{\sqrt{5}}\left( \left(\frac{1 + \sqrt{5}}{2}\right)^{k} - \left(\frac{1 - \sqrt{5}}{2}\right)^{k} \right) \\\
   &= \frac{1}{\sqrt{5}}\left(\frac{\left(1 + \sqrt{5}\right)^{k} - \left(1 - \sqrt{5}\right)^{k}}{2^{k}}\right)
\end{align}
$$

&nbsp;  

ここで、 a = 1, b = √5 とおいて、分子の部分を計算してみます。  

&nbsp;  

$$
\left(1 + \sqrt{5}\right)^{k} - \left(1 - \sqrt{5}\right)^{k}
$$

$$
= \left(a + b\right)^{k} - \left(a - b\right)^{k}
$$

$$
= \sum_{i = 1}^{k}\left(\binom{k}{i} \cdot a^{k-i} \cdot b^{i} - \binom{k}{i} \cdot a^{k-i} \cdot \left(-b\right)^{i}\right)
$$

$$
= 2 \cdot \sum_{i = 1}^{\lceil\frac{k}{2}\rceil}\left(\binom{k}{2i - 1} \cdot a^{k - \left(2i - 1\right)} \cdot b^{2i - 1}\right)
$$

$$
= 2 \cdot b \cdot \sum_{i = 1}^{\lceil\frac{k}{2}\rceil}\left(\binom{k}{2i - 1} \cdot a^{k - \left(2i - 1\right)} \cdot b^{2\left(i - 1\right)}\right)
$$

$$
= 2 \cdot \sqrt{5} \cdot \sum_{i = 1}^{\lceil\frac{k}{2}\rceil}\left(\binom{k}{2i - 1} \cdot 5^{i - 1}\right)
$$

&nbsp;  

2項展開を行うと、奇数番目の項が打ち消しあうので、偶数項目だけが残ります。  
さて、頑張って√5を減らすことがでできました。  
ここで、もとのビネの公式に戻ってみましょう。  

&nbsp;  

$$
F(k) = \frac{1}{\sqrt{5}}\left( \left(\frac{1 + \sqrt{5}}{2}\right)^{k} - \left(\frac{1 - \sqrt{5}}{2}\right)^{k} \right)
$$

$$
= \frac{1}{\sqrt{5}}\left(\frac{\left(1 + \sqrt{5}\right)^{k} - \left(1 - \sqrt{5}\right)^{k}}{2^{k}}\right)
$$

$$
= \frac{1}{\sqrt{5}} \cdot \frac{1}{2^{k}} \cdot 2 \cdot \sqrt{5} \cdot \sum_{i = 1}^{\lceil\frac{k}{2}\rceil}\left(\binom{k}{2i - 1} \cdot 5^{i - 1}\right)
$$

$$
= \frac{1}{2^{k - 1}} \cdot \sum_{i = 1}^{\lceil\frac{k}{2}\rceil}\left(\binom{k}{2i - 1} \cdot 5^{i - 1}\right)
$$

&nbsp;  

さて、なんとか√5が消えるところまで計算を進めました。  
最後にこの式を用いた計算を行った際の計算時間を見積もります。  

&nbsp;  

# 5. 最終的な計算時間
まずシグマの部分は `O(n/2) = O(n)` 、2項係数の部分は `O(n)` 、累乗の部分は `O(log n)` です。  
なので、最終的な計算時間を単純に見積もると `O(n * (n + log n)) = O(n^2)` となります。  
変数のメモ化などの工夫をすると `O(n)` まではもっていけそうです。  

さて、ここまで頑張ってきたのですが、 計算の過程でコンピュータで計算するのに時間がかかる式に変わってしまいました。  
ということで、ここまで頑張っても `O(n)` になってしまいました。  
こんな複雑な数式を実装するぐらいだったら、普通に繰り返しで実装した方がよさそうです。  

今回、フィボナッチ数列の一般項を計算するために色々考えてきました。  
個人的に面白いと思ったのは、どう工夫しても `O(n)` を超えられなかったことです。  
数学ってよく出来ているというか、数学的な下限にぶつかっている感じがしました。  
久々に自分で考えながら数式をいじったりしたので、ここに記録しておきます。  

&nbsp;  

# 6. サンプルコード
`Python` でサンプルコードは下記です。  

&nbsp;  

```python
#! /usr/bin/env python
# -*- coding:utf-8 -*-

import math

def combination(n, k):
    result = 1
    for i in range(n - k + 1, n + 1):
        result *= i
    for i in range(1, k + 1):
        result /= i
    return result

def fibonacci(k):
    result = 0
    for i in range(1, int(math.ceil(k / 2.0) + 1)):
        result += combination(k, 2 * i - 1) * pow(5, i - 1)
    return int(result / pow(2, k - 1))

if __name__ == '__main__':
    
    for i in range(10):
        print fibonacci(i)
```