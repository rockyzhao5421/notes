# Node.js
Node.js 是一个开源、跨平台的 JS 运行环境。

[官网](https://nodejs.org/en/)

[Guides](https://nodejs.org/en/docs/guides/)

# NVM
全称是 **Node Version Manger**，就是 Node.js 的版本管理工具。通过它可以很方便的安装多个 Node.js 版本，并在版本间快速切换。

NVM 本身的安装可以通过 pacman 安装，也可以通过 GitHub 提供的脚本安装。
用 pacman 安装后要仔细看它的说明：

    You need to source nvm before you can use it. Do one of the following
    or similar depending on your shell (and then restart your shell):

    echo 'source /usr/share/nvm/init-nvm.sh' >> ~/.bashrc
    echo 'source /usr/share/nvm/init-nvm.sh' >> ~/.zshrc

    You can now install node.js versions (e.g. nvm install 10) and
    activate them (e.g. nvm use 10).

    init-nvm.sh is a convenience script which does the following:

    [ -z "$NVM_DIR" ] && export NVM_DIR="$HOME/.nvm"
    source /usr/share/nvm/nvm.sh
    source /usr/share/nvm/bash_completion
    source /usr/share/nvm/install-nvm-exec

    You may wish to customize and put these lines directly in your
    .bashrc (or similar) if, for example, you would like an NVM_DIR
    other than ~/.nvm or you don't want bash completion.

    See the nvm readme for more information: https://github.com/creationix/nvm

我用的是 bash，所以执行下面语句就行了：

    echo 'source /usr/share/nvm/init-nvm.sh' >> ~/.bashrc

### 常用命令
下载、编译、安装指定版本：

    $ nvm install 12

下载、编译、安装最新版本：

    nvm install node // "node" is an alias for the latest version

下载、编译、安装最新的 LTS 版本

    nvm install --lts

列出所有在服务器可下载安装的的版本：

    nvm ls-remote

看看已安装了哪些版本以及正使用的版本：

    nvm list
    -> v11.12.0
       v12.18.3
       v18.10.0
切换版本：

    $ nvm use 16

看看某一个可执行版本安装在哪儿了：

    nvm which 12.22
要把源换成淘宝源，只用在自己的 home 目录的 .bashrc 文件的末尾加上：

    NVM_NODEJS_ORG_MIRROR=http://npm.taobao.org/mirrors/node  

[GitHub](https://github.com/nvm-sh/nvm)

# NPM
全称是 **Node Packaged Modules**，也就是 Node.js 第三方库管理器，用于管理 Node 的大量扩展API。在安装 Node.js 时就会自动安装相应版本的 NPM。

### 查看当前源

    npm get registry
    https://registry.npmjs.org/ # 这是官方源

### 设置为淘宝源

    npm config set registry http://registry.npm.taobao.org/


### 安装第三方库
任何第三方库都可以用一下命令安装：

    $ npm install packageName
这个命令会将包安装在当前目录下 node_modules 目录内，可执行命令（如果有）安装在 node_modules/.bin 目录下.

作为系统级的全局安装使用 -g 选项:

    # npm -g install packageName
默认情形下这个命令会将包安装至 ```/usr/lib/node_modules/npm```，需要管理员权限.

### 更新第三方库

更新包只需要执行

    $ npm update packageName

对于全局环境安装的包 ( -g )

    # npm update -g packageName
全局安装的包需要管理员权限，除非使用 prefix 设置到用户可写目录。

有时您只希望更新所有包，去掉包名将试图更新所有包。

    $ npm update
或者添加 -g 标记更新全局环境安装的包

    # npm update -g

### 删除第三方库

删除使用 -g 标记安装的包，只需：

    # npm -g uninstall packageName

若删除个人用户目录下的包去掉标记执行：

    $ npm uninstall packageName

### 列出所有的第三方库

若要显示已安装的包的树形视图执行：

    $ npm -g list

仅显示顶层树：

    $ npm list --depth=0

要显示需要更新的过期软件包：

    $ npm outdated

**NPM 的版本要和 Node.js 的版本一一对应才能使用。而装 Node.js 的时候自带安装对应版本的 NPM，所以一般我们不单独安装 NPM。**

### npm init

npm init 用来初始化生成一个新的package.json文件。它会向用户提问一系列问题，如果觉得不用修改默认配置，一路回车就可以了。

当你的 package.json 文件中配置了相关的依赖，```npm install``` 会将配置中的依赖下载到当前目录的node_modules文件夹中（如果没有这个文件夹则自动生成一个）。

    "dependencies": { 					//生产环境
        "vue": "2.6.10",
        "vue-router": "3.0.6",
        "vuex": "3.1.0"
    },
    "devDependencies": {				//开发环境
        "sass": "1.26.8"
    }
比如你的package.json中配置了上述内容，那么执行npm install则会下载vue,vue-router,vuex,sass模块下载到node_modules文件夹中

### 安装指定包依赖（以vue为例）

+ npm install vue --global //全局安装vue,下载vue到电脑中，而不是当前项目文件中
  
  npm i vue -g //缩写 

  因为vue插件包的版本总会更新迭代，不建议当前的vue版本直接下载到本机
+ npm install vue --save //安装运行时的依赖
  
  npm install vue -S //缩写

  安装vue到当前项目的node_modules文件夹中，并在package.json中的dependencies添加"vue":"版本号"
+ npm install vue --save-dev //安装开发环境的依赖

  npm install vue -D //缩写
  
  安装vue到当前项目的node_modules文件夹中，并在package.json中的devDependencies添加"vue":"版本号"

### 5种npm依赖

1. dependencies => 放置项目中代码运行时需要用到的依赖
2. devDependencies => 放置本地开发过程中需要使用到的编译、打包、测试、格式化模块等
3. peerDependencies => 放置本模块需要宿主环境提供的模块依赖（通常本模块是为了给引用方提供服务时设置依赖）
4. bundledDependencies => 和上面的配置不同，为数组格式，其中包含需要被打包进本地 package 里的依赖模块名，通过 npm pack 命令生成一个模块包
-B //缩写
5. optionalDependencies => 放置一些项目中可忽略其各种错误的包模块，和 dependencies 一样，但该模块可有可无
-O //缩写


[NPM 官方文档](https://docs.npmjs.com/)

# NPX

npm 从5.2版开始，增加了 npx 命令，我们就了解一下它的作用和原理。

### 调用项目已安装模块

就是在命令行执行本地已安装的依赖包命令。

比如，项目内部安装了测试工具 Mocha:

    npm install -D mocha

就可以直接在命令行这样调用：

    npx mocha --version

也可以写到package.json的scripts中，缺点是很麻烦：

    //package.json
    "scripts": {
        "findmocha": "mocha --version",
    }

### 避免全局安装

有很多命令，我们只需要执行一次的，但是却要全局安装。使用 npx，可以在不全局安装依赖包的情况下，运行命令，而且运行后不会污染全局环境。

比如，create-react-app这个模块是全局安装，npx 可以运行它，而且不进行全局安装：

    npx create-react-app my-react-app

上面代码运行时，npx 将create-react-app下载到一个临时目录，使用以后再删除。所以，以后再次执行上面的命令，会重新下载create-react-app。

### 原理

npx的原理很简单，就是在运行它时，执行下列流程：

1. 去 node_modules/.bin 路径检查 npx 后面跟的命令是否存在，找到之后执行；
2. 找不到，就去环境变量$PATH里面，检查npx后的命令是否存在，找到之后执行;
3. 还是找不到，自动下载一个临时的依赖包最新版本在一个临时目录，然后再运行命令，运行完之后删除，不污染全局环境。




[阮一峰 NPX 教程](https://www.ruanyifeng.com/blog/2019/02/npx.html)

# yarn

Yarn 是 Facebook, Google, Exponent 和 Tilde 开发的一款新的 JavaScript 包管理工具，是 NPM 的替代品，大部分特性不需要关注，反正更牛逼就是了。重要的是它默认解决了团队工作中第三包版本不一致的问题。

### 查看当前源

    yarn config get registry
    https://registry.yarnpkg.com # 这是官方源头

### 改成淘宝源

yarn config set registry http://registry.npm.taobao.org/



### 安装

官网安装脚本:

    curl -o- -L https://yarnpkg.com/install.sh | bash -s -- --nightly

用 NPM 安装：

    npm install -g yarn

### package.json 中的版本号
NPM 为什么会出现包版本不一致的问题呢？ 还要从 package.json 中包的版本号说起：

当我们查看package.json中已安装的库的时候，会发现他们的版本号之前都会加一个符号，有的是插入符号（^），有的是波浪符号（~）。那么他们到底有什么区别呢？先贴一个例子，对照例子来做解释：

    “pageckage1” : "5.0.3",//表示安装指定的5.0.3版本
    "pageckage2" : "~5.0.3",//表示安装5.0.X中最新的版本
    "pageckage3" : "^5.0.3"//表示安装5.X.X中最新的版本

+ 波浪符号（~）：他会更新到当前minor version（也就是中间的那位数字）中最新的版本。放到我们的例子中就是："pageckage2" : "~5.0.3"，这个库会去匹配更新到5.0.x的最新版本，如果出了一个新的版本为5.1.0，则不会自动升级。波浪符号是曾经npm安装时候的默认符号，现在已经变为了插入符号。
+ 插入符号（^）：这个符号就显得非常的灵活了，他将会把当前库的版本更新到当前major version（也就是第一位数字）中最新的版本。放到我们的例子中就是："pageckage3" : "^5.0.3"，这个库会去匹配5.x.x中最新的版本，但是他不会自动更新到6.0.0。

最后解释下之前提到的minor verision和major version：

1.15.2对应就是MAJOR,MINOR.PATCH：1是 marjor version；15是minor version；2是patch version。

+ MAJOR：这个版本号变化了表示有了一个不可以和上个版本兼容的大更改。
+ MINOR：这个版本号变化了表示有了增加了新的功能，并且可以向后兼容。
+ PATCH：这个版本号变化了表示修复了bug，并且可以向后兼容。

因为major version变化表示可能会影响之前版本的兼容性，所以无论是波浪符号还是插入符号都不会自动去修改major version，因为这可能导致程序crush，可能需要手动修改代码。

好了，现在想一下这种情况：你在开发某一功能时导入了一个第三库的版本是 “~5.0.3”，你实际安装的版本是 "5.0.4"，开发结束你上传了代码，同时也上传了 package.json 文件。过了几天这个库的作者发布了新版本 “5.0.5”， 你的同事同步了你的代码，然后运行 ```npm install xxx -S``` 就安装了 “5.0.5”，你们两个版本就不一致了。

### Yarn 解决了这个问题

为了防止拉取到不同的版本，Yarn 有一个锁定文件 (lock file) 记录了被确切安装上的模块的版本号。

每次只要新增了一个模块，Yarn 就会创建（或更新）yarn.lock 这个文件。这么做就保证了，每一次拉取同一个项目依赖时，会优先读取这个 Lock 文件，使用一样的模块版本。

npm 其实也有办法实现处处使用相同版本的 packages，但需要开发者执行 npm shrinkwrap 命令。这个命令将会生成一个锁定文件，在执行 npm install 的时候，该锁定文件会先被读取，和 Yarn 读取 yarn.lock 文件一个道理。

npm 和 Yarn 两者的不同之处在于，Yarn 默认会生成这样的锁定文件，而 npm 要通过 shrinkwrap 命令生成 npm-shrinkwrap.json 文件，只有当这个文件存在的时候，packages 版本信息才会被记录和更新。

### Yarn 的离线缓存

每次从远程位置下载一个软件包后，都会在缓存一份副本。下次需要安装相同的软件包时，Yarn 将使用缓存中存储的版本，从而无需从其原始位置下载了。

本地缓存的位置是相对于项目根目录的，可以通过 cacheFolder 进行配置。默认情况下，其位于 .yarn/cache。

当你删除或升级软件包时，Yarn 将自动将不需要的软件包从缓存中清除。如果需要手动清理缓存，可以使用 yarn cache clean 命令。

对于全局镜像，必须使用 yarn cache clean --mirror 命令手动清除。

### 常用命令




| NPM      | Yarn |  说明     |
| :---        |    :-----   |          :--- |
| npm init   | yarn init        |     |
| npm         | yarn        |安装项目的全部依赖            |
| npm install   | yarn        |        安装项目的全部依赖|
| npm install react --save   | yarn add react       |     |
| npm install react@xx.xx --save  | yarn add react@xx.xx        |安装指定版本的包  |
| npm install react --save-dev  | yarn add react --dev        |       |
| npm uninstall react --save	   | yarn remove react       |      |
| npm update --save   | yarn upgrade       |       |


[官网](https://yarnpkg.com/)

[中文官网](https://www.yarnpkg.cn/)




