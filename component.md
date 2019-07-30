# Component に 複数Fileをまとめたら駄目なの？

複数Fileを1Componentにまとめると、以下の不都合が生じます。

## コピーされないファイルが生じる

```
 <Component Id="FooDLLs" Guid="74FF636D-88BD-4066-9DD3-9D1DE1C4604A">   
   <File Id="x86.a.dll" Name="a.dll" Source="D:¥Src¥x86¥a.dll" />  
   <File Id="x86.b.dll" name="b.dll" Source="D:¥Src¥x86¥b.dll" />  
 </Component>  
 ```
 
 このような構成でインストーラが実行される時、インストール先に a.dll が存在すると、b.dll がコピーされません。
中途半端にファイルが残った状態でインストーラを実行すると必要なファイルがコピーされないままインストールが完了してしまうインストーラが出来上がります。
このような挙動は、誰にも望まれていません。1Component=1Fileにしましょう。  

## インストールされたファイルの場所を取得しづらくなる。

インストーラAPIの中にComponentのパスを取得する関数 MsiGetComponentPath があります。  
指定する引数にFileのIdは含まれません。  
1Component=1Fileにしましょう。  

## ファイルのパスから、XMLを生成するためのツール

ツールセットによるXMLの生成が、うまく動作しなかったので、Component, File の XMLを生成するためのツールをやっつけで書きました。
流石に手書きは、やってられません。32bitと64bitのファイル名が同じだとIdがバッティングするので、32bit版には x86. をIdに加えるようにしました。    

```
// set lib=%lib%;"D:\vcpkg\installed\x64-windows\lib"
// cl /EHsc /GR /MT mk_component.cpp /I D:\vcpkg\installed\x64-windows\include boost_filesystem-vc140-mt.lib ole32.lib

#define WIN32_LEAN_AND_MEAN
#include <windows.h>
#include <combaseapi.h>

#include <iostream>
#include <cstdlib>
#include <iomanip>
#include <fstream>
#include <string>
#include <vector>
#include <boost/filesystem.hpp>

namespace fs = boost::filesystem;

void show_usage() {
  std::cout << "---------------------------" << std::endl;
  std::cout << "mk_component filelist.txt [shared(0=no,1=yes)] [win64(0=no,1=yes)]" << std::endl;
  std::cout << "C:¥¥>mk_component filelist.txt 0 0" << std::endl;
}

std::ostream& operator << (std::ostream& ofs, GUID guid) {
  ofs << std::setfill('0') << std::hex << std::uppercase << std::setw(8) << guid.Data1 << "-" 
    << std::hex << std::setw(4) << guid.Data2 << "-"
    << std::hex << std::setw(4) << guid.Data3 << "-";
  for( int i = 0; i < 2; i++ ) {
    ofs << std::hex << std::setw(2) << (int)guid.Data4[i];
  }
    ofs << "-";
  for( int i = 2; i < 8; i++ ) {
    ofs << std::hex << std::setw(2) << (int)guid.Data4[i];
  }
  return ofs;
}

std::string get_filename( fs::path p ) {
  std::string result = p.filename().generic_string();
  std::replace( result.begin(), result.end(), '-', '_' );
  return result;
}

int main(int argc, char* argv[]) {
  if( argc < 2 ) {
    show_usage();
    return 1;
  }
  std::ifstream ifs;
  ifs.open( argv[1] );

  if( !ifs.good() ) return 1;

  std::vector<std::string> components;
  
  std::string x86 = "CP_x86.";
  std::string fx86 = "";
  if( argc >= 4 && std::atoi(argv[3]) ) {
    x86 = "CP_";
    fx86 = "x86.";
  }
  std::string opt = "";
  if( argc >= 3 && std::atoi(argv[2]) ) {
    opt = " Shared=\"yes\"";
  }
  std::string str;
  while(std::getline(ifs, str)) {
    if( str.empty() ) continue;
    GUID guid;
    HRESULT hr = CoCreateGuid( &guid );
    if( FAILED(hr) ) {
      std::cerr << "Fail to create GUID: " << std::hex << std::endl;
      return 1;
    }
    fs::path pth(str);
    std::string cid = x86 + get_filename(pth);
    std::cout << "<Component Id=\"" << cid << "\" Guid=\"" << guid << "\"" << opt << ">\n";
    std::cout << "<File Id=\"" << fx86 << get_filename( pth ) << "\" Name=\"" << pth.filename().generic_string() << "\" Source=\"" << pth.generic_string() << "\" />\n";
    std::cout << "</Component>\n";
    components.push_back( cid );
    //std::cout << pth.filename().generic_string() << std::endl;
  }
  
  std::cout << std::endl;
  std::cout << std::endl;
  for( size_t i = 0; i < components.size(); i++ ) {
    std::cout << "<ComponentRef Id=\"" << components[i] << "\"/>\n";
  }
  return 0;
}
```
