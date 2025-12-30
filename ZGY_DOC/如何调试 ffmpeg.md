* https://zhuanlan.zhihu.com/p/467257873
-----
### configure选项（这一步是必须的）
* enable-debug 是最重要的一处，否则编译会启用优化，导致很多调试符号丢失。
```shell
./configure --enable-debug --disable-x86asm --disable-stripping --disable-optimizations
```
```shell
pwd
```