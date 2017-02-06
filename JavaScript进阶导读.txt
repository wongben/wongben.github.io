JS学习进阶指南

语法学习
1.学习JS的基本语法，大概有个认识就行，如if ,else ,for in 还有一些
1.1 基础概念，什么是DOM, 什么是CSS，HTML的常用标签<div><img><a><h1><button>
1.2 常用函数调试console.info()
1.3 JSON的常用函数，JSON.parse()和JSON.stringify()，别的在开发过程中在学习
1.4 整个过程可以使用Chrome F12进入控制台进行测试

2.学习最新ES6语法，看es6.ruanyifeng.com就够了，需要着重了解：
2.1 ...spread语法，{}析构语法，map,reduce,filter语法，导入导出Module
2.2 Promise对象,Generator函数，为看懂之后的Redux-saga做技术储备
2.3 异步操作和Async函数
2.4 建议之后空闲时间能把es6.ruanyifeng.com全文通读

3.React
3.1 看官方文档就够了，http://reactjs.cn/react/docs/getting-started-zh-CN.html,http://react-cn.com/docs/getting-started.html
3.2 重点了解什么是state，props
3.2 学习FlexBox布局
	Flex 布局教程：实例篇http://www.ruanyifeng.com/blog/2015/07/flex-examples.html?bsh_bid=683103006
	Flex 布局教程：语法篇http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html
3.3 学习React Router库

4.Redux
学习React Redux 文档http://cn.redux.js.org/docs/react-redux/quick-start.html
	重点了解Action,Reducer,Store,Middleware中间件
4.1 学习Redux Saga中间件，处理网络请求（非纯函数Action）http://leonshi.com/redux-saga-in-chinese/docs/introduction/BeginnerTutorial.html

5.整合上述的Dvajs
5.1 dva是什么，阿里出的整合react + redux + redux-saga + react-router的轻量级开发框架
5.2 查看https://github.com/dvajs/dva
5.3 https://github.com/dvajs/dva-docs/blob/master/v1/zh-cn/tutorial/01-%E6%A6%82%E8%A6%81.md
5.4 https://github.com/sorrycc/blog/issues/9
5.5 内置有dora服务器,Mock api（详细看项目目录的mock文件夹），可以直接做自己构造接口（当然接口数据格式需要），脱离后端开发，文档https://ant-tool.github.io/dora.html

6.UI组件
6.1 阿里相关组件 https://mobile.ant.design/
6.2 UI转场 https://github.com/trungdq88/react-router-page-transition

7.在实践中持续学习
7.1 在学习到5步时候，开始完成交给本人的3-4个页面

8.迁移至React Native
8.1 赛事模块，代码复用至React Native，https://github.com/sorrycc/dva-example-react-native，理论上可复用90%代码

9.迁移至微信小程序
9.1 待定

10.布局
http://www.alloyteam.com/2016/01/let-see-css-world/


建议学习计划安排：本周了解把上述1-5学习完毕，下两周进入开发阶段，完成页面和逻辑开发
