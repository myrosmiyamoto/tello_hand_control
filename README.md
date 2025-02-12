# Telloドローンを手の動きで操作するプログラムの説明

## 1. このプログラムは何をするの？
このプログラムは **DJI Telloドローンを手の動きで操作** できるようにするものです。Webカメラの映像を使い、手の動きを検出し、ドローンの移動を自動で指示できます。

## 2. 必要なライブラリ
このプログラムを実行するには、以下のライブラリをインストールしておく必要があります。

```bash
pip install djitellopy opencv-python mediapipe numpy
```

## 3. プログラムの仕組み

### (1) ライブラリのインポート
```python
from djitellopy import Tello
import cv2, time, sys, numpy as np
import mediapipe as mp
from threading import Thread
from queue import Queue
```
ここでは **Telloドローンの制御、カメラ映像の取得、手の動き検出** に必要なライブラリを読み込んでいます。

### (2) Telloドローンの接続と設定
```python
self.tello = Tello()
self.tello.connect()
self.tello.streamon()
```
ここで **Telloドローンとの接続を開始** し、 **映像ストリームをON** にしています。

### (3) カメラ映像の取得
```python
self.cap = cv2.VideoCapture(f'udp://{ip}:{port}')
```
ここで **Telloドローンのカメラ映像** を取得します。

### (4) 手の検出を設定
```python
self.mp_hands = mp.solutions.hands
self.hands = self.mp_hands.Hands(max_num_hands=1, min_detection_confidence=0.7, min_tracking_confidence=0.7)
```
ここで **MediaPipeの手検出機能** を設定します。

### (5) 手の動きを検出
```python
rgb_frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
result = self.hands.process(rgb_frame)
```
カメラの映像をRGBに変換し、MediaPipeを使って手の動きを解析します。

### (6) 手の動きに応じた指示を送信
```python
# 移動量の定数
MOVE_SPEED = 30
commands = {
    "Right": (MOVE_SPEED, 0, 0, 0),
    "Left": (-MOVE_SPEED, 0, 0, 0),
    "Down": (0, 0, -MOVE_SPEED, 0),
    "Up": (0, 0, MOVE_SPEED, 0),
    "Still": (0, 0, 0, 0)  # 停止状態を明示
}

# コマンドを取得し、Telloに送信
if direction in commands:
    x, y, z, yaw = commands[direction]
    self.tello.send_rc_control(x, y, z, yaw)
```
検出した手の動きに応じて、 **Telloに移動コマンドを送信** します。

### (7) キー操作でドローンを制御
```python
elif key == ord('t'):
    self.tello.takeoff()
elif key == ord('l'):
    self.tello.land()
elif key == ord('1'):
    self._toggle_automode(True)
elif key == ord('0'):
    self._toggle_automode(False)
```
`'t'` を押すと **離陸**、 `'l'` を押すと **着陸** するように設定しています。また、`'1'` を押すと **オートモードがON** になり、`'0'` を押すと **オートモードがOFF** になります。

### (8) プログラムの実行と終了
```python
def main():
    ip = '192.168.10.1'
    port = '11111'
    tello = TelloControl(ip, port)
    try:
        tello.run()
    except KeyboardInterrupt:
        tello.stop()
        sys.exit()
```
この関数がプログラムのメイン部分で、Telloの制御を開始します。

## 4. 実行方法
ターミナルまたはコマンドプロンプトで以下のコマンドを実行します。

```bash
python tello_hand_control.py
```

`'t'` を押すと **離陸**、 `'l'` を押すと **着陸** するように設定しています。また、`'1'` を押すと **オートモードがON** になり、`'0'` を押すと **オートモードがOFF** になります。**オートモードがON** にすると、ドローンのカメラが手を動きを検出し、動かした方向にドローンが移動するようになります。

**`'ESC'` キーを押すとプログラムが終了** します。

## 5. まとめ
- **TelloドローンをPythonで制御**
- **手の動きをカメラで検出し、Telloに指示を送信**
- **手の動きに応じてドローンが上下左右に移動**
- **キーボード操作で離陸・着陸などの操作も可能**
- **'1' を押すとオートモードON、'0' を押すとオートモードOFF**
