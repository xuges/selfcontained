# self-contained 自包含

### 说明

只要涉及到原生开发，动态链接是绕不开的话题，即使能够保证静态链接第三方库，系统的libc版本也可能是巨大的障碍(也可以使用musl libc，属于另一种方式)。

- 在低版本Linux发行版上运行高版本编译的程序可能会报错 `GLIBC_XXX not found` 或 `GLIBCXX_XXX not found`

- 开发时运行正常的软件发布后运行可能会报错 `load shared object no such file`

这些都是常见的动态链接依赖问题。

自包含(self-contained)是指应用程序包含自己运行所需要的一切依赖，本项目可帮助开发者将自己的二进制可执行文件进行自包含打包，从而解决动态链接依赖问题。

### 用法

假设二进制可执行文件为 `a.out` ，使用以下命令生成自包含目录：

```bash
chmod +x make-selfcontained
./make-selfcontained a.out
```

命令执行成功后，会在当前目录下生成 `selfcontained` 目录：

```
selfcontained/
├── bin
│   └── a.out
├── lib
│   ├── ld-linux-x86-64.so.2
│   ├── libc.so.6
│   ├── libgcc_s.so.1
│   ├── libm.so.6
│   └── libstdc++.so.6
└── run.sh
```

说明：
- `bin` 存放 `make-selfcontained` 传入的 `a.out` 复制体
- `lib` 下放入 `a.out` 依赖的所有动态链接库(包括 `libc`)
- `run.sh` 是运行 `a.out` 的执行脚本，文件名称可自行修改，位置不能动

生成 `selfcontained` 目录后，可自行添加其他运行时用到的资源，`selfcontained` 目录名称可自行修改。

**注意：如果使用 `dlopen` 的方式动态调用动态链接库，需要手动管理动态库文件。**

### 原理

`make-selfcontained` 本身也是一个脚本，它会查找目标可执行文件依赖的动态链接库文件并复制到 `selfcontained` 目录里，然后在生成的 `run.sh` 脚本里通过直接调用 `ld-linux-x86-64.so.2` 加载器的方式运行目标可执行文件，实现从 `libc` 开始，所有的动态链接库都优先使用自带的。

参考资料：https://www.cnblogs.com/pengdonglin137/p/17623177.html

### 依赖

`make-selfcontained` 脚本使用了以下命令：

```bash
ldd
awk
```
