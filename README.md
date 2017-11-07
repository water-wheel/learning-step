## 目录大纲

### 第一讲
#### 前言
- 假设，接到一个新的项目：演唱会售票系统，在做系统之前，要考虑很多问题
	- 项目目录如何组织
	- 使用何种类库
	- dev和prod环境如何搞
	- 团队内部其他系统都用了什么
	- 多人合作开发，团队环境和代码规范问题怎么解决
	- 是否需要出文档，教其他团队成员配置环境，安装依赖等
	- 每一个项目皆是如此，效率何在
- 我们这次脚手架是以react为例，主要是讲解开发思路，当然用vue的团队也可模仿，或者 后面我们的开源脚手架可以提供创建vue项目的功能

#### 为什么不用create-react-app
- 不满足当前团队需求
- 不可以定制化，需要二次修改配置
- 没有开发组件的能力（后续有讲解组件开发的重要性）
- 为何不自己开发一个类似于 create-react-app 或者 vue-cli的工具呢

#### yeoman简单介绍
- yeoman 提供了一个我们开发脚手架的能力
- 按照yeoman的规范，我们开发一个npm包，yeoman就会帮我们执行这个npm的功能（其实就是拿node操作一些相关的文件）
- yeoman的安装
	- ```npm install yo -g``` 会生成一个yo的全局命令
- 最最基础的脚手架功能
	- 把全局环境node_modules包下面的模板工程 copy 到你的当前项目路径文件夹下

<img src="http://oyoee89se.bkt.clouddn.com/A4C21ECA-C440-4BF7-9E24-96C85A8FA4E6.png" width="500" />

- 下面讲一下yo这个命令 是怎么生成到全局命令里面去的

#### 启动命令原理示例
- ```mkdir flash-cli && cd ./flash-cli/```
- ```npm init``` 初始化package.json
- 添加 index.js 文件
	- ```#!/usr/bin/env node``` 
	- linux的 \*shell* 脚本，需在开头一行指定脚本的解释程序，此地规定的解释程序为node
	- env的作用：因为脚本解释器在linux中可能被安装于不同的目录，env可以在系统的PATH目录中查找。同时，env还规定一些系统环境变量
		- Mac系统下执行 ```env``` 可以看到log

	``` javascript
	#!/usr/bin/env node

	console.log("Hello Flash!")

	```
- package.json 修改
	
	```json
	{
	  "name": "flash-cli",
	  "version": "0.0.1",
	  "description": "",
	  "main": "index.js",
	  
	  "bin": {
	    "flash": "index.js"
	  },
	  
	  "author": "战楼",
	  "license": "ISC"
	}

	```
- 目录结构
	
	<img src="http://oyoee89se.bkt.clouddn.com/D8047137-8A5F-4C2F-B1FB-A9CD1EAF3D7A.png" width="500" />
	
- ```npm link```
	- 将此工程包 添加到 全局node-modules 下进行本地调试 （npm link 相当于 npm install flash-cli -g ，npm link 将当前包 以软链接的形式 放在全局node_modules环境下）
	- ```npm unlink``` 可以取消此软链接

		<img src="http://oyoee89se.bkt.clouddn.com/7DDD68E3-E8FF-4470-915F-67C1AA819D0C.png" width="500" />

	- ```/usr/local/bin/flash -> /usr/local/lib/node_modules/flash-cli/index.js```
	- ```/usr/local/lib/node_modules/flash-cli -> /Users/fengdewang/myApp/water-wheel/flash-cli```	
- 任意目录下执行 ```flash```

	<img src="http://oyoee89se.bkt.clouddn.com/EC230FF9-9749-4EF6-81FD-E62633BE6419.png" width="500" />
		
- 下面我们进一步的完善此工具		

#### 简单的shell交互和文件操作
- 实现目标功能
	- 打印欢迎语
	- 用户输入姓名、选择性别、选择爱好标签、输入创建文件的文件名
	- 根据模板生成html文件
	- 浏览器自动打开生成的文件
	- 打印结束语
- 实现效果预览

<img src="http://oyoee89se.bkt.clouddn.com/2017-10-31-16_48_31.gif" width="500" />

- dependencies介绍
	- colors: 可以在terminal打印自定义样式的字
	- ejs: 模板渲染工具
	- inquirer: 提供terminal和用户交互的能力
	- mkdirp: 生成文件夹 or 生成指定路径的文件夹(语法糖)
	- shelljs: 提供给node运行shell命令的能力
- 其他node的知识点
	- ```__dirname``` 当前模块的完整绝对路径
	- ```fs.readFileSync(filename, [encoding])``` 同步读取文件
	- ```fs.writeFileSync(filename, data, [options])``` 同步写入文件
	- ```process``` node进程对象
		- ```process.platform``` 获取当前操作系统识别符
		- ```darwin、win32、linux``` 等系统
	- ```path``` node路径解析模块
	- ```open <filePath>``` mac下使用默认工具打开文件
	- ```start <filePath>``` windows下使用默认工具打开文件
- 安装依赖
	- ```npm i colors ejs inquirer mkdirp shelljs --save```
- 基于已有目录结构新建tpl.html模板文件
- 当前目录结构

<img src="http://oyoee89se.bkt.clouddn.com/923AC3E6-4048-41C4-B66E-343D068C8137.png" width="500" />

- index.js修改如下
	- 引入node模块
	
	```javascript
	#!/usr/bin/env node
	
	require('colors');
	const fs = require('fs');
	const path = require('path');
	const inquirer = require('inquirer');
	const ejs = require('ejs');
	const mkdirp = require('mkdirp');
	const shelljs = require('shelljs');	
	```
	- 打印欢迎语
		- ```colors```模块赋予了字符串变换颜色的能力
		
		```javascript
		//欢迎语
		console.log("\n" + "Hello World, I'm flash-cli".magenta + "\n");
		console.log("It's just a test".red + "\n");
		```
	- 常量声明
		- 同一个字符串 尽量不要出现两次
		
		```javascript
		//常量
		const ENCODE = 'utf-8';
		const BUILD_PATH = './build';
		const BUILD_FILE_TYPE = '.html';
		```
	- 用户交互 inquirer
	
	```javascript
	//用户交互问题列表
	const question = [
		...
		{
			type: 'input',
			name: 'fileName',
			default: 'index',
			message: '请输入你要生成文件的名字'
		}
		...
	];

	//交互
	inquirer.prompt(question).then(answer => {
		const fileName = `${answer.fileName}${BUILD_FILE_TYPE}`;//文件名
		createFile(answer, fileName);//创建文件
		openFile(`${BUILD_PATH}/${fileName}`);//打开创建的文件
	});
	```
	- 创建文件

	```javascript
	let createFile = (data, fileName) => {
		let tpl = fs.readFileSync(__dirname + '/tpl.html', ENCODE);//读取模板文件
		mkdirp.sync(BUILD_PATH);//生成build目录文件夹
		fs.writeFileSync(`${BUILD_PATH}/${fileName}`, ejs.render(tpl, data), ENCODE); //写入index.html文件
	};
	```
	- 打开所创建的文件	

	```javascript
	let openFile = buildFilePath => {
		//mac 
		if(process.platform == 'darwin'){
			shelljs.exec(`open ${buildFilePath}`);
		} else if(process.platform == 'win32') { //windows
			shelljs.exec(`start ${buildFilePath}`);
		} else {
			console.log('This platform is ' + process.platform);
		}

		endTip(buildFilePath);
	};
	```
	- 打印结束语
	
	```javascript
	const endTip = buildFilePath => {
		console.log("\n" + "build file: " + (path.resolve(buildFilePath)).magenta + "\n");
	};
	```
	
- 教程代码：https://github.com/water-wheel/flash-cli
- 至此，我们已经从步行阶段到了骑自行车阶段了
	
#### generator-flash的简单介绍
- 下面我们先看下用yo开发出来的flash框架的效果

	<img src="http://oyoee89se.bkt.clouddn.com/2017-11-06-15_11_57_ya.gif" width="800" />

	<img src="http://oyoee89se.bkt.clouddn.com/2017-11-06-15_21_35_ya.gif" width="800" />

- flash github地址: https://github.com/water-wheel/generator-flash

#### yeoman generator的使用

- yeoman: http://yeoman.io/
- yeoman是什么？	
	- 明河说 如果前端项目是工厂的产品的话，yeoman就像工厂的流水线，标准化、傻瓜化、批量化产品生产，生产过程乏味了，但效率提高了。
	- yeoman是定义了一套用于提高前端工程师效率规范的工作流工具
- yeoman的使用
	- 创建一个项目
		- ```yo generator-generator``` 创建
		- ```mkdir generator-flash && cd generator-flash && npm init```
	- dependencies
		- ```yeoman-generator```
	- package.json

		```json
		{
		  "name": "generator-flash",
		  "version": "0.0.1",
		  "description": "",
		  "files": [
		    "generators"
		  ],
		  "keywords": ["yeoman-generator"],
		  "dependencies": {
		    "yeoman-generator": "^1.0.0"
		  }
		}
		...
		{
		  "name": "generator-flash",
		  "version": "0.0.1",
		  "description": "",
		  "files": [
		    "app",
		    "project",
		    "component"
		  ],
		  "keywords": ["yeoman-generator"],
		  "dependencies": {
		    "yeoman-generator": "^1.0.0"
		  }
		}
		
		```
		- 支持两种目录结构
		
		```md
		├───package.json
		└───generators/
		    ├───app/
		    │   └───index.js
		    └───router/
		        └───index.js
		        
		        
		        
		├───package.json
		├───app/
		│   └───index.js
		└───router/
		    └───index.js
		```
		- ```yo name```的使用 和 ```yo name:router```的使用 将直接唤起并运行 其下的index.js文件
		- 如果是第二种目录结构，则需要在package.json的files中添加多个app目录和router目录
		- yeoman规定files必须是数组形式
	- app/index.js
		
		```javascript
		const Generator = require('yeoman-generator');
		module.exports = class extends Generator {
			
			method1() {
			    this.log('method 1 just ran');
			}

			method2() {
			    this.log('method 2 just ran');
			}
			
		};
		``` 
	- method 按照顺序执行 
		- Yeoman是按照优先级顺序依次执行所定义的方法。当你定义的函数名字是Yeoman定义的优先级函数名时，会自动将该函数列入到所在优先级队列中，否则就会列入到 default 优先层级队列中。
		- ```initializing、prompting、configuring 、default 、writing 、conflicts 、install 、end ```
	- Prefix method name by an underscore  (e.g. \_private_method).
	- 异步流程控制

		```javascript
		asyncTask() {
		  var done = this.async();

		  getUserEmail(function (err, name) {
		    done(err);
		  });
		}
		``` 
	- The prompt module is provided by Inquirer.js
		
		```javascript
			prompting() {
			    return this.prompt([{
			      type    : 'input',
			      name    : 'name',
			      message : 'Your project name',
			      default : this.appname // Default to current folder name
			    }, {
			      type    : 'confirm',
			      name    : 'cool',
			      message : 'Would you like to enable the Cool feature?'
			    }]).then((answers) => {
			      this.log('app name', answers.name);
			      this.log('cool feature', answers.cool);
			    });
			  }
		```
	- generator.composeWith()
		- 执行另一个generator
			
		```javascript
			this.composeWith('name:route');
		```
	- 自动安装依赖
		- this.npmInstall
		- this.yarnInstall
		- this.bowerInstall
		- 合并安装this.installDependencies({npm: false, bower: true, yarn: true})
	- Template context
		- this.sourceRoot() // returns './templates'
		- this.templatePath('index.js');// returns './templates/index.js'
	- copy template
		- this.fs.copyTpl(src, output, data)
		- using ejs template syntax


### 第二讲
#### 整体架构分析
- 团队目的
	- 解决复杂的且重复的问题，给团队带来效率提升
	- 从团队leader的角度看：我们每个团队成员，应该尽快的去完成业务需求，而不是天天配置开发环境，如果有现成的代码可用，就不要重复造轮子，拿来即用就好。
	- 我们的目的就是要那一堆零件过来，然后拼装成一辆汽车，而不需要了解零件的制作工艺
- 剥离组件
	- 组件剥离项目是非常重要的一个事情，它使得项目依赖清晰，开发更快捷
	- 代码解耦、可复用性强、维护方便单一
	- 组件最好维护到公司的npm私服上，没有npm私服的建议组件名 加个前缀，发到npm上
- 减少开发者的学习成本
	- 项目的配置项越少 对开发人员的上手成本就越低
	- 把webpack的配置内敛至npm包中，仅留下几个路径配置项
- 架构对比
	- 传统架构设计
	<img src="http://oyoee89se.bkt.clouddn.com/%E4%BC%A0%E7%BB%9F%E6%9E%B6%E6%9E%84%E8%AE%BE%E8%AE%A1.png" width="600" />
	- 新架构设计
	<img src="http://oyoee89se.bkt.clouddn.com/%E6%96%B0%E6%9E%B6%E6%9E%84%E8%AE%BE%E8%AE%A1.png" width="600" />

#### 模板工程的分类
- 根据前端框架分类
	- React
	- Vue
	- JQuery
- 根据功能拆分
	- project
	- component
	- common
- 根据使用的主要功能插件分
	- react + router
	- react + router + redux
	- react + mobx + typescript

- 究其结果，都是要进行模板文件的copy，只不过 根据用户的选择不同，copy的内容不同罢了
	<img src="http://oyoee89se.bkt.clouddn.com/A4C21ECA-C440-4BF7-9E24-96C85A8FA4E6.png" width="500" />

- 下面就从平时遇到的比较多的问题上开始看，一系列的影响效率的问题，然后去解决他们



#### 扩展一般webpack配置的兼容性
- webpack 具体配置不细讲了哈，推荐给大家看一个webpack的视频课程：https://m.qlchat.com/live/channel/channelPage/2000000172777118.htm
- 问题：webpack在多页应用开发时候，会出现 entry 多入口路径的配置问题，相信很多前端同学，没有玩过webpack时候，或者对webpack一知半解时候，配置此多页面入口路径是相当麻烦的一件事儿

	<img src="http://oyoee89se.bkt.clouddn.com/8510E9A8-467B-4FBB-B519-1C81A3B4523D.png" width="400" />

- 我们自己的脚手架完全可以解决这个问题
- 解决：
	- 原理：按照定义的规范去写目录结构，在运行代码之前，先用node解析一遍，当前的路径，自动计算出当前的 entry map，完美解决
	- 代码：https://github.com/water-wheel/flash-scripts/blob/master/config/paths.js#L26

		<img src="http://oyoee89se.bkt.clouddn.com/3CCF5438-5270-4FA3-AD10-8947724E5C0C.png" width="500" />
		
#### 为模板工程添加语法糖
- 问题：在团队中，有多个业务线，每个业务线也都有多个项目，每个项目都会用到的功能，都需要开发人员开发一遍，或者从其他项目中copy一份出来，导致了业务代码乱七八糟，而且代码质量参差不一
- 解决：我们通过脚手架，为每个项目都内置一些工具方法，
	- 例如一些工具函数：获取url的参数、判断当前容器类型、监听页面回退、获取cookie值等
	
		<img src="http://oyoee89se.bkt.clouddn.com/FEBD6976-6BAA-4FC8-B494-0EEE1B3769DA.png" width="500" />

	- reset.css
	- scss mixin：兼容性flex方案、1像素线边框、单行（多行）文本截断、渐变、rem计算函数等
		
		<img src="http://oyoee89se.bkt.clouddn.com/BC456FB8-FCCA-4424-AB73-09AA6C293612.png" width="500" />
		
- 当然如果你们团队是hybrid开发时候，里面还可以内置一些 jsBridge 进去
- flash脚手架的语法糖代码：https://github.com/water-wheel/generator-flash/tree/master/common

#### 数据mock功能的开发
- 问题：前后端分离的项目，后端接口开发缓慢，前端业务依赖后端接口数据，如若是写一个常量来mock数据，则上线前 需要更换接口的url，需要前端开发重新进行自测，才能交付
- 解决：
	- 启一个本地服务来作为接口服务器
	- 建一个mock文件夹 来放置 mock的接口数据，和mock的config配置(type: get | post | delay 等)
	- 监听mock文件夹，如果有modify，则重启mock服务来更新接口数据
- 具体解决方案：
	- 在npm start时候，我们fork一个进程，把mock server跑起来，然后监听mock文件夹，当mock文件夹发生修改时候，则重启这个进程
		- 代码地址：https://github.com/water-wheel/flash-scripts/blob/master/scripts/mock.js
		
		<img src="http://oyoee89se.bkt.clouddn.com/DE09069D-40E3-4444-A047-9C841F0B9BD6.png" width="500" />
	- mock接口服务 (读取本地mock文件的配置内容 和 配置列表，缓存起来，然后启动服务)
		- 代码展示：https://github.com/water-wheel/flash-scripts/blob/master/mock/worker.js
		
		<img src="http://oyoee89se.bkt.clouddn.com/CA8B871C-A24C-43E5-B0E6-0F93117DF386.png" width="500" />
	

#### 组件中关于readme的统一化
- 问题：即使有了脚手架 帮我们把组件的工程创建好，但我们还是经常遇到其他人开发的组件，我们无法快速使用的问题
	- 例如我们知道有loading这个组件，但是props的入参是什么，我们只能通过代码找到，有好一点儿的，作者在readme里面 写了loading的使用方式，但是可能下个dialog的组件 就在readme里面 以另一种风格去写dialog的组件使用了，这是一件很蛋疼的事儿
- 解决方案：
	- 让readme风格 和 规范统一化
	- 让使用者 在第一时间能找到 最简单的使用示例 和 配置参数，只需要copy过来就能用，不需要关系源码是怎么开发的，如果遇到问题，小窗下作者，立马能得到解决方法
	- 我们readme的模板
	
	<img src="http://oyoee89se.bkt.clouddn.com/D4306A1C-0AF4-4FB2-BBE9-C4749C8AEC57.png" width="500" />