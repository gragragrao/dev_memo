# CUDAの環境設定

## GPUの情報を調べる

```.sh
$ lspci | grep -i nvidia

00:1e.0 3D controller: NVIDIA Corporation GV100GL [Tesla V100 SXM2 16GB] (rev a1)
```

より詳しく...

```.sh
$ lspci -s 00:1e.0 -v  # ↑で確認した左の番号を入れる

00:1e.0 3D controller: NVIDIA Corporation GV100GL [Tesla V100 SXM2 16GB] (rev a1)
  Subsystem: NVIDIA Corporation Device 1212
  Physical Slot: 30
  Flags: bus master, fast devsel, latency 0, IRQ 43
  Memory at 84000000 (32-bit, non-prefetchable) [size=16M]
  Memory at 1000000000 (64-bit, prefetchable) [size=16G]
  Memory at 82000000 (64-bit, prefetchable) [size=32M]
  Capabilities: <access denied>
  Kernel driver in use: nvidia
  Kernel modules: nvidia_drm, nvidia
```

## CUDAのバージョンを調べる

```sh
# 2021/02/14に調べたときはKERNELではCUDA9.0を使っていた
$ nvcc -V
```

## tensorflowからGPUが使えるか確かめる

```py
tf.config.list_physical_devices('GPU')
```

## pytorchからGPUが使えるか確かめる

```py
torch.cuda.is_available()
```

## CUDA周りを完全に消す

```sh
# すでにドライバーがないか確認
$ dpkg -l | grep nvidia
$ dpkg -l | grep cuda

# 古いドライバーの削除
$ sudo apt-get --purge remove nvidia-*
$ sudo apt-get --purge remove cuda-*

$ sudo apt-get autoremove
$ sudo apt-get autoclean
```

## CUDAドライバーを入れる

- [Qiita記事](https://qiita.com/conta_/items/d639ef0068c9b7a0cd12)

```sh
# リポジトリの登録
$ sudo add-apt-repository ppa:graphics-drivers/ppa
$ sudo apt-get update

# ドライバーのインストール
$ sudo apt-get install nvidia-440
$ sudo reboot
```

## 使用するバージョン

### Tensorflow

- nvidia driver：450
- CUDA：11.0
- python：3.8.5
- tensorflow：2.4.0
  - [Tensorflow公式](https://www.tensorflow.org/install/gpu)
  - sudo apt-get install --no-install-recommends nvidia-driver-450 だとうまく行かなかったので、 sudo apt-get install nvidia-450 を実行した

### pytorch

- nvidia driver：440 （460だと強制的にCUDA11.2になる？）
- CUDA：10.2
- python：3.8.5
- pytorch：1.6.0

## その他の便利なコマンド

```sh
# PATH以下で使われている容量を調べる
$ sudo du -sh PATH

# デバイスの空き容量を調べる
$ df -h
```
