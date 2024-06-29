> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/LiuChunfu/p/6804437.html)

零、引言
====

之前写 FTP 工具库，用的是 ftp4j，他使用其他非常简单方便，但是在细节上提供的可选项比较少（当然也可能是我了解不够深刻）

最新的项目重写了 FTP 工具类，选择了 apache net 中的 ftp 库，选择 apache 的原因有如下几个：1 是我相信 apche 2 是它的注释完善（apache 的代码注释值得每一位程序猿学习） 3 是提供的可选配置（FTPConfig）有跟多选择（比如主动被动模式，断点续传等）。

本人在使用 ftp4j 判定文件是否存在的时候，通过 API（具体那个忘了）获取 FTPFile 对象时，在部分 FTP 服务时（filezilla）会遇到返回值为 null 的问题（备注：原因是时间格式化的问题），当时解决判定文件是否存在改用只通过获取文件名来解决。

本次改用 apache net 的时候，在使用 API listFiles() 获取的也是 null，经过详细查看源码，发现了一个 API 是 mlistFile() 这样获取结果 OK。

一、Apache 的 mlistFile 源码分析
=========================

源码如下：

[![](http://assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
    /**
     * Get file details using the MLST command
     *
     * @param pathname the file or directory to list, may be {@code null}
     * @return the file details, may be {@code null}
     * @throws IOException on error
     * @since 3.0
     */
    public FTPFile mlistFile(String pathname) throws IOException
    {
        boolean success = FTPReply.isPositiveCompletion(sendCommand(FTPCmd.MLST, pathname));
        if (success){
            String reply = getReplyStrings()[1]; //5:check /* check the response makes sense.
             * Must have space before fact(s) and between fact(s) and filename
             * Fact(s) can be absent, so at least 3 chars are needed.
             */
            if (reply.length() < 3 || reply.charAt(0) != ' ') {
                throw new MalformedServerReplyException("Invalid server reply (MLST): '" + reply + "'");
            }
            String entry = reply.substring(1); // skip leading space for parser
            return MLSxEntryParser.parseEntry(entry);
        } else {
            return null;
        }
    }

```

[![](http://assets.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

我看源码大概分析出了 2 点：

1. 此方法调用的是 MLST，而 listFile 调用的是 LIST。问题来了：什么是 MLST？？

2. 代码方法体第 4 行: **String reply** **= getReplyStrings()[1]; 获取返回码其目的是为了判定 FTP 服务器是否支持 MLST？？有些 FTPServer 不支持？**

二、什么是 MLST（红色部分）
================

Older servers supports LIST command only for directory listing, this way the FTP client gets a non-userfriendly raw format for parsing, and only FTP clients knows what it meaning. Since the file timestamp based on the server timezone, it makes more different to doing FTP synchronize with folders and files because of there is no way to get current file timestamp in the server.

**旧的服务对目录列表仅支持 LIST 命令，这样 FTP 客户端会获得一个非用户友好的待解析的原始格式，仅仅只有 FTP 客户端知道它的意义。由于文件的时间戳基于服务器的时区，它使得在 FTP 对目录和文件同步时有许多的不同，因为没有方法获得服务器上当前文件的时间戳。**

For example there is a normally LIST format of raw directory:

-rw-r--r-- 1 user user 7080 Mar 9 05:24 faq.html

例如：这是对原始目录的普通 LIST 命令格式

-rw-r--r-- 1 user user 7080 Mar 9 05:24 faq.html

Is it readable for you?

是否可读呢？

The MLSD command provided by newer servers to gives users a standarded, detailed, readable directory listing, by sending MLSD command through FTP clients,the server returns accurate file information such as file create time, modified time, size and file owner. Since MLSD directory listing includes file modified time in UTC, so it's very useful for FTP client to converts remote file's timestamp to your Local time when synchronize folders. Also the MLST command could be used to get timestamp of single remote file only.

**新的服务器提供的 MLSD 命令通过 FTP 客户端发送 MLSD 命令，服务器收集文件信息，如文件创建时间，修改时间，文件大小及文件所有用，向用户返回一个标准，详细且可读格式的目录列表。由于 MLSD 目录列表包含 UTC 格式的文件修改时间，因此这对于 FTP 客户端非常有用，当需要同步目录时，它可转换远程文件的时间戳到你的本地时间。同时，MLST 命令也被用于获得单个远程文件的时间戳。**

三、结尾
====

最近打王者有些沉迷，该反省。

**学知识，做技术，不能不求甚解。**