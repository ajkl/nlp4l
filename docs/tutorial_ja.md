### 目次

* [はじめよう！](#gettingStarted)
* [インストール](#install)
* [練習用コーパスの入手](#getCorpora)
    * [NLP4Lの対話型シェル](#getCorpora_repl)
    * [インデックスとは？](#getCorpora_index)
    * [livedoorニュースコーパスの入手とインデックスの作成](#getCorpora_ldcc)
    * [書籍「言語研究のための統計入門」付属CD-ROMデータを使ったインデックスの作成](#getCorpora_book)
    * [ブラウンコーパスの入手とインデックスの作成](#getCorpora_brown)
    * [ロイターコーパスの入手とインデックスの作成](#getCorpora_reuters)
    * [Wikipediaデータの入手とインデックスの作成](#getCorpora_wiki)
    * [NLP4L のスキーマについて](#getCorpora_schema)
    * [CSV ファイルのインポート](#getCorpora_csv)
* [NLPツールとしての利用](#useNLP)
    * [単語の数を数える](#useNLP_wordcounts)
    * [隠れマルコフモデル](#useNLP_hmm)
    * [連語分析モデル](#useNLP_collocanalysis)
    * [固有表現抽出](#useNLP_nee)
    * [バディワード抽出](#useNLP_buddy)
    * [専門用語抽出](#useNLP_te)
    * [仮説検定](#useNLP_hypothesistesting)
    * [相関分析](#useNLP_correlationanalysis)
* [インデックスブラウザを使う](#indexBrowser)
    * [フィールド、単語のブラウジング](#indexBrowser_fields)
    * [ドキュメントの閲覧](#indexBrowser_docs)
    * [Position / Offsets](#indexBrowser_posoff)
    * [出現頻度の高い Top N 単語の抽出](#indexBrowser_topn)
* [Solrユーザの皆様](#dearSolrUsers)
* [Elasticsearchユーザの皆様](#dearESUsers)
* [Mahoutと連携する](#useWithMahout)
* [Sparkと連携する](#useWithSpark)
    * [MLLibと連携する](#useWithSpark_mllib)
* [Luceneを使う](#useLucene)
    * [NLP4Lが提供するLuceneの機能](#useLucene_functions)
    * [Analyzer](#useLucene_analyzer)
    * [既存インデックスを検索する](#useLucene_search)
    * [簡単ファセット](#useLucene_facet)
    * [Term Vector から TF/IDF 値を求める](#useLucene_tfidf)
    * [文書類似度を計算する](#useLucene_similarity)
    * [More Like This](#useLucene_mlt)
    * [独自インデックスの作成](#useLucene_index)
    * [FST を単語辞書として使う](#useLucene_fst)
* [Apache Zeppelin から NLP4L を使う](#withZeppelin)
    * [Apache Zeppelin のインストール](#withZeppelin_install)
    * [NLP4L ライブラリの Apache Zeppelin へのデプロイ](#withZeppelin_deploy)
    * [Apache Zeppelin の起動](#withZeppelin_start)
    * [ノートの作成と NLP4LInterpreter の保存](#withZeppelin_save)
    * [NLP4L のコマンドやプログラムの実行](#withZeppelin_exec)
    * [単語カウントの視覚化](#withZeppelin_visualize)
    * [ジップの法則を確認する](#withZeppelin_zipfslaw)
* [NLP4Lプログラムを開発して実行する](#develop)
    * [REPLから実行する](#develop_repl)
    * [コンパイルして実行する](#develop_exec)
* [帰属](#tm)

# はじめよう！{#gettingStarted}

# インストール{#install}

## インストール後のディレクトリ構造

インストール後のディレクトリ構造は以下のようになっています。$NLP4L_HOME は、NLP4L のインストールディレクトリを指しています。

```shell
$NLP4L_HOME/
    bin/
    docs/
    examples/
    lib/
```

# 練習用コーパスの入手{#getCorpora}

NLP4Lを使って自分自身のテキストファイルの分析を始める前に、練習用コーパスを使って動作確認することをお勧めします。いきなり独自のテキストファイルを分析すると、うまく動作させるのに時間がかかったり、分析結果をどのように評価していいか悩んでしまうことがあるかもしれません。

ここで説明する練習用コーパスを使ってインデックスを作成しておくと、これ以降に書かれている解説も実際に試すことができるので理解も容易になるでしょう。

なおここで紹介するコーパスは、livedoorニュースコーパスとWikipediaを除き、研究目的以外での利用が禁止されています。使用に際しては十分ご注意ください。

## NLP4Lの対話型シェル{#getCorpora_repl}

NLP4LにはコマンドやScalaコードを実行するのに便利な対話型シェルが付属しています。NLP4Lをビルドしたら次のように対話型シェル（REPL）を起動してください。

```shell
$ ./target/pack/bin/nlp4l
Welcome to NLP4L!
Type in expressions to have them evaluated.
Type :help for more information
Type :? for information about NLP4L utilities

nlp4l> 
```

## インデックスとは？{#getCorpora_index}

NLP4Lでは自然言語処理を行うテキストファイルをLuceneの転置インデックスに保存します。転置インデックスは単語をキーにしてその単語を含むドキュメント番号のリストを得られるように整理されたファイル構造です。転置インデックスを本書では単にインデックスと呼ぶことにします。

インデックスはNLP4Lの機能を使ってテキストファイルから新規に作成することもできますし、Apache Solr や Elasticsearch を使って作られた既存のインデックスをNLP4Lの処理対象とすることもできます。ただしその場合は Solr や Elasticsearch が使っている Lucene のバージョンに注意しましょう。あまりに古いバージョンで作成されたインデックスはNLP4LのLuceneライブラリで読めない可能性もあります。

以降では練習用のコーパス（テキストファイル）を入手して新規にインデックスを作る方法を説明します。

## livedoorニュースコーパスの入手とインデックスの作成{#getCorpora_ldcc}

次のように実行してロンウイットのサイトから livedoorニュースコーパスをダウンロードして展開します。

```shell
$ mkdir -p $NLP4L_HOME/corpora/ldcc
$ cd $NLP4L_HOME/corpora/ldcc
$ wget http://www.rondhuit.com/download/ldcc-20140209.tar.gz
$ tar xvzf ldcc-20140209.tar.gz
```

Windows ユーザーの方は、適宜読み替えて実行してください。

[Note] NLP4L の対話型シェルには、上記手順を実行するコマンド(Unix系OSのみ対応)が用意されています。

```shell
nlp4l> downloadLdcc
Successfully downloaded ldcc-20140209.tar.gz
Try to execute system command: tar xzf /Users/tomoko/repo/NLP4L/corpora/ldcc/ldcc-20140209.tar.gz -C /Users/tomoko/repo/NLP4L/corpora/ldcc
Success.
```

livedoorニュースコーパスは展開するとtextディレクトリの直下に以下のようなカテゴリ名のついたサブディレクトリを持ちます。

```shell
$ ls -l text
total 16
-rw-r--r--    1 koji  staff    223  9 16  2012 CHANGES.txt
-rw-r--r--    1 koji  staff   2182  9 13  2012 README.txt
drwxr-xr-x  873 koji  staff  29682  2  9  2014 dokujo-tsushin
drwxr-xr-x  873 koji  staff  29682  2  9  2014 it-life-hack
drwxr-xr-x  867 koji  staff  29478  2  9  2014 kaden-channel
drwxr-xr-x  514 koji  staff  17476  2  9  2014 livedoor-homme
drwxr-xr-x  873 koji  staff  29682  2  9  2014 movie-enter
drwxr-xr-x  845 koji  staff  28730  2  9  2014 peachy
drwxr-xr-x  873 koji  staff  29682  2  9  2014 smax
drwxr-xr-x  903 koji  staff  30702  2  9  2014 sports-watch
drwxr-xr-x  773 koji  staff  26282  2  9  2014 topic-news
```

さらにそれぞれのサブディレクトリの下に1記事1ファイルに分かれたファイルを持っています。1つの記事ファイルは次のようになっています。

```shell
$ head -n 5 text/sports-watch/sports-watch-6577722.txt
http://news.livedoor.com/article/detail/6577722/
2012-05-21T09:00:00+0900
渦中の香川真司にインタビュー、「ズバリ次のチーム、話を伺いたい」
20日放送、NHK「サンデースポーツ」では、山岸舞彩キャスターが日本代表・香川真司に行ったインタビューの模様を放送した。

```

1行目がlivedoorニュース記事URL、2行目が記事の日付、3行目が記事のタイトル、4行目以降が記事本文です。

では次に、nlp4l コマンドプロンプトから examples/index_ldcc.scala プログラムを実行してlivedoorニュースコーパスをLuceneインデックスに登録します。それには次のようにnlp4lを起動してloadコマンドで examples/index_ldcc.scala プログラムを実行すればOKです。

```shell
$ bin/nlp4l
nlp4l> :load examples/index_ldcc.scala
```

なお、このプログラムでは冒頭で、次のように作成するLuceneインデックスのディレクトリを次のように定義しています。

```scala
val index = "/tmp/index-ldcc"
```

このディレクトリで都合が悪い場合（Windowsを使っているときなど）は別の場所を指すように変更してからloadコマンドで再度プログラムを実行してください。

このプログラムの実行後、Luceneインデックスのディレクトリは次のようになっています。

```shell
$ ls -l /tmp/index-ldcc
total 67432
-rw-r--r--  1 koji  wheel  16359884  2 24 13:40 _1.fdt
-rw-r--r--  1 koji  wheel      4963  2 24 13:40 _1.fdx
-rw-r--r--  1 koji  wheel       520  2 24 13:40 _1.fnm
-rw-r--r--  1 koji  wheel      7505  2 24 13:40 _1.nvd
-rw-r--r--  1 koji  wheel       147  2 24 13:40 _1.nvm
-rw-r--r--  1 koji  wheel       453  2 24 13:40 _1.si
-rw-r--r--  1 koji  wheel  11319391  2 24 13:40 _1.tvd
-rw-r--r--  1 koji  wheel      5636  2 24 13:40 _1.tvx
-rw-r--r--  1 koji  wheel   2169767  2 24 13:40 _1_Lucene50_0.doc
-rw-r--r--  1 koji  wheel   3322315  2 24 13:40 _1_Lucene50_0.pos
-rw-r--r--  1 koji  wheel   1272515  2 24 13:40 _1_Lucene50_0.tim
-rw-r--r--  1 koji  wheel     26263  2 24 13:40 _1_Lucene50_0.tip
-rw-r--r--  1 koji  wheel       136  2 24 13:40 segments_1
-rw-r--r--  1 koji  wheel         0  2 24 13:40 write.lock
```

## 書籍「言語研究のための統計入門」付属CD-ROMデータを使ったインデックスの作成{#getCorpora_book}

以下の書籍[1]をお持ちの方は、付属CD-ROMのデータをコーパスとして使えます。この書籍をお持ちでない方は次の節へお進みください。

```shell
[1] 言語研究のための統計入門
石川慎一郎、前田忠彦、山崎誠 編
くろしお出版
ISBN978-4-87424-498-2
```

corpora/CEEAUS 以下に「言語研究のための統計入門」付属CD-ROMのデータの"INDIVIDUAL WRITERS"以下のフォルダー（"INDIVIDUAL WRITERS"は含まない）と、"PLAIN"以下のフォルダー（"PLAIN"を含む）をコピーしてください。コピー後、次のようになっていることを確認してください。

```shell
# ディレクトリの作成
$ mkdir -p corpora/CEEAUS

# ここで CD-ROM をコピー

# コピー内容の確認
$ find corpora/CEEAUS -type d
corpora/CEEAUS
corpora/CEEAUS/CEECUS
corpora/CEEAUS/CEEJUS
corpora/CEEAUS/CEENAS
corpora/CEEAUS/CJEJUS
corpora/CEEAUS/PLAIN
```

CEEAUSコーパスは執筆条件が高度に統制されたコーパスとなっているのが特徴です。記事は2種類あり、1つは「大学生のアルバイト」（ファイル名にptjがつくもの）、もう一つは「レストラン全面禁煙」（ファイル名にsmkがつくもの）というテーマについて書かれたものとなっています。CEEAUS以下のサブディレクトリは次のように分かれています。

|サブディレクトリ|内容|
|:----:|:----------------------------|
|CEEJUS|日本人大学生による英作文770本|
|CEECUS|中国人大学生による英作文92本|
|CEENAS|成人英語母語話者による英作文92本|
|CJEJUS|日本人大学生による日本語作文50本|
|PLAIN|上記すべてを含む|

ではCEEAUSコーパスからLuceneインデックスを作成します。最初に上記一覧表のPLAIN以外のコーパスからLuceneインデックスを作成します。

```shell
$ bin/nlp4l
nlp4l> :load examples/index_ceeaus.scala
```

次にPLAINからLuceneインデックスを作成します。

```shell
$ bin/nlp4l
nlp4l> :load examples/index_ceeaus_all.scala
```

それぞれのプログラムの冒頭を見ていただければわかりますが、Luceneインデックスはそれぞれ、/tmp/index-ceeausと/tmp/index-ceeaus-allに作られます。前の例と同じように、別のディレクトリに作成したいときは、書き換えて再度プログラムを実行してください。

## ブラウンコーパスの入手とインデックスの作成{#getCorpora_brown}

corpora/brown 以下に次のようにしてブラウンコーパスをダウンロードし、展開します。

```shell
$ mkdir $NLP4L_HOME/corpora/brown
$ cd $NLP4L_HOME/corpora/brown
$ wget https://ia600503.us.archive.org/21/items/BrownCorpus/brown.zip
$ unzip brown.zip
```

Windows ユーザーの方は、適宜読み替えて実行してください。

[Note] NLP4L の対話型シェルには、上記手順を実行するコマンド(Unix系OSのみ対応)が用意されています。

```shell
nlp4l> downloadBrown
Successfully downloaded brown.zip
Try to execute system command: unzip -o /Users/tomoko/repo/NLP4L/corpora/brown/brown.zip -d /Users/tomoko/repo/NLP4L/corpora/brown
Success.
```

次に、nlp4l プロンプトからLuceneインデックスを作成します。

```shell
$ bin/nlp4l
nlp4l> :load examples/index_brown.scala
```

プログラムの冒頭を見ていただければわかりますが、Luceneインデックスは/tmp/index-brownに作られます。前の例と同じように、別のディレクトリに作成したいときは、書き換えて再度プログラムを実行してください。

## ロイターコーパスの入手とインデックスの作成{#getCorpora_reuters}

ロイターコーパスはアメリカ国立標準技術研究所（NIST）に[申し込む](http://trec.nist.gov/data/reuters/reuters.html)ことで入手できます。ここでは同サイトに紹介されている[David D. Lewis博士のサイト](http://www.daviddlewis.com/resources/testcollections/rcv1/)からダウンロードできるアーカイブを使ってインデックスを作成する方法を参考までにご紹介します。

corpora/reuters 以下に次のようにしてロイターコーパスをダウンロードし、展開します。

```shell
$ mkdir $NLP4L_HOME/corpora/reuters
$ cd $NLP4L_HOME/corpora/reuters
$ wget http://www.daviddlewis.com/resources/testcollections/reuters21578/reuters21578.tar.gz
$ tar xvzf reuters21578.tar.gz
```

Windows ユーザーの方は、適宜読み替えて実行してください。

[Note] NLP4L の対話型シェルには、上記手順を実行するコマンド(Unix系OSのみ対応)が用意されています。

```shell
nlp4l> downloadReuters
Successfully downloaded reuters21578.tar.gz
Try to execute system command: tar xzf /Users/tomoko/repo/NLP4L/corpora/reuters/reuters21578.tar.gz -C /Users/tomoko/repo/NLP4L/corpora/reuters
Success.
```

次に、nlp4l プロンプトからLuceneインデックスを作成します。

```shell
$ bin/nlp4l
nlp4l> :load examples/index_reuters.scala
```

これまで同様プログラムを見れば、Luceneインデックスは/tmp/index-reutersに作られることがわかります。前の例と同じように、別のディレクトリに作成したいときは、書き換えて再度プログラムを実行してください。

## Wikipediaデータの入手とインデックスの作成{#getCorpora_wiki}

Wikipediaデータは今NLP研究のコーパスとしても最も人気の高いものの一つとなっています。ただWikipediaの記事テキストはWikipedia独特のルールに従って書かれているため、Luceneインデックスに取り込む前になるべくテキストデータだけを抽出する前処理をする必要があり、Wikipediaデータを使う際のハードルともなっています。

ここでは[json-wikipedia](https://github.com/diegoceccarelli/json-wikipedia)を使って簡単にWikipediaデータをLuceneインデックスに取り込む方法をご紹介します。

### json-wikipedia のダウンロードとビルド

まず[json-wikipedia](https://github.com/diegoceccarelli/json-wikipedia)をダウンロードしますが、作業用としてworkというディレクトリを作成し、そこにjson-wikipediaをダウンロードします。

```shell
$ mkdir work
$ cd work
$ wget https://github.com/diegoceccarelli/json-wikipedia/archive/master.zip
$ unzip master.zip
```

次に、ZIPを展開してできたディレクトリの下でjson-wikipediaをビルドします。

```shell
$ cd json-wikipedia-master
$ mvn assembly:assembly
```

これによりtargetディレクトリ以下に作成されたJARファイルは、NLP4LからJSONに変換されたWikipediaデータをLuceneインデックスに登録する際に参照します。

```shell
$ ls target
archive-tmp                                    json-wikipedia-1.0.0.jar
classes                                        maven-archiver
generated-sources                              surefire-reports
generated-test-sources                         test-classes
json-wikipedia-1.0.0-jar-with-dependencies.jar
```

### WikipediaデータのダウンロードとJSONへの変換

[Wikipediaのダウンロードサイト](https://dumps.wikimedia.org/backup-index.html)から各国語のリンクをたどります（日本語ならjawiki、英語ならenwiki）。そしてXXwiki-YYYYMMDD-pages-articles.xml.bz2（XXは言語、YYYYMMDDは日付）という名前のファイルをダウンロードして展開します。

```shell
$ wget https://dumps.wikimedia.org/jawiki/20150512/jawiki-20150512-pages-articles.xml.bz2
$ bunzip2 jawiki-20150512-pages-articles.xml.bz2
```

そしてjson-wikipediaを次のように実行してJSON形式に変換します。

```shell
$ ./scripts/convert-xml-dump-to-json.sh en jawiki-20150512-pages-articles.xml /tmp/jawiki.json
```

第1引数には言語を指定します。現在json-wikipediaがサポートする言語は英語（en）やイタリア語（it）など非常に限られており、日本語のサポートはありませんので、上の例では英語（en）を第1引数に指定しています（それでも問題なく動作するようです）。日本語WikipediaのJSON形式への変換はおよそ30分程度です。

### Luceneインデックスの作成

最後にJSON形式のデータをNLP4LからLuceneインデックスに登録しますが、ビルドしてできたjson-wikipediaのJARファイルを、NLP4Lのクラスパスに含める必要があることに注意してください。

```shell
$ ./target/pack/bin/nlp4l -cp json-wikipedia-1.0.0-jar-with-dependencies.jar 
nlp4l> 
```

あとは次のようにexamples/index_jawiki.scalaを実行するだけです。ただしこのサンプルプログラムは日本語Wikipedia用なので、英語など他の言語の場合はこのサンプルプログラムをコピーして他言語用のプログラムを作る必要があります。特に注意が必要なのは、プログラムから参照しているスキーマ設定ファイルです。examples/schema/jawiki.confは日本語を処理するのでJapaneseAnalyzerを指定していますが、英語などのスペースで分かち書きされている言語ではStandardAnalyzerにするのがよいでしょう。

```shell
nlp4l> :load examples/index_jawiki.scala
```

日本語Wikipediaの場合はおよそ30分程度でLuceneインデックスの作成が完了します。

## NLP4L のスキーマについて{#getCorpora_schema}

Luceneのインデックスは基本的にスキーマレスですが、NLP4Lではスキーマを設定できます。ここで練習用コーパスのうち、livedoorニュースコーパス（ldcc）、CEEAUS、ブラウンコーパス（brown）で定義されているスキーマを概観しましょう。

以下の表は各コーパスが持つフィールド名を示しています（コーパスで有効なフィールド名にチェック（x）が入っています）。

|フィールド名|ldcc|CEEAUS|brown|
|:----------:|:--:|:----:|:---:|
|file        |    |  x   |  x  |
|type        |    |  x   |     |
|cat         | x  |  x   |  x  |
|url         | x  |      |     |
|date        | x  |      |     |
|title       | x  |      |     |
|body        | x  |      |  x  |
|body_en     |    |  x   |     |
|body_ws     |    |  x   |     |
|body_ja     |    |  x   |     |
|body_pos    |    |      |  x  |

このうち、titleフィールド以下はLuceneのAnalyzerによってコーパスの登録時に文章が単語単位に分割されます。LuceneのAnalyzerは非常に高機能であり、分割だけでなくストップワードの除去、ステミング、各種文字変換などの正規化も行います。どのように分割／正規化されるかは後述します。

それ以外のフィールドは、登録時に単語分割が行われず、コーパスの文字列がそのまま登録されます。特にcatフィールドには文書カテゴリが記録されているため、文書分類タスクなどに利用することができます。

### ldcc のスキーマ

title にはニュース記事のタイトルが、body には記事本文が格納されています。2つのフィールドとも、Lucene の JapaneseAnalyzer によって単語分割が行われます。

### CEEAUS のスキーマ

type には CEEJUS/CEECUS/CEENAS/CJEJUS の区別が、cat には ptj（大学生のアルバイト）/smk（レストラン全面禁煙）のカテゴリが入れられています。body_en と body_ws には同じ英文が格納されていますが、body_en は Lucene の StandardAnalyzer が、body_ws には Lucene の WhitespaceAnalyzer が適用されています。これらのフィールドの使い分けですが、後述の仮説検定ではbody_enを、相関分析ではbody_wsを使っています。body_ja はサブコーパス CJEJUS の日本語文章が格納されています。このフィールドには Lucene の JapaneseAnalyzer が適用されています。

### brown のスキーマ

ブラウンコーパスの記事は次のように品詞タグが各単語ごとについています。

```shell
The/at Fulton/np-tl County/nn-tl Grand/jj-tl Jury/nn-tl said/vbd Friday/nr an/at investigation/nn of/in Atlanta's/np$ recent/jj primary/nn election/nn produced/vbd ``/`` no/at evidence/nn ''/'' that/cs any/dti irregularities/nns took/vbd place/nn ./.
```

この文章がそのまま格納されているのが body_pos フィールドで、同じ内容だが品詞タグが取り除かれているのが body フィールドとなっています。

## CSV ファイルのインポート{#getCorpora_csv}

練習用コーパスを使ってLuceneインデックスを作成してきましたが、最後に独自のCSVファイルをLuceneインデックスにインポートする方法を見てみましょう。

例として次のようなCSVファイルがあるとします。

```shell
$ cat << EOF > /tmp/data.csv
1, NLP4L, "NLP4L is a natural language processing tool for Apache Lucene written in Scala."
2, NLP4L, "The main purpose of NLP4L is to use the NLP technology to improve Lucene users' search experience."
3, LUCENE, "Apache Lucene is a high-performance, full-featured text search engine library written entirely in Java."
4, SOLR, "Solr is highly reliable, scalable and fault tolerant, providing distributed indexing, replication and load-balanced querying, automated failover and recovery, centralized configuration and more."
5, SOLR, "Solr powers the search and navigation features of many of the world's largest internet sites."
EOF
```

またスキーマファイルは次のようになっているとします。

```shell
$ cat << EOF > /tmp/schema.conf
schema {
  defAnalyzer {
    class : org.apache.lucene.analysis.standard.StandardAnalyzer
  }
  fields　= [
    {
      name : id
      indexed : true
      stored : true
    }
    {
      name : cat
      indexed : true
      stored : true
    }
    {
      name : body
      analyzer : {
        tokenizer {
          factory : standard
        }
        filters = [
          {
            factory : lowercase
          }
        ]
      }
      indexed : true
      stored : true
      termVector : true
      positions : true
      offsets : true
    }
  ]
}
EOF
```

このとき、CSVファイルをインポートするコマンドは次のように実行します。

```shell
$ java -cp "target/pack/lib/*" org.nlp4l.core.CSVImporter --index /tmp/index-tmp --schema /tmp/schema.conf --fields id,cat,body /tmp/data.csv
```

# NLPツールとしての利用{#useNLP}

NLP4L を NLP ツールとして使う方法を紹介します。前述のLuceneインデックスに登録した練習用コーパスを使いますので、あらかじめ用意しておくとよいでしょう。

## 単語の数を数える{#useNLP_wordcounts}

コーパス中に出現する単語の数を数えるのは NLP 処理の基本です。NLP4L はコーパスをLuceneのインデックスに登録してから処理を行いますが、検索エンジン（Lucene）では単語をキーにした転置インデックスというものを持っているので、単語の数を数える処理は大変得意としています。

ここではロイターコーパスを使って単語出現頻度を調べる方法を説明します。準備として次のプログラムをコピー＆ペーストでnlp4lプロンプトに貼り付けて実行してください。なお、コピー＆ペースト操作がしやすいように、複数行のプログラムでは nlp4l プロンプト表示を省略しています。

```scala
// (1)
import org.nlp4l.core._
import org.nlp4l.core.analysis._
import org.nlp4l.stats.WordCounts

// (2)
val index = "/tmp/index-reuters"

// (3)
val reader = RawReader(index)
```

(1)で必要なScalaプログラムのパッケージをインポートしています。このうち、単語の出現頻度はWordCountsオブジェクトを使います。(2)でロイターコーパスのLuceneインデックスディレクトリを指定しています。(3)でRawReaderに使用するLuceneインデックスディレクトリを指定してreaderを取得しています。RawReaderというのはNLP4Lでは比較的低レベルのReaderとなります。別にスキーマを管理する高レベルのIReaderというReaderもありますが、スキーマを渡す手間を省くため、ここではあえてRawReaderを使っています。

取得したreaderを通じて以下で単語の出現頻度を数えていきます。なお、Luceneはフィールドごとに独立した転置インデックスを持っているため、単語数をカウントするなどの処理に際してはフィールド名を指定する必要があります。以下ではロイターの記事本文を登録しているbodyフィールドを指定することにします。

### のべ語数と異なり語数

最初はのべ語数です。Luceneにはもともとあるフィールドののべ語数を返す機能があるので、そのScalaラッパーであるsumTotalTermFreq()関数を使えば簡単に調べられます。

```shell
nlp4l> val total = reader.sumTotalTermFreq("body")
total: Long = 1899819
```

次は異なり語数です。異なり語数とは、単語の種類数です。転置インデックスは異なり語をキーとする構造を持っているので転置インデックスのサイズが異なり語数となります。当然Luceneでは簡単に調べることができ、そのScalaラッパーは次のようになります。

```shell
nlp4l> val count = reader.field("body").get.terms.size
count: Int = 64625
```

### 単語ごとに数える

今度はもう少し細かく、単語ごとに数えてみましょう。WordCountsオブジェクトのcount()関数を使えば単語ごとに数えることができます。しかし、count()関数はいくつかの引数をとるため、少し準備が必要です。以下にプログラムを示します（nlp4lのプロンプトにコピー＆ペーストして実行できます）。

```scala
// (4)
val allDS = reader.universalset()

// (5)
val analyzer = Analyzer(new org.apache.lucene.analysis.standard.StandardAnalyzer(null.asInstanceOf[org.apache.lucene.analysis.util.CharArraySet]))

// (6)
val allMap = WordCounts.count(reader, "body", Set.empty, allDS, -1, analyzer)
```

(4)ではcount()関数で使うカウント対象となるLuceneの文書番号を取得しています。ここでは全文書を対象にするので、文書の全体集合を取得するuniversalset()関数を使っています。(5)ではLuceneの標準的なAnalyzerであるStandardAnalyzerを指定してScalaのAnalyzerを作成しています。StandardAnalyzerの引数に指定しているnullは、ストップワードを使用しないということを表しています（nullを指定しないとStandardAnalyzerのデフォルトのストップワードが使われます）。(6)のcount()関数で単語ごとの出現頻度を求めています。引数に指定しているSet.emptyは、「すべての単語」を対象にカウントを求めることを指定しています。-1の部分は出現頻度の多い単語上位N個を取得したいときにそのNを指定します。-1はすべての単語を対象に調べる場合に指定します。

結果は表示されましたが、このままだと結果が見にくいので、表示数を絞ってみましょう。たとえばScalaのコレクション関数の機能を使って次のようにすると、"g"で始まる単語を10個選んで単語とその出現数を表示することができます。

```shell
nlp4l> allMap.filter(_._1.startsWith("g")).take(10).foreach(println(_))
(generalize,1)
(gress,1)
(germans,18)
(goiporia,2)
(garcin,2)
(granma,7)
(gorbachev's,10)
(gamble,9)
(grains,110)
(gienow,1)
```

allMapの結果のすべての数を足し合わせてみます（以下のプログラムはWordCountsのtotalCount()関数とやっていることは同じです）。

```shell
nlp4l> allMap.values.sum
res2: Long = 1899819
```

これは最初に調べたのべ語数と確かに一致していることがわかります。また、allMapの大きさは異なり語数と一致しているはずです。やってみましょう。

```shell
nlp4l> allMap.size
res3: Int = 64625
```

こちらも一致しました。

上のcount()ではSet.emptyを渡して全単語を対象に出現頻度をカウントしていましたが、Set.emptyの代わりに特定の単語集合を与えることで、その単語集合だけを対象にカウントできます（特定の単語の数だけを数えたいこともNLPではよくあります）。やってみましょう。

```scala
val whWords = Set("when", "where", "who", "what", "which", "why")
val whMap = WordCounts.count(reader, "body", whWords, allDS, -1, analyzer)
whMap.foreach(println(_))
```

結果は次のようになります。

```shell
nlp4l> whMap.foreach(println(_))
(why,125)
(what,850)
(who,1618)
(which,7556)
(where,507)
(when,1986)
```

### カテゴリごとに数える

では次に単語出現頻度をカテゴリごとに区別して求める方法を見てみましょう。たとえばNLPタスクのひとつ、文書分類ではカテゴリに分類するにあたってその学習データとしてカテゴリごとの単語出現頻度を使うことがあります。そのようなときに使えるテクニックです。

ここの例ではロイターコーパスを使っていますが、placesフィールドをカテゴリの代わりに使ってみたいと思います。そのためにはまず、placesフィールドに入っている単語を次のようにして調べます。

```scala
// (7)
reader.terms("places").get.map(_.text)

// (8) 整形して表示したい場合
reader.terms("places").get.map(_.text).foreach(println(_))
```

(7)または(8)を実行すると、placesフィールドに登録されているすべての単語一覧が表示されます。ここではusaとjapanに注目しましょう。(9)のように実行してそれぞれの文書部分集合を取得します。

```scala
// (9)
val usDS = reader.subset(TermFilter("places", "usa"))
val jpDS = reader.subset(TermFilter("places", "japan"))
```

最後にcount()関数に(9)で求めたそれぞれの部分集合を渡してusaのカウントとjapanのカウントを求めますが、ここでは簡単にwarとpeaceの2語だけのカウントを求めることにします(10)。

```shell
// (10)
nlp4l> WordCounts.count(reader, "body", Set("war", "peace"), usDS, -1, analyzer)
res22: Map[String,Long] = Map(war -> 199, peace -> 14)

nlp4l> WordCounts.count(reader, "body", Set("war", "peace"), jpDS, -1, analyzer)
res23: Map[String,Long] = Map(war -> 75, peace -> 2)
```

これでwarとpeaceの2語に対する、placesがusaの場合とjapanの場合でのカウントがそれぞれ求めることができました。めでたしめでたし・・・としていいでしょうか。

実はplacesフィールドは、地名が複数入っている可能性があります。つまり、usaとjapanの記事が重なっている可能性があります。確かめてみましょう。usDSとjpDSはScalaのSetコレクションオブジェクトですから、&演算子（関数）を使って両者の積集合を簡単に求めることができます。

```shell
nlp4l> (usDS & jpDS).size
res24: Int = 452
```

sizeを使って積集合の大きさを求めています。たしかに重複があるようです。この場合、(11)(12)のように、ScalaのSetの演算子（関数）の&\~を使って集合の差を使えば、重複のない部分に関して単語出現頻度を求めることができます（ただしそのままだとSortedSetには&\~演算子（関数）がないので、toSetを使ってSetに変換しています）。

```shell
nlp4l> // (11) placesにusaの値を持つがjapanの値を持たない記事を対象とする
nlp4l> WordCounts.count(reader, "body", Set("war", "peace"), usDS.toSet &~ jpDS.toSet, -1, analyzer)
res25: Map[String,Long] = Map(war -> 140, peace -> 13)

nlp4l> // (12) placesにjapanの値を持つがusaの値を持たない記事を対象とする
nlp4l> WordCounts.count(reader, "body", Set("war", "peace"), jpDS.toSet &~ usDS.toSet, -1, analyzer)
res26: Map[String,Long] = Map(war -> 16, peace -> 1)
```

## 隠れマルコフモデル{#useNLP_hmm}

NLP4L では、ラベルつき訓練データから隠れマルコフモデルを学習することができる HmmModel クラスが提供されています。HmmModel と HmmModelIndexer はともに次の HmmModelSchema で定義されている Luceneインデックスのスキーマを参照しています。

```scala
trait HmmModelSchema {
  def schema(): Schema = {
    val analyzer = Analyzer(new org.apache.lucene.analysis.core.WhitespaceAnalyzer)
    val builder = AnalyzerBuilder()
    builder.withTokenizer("whitespace")
    builder.addTokenFilter("shingle", "minShingleSize", "2", "maxShingleSize", "2", "outputUnigrams", "false")
    val analyzer2g = builder.build
    val fieldTypes = Map(
      "begin" -> FieldType(analyzer, true, true, true, true),
      "class" -> FieldType(analyzer, true, true, true, true),
      "class_2g" -> FieldType(analyzer2g, true, true, true, true),
      "word_class" -> FieldType(analyzer, true, true, true, true),
      "word" -> FieldType(analyzer, true, true, true, true)
    )
    val analyzerDefault = analyzer
    Schema(analyzerDefault, fieldTypes)
  }
}
```

隠れマルコフモデルでは、ある状態（クラス）からある状態へ遷移する確率とある状態における記号（単語）の出力確率を使います。NLP4Lではこれらの確率を HmmModelSchema で定義されるスキーマを持つ Luceneインデックスを使って求めます。状態遷移確率は class と class_2g フィールドのクラス出現数を使います。class_2g フィールドは Lucene の ShingleFilter を使ってクラス2グラムを記録しているフィールドです。class フィールドは単純にクラスを記録しているフィールドです。この2つのフィールドを使って P( vb | nn ) すなわち品詞 nn のあとに品詞 vb が出現する確率を求めることを考えます。これは実は非常に簡単で、先の単語の数を数える totalTermFreq() を使って次のように計算できます。

```math
P( vb | nn ) = "nn vb".totalTermFreq() / "nn".totalTermFreq()
```

"nn vb" という2つのクラスの連続は class_2g フィールドを参照します。"nn" は class フィールドを参照します。同様にクラス nn における単語 program の出力確率は次のように計算できます。

```math
P( program | nn ) = "program_nn".totalTermFreq() / "nn".totalTermFreq()
```

ここで "program_nn" は単語 program とクラス nn が同時に観測された場合の Luceneインデックスに登録する文字列です。これは word_class フィールドに記録されます。

begin フィールドには Lucene ドキュメントの最初のクラスが記録されます。このフィールドの totalTermFreq() を使えば、各クラスの初期状態確率分布が計算できます。

それでは HmmModel を使った具体例を見てみましょう。

### 英語の品詞タグ付け

隠れマルコフモデルを応用した成功例には英語の品詞タグ付けが知られています。ブラウンコーパスの英文には品詞タグがつけられていますので、ここから HmmModel を学習して、未知の英文に品詞タグをつけてみましょう。ブラウンコーパスを用意した状態で、次のようにサンプルスクリプトを実行します。

```shell
nlp4l> :load examples/hmm_postagger.scala
```

プログラムの最後に学習後のLuceneインデックスモデルを用いて未知の英文に品詞タグ付けを行っています。たとえば、"i like to go to france ." という英文の品詞タグ付け結果は次のように出力されます。

```scala
res8: Seq[org.nlp4l.lm.Token] = List(Token(i,ppss), Token(like,vb), Token(to,to), Token(go,vb), Token(to,in), Token(france,np), Token(.,.))
```

ではサンプルプログラム冒頭の学習部分を見てみましょう。

```scala
// (1)
val index = "/tmp/index-brown-hmm"

// (2)
val c: PathSet[Path] = Path("corpora", "brown", "brown").children()

// (3)
val indexer = HmmModelIndexer(index)
c.filter{ e =>
  val s = e.name
  val c = s.charAt(s.length - 1)
  c >= '0' && c <= '9'
}.foreach{ f =>
  val source = Source.fromFile(f.path, "UTF-8")
  source.getLines().map(_.trim).filter(_.length > 0).foreach { g =>
    val pairs = g.split("\\s+")
    val doc = pairs.map{h => h.split("/")}.filter{_.length==2}.map{i => (i(0).toLowerCase(), i(1))}
    indexer.addDocument(doc)
  }
}

// (4)
indexer.close()

// (5)
val model = HmmModel(index)
```

最初に (1) でブラウンコーパスを登録するLuceneインデックスを指定しています。(2) でブラウンコーパスのディレクトリを指定しています。これは下の(3) でファイルをひとつずつ取り出すのに参照されます。(3) で HmmModelIndexer を作成し、ブラウンコーパスのファイルの1行をLuceneドキュメントの1つとみなしてaddDocument()で HmmModelIndexer に追加しています。追加するドキュメントの形式は、単語と品詞のタプルの配列です。

Luceneインデックスを作成し終わったら(4)でクローズします。このようにして作成したLuceneインデックスを(5)で HmmModel で読み込んだときに隠れマルコフモデルが計算されます。

英語の品詞タグ付けでモデルを使う場合は、次のように HmmTagger にモデルを適用し、HmmTagger の tokens() を呼び出します。

```scala
val tagger = HmmTagger(model)

tagger.tokens("i like to go to france .")
tagger.tokens("you executed lucene program .")
tagger.tokens("nlp4l development members may be able to present better keywords .")
```

実行結果は次のように、単語とクラス（品詞）を要素に持つTokenオブジェクトのListとして返されます。

```shell
res23: Seq[org.nlp4l.lm.Token] = List(Token(i,ppss), Token(like,vb), Token(to,to), Token(go,vb), Token(to,in), Token(france,np), Token(.,.))
res24: Seq[org.nlp4l.lm.Token] = List(Token(you,ppo-tl), Token(executed,vbn), Token(lucene,X), Token(program,nil), Token(.,.))
res25: Seq[org.nlp4l.lm.Token] = List(Token(nlp4l,X), Token(development,nn), Token(members,nns), Token(may,md), Token(be,be), Token(able,jj), Token(to,to), Token(present,vb), Token(better,rbr), Token(keywords,X), Token(.,.-hl))
```

### カタカナ語からの英単語推定

NLP4L にはカタカナ語とその元になった英単語のペアに、独自にアライメントをつけた訓練データが付属しています（train_data/alpha_katakana_aligned.txt）。

```shell
$ head train_data/alpha_katakana_aligned.txt 
アaカcaデdeミーmy
アaクcセceンnトt
アaクcセceスss
アaクcシciデdeンnトt
アaクcロroバッbaトt
アaクcショtioンn
アaダdaプpターter
アaフfリriカca
エaアirバbuスs
アaラlaスsカka
```

このデータを使って、カタカナ部分を単語に、アルファベット部分を品詞に見立てて隠れマルコフモデルを学習することを考えてみましょう。すると未知のカタカナ語から英単語を予測する事ができます。それを実装したのがexamples/trans_katakana_alpha.scala です。

```shell
nlp4l> :load examples/trans_katakana_alpha.scala
```

前の examples/hmm_postagger.scala とほぼ同じプログラムなので興味がある方は読み解いてみてください。

同じくプログラムの最後は学習したモデルを使って、未知のカタカナ語から英単語を推定した結果を表示しています。

```scala
val tokenizer = HmmTokenizer(model)

tokenizer.tokens("アクション")
tokenizer.tokens("プログラム")
tokenizer.tokens("ポイント")
tokenizer.tokens("テキスト")
tokenizer.tokens("コミュニケーション")
tokenizer.tokens("エントリー")
```

この実行結果は次のようになります。

```shell
res35: Seq[org.nlp4l.lm.Token] = List(Token(ア,a), Token(ク,c), Token(ショ,tio), Token(ン,n))
res36: Seq[org.nlp4l.lm.Token] = List(Token(プ,p), Token(ロ,ro), Token(グ,g), Token(ラ,ra), Token(ム,m))
res37: Seq[org.nlp4l.lm.Token] = List(Token(ポ,po), Token(イ,i), Token(ン,n), Token(ト,t))
res38: Seq[org.nlp4l.lm.Token] = List(Token(テ,te), Token(キス,x), Token(ト,t))
res39: Seq[org.nlp4l.lm.Token] = List(Token(コ,co), Token(ミュ,mmu), Token(ニ,ni), Token(ケー,ca), Token(ショ,tio), Token(ン,n))
res40: Seq[org.nlp4l.lm.Token] = List(Token(エ,e), Token(ン,n), Token(ト,t), Token(リー,ree))
```

### 日本語Wikipediaからカタカナ語と英単語のペアを抽出する{#useNLP_hmm_lw}

前述のプログラム examples/trans_katakana_alpha.scala はカタカナ語から英単語を出力するプログラムですが、出力された英単語はあくまでも推定なので、あっているかもしれませんし、間違っているかもしれません。そのため、そのままでは Lucene/Solr の類義語辞書には使えません。しかし、大量の文書からお互いに近い距離に出現するカタカナ語とアルファベット文字列のペアを拾い、両者が同じ意味を持つかどうかを調べるために推定値を使うことはできます。拾ったカタカナ語から英単語を推定して拾ったアルファベット文字列と似ていれば拾ったカタカナ語とアルファベット文字列は互いに同じ意味を持つとして類義語辞書のエントリとして使えると考えてよいでしょう。

[Wikipediaデータの入手とインデックスの作成](#getCorpora_wiki)で日本語Wikipediaから作成したLuceneインデックス /tmp/index-jawiki にはka_pairというフィールドが定義されており、そのフィールドには互いに近い距離で出現したカタカナ語とアルファベット文字列が登録されています。このフィールドを対象にプログラム LoanWordsExtractor を実行すればカタカナ語とその語源であるアルファベット文字列のペアを抽出できます。

```shell
$ java -Dfile.encoding=UTF-8 -cp "target/pack/lib/*" org.nlp4l.syn.LoanWordsExtractor --index /tmp/index-jawiki --field ka_pair loanwords.txt
```

このプログラムを実行しただけだと重複する行がある恐れがあるので、次のように SynonymRecordsUnifier を実行します。

```shell
$ java -Dfile.encoding=UTF-8 -cp "target/pack/lib/*" org.nlp4l.syn.SynonymRecordsUnifier loanwords.txt
```

最終的に得られたファイル loanwords.txt_checked は Lucene/Solr の類義語辞書として適用できます。

## 連語分析モデル{#useNLP_collocanalysis}

NLP4Lでは、コーパス中に発生するある単語に注目したとき、その単語の前後に発生する単語を発生頻度付きで分析することができます。この分析を行うデータモデルをNLP4Lでは連語分析モデル（CollocationalAnalysisModel）と名付けました。連語分析モデルを使うと、ある動詞と共起しやすい前置詞などがわかります。たとえば英語学習者が英語コーパスを連語分析モデルを使って調べることで、よく使われる言い回しを得ることができるでしょう。

examples/colloc_analysis_brown.scala はブラウンコーパスを分析して単語"found"の前後に出現しやすい単語を出現順に表示します。examples/colloc_analysis_brown.scala の出力を見やすく表にすると、次のようになります。

| 3単語前 | 2単語前 | 1単語前 | 注目している単語 | 1単語後 | 2単語後 | 3単語後 |
|:------------:|:------------:|:------------:|:--------:|:------------:|:------------:|:------------:|
|       the(36)|        to(26)|        be(60)|     found|        in(77)|       the(46)|       the(28)|
|       and(13)|        he(20)|        he(50)|          |         a(43)|        in(24)|        to(19)|
|         a(12)|       the(17)|       was(32)|          |      that(42)|         a(13)|        in(15)|
|        of(11)|       and(15)|       and(26)|          |       the(38)|        be(13)|        of(13)|
|        is( 9)|      have(13)|      been(26)|          |        it(28)|        to(12)|       and( 9)|
|       was( 8)|       can(10)|         i(26)|          |        to(28)|       and(11)|         a( 8)|
|        he( 7)|         i(10)|       she(23)|          |   himself(13)|        of(11)|      with( 8)|
|        in( 7)|       has( 9)|      have(22)|          |       out(12)|        he( 7)|       had( 7)|
|      that( 7)|        of( 9)|       had(21)|          |       him( 9)|        at( 6)|        be( 6)|
|        as( 5)|     could( 8)|      they(19)|          |    myself( 9)|       had( 5)|       men( 4)|

"found in" というフレーズが多く発生していることがわかります。また、注目している単語 "found" の前の単語もわかるので、"be found" や "to be found", "can be found" といったフレーズも発生している可能性がうかがえます。

次の例はlivedoorニュースコーパスのうち、「ITライフハック」という記事カテゴリを用いて「高」という単語について連語分析を行った結果です。

| 3単語前 | 2単語前 | 1単語前 | 注目している単語 | 1単語後 | 2単語後 | 3単語後 |
|:------------:|:------------:|:------------:|:--------:|:------------:|:------------:|:------------:|
|         の(17)|         で(11)|         の(54)|         高|         さ(66)|         な(36)|         の(16)|
|         を(10)|       できる( 9)|         は(25)|          |       解像度(51)|         の(27)|         を(16)|
|         に( 9)|         に( 9)|         で(16)|          |        精細(32)|        mm(21)|         が(13)|
|        充電( 8)|        可能( 9)|         だ(12)|          |        機能(30)|         で(18)|     バッテリー( 8)|
|        mm( 7)|       奥行き( 9)|         な(12)|          |        品質(19)|         を(17)|        重量( 8)|
|         も( 7)|        mm( 7)|         に(11)|          |        容量(12)|         が(16)|       上げる( 7)|
|        対応( 7)|         が( 7)|         も(10)|          |        音質(10)|         化(13)|         重( 7)|
|         た( 6)|         し( 7)|        万能( 8)|          |       高性能( 9)|    ディスプレイ(12)|         に( 6)|
|       奥行き( 6)|         て( 6)|         が( 6)|          |       コスト( 8)|        性能(12)|         し( 5)|
|         4( 5)|         の( 6)|         を( 6)|          |        収益( 7)|         に( 8)|   パフォーマンス( 5)|

「ITライフハック」というカテゴリらしく、「高さ」という一般的な単語に次いで、「高解像度」「高精細」「高機能」などが頻出していることがうかがえます。

なおこの結果は、examples/colloc_analysis_ldcc.scala というサンプルスクリプトを実行すると得られます。

## 固有表現抽出{#useNLP_nee}

NLP4Lは他のNLPツールの固有表現抽出機能と連携するためのクラスを提供しています。これを使うと、Luceneインデックス作成時にあるフィールドの文章から固有表現を抽出して他のフィールドにコピーすることができます。これにより、検索のための新しい絞り込み軸を提供することができます。

NLP4Lは現在、Apache OpenNLPと連携して固有表現抽出を行うOpenNLPExtractorクラスを提供しています。

すでに紹介した CSVImporter コマンドはオプション指定するだけでOpenNLPExtractorクラスを使って固有表現抽出機能が適用できますので、ここではその方法を紹介しましょう。

例として次のようなCSVファイルがあるとします。

```shell
$ cat << EOF > /tmp/data.csv
1, SPORTS, "The Washington Nationals have released right-handed pitcher Mitch Lively."
2, SPORTS, "Chris Heston who no hit the New York Mets on June 9th."
3, SPORTS, "Mark Warburton wants to make veteran midfielder John Eustace the next captain of Rangers."
EOF
```

またスキーマファイルは次のようになっているとします。

```shell
$ cat << EOF > /tmp/schema.conf
schema {
  defAnalyzer {
    class : org.apache.lucene.analysis.standard.StandardAnalyzer
  }
  fields　= [
    {
      name : id
      indexed : true
      stored : true
    }
    {
      name : cat
      indexed : true
      stored : true
    }
    {
      name : body
      analyzer : {
        tokenizer {
          factory : standard
        }
        filters = [
          {
            factory : lowercase
          }
        ]
      }
      indexed : true
      stored : true
      termVector : true
      positions : true
      offsets : true
    }
    {
      name : body_person
      indexed : true
      stored : true
    }
  ]
}
EOF
```

スキーマにはbodyフィールドと、そのbodyフィールドから抽出した人名を登録するためのbody_personフィールドを用意しています。

また、OpenNLPのモデルファイルをダウンロードしておく必要があります。

```shell
$ mkdir models
$ cd models
$ wget http://opennlp.sourceforge.net/models-1.5/en-sent.bin
$ wget http://opennlp.sourceforge.net/models-1.5/en-token.bin
$ wget http://opennlp.sourceforge.net/models-1.5/en-ner-person.bin
$ cd ..
```

そしてCSVImporter を次のように--neeModelsと--neeFieldオプション付きで実行します。

```shell
$ java -cp "target/pack/lib/*" org.nlp4l.core.CSVImporter --index /tmp/index-nee --schema /tmp/schema.conf --fields id,cat,body --neeModels models/en-sent.bin,models/en-token.bin,models/en-ner-person.bin --neeField body /tmp/data.csv
```

では固有表現抽出ができたかどうか確認します。（後述の）インデックスブラウザ機能を使ってbody_personフィールドの中を確認します。

```shell
$ ./target/pack/bin/nlp4l
nlp4l> open("/tmp/index-nee")
nlp4l> status

========================================
Index Path       : /tmp/index-nee
Closed           : false
Num of Fields    : 4
Num of Docs      : 3
Num of Max Docs  : 3
Has Deletions    : false
========================================
        
Fields Info:
========================================
  # | Name        | Num Terms 
----------------------------------------
  0 | id          |          3
  1 | cat         |          1
  2 | body        |         34
  3 | body_person |          4
========================================

nlp4l> browseTerms("body_person")

nlp4l> nt
Indexed terms for field 'body_person'
Chris Heston (DF=1, Total TF=1)
John Eustace (DF=1, Total TF=1)
Mark Warburton (DF=1, Total TF=1)
Mitch Lively (DF=1, Total TF=1)
```

結果は上のようになり、確かに人名が抽出されています。

## バディワード抽出{#useNLP_buddy}

BuddyWordsFinder を使うと、Lucene インデックス中のある単語と近くで共起する単語を抽出できます。便宜的に NLP4L ではそのような単語を「バディワード」と呼びます。基になったアイディアは[LUCENE-474](https://issues.apache.org/jira/browse/LUCENE-474)です。

```shell
$ java -cp "target/pack/lib/*" org.nlp4l.colloc.BuddyWordsFinder

Usage:
BuddyWordsFinder
       --index <index dir>
       --field <field name>
       [--srcField <source field name>]
       [--text]                          specify this option when you want a text for output instead of index
       [--maxDocsToAnalyze <num>]        default is 1000
       [--slop <num>]                    default is 5
       [--maxCoiTermsPerTerm <num>]      default is 20
       [--maxBaseTermsPerDoc <num>]      default is 10000
       out_index                         specify output index directory (or name of text file when you use --text)
```

最低限必要な設定は Lucene インデックスとフィールド名です。また、抽出結果は別の Lucene インデックス（デフォルト）またはテキストファイル（--text を指定した場合）に出力されます。

たとえば、ブラウンコーパスの body フィールドにおけるバディワードを抽出して結果を out.txt ファイルに出力するには次のように実行します。

```shell
$ java -cp "target/pack/lib/*" org.nlp4l.colloc.BuddyWordsFinder --index /tmp/index-brown --field body --text out.txt
```

出力結果は次のようになります。

```shell
$ head -n 30 out.txt 
15th => 16th,poets
16th => 15th,poets
accustomed => artist,studying
advances => quit
adventure => sparkling,pair
adventuring => masks,uncle
ages => vaguely,wondered
animal's => reconstruct,bone,artist
ankle => bosom,calf,forgive,magnificent,artist,perfect
antique => vieux,shops,orleans,impressed,uncle,store
antiques => cavernous,cluttered,echoed,heels,depth
appetite => tantalizing,calf,magnificent
appraisingly => coldly,uncertain
appreciates => joys,signs
approached => cautiously,darkness
arrangement => paint,quarter
artist => animal's,ankle,accustomed,studying
ashamed => smack,nephew
aunt => subsequently,cheeks,mistaken,uncle,nearby,combination,december
ball => masks,romance
bear => encircled,crush
bent => napped,kissed,pink
bitch => timing,exclaimed,loyal,cheap
blooming => everlastingly,plants,flowers
bodied => sexy,wine,rare
bolt => securely,slip
bone => animal's,reconstruct,scientists
bore => expectation,fun
bosom => mind's,ankle,calf
bride => indisposed,store
```

livedoor ニュースコーパスでは次のようになります。

```shell
$ head -n 30 out.txt 
250 => synth,セール,ミュージック,for,カテゴリ,ipad,限定,期間
303 => synth,mi,for
app => synth,itunes,apple,for,id,com,ipad,http
apple => synth,itunes,app,store,com,http
com => synth,itunes,app,apple,for,store,http
for => synth,隣る,303,mt,250,app,id,com,ipad,限定,期間
http => itunes,app,apple,store,com,以降
id => synth,mt,app,for,ipad
inc => ミュージック,カテゴリ,条件,バージョン
ios => 互換,条件,ipad,以降
ipad => シンセサイザーアプリ,オシレータ,synth,シンセサイザ,プリセット,セール,統合,互換,mt,250,app,for,背景,ios,本格,ボディ,id,条件,バージョン,以降
itunes => app,apple,store,com,以降,http
mi => シンセサイザーアプリ,シンセサイザ,303
mt => synth,for,id,ipad
store => itunes,apple,com,以降,http
synth => シーケンサ,隣る,303,mt,250,app,apple,for,id,com,ipad,期間
いじる => 個性,色々,見つける
さっそく => シンセサイザーアプリ,シンセサイザ
じれる => カットオフ,芸,個別,細かい
ちょっとした => シーケンサ,隣る,芸,パターン,細かい
ならでは => 強み,音
まだまだ => 使いこなせる
もう少し => やんちゃ,フィルタ,効く
やんちゃ => フィルタ,もう少し,効く
アクセント => ゲート,凝る,個別,タイム
エンベロープ => パラメータ,フィルタ,贅沢,上手い,並ぶ,各,以外
オシレータ => 波形,オーソドックス,プリセット,フィルタ,ipad
オーソドックス => オシレータ,飛び道具,波形,プリセット
カットオフ => じれる,芸,個別
カテゴリ => セール,ミュージック,250,inc,限定,期間
```

## 専門用語抽出{#useNLP_te}

Lucene インデックスから専門用語を抽出するツール TermsExtractor を紹介します。TermsExtractor の実装は論文「[出現頻度と連接頻度に基づく専門用語抽出](http://www.r.dl.itc.u-tokyo.ac.jp/~nakagawa/academic-res/jnlp10-1.pdf)」に基づいて行われています。

専門用語とは、ある文書に頻出する1つ以上の単語の連なりです。品詞情報まで参照するため、現在は JapaneseTokenizer が使える日本語のみ対応しています。

TermsExtractor を実行するには、専門用語を抽出したい文書（のフィールド）を Luceneインデックスに登録する必要があります。またそのLuceneフィールドは、CompoundNounCnAnalyzer、CompoundNounRn2Analyzer、CompoundNounLn2Analyzerという3つのLucene Analyzerでトークナイズされている必要があります。

サンプルとして、livedoor ニュースコーパスの sports-watch カテゴリの文書から専門用語抽出をしてみましょう。examples/index_ldcc_sports-watch.scala を次のように実行します。

```shell
nlp4l> :load examples/index_ldcc_sports-watch.scala
```

すると、/tmp/index-ldcc-sports-watch という Luceneインデックスができます。このインデックスをインデックスブラウザで見てみると、body というフィールド以外に body_rn2 と body_ln2 というフィールドができていることがわかるでしょう。これらのフィールドはそれぞれ CompoundNounCnAnalyzer、CompoundNounRn2Analyzer、CompoundNounLn2Analyzer によってトークナイズされています。

```shell
nlp4l> open("/tmp/index-ldcc-sports-watch")

nlp4l> status

========================================
Index Path       : /tmp/index-ldcc-sports-watch
Closed           : false
Num of Fields    : 7
Num of Docs      : 900
Num of Max Docs  : 900
Has Deletions    : false
========================================
        
Fields Info:
========================================
  # | Name     | Num Terms 
----------------------------------------
  0 | body_ln2 |      28032
  1 | body     |      25249
  2 | url      |        900
  3 | date     |        898
  4 | title    |       2968
  5 | cat      |          1
  6 | body_rn2 |      28032
========================================
```

このインデックス /tmp/index-ldcc-sports-watch とフィールド body を対象に、TermsExtractor で専門用語抽出するには、次のように実行します。

```shell
$ java -cp "target/pack/lib/*" org.nlp4l.extract.TermsExtractor --field body --out terms.txt /tmp/index-ldcc-sports-watch
```

出現頻度と連接頻度に基づくスコア計算がなされ、スコアの高い順にソートされて結果が--out オプションで指定したテキストファイル terms.txt に出力されます。

```shell
$ head terms.txt
日本, 130058.945313
選手, 86330.593750
日本代表, 58211.601563
試合, 55407.371094
放送, 55291.871094
一人, 38042.597656
一戦, 26854.021484
番組, 24922.732422
監督, 23383.238281
チーム, 21691.521484
```

## 仮説検定{#useNLP_hypothesistesting}

先行研究では非母語話者の書いた英語にはIやyouが多用されがちで、書き手・読み手の可視性が母語話者以上に顕著であるとされます（[1]の3章、[2]）。

```shell
[2] Petch-Tyson, S. (1998). Writer/reader visibility in EFL written discourse. In S. Granger (Ed.), Learner English on Computer (pp. 107-118). London, UK: Longman.
```

そこでここではCEEAUSのデータを使って、日本人英語学習者は英作文で1、2人称代名詞を過剰使用していないことを帰無仮説としてα=.05でカイ2乗検定を行ってみましょう。インデックスデータとしては/tmp/index-ceeaus-allを使いますので、前述のプログラム examples/index_ceeaus_all.scala をあらかじめ実行してインデックスを作成しておきます。次に同じくnlp4lプロンプトから examples/chisquare_test_ceeaus.scala を実行します。

```shell
nlp4l> :load examples/chisquare_test_ceeaus.scala
```

実行すると、次のような一覧が表示されます（カイ2乗統計量が[1]の3章の結果と微妙に異なりますが、単語数が違っているためです。どのようなトークナイザを使用したかによって単語数は異なってきます）。

```Shell
		CEEJUS	CEENAS	chi square
==============================================
       i         4,367     384   324.8068
      my           526     101     1.4384
      me           230      28     8.5172
     you           802      73    55.1096
    your           160      24     2.7664
```

2種類のカテゴリについて検定を多重に繰り返すため、ボンフェローニ補正を行いαb=.025を用います。統計の参考書に付録としてついているカイ2乗分布表を見ると、自由度=1、αb=.025の限界値は5.02389ですから、この値を超えるi、me、youの語については帰無仮説が棄却され、日本人英語学習者はこれらの語を英語母語話者に比べて過剰使用している（有意差がある）といえることがわかりました（書籍[1]の結果と同じです）。

では仮説検定を行うプログラムを順に見てみましょう。

```scala
// (1)
val index = "/tmp/index-ceeaus-all"

// (2)
val schema = SchemaLoader.load("examples/schema/ceeaus.conf")

// (3)
val reader = IReader(index, schema())
```

(1)でLuceneのインデックスを指定しています。(2)はLuceneインデックスのスキーマを設定しています。(3)は(1)(2)で設定したインデックスとスキーマ情報を渡し、IReaderオブジェクトを作成して定数readerに代入しています。readerを通じてインデックス（コーパス）にアクセスできます。

次の行からは早速そのreaderを使ってインデックスのサブセットを取得しています。

```scala
// (4)
val docSetJUS = reader.subset(TermFilter("file", "ceejus_all.txt"))
val docSetNAS = reader.subset(TermFilter("file", "ceenas_all.txt"))
```

インデックスのサブセットを取得するには、IReaderのsubset()関数にFilterを渡して行います。ここではFilterとしてTermFilterを使っています。TermFilterは2つの引数を取り、最初の引数でLuceneインデックスのフィールド名を指定し、2つめの引数でそのフィールド値を指定します。このようにすることで、Luceneインデックスの中から該当するフィールド値を持つ文書セットだけを指定できます。(4)の1行目はファイル名がceejus_all.txtとなっているものを指定しているので、日本人英語学習者の英作文だけを取り出しています。2行目はファイル名がceenas_all.txtとなっているものを指定しているので、英語母語話者の英作文を対象にしています。

次にWordCountsオブジェクトのtotalCount()関数を使ってbody_enフィールド中の全単語数を数えています。

```scala
// (5)
val totalCountJUS = WordCounts.totalCount(reader, "body_en", docSetJUS)
val totalCountNAS = WordCounts.totalCount(reader, "body_en", docSetNAS)
```

このとき、(4)で取得した文書セットを第3引数で渡すことで、それぞれ日本人英語学習者と英語母語話者のカウントを分けて取得するようにしています。

次にwordsに今回注目する単語のリストを保持します。

```scala
// (6)
val words = List("i", "my", "me", "you", "your")
```

次にその注目する単語について、count()関数を使って日本人英語学習者と英語母語話者それぞれの単語数を単語ごとに数えます（単語別のカウントがMap[String,Long]で得られます）。

```scala
// (7)
val wcJUS = WordCounts.count(reader, "body_en", words.toSet, docSetJUS)
val wcNAS = WordCounts.count(reader, "body_en", words.toSet, docSetNAS)
```

最後にStatsオブジェクトのchiSquare()関数を使ってカイ2乗統計量を求めています。

```scala
// (8)
println("\t\tCEEJUS\tCEENAS\tchi square")
println("==============================================")
words.foreach{ w =>
  val countJUS = wcJUS.getOrElse(w, 0.toLong)
  val countNAS = wcNAS.getOrElse(w, 0.toLong)
  val cs = Stats.chiSquare(countJUS, totalCountJUS - countJUS, countNAS, totalCountNAS - countNAS, true)
  println("%8s\t%,6d\t%,6d\t%9.4f".format(w, countJUS, countNAS, cs))
}

// (9)
reader.close
```

chiSquare()関数の第1、3引数にはそれぞれの単語のカウント数を、第2、4引数にはそれ以外の単語のカウント数を渡します。また、第5引数にはイェーツ補正を行う（true）か否か（false）を指定します。

プログラムの最後にIReaderをclose()します(9)。

同じように、ブラウンコーパスを使って特定の単語の使用頻度に有意差があるかどうかを見てみましょう。書籍[3]2.1.3には法助動詞couldとwillの使用頻度が、前者はromanceカテゴリにおいて高く、後者はnewsカテゴリにおいて高いという結果が示されています。[NLTK web](http://www.nltk.org/book/ch02.html)

```shell
[3] Natural Language Processing with Python
Steven Bird, Ewan Klein, Edward Loper
O'Reilly

入門自然言語処理
萩原正人、中山敬広、水野貴明
オライリー・ジャパン
```

ここではもう少し突っ込んで仮説検定を行い、この使用頻度に有意差があるか計算します。同様にαb=.025を用います。実行結果は次のようになります。

```shell
nlp4l> :load examples/chisquare_test_brown.scala
       	     romance	news	chi square
==============================================
   could         195      87     101.2919
    will          49     390     148.4053
```

どちらも自由度=1、αb=.025の限界値5.02389を超えているので有意差があるといえます。プログラムは前掲のものとほとんど同じなので解説は省略します。

## 相関分析{#useNLP_correlationanalysis}

接続副詞は文内要素の論理的な結束性を担う重要な役割を果たしています（[1]の4章）。ここではCEEAUSのCEEJUS（日本人英語学習者）、CEECUS（中国人英語学習者）、CEENAS（英語母語話者）のサブコーパス間で接続副詞の使用頻度を比較し、どの程度の関係性が存在するのか見てみましょう。仮にどのような書き手であっても接続表現の使用パターンが安定しているのであれば、書き手間に高い相関関係が見えるでしょうし、逆に母語話者かどうかで使用頻度が変わるのであれば、相関係数は低くなるでしょう。

では早速プログラムを実行してみます。同じくインデックスデータとしては/tmp/index-ceeaus-allを使いますので、前述のプログラム examples/index_ceeaus_all.scala をあらかじめ実行してインデックスを作成しておきます。次にnlp4lプロンプトから examples/correlation_analysis_ceeaus.scala を実行します。

```shell
nlp4l> :load examples/correlation_analysis_ceeaus.scala
```

実行すると2つの表が表示されます。1つめの表は次の通りです。

```shell
word counts
========================================
    word    CEEJUS    CEECUS    CEENAS
  besides,      21         5         2
nevertheless,    2         1         1
     also,      18         5         8
 moreover,      68         8         4
  however,     165        18        33
therefore,      86         7        13
       so,     242         9         5
```

wordの列にある単語が接続副詞です。各接続副詞の後ろにはカンマ（","）がついていますが、これは他の一般副詞と区別するため、接続副詞をコーパスから検索する際に後ろにカンマを伴っている単語を探すところから来ています。数字は各サブコーパスにおける出現頻度を示しています。ただし出現頻度を単純に比較するだけでは各サブコーパス間で相関性があるのかはわかりません。そこで2つめの表を見てみます。

```shell
Correlation Coefficient
================================
	CEEJUS	CEECUS	CEENAS
CEEJUS	1.000	0.690	0.433
CEECUS	0.690	1.000	0.886
CEENAS	0.433	0.886	1.000
```

2つめの表は単相関行列表を示しています。この結果から、日本人英語学習者と英語母語話者は中程度の正の相関が認められ（r=.43）、中国人英語学習者と英語母語話者は強い正の相関が認められる（r=.89）ことがわかりました。また日本人英語学習者と中国人英語学習者は中程度の正の相関があります（r=.69）。ただしこのうち相関係数が有意となったものは中国人英語学習者と英語母語話者の相関のみです（[1]の4章）。

では相関分析を行うプログラム examples/correlation_analysis_ceeaus.scala を順に見てみましょう。プログラムの最初の部分（インデックスの指定、スキーマの設定、IReader オブジェクトの作成）は前述の仮説検定で説明した(1)から(3)と同じなので省略します。

プログラムの次の部分(4)はやはり仮説検定のところでも使ったテクニックで、インデックスから各サブコーパスを対象に処理ができるように指定をしています。ここでは指定するサブコーパスに CEECUS を追加しています。

```scala
// (4)
val docSetJUS = reader.subset(TermFilter("file", "ceejus_all.txt"))
val docSetCUS = reader.subset(TermFilter("file", "ceecus_all.txt"))
val docSetNAS = reader.subset(TermFilter("file", "ceenas_all.txt"))
```

次にwordsに今回注目する単語のリストを保持します(5)。

```scala
// (5)
val words = List("besides,", "nevertheless,", "also,", "moreover,", "however,", "therefore,", "so,")
```

各単語には一般副詞と区別するために後ろにカンマをつけています。このリストを次のWordCountsのcount()関数に指定することでこれらの単語だけに注目してそれぞれのコーパスから単語の出現頻度を取得します。

```scala
// (6)
val wcJUS = WordCounts.count(reader, "body_ws", words.toSet, docSetJUS)
val wcCUS = WordCounts.count(reader, "body_ws", words.toSet, docSetCUS)
val wcNAS = WordCounts.count(reader, "body_ws", words.toSet, docSetNAS)
```

ここで注目していただきたいのが対象フィールドにbody_wsを指定している点です。前述の仮説検定ではbody_enフィールドを対象にしていました。

スキーマ設定ファイル examples/schema/ceeaus.confを見ると、body_enはLuceneのStandardAnalyzerを使って作成した(7)を、body_wsはLuceneのWhitespaceTokenizerにLowerCaseFilterを組み合わせて作成した(8)を使って単語分割されています。

```json
schema {
  defAnalyzer {
    class : org.apache.lucene.analysis.standard.StandardAnalyzer
  }
  fields　= [
    {
      name : file
      indexed : true
      stored : true
    }
    {
      name : type
      indexed : true
      stored : true
    }
    {
      name : cat
      indexed : true
      stored : true
    }
    {
      name : body_en  // (7)
      analyzer : {
        tokenizer {
          factory : standard
        }
        filters = [
          {
            factory : lowercase
          }
        ]
      }
      indexed : true
      stored : true
      termVector : true
      positions : true
    }
    {
      name : body_ws  // (8)
      analyzer : {
        tokenizer {
          factory : whitespace
        }
        filters = [
          {
            factory : lowercase
          }
        ]
      }
      indexed : true
      stored : true
      termVector : true
      positions : true
    }
    {
      name : body_ja
      analyzer : {
        class : org.apache.lucene.analysis.ja.JapaneseAnalyzer
      }
      indexed : true
      stored : true
      termVector : true
      positions : true
    }
  ]
}
```

body_wsとbody_enの単語分割の違いを見てみましょう。examples/correlation_analysis_ceeaus.scala プログラムを実行した状態の nlp4l のプロンプトから次のようにしてスキーマのそれぞれのフィールドから Analyzer を取得します。

```shell
nlp4l> val analyzerWs = schema.getAnalyzer("body_ws").get
nlp4l> val analyzerEn = schema.getAnalyzer("body_en").get
```

次にbody_wsフィールドに設定されているanalyzerWsを使って"Also, when we enter college, we are no longer children"という文を単語分割します。

```shell
nlp4l> analyzerWs.tokens("Also, when we enter college, we are no longer children")
res19: org.nlp4l.core.analysis.Tokens = Tokens(Map(startOffset -> 0, type -> word, positionIncrement -> 1, endOffset -> 5, term -> also,, positionLength -> 1, bytes -> [61 6c 73 6f 2c]), Map(startOffset -> 6, type -> word, positionIncrement -> 1, endOffset -> 10, term -> when, positionLength -> 1, bytes -> [77 68 65 6e]), Map(startOffset -> 11, type -> word, positionIncrement -> 1, endOffset -> 13, term -> we, positionLength -> 1, bytes -> [77 65]), Map(startOffset -> 14, type -> word, positionIncrement -> 1, endOffset -> 19, term -> enter, positionLength -> 1, bytes -> [65 6e 74 65 72]), Map(startOffset -> 20, type -> word, positionIncrement -> 1, endOffset -> 28, term -> college,, positionLength -> 1, bytes -> [63 6f 6c 6c 65 67 65 2c]), Map(startOffset -> 29, type -> word, positionIn...
```

LuceneのAnalyzerは単語分割する際に分割された単語文字列だけでなく、文章内の単語の位置情報や種類など、非常に多くの情報を付加してくれます。そのためこのままだと表示が見にくいので、次のように少し工夫します（Analyzerの使い方は本書で後述します）。

```shell
nlp4l> analyzerWs.tokens("Also, when we enter college, we are no longer children.").map(_.get("term").get).foreach(println(_))
also,
when
we
enter
college,
we
are
no
longer
children.
```

同様に、analyzerEnを使って同じ文章を単語分割してみます。

```shell
nlp4l> analyzerEn.tokens("Also, when we enter college, we are no longer children.").map(_.get("term").get).foreach(println(_))
also
when
we
enter
college
we
are
no
longer
children
```

これらの結果から、analyzerWsを使っているbody_wsフィールドでは単語の後ろについているカンマやピリオドを取り除かないが、analyzerEnを使っているbody_enフィールドでは単語の後ろについているこれらの記号を取り除いていることがわかります。

したがって、"also,"という接続副詞としてのalsoのみを検索したい今回はbody_enではなくbody_wsを処理対象にしています。

ではプログラムに戻りましょう。次の部分(9)では1つめの表である単語出現頻度を表示しています。

```scala
// (9)
println("\n\n\nword counts")
println("========================================")
println("\tword\tCEEJUS\tCEECUS\tCEENAS")
words.foreach{ e =>
  println("%10s\t%,6d\t%,6d\t%,6d".format(e, wcJUS.getOrElse(e, 0), wcCUS.getOrElse(e, 0), wcNAS.getOrElse(e, 0)))
}
```

プログラムの最後の部分(10)では単相関行列表を表示しています。行列表なので、2重ループのプログラムになっています。

```scala
// (10)
val lj = List( ("CEEJJS", wcJUS), ("CEECUS", wcCUS), ("CEENAS", wcNAS) )
println("\n\n\nCorrelation Coefficient")
println("================================")
println("\tCEEJUS\tCEECUS\tCEENAS")
lj.foreach{ ej =>
  print("%s".format(ej._1))
  lj.foreach{ ei =>
    print("\t%5.3f".format(Stats.correlationCoefficient(words.map(ej._2.getOrElse(_, 0.toLong)), words.map(ei._2.getOrElse(_, 0.toLong)))))
  }
  println
}

// (11)
reader.close
```

相関係数を計算するためにStatsオブジェクトのcorrelationCoefficient()関数を使って求めています。correlationCoefficient()関数には相関を求めたい2つの単語の頻度ベクトルを渡します。

プログラムの最後にreaderをcloseしています(11)。

同じように、ブラウンコーパスを使ってカテゴリ間の相関分析を行ってみます。書籍[3]2.1.3に紹介されている法助動詞（"can", "could", "may", "might", "must", "will"）について調べます。プログラム examples/correlation_analysis_brown.scala を実行すると、同様に単語頻度表、単相関行列表が表示されます。

```shell
nlp4l> :load examples/correlation_analysis_brown.scala

word counts
========================================
word       gov  news  romance  SF
     can   119    94       79  16
   could    38    87      195  49
     may   179    93       11   4
   might    13    38       51  12
    must   102    53       46   8
    will   244   390       49  17


Correlation Coefficient
================================
            gov   news  romance  SF
gov        1.000  0.789 -0.500 -0.385
news       0.789  1.000 -0.127  0.029
romance   -0.500 -0.127  1.000  0.980
SF        -0.385  0.029  0.980  1.000
```

2つめの表より、governmentとnews および romanceとscience_fiction が互いに強い正の相関があることがわかります。プログラムは前掲のものとほぼ同じなので解説は省略します。

# インデックスブラウザを使う{#indexBrowser}

NLP4L の対話型シェルには、CUIのLucene インデックスブラウザが付属しています。インデックスブラウザを使うと、Javaプログラムを書かずに、Luceneインデックス中のフィールドや単語の情報を簡単に閲覧、デバッグができます。

ここでは、前の節で作成した、Livedoorニュースコーパスのインデックスの中をブラウズしてみましょう。

なお、インデックスブラウザのコマンド（メソッド）一覧は、nlp4lのプロンプトに":?"と入力すると見ることができます。

```shell
nlp4l> :?
```

また、":?" のうしろにコマンドをつけると、さらに詳しいヘルプが表示されます。

```shell
nlp4l> :? open
-- method signature --
def open(idxDir: String): RawReader

-- description --
Open Lucene index in the directory. If an index already opened, that is closed before the new index will be opened.

-- arguments --
idxDir       Lucene index directory

-- return value --
Return : index reader

-- usage --
nlp4l> open("/tmp/myindex")
```

## フィールド、単語のブラウジング{#indexBrowser_fields}

open() 関数にインデックスディレクトリのパスを渡すことでインデックスをオープンします。

```shell
nlp4l> open("/tmp/index-ldcc")
Index /tmp/index-ldcc was opened.
res4: org.nlp4l.core.RawReader = IndexReader(path='/tmp/index-ldcc',closed=false)
```

close で、現在オープンしているインデックスをクローズします。

```shell
nlp4l> close
Index /tmp/index-ldcc was closed.
```

status とタイプすると、現在オープンしているインデックスについて、ドキュメント数やフィールドの情報、各フィールドに含まれるユニークな単語数などの概略が表示されます。

```shell
nlp4l> open("/tmp/index-ldcc")
Index /tmp/index-ldcc was opened.
res4: org.nlp4l.core.RawReader = IndexReader(path='/tmp/index-ldcc',closed=false)

nlp4l> status

========================================
Index Path       : /tmp/index-ldcc
Closed           : false
Num of Fields    : 5
Num of Docs      : 7367
Num of Max Docs  : 7367
Has Deletions    : false
========================================
        
Fields Info:
========================================
  # | Name  | Num Terms 
----------------------------------------
  0 | body  |      64543
  1 | url   |       7367
  2 | date  |       6753
  3 | title |      14205
  4 | cat   |          9
========================================
```

さらに、browseTerms() 関数に、フィールド名を渡すと、nextTerms および prevTerms 関数で、フィールド中に含まれる単語の情報を閲覧できるようになります。

nextTerms(), prevTerms() には、スキップするページ数を渡すことができます。また、nextTerms(1) には nt, prevTerms(1) には pt というショートカット関数が定義されています。

title フィールド中に含まれる単語を閲覧してみましょう。単語は辞書順で並んでおり、各行の冒頭がインデックスされている単語、DF はその単語を含むドキュメント数(document frequency)、Total TF (term frequency) はその単語の出現回数の総計を表します。

```shell
nlp4l> browseTerms("title")
Browse terms for field 'title', page size 20
Type "nextTerms(skip)" or "nt" to browse next terms.
Type "prevTerms(skip)" or "pt" to browse prev terms.
Type "topTerms(n)" to find top n frequent terms.

// nt で次のページ
nlp4l> nt
Indexed terms for field 'title'
0 (DF=152, Total TF=176)
000 (DF=13, Total TF=13)
003 (DF=3, Total TF=3)
0048 (DF=1, Total TF=1)
007 (DF=8, Total TF=8)
...

// 何度かページング, または nextTerms(n)でスキップすると、アルファベットから始まる単語が見える
nlp4l> nt
Indexed terms for field 'title'
cocorobo (DF=1, Total TF=1)
code (DF=1, Total TF=1)
coin (DF=1, Total TF=1)
collection (DF=3, Total TF=3)
...

// pt で前のページに戻る
nlp4l> pt
Indexed terms for field 'title'
chat (DF=1, Total TF=1)
check (DF=2, Total TF=2)
chochokure (DF=1, Total TF=1)
christian (DF=1, Total TF=1)
...
```

1ページに表示される単語数はデフォルトで20件ですが、browseTerms("title",100)のように第２引数でページサイズを指定することもできます。

さらに、browseTermDocs() 関数に、フィールド名と単語を渡すと、nextDocs および prevDocs 関数で、指定されたフィールドにその単語を含むドキュメントの情報を閲覧できるようになります。

nextDocs(), prevDocs() には、スキップするページ数を渡すことができます。また、nextDocs(1) には nd, prevDocs(1) には pd というショートカット関数が定義されています。

title フィールドに "iphone" という単語を含むドキュメントを閲覧してみましょう。ドキュメントはLucene内部のドキュメントID順で並んでおり、idはドキュメントID、freq はドキュメント中に指定の単語（ここでは"iphone"）が出現する頻度を表します。

インデックス時に position や offset (その単語がドキュメント中のどこに出現するかの情報。後述) を保存していると、これらの情報もあわせて表示されます。

```shell
nlp4l> browseTermDocs("title", "iphone")
Browse docs for term 'iphone' in field 'title', page size 20
Type "nextDocs(skip)" or "nd" to browse next terms.
Type "prevDocs(skip)" or "pd" to browse prev terms.

// nd で次のページ
nlp4l> nd
Documents for term 'iphone' in field 'title'
Doc(id=49, freq=1, positions=List(pos=5))
Doc(id=270, freq=1, positions=List(pos=0))
Doc(id=648, freq=1, positions=List(pos=0))
Doc(id=653, freq=1, positions=List(pos=2))
Doc(id=778, freq=1, positions=List(pos=2))
Doc(id=780, freq=2, positions=List(pos=0, pos=15))
...

// pd で前のページに戻る
nlp4l> pd
Documents for term 'iphone' in field 'title'
Doc(id=1173, freq=1, positions=List(pos=1))
Doc(id=1176, freq=1, positions=List(pos=0))
Doc(id=1180, freq=1, positions=List(pos=2))
Doc(id=1195, freq=1, positions=List(pos=5))
Doc(id=1200, freq=1, positions=List(pos=11))
Doc(id=1203, freq=1, positions=List(pos=5))
...
```

1ページに表示されるドキュメント数はデフォルトで20件ですが、browseTermDocs("title","iphone",100)のように第３引数でページサイズを指定することもできます。

## ドキュメントの閲覧{#indexBrowser_docs}

より詳しくドキュメントの内容を見たい場合は、showDoc() 関数にドキュメントIDを渡すことでフィールド値を表示させることができます。

前章の browseTermDocs / nd / pd 関数で取得したドキュメントID (ここでは id=1195) に対応するドキュメント内容を表示してみましょう。"iPhone"という単語が、title フィールドに出現することが確認できます。

```shell
nlp4l> showDoc(1195)
Doc #1195
(Field) cat: [it-life-hack]
(Field) url: [http://news.livedoor.com/article/detail/6608703/]
(Field) title: [GoogleドライブのファイルをiPhoneからダイレクトに編集する【知っ得！虎の巻】]
...
```

## Position / Offsets{#indexBrowser_posoff}

Position / Offsets はインデックスに付加的に保存できる情報で、各単語のドキュメント中の出現箇所を示します。

たとえば、以下の例のドキュメントID=199のドキュメントに着目すると、

* body フィールドに "北海道" という単語が2回出現する
* 最初の出現は 126 単語め(position=126)で、詳細な文字位置は 247-250
* 2回めの出現は 129 単語め(position=129)で、詳細な文字位置は 255-258

ということがわかります。

```shell
nlp4l> browseTermDocs("body","北海道")
Browse docs for term body in field 北海道, page size 20
Type "nextDocs(skip)" or "nd" to browse next terms.
Type "prevDocs(skip)" or "pd" to browse prev terms.

nlp4l> nd
Documents for term '北海道' in field 'body'
Doc(id=148, freq=1, positions=List((pos=491,offset={997-1000})))
Doc(id=199, freq=2, positions=List((pos=126,offset={247-250}), (pos=129,offset={255-258})))
...
```

## 出現頻度の高い Top N 単語の抽出{#indexBrowser_topn}

browseTerms() でフィールドを指定した後、topTerms() 関数でそのフィールドでドキュメント出現頻度(DF)の高い上位N件の単語とその頻度を取得できます。

```shell
nlp4l> browseTerms("title")
Browse terms for field title, page size 20
Type "nextTerms(skip)" or "nt" to browse next terms.
Type "prevTerms(skip)" or "pt" to browse prev terms.
Type "topTerms(n)" to find top n frequent terms.

nlp4l> topTerms(10)
Top 10 frequent terms for field title
  1: 話題 (DF=587, Total TF=607)
  2: sports (DF=493, Total TF=493)
  3: watch (DF=493, Total TF=493)
  4: 映画 (DF=373, Total TF=402)
  5: 女 (DF=319, Total TF=356)
  6: 1 (DF=318, Total TF=342)
  7: android (DF=307, Total TF=322)
  8: 女子 (DF=301, Total TF=318)
  9: 3 (DF=299, Total TF=323)
 10: アプリ (DF=291, Total TF=337)
```

# Solrユーザの皆様{#dearSolrUsers}

# Elasticsearchユーザの皆様{#dearESUsers}

# Mahoutと連携する{#useWithMahout}

# Sparkと連携する{#useWithSpark}

この節では、事前に Spark 1.3.0 以上がインストールされていることが必要です。(1.3.0 以前の Spark をお使いの場合は、適宜読み替えてください。)

## MLLibと連携する{#useWithSpark_mllib}

NLP4L でコーパスの特徴量を抽出し、 Spark MLlib への入力として与えることができます。

### サポートベクトルマシンで文書分類{#useWithSpark_svm}

Spark MLlib にはいくつかの分類器が提供されていますが、ここではサポートベクトルマシン（SVM）を用いてlivedoorニュースコーパス（ldcc）を文書分類する方法を紹介します。

まだ実行していない場合は examples/index_ldcc.scala を実行して ldcc のコーパスをLuceneインデックスに用意します。次に examples/extract_ldcc.scala を使ってLuceneインデックスから2つのカテゴリー"dokujo-tsushin" と "sports-watch"を持つ記事だけを抽出し、/tmp/index-ldcc-partという名前の別のLuceneインデックスを作成します。

```shell
nlp4l> :load examples/extract_ldcc.scala
```

次にコマンドラインプログラム LabeledPointAdapter を次のように実行して、/tmp/index-ldcc-part から2クラスの全文書のベクトルデータを出力します。

```shell
$ java -Dfile.encoding=UTF-8 -cp "target/pack/lib/*" org.nlp4l.spark.mllib.LabeledPointAdapter -s examples/schema/ldcc.conf -f body -l cat /tmp/index-ldcc-part
```

ここで -s オプションにはスキーマ定義ファイル名を、-f オプションには特徴ベクトル抽出対象のフィールド名を、-l にはラベルが記録されているフィールド名を指定します。実行結果は labeled-point-out/ ディレクトリ以下に出力されます。labeled-point-out/data.txt ファイルは Spark MLlib への入力とすることができる libsvm 形式のファイルになっています。1カラム目が数値ラベル、2カラム目以降が特徴ベクトルです。数値ラベルと実際のラベル名の対応関係は labeled-point-out/label.txt ファイルに出力されます。

```shell
$ cat labeled-point-out/label.txt
dokujo-tsushin	0
sports-watch	1
```

ではSpark MLlib のSVMを実行してみましょう。spark-shell を起動して次のプログラムを実行します。

```scala
import org.apache.spark.SparkContext
import org.apache.spark.mllib.classification.{SVMModel, SVMWithSGD}
import org.apache.spark.mllib.evaluation.BinaryClassificationMetrics
import org.apache.spark.mllib.regression.LabeledPoint
import org.apache.spark.mllib.linalg.Vectors
import org.apache.spark.mllib.util.MLUtils

// Load training data in LIBSVM format.
val data = MLUtils.loadLibSVMFile(sc, "labeled-point-out/data.txt")

// Split data into training (70%) and test (30%).
val splits = data.randomSplit(Array(0.7, 0.3), seed = 11L)
val training = splits(0).cache()
val test = splits(1)

// Run training algorithm to build the model
val numIterations = 100
val model = SVMWithSGD.train(training, numIterations)

// Clear the default threshold.
model.clearThreshold()

// Compute raw scores on the test set.
val scoreAndLabels = test.map { point =>
  val score = model.predict(point.features)
  (score, point.label)
}

// Get evaluation metrics.
val metrics = new BinaryClassificationMetrics(scoreAndLabels)
val auROC = metrics.areaUnderROC()

println("Area under ROC = " + auROC)
```

実行結果が次のように表示されます。Area under ROCが0.9989017178259646と表示されました。

```scala
Area under ROC = 0.9989017178259646
```

### クラスタリング

Spark MLlib にはいくつかのクラスタリングアルゴリズム(k-means, Gaussian mixture 等)が実装されていますが、ここでは LDA (Latent Dirichlet allocation) を利用する例を紹介します。なお、Spark MLlib に実装されているクラスタリングアルゴリズムの詳細は [公式リファレンス (v1.3.0)](http://spark.apache.org/docs/1.3.0/mllib-clustering.html) を参照してください。

注: LDA は Spark 1.3.0 から導入されました。

コマンドラインから、プログラム VectorsAdapter を実行します。(事前に、examples/index_ldcc.scalaを実行して livedoor ニュースコーパスをインデックスしておいてください。)

```
$ java -Dfile.encoding=UTF-8 -cp "target/pack/lib/*" org.nlp4l.spark.mllib.VectorsAdapter -s examples/schema/ldcc.conf -f body --idfmode n --type int /tmp/index-ldcc
```

実行すると、vectors-out ディレクトリ以下に2つのファイルが生成されます。

* data.txt   // データファイル(カンマ区切りCSV形式)。一行目はヘッダ(単語ID)。各行の1カラム目は文書ID。
* words.txt  // 単語IDと単語の一覧

特徴量を抽出したら、Spark でLDAを実行します。spark-shell を起動し、以下のように入力してください。

```shell
$ spark-shell

scala> import org.apache.spark.mllib.clustering.LDA
scala> import org.apache.spark.mllib.linalg.Vectors
// data.txt を入力として与える
scala> val data = sc.textFile("/path/to/vectors-out/data.txt")
scala> val parsedData = data.map(s => Vectors.dense(s.trim.split(' ').map(_.toDouble)))
scala> val corpus = parsedData.zipWithIndex.map(_.swap).cache()
// K=5 を指定してモデルを作成する
scala> val ldaModel = new LDA().setK(5).run(corpus)

// 推定されたトピックを取得する. トピックは各単語の出現確率分布として表現される.
scala> val topics = ldaModel.topicsMatrix
6.582992067532604     0.12712473287343715   2.0749605231892994    ... (5 total)
1.7458039064383513    0.00886714658883468   4.228671695274331     ...
0.993056435220057     4.64132780991838      0.18921245384121668   ...
...

// 各ドキュメントがどのトピックに属するかの確率分布を取得する.
scala> val topics = ldaModel.topicDistributions
scala> topics.take(5).foreach(println)
(384,[0.13084037186524378,0.02145904901484863,0.30073967633170434,0.18175275728283377,0.36520814550536945])
(204,[0.5601036760461913,0.04276689792374281,0.17626863743620377,0.06992184061352519,0.15093894798033694])
(140,[0.01548241660044312,0.8975654153324738,0.013671563672420709,0.061526631681883964,0.011753972712778454])
(466,[0.052798328682649866,0.04602366817727088,0.7138181945792464,0.03541828992076265,0.1519415186400702])
(160,[0.20118704750574637,0.12811775189441738,0.23204896620959134,0.1428791353110324,0.29576709907921245])
```

詳しい Spark MLlib の使い方は Spark の API ドキュメントを参照してください。

# Luceneを使う{#useLucene}

## NLP4Lが提供するLuceneの機能{#useLucene_functions}

## Analyzer{#useLucene_analyzer}

## 既存インデックスを検索する{#useLucene_search}

## 簡単ファセット{#useLucene_facet}

## Term Vector から TF/IDF 値を求める{#useLucene_tfidf}

## 文書類似度を計算する{#useLucene_similarity}

## More Like This{#useLucene_mlt}

## 独自インデックスの作成{#useLucene_index}

### Schemaの定義

#### 数値型

NLP4LをNLPツールとして使う場合、IntやLongなどの数値型のフィールドを意識する必要はないかもしれません。しかし、Luceneはこれらの型をサポートしており、それぞれの型に適した形で値をインデックスに保存しています。ここでは簡単にこれらの型を扱う方法を説明します。

### 文書の登録

## FST を単語辞書として使う{#useLucene_fst}

NLP4LはLuceneのFSTを簡単な単語辞書として使えるようにシンプルにラップしたSimpleFSTを提供しています。FSTを単語辞書として使うと、対象文字列の最左部分文字列（左端から始まるすべての接頭辞）が1回の走査で探索できるため、Luceneプロジェクトでは日本語形態素解析器やシノニム検索で利用されています。

SimpleFSTを利用したサンプルプログラム examples/use_fst.scala を以下に示します。

```scala
import org.nlp4l.core._

// (1)
val index = "/tmp/index-brown"
val schema = SchemaLoader.load("examples/schema/brown.conf")
val reader = IReader(index, schema)

// (2)
val fst = SimpleFST()

// (3)
reader.field("body_pos").get.terms.foreach { term =>
  fst.addEntry(term.text, term.totalTermFreq)
}

// (4)
fst.finish

// (5)
val STR = "iaminnewyork"

// (6)
for(pos <- 0 to STR.length - 1){
  fst.leftMostSubstring(STR, pos).foreach { e =>
    print("%s".format("             ".substring(0, pos)))
    println("%s => %d".format(STR.substring(pos, e._1), e._2))
  }
}
```

このプログラムは前述のブラウンコーパスを登録したLuceneインデックスを参照し、ブラウンコーパスで使用されているすべての単語をFSTに登録します。そのため、あらかじめ examples/index_brown.scala を実行しておく必要があります。

(1)でブラウンコーパスが登録されているインデックスをオープンします。(2)でSimpleFSTのインスタンスを取得しています。(3)で、(1)でオープンしたブラウンコーパスのインデックスのbody_posフィールドから単語一覧を取得しながら、SimpleFSTのaddEntry()を使って単語とその出現頻度をSimpleFSTに登録しています。addEntry()の呼び出しは、ソート済みの文字列リストを使って順に呼び出さなければならないことに注意してください。このプログラムではソートらしきことはしていないように見えますが、Luceneインデックスには単語がソート済みの状態で登録されているため、明示的なソートは不要になっています。登録が終了したら、(4)でfinish()を呼んでいます。

(5)以降は、作成したSimpleFSTの単語辞書を使って文字列STR（"I am in New York"という文を小文字にして詰めた文字列です）にどんな単語が含まれているか、辞書引きをしながらパースしています。(6)のforループは文字列STRの走査開始位置を左から1文字ずつずらしています。最左部分文字列を走査するにはSimpleFSTのleftMostSubstring()を使います。第1引数には走査する文字列を、第2引数には捜査開始位置を渡します。戻り値は(Int,Long)のタプルのSeqで返ってきます。タプルは順に部分文字列の終了位置、単語登録時に指定されたLong値（ここでは当該単語の出現頻度）です。ためしに"welcome"という文字列を捜査開始位置0で辞書引きしてみると次のようになります。

```shell
nlp4l> fst.leftMostSubstring("welcome", 0)
res7: Seq[(Int, Long)] = List((1,5), (2,2656), (7,2705))
```

タプルが3つ返ってきました。最初のタプル(1,5)は終了位置が1の単語（つまり単語"w"）が5回、2番目のタプル(2,2656)は終了位置が2の単語（つまり単語"we"）が2656回、3番目のタプル(7,2705)は終了位置が7の単語（つまり単語"welcome"）が2705回ブラウンコーパスに出現したことを意味しています。また、終了位置がこれら以外の単語（"wel","welc","welco","welcom"）は作成した単語辞書にないこともわかります。

さて、最初のサンプルプログラムの実行結果は次のようになります。なんとなくですが、"I am in New York" という単語で区切ったときにそれぞれの単語が頻出していることが読み取れると思います。

```shell
nlp4l> :load examples/use_fst.scala
i => 5164
 a => 23195
 am => 23431
  m => 16
  mi => 18
  min => 22
   i => 5164
   in => 26500
   inn => 26508
    n => 38
     n => 38
     ne => 43
     new => 1677
      e => 40
       w => 5
        y => 5
        york => 306
         o => 26
         or => 4231
          r => 38
           k => 10
```

SimpleFSTにはleftMostSubstring()以外にも、メモリ上に作成した単語辞書をディスクに保存するsave()や、保存した単語辞書を読み込むためのload()という関数もありますので、いろいろ活用してください。

# Apache Zeppelin から NLP4L を使う{#withZeppelin}

Apache Zeppelin から NLP4L を使う方法について説明します。

## Apache Zeppelin のインストール{#withZeppelin_install}

以下の手順にしたがい、Apache Zeppelin をインストールします。インストール場所はどこでもかまいませんが、ここでは ~/work-zeppelin ディレクトリにインストールすることとします。

```shell
$ mkdir ~/work-zeppelin
$ cd ~/work-zeppelin
$ git clone https://github.com/apache/incubator-zeppelin.git
$ cd incubator-zeppelin
$ mvn install -DskipTests
$ cd conf
$ cp zeppelin-site.xml.template zeppelin-site.xml
```

上記最後のコマンドで用意したファイル zeppelin-site.xml をエディタで開き、プロパティ zeppelin.interpreters の value の最後に次のように org.nlp4l.zeppelin.NLP4LInterpreter を加えます。

```xml
<property>
  <name>zeppelin.interpreters</name>
  <value>org.apache.zeppelin.spark.SparkInterpreter,org.apache.zeppelin.spark.PySparkInterpreter,org.apache.zeppelin.spark.SparkSqlInterpreter,org.apache.zeppelin.spark.DepInterpreter,org.apache.zeppelin.markdown.Markdown,org.apache.zeppelin.angular.AngularInterpreter,org.apache.zeppelin.shell.ShellInterpreter,org.apache.zeppelin.hive.HiveInterpreter,org.apache.zeppelin.tajo.TajoInterpreter,org.apache.zeppelin.flink.FlinkInterpreter,org.apache.zeppelin.lens.LensInterpreter,org.apache.zeppelin.ignite.IgniteInterpreter,org.apache.zeppelin.ignite.IgniteSqlInterpreter,org.nlp4l.zeppelin.NLP4LInterpreter</value>
  <description>Comma separated interpreter configurations. First interpreter become a default</description>
</property>
```

また同じファイルにプロパティ zeppelin.notebook.autoInterpreterBinding を false に設定する次の項目を新規追加します。

```xml
<property>
  <name>zeppelin.notebook.autoInterpreterBinding</name>
  <value>false</value>
  <description></description>
</property>
```

## NLP4L ライブラリの Apache Zeppelin へのデプロイ{#withZeppelin_deploy}

$NLP4L_HOME/target/pack/lib/ 以下の zeppelin-interpreter-XXX.jar ファイルを除く JAR ファイルを、~/work-zeppelin/incubator-zeppelin/interpreter/nlp4l/ 以下にコピーします。

```shell
$ mkdir ~/work-zeppelin/incubator-zeppelin/interpreter/nlp4l
$ cd $NLP4L_HOME
$ cp target/pack/lib/*.jar ~/work-zeppelin/incubator-zeppelin/interpreter/nlp4l
$ rm ~/work-zeppelin/incubator-zeppelin/interpreter/nlp4l/zeppelin-interpreter-*.jar
```

## Apache Zeppelin の起動{#withZeppelin_start}

次のようにして Apache Zeppelin を起動します。

```shell
$ cd ~/work-zeppelin/incubator-zeppelin
$ bin/zeppelin-daemon.sh start
```

なお、停止は次のように stop で行います。

```shell
$ bin/zeppelin-daemon.sh stop
```

ここでは停止せずに次に進みます。

## ノートの作成と NLP4LInterpreter の保存{#withZeppelin_save}

Web ブラウザから [http://localhost:8080/](http://localhost:8080/) にアクセスします。そして、Notebook メニューの Create new note をクリックして新しいノートを作成します。すると次のような画面が現れますので、Save ボタンをクリックして NLP4LInterpreter を保存します。

![Zeppelin Note 初期画面](zeppelin-note-nlp4l-save.png)

## NLP4L のコマンドやプログラムの実行{#withZeppelin_exec}

以降は、Apache Zeppelin のノートのプロンプトから NLP4L のコマンドやプログラムが実行できます。NLP4LInterpreter を呼び出すには、%nlp4l ディレクティブを使います。以下は status コマンドまで入れたところで、Zeppelin 画面のプレイボタン（三角形のボタン）をクリックして実行した様子です。

```shell
%nlp4l
open("/tmp/index-ldcc")
status
Index /tmp/index-ldcc was opened.
res0: org.nlp4l.core.RawReader = IndexReader(path='/tmp/index-ldcc',closed=false)

========================================
Index Path       : /tmp/index-ldcc
Closed           : false
Num of Fields    : 5
Num of Docs      : 7367
Num of Max Docs  : 7367
Has Deletions    : false
========================================
        
Fields Info:
========================================
  # | Name  | Num Terms 
----------------------------------------
  0 | body  |      64543
  1 | url   |       7367
  2 | date  |       6753
  3 | title |      14205
  4 | cat   |          9
========================================
```

## 単語カウントの視覚化{#withZeppelin_visualize}

Zeppelin は数値テーブルをチャート表示してくれる視覚化機能がありますので、[単語の数を数える](#useNLP_wordcounts)で実行した結果をチャート表示してみましょう。方法は簡単で、表示したいところで table() という関数を適用するだけです。

以下はロイターコーパスの記事（Luceneのbodyフィールド）において、"g"で始まる単語から10個を選んで表示するプログラムです。

```scala
%nlp4l

import org.nlp4l.core._
import org.nlp4l.core.analysis._
import org.nlp4l.stats.WordCounts

val index = "/tmp/index-reuters"

val reader = RawReader(index)

val allDS = reader.universalset()
val analyzer = Analyzer(new org.apache.lucene.analysis.standard.StandardAnalyzer(null.asInstanceOf[org.apache.lucene.analysis.util.CharArraySet]))
val allMap = WordCounts.count(reader, "body", Set.empty, allDS, -1, analyzer)
table(allMap.filter(_._1.startsWith("g")).take(10), "word", "count")
```

棒グラフは次のようになります（Zeppelinの棒グラフアイコンをクリックして棒グラフ表示に切り替えます）。

!["g"で始まる単語の出現頻度](zeppelin-wordcounts.png)

また、RawReader の topTermsByDocFreq() や topTermsByTotalTermFreq() を使って出現頻度の大きい単語の出現数を視覚化できます。なお、table() 関数の第1引数はArrayでなければならないので、これらの関数を使うときはうしろに toArray をつける必要があります。

```scala
%nlp4l
table(reader.topTermsByTotalTermFreq("body",5).toArray,"word","docFreq","termFreq")
```

結果は次のようになります（SETTINGS の設定を変更して表示を変えられます）。

![出現頻度の大きい単語](zeppelin-topterms.png)

## ジップの法則を確認する{#withZeppelin_zipfslaw}

topTermsByTotalTermFreq() を使ってジップの法則を確認してみましょう。ジップの法則は、出現頻度が第N位の単語の出現確率は第1位の出現確率のk/Nになるというものです（kは定数）。プログラムは次のようになり、Zeppelinで視覚化すると、おおよそ成り立っていることがうかがえます。

```scala
%nlp4l
val index = "/tmp/index-reuters"
val reader = RawReader(index)

val sum_words = reader.sumTotalTermFreq("body")
val tttf = reader.topTermsByTotalTermFreq("body")
val top = tttf(0)._3.toFloat
table(tttf.map(a => (a._1,top/a._3.toFloat)).toArray,"word","N")
```

![ロイターコレクションにおけるジップの法則](zeppelin_zipfslaw.png)

# NLP4Lプログラムを開発して実行する{#develop}

## REPLから実行する{#develop_repl}

## コンパイルして実行する{#develop_exec}

# 帰属{#tm}
Apache Lucene, Solr and their logos are trademarks of the Apache Software Foundation.

Elasticsearch is a trademark of Elasticsearch BV, registered in the U.S. and in other countries.
