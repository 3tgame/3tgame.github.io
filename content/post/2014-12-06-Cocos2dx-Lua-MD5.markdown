---
layout: post
title: Cocos2dx 增加Lua MD5扩展
date: 2014-12-06 16:50:18
categories: [Cocos2dx, Lua]
---

我们使用的Lua MD5扩展 是 [MD5: Cryptographic Library for Lua](http://keplerproject.org/md5/index.html)，是1个用C语言实现的扩展。构建、安装该扩展的方法，在其页面都有介绍。不过，这里我不是使用其主页上介绍的方法集成到Cocos2dx中。

# 下载Lua MD5扩展

到 GitHub的 [keplerproject/md5](https://github.com/keplerproject/md5)下载整个项目的压缩包。

# 集成Lua MD5扩展

集成方法参照 Coccos2dx集成luasocket的方法，具体如下：

## 新建目录，拷贝文件
在cocos2d-x-3.2\external\lua 目录下新建目录 luamd5，把下载压缩包解压后里面的src里的所有文件复制进 luamd5 目录。

## 在Visual Studio 引入代码
在liblua下新建一个名为luamd5的Filter，把luamd5目录下的文件加入到该Filter下

## 修改Cocos2dx代码
> 其中luasocket的注册是在lua_extensions.c 的 luaopen_lua_extensions中执行的

{% highlight c++ %}
bool LuaStack::init(void)
          |--> luaopen_lua_extensions(_state); 

static luaL_Reg luax_exts[] = {
    {"socket.core", luaopen_socket_core},
    {"mime.core", luaopen_mime_core},
    { "md5.core", luaopen_md5_core },
    {NULL, NULL}
};

void luaopen_lua_extensions(lua_State *L)
{
    // load extensions
    luaL_Reg* lib = luax_exts;
    lua_getglobal(L, "package");
    lua_getfield(L, -1, "preload");
    for (; lib->func; lib++)
    {
        lua_pushcfunction(L, lib->func);
        lua_setfield(L, -2, lib->name);
    }
    lua_pop(L, 2);
}
{% endhighlight %}

*修改内容为*：

   增加引入头文件：#include "luamd5/md5.h"
   
   在luax_exts中增加一项：    { "md5.core", luaopen_md5_core },

# 使用Lua MD5扩展

在自带的lua测试项目 lua-tests 的 src\controller.lua 中加入如下代码：

{% highlight lua %}
local md5 = require"md5"
print(string.rep('a',53) .. "'s md5:" .. md5.sumhexa(string.rep('a',53)))
{% endhighlight %}

更多使用方法、测试例子，可参看压缩包里的test.lua。
