## プライベート・ネットワークに接続する

Gethのインストールが完了したら早速Gethを起動します。

Ethereumでは以下の３つの形態のP2Pネットワークを構築しブロックチェーンを運用していくことが可能です。

* **パブリック・ネットワーク**：不特定多数のノードのノードが参加し、かつその参加に制限が全くないネットワークです。参加ノードはそのネットワーク上で共有管理されたブロックチェーンに対して自由に、読み取り、トランザクションの発行、マイニングが可能です。仮想通貨としてのEthereumや、多くのパブリックなdAppはこのパブリックネットワーク上で運用されています。
* **コンソーシアム・ネットワーク**：あらかじめ参加を許されたノードのみが参加することが可能なネットワークです。参加を許されるノードは一つの組織のみに管理されたものとは限らず、複数の利害関係が一致しない組織がそれぞれのノードを管理することが通常です。例えば国際送金の管理を行うブロックチェーンを構築したい場合、予め参加を許された複数の金融企業がそれぞれ管理するノードをこのP２Pのネットワークに参加することで、一つの企業にのみ管理されたシステムではない半非中央集権なシステムが構築可能になります。
* **プライベート・ネットワーク**：一つの組織のみに管理されたノードのみが参加することが可能なネットワークです。ネットワークは自身の管理下に置くことが可能になり、中央集権的なP２Pシステムが可能になります。

ここでプライベート・ネットワークは、自分自身のみのネットワークなので容易にEtherの採掘[^1]が可能ですし、安全性も高いネットワークです。そのため、Ethereumの動作を調べたり、分散型アプリケーション（Dapp）の開発作業など個人的な作業を行うには、プライベート・ネットワークを立ち上げてそこでいろいろ弄ってみると便利です。

そこで本節では、インストールしたGethを起動し、プライベート・ネットに接続するところまでを解説していきます。

### Genesisファイルを作成する

Genesisファイルとは、ネットワークでやり取りされるブロックチェーンの最初（Block番号 "0"）のブロックであるGenesisブロックの情報を記述したファイルです。プライベート・ネットでは独自のブロックチェーンをやり取りしていくため、独自のGenesisブロックを定義したGenesisファイルを用意して利用します[^2]。

まず任意の場所にプライベート・ネットのブロック情報やノード情報など各種データを格納するディレクトリ（データ・ディレクトリ）を作成します。ここでは、ログイン・ユーザー（今回の例ではtest\_u）のhomeディレクトリ直下に作成します[^3]。

```plain
$ mkdir /home/test_u/eth_private_net
```

次に上記ディレクトリ内にjson形式の下記の内容[^4] を記述した`myGenesis.json`ファイルを配置します。

```javascript
{
  "config": {
    "chainId": 15
  },
  "nonce": "0x0000000000000042",
  "timestamp": "0x0",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "extraData": "",
  "gasLimit": "0x8000000",
  "difficulty": "0x4000",
  "mixhash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "coinbase": "0x3333333333333333333333333333333333333333",
  "alloc": {}
}
```

### Gethをプライベート・ネットで起動する

#### genesisブロックの初期化

データ・ディレクトリとgenesisファイルを作成したら、以下のコマンドを実行しブロックチェーン情報をgenesisファイルの内容で初期化します。

```plain
$ geth --datadir /home/test_u/eth_private_net init /home/test_u/eth_private_net/myGenesis.json
```

本コマンドを実行すると、`--datadir`で指定したディレクトリ以下にディレクトリが新しく作成されて、その中にgenesisブロックのブロックチェーン情報が保存されます。ここで実行時に

```
WARN [02-04|09:03:55] No etherbase set and no accounts found as default
```

という警告が表示されますがこのノード（geth）でのデフォルトのウォレットのアドレスを作成していないために表示されるものです。これは次節で作成していくので、現時点では無視して構いません。

#### gethの起動

次に以下のコマンドを実行することでGethを起動します。

```plain
$ geth --networkid "15" --nodiscover --datadir "/home/test_u/eth_private_net" console 2>> /home/test_u/eth_private_net/geth_err.log
```

ここで各オプションの意味は以下の通りです。

* `--datadir` :本オプションはgethの動作時のブロックチェーンデータや各種ログの出力先を指定します。genesisブロックの初期化で指定したディレクトリと同一のものを指定してください。
* `--networkid "15"` ：本オプションで任意の正の整数のIDを指定することで、ライブ・ネットとは異なるネットワークを立ち上げることが可能です（ここでは15を指定）。genesisブロックの初期化で指定した`chainid`と同一の値を指定する必要があります。
* `--nodiscover` ：Gethはデフォルトで自動的に（同じネットワークID）のEthereumネットワークのノード（Peer）を探し接続を試みます。プライベート・ネットでは未知のノードとの接続を避けるため、このオプションで自動Peer探索機能を無効にします。
* `console`：Gethには採掘やトランザクションの生成などを対話的に進めることができるコンソールが用意されています。`console`サブ・コマンドを指定することで、Gethの起動時に同時にコンソール立ち上げることが可能です。なお、`console`サブ・コマンドを付加せずに、Gethのプロセスをバックグラウンドで起動させておき、後からそのプロセスのコンソールを起動する事も可能です（下記TIP参照）。

上記コマンドを実行すると、下記の実行結果のように、幾つかの情報の表示の後に「&gt;」のプロンプトが表示され、コンソールが起動されます。今後、特にことわりのない限りこのコマンドで起動したGethプロンプト上で作業していく前提で進めていきます。

```
Welcome to the Geth JavaScript console!

instance: Geth/v1.4.10-stable/linux/go1.5.1
 modules: admin:1.0 debug:1.0 eth:1.0 miner:1.0 net:1.0 personal:1.0 rpc:1.0 txpool:1.0 web3:1.0
```

実際に今回立ち上げたプライベート・ネットのGenesisブロックが`myGenesis.json`に記載されたものになっているのかを確認してみます。Gethプロンプト上で

```
> eth.getBlock(0)
```

のコマンドを実行してみます。このコマンドは指定したブロック番号のブロック情報を表示するもので、今回はブロック番号"0"を指定してGenesisブロックの情報を表示します。下の結果のように例えば`difficulty`が`myGenesis.json`に指定したものになっているはずです。（ただし16進表記から10進表記に変換されています。）

```
> eth.getBlock(0)
{
  difficulty: 16384,
  extraData: "0x00",
  gasLimit: 134217728,
（中略）
  miner: "0x3333333333333333333333333333333333333333",
  nonce: "0x0000000000000042",
  number: 0,
  parentHash: "0x0000000000000000000000000000000000000000000000000000000000000000",
（後略）
}
```

###### ■■ TIP ■■

今後Gethを使用していくなかで、採掘等のためにGethをバックグラウンドで常時起動しておき、必要に応じてそのGethプロセスに対してコンソールを用いて対話的に操作をしたいといった場合が発生します。その際は、下記のように`attach`サブ・コマンドを利用することで、既に起動されたGethプロセスのコンソールを起動することが可能です。

```
$ # gethプロセスをconsoleサブ・コマンドを付加せず、かつ最後に"&"を付加することで、バックグラウンドで起動します。
$ # この場合、起動時にはコンソールは立ち上がりません。
$ geth --networkid "10" --nodiscover --datadir "/home/test_u/eth_private_net" --genesis "/home/test_u/eth_private_net/myGenesis.json" 2>> /home/test_u/eth_private_net/geth_err.log &
$
$ # attachサブ・コマンドを用いて先に立ち上げたプロセスのコンソールを立ち上げます。
$ # ここで、ipc:以降に先に立ち上げたgethプロセスのデータ用ディレクトリ以下のgeth.ipcファイル（実際はソケット）のパスを指定します。
$ geth --datadir "/home/test_u/eth_private_net" attach ipc:/home/test_u/eth_data/geth.ipc

instance: Geth/v1.3.5/linux/go1.5.1
（実行結果 中略）
modules: admin:1.0 db:1.0 debug:1.0 eth:1.0 miner:1.0 net:1.0 personal:1.0 shh:1.0 txpool:1.0 web3:1.0
>
```

### 脚注

[^1]: ただし、テスト・ネットで採掘したEtherはテスト・ネット内のみで有効で、ライブ・ネットでは使用できないことに注意してください。

[^2]: 独自のgenesisファイルを使用することの最大の目的は「採掘を容易にする」ことです。プライベート・ネットでもデフォルトのgenesisブロックを使うことも可能ですが、デフォルトのgenesisブロック（＝Frontierのgenesisブロック）の採掘難易度（Difficulty）が非常に高く（17,179,869,184）設定されたため、プライベート・ネットでの採掘が非常に困難になりました。そのためDifficultyの小さい独自のGenesisファイルを用意してプライベート・ネット内での採掘を容易にすることが慣習になっています。 

[^3]: Gethのデフォルトではホーム・ディレクトリ以下に「.ethereum」という名前の隠しディレクトリが作成され、それがデータ・ディレクトリになります。ただし、今後プライベート・ネットとライブ・ネットとでデータディレクトリを分ける方が管理がしやすいことを考えると、データディレクトリをプライベート・ネット用に明示的に指定したほうがよいでしょう。 ↩

[^4]: 特にgenesisファイルの内容については詳しくはここでは述べませんが、興味があるかたはgethのソースを参考にしてみてください［[ココ](https://github.com/ethereum/go-ethereum/blob/4bb3c89d44e372e6a9ab85a8be0c9345265c763a/params/config.go#L103)と[ココ](https://github.com/ethereum/go-ethereum/blob/4bb3c89d44e372e6a9ab85a8be0c9345265c763a/core/genesis.go)］。
