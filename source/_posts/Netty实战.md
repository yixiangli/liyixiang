---
title: Netty实战
toc: true
date: 2017-01-24 10:21:07
category: 
	- 技术贴
	- Java
	- 框架
tags: 
    - netty
    - socket
---


# 什么是Netty

## 动态Handler
ChannelPipeline作为ChannelHandler容器，支持运行时动态添加或者删除ChannelHandler，而ChannelHandler主要负责对事件进行拦截或者处理，因此如当业务高峰期（很多业务一般是晚上）时，可以通过系统时间来控制动态添加一个ChannelHandler对系统进行用塞保护，高峰期过去后可以删除达到服务恢复。

### 5.0被废弃了
听到这个消息，博主感到很震惊，之前做breeze框架（http://www.fishfly.cn/2016/09/08/breeze官方文档/） 中主要使用的就是Netty框架天生异步高性能优势，当时Netty5.0才出不久，所以博主选择的是4.0的版本，不过很多5.0的口子都已经开了，如
<!--more-->
```
@Override
	protected void channelRead0(ChannelHandlerContext ctx, FullHttpRequest request) throws Exception {
		// TODO Auto-generated method stub
		messageReceived(ctx,request);
	}
	
	
	
	/**
	 * 
	 * @use 方便今后升级netty5
	 * @param
	 * @return
	 */
	private void messageReceived(ChannelHandlerContext ctx, FullHttpRequest request) throws Exception {
		template.process(ctx, request);
	}
```

万万没想到，作者居然自己给废弃了，那么废弃一定是有原因的，大致的原因作者在自己的GitHub上也进行了说明（https://github.com/netty/netty/issues/4466）  大概看了下，也综合了下其他爱好者的说法，大致是
```
1. netty5 中使用了 ForkJoinPool，增加了代码的复杂度，但是对性能的改善却不明显
2. 多个分支的代码同步工作量很大
3. 作者觉得当下还不到发布一个新版本的时候
4. 在发布版本之前，还有更多问题需要调查一下，比如是否应该废弃 exceptionCaught， 是否暴露EventExecutorChooser等等。
```





