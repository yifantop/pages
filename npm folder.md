# npm folder

npm安装的各种packages会放在个位置：

1. local install，pacakges会被放在当前package root目录下的./node_modules
2. global install，packages会被放在prefix/lib/node_modules

使用package的方式不同，就有了以上两种不同的安装方法：

1. 如果希望在项目里使用require()引入package，采用local install package
2. 如果希望在命令行使用package中的命令，比如whistle package的w2命令，就采用global install package
3. npm link也可以实现在本地代码里require()全局package

## prefix

prefix是node的位置，在global mode下，它是node命令所在的folder。否则，它是从当前目录开始（包含当前目录）往上找最近的、包含package.json或node_modules的folder。

有两个命令可以查看直接prefix，执行 `npm prefix -g` 返回的是全局prefix。

这个folder通常有以下内容：

- CHANGELOG.md
- LICENSE
- README.md
- bin，node、npm命令都放在这里
- include
- lib，node_modules在里面
- share

执行 `npm prefix` 返回的是当前package的root目录，如果没找到package.json或node_modules（向上搜寻也没有），就返回当前工作目录。

## node_modules

安装的packages会被放到prefix下的node_modules folder，这意味着locally安装package后，可以用require('package-name')引入主module，用require('package-name/lib/path/to/sub/module')引入其它module。

准确地说，全局安装的package被放在prefix/lib/node_modules中（Unix系统），

## executables

全局安装时，executables都放在prefix/bin下（Unix），请确保prefix/bin被加入到`PATH`环境变量中，便于在任意目录执行。

局部安装时，executables会在./node_modules/.bin下创建symbol link，于是就可以使用npm scripts执行这些命令。

## packages installation process

1. 执行`npm install foo@1.2.3`，局部安装；
2. 找合适的prefix。从`$PWD`开始，往上查找folder tree，找到包含package.json或node_modules的folder，这个folder就是prefix，如果没找到，当前folder就是prefix；
3. pacakge首先被加载到cache，然后unpack到prefix/node_modules/foo，对于foo package自身依赖的其它package，会被unpack到prefix/node_modules/foo/node_modules/...；
4. 所有可执行文件都被创建软链，放到prefix/node_moduels/.bin/下；

全局安装和上述步骤一致，只是安装的folder不同。

## package循环依赖、冲突

对于循环依赖（Cycles），比如foo -> bar -> baz -> bar，node的模块系统会从当前package本应安装的folder向上查找，如果该package已经在node_modules安装过了，就不会重复安装，两个package的版本也必须相同才能叫已经安装过，通常node还会有其它的优化手段，比如它会像React中的 state 提升那样，把两个相同的深层package提升到更高的folder层级。

以下是一个例子

```plain
foo
+-- blerg@1.2.5
+-- bar@1.2.3
|   +-- blerg@1.x (latest=1.3.7)
|   +-- baz@2.x
|   |   `-- quux@3.x
|   |       `-- bar@1.2.3 (cycle)
|   `-- asdf@*
`-- baz@1.2.3
    `-- quux@3.x
        `-- bar
foo
+-- node_modules
    +-- blerg (1.2.5) <---[A]
    +-- bar (1.2.3) <---[B]
    |   +-- node_modules
    |       +-- baz (2.0.2) <---[C]
    +-- asdf (2.3.4)
    +-- baz (1.2.3) <---[D]
    +-- quux (3.2.0) <---[E]
```

## publishing

在发布过程中，npm查看node_modules folder，如果某些package并没有被放在`bundleDependencies`数组中，它就不会被包含到package tarball，这样可以避免一些仅用于开发的依赖也被打包发布。