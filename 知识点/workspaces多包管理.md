workspaces 项目多包管理和使用

一、简介

Workspaces 是一个用来在本地的root package 包下面管理多个包的npm 术语和功能。（其实yarn 很早就支持了，npm 7.x， Node@15.0.0 中开始支持）
这个功能让我们在本地开发包，尤其是多个互相依赖的包时更加得心应手。它可以避免我们再手动的去执行npm link 命令，而是在npm install 的时候，会自动把workspaces 下面的合法包，自动创建符号链接到根目录的node_modules 里。
能够被单独作为一个包创建符号链接的文件夹，我们就称为一个workspace，所以是可以有多个workspace 的，可以在package.json 的 workspaces 字段中进行配置。

二、npm workspace的作用
![alt text](<image/Pasted Graphic 6.png>)
￼
1)依赖共享。子工作区可以使用主工作区的所有依赖，无需重复安装
2)导出子工作区，供所有工作区使用。可以将子工作区导出到node_modules中，供所有工作区使用
```
# /package.json 
"devDependencies": { 
"@fai/divide": "workspace:*", 
"@vitejs/plugin-vue": "^2.0.0", 
"rimraf": "3.0.2", 
"virtual-module": "^0.4.0", "vite": "2.7.13" }
```
"@fai/divide": "workspace:*" 将divide整个工作区的文件放到了node_modules中
1.如何使用divide工作区文件
```
<script setup>
`import HelloWorld from '@fai/divide'
</script>
```
注意：divde里的文件修改的同时node_modules里的文件也会修改，两者的关系是实时同步的，那么这样的话，我们在任意一个工作空间修改东西，别的工作空间都能实时同步了。
2. workspace常用的命令
1）给模块安装依赖
```
npm install 依赖名字 --workspace 包的位置
简写：
npm i 依赖名字 -w 包的位置

例如：
npm install number-precision --workspace packages/divide
简写为
npm i number-precision -w packages/divide
```

2）给所有模块安装同一个依赖
```
# 注意 workspaces 这里多个 `s`
npm install 依赖名字 --workspaces
# 也可简写为
npm i 依赖名字 -ws

npm install dayjs --workspaces
# 也可简写为
npm i dayjs -ws
```

3）移除依赖
```
npm uninstall 依赖名字 -w 包位置

例如：
npm uninstall number-precision -w packages/divide
```

4）执行模块里面的 scripts
```
npm run dev -w 包位置

例如：
npm run dev -w packages/divide
```

三、配置workspaces 

workspaces 字段接收一个数组，数组里面可以填写相对根目录的文件夹名称或者是glob 通配符。例如：
```
{
  "name": "my-workspaces-powered-project",
  "workspaces": [
    "workspace-a"
  ]
}
```
上面的配置表明，在根目录下，有一个workspace-a文件夹，它作为一个npm 包，包含一个package.json。

```
.
+-- package.json
`-- workspace-a
   `-- package.json
```
预期的效果是，在根目录下执行npm install 命令，文件夹workspace-a 会被符号链接到根目录的node_modules 文件夹下。对于包的使用和查找，和正常安装这个包并无差别。
这个例子如果执行npm install后，得到的目录结构如下：
```
. 
+-- node_modules 
| 	`-- workspace-a -> ../workspace-a 
+-- package-lock.json 
+-- package.json 
`-- workspace-a 
	`-- package.json
```
四、使用 workspaces

根据 nodejs 规范定义的包查找规则 ，任何合法定义了package.json 文件的workplace 包都可以被正常使用，通过package.json里定义的name 字段来引用包.
在上面的例子中，我们可以这样来使用workspace-a 包：
```
// ./workspace-a/index.js 
module.exports = 'a' 

// ./lib/index.js 
const moduleA = require('workspace-a')
console.log(moduleA) // -> a

```

执行：
```
node lib/index.js

```



