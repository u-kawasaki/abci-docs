# FAQ

## Q. インタラクティブ利用時にCtrl+Sを入力するとそれ以降のキー入力ができない

macOS、Windows、Linuxなどの標準のターミナルエミュレータでは、デフォルトでCtrl+S/Ctrl+Qによる出力制御が有効になっているためです。無効にするためには、ローカルPCのターミナルエミュレータで以下を実行してください。

```shell
$ stty -ixon
```

インタラクティブノードにログインした状態で実行しても同等の効果があります。

## Q. グループ領域が実サイズ以上に消費されてしまう

一般にファイルシステムにはブロックサイズがあり、どんなに小さいファイルであってもブロックサイズ分の容量を消費します。

ABCIでは、グループ領域のブロックサイズは128KB、ホーム領域のブロックサイズは4KBとしています。このため、グループ領域に小さいファイルを大量に作ると利用効率が下がります。例えば、4KB未満のファイルを作る場合には、ホーム領域の約32倍の容量が必要になります。

## Q. 認証が必要なコンテナレジストリをSingularityで利用できない

SingularityPROには``docker login``相当の機能として、環境変数で認証情報を与える機能があります。

```shell
[username@es1 ~]$ export SINGULARITY_DOCKER_USERNAME='username'
[username@es1 ~]$ export SINGULARITY_DOCKER_PASSWORD='password'
[username@es1 ~]$ singularity pull docker://myregistry.azurecr.io/namespace/repo_name:repo_tag
```

SingularityPRO の認証に関する詳細は、以下をご参照ください。

* [SingularityPRO 3.7 User Guide](https://repo.sylabs.io/c/0f6898986ad0b646b5ce6deba21781ac62cb7e0a86a5153bbb31732ee6593f43/guides/singularitypro37-user-guide/)
    * [Making use of private images from Private Registries](https://repo.sylabs.io/c/0f6898986ad0b646b5ce6deba21781ac62cb7e0a86a5153bbb31732ee6593f43/guides/singularitypro37-user-guide/singularity_and_docker.html?highlight=support%20docker%20oci#making-use-of-private-images-from-private-registries)

## Q. NGC CLIが実行できない

ABCI上で、[NGC Catalog CLI](https://docs.nvidia.com/ngc/ngc-catalog-cli-user-guide/index.html)を実行すると、以下のエラーメッセージが出て実行できません。これはNGC CLIがUbuntu 14.04以降用にビルドされているためです。

```
ImportError: /lib64/libc.so.6: version `GLIBC_2.18' not found (required by /tmp/_MEIxvHq8h/libstdc++.so.6)
[89261] Failed to execute script ngc
```

以下のようなシェルスクリプトを用意することで、Singularityを使って実行させることができます。NGC CLIに限らず、一般的に使えるテクニックです。

```
#!/bin/sh
source /etc/profile.d/modules.sh
module load singularitypro

NGC_HOME=$HOME/ngc
singularity exec $NGC_HOME/ubuntu-18.04.simg $NGC_HOME/ngc $@
```

## Q. 複数の計算ノードを割り当て、それぞれの計算ノードで異なる処理をさせたい

`qrsh`や`qsub`で`-l rt_F=N`オプションを与えると、N個の計算ノードを割り当てることができます。割り当てられた計算ノードでそれぞれ異なる処理をさせたい場合にもMPIが使えます。

```
$ module load openmpi/2.1.6
$ mpirun -hostfile $SGE_JOB_HOSTLIST -np 1 command1 : -np 1 command2 : ... : -np1 commandN
```

## Q. SSHのセッションが閉じられてしまうのを回避したい

SSHでABCIに無事接続したしばらく後に、SSHのセッションが閉じられてしまうことがあります。このような場合は、SSHクライアントとサーバ間でKeepAliveの通信をすることで回避できる場合があります。

KeepAliveを適用するには、利用者の端末でシステムのssh設定ファイル(/etc/ssh/ssh_config)、またはユーザ毎の設定ファイル(~/.ssh/config)に、オプション ServerAliveInterval を60秒程度で設定してください。

```
[username@userpc ~]$ vi ~/.ssh/config
[username@userpc ~]$ cat ~/.ssh/config
(snip)
Host as.abci.ai
   ServerAliveInterval 60
(snip)
[username@userpc ~]$
```

!!! note
    ServerAliveInterval の初期値は 0 (KeepAliveなし)です。

## Q. Open MPIの新しいバージョンが使いたい

ABCIではCUDA対応版Open MPIとCUDA非対応版Open MPIを提供しており、提供の状況は、[MPIの利用](08.md#open-mpi)で確認できます。

ABCIが提供するEnvironment Modulesでは、事前に`cuda`モジュールがロードされている場合に限り、`openmpi`モジュールのロード時にCUDA対応版Open MPIの環境設定を試みます。

したがって、CUDA対応版MPIが提供されている組み合わせ(`cuda/10.0/10.0.130.1`, `openmpi/2.1.6`)では環境設定に成功します。

```
$ module load cuda/10.0/10.0.130.1
$ module load openmpi/2.1.6
$ module list
Currently Loaded Modulefiles:
  1) cuda/10.0/10.0.130.1   2) openmpi/2.1.6
```

CUDA対応版MPIが提供されていない組み合わせ(`cuda/9.1/9.1.85.3`, `openmpi/3.1.6`)では環境設定に失敗し、`openmpi`モジュールはロードされません。

```
$ module load cuda/9.1/9.1.85.3
$ module load openmpi/3.1.6
ERROR: loaded cuda module is not supported.
WARNING: openmpi/3.1.6 cannot be loaded due to missing prereq.
HINT: at least one of the following modules must be loaded first: cuda/9.2/9.2.88.1 cuda/9.2/9.2.148.1 cuda/10.0/10.0.130.1 cuda/10.1/10.1.243 cuda/10.2/10.2.89 cuda/11.0/11.0.3 cuda/11.1/11.1.1 cuda/11.2/11.2.2
$ module list
Currently Loaded Modulefiles:
  1) cuda/9.1/9.1.85.3
```

一方、Horovodによる並列化のためにOpen MPIが使いたいなど、Open MPIのCUDA版機能が不要な場合もあります。この場合は、先に`openmpi`モジュールをロードすることで、より新しいバージョンのCUDA非対応版Open MPIを利用できます。

```
$ module load openmpi/3.1.6
$ module load cuda/9.1/9.1.85.3
module list
Currently Loaded Modulefiles:
  1) openmpi/3.1.6       2) cuda/9.1/9.1.85.3
```

!!! note
    CUDA対応版の機能はOpen MPIのサイトで確認できます: [FAQ: Running CUDA-aware Open MPI](https://www.open-mpi.org/faq/?category=runcuda)

## Q. ジョブの混雑状況を知りたい

ジョブの混雑状況に加え、計算ノードの利用状況、データセンター全体の消費電力やPUE、冷却設備の稼働状況等を可視化するWebサービスを動作させています。
ABCI内部サーバ`vws1`の3000/tcpポートで動作していますので、以下の通りにアクセスできます。

SSHトンネルの設定をしてください。
以下の例では、ローカルPCの`$HOME/.ssh/config`に、ProxyCommandを用いてas.abci.ai経由でABCI内部サーバにSSHトンネル接続する設定をしています。
ABCIシステム利用環境の[SSHクライアントによるログイン::一般的なログイン方法](./02.md#general-method)も参考にしてください。

```shell
Host *.abci.local
    User         username
    IdentityFile /path/identity_file
    ProxyCommand ssh -W %h:%p -l username -i /path/identity_file as.abci.ai
```

ローカルPCの3000番ポートをvws1サーバの3000/tcpポートに転送するSSHトンネルを作成します。

```shell
[username@userpc ~]$ ssh -L 3000:vws1:3000 es.abci.local
```

ブラウザで`http://localhost:3000/`にアクセスします。

!!! note
    ABCI User Groupでは、インタラクティブノード上で混雑状況を確認する方法が紹介されていますので、こちらもご参照ください。

    - [ABCIの混雑具合を確認する](https://abciug.abci.ai/abci%e5%88%a9%e7%94%a8%e3%81%ae%e8%b1%86%e7%9f%a5%e8%ad%98/abci%e3%81%ae%e6%b7%b7%e9%9b%91%e5%85%b7%e5%90%88%e3%82%92%e7%a2%ba%e8%aa%8d%e3%81%99%e3%82%8b_i6)
    - [ABCIの混雑具合を確認する（その２）](https://abciug.abci.ai/abci%e5%88%a9%e7%94%a8%e3%81%ae%e8%b1%86%e7%9f%a5%e8%ad%98/abci%e3%81%ae%e6%b7%b7%e9%9b%91%e5%85%b7%e5%90%88%e3%82%92%e7%a2%ba%e8%aa%8d%e3%81%99%e3%82%8b%e3%81%9d%e3%81%ae%ef%bc%92_i10)
    - [ABCIの空ノード数を調べる](https://abciug.abci.ai/abci%e5%88%a9%e7%94%a8%e3%81%ae%e8%b1%86%e7%9f%a5%e8%ad%98/abci%e3%81%ae%e7%a9%ba%e3%83%8e%e3%83%bc%e3%83%89%e6%95%b0%e3%82%92%e8%aa%bf%e3%81%b9%e3%82%8b_i16)

## Q. ダウンロード済みのデータセットはありませんか?

[こちら](tips/datasets.md)のページをご参照ください。

## Q. バッチジョブでSingularity pullでのイメージファイル作成に失敗する

バッチジョブでSingularity pullでのイメージファイルを作成しようとした際に、mksquashfs の実行ファイルが見つからず作成に失敗することがあります。

```
INFO:    Converting OCI blobs to SIF format
FATAL:   While making image from oci registry: while building SIF from layers: unable to create new build: while searching for mksquashfs: exec: "mksquashfs": executable file not found in $PATH
```

これは、以下のように、`/usr/sbin` にパスを通すことで回避できます。 

実行例）
```
[username@g0001 ~]$ export PATH="$PATH:/usr/sbin"
[username@g0001 ~]$ module load singularitypro
[username@g0001 ~]$ singularity run --nv docker://caffe2ai/caffe2:latest
```

## Q. ジョブ ID を調べるには？ {#q-how-can-i-find-the-job-id}

`qsub` コマンドを使ってバッチジョブを投入した場合は、コマンドがジョブ ID を出力しています。

```
[username@es1 ~]$ qsub -g grpname test.sh
Your job 1000001 ("test.sh") has been submitted
```

`qrsh` を使っている場合は、環境変数 JOB_ID の値を見ることで確認できます。この変数は、qsub (バッチジョブ) の場合でも利用可能です。

```
[username@es1 ~]$ qrsh -g grpname -l rt_C.small=1 -l h_rt=1:00:00
[username@g0001 ~]$ echo $JOB_ID
1000002
[username@g0001 ~]$
```

すでに投入済みのジョブに付与されているジョブ ID を確認するには、`qstat` コマンドを使ってください。

```
[username@es1 ~]$ qstat
job-ID     prior   name       user         state submit/start at     queue                          jclass                         slots ja-task-ID
------------------------------------------------------------------------------------------------------------------------------------------------
   1000003 0.00000 test.sh username   qw    08/01/2020 13:05:30
```

対象のジョブがすでに完了している場合は、`qacct -j` を使ってジョブ ID を調べます。`-b` や `-e` オプションが、対象範囲を限定するために役に立ちます。qacct(1) man ページ (インタラクティブノードで `man qacct`) で使い方を確認できます。下記の例は、完了したジョブのうち、2020年 9月 1日 以降に開始されていたものを出力しています。`jobnumber` がジョブ ID です。

```
[username@es1 ~]$ qacct -j -b 202009010000
==============================================================
qname        gpu
hostname     g0001
group        grpname
owner        username

:

jobname      QRLOGIN
jobnumber    1000010

:

qsub_time    09/01/2020 16:41:37.736
start_time   09/01/2020 16:41:47.094
end_time     09/01/2020 16:45:46.296

:

==============================================================
qname        gpu
hostname     g0001
group        grpname
owner        username

:

jobname      testjob
jobnumber    1000120

:

qsub_time    09/07/2020 15:35:04.088
start_time   09/07/2020 15:43:11.513
end_time     09/07/2020 15:50:11.534

:
```

## Q. 割り当てられた全計算ノードでコマンドを並列に実行したい

ABCIでは、割り当てられた全計算ノードで並列にLinuxコマンドを実行する`ugedsh`コマンドを用意しています。
`ugedsh`コマンドの引数で指定したコマンドは、各ノードで1回ずつ実行されます。

実行例) 

```
[username@es1 ~]$ qrsh -g grpname -l rt_F=2
[username@g0001 ~]$ ugedsh hostname
g0001: g0001.abci.local
g0002: g0002.abci.local
```

## Q. ABCIグループ領域には新領域と旧領域があるのですか？その違いは？

ABCIは機能増強のため、グループ領域として新しいストレージを増設いたしました。新しいストレージは2021年度にご利用申請いただいたグループ領域に順次割り当てられます。その場所は "/groups/【グループ名】" となります。<br/>
一方、2020年度以前からご利用いただいていた利用者の皆様は旧グループ領域として "/groups1/【グループ名】" または "/groups2/【グループ名】" から引き続きその内容を参照することができます。2021年度以降にご利用申請いただいたグループには旧グループ領域は割り当てられません。<br/>
但し、旧グループ領域に割り当てられているストレージはその内容を新ストレージ内にコピーした後に順次廃止していく予定です。旧ストレージ廃止後も旧グループ領域に収められていた内容は引き続き "/groups1/【グループ名】" または "/groups2/【グループ名】" から参照することができますので、利用者様はあまり意識していただく必要はありません。しかしながら、ストレージ移行作業をスムーズに実施するため、なるべく旧領域のデータは新領域にコピーまたは移動してからご利用ください。ご不要になった旧グループ領域内のデータは削除していただきますと助かります。

## Q. グループ領域の移行にあたって注意することはありますか？

旧グループ領域は、2021年5月から運用開始(予定)の新しい計算ノード(A)からは参照することができません。計算ノード(A)でご利用になりたいデータは、新グループ領域にコピーするか、旧領域のデータが新ストレージに移行を完了するまでお待ちいただいてからご利用いただく必要があります。<br/>
データの移行作業は運用側で行いますが、移行作業中は一時的に新領域のディスククォータが旧領域の2倍に増えます。作業完了後は猶予期間を経て元のクォータ値に戻させていただきます。
移行完了の確認のため、旧領域を参照できない期間が数日発生する予定ですのでご了承ください。その間でも新領域への R/W および 削除は可能です。

## Q. 新グループ領域と旧グループ領域を使い分けるポイントは？

旧グループ領域は、2021年5月から運用開始(予定)の新しい計算ノード(A)からは参照することができません。計算ノード(A)でご利用になりたいデータは、新グループ領域にコピーするか、旧領域のデータが新ストレージに移行を完了するまでお待ちいただいてからご利用いただく必要があります。<br/>
尚、従来からサービスを提供しております計算ノード(V)からは、新領域・旧領域のいずれのデータもご利用いただくことができますが、なるべく旧領域のデータを新領域にコピーまたは移動してからご利用ください。また、移行作業がスムーズに行われるように、ご不要になった旧グループ領域内のデータは削除していただきますと助かります。<br/>
旧ストレージ内の全データを新ストレージに移行するための期間は概ね1年と考えております。

## Q. 旧グループ領域内のデータが新ストレージに移行される時期は？

旧グループ領域 (/groups1/【グループ名】 または /groups2/【グループ名】) にあるデータは運用側で順次新ストレージ内に移行させていただきます。移行後のデータの参照先も "/groups1/【グループ名】" または "/groups2/【グループ名】" で変更ありません。移行完了時にはご利用グループの責任者宛てにメールでお知らせいたします。<br/>
移行作業は 2021年4月から概ね1年かけて運用側で実施させていただきます。作業の順番は /groups2 → /groups1 → /fs3 の予定です。

