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


