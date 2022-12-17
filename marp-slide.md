---
marp: true
header: "Marpを使ってスライドを作る方法"
footer: "by @nnydtmg © 2022"
paginate: true
---
<!--
headingDivider: 1
-->
<style>
    h1{
        position: absolute;
        left: 50px; top: 100px;
    }
</style>


# Marpを使ってスライドを作る方法

## 目次

* Marpについて
* Marpの基本設定
* Marpの基本書式
* Markdownの書式


# Marpってなに？

Markdownからスライドを作成してくれるツール
原本はMarkdownで記載できるため、Git等のコード管理が可能
全スライドに適用する書式設定(futterやheader等)や個別スライドに書式設定が可能
独自のCSSをあてることも可能(参照3)

参照1：https://qiita.com/tomo_makes/items/aafae4021986553ae1d8
参照2：https://tracpath.com/works/development/marp/
参照3：https://techblog.istyle.co.jp/archives/6356

# 基本設定1

Markdownの最初に以下を記載するとMarpが有効化される
```
---
marp: true
---
```
この中にheaderやfooterが設定可能
```
header: "Marpを使ってスライドを作る方法"  # header設定
footer: "by @nnydtmg © 2022"            # footer設定
paginate: true                          # ページ番号表示設定
```

# 基本設定2

全スライドへの設定方法として以下もOK
```
<!-- headingDivider: 1 -->
```
※これは#(h1タグ)があると新スライドと認識する設定(詳しくは後ほど）

# 基本設定3

他にもCSSの設定も可能
```
<style>
    h1{
        position: absolute;
        left: 50px; top: 100px;
    }
</style>
```
※h1タグの位置を決めている

# 基本書式1

ページの構成は以下の通り
```
# タイトル1
1ページ目文章1
---
# タイトル2
2ページ目文章
---
```
※ここの---については、基本設定2の設定をしていると不要になる




