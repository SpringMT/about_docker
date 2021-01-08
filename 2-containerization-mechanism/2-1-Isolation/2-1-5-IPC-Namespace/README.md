# IPC Namespace(IPC 名前空間)
IPC (POSIX/SysV IPC) 名前空間は、共有メモリ・セグメント、セマフォ、メッセージ・キュー(POSIXメッセージキュー)と呼ばれる分離を提供します。

異なる名前空間の共有メモリやセマフォにアクセスを制限できます。

メッセージキューに関しては詳細Unixプログラミング第3版 p529 ではそもそも使わないほうがよいとあります。

共有メモリはmmap(2)を使う場合などですね。

## 参考文献
* https://linuxjm.osdn.jp/html/LDP_man-pages/man7/namespaces.7.html#lbAF
* https://blog.yadutaf.fr/2013/12/28/introduction-to-linux-namespaces-part-2-ipc/
* https://ja.wikipedia.org/wiki/%E3%83%97%E3%83%AD%E3%82%BB%E3%82%B9%E9%96%93%E9%80%9A%E4%BF%A1
* https://docs.docker.jp/engine/reference/run.html
* https://mcommit.hatenadiary.com/entry/2019/07/24/170000
