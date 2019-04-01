Vue-CLI 3.x 自动部署项目至服务器
======================

前言
--

平时部署前端项目流程是：先部署到测试环境ok后再发布到生产环境上，部署到测试环境用 xshell 连上服务器，然后用 xftp 连接服务器，然后本地 build 项目，接着把 build 好的文件通过 xftp 上传到服务器上，整个流程感觉稍有繁琐，重复。

本教程讲解的是 Vue-CLI 3.x 脚手架搭建的vue项目, 利用[scp2](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fspmjs%2Fnode-scp2)自动化部署到静态文件服务器 Nginx

一 安装scp2
--------

[scp2](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fspmjs%2Fnode-scp2)是一个基于ssh2增强实现，纯粹使用JavaScript编写。 而ssh2就是一个使用nodejs对于SSH2的模拟实现。scp，是secure copy的缩写, scp是Linux系统下基于SSH登陆进行安全的远程文件拷贝命令。这里我们就用这个功能，在Vue编译构建成功之后，将项目推送至测试/生产环境，以方便测试，提高效率。 安装scp2：

```js
    npm install scp2 --save-dev
```
 

二、配置测试/生产环境 服务器SSH远程登陆账号信息
--------------------------

#### 1\. 在项目根目录下, 创建 .env.dev 文件 (测试环境变量)

VUE\_APP\_SERVER_ID变量表示 当前需部署的测试服务器ID为0

```js
    // .env.dev文件中
    VUE_APP_SERVER_ID=0
```

#### 2\. 在项目根目录下, 创建 .env.prod 文件 (生产环境变量)

VUE\_APP\_SERVER_ID变量表示 当前需部署的生产服务器ID为1

```js
    // .env.prod文件中
    VUE_APP_SERVER_ID=1
```

#### 3\. 在项目根目录下, 创建 deploy/products.js 文件

```js
    /*
     *读取env环境变量
     */
    const fs = require('fs');
    const path = require('path');
    // env 文件 判断打包环境指定对应的服务器id
    const envfile = process.env.NODE_ENV === 'prod' ? '../.env.prod' : '../.env.dev';
    // env环境变量的路径
    const envPath = path.resolve(__dirname, envfile);
    // env对象
    const envObj = parse(fs.readFileSync(envPath, 'utf8'));
    const SERVER_ID = parseInt(envObj['VUE_APP_SERVER_ID']);
    
    function parse(src) {
      // 解析KEY=VAL的文件
      const res = {};
      src.split('\n').forEach(line => {
        // matching "KEY' and 'VAL' in 'KEY=VAL'
        // eslint-disable-next-line no-useless-escape
        const keyValueArr = line.match(/^\s*([\w\.\-]+)\s*=\s*(.*)?\s*$/);
        // matched?
        if (keyValueArr != null) {
          const key = keyValueArr[1];
          let value = keyValueArr[2] || '';
    
          // expand newlines in quoted values
          const len = value ? value.length : 0;
          if (len > 0 && value.charAt(0) === '"' && value.charAt(len - 1) === '"') {
            value = value.replace(/\\n/gm, '\n');
          }
    
          // remove any surrounding quotes and extra spaces
          value = value.replace(/(^['"]|['"]$)/g, '').trim();
    
          res[key] = value;
        }
      });
      return res;
    }
    
    /*
     *定义多个服务器账号 及 根据 SERVER_ID 导出当前环境服务器账号
     */
    const SERVER_LIST = [
      {
        id: 0,
        name: 'A-生产环境',
        domain: 'www.prod.com',// 域名
        host: '46.106.38.24',// ip
        port: 22,// 端口
        username: 'root', // 登录服务器的账号
        password: 'root',// 登录服务器的账号
        path: '/mdm/nginx/dist'// 发布至静态服务器的项目路径
      },
      {
        id: 1,
        name: 'B-测试环境',
        domain: 'test.xxx.com',
        host: 'XX.XX.XX.XX',
        port: 22,
        username: 'root',
        password: 'xxxxxxx',
        path: '/usr/local/www/xxx_program_test/'
      }
    ];
    
    module.exports = SERVER_LIST[SERVER_ID];
```

三、使用scp2库,创建自动化部署脚本
-------------------

在项目根目录下, 创建 deploy/index.js 文件

```js
    const scpClient = require('scp2');
    const ora = require('ora');
    const chalk = require('chalk');
    const server = require('./products');
    const spinner = ora('正在发布到' + (process.env.NODE_ENV === 'prod' ? '生产' : '测试') + '服务器...');
    spinner.start();
    scpClient.scp(
      'dist/',
      {
        host: server.host,
        port: server.port,
        username: server.username,
        password: server.password,
        path: server.path
      },
      function (err) {
        spinner.stop();
        if (err) {
          console.log(chalk.red('发布失败.\n'));
          throw err;
        } else {
          console.log(chalk.green('Success! 成功发布到' + (process.env.NODE_ENV === 'prod' ? '生产' : '测试') + '服务器! \n'));
        }
      }
    );
```

四、添加 package.json 中的 scripts 命令, 自定义名称为 "deploy",
-------------------------------------------------

发布到测试环境命令为 `npm run deploy:dev`,生产环境为 `npm run deploy:prod`

```js
      "scripts": {
        "serve": "vue-cli-service serve --mode dev",
        "build": "vue-cli-service build --mode prod",
        "deploy:dev": "npm run build && cross-env NODE_ENV=dev node ./deploy",
        "deploy:prod": "npm run build && cross-env NODE_ENV=prod node ./deploy",
      },

```

ps 这里用到了cross_env 得安装 `npm i --save-dev cross-env` cross-env能跨平台地设置及使用环境变量，这里用来设置是生产环境还是测试环境。

图解
--
<img alt="vue config" class="lazyload" src="https://user-gold-cdn.xitu.io/2019/4/1/169d81456e6dc85c?imageView2/0/w/1280/h/960/ignore-error/1" data-width="891" data-height="962">