# mgl77

77回文化祭で使用した（78文化祭でも一部を改修して利用）ミニゲームランチャーです。

言語はPythonで、パッケージマネージャとして [rye](https://rye-up.com/) を使って開発されているので開発やビルドを行う場合はインストールしてください

GUIなどはFletというフレームワークで制作されています。Flet自体の機能などについては調べて欲しいんですが、ベースの思想として命令的にUIを記述するというのがあるので、初心者でも仕組みが分かりやすいとは思います

## 開発時

まずryeをインストールしてください

- `rye sync` で依存関係やPythonインタプリタ本体をインストールする(最初に実行)
- `rye add <パッケージ名>` で依存関係を追加。`pip install` は使わないで❗️
- `rye run <コマンド>` で依存パッケージが提供するcliコマンドを実行
- `rye run flet run ./src/mgl77/main.py` で実行する(ウィンドウを出す)。ホットリロードに対応していて、ファイルを変更して保存すると(少し時間かかるけど)再起動しなくとも反映される
- `rye fmt` でコードを整形

## ビルド(exe生成)

```commandline
rye run pyinstaller ./src/mgl77/main.py --name mgl77.exe
rye run pyinstaller ./src/mgl77/back_loop.py --name mgl77-loop.exe
```

- すでに存在するけど上書きする？みたいな警告が出た場合はyを押す
- かなり時間かかる
- 終わったらdist/かbuild/とかに2つそれぞれのexeとinternalディレクトリが生成される　mgl77.exeがランチャー本体で、mgl77-loopはmgl77.exeを(落とされても)実行し続けるプロセスになっていて、展示ではloopのほうを実行する
- internalの中身はexeが依存するPythonパッケージで、exeと同じディレクトリに必ず置く
- ゲームデータはexeと同じディレクトリのassets/games下に後述するフォーマットで置く。ゲームデータはgitで自動的にfetchされる仕様になっているので手動で置いたり更新する必要はない
  - fetchしてくるgitリポジトリは今は `https://github.com/nahco314/agc77-minigames` になっている(コードの中にハードコートされてます)

## 仕様など
- タイトルなどが表示されるタイトル画面と、ゲームを選んで起動するメイン画面がある。メイン画面を開いて5分間立つと「そろそろ交代してね」という警告が出て、~~10分~~7分（78文化祭で変更）経つと「交代してね」の警告と共にランチャーとゲームが落ちる
- ランチャーを落とすか左上の戻るを押すとタイマーがリセットされる
- ランチャーが落ちると(mgl77-loop.exeの方を実行している場合)自動で再起動される
- 起動したゲームのプロセスが生きてる状態で別のを開くとエラーのダイアログが出る
- ランチャーを閉じるとゲームのプロセスも落ちる
- タイトル画面が開くたびにgitでゲームデータをfetchする。時間がかかる場合もある
  - ネットに繋がってなかったりでfetchに失敗したらログは出るけど普通に無視して続行されるので、ネットが落ちても大丈夫
- ゲームデータはassets/games/下に各ゲーム用のフォルダを用意して設定する
  - 各フォルダには、exeとexeが依存する諸々と、ss.pngとgame.tomlを置く
  - ss.pngはゲームのスクリーンショット、ランチャーで表示されるやつ。必須ではなくて、存在しない場合適当な灰色の画像が表示される
  - game.tomlはゲームの設定を記述するファイル。このリポジトリのassets/gameやexample_game.tomlに例があるので参考に。各項目は以下の通り:
    - title: ゲームのタイトル
    - description: 1〜2行程度のゲームの軽い説明
    - author: 作った人の名前。表示されるので、学年+本名でもペンネームでも
    - game_exe_name: ゲームのexeの名前(.exeは付けない)
  - game.tomlの内容とかが間違えてるとランチャー本体の起動に失敗する。ログは出る
- ランチャー自体の起動に失敗したらログを確認してね
- ランチャーを再起動したいときはloop.exeのウィンドウをしっかり落とさないといけない
 
## あったら便利だけど実現できなかった機能
- PC本体が10秒程度のあいだ無操作状態だったら自動でタイトル画面に戻る(タイマーがリセットされる)
- すでに存在しているけどgit上で消えたゲームのデータをfetchの時に消す
- ~~起動したゲームのウィンドウを最前面に持ってくる。起動しても後ろに隠れちゃって、起動したことに気づかないお客さんが「もう起動してるってエラーが出るよ〜」って言ってくることが結構多い~~（78文化祭で実装した）

## あったら便利だけど実現できなかった機能（補遺・78文化祭時に考えたもの）
 - やめるボタンを押したときにすべてのゲームが閉じられるようにする（おそらくそのためにはゲームのプロセス管理を確認する必要がある）
 - 特定のキーの組み合わせでやめるボタンの代わりにできるようにする
 - なんかダイアログがうまく出てくれない例がある（これもたぶんゲームのプロセス管理の仕方を見直す必要がある）