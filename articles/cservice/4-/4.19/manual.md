# 自定义IRIS拦截插件

微服务支持用户自定义的IRIS拦截插件，包含客户端调用方和服务端接收方的插件编写，整个过程包含两个步骤分别是编写插件和注册插件。

下文将分别演示客户端调用方拦截插件和服务端接收方拦截插件的开发示例。

## 客户端调用方拦截插件

1. 编写客户端调用方拦截插件(其中包名为示例, 应根据项目变动):
		
		package com.yonyou.xbiz.plugin;

		public class XBizBeforeInvokePlugin implements IBeforeInvoke {

			@Override
			public String getPluginName() {
				return this.getClass().getName();
			}

			@Override
			public int order() {
				return PluginConstants.PLUGIN_START_INDEX + 6;
			}

			@Override
			public void run(InvokeRequest req, InvokeResponse resp, InvokeChain chain) {

				RPCRequest rpcRequest = req.getRequest(RPCRequest.class);
				RemoteInvocation remoteInvocation = rpcRequest.getRemoteInvocation();

				//这里填写需要添加自定义上下文及对应值
				remoteInvocation.addAttribute("ctxCode", "自定义上下文Code");
				remoteInvocation.addAttribute("bizCode", "自定义其他业务上下文");


				chain.run(req, resp, chain);
    		}
		}

2. 注册插件：在src/main/resources下新建META-INF/services文件夹下新建com.yonyou.cloud.middleware.iris.IBeforeInvoke 文件，文件内容为上述插件的全类名:

>com.yonyou.xbiz.plugin.XBizBeforeInvokePlugin

## 服务端接收方拦截插件
1.	编写服务端接收方拦截插件(其中包名为示例, 应根据项目变动):

		package com.yonyou.xbiz.plugin;

		public class XBizExecuteBeforePlugin implements IBeforeExecute {

    		@Override
    		public void run(InvokeRequest invokeRequest, InvokeResponse invokeResponse, InvokeChain invokeChain) {
        		RemoteInvocation remoteInvocation = invokeRequest.getAttribute(IrisConstans.METHOD_INVOCATION, RemoteInvocation.class);

				//这里编写对于调用方传入的属性的处理过程，依然以ctxCode及bizCode为例
				String ctxCode = remoteInvocation.getAttribute("ctxCode");
				String bizCode = remoteInvocation.getAttribute("bizCode");
        		}
				//处理过程编写完成，下方内容保持不变。

        		invokeChain.run(invokeRequest, invokeResponse, invokeChain);
    		}

    		@Override
    		public int order() {
        		return PluginConstants.PLUGIN_START_INDEX;
    		}

    		@Override
    		public String getPluginName() {
        		return DubboFilterExecuteBefore.class.getName();
    		}
		}



2.	注册插件：在src/main/resources下新建META-INF/services文件夹下新建 com.yonyou.cloud.middleware.iris.IBeforeExecute 文件, 文件内容为上述插件的全类名:

>com.yonyou.xbiz.plugin.XBizExecuteBeforePlugin