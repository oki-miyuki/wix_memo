# Upgrade または Uninstall 時にインストールされたファイルのパスを取得する

ユーザがインストール先を変更した場合に、どこにインストールをされたか？という情報は、レジストリに書き込んで保管するのが定石となっています。  
だがしかし、実際に実装してみると思うようにコントロールできなくてイライラする割には、心配する事が多いです。  
(レジストリが消されたら？呼び出すタイミングによってはインストール先が取得できない。癖のある変数…組み込みの挙動との綱引きに翻弄される。書き方によってはインストーラが失敗する。)　　
キーとなるファイルから、インストールされているパスを取得する手法が有効です。  

以下の a.dll がインストールされた場所を取得する関数の使い方です。  

 <Component Id="FooDLLs" Guid="74FF636D-88BD-4066-9DD3-9D1DE1C4604A">   
   <File Id="x86.a.dll" Name="a.dll" Source="D:¥Src¥x86¥a.dll" />  
 </Component>  
 
アンインストール時、または、アップグレード時に取得する事が可能です。  
```
/// Get File Path from 'Component Guid'
/// @param hInstall MSI-Handle
/// @param component_guid Component Guid format=L"{XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXX}"
/// @retval File path
/// @notice if not found path. return value is empty string.
std::wstring getInstalledFilePath(MSIHANDLE hInstall, const wchar_t* component_guid) {
  std::wstring productcode(39, 0);
  MsiGetProductCode(component_guid, &productcode[0]);
  while (1) {
    std::wstring path(4096, 0);
    DWORD dwSize = path.size();
    int res = MsiGetComponentPath(&productcode[0], component_guid, &path[0], &dwSize);
    if (res <= 0) {
      //CMsiWriteLog(hInstall, (boost::wformat(L"Component = %s not found") % component_guid).str().c_str());
      path.resize(0);
      path.shrink_to_fit();
      return path;
    }
    path.resize(dwSize);
    path.shrink_to_fit();
    return path
  }
}
```

インストール時のカスタムアクションは、not Installed を指定する
<Custom Action="CA_Bar" After="LaunchConditions">not Installed</Custom>

アンインストール時のカスタムアクションは、Remove="ALL" を指定する
<Custom Action="CA_Foo" After="InstallInitialize">Remove="ALL"</Custom>

アップグレード時のカスタムアクションは、 Installed を指定する
<Custom Action="CA_Bar" After="LaunchConditions">Installed</Custom>

アンインストール時、または、アップグレード時
<Custom Action="CA_Bar" After="LaunchConditions">Installed OR Remove="ALL"</Custom>

MsiGetComponentPath API のドキュメントが酷い。  
まず、GUID の入力フォーマットが書かれていないので、GUIDパラメータに何を渡せば動作するのかわからない。  
戻り値も定数が並んでいるだけで、それぞれの定数にどういう意味があるのか読み取りにくい。  
この関数をまともに動作させるのに半日ぐらい試行錯誤しました。  
Visual Studio 付属のインストーラ・プロジェクトでさえ、ファイルの場所を特定する方法がわからなくて、ディレクトリを変数にセットしないとプログラムを呼び出せない始末です。  

