---
layout: post
title: 使用Intellij IDEA 调试 Erlang OTP应用
date: 2015-03-31 18:30:18
categories: [Erlang]
---

# 方法

按照 [Using Intellij IDEA to write and debug Erlang code](http://zewaren.net/site/?q=node/144) 里介绍的方法可调试OTP应用。

注意要创建1个不会停止的代理文件，才可调试OTP应用。

{% highlight erlang %}
-module(server).
%% -export([]).

-export([debug/0]).

loop_sleep() ->
    timer:sleep(5000),
    loop_sleep().

debug() ->
    %% Start your dependencies here
    application:start(server),
    loop_sleep().
{% endhighlight %}

点击 Edit Configurations，创建1个Name为“debug”的 Erlang Application，填写如下信息：

     Module and function: server debug
     Function arguments: 不填，留空
     Flags for 'erl':  -pa ./apps/server/ebin -pa ./apps/web/ebin -pa ./deps/goldrush/ebin -pa ./deps/lager/ebin -pa ./deps/meck/ebin -pa ./deps/protobuffs/ebin -pa ./deps/mochiweb/ebin -pa ./deps/recon/ebin -boot start_sasl -config config/run -setcookie server -name wxhl_1_1@127.0.0.1

# 问题1

如果提示

     "Failed to interpret modules on node wxhl_1_1@127.0.0.1:server_reader, ... Make sure they are compiled with debug_info option, their sources are located in same directory as .beam files, modules are available on the node."

## 先检查编译选项是否有如下选项：
     {erl_opts, [
         debug_info
     ]}.

## 在确保.erl 文件与 .beam 在相同目录

使用 Intellij IDEA 创建的项目，在使用Build菜单构建项目，可看到 输出目录 out/production/xxx 目录下 包含 .erl 和 .beam 文件。

如果 是使用rebar创建的项目，且放在apps目录下，则可能compile后 .erl 和 .beam 不在相同目录。可创建一个 sync.bat 文件：

     for /R "%SOURCE_DIR%\apps\server\src" %%i in (*.erl) do (
         copy "%%~fi" "%SOURCE_DIR%\apps\server\ebin\%%~nxi"
     )

在 刚创建的 Name为“debug” 的 Debug Configurations 中的 Before launch 中增加执行一个 External Tools，Name 为“sync erl to ebin”，Tool settings 填写如下信息：

     Program: $ProjectFileDir$\script\sync.bat
     其中$ProjectFileDir$为1个宏，对应项目目录

# 问题2

如果输出错误

     ***WARNING*** Unexp msg {trace,<0.806.0>,'receive',
                            {mysql_recv,<0.808.0>,data,
                                <<17,49,56,48,50,51,49,57,52,54,48,50,53,52,56,
                                  48,56,55,1,48,18,233,178,129,232,142,189,231,
                                  154,132,233,135,142,232,155,174,228,186,186,
                                  1,49>>,
                                37}}, info {running,
                                            [{'_Num',14},
                                             {'Packet',
                                              <<17,49,56,48,50,51,49,57,52,54,
                                                48,50,53,52,54,48,48,56,1,48,
                                                15,231,132,149,229,143,145,
                                                231,154,132,231,155,151,232,
                                                180,188,1,49>>},
                                             {'Version',41},
                                             {'Res',
                                              [[18023194602546007,0,
                                                <<232,139,151,230,157,161,231,
                                                  154,132,229,143,164,229,143,
                                                  140,229,164,180,233,190,153>>,
                                                1],
                                               [18023194602546006,0,
                                                <<230,135,166,229,188,177,231,
                                                  154,132,233,173,133,233,173,
                                                  148,229,165,179>>,
                                                1],

     那是在调试情况下，运行速度变慢，有些操作超时，把超时时间加大。对应上述错误，是意外收到mysql返回结果，把执行mysql语句的超时时间加长即可。

# 遗留问题

使用 build 不能编译构建apps下的项目，在Project Structure --> Modules 里配置，把apps下每个项目当做一个Module。

参见：

[Using Intellij IDEA to write and debug Erlang code](http://zewaren.net/site/?q=node/144)
