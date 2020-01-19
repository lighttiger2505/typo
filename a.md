# さくらのナレッジ、最終回（Vim script編）
こんにちは。ゴリラです。

ゴリラと学ぶVim講座の連載の最終回となりました。これまでみなさんにVimの基本的な操作から筆者のおすすめプラグインまで、色々と解説してきました。
最終回はみなさんのお待ちかねのプラグインの作成について解説していきます。

前回の記事ではプラグインがVim scriptを使って作られていることを解説しました。
本記事ではまずVim scriptについて最低限の構文を解説してから、実際にテーマとしてセッション管理プラグインを作っていきます。

## Vim scriptの基本
### Vim scriptの実行
Vim scriptはExコマンド(:から始まるコマンド)の集まりです。Exコマンド群をファイルに記述して、それを`:source {file}`で読み込むことで実行できます。
ファイルの拡張子は`.vim`とつけるのが一般的です。

一例ですが、次のコードを`sample.vim`というファイルに保存して、`:source sample.vim`を実行するとコマンドラインに`gorilla`が出力されます。

```vim
let name = 'gorilla'
echo name
```

### コメント
Vim scriptでは次のように`"`から行末までがコメントとして解釈されます。

```vim
" この行はコメント、処理されない
```

### データ型
Vim scriptで主に扱うことができるデータ型は次になります。


### 文字列
Vim scriptで`'`と`"`で囲ったテキストは文字列として解釈されます。
`'`はテキストをそのまま文字列として扱いますが、`"`はタブを表す`\t`といった特殊な文字列を扱うことができます。

例えば`echo "hello\tgorilla"`は`hello`と`gorilla`の間にタブが入りますが、`echo 'hello\tgorilla'`はそのまま出力されます。

`.`で文字列同士を結合できます。例えば`'hello ' . 'gorilla'`は`'hello gorilla'`になります。

### 変数
変数名はアルファベット、数字、アンダースコアを使用できます。ただし数字で開始することができません。
OKとNGの例はそれぞれ次になります。

|OK          |NG      |
|------------|--------|
|name        |2hand   |
|_name       |tow-hand|
|GODZILLA    |gori+lla|

変数や後述する関数にはスコープがあり、接頭子によってスコープが変わります。主なスコープは次になります。

| 接頭子 | スコープ                                             |
|--------|------------------------------------------------------|
| `g:`   | グローバルスコープ、どこからも利用可能               |
| `s:`   | スクリプトスコープ、スクリプトファイル内のみ使用可能 |
| `l:`   | ローカルスコープ、関数内のみ使用可能                 |
| `a:`   | 関数の引数、関数内のみ使用可能                       |
| `v:`   | グローバルスコープ、Vimが予め定義している変数        |

### 辞書
Vim scriptにおける辞書はいわゆる連想配列で、1要素はkeyとvalueからなります。
keyは文字列でなければいけません。次は辞書の例になります。

```vim
let animal = {'name': 'gorilla', 'age': 27}
" 結果 => {'age': '27', 'name': 'gorilla'}
echo animal
```

辞書に新たな要素を追加する方法は次になります。

```vim
let animal = {}
let animal.name = 'gorilla'
let animal['age'] = 27

" 結果 => {'age': 27, 'name': 'gorilla'}
echo animal
```

要素を削除する方法は次になります。

```vim
call remove(animal, 'age')
" 結果 => {'name': 'gorilla'}
echo animal
```

### リスト
Vim scriptでは次にように定義できます。型がないため、異なるデータ型を保持できます。

```vim
let list = ['cat', 10, {'name': 'gorilla'}]
```

要素を取り出すにはインデックスを指定するか、`get({target}, {idx}, {default})`を使用します。
`get({target}, {idx}, {default})`を使用すると存在しないインデックスを指定した場合第3引数で指定した値を返します。

```vim
" 結果 => cat
echo list[0]

" 結果 => NONE
echo get(list, 3, 'NONE')
```

`get()`はリストだけではなく、辞書も使えます。その場合は`{target}`は辞書、`{idx}`はキーを指定します。

```vim
let dict = {'name': 'gorilla'}
" 結果 => gorilla
echo get(dict, 'name', 'cat')
```

また、リストは`join({list}, {sep})`を使用することで{list}の要素を{sep}を使って結合し1つの文字列として返します。

```vim
let list = ['hello', 'my', 'name', 'is', 'gorilla']

" 結果 => hello my name is gorilla
echo join(list, ' ')
```

### if文
if文の基本形は次になります。`{expr}`は式で結果が1の場合はtrue、0の場合はfalseになります。

```vim
if {expr}
  " do something
elseif {expr}
  " do something
else
  " do something
endif
```

### 三項演算子
三項演算子は次のようになります。

```vim
" a の評価結果がtrue(1)ならb、false(0)ならc
a ? b : c
```

### 論理積・和
Vim scriptでの論理積・和は次になります。

```vim
" 結果 => 1(true)
echo 1 && 1
" 結果 => 0(false)
echo 1 && 0
" 結果 => 1(true)
echo 1 || 0
" 結果 => 0(false)
echo 0 || 0
```

### 比較演算子
Vim scriptで主な比較演算子は次になります。ここで注意する必要があるのは`ignorecase`の設定次第で動きが変わる演算子があることです。

| `ignorecase`次第 | 大小文字考慮 | 大小文字無視 | 意味                 |
|------------------|--------------|--------------|----------------------|
| `==`             | `==#`        | `==?`        | 等しい               |
| `!=`             | `!=#`        | `!=?`        | 等しくない           |
| `>`              | `>#`         | `>?`         | より大きい           |
| `>=`             | `>=#`        | `>=?`        | より大きいか等しい   |
| `<`              | `<#`         | `<?`         | より小さい           |
| `<=`             | `<=#`        | `<=?`        | より小さいか等しい   |
| `is`             | `is#`        | `is?`        | 同一のインスタンス   |
| `isnot`          | `isnot#`     | `isnot?`     | 異なるのインスタンス |

### バッファについて
バッファはメモリ上にロードされたファイルのことです。
バッファには名前と番号があり、名前はファイル名で、番号は作成された順で割り当てられます。
バッファは`:bwipeout`で明示的に削除するかVimを終了しなければメモリに残ります。

#### バッファの存在チェック
`bufexists({expr})`で`{expr}`のバッファがあるかを確認できます。ある場合はtrue、なければfalseが返ります。

#### バッファのテキストを取得
カレントバッファからテキストを取得するには`getline({lnum}, {end})`を使用します。
`{end}`を指定しない場合は`{lnum}`で指定した行だけを取得します。

```vim
" 結果 => 1行目のテキストが出力される
echo getline(1)

" 結果 => 1~3行目のテキストがリストで取得できる
echo getline(1, 3)
```

#### バッファにテキストを挿入
カレントバッファにテキストを挿入するには`setline({lnum}, {text})`を使用します。
`{text}`はリストの場合は、`{lnum}`行目とそれ以降の行に要素が挿入されます。

```vim
" 結果 => 1行目に my name is gorilla が挿入される
call setline(1, 'my name is gorilla')

" 結果 => 1行目がmy、2行目がnameが挿入される
call setline(1, ['my', 'name'])
```

### ウィンドウについて
ウィンドウはバッファを表示するための領域です。ウィンドウには一意のIDが割り当てられます。
複数のウィンドウで複数のバッファを表示できます。
`:q`といったコマンドではウィンドウを閉じるだけなのでバッファは残ります。

#### ウィンドウIDを取得
`winnr()`で現在のウィンドウIDを取得できます。引数を受け取ることもできるので、知りたい方は`:h winnr()`を参照してください。

#### ウィンドウに移動
`win_gotid({expr})`で`{expr}`のIDのウィンドウに移動します。

#### バッファが表示されているウィンドウのIDを取得
プラグインを作るときに、すでにあるバッファを使い回すことがよくあります。
そのバッファが表示されているウィンドウのIDを取得には`bufwinid({expr})`を使います。

### 関数
Vim scriptではあ関数を使用して処理をまとめることができます。関数の定義は次のようになります。

```vim
function! Echo(msg) abort
  echo a:msg
endfunction
```

関数は`function`と`endfunction`で囲います。処理はその間に記述します。

#### `!`と`abort`
`!`は同名の関数がある場合は上書きします。
`abort`は関数内でエラーが発生した場合、そこで処理を終了します。Vim scriptはデフォルトエラーがあっても処理が継続されるため基本的に`abort`をつけます。

#### 引数
引数を使用するときは`a:`スコープ接頭子を付ける必要があります。

#### 戻り値
`return {expr}`で`{expr}`の評価結果を返すことができます。

```vim
" 結果 => gorillaが返る
function! MyName() abort
  return 'gorilla'
endfunction
```

### Exコマンド実行
`execute {expr} ..`で`{expr}`の評価結果の文字列をExコマンドとして実行できます。
複数の引数がある場合、それらはスペースで結合されます。

```vim
" 結果 => godzilla
execute 'echo' '"godzilla"'

" 結果 => gorilla godzilla
execute 'echo' '"gorilla"' '"godzilla"'
```

### 外部コマンド実行
Vim scriptで外部コマンドを実行する方法の1つは`system({expr}, {input})`です。
`{epxr}`は実行したいコマンドの文字列、`{input}`はオプションで指定されたい場合は指定した文字列をそのままコマンドの標準入力として渡されます。

```vim
" 結果 => my name is gorilla
echo system('echo "my name is gorilla"')

" 結果 => my name is gorilla
echo system('cat', 'my name is gorilla')
```

### Lambda
{ 引数 -> 式 } という形でLambdaを書くことができます。
次の例は`sort`関数は第2引数にLambdaを受け取ることができて、自前のソート処理を使用できます。

```vim
let F = {a, b -> a - b}
" 結果 => [1, 2, 3, 4, 7]
echo sort([3, 7, 2, 1, 4], F)
```

## プラグイン
Vim scriptについて基本的な構文を解説したので、いよいよプラグインを作っていきましょう。
プラグインを作るときに、まずプラグインの名前を決める必要がありますが、ここで1点注意です。

つけようとしている名前のプラグインがすでにあるかどうかを調べておきましょう。
名前が被ってしまうとプラグインがバッティングしてしまい、動かなくなる可能性があります。

プラグインを検索するときは[VimAwesome](https://vimawesome.com/)をまず使いましょう。そこで見つからなければGoogleで検索しましょう。

今回の作成するプラグインはサンプルなので、とくに名前のバッティングを考慮せず`session.vim`というプラグイン名にします。

### セッション管理プラグイン
今回作成するプラグインはVimのセッション機能を少し便利にするプラグインで、仕様は以下になります。

- `let g:session_path = {path}`でセッション保存先を設定できる（必須オプション）
- `:SessionCreate {name}`で`{name}`の名前でセッションを保存できる
- `:SessionList`でセッション一覧をバッファに表示し、`Enter`を押下するとカーソル上にあるセッションをロードできる

### ディレクトリ構成
プラグインの基本的なディレクトリ構成は次のようになります。
`*.vim`はスクリプトファイルと呼びます。

```
session.vim/
├── autoload
│   └── session.vim
├── doc
│   └── session.txt
└── plugin
    └── session.vim
```

それぞれのディレクトリについて解説していきます。

#### `plugin`ディレクトリについて
`plugin`はExコマンドやプラグインのオプションを記述したスクリプトファイルを置きます。
メインの処理はここではなく後述する`autoload`に記述します。

スクリプトファイル名はプラグイン名と同じにするのが一般的です。

#### `autoload`ディレクトリについて
`autoload`配下はメインの処理を記述したスクリプトファイルを置きます。
配下のスクリプトファイルはVim起動時ではなく、コマンド実行時に一度だけ読み込まれます。

また、スクリプトファイル名はプラグイン名にすることが一般的です。

`plugin`から呼ぶことができる関数を`autoload`配下に定義する時、`ファイル名#関数名()`という命名規則に従う必要があります。
これはコマンドを実行する時に`autoload`配下のどのファイルのどの関数を呼べば良いのかを知る必要があるからです。

そのため、プラグイン名が被ると`autoload`配下のスクリプトファイル名も被り、最悪違うプラグインの関数に上書きされる可能性があります。
これがプラグイン名がかぶらないようにする必要がある理由です。

#### `doc`ディレクトリについて
`doc`配下はヘルプファイルを置きます。`:h SessionList`というようにコマンドのヘルプを引けるようにするためです。
基本的にヘルプにかかれていることは公式、書かれていないのは非公式の機能になります。プラグインを公開する時はREADME.mdだけでなくヘルプを書きましょう。

## 開発の大まかな流れ
少し難しい説明が続きましたが、理解できなくても気をつけるポイントだけ押さえておけば、ひとまずは問題ないです。
ここから先はプラグインを実際作っていきます。大まかに以下の流れになります。

1. ディレクトリ構成通りにファイルとディレクトリを作成
2. `{packpath}/pack/plugins/start`配下にプラグインディレクトリを配置
3. `autoload/session.vim`にセッション保存、一覧取得とロードの処理をを実装
4. `plugin/session.vim`にコマンドとオプションを定義
5. `doc/session.txt`にヘルプを記述

では、作っていきましょう。

### `autoload/session.vim`の実装
#### セッション保存処理
まず`g:session_path`にセッションファイルを作成する関数を実装します。

```vim
" 現在のパスからパスセパレータを取得しています。
" ここはそれほど重要ではないので、おまじないと考えておきましょう。
" 詳しく知りたい方は`:h fnamemodify()`を参照してください。
let s:sep = fnamemodify('.', ':p')[-1:]

function! session#create_session(file) abort
  " SessionCreateの引数がfileで受け取れるようにします。
  " join()でセッション保存先へのフルパスを生成し、mksession!でセッションファイルを作成します。
  execute 'mksession!' join([g:session_path, a:file], s:sep)

  " redrawで画面を再描画してメッセージを出力します。
  redraw
  echo 'session.vim: created'
endfunction
```

#### セッションロード処理
セッション一覧で`Enter`を押下したときにセッションをロードする関数を用意します。

```vim
function! session#load_session(file) abort
  " `:source`で渡されるセッションファイルをロードします。
  execute 'source' join([g:session_path, a:file], s:sep)
endfunction
```

#### セッション一覧取得処理
セッション一覧を取得する処理は大きく分けて2ステップになります。

1. セッションファイルのリストを取得
2. リストを表示するバッファを作成し(すでにあれば表示)、バッファ破棄とセッションをロードするキーマップを設定する

では、それぞれのステップを実装していきましょう。

##### セッションファイルのリストを取得
`g:session_path`からファイルを取得するには`raddir({dir}, {expr})`を使用します。
具体的な説明はコメントを参照してください。

```vim
" エラーメッセージ(赤)を出力する関数
function! s:echo_err(msg) abort
  echohl ErrorMsg
  echomsg 'session.vim:' a:msg
  echohl None
endfunction

" 結果 => ['file1', 'file2', ...]
function! s:files() abort
  " g:session_pathからセッションファイルの保存先を取得します
  let session_path = get(g:, 'session_path', '')

  " g:session_pathが設定されていない場合はエラーメッセージを出し空のリストを返す
  if session_path is# ''
    call s:echo_err('session_path is empty')
    return []
  endif

  " Vim scriptでLambdaを使用できます
  " これは file という引数を受けとり、そのファイルがディレクトリじゃないければ1を返す処理になります
  let Filter = { file -> !isdirectory(session_path . s:sep . file) }

  " reddir の第2引数にLambdaの Filter を使用することで
  " ファイルだけが入ったリストを取得できます
  return readdir(session_path, Filter)
endfunction
```

##### リストを表示するバッファを作成（すでにあれば表示）
`s:files()`で取得できたファイル一覧を一時バッファに書き出し、ユーザが選択できるようにします。

```vim
" セッション一覧を表示するバッファ名
let s:session_list_buffer = 'SESSIONS'

function! session#sessions() abort
  let files = s:files()
  " リストが空の場合は何もしない
  if empty(files)
    return
  endif

  " バッファが存在ている場合
  if bufexists(s:session_list_buffer)
    " バッファがウィンドウに表示されているいる場合は
    " `win_gotoid`でウィンドウに移動します
    let winid = bufwinid(s:session_list_buffer)
    if winid isnot# -1
      call win_gotoid(winid)

    " バッファがウィンドウに表示されていない場合は
    " `sbuffer`で新しいウィンドウを作成してバッファを開きます
    else
      execute 'sbuffer' s:session_list_buffer
    endif
  else
    " バッファが存在していない場合は`new`で新しいバッファを作成します
    execute 'new' s:session_list_buffer

    " バッファの種類を指定します
    " 一時バッファは`nofile`に設定します
    " 詳細を知りたい方は`:h buftype`を参照してください
    set buftype=nofile

    " セッション一覧バッファで`q`を押下するとバッファを破棄
    " `Enter`でセッションをロード
    " の2つのキーマッピングを定義します。
    "
    " <C-u>と<CR>はそれぞれコマンドラインでCTRL-uとEnterを押下した時の動作になります。
    "
    " <Plug>という特殊な文字を使用するとキーを割り当てないマップを用意できます。
    " このマップを使用してユーザが好きなようにキーマッピングできるようにするのが一般的です。
    "
    " \ は改行するときに必要です。
    nnoremap <silent> <buffer>
    \   <Plug>(session-close)
    \   :<C-u>bwipeout!<CR>

    nnoremap <silent> <buffer>
    \   <Plug>(session-open)
    \   :<C-u>call session#load_session(trim(getline('.')))<CR>

    " <Plug>マップをキーにマッピングします。
    " `q` で最終的に :<C-u>bwipeout!<CR>
    " `Enter`で最終的に :<C-u>call session#load_session()<CR>
    " が呼ばれます
    nmap <buffer> q <Plug>(session-close)
    nmap <buffer> <CR> <Plug>(session-open)
  endif

  " セッションファイルを表示する一時バッファのテキストをすべて削除して、
  " 取得したファイル一覧をバッファに挿入
  %delete _
  call setline(1, files)
endfunction
```

### `plugin/session.vim`の実装

