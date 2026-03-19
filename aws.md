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
export STORAGE_LOCATION=$HOME/anythingllm && mkdir -p $STORAGE_LOCATION && touch "$STORAGE_LOCATION/.env" && sudo docker run -d -p 3001:3001 --cap-add SYS_ADMIN -v "$STORAGE_LOCATION:/app/server/storage" -e STORAGE_DIR="/app/server/storage" mintplexlabs/anythingllm
```

### 4. ネットワークとセキュリティの設定
Lightsailの管理画面にあるネットワーク設定から、ポート3001番の通信を許可してください。
特定の接続元（自分のPCなど）からのみアクセスできるように制限をかけることが、安全性を保つために極めて重要です。

### 5. Ollamaのインストール
```
curl -fsSL https://ollama.com/install.sh | sh
ollama pull qwen3:4b
```

#### 1. 接続許可の設定
標準の状態では、Ollamaは自分自身の内部からの通信しか受け付けません。
Docker経由で接続するために、以下の命令を入力して設定を書き換えます。
```
sudo mkdir -p /etc/systemd/system/ollama.service.d
echo -e "[Service]\nEnvironment="OLLAMA_HOST=0.0.0.0"" | sudo tee /etc/systemd/system/ollama.service.d/override.conf
```

#### 2. 設定の反映と再起動
書き換えた内容をシステムに認識させて、再起動を実行します。
```
sudo systemctl daemon-reload
sudo systemctl restart ollama
```
#### 3. AnythingLLMでの設定値
ブラウザの画面に戻り、設定項目を以下のように入力してください。

- プロバイダーを選択: Ollama
- Ollama URL: http://172.17.0.1:11434
- モデル名を入力: qwen2.5:3b

これで、サーバー上の知能がAnythingLLMと繋がります。172.17.0.1という数字は、Dockerという箱から見たサーバー本体の住所を指しています。

### 4. 動作の最適化
設置が完了した後、動作が重いと感じる場合は設定を見直してください。
軽量なモデルへの変更や、記憶する範囲の制限、GPUの活用といったソリューションを検討します。
計算の無駄を省く最新の技術を環境に合わせて活用することで、処理速度は飛躍的に向上します。
