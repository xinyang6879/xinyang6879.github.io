---
title: npm、npx与pnpm
categories: 前端
tag: 前端
---

# npm、npx 与 pnpm

## npm

### `npm i`与`npm update`

- `npm i`：先检查 node_modules 模块是否有对应的指定模块，如果存在，就不再进行安装，即使远程有新版本也不会重新获取

  npm i在安装包的依赖包时，包的依赖包会安装符合规则的<font color=red>最高</font>版本

  > npm 都要强制重新安装，可以使用-f 或--force 参数。

- `npm update`：每次安装都会先请求远程仓库的最新版本，然后查询本地版本。
  > 如果本地版本不存在或者远程版本较新，那么更新版本

#### 安装非发布的包

[官网](https://docs.npmjs.com/about-packages-and-modules/)表明了如下几个情况可以直接使用

- a:包含一个由 package.json 文件描述的程序的文件夹。
- b:包含（a）的 gzipped tarball 。
- c:解析为（b）的 URL。
- d:\<name\>@\<version\>: 在 registry 上发布的(c)
- e:\<name\>@\<tag\> : 能指向（d）。
- f:\<name\> 具有 latest 标签，且满足（e）。
- g:git url，当 clone 时，得到（a）

```
git://github.com/user/project.git#commit-ish
git+ssh://user@hostname:project.git#commit-ish
git+http://user@hostname/project/blah.git#commit-ish
git+https://user@hostname/project/blah.git#commit-ish
```

commit-ish 可以是任何 tag、分支或者 sha，可以用`git checkout`切换的，默认是`master`

<h1></h1>

### npm 缓存

由于安装时，即使本地有缓存，但是也不会进行读取，那么这就导致在弱网或者无网情况下，无法安装依赖包或者安装速度极低

- `--cache-min`：参数指定了一个时间(以分钟为单位)，只有超过这个时间的模块，才会从远程进行下载安装

  ```js
  npm install --cache-min 9999999 <package-name>
  ```

<h1></h1>

### `npm i`流程

- 发出`npm i`命令

- 执行工程自身 preinstall

  > 如果当前工程定义了 preintsall 钩子，那么会调用这个钩子函数

- 确定首层依赖

  > 确定当前项目中`package.json`中的`dependencies`和`devDependencies`指定的模块

  > 当前项目中的每个依赖包是每个依赖树的根节点，所以 npm 开启多线程对每个依赖包进行更深层级的节点

- 获取模块

  > 确定下载的模块版本

  > 如果版本描述文件（`npm-shrinkwrap.json` 或 `package-lock. json`）有对应模块的信息，那么直接依据对应的信息获取

  > 如果没有，向[registry](https://registry.npmjs.org/)查询模块压缩 包的网址，然后依据`package.json`文件中的版本去仓库中获取。

  > 获取到模块的 resloved 字段，即压缩包的地址后。获取到后，npm 依据此地址检查本地缓存，如果缓存中有，那么直接从缓存中(只会检查`node_modules`目录，而不会检查`~/.npm`目录)拿；如果没有，那么从仓库中进行下载

  > 查找该模块的依赖，如果有依赖则回到第一步，没有则停止

- 扁平化模块

  > 获取到完整的依赖树后，里面可能包含大量的重复模块，npm3 之前会严格按照依赖树的结构进行安装，这样会造成大量的模块冗余；

  > npm3 之后默认加入了一个`dedupe`的过程，遍历所有节点，将模块放在根节点下面（node_modules 下），当发现有了重复模块时，将其丢弃
  >
  > > 重复模块：**模块名**相同且**semver**兼容。每个 semver 基本都对应一段*版本允许范围*，如果两个模块的版本允许范围有**交集**，那么可以得到一个**兼容**版本。这样就不需要版本号完全一致了，减少更多的冗余模块在这个阶段中直接去掉
  > >
  > > > 比如 A 模块依赖 package@^1.0.0，B 模块依赖 package@^1.1.0，则 1.1.0 为兼容版本<br /> node_modules--A<br /> node_modules--B<br /> node_modules--package@^1.1.0<br /><br /> 比如 A 模块依赖 package@^1.0.0，B 模块依赖 package@^2.1.0，则没有兼容版本，会将一个版本放在 node_modules，一个继续保留在依赖树中<br /> node_modules--A--package@^1.0.0<br /> node_modules--B--package@^1.1.0<br />

- 安装模块

  > 下载压缩包，存放在~/.npm 目录

  > 解压到当前项目的 node_modules 目录

  > 执行模块中的生命周期函数（按照 preinstall、install、postinstall 的顺序）。

- 执行当前项目自身的生命周期

  > 当前 npm 工程如果定义了钩子此时会被执行（按照 install、postinstall、prepublish、prepare 的顺序）。

- 生成或者更新版本描述文件（`npm-shrinkwrap.json` 或 `package-lock. json`）

## npx

`npx`是`npm@5.25.2`增加的命令，如果 npm 版本低于这个版本，那么用`npm i -g npx`安装即可

### 与 npm 的不同

- npx 是下载到一个临时目录中，然后使用完成之后，进行删除。没有 npm 一样的缓存

- npx 还可以运行可执行文件(远程的也可以)，比如只安装 webpack 了，那么用`npm run webpack`会报错，用`npx run webpack`就可以成功运行

- npx 会检查 node_modules 和系统变量`$PATH`的命令是否存在

- `--no-install`: 如果本地不存在该模块，那么会报错。可以用于强制使用本地模块

- `--ignore-existing`: 不管是否本地存在，都强制安装使用远程模块。可以用于获取最新的包，即用即删

## [pnpm](https://pnpm.io/zh/motivation)

### [硬连接与符号连接](https://juejin.cn/post/7032116303737389086/)

- 硬连接：使用 inode 指向源文件，即使源文件目录地址变化了，但是依旧能进行访问。因为其 inode 仍指向该文件。没有对原始文件的引用。

- 符号连接：指向的源文件地址，如果源文件目的地址修改了，那么就无法再访问；如果有一个新的文件名字与源文件一致，那么再次访问时，访问的是新文件

<h1></h1>

### pnpm

#### 缓存

pnpm 和 npm 一样，有一个缓存目录

- Mac/linux 中默认会设置到{home dir}>/.pnpm-store/v3；windows 下会设置到当前盘的根目录下，比如 C（C/.pnpm-store/v3）、D 盘（D/.pnpm-store/v3）。

- pnpm 可以在一个电脑上不同的磁盘设置[同一个分区](https://pnpm.io/zh/workspaces)，在这种情况下，pnpm 将复制包而不是硬链接它们，因为硬链接只能发生在同一文件系统同一分区上

- npm 在安装时，不会去检查缓存；而 pnpm 在安装时，会先检查是否有对应包及其版本的缓存，如果有的话，直接硬链接到这个缓存地址

<h1></h1>

#### 模块依赖

- pnpm 使用平铺的方式，类似于`npm2`的结构，但是增加了一个`.pnpm`目录，其中的包命名格式如下：

  ```js
   .pnpm/<organization-name>+<package-name>@<version>/node_modules/<name>
  // 组织名(若无会省略)+包名@版本号/node_modules/名称(项目名称)
  ```

- 对于 [PeerDependencies](https://pnpm.io/zh/how-peers-are-resolved) 来说，命名规则稍许不同

  ```js
   .pnpm/<organization-name>+<package-name>@<version>_<organization-name>+<package-name>@<version>/node_modules/<name>
   // peerDep组织名(若无会省略)+包名@版本号_组织名(若无会省略)+包名@版本号/node_modules/名称(项目名称)
  ```

- pnpm 使用硬链接，将 node_modules 的包地址硬链接到 pnpm 的缓存中；对于同包同版本使用符号连接.

  ![image](https://pnpm.io/zh/assets/images/node-modules-structure-8ab301ddaed3b7530858b233f5b3be57.jpg)

<h1></h1>

#### [npm 与 pnpm 差别](https://pnpm.io/zh/feature-comparison)

| 功能                             | pnpm                            | npm |
| -------------------------------- | ------------------------------- | --- |
| 隔离的 node_modules              | ✅                              | ❌  |
| 自动安装 peers                   | ✅ 通过 auto-install-peers=true | ✅  |
| Plug'n'Play(即插即用)            | ✅                              | ❌  |
| 管理 Node.js 版本                | ✅ `pnpm env <cmd>`             | ❌  |
| 内容可寻址存储                   | ✅                              | ❌  |
| Side-effects cache(缓存的副作用) | ✅                              | ❌  |
|有锁文件|```pnpm-lock.yaml```   | ```package-lock.json```  |
|即用即删|```pnpm dlx```   | ```npx```  |

## npm缺点

### 重复安装

npm3.0是把所有的依赖包拉平的，但是如果依赖包A里面的子依赖包和主项目的依赖包一致但版本冲突时，npm会在依赖包A的`node_modules`中安装该子依赖包，如果这个时候主项目的依赖包B和依赖包B的子依赖包版本也一致的情况下，npm3也会在依赖包B中再次安装一次子依赖包，这样导致重复安装，从而导致安装包体积变大

例如：

主项目依赖subPackA1.0，packA、packB

packA和packB依赖subPackA2.0

那么安装结果就是：

主项目node_modules: packA、packB、subPackA1.0

packA的node_modules: subPackA2.0

packB的node_modules: subPackA2.0


这样造成了subPackA2.0的重复安装

### 幽灵依赖

由于npm3.0为了减轻依赖包（`node_modules`）的大小，因此是主项目依赖包和依赖包的子依赖包所有拉平放入`node_modules`

因此就会出现明明主项目没有声明某个包，但是主项目也可以直接引入使用这个依赖包

例如：主项目依赖了packA，packA依赖subPackA

主项目可以直接引入subPackA

而pnpm由于使用软连接的方式访问文件，因此解决了npm的以上两个问题

## 参链

> npm

- [npm 模块安装机制与实现原理 ](https://blog.csdn.net/qq_40988677/article/details/125364305)

- [npm 和 package.json 那些不为常人所知的小秘密 ](https://juejin.cn/post/6844903702063480846)

> npx

- [npx 简介 ](https://zhuanlan.zhihu.com/p/269419296)
- [npx 是什么命令？npx 和 npm 有什么区别？ ](https://newsn.net/say/npx.html#npx--ignore-existing)

> pnpm

- [pnpm](https://zhuanlan.zhihu.com/p/457698236)

> npm缺点

- [幽灵依赖是什么，pnpm出现的意义，使用pnpm创建一个vue3项目](https://blog.csdn.net/weixin_59816940/article/details/131395326)
- [💻《npm 进阶指南📖：掌握机制、规避幽灵依赖，开启高效开发》](https://juejin.cn/post/7456649142510682146)