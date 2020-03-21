---
layout: post
title: Erlang 日志框架lager的使用
date: 2015-04-01 21:30:18
categories: [Erlang]
---

# 介绍

lager 是一个Erlang 日志框架，github地址为 https://github.com/basho/lager。
特性为：支持分日志级别；按特定时间、日志文件大小滚动日志文件；终端以不同颜色输出信息；日志信息包含模块、函数、行号、进程号信息。

# 启动lager应用

在 server_app.erl 中 执行 lager:start().

# 支持的日志级别

从上到下，级别递增。设置日志级为特定级别，会把该级别及以上级别的日志输出。

    debug
    info
    notice
    warning
    error
    critical
    alert
    emergency

# 运行时改变日志级别

    lager:set_loglevel(lager_console_backend, debug).
    lager:set_loglevel(lager_file_backend, "console.log", debug).

# 记录日志

如记录Level级别的日志，有2种方式

## 1. lager:Level(Msg, Format)).

这种需要 在编译选项中，增加
    {erl_opts, [
        {parse_transform, lager_transform},
    ]}.

输出日志为：

    2015-04-01 20:59:16.024 [info] <0.151.0>@server_reader:get_socket_data:102 send data reply ok

## 2. lager:log(Level, [{pid,self()},{module, ?MODULE},{line, ?LINE}], Format, Msg)).

这种方式不需要加 {parse_transform, lager_transform} 编译选项。

# 配置日志文件存放的根目录
   {lager, [
     {log_root, "/var/log/hello"}
   ]}.

# 追踪（可用于根据日志调试）

## 显示所有日志级别和追踪
    lager:status().

## 追踪特定模块特定函数的日志：
    lager:trace_file("log/trace.log", [{module, mymodule}, {function, myfunction}], warning).

## 追踪特定进程的日志：
    lager:trace_console([{pid, "<0.410.0>"}]).

注意：不能使用 lager:log(debug, Format, Msg), 而应使用 lager:debug(Format, Msg)，否则根据PID会追踪不到。

## 清除追踪，使用 stop_trace：
    {ok, Trace} = lager:trace_file("log/error.log", [{module, mymodule}]),
    lager:stop_trace(Trace).

## 停止所有trace
    lager:clear_all_traces().

## 根据属性重定向日志

    lager:warning([{request, RequestID},{vhost, Vhost}], "Permission denied to ~s", [User]).
现可根据这些额外属性进行追踪
    lager:trace_console([{request, '>', 117}, {request, '<', 120}]).

参见：

[basho/lager](https://github.com/basho/lager)
