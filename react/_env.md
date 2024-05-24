# バージョンマネージャのインストール
- asdf
  - プラグインとして管理できる言語環境が500近くもあり、それぞれの最新バージョンへの追従も早い
```
brew install asdf
echo -e "\n. $(brew --prefix asdf)/libexec/asdf.sh" >> ~/.zshrc
exec $SHELL -l
```

- .default-npm-packages
  - Nodeの任意のバージョンをインストールしたときに、デフォルトで一緒にインストールされるnpmパッケージを登録するもの