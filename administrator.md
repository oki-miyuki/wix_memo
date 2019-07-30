# 管理者権限が必要な場合の注意点

インストーラで管理者権限が必要な場合には、暗黙の了解があります。

インストーラが管理者権限で実行されるタイミングがあります。
InstallExecuteSequence の InstallInitialize から　InstallFinalize の区間というように、限定されています。  
これに加えて、インストールファイルがコピーされる前に実行したいのか、削除される前に実行したいのか、制約を受けます。  

また、管理者権限で実行するためには、CustomAction に対して Execute="deferred" 及び Impersonate="no" という属性が必要です。

```
<CustomAction Id="CA_InstallOnRemove" BinaryKey="CustomActionDll" DllEntry="RemoveDllAction" Execute="deferred" Impersonate="no" Return="check" />
```

## System Folder の特定ファイルを削除したい
レガシーシステムの保守で、インストーラで古いバージョンに戻したいという事は日常茶飯事にあります。  
ところが、Windows Installer は新しきに流れる事はあっても古きに戻る事は一切考慮されていません。  
そんな事は、あり得ないからと一蹴されています。  
これを実現するためには、カスタム・アクションを書くしかありません。  
そんな場合のタイミングは、以下になります。  

インストール前に特定のファイルを削除したい場合は、InstallExecuteSequence の After InstallInitialize です。
```
<Custom Action="CA_InstallOnRemove" After="InstallInitialize">not Installed</Custom>
```

アンインストール後に特定ファイルを削除したい場合は、InstallExecuteSequence の Before InstallFinalize です。
```
<Custom Action="CA_RemoveDll" Before="InstallFinalize">REMOVE="ALL"</Custom>
```
