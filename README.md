# build tensorflow r1.13 from source on windows

Basically follow https://tensorflow.google.cn/install/source_windows  but  make  some  modifications.

## 1 install vs2015

## 2 add a  lds  file  like  tf_sy_msvc.lds  in  the  folder,  modify  BUILD  file  as  follows:
1> add  "//tensorflow:windows"  content  in  tf_cc_shared_object  linkopts  with  name  =  "libtensorflow_cc.so"
from []
to   [
       "-def:"  +    #  This  line  must  be  directly  followed  by  the  exported_symbols_msvc.lds  file
       "$(location  //tensorflow:tf_sy_msvc.lds)",	
     ]
     
2> in  the  following  deps,  add  "//tensorflow:tf_sy_msvc.lds"
3> in  the  following  exports_files,  add  "tf_sy_msvc.lds"
reference: https://github.com/tensorflow/tensorflow/issues/22047

## 3 build from source
type this command in your workdir(such as D:/tensorflow-r1.13/)
''' bazel  build  --config=opt  --config=cuda  --define=no_tensorflow_py_deps=true  --copt=-nvcc_options=disable-warnings  //tensorflow:libtensorflow_cc.so  '''

## 4 rename  libtensorflow_cc.so  to  tensorflow.dll

## 5 extract  symbols  from  dll  file  to  lib  file
'''dumpbin  /EXPORTS  .\tensorflow.dll  >  .\tensorflow.txt '''

## 6 delete  context  exclude  symbols  like
1        0  01A4F550  ??0?$MaybeStackArray@D$0CI@@icu_62@@AEAA@AEBV01@@Z

## 7 remove  the  ordinal  and  hint  RVA  and  only  keep  name
'''awk  -F  '  '  '{print  $NF}'  .\tensorflow1.txt  >  .\tensorflow.def '''
** if  encounter  scrambled  code,  transfer  tensorflow1.txt  to  utf8  format  first

## 8 generate  lib  file
'''lib  /def:".\tensorflow.def"  /out:".\tensorflow.lib"  /machine:x64 '''

## 9 config  env  in  vs2015
if  still  have  link  error,  add  unresolved  external  symbols  in  tf_sy_msvc.lds  in  step  2  and  repeat  following  steps.

Include path in VS:
D:\tensorflow-r1.13\bin\lib
D:\tensorflow-r1.13\bin\include
D:\tensorflow-r1.13\bin\include\tensorflow\include
D:\tensorflow-r1.13\bazel-tensorflow-r1.13\external\eigen_archive
D:\tensorflow-r1.13\bin\include\tensorflow\include\external\com_google_absl
D:\tensorflow-r1.13\bin\include\tensorflow\include\external\protobuf_archive\src

if encounter syntax error in logging.h

C:\tensorflow\tensorflow/core/platform/default/logging.h(232): error C2589: '(': illegal token on right side of '::'  
C:\tensorflow\tensorflow/core/platform/default/logging.h(232): error C2059: syntax error: '::'  
C:\tensorflow\tensorflow/core/platform/default/logging.h(232): error C2143: syntax error: missing ';' before '{'

it is the windows problem of confusion between max() and std::max(). 
solution: change std::xxx::max() to (std::xxx::max)()
