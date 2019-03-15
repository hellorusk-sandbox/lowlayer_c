## 基本的なプロセスAPI

- fork(2)
- exec(2)
- wait(2)

#### fork(2)

自プロセスを複製して新しいプロセスを作る

fork() を呼び出すと、2つのプロセスに分裂し、その時点でその2つのプロセスはどちらも fork() を呼び出した状態になる。呼び出しが戻るとき、親プロセスではプロセスの PID が戻り値に、子プロセスでは0が戻り値になる。失敗した場合は子プロセスが作成されず、親プロセスでのみ -1 が戻る。

#### exec（exec 族）

自プロセスを新しいプログラムで上書きする  
exec を実行すると、その時点で現在実行しているプログラムが消滅し、自プロセス上に新しいプログラムをロードして実行する

`execl`, `execv` など

ex) 

```
execl("/bin/cat", "cat", "hello.c", NULL);
```

```
char *argv[3] = { "cat", "hello.c", NULL };
execv("/bin/cat", argv);
```

#### wait(2)

fork() したプロセスの終了を待つ

## プログラムを実行して結果を待つ

1. fork() する
2. 子プロセスで新しいプログラムを exec する
3. 親プロセスは子プロセスを wait する

という流れ

## プロセスの一生

#### _exit(2)

プロセスを自発的に終了するシステムコール

#### exit(3)

プロセスを自発的に終了するライブラリ

_exit(2)との違い

- stdio のバッファを全てフラッシュする
- atexit() で登録した処理を実行する

#### ゾンビプロセス

親プロセスがいつまでたっても wait() を呼ばず、動作し続ける子プロセス

## パイプ

ファイル同士がストリームでつながったように、プロセス同士はパイプでつながる  
パイプもファイルディスクリプタ（ファイル記述子）で表現される

#### pipe(2)

両端とも自プロセスにつながったストリームを作成し、その両端のファイルディスクリプタを返す

#### dup(2), dup2(2)

ファイルディスクリプタを複製する dup() を活用することで、プロセス間でパイプをつなぐことができる

例えば3番につながったパイプを0番にうつすには

1. close(0);
2. dup2(3, 0);
3. close(3);

#### popen(3)

プログラム command を起動してそれにパイプをつなぎ、そのパイプを表す stdio ストリームを返す

#### pclose(3)

popen() で開いた FILE* を閉じる

## プロセスの関係

プロセスは fork() やそれに類するAPIで生成されていくので、全体として一つのツリー構造になっている。`pstree` で確認できる

#### getpid(2), getppid(2)

プログラムから自分のプロセスIDや親プロセスのプロセスIDを確認できる

#### プロセスグループ、セッション

プロセスグループは大まかにいってシェルのためにある。例えばプロセスがパイプで繋がっている時、パイプを構成する全プロセスをまとめて扱えば、Ctrl + C による全ての停止などができるようになる。

セッションはユーザのログインからログアウトまでの流れを管理する。セッションと関連づけられた端末のことを**制御端末**と呼ぶ。

プロセスグループやセッションは`ps j`で確認できる。  
PID（プロセスID）とPGID（プロセスグループID）が等しいプロセスは**プロセスグループリーダー** （最初にそのプロセスグループを作ったプロセス）

PID（プロセスID）とSID（セッションID）が等しいプロセスは**セッションリーダー** （最初にそのセッションを作ったプロセス）

#### デーモンプロセス（デーモン）

制御端末を持たないプロセス。サーバーのためにある

#### setpgid(2)

新しいプロセスグループを作るシステムコール

#### setsid(2)

新しいセッションを作るシステムコール