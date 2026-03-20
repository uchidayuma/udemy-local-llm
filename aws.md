## AWS Lightsailを使用したAnythingLLM構築ガイド

### 1. サーバー（インスタンス）の作成
AWSの管理画面からAmazon Lightsailを選択し、新しいインスタンスを作成してください。
＊オペレーションシステム（OS）
OSはUbuntuを選びます。
ローカルAIの動作はサーバーの性能に依存するため、メモリが8GB以上のプランを選択することが安定した運用の第一歩です。

### 2. 環境の準備
サーバーに接続し、土台となるシステム（Docker）を導入します。
以下の命令を順番に実行してください。
```
sudo apt-get update
sudo apt-get install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
```

### 3. AnythingLLMのインストールと起動
データを保存するための場所を作成し、AnythingLLMを起動させます。
以下の命令をコピーして実行してください。
```
export STORAGE_LOCATION=$HOME/anythingllm && mkdir -p $STORAGE_LOCATION && touch "$STORAGE_LOCATION/.env" && sudo docker run -d -p 3001:3001 --cap-add SYS_ADMIN -v "$STORAGE_LOCATION:/app/server/storage" -e STORAGE_DIR="/app/server/storage" -e JWT_SECRET="c3bd634381ccb9994b77c254c98c281c3f59eb215e3c3ca3903af7de5dcbd9fd" mintplexlabs/anythingllm
```

### 4. ネットワークとセキュリティの設定
Lightsailの管理画面にあるネットワーク設定から、ポート3001番の通信を許可してください。
特定の接続元（自分のPCなど）からのみアクセスできるように制限をかけることが、安全性を保つために極めて重要です。

### 5. AWS Bedrock を活用した爆速 AI 環境の構築手順

#### 1. AWS Bedrock でモデルの使用許可を取る
まず、AWSが提供するAIモデルを使える状態にします。
1. AWS管理画面にログインし、検索窓で Bedrock と入力して選択します。
2. 左側のメニューから、一番下にある Model access を選びます。
3. 画面右上の Modify model access ボタンを押します。
4. Anthropic の項目にある Claude 3.5 Sonnet や Claude 3 Haiku にチェックを入れます。
5. 一番下の Next ボタンを押し、内容を確認して送信します。
通常、数分から数十分でステータスが利用可能に変わります。

#### 2. 接続用の合鍵（アクセスキー）を作成する
AnythingLLMがAWSに安全に接続するための鍵を作ります。
1. AWSの検索窓で IAM と入力して選択します。
2. 左メニューの ユーザー から ユーザーを作成 を選びます。
3. ユーザー名に anythingllm-user など、分かりやすい名前を入力して次へ進みます。
4. 許可のオプション で 既存のポリシーを直接アタッチ を選びます。
5. 検索窓に AmazonBedrockFullAccess と入力し、表示された項目にチェックを入れて次へ進み、作成を完了させます。
6. 作成したユーザーの詳細画面を開き、セキュリティ認証情報 タブを選択します。
7. アクセスキーを作成 を押し、ユースケースで その他 を選んで次へ進みます。
8. 発行された アクセスキー ID と シークレットアクセスキー を必ず控えてください。これは一度しか表示されない重要な情報です。

#### 3. AnythingLLM で Bedrock を設定する
構築済みのAnythingLLMの知能を、自前サーバーからBedrockへ切り替えます。
1. ブラウザでAnythingLLMの管理画面を開きます。
2. 設定（Settings）から LLM Provider を選択します。
3. プロバイダーの一覧から AWS Bedrock を選びます。
4. 以下の情報を入力します。
   Region: ap-northeast-1（東京）など、モデルの使用許可を取った地域を選びます。
   Access Key ID: 先ほど控えた ID を貼り付けます。
   Secret Access Key: 先ほど控えた シークレットキー を貼り付けます。
5. Model の項目で Claude 3.5 Sonnet を選択し、保存（Save）ボタンを押します。

## 4. 動作確認
チャット画面に戻り、適当な質問を投げてみてください。
これまでの動作が嘘のように、瞬時に回答が返ってくれば成功です。

AWSでドメインを取得し、Cloudflareを組み合わせて安全な通信（HTTPS）を実現する手順をまとめました。

この構成にすることで、サーバーの設定を複雑にすることなく、スマホや外部から安心してアクセスできる環境が整います。

-----

# AWSでのドメイン取得とCloudflareによるHTTPS化の手順

## 1\. AWS Route 53 でドメインを取得する

1.  AWS管理画面の検索窓で Route 53 と入力して選択します。
2.  登録済みドメイン のメニューから ドメインの登録 ボタンを押します。
3.  希望するドメイン名を入力して検索し、空いていればカートに入れてチェックアウトに進みます。
4.  氏名や住所などの連絡先情報を入力し、注文を完了させます。取得完了まで数分から数時間かかる場合があります。

## 2\. Cloudflare にドメインを登録する

1.  Cloudflareのアカウントを作成してログインし、サイトを追加 をクリックします。
2.  AWSで購入したドメイン名を入力します。
3.  プラン選択画面で、ページ下部にある Free（0円） を選択して続行します。
4.  現在のDNS設定が自動で読み込まれるので、そのまま次へ進みます。

## 3\. ネームサーバーを書き換える

ここが最も重要な連携作業です。

1.  Cloudflareの画面に、2つのネームサーバー（例：https://www.google.com/search?q=dani.ns.cloudflare.com など）が表示されます。これをコピーします。
2.  AWSの Route 53 画面に戻り、登録済みドメイン から購入したドメインを選択します。
3.  ネームサーバーの追加/編集 をクリックし、もともと入っているアドレスを消して、Cloudflareの2つのアドレスを貼り付けて保存します。

## 4\. HTTPS（鍵マーク）を有効にする

1.  Cloudflareのメニューから SSL/TLS を選択します。
2.  暗号化モードを フル または フレキシブル に設定します。
3.  エッジ証明書 のタブで、常にHTTPSを使用 をオンにします。
    これで、ドメインに自動で証明書が発行され、安全な通信ができるようになります。

