Chapter5 分割表の解析
================

-   [クロス集計表を作る](#クロス集計表を作る)
-   [クロス集計表を検定する](#クロス集計表を検定する)
-   [おまけ：クロス集計表はエクセルからコピペできるよ(非推奨)](#おまけクロス集計表はエクセルからコピペできるよ非推奨)

**Chapter5と書いておきながら、この教材は教科書5章とは内容の関連が薄いので、注意されたい。**

クロス集計表を作る
------------------

分割表とは、クロス集計表のことである。以上。

タイタニック号の乗客のデータセットを使って、いくつかクロス集計表を見てみよう。タイタニックのデータセットは、Rにプリセットされている。データは、Age(子供か大人か)、Survived(生き残ったか、否か)、Class(船室の等級、船員での属性分け)、Sex(性別)の4つの変数からなる。以下のコードで、データを味わってみよう。

``` r
Titanic
```

    ## , , Age = Child, Survived = No
    ## 
    ##       Sex
    ## Class  Male Female
    ##   1st     0      0
    ##   2nd     0      0
    ##   3rd    35     17
    ##   Crew    0      0
    ## 
    ## , , Age = Adult, Survived = No
    ## 
    ##       Sex
    ## Class  Male Female
    ##   1st   118      4
    ##   2nd   154     13
    ##   3rd   387     89
    ##   Crew  670      3
    ## 
    ## , , Age = Child, Survived = Yes
    ## 
    ##       Sex
    ## Class  Male Female
    ##   1st     5      1
    ##   2nd    11     13
    ##   3rd    13     14
    ##   Crew    0      0
    ## 
    ## , , Age = Adult, Survived = Yes
    ## 
    ##       Sex
    ## Class  Male Female
    ##   1st    57    140
    ##   2nd    14     80
    ##   3rd    75     76
    ##   Crew  192     20

4個のクロス集計表が表れた。少し扱いにくい形なので、データの持ち方を変えてみる。以下のコードを実行すると、データが扱いやすい形になる。

参考：<https://qiita.com/1000gou/items/a8677728e432ea734124>

``` r
# epitoolsパッケージをインストールする。初回のみ、このコードを実行してください
# install.packages(epitools)

# epitoolsパッケージのexpand.table関数でデータの形を変え、data_titanicに格納する
data_titanic <- epitools::expand.table(Titanic)

# data_titanicの冒頭10個のデータを見てみる。乗客1人が、データの1行になっていることがわかる
head(data_titanic, 10)
```

    ##    Class  Sex   Age Survived
    ## 1    1st Male Child      Yes
    ## 2    1st Male Child      Yes
    ## 3    1st Male Child      Yes
    ## 4    1st Male Child      Yes
    ## 5    1st Male Child      Yes
    ## 6    1st Male Adult       No
    ## 7    1st Male Adult       No
    ## 8    1st Male Adult       No
    ## 9    1st Male Adult       No
    ## 10   1st Male Adult       No

``` r
# 行の数=データの数、を見てみる。2201人がタイタニック号に乗り合わせていたことが分かる。
nrow(data_titanic)
```

    ## [1] 2201

``` r
# 列の数=変数の数、を見てみる。もちろん、冒頭で説明した4つしかない。
ncol(data_titanic)
```

    ## [1] 4

さて、Rにはデータから分割表を作る便利なtable関数がある。この関数で、クロス集計表を作ってみよう。data\_titanicから好みの変数を$で取り出し(ドルマークについては、3章3.2の最後に紹介した)、table関数で作成する。まずは、性別と年齢でクロス集計表を作ってみる。

``` r
table(data_titanic$Sex, data_titanic$Age)
```

    ##         
    ##          Child Adult
    ##   Male      64  1667
    ##   Female    45   425

割合を求めてみよう。prop.table関数に、table関数で作ったクロス集計表をインプットすれば、割合にしてくれる。乗客の76%が大人の男性、19%が大人の女性であることが分かる。

``` r
prop.table(table(data_titanic$Sex, data_titanic$Age))
```

    ##         
    ##               Child      Adult
    ##   Male   0.02907769 0.75738301
    ##   Female 0.02044525 0.19309405

prop.table関数のmarginオプション(1または2を指定する)を使えば、行方向または列方向の割合に直すこともできる。

例えば、女性のうち、9.5%は子供だったことが分かる。

``` r
prop.table(table(data_titanic$Sex, data_titanic$Age), margin = 1)
```

    ##         
    ##               Child      Adult
    ##   Male   0.03697285 0.96302715
    ##   Female 0.09574468 0.90425532

子供でみれば、6割が男の子、4割が女の子であった。

``` r
prop.table(table(data_titanic$Sex, data_titanic$Age), margin = 2)
```

    ##         
    ##              Child     Adult
    ##   Male   0.5871560 0.7968451
    ##   Female 0.4128440 0.2031549

prop.table関数と同様の使い方で、各行・各列の合計をクロス集計に追加してくれる、addmargins関数もある。

``` r
addmargins(table(data_titanic$Sex, data_titanic$Age))
```

    ##         
    ##          Child Adult  Sum
    ##   Male      64  1667 1731
    ##   Female    45   425  470
    ##   Sum      109  2092 2201

addmagins関数も、marginオプションにより、合計を計算する方向(行方向・列方向)を制御できる(何も指定しなければ、全ての方向に合計を計算する)

``` r
addmargins(table(data_titanic$Sex, data_titanic$Age), margin = 1)
```

    ##         
    ##          Child Adult
    ##   Male      64  1667
    ##   Female    45   425
    ##   Sum      109  2092

``` r
addmargins(table(data_titanic$Sex, data_titanic$Age), margin = 2)
```

    ##         
    ##          Child Adult  Sum
    ##   Male      64  1667 1731
    ##   Female    45   425  470

クロス集計表を検定する
----------------------

クロス表を検定することができる。カイ2乗検定(chi-squred test)を使う例が多く、この検定は名前の通り、カイ2乗分布という確率分布を使った検定である。カイ2乗分布の説明(教科書2章2.7)と、クロス集計表の検定にカイ2乗分布を使う理由の説明(教科書5章5.3)は、教科書に譲る。詳しく知りたい方は、お知らせください。

さて、「クロス集計表を検定する」とはどのようなことだろうか?先ほど、タイタニックのデータで、性別と年齢のクロス集計表をつくった。

``` r
table(data_titanic$Sex, data_titanic$Age)
```

    ##         
    ##          Child Adult
    ##   Male      64  1667
    ##   Female    45   425

``` r
prop.table(table(data_titanic$Sex, data_titanic$Age), margin = 2)
```

    ##         
    ##              Child     Adult
    ##   Male   0.5871560 0.7968451
    ##   Female 0.4128440 0.2031549

年齢を基準にしてみると、子供の性別の割合と、大人の性別の割合は、大きく異なっているように見える。大人の方が、男性の割合がとても多い。さて、この割合は、偶然と呼べるくらいの変動で説明できるのだろうか。それとも、偶然とは言えないレベルで、子供と大人では、性別の割合が異なるのだろうか。これを検定の枠組みで考えてみよう。

-   帰無仮説：子供と大人で、男女比は同じである
-   対立仮説：子供と大人で、男女比は異なる

となるだろう。検定は、chisq.test関数に、table関数で作ったクロス集計表をインプットしてみればよい。

``` r
chisq.test(table(data_titanic$Sex, data_titanic$Age))
```

    ## 
    ##  Pearson's Chi-squared test with Yates' continuity correction
    ## 
    ## data:  table(data_titanic$Sex, data_titanic$Age)
    ## X-squared = 25.89, df = 1, p-value = 3.613e-07

p値はとても小さく(3.61344110^{-7})、帰無仮説「子供と大人で、男女比は同じである」は否定でき、対立仮説「子供と大人で、男女比は異なる」と言えるだろう。

それでは、もっと生々しい例題を。

年齢と生存でクロス集計表にすると、子供は50%以上助かり、大人は30%しか助かっていない。子供と大人で生存率に差があると言っていいだろうか。

``` r
table(data_titanic$Age, data_titanic$Survived)
```

    ##        
    ##           No  Yes
    ##   Child   52   57
    ##   Adult 1438  654

``` r
prop.table(table(data_titanic$Age, data_titanic$Survived), margin = 1)
```

    ##        
    ##                No       Yes
    ##   Child 0.4770642 0.5229358
    ##   Adult 0.6873805 0.3126195

検定してみると、p値はとても小さく、「子供と大人の生存率は同じ生存率で、今回の差はブレの範囲である」とは、とても言えない。子供が優先的に助けられた、もしくは、大人は見捨てられがちだった、と言えるだろう。

``` r
chisq.test(table(data_titanic$Age, data_titanic$Survived))
```

    ## 
    ##  Pearson's Chi-squared test with Yates' continuity correction
    ## 
    ## data:  table(data_titanic$Age, data_titanic$Survived)
    ## X-squared = 20.005, df = 1, p-value = 7.725e-06

同様に、女性の方が助かっており、有意差もある。つまり、女性の方が優先的に救助された可能性が高い。

``` r
table(data_titanic$Sex, data_titanic$Survived)
```

    ##         
    ##            No  Yes
    ##   Male   1364  367
    ##   Female  126  344

``` r
prop.table(table(data_titanic$Sex, data_titanic$Survived), margin = 1)
```

    ##         
    ##                 No       Yes
    ##   Male   0.7879838 0.2120162
    ##   Female 0.2680851 0.7319149

``` r
chisq.test(table(data_titanic$Sex, data_titanic$Survived))
```

    ## 
    ##  Pearson's Chi-squared test with Yates' continuity correction
    ## 
    ## data:  table(data_titanic$Sex, data_titanic$Survived)
    ## X-squared = 454.5, df = 1, p-value < 2.2e-16

一方、船室等級毎に見ると、一等船室の旅客の生存率が一番高く、次いで二等船室、三等船室、船員、と言う順で生存率が下がっていく。

``` r
table(data_titanic$Class, data_titanic$Survived)
```

    ##       
    ##         No Yes
    ##   1st  122 203
    ##   2nd  167 118
    ##   3rd  528 178
    ##   Crew 673 212

``` r
prop.table(table(data_titanic$Class, data_titanic$Survived), margin = 1)
```

    ##       
    ##               No       Yes
    ##   1st  0.3753846 0.6246154
    ##   2nd  0.5859649 0.4140351
    ##   3rd  0.7478754 0.2521246
    ##   Crew 0.7604520 0.2395480

これも検定にかけるとp値は十分に低く、船室等級や船員であることにより、生存率は異なる、と結論できる。

``` r
chisq.test(table(data_titanic$Class, data_titanic$Survived))
```

    ## 
    ##  Pearson's Chi-squared test
    ## 
    ## data:  table(data_titanic$Class, data_titanic$Survived)
    ## X-squared = 190.4, df = 3, p-value < 2.2e-16

ただし、この結論については、若干注意が必要である。2×2のクロス集計表**以外**の場合、「どの等級が生存率が**高い** and/or **低い**」ということまでは、ここまでの検定方法では結論できない(検定する方法はある、すぐに後述する)。結論できるのは「船室等級や船員であることにより、生存率は異なる」という事実までである。

さて、以下のコードで、分割表全体のp値ではなく、全体生存率に対する個別の生存率のp値も計算できる(以下のクロス集計表がそれぞれのセルのp値である)。今回の場合、いずれのセルの生存率も、全体生存率32.3%よりも、有意に高い、または、有意に低い、と結論できそうである。

参考：<http://langstat.hatenablog.com/entry/20150211/1423634853>

``` r
pnorm(abs(chisq.test(table(data_titanic$Class, data_titanic$Survived))$stdres), lower.tail = FALSE)*2
```

    ##       
    ##             No     Yes
    ##   1st  2.3e-36 2.3e-36
    ##   2nd  4.3e-04 4.3e-04
    ##   3rd  1.0e-06 1.0e-06
    ##   Crew 6.5e-12 6.5e-12

さて、クロス集計表をよく見ると、三等船室と船員はほとんどかわらない生存率である。

``` r
table(data_titanic$Class, data_titanic$Survived)
```

    ##       
    ##         No Yes
    ##   1st  122 203
    ##   2nd  167 118
    ##   3rd  528 178
    ##   Crew 673 212

``` r
prop.table(table(data_titanic$Class, data_titanic$Survived), margin = 1)
```

    ##       
    ##          No  Yes
    ##   1st  0.38 0.62
    ##   2nd  0.59 0.41
    ##   3rd  0.75 0.25
    ##   Crew 0.76 0.24

若干三等船室の方が高いものの、これは意味があるほど高いと言える生存率なのだろうか。三等船室と船員だけでクロス集計表を作り、これを検定にかけてみよう。

``` r
# まずは、三等船室と船員のみのデータを作る
# コードの意味が知りたかったら、質問してください
data_titanic_3rdClass_Crew <-
  droplevels(data_titanic[(data_titanic$Class == "3rd" | data_titanic$Class == "Crew"),])

table_3rdClass_Crew_survival<-
  table(data_titanic_3rdClass_Crew$Class, data_titanic_3rdClass_Crew$Survived) # クロス集計表を代入する

table_3rdClass_Crew_survival # 代入したクロス集計表を表示する
```

    ##       
    ##         No Yes
    ##   3rd  528 178
    ##   Crew 673 212

``` r
prop.table(table_3rdClass_Crew_survival, margin = 1)
```

    ##       
    ##          No  Yes
    ##   3rd  0.75 0.25
    ##   Crew 0.76 0.24

``` r
chisq.test(table_3rdClass_Crew_survival)
```

    ## 
    ##  Pearson's Chi-squared test with Yates' continuity correction
    ## 
    ## data:  table_3rdClass_Crew_survival
    ## X-squared = 0.3, df = 1, p-value = 0.6

検定の結果、p値は低くなく、三等船室と船員の生存率に差はない。つまり、「船員は、三等船室の旅客の救助を、自分たちが助かることよりも優先して行った」とは言えない(実際には、救助活動の有無というファクター以外にも、三等船室が位置(船体下部)により避難自体が難しかった、という説もある)。

おまけ：クロス集計表はエクセルからコピペできるよ(非推奨)
--------------------------------------------------------

クロス集計表を検定にかけられることは分かっていただけたと思う。が、Excelで作ったクロス集計表が手元にあるのに、わざわざRでデータをインポートし、データ加工し、クロス集計表をつくり、検定にかけなけれならないとなると、(現時点では)ちょっと大変である。

理想的には、Rですべてを行ってほしいが、今ある情報試算を活用してもらうべく、ExcelとRのちょっとした連携方法をお伝えしたい。その方法とは、windowsのクリップボードから、クロス集計表をコピーし、Rに取り込むことである。やり方は以下の通り。

``` r
# x <- read.table("clipboard") 
# x
# chisq.test(x)
```

なお、(すこし加工が必要だが)Excelでカイ2乗検定を行うこともできる。興味がある人は調べてみてください。
