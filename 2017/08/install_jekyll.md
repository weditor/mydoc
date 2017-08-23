安装jekyll的错误。

# 第一个问题

```shell
/root/# gem install jekyll
Building native extensions.  This could take a while...
ERROR:  Error installing jekyll:
	ERROR: Failed to build gem native extension.

    current directory: /var/lib/gems/2.3.0/gems/ffi-1.9.18/ext/ffi_c
/usr/bin/ruby2.3 -r ./siteconf20170820-3605-bjkvyy.rb extconf.rb
mkmf.rb can't find header files for ruby at /usr/lib/ruby/include/ruby.h

extconf failed, exit code 1
```

## 问题原因

有些库不是以二进制可执行文件提供，而是提供的源码，安装这种ruby库时需要编译，因此需要ruby的头文件，进行动态链接。

## 解决方法

安装头文件

```shell
apt-get install ruby-dev
```

# 第二个问题

```shell
~/work> jekyll new blog1
  Dependency Error: Yikes! It looks like you don't have bundler or one of its dependencies installed. In order to use Jekyll as currently configured, you'll need to install this gem. The full error message from Ruby is: 'cannot load such file -- bundler' If you run into trouble, you can find helpful resources at https://jekyllrb.com/help/! 

```

## 问题原因

这个问题是因为ubuntu自带的ruby没有安装bundler。

## 解决方法

```
gem install bundler
```

bundler是ruby的包管理工具，可以解决包之间的相互依赖。

