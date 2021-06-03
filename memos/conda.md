# condaで作成した仮想環境をjupyter notebookに追加する方法

## 目的： tensorflow_p36 というカーネルが壊れたので、作り直す

```sh
$ conda create -n tensorflow_p36 python=3.6   # condaの仮想環境を作成する
$ source activate tensorflow_p36  # 仮想環境に入る
$ conda install ipykernel tensorflow numpy pandas matplotlib scikit-learn tqdm  # 必要なパッケージをinstall
$ ipython kernel install --user --name=tensorflow_p36  # jupyter notebook にカーネルを追加
    Installed kernelspec tensorflow_p36 in /home/ubuntu/.local/share/jupyter/kernels/tensorflow_p36  # ここのフォルダに設定ファイルが入っている
```

- jupyter notebook 上でのカーネルの表示名を変更するには、設定ファイルが入っているフォルダ（今回なら /home/ubuntu/.local/share/jupyter/kernels/tensorflow_p36 ）に kernel.json というファイルがあるので、そこの Display Name を変更すればOK。

## その他使用したコマンド

```sh
conda info -e  # condaの仮想環境一覧
conda list  # （現在の環境の）パッケージ一覧
conda env remove -n tensorflow_p36  # 仮想環境を削除する

jupyter notebook list  # running中のnotebook一覧
jupyter notebook stop 8888  # notebookを終了する

jupyter kernelspec list  # jupyter notebook のカーネル一覧
```

## jupyter notebookでのpythonパスを書き換える必要がある

```sh
$ jupyter kernelspec list
> tensorflow_p38    /home/ubuntu/.local/share/jupyter/kernels/tensorflow_p38  # ここに設定ファイルがある

$ less /home/ubuntu/.local/share/jupyter/kernels/tensorflow_p38/kernel.json
{
 "argv": [
  "/home/ubuntu/anaconda3/bin/python",  # ここのpythonを使う（＊）
  "-m",
  "ipykernel_launcher",
  "-f",
  "{connection_file}"
 ],
 "display_name": "Environment (conda_tensorflow_p38)",
 "language": "python"
}
```

- （＊）にanacondaの仮想環境のpythonパスを通す
  - /home/ubuntu/anaconda3/envs/tensorflow_p38/bin/python

## anacondaのフォルダが肥大化した時にcleanする

```sh
$ conda clean --all
仮想環境やpackage, cacheなどが圧迫する可能性がある
```

## Anacondaが死ぬケース

- baseの環境で conda install python=3.8 とかやると死ぬ（condaコマンドが使えなくなる）
- conda module の version が変更後の python と incompatible になってしまったため

### Anacondaを完全に消去してインストールし直す

```sh
$ rm -rf ~/anaconda3

# .zshrcからanacondaへのpathを削除する
$ rm -rf ~/.condarc ~/.conda ~/.continuum

# Anacondaのインストール
# https://www.anaconda.com/products/individual#linux
$ wget https://repo.anaconda.com/archive/Anaconda3-2020.11-Linux-x86_64.sh
$ sudo sh Anaconda3-2020.11-Linux-x86_64.sh 

# 書き換えの権限を与える
$ sudo chown -R ubuntu /home/ubuntu/anaconda3
```
