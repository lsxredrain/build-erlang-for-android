# build-erlang-for-android
Build Erlang For Android

# 在Android上跑命令行程序的常规方法:

  1. 交叉编译
  2. adb push /data/local/tmp/somedir
  3. adb shell
  4. cd /data/local/tmp/somedir
  5. run your bin


但，上述方法对Erlang有一定限制:
`crypto:start().`
执行失败。

具体原因: android7.0之后对加载so做了白名单限制。
`crypto:start().`
方法会通过dlopen加载"$ROOTDIR/lib/crypto-4.3.3/priv/lib/crypto.so"，dlopen因为白名单限制，
执行失败。


# 绕过该限制的方法：

  - 修改rom
  - 开发一个apk壳 

修改rom的方法，过于复杂，破坏android安全限制，且通用型差。

方法2，具体实现:
  1. 开发一个apk壳
  2. 把Erlang整个环境放到assets中
  3. apk运行时，把Erlang部署到 "/data/data/com.your-apk-packagename/erlang"下
  4. adb shell
  5. run-as com.your-apk-packagename
  6. cd erlang
  7. sh bin/erl


# Tools and Code Info

- Ubuntu 16.04.2
- android ndk r18b
- openssl-1.1.0f
- otp_src_21.1
- 小米 mix 2

# NDK toolchain gen(64)

`$NDK/build/tools/make_standalone_toolchain.py --arch arm64 --api 24 --install-dir /tmp/toolchain64_24`


# OpenSSL 编译(64)

参考: https://github.com/leenjewel/openssl_for_ios_and_android.git

`bash ./build-openssl4android.sh android64-aarch64`


# Erlang 编译(64)

参考: 
https://github.com/erlang/otp/blob/master/HOWTO/INSTALL-ANDROID.md
https://bluishcoder.co.nz/2015/06/21/building-erlang-for-android.html

把erl-xcomp-arm64-android.conf置于xcomp目录

1. setup x-compile env
2. ./opt_build autoconf
3. ./opt_build configure --xcomp-conf=xcomp/erl-xcomp-arm64-android.conf
4. ./opt_build boot -a


# android 7.0后对加载so的限制

https://developer.android.com/about/versions/nougat/android-7.0-changes?hl=zh-cn#ndk
https://android.googlesource.com/platform/bionic/+/master/android-changes-for-ndk-developers.md
https://source.android.google.cn/devices/architecture/vndk/linker-namespace
http://jackwish.net/namespace-based-dynamic-linking.html

https://www.jianshu.com/p/4be3d1dafbec

查看平台信息(主要查看android api版本信息)
adb shell getprop ro.build.version.release
adb shell getprop ro.build.version.sdk 
adb shell getprop


# linux gen core dump

  - gcc -g才能产生core
  - ulimit -c unlimited
