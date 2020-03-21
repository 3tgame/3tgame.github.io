---
layout: post
title:  "在Visual Studio中配置环境，用于C代码调用Lua代码"
date:   2014-01-06 01:22:18
categories: [Lua, C]
---

# 一、使用安装LuaForWindow后的库文件

该方法不支持调试lua.lib的C代码

参见  [lua的VS或者VC环境的搭建调试] (http://blog.csdn.net/pbymw8iwm/article/details/8191947)

步骤：

1. 安装LuaForWindow，

2. 创建C++工程，配置环境

     项目属性-> 配置属性 -> C/C++ -> 常规 -> 附加包含目录，设置 D:\Lua\5.1\include

     项目属性-> 配置属性 -> 链接器 -> 常规 -> 附加库目录，设置 D:\Lua\5.1\lib

     项目属性-> 配置属性 -> 链接器 -> 输入 -> 附加依赖项，新增  lua5.1.lib;lua51.lib

3. 创建cpp文件，输入如下代码：

    {% highlight c++ linenos %}
    extern "C" {
    #include <lua.h>
    #include <lauxlib.h>
    #include <lualib.h>
    } 
    //#pragma comment(lib, "lua5.1.lib")

    const char *buf = "print('hello, world!')";
    int main(int argc, char* argv[]) {
        lua_State *L = lua_open();     //创建一个指向lua解释器的指针
        luaopen_base(L);  //
        luaL_dostring(L,buf);
        lua_close(L);

        return 0;
    }
    {% endhighlight %}

4. 运行程序，即可

# 二、从Lua代码生成lib

该方法支持调试lua.lib的C代码

参见 [在VS2005中配置LUA](http://www.cppblog.com/lai3d/archive/2008/10/29/65465.html)

步骤：

1. 新建一个空的Console工程，在这里该工程名暂为“lua”

2. 将src中除了“lua.c”与“luac.c”以外的全部文件拷贝到该项目文件夹下，并添加进项目工程中。

3. 更改[项目属性]->[配置属性]->[常规]->[项目类型]为“静态库文件(.lib)”

4. 执行生成，即可在生成目录下得到lua.lib

5. 新建另一个空的console工程，添加1个空的源文件，输入一下代码（由于没有设置项目配置，代码中使用了绝对路径，自行修改路径）：

    {% highlight c++ linenos %}
    extern "C" {
    #include "F:\\My Documents\\Visual Studio 2012\\Projects\\test\\lua\\lua.h"
    #include "F:\\My Documents\\Visual Studio 2012\\Projects\\test\\lua\\lauxlib.h"
    #include "F:\\My Documents\\Visual Studio 2012\\Projects\\test\\lua\\lualib.h"
    } 
    #pragma comment(lib, "F:\\My Documents\\Visual Studio 2012\\Projects\\test\\Debug\\lua.lib")

    const char *buf = "print('hello, world!')";
    int main(int argc, char* argv[]) {
        lua_State *L = luaL_newstate();
        luaL_openlibs(L);
        luaL_dostring(L,buf);
        lua_close(L);

        return 0;
    }
    {% endhighlight %}

# 三、在项目中直接添加Lua代码

该方法支持调试lua.lib的C代码

步骤：

1. 新建一个空的Console工程，在这里该工程名暂为“testLua”

2. 该项目文件夹中新建文件夹lua，将src中除了“lua.c”与“luac.c”以外的全部文件拷贝到该文件夹下，并添加进项目工程中。

3. 添加1个空的源文件，输入一下代码：

    {% highlight c++ linenos %}
    extern "C" {
    #include "lua/lua.h"
    #include "lua/lauxlib.h"
    #include "lua/lualib.h"
    } 

    const char *buf = "print('hello, world!')";
    int main(int argc, char* argv[]) {
        lua_State *L = luaL_newstate();
        luaL_openlibs(L); 
        luaL_dostring(L,buf);
        lua_close(L);

        system("pause");
        return 0;
    }
    {% endhighlight %}
