---
title: JavaIO
toc: true
date: 2017-02-08 18:29:41
category: 
	- 技术贴
	- Java
	- 基础
tags: 
    - I/O
---

# I/O
input/output的缩写，是一组输入输出接口。计算机是一种高效处理数据的工具，必然会涉及到输入输出。

## Java中的I/O
概念一
    输入与输出：一定要清楚，要以程序（计算机）为本体，输入就是向程序输入的流，而输出就是程序向外输出，而不是以人（程序员）为主体。


### 基础流

类型    |  输入流  |  输出流
 --- | --- | ---  
字节流  |   InputStream  |  OutputStream
字符流  |   Reader   | Writer

### 节点流
<!--more-->
type  |  column1  | column2  | column3  | column4
--- | --- | --- | --- | ---
基类 | InputStream |  OutputStream | Reader | Writer
文件流  | FileInputStream |  FileOutputStream | FileReader | FileWriter
数组流  | ByteArrayInputStream |  ByteArrayOutputStream | CharArrayReader | CharArrayWriter
管道流  | PipedInputStream |  PipedOutputStream | PipedReader | PipedWriter

### 处理流
type  |  column1  | column2  | column3  | column4
--- | --- | --- | --- | ---
基类 | InputStream |  OutputStream | Reader | Writer
转换流  |  |   | InputStreamReader | OutputStreamWriter
缓冲流  | BufferedInputStream |  BufferedOutputStream | BufferedReader | BufferedWriter
数据流  | DataInputStream |  DataOutputStream |  | 
对象流  | ObjectInputStream |  ObjectOutputStream |  | 
打印流  |  |  PrintStream |  | PrintWriter