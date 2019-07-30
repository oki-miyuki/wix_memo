# Wix memo

## このメモについて
[Wix toolset](https://wixtoolset.org/) によりインストーラを作成した時に得たノウハウを記すものです。 
インストーラを作成するにあたって、[WiX 3.6: A Developer's Guide to Windows Installer XML (English Edition)](https://www.amazon.co.jp/dp/B009YW82A0)  
[WiX Cookbook (English Edition)](https://www.amazon.co.jp/dp/B00T0C8EUM) を入手しました。  
しかしながら、ドキュメントに記述されているのかもしれないが、暗黙の了解が多く苦労しました。  
Wix に関しては情報が少なく、簡単そうに見える事が簡単にできませんでした。  
ネットで検索してもインストール・ログを見ろで終わっており、解決されていないものが多かったです。  
ハマった事、自ら調べてコードを書いて解決した事を残します。  

## ノウハウ一覧

 * [Component に 複数Fileをまとめたら駄目なの？](component.md)
 
 * [Upgrade または Uninstall 時にインストールされたファイルのパスを取得する](get_component_path.md)

 * カスタム動作の実装
 
 * [管理者権限が必要な場合の注意点](administrator.md)
 
 * Wix UI の使い方と注意点(断片的な知識)
 
 * インストール先(ターゲット)とProgram Files フォルダ、System フォルダの関係
 
 * Upgrade 時の継承性の無さと対応について、あれこれ
 
 * System フォルダのファイルをバージョンに関係なく強制的に削除・更新する方法
 
 * Target ドライブの変更
 

