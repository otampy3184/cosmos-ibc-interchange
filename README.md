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




## Create the Blockchain

```:
% ignite scaffold chain interchange --no-module
```

```:
% cd interchange
```

## Create the module

```:
% ignite scaffold module dex --ibc --ordering unordered --dep bank
```