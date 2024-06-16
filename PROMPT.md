# 出力

仕様を満たすコードの出力、次を含む

- main.go
- app.yaml
- go.mod
- README.md

生成物の冒頭に、ChatGPTによって生成された旨を生成日時とともにコメントする

# 仕様

##　概要

GAEからffmpegでrtmpのストリームを流し続ける

- 流す機能
- 流し始める機能
- 止める機能

## 環境

- ワーキングディレクトリは ~/go/src/github.com/n0mimono/auto-live-streamer
- 言語はGo、GAEにディプロイする

## API

サーバ

- http://loclahost/api/v1

以下の通り

- 疎通確認
    - /ping
    - get して 200 を返す
- ストリームを作成
    - /start_stream
    - post, json
    - rtmp のキーを渡す
    - ストリームを作成し、成功したら 200 を返す
        - すでに作成済みのストリームがある場合、エラーを返す
        - rtmp のキーが無効な場合、エラーを返す
- ストリームを終了
    - /stop_stream
    - post, json
    - rtmp のキーを渡す
    - ストリームを削除し、成功したら 200 を返す
        - ストリームが存在しない場合、エラーを返す
        - rtmp のキーが無効な場合、エラーを返す
- ストリームの取得
    - /streams
    - get
    - 存在しているストリームを取得

## ストリーム

ストリームの作成時は、goroutine で次に相当する処理を行う

```
ffmpeg -re -loop 1 -f image2 -i path/to/your/image.png -f lavfi -i anullsrc -c:v libx264 -preset ultrafast -tune stillimage -pix_fmt yuv420p -b:v 200k -maxrate 200k -bufsize 400k -c:a aac -ar 44100 -b:a 64k -f flv rtmp://a.rtmp.youtube.com/live2/your_stream_key
```

ストリームの終了時は上記を止める

path/to/your/image.png はデフォルトでは imgs/default.png

## その他

- 異なるキーに対して、ストリームは複数所持できる
- オプションとして、流すストリームの静止画のパスをストリーム作成のAPIで渡せる
- 実行中は適度な頻度でログを表示する