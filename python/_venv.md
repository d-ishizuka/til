# Macでvenvを使う

## 1.pyenvのインストール
```
brew update
brew install pyenv
pyenv --version
```

## 2.shell設定(zsh)
```
echo 'PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.zshrc

# .zshrcに設定を反映
exec zsh

# PATHに./pyenv/shimsが追加されていることを確認
echo $PATH /Users/xxxxx/.pyenv/shims:/Users/xxxxx/.pyenv/bin:/usr/local/bin:/usr/bin:/usr/sbin:/sbin
```

## Pythonのインストール
```
# サポートしているPythonのバージョンを確認する
pyenv install --list

# サポートされているバージョンをインストール
pyenv install 3.12.3
```

## 環境の設定を行う
```
# 特定のディレクトリでのみpythonを設定する
cd <作成したプロジェクトフォルダ>
pyenv local 3.12.3

# グローバルでpythonのバージョンを設定する
pyenv global 3.12.3
```

## 仮想環境の作成
```
python3 -m venv <仮想環境名>
```

## 作成した仮想環境をアクティベートする
```
source <仮想環境名>/bin/activate
```

## デアクティベート
```
deactivate
```