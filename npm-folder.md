# npm folder

npm 安装的包会被放在node_modules中，有两个位置：

1. 如果是local install，pacakges会被放在当前package root目录下的node_modules；
2. 如果是global install，packages会被放在prefix/lib/node_modules；

先确定好如何使用package，再选取对应的安装方式：

1. 如果希望在项目里用require()引入package，应采用local install，而global install无法做到；
2. 如果只需在命令行使用某些package中的命令，比如whistle package的w2命令，就采用global install；
3. 如果想在本地项目中引入全局包，则用npm link命令在本地项目创建全局包的软链；这主要用于调试依赖包的场景

## prefix

在global mode下，prefix是node的位置（执行npm prefix -g返回的目录）；在local mode下，它是从当前目录开始（包含当前目录）往上找最近的、包含package.json或node_modules的folder（执行npm prefix返回的目录），没找到就返回当前工作目录。

全局模式下的prefix folder下通常有以下内容：

- CHANGELOG.md
- LICENSE
- README.md
- bin（node、npm命令都放在这里）
- include
- lib（node_modules在里面）
- share

## node_modules

被安装的 packages 放在 prefix 下的node_modules folder中，这意味着locally安装package后，可以用require('package-name')引入主module，用require('package-name/lib/path/to/sub/module')引入其它module。

全局安装的package被放在prefix/lib/node_modules中（Unix系统），

## executables

全局安装时，executables会在prefix/bin下（Unix）创建symbol link，请确保prefix/bin被加入到`PATH`环境变量中，便于在任意目录执行。

局部安装时，executables会在./node_modules/.bin下创建symbol link，执行npm run xxx时，npm 会查找 `node_modules/.bin` 目录下是否有对应的可执行文件。如果找到了，npm 会使用该可执行文件执行对应的命令。

## packages installation process

1. 执行`npm install foo@1.2.3`，局部安装；
2. 找合适的prefix。从`$PWD`开始，往上查找folder tree，找到包含package.json或node_modules的folder，这个folder就是prefix，如果没找到，当前folder就是prefix；（这就意味着在当前项目的深层目录下（比如src下）执行npm install，也能正确安装依赖）
3. pacakge首先被加载到cache，然后unpack到prefix/node_modules/foo，对于foo package自身依赖的其它package，会被unpack到prefix/node_modules/foo/node_modules/...；
4. 所有可执行文件都被创建软链，放到prefix/node_moduels/.bin/下；

全局安装和上述步骤一致，只是安装位置不同。

## package循环依赖、冲突

对于循环依赖（Cycles），比如foo -> bar -> baz -> bar，node的模块系统会从当前package本应安装的folder**向上查找**，如果该package已经在node_modules安装过了，就不会重复安装，两个package的版本也必须相同才能叫已经安装，通常node还会有其它的优化手段，比如它会像React中的 state 提升那样，把两个相同的深层package提升到更高的folder层级。

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

在发布过程中，npm查看node_modules folder，如果某些package并没有被放在`bundleDependencies`数组（已弃用，现在直接用de vDependencies和dependencies区分开）中，它就不会被包含到package tarball，这样可以避免一些仅用于开发的依赖也被打包发布。

`devDependencies` 用于指定仅在开发过程中使用的依赖项，而不是在生产环境中运行应用程序时需要的依赖项。这些依赖项可能包括测试框架、构建工具等。

`dependencies` 字段用于指定在生产环境中运行应用程序时需要的依赖项。这些依赖项将会被包含在最终的生产环境包(bundle)中。