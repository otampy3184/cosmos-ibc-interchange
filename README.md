# Cosmos IBC advanced

IgniteCLIが用意しているCosmosSDK Tutorialの[Advanced Module: Interchange](https://docs.ignite.com/guide/interchange)を試し、チェーン間でやりとりされるMessageやModuleの動きについて理解する

## Tutorialについて

今回は異なるチェーン間でTokenのやりとりができるDEX的なApplication Blockchainを用意する.
用意したモジュールによって、ブロックチェーン間で売買注文を作成することができる.
注文ペア、買い注文、売り注文を作成できるCosmos SDKモジュールを作成し、ブロックチェーン間でオーダーブックと売買注文を作成し、それによって、あるブロックチェーンから別のブロックチェーンにトークンをスワップすることが可能になる.

以下の内容がまとめられている.

- Cosmos SDK IBCモジュールの作成
- モジュールで売買注文を受け入れるオーダーブックを作成
- あるブロックチェーンから別のブロックチェーンへIBCパケットを送信
- IBCパケットのタイムアウトと確認応答への対応

## アプリデザイン

Interexchange Moduleがどのように機能するかをまとめる.

このModuleには「オーダーブック」「買いオーダー」「売りオーダー」が存在し、以下のルールに従って取引が行われる

- 最初にトークンのペアが記載されたオーダーブックが作成される
- オーダーブックが存在した場合、ペアとなっているトークンに対して買いオーダーと売りオーダーを作成できる
- オーダーブックは別々のブロックチェーン上のトークンをペアとして登録できる
- やりとりされるBlockchainの双方にDexモジュールが存在する必要がある
- 一つのオーダーに対して同時に存在するオーダーブックは一つだけ

例えば、二つの異なるBlockchainとして「火星チェーン」と「金星チェーン」があるとする.

火星チェーンのネイティブトークン -> marscoin

金星チェーンのネイティブトークン -> venuscoin

火星から金星にTokenを交換する場合

- 金星チェーンはIBC `Voucher` tokenを持ち、`ibc/B5CB286...A7B21307F`のようなDenomを持つ
- VoucherTokenのibc/移行の文字列はIBCを使ってTransferされたTOkenのDenom traceのハッシュ値

ブロックチェーンのAPIを利用することで、Denom traceを上記のハッシュ値から取得できる. Denom traceは`base_denom`と`path`から構成される.

今回の場合、`base_denom`は`marscoin`、`path`はトークンがやりとりされたチャンネルとポートのペア

また、single-hop transferの場合、`path`は`transfer/channel-0`となる

:::note info
`ibc/Venus/marscoin`は同じオーダーブックを用いて売り戻すことはできない。もし交換を「リバース」したい場合は、`ibc/Venus/marscoin`から`marscoin`への新しいオーダーブックを作成する必要がある
:::


## Create the Blockchain

IgniteCLIでBlockchainの下地を作成する

```:
% ignite scaffold chain interchange --no-module
```

```:
% cd interchange
```

## Create the module

新規のModuleとして`dex`を作成する

```:
% ignite scaffold module dex --ibc --ordering unordered --dep bank
```

## OrderBookのBuyとSellのためのCRUDロジックを作成

```;
% ignite scaffold map sell-order-book amountDenom priceDenom --no-message --module dex
% ignite scaffold map buy-order-book amountDenom priceDenom --no-message --module dex
```

## Packetを作成

```:
% ignite scaffold packet create-pair sourceDenom targetDenom --module dex
% ignite scaffold packet sell-order amountDenom amount:int priceDenom price:int --ack remainingAmount:int,gain:int --module dex
% ignite scaffold packet buy-order amountDenom amount:int priceDenom price:int --ack remainingAmount:int,purchase:int --module dex
```

## Order cancel用のメッセージ作成

```:
% ignite scaffold message cancel-sell-order port channel amountDenom priceDenom orderID:int --desc "Cancel a sell order" --module dex
% ignite scaffold message cancel-buy-order port channel amountDenom priceDenom orderID:int --desc "Cancel a buy order" --module dex
```

## Denomトレース用のマッピング作成

```:
% ignite scaffold map denom-trace port channel origin --no-message --module dex
```

## テスト準備

動作確認用に、それぞれ指定のトークンを持つ用にmars.ymlとvenus.ymlファイルを用意する

```yml:
    # venus.yml
    accounts:
    - name: alice
        coins: ["1000token", "1000000000stake", "1000venuscoin"]
    - name: bob
        coins: ["500token", "1000venuscoin", "100000000stake"]
    validator:
    name: alice
    staked: "100000000stake"
    faucet:
    host: ":4501"
    name: bob
    coins: ["5token", "100000stake"]
    host:
    rpc: ":26659"
    p2p: ":26658"
    prof: ":6061"
    grpc: ":9092"
    grpc-web: ":9093"
    api: ":1318"
    genesis:
    chain_id: "venus"
    init:
    home: "$HOME/.venus"
```

```yml:
    # venus.yml
    accounts:
    - name: alice
        coins: ["1000token", "1000000000stake", "1000venuscoin"]
    - name: bob
        coins: ["500token", "1000venuscoin", "100000000stake"]
    validator:
    name: alice
    staked: "100000000stake"
    faucet:
    host: ":4501"
    name: bob
    coins: ["5token", "100000stake"]
    host:
    rpc: ":26659"
    p2p: ":26658"
    prof: ":6061"
    grpc: ":9092"
    grpc-web: ":9093"
    api: ":1318"
    genesis:
    chain_id: "venus"
    init:
    home: "$HOME/.venus"
```