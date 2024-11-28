
在日常的前端开发工作中，我们经常依赖各种命令行工具来提高效率和代码质量。例如，`create-react-app` 和 `eslint` 等工具不仅简化了项目的初始化过程，还能自动执行代码检查和格式化任务。当我们使用这些工具时，它们通常会通过一系列互动式的问答来收集必要的信息，从而根据我们的选择进行相应的配置和安装。


以 `eslint` 工具为例（如下图所示），当你首次运行 `eslint --init` 命令时，它会引导你完成一系列选择题，包括你使用的框架（如 `React`、`Vue.js` 或其他），以及其他配置选项。通过这种方式，`eslint` 能够为你生成一个最适合项目需求的配置文件。


![](https://img2024.cnblogs.com/blog/1408181/202411/1408181-20241127210040266-545722499.png)


本篇文章将介绍在开发命令行工具过程中常用的第三方库。这些库主要分为三类：


* **脚手架框架**：用于解析命令行参数，例如 `eslint --init` 中的 \-\-init。常用的脚手架框架有 `yargs` 和 `command`。
* **命令行输出美化库**：基于 `ANSI Escape` 规范，用于对命令行输出进行颜色和样式美化。常用的美化库有 `chalk` 和 `ora`。
* **交互式命令行库**：用于创建交互式的命令行界面，例如 `eslint` 初始化过程中会提出的问答，如 "Which framework does your project use?"。常用的交互式命令行库有 `inquirer`。


# 脚手架框架


首先，让我们简单回顾一下 `Node.js` 脚手架的开发流程：


* **创建 npm 项目**：使用 `npm init` 命令创建一个新的 npm 项目，并填写相关项目信息。
* **创建脚手架入口文件**：在项目根目录下创建一个入口文件，例如 index.js，并在文件顶部添加 `#!/usr/bin/env node` 以便将其识别为可执行文件。
* **配置 package.json**：在 `package.json` 文件中添加 `bin` 属性，指定脚手架的入口文件路径。
* **添加 npm link**：使用 `npm link` 命令将项目链接到全局环境中，这样就可以在本地通过短指令访问脚手架。


详细的创建及实现功能逻辑可以参考文章[《Node.js 构建命令行工具：实现 ls 命令的 \-a 和 \-l 选项》](https://github.com)


例如，假设我们创建了一个名为 `ice-cli` 的项目，并在其中添加了 `--init` 指令的执行逻辑。那么，我们如何知道用户输入了这项指令呢？这就需要我们在项目中解析命令行参数。


## 自行解析参数


在 `Node.js` 中，我们可以利用内置的 `process` 对象来解析命令行参数。具体来说，可以在入口文件 `index.js` 中执行以下代码：



```
const argv = require("process").argv;
console.log("argv", argv);

```

当用户在命令行中输入 `ice-cli create project --help` 这一长串指令后，通过 `process.argv` 获取到的是一个数组。数组的第一个元素代表 `Node.js` 的执行路径，第二个元素代表当前指令文件的路径，从第三个元素开始则是用户输入的内容。


例如，对于命令 ice\-cli create project \-\-help，process.argv 的输出可能如下所示：



```
[
    '/usr/local/bin/node', // Node.js 执行路径
    '/usr/local/lib/node_modules/ice-cli/index.js', // 当前指令文件路径
    'create', // 用户输入的第一个参数
    'project', // 用户输入的第二个参数
    '--help' // 用户输入的第三个参数
]

```

拿到用户输入的内容后，我们需要对其进行进一步的拆分和处理。用户输入的内容通常包括 **命令（command）** 和 **选项（options）**。例如，在命令 `webpack config ./webpack.config.js` 中，`config` 是命令，`./webpack.config.js` 是命令后面的参数；而在命令 `webpack --help` 中，`--help` 是选项。


通过解析 `process.argv` 数组，我们可以提取出命令和选项，并根据它们执行相应的逻辑。例如：



```
const command = argv[2];  // 获取命令
const args = argv.slice(3);  // 获取命令后面的参数

if (command === 'create') {
  if (args.includes('--help')) {
    console.log('Usage: ice-cli create ');
  } else {
    const projectName = args[0];
    console.log(`Creating project ${projectName}...`);
  }
}

```

在日常开发中，我们通常不会自己去解析命令行参数，因为这涉及到大量的边界情况和错误处理。使用社区广泛认可的第三方库可以更加高效和严谨。其中，`yargs` 和 `commander` 是两个非常优秀的推荐库。


## yargs


`yargs` 是一个功能强大且易于使用的命令行参数解析库。它提供了丰富的 API，可以帮助你轻松地解析命令和选项，并生成详细的帮助信息。


### 安装 yargs


首先，通过 npm 安装 `yargs`：



```
npm install yargs

```

### 实现 \-\-help 和 \-\-version 功能


通过简单的代码就可以实现 `--help` 和 `--version` 功能：



```
const yargs = require("yargs/yargs");
const { hideBin } = require("yargs/helpers");
const arg = hideBin(process.argv);
yargs(arg).argv;

```

运行 `ice-cli --help` ，结果如下图所示：


![](https://img2024.cnblogs.com/blog/1408181/202411/1408181-20241127210054598-1394987870.png)


### 常用属性


`yargs` 采用链式调用的方式为命令设置属性。以下是一些常用的属性：


* **usage()** ：在输入 \-\-help 时会显示的提示信息。
* **demandCommand()** ：最少要输入的命令数量，以及当没有输入命令时的提示。
* **recommendCommands()** ：如果输入的指令不完整，会给出最近似命令的提示，例如：“Did you mean xx?”
* **strict()** ：严格模式，输入错误命令时会给出提示。
* **alias()** ：为指令取别名。
* **options()** ：定义多个全局选项，在任何场景都可以访问到。
* **option()** ：定义单个全局选项，在任何场景都可以访问到。
* **group()** ：将一些命令聚合到一个分类中。
* **command()** ：定义指令。
* **epilogue()** ：定义结尾信息。


### 示例代码


将上述命令组合起来，示例如下：



```
const yargs = require("yargs/yargs");
const { hideBin } = require("yargs/helpers");
const arg = hideBin(process.argv);
const cli = yargs(arg);
cli
  .usage("Usage: ice-ls [command] ")
  .demandCommand(
    1,
    "A command is required. Pass --help to see all avaiable commands and options."
  )
  .recommendCommands()
  .strict()
  .alias("h", "help")
  .options({
    debug: {
      type: "boolean",
      describe: "Bootstarap debug mode",
      alias: "d",
    },
  })
  .group(["debug"], "Dev options:")
  .command({
    command: "list",
    aliases: ["ls", "la", "ll"],
    describe: "List total packages",
    builder: (argv) => {
      console.log("builder", argv);
    },
    handler: (argv) => {
      console.log("handler", argv);
    },
  })
  .epilogue("You own footer description").argv;

```

当执行 `ice-ls` 命令时，输出如下：


* 第一行出现 `usage` 函数配置的提示：Usage: ice\-ls \[command] 。
* 接着是 `command` 函数配置的指令 list，以及它的别名 ls, la, ll。
* 然后是通过 `group` 函数分组的 Dev options。
* 下面是 `yargs` 默认提供的选项 \-\-version 和 \-\-help。
* 接着是 `epilogue` 函数配置的尾部描述。
* 最后一行是 `demandCommand` 提示： A command is required. Pass \-\-help to see all avaiable commands and options。因为执行命令 ice\-ls 的时候没有提供具体指令。


执行 `ice-cli list` 和 `ice-ls list --debug`，此时程序进入 `command` 函数中，执行 `builder` 函数 和 `handler` 函数，可以在这里编写实际的功能逻辑。


![](https://img2024.cnblogs.com/blog/1408181/202411/1408181-20241127210106950-1412351925.png)


## commander


`commander` 也是一个功能强大的命令行参数解析库，但它在使用方式和 `API` 设计上和 `yargs` 有一些差异。


### 安装 commander


首先，通过 npm 安装 `commander`：



```
npm install commander

```

### 实现 \-\-help 和 \-\-version 功能


`commander` 通过简单的配置就可以生成 `usage` 提示以及 `--help` 和 `--version` 功能。



```
const commander = require("commander");
const pkg = require("../package.json");
const program = new commander.Command();
program
  .name(Object.keys(pkg.bin)[0])
  .usage(" [options]")
  .version(pkg.version);

program.parse(process.argv);

```

执行上述代码后，运行 `ice-cli --help` 的输出如下所示：



```
Usage: ice-cli  [options]

Options:
  -V, --version  output the version number
  -h, --help     display help for command

```

### 注册指令


`commander` 和 `yargs` 在注册指令的语法上有一些区别。`yargs` 使用链式调用，而 `commander` 注册指令后返回值并不是自身，因此不能通过链式调用来注册多个指令。



```
// 注册 clone 命令
program
  .command("clone  [destination]")
  .description("clone a repository")
  .option("-f --force", "是否强制克隆")
  .usage("[options]")
  .action((source, destination, cmdObj) => {
    console.log("do clone", source, destination);
  });
  
// 劫持所有未定义的指令
program
  .arguments(" [options]")
  .description("test command", {
    cmd: "command to run",
    options: "options for command",
  })
  .action((cmd, options) => {
    console.log(cmd, options);
  });  

```

当执行 `ice-cli clone a b` 时，输出为 "do clone a b"。当执行 `ice-cli create c` 时，输出为 "create c"。


### 注册子命令


`commander` 注册子命令的方式也非常简单。以下是一个示例：



```
const service = new commander.Command("service");
service
  .command("start [port]")
  .description("start service at some port")
  .action((port) => {
    console.log('>>>service start', port)
  });
service
  .command("stop")
  .description("stop service")
  .action(() => {
    console.log('>>>service stop')
  });

```

当执行 `ice-cli service start 8000` 时，会输出 "\>\>\>service start 8000"。当执行 `ice-cli service stop` 时，会输出 "\>\>\>service stop"。


`yargs` 和 `commander` 解析命令行参数，生成帮助信息，并注册命令。它们提供了强大的命令行接口构建能力，使得命令行工具更加灵活和易用。


# 命令行输出美化库


在进行命令行交互时，经常需要对某些内容加粗、加字体颜色，以区分用户选中的内容和需要重点关注的问题。为了实现这些效果，存在一个命令行渲染标准，称为 `ANSI escape code`。此外，还有一些成熟的第三方库，如 `chalk` 和 `ora`，可以帮助我们更方便地实现这些功能。


## ANSI escape code


`ANSI escape code` 是一种用于控制终端输出的标准。通过特定的编码序列，可以在命令行中实现颜色、加粗等效果。


### 示例


在 bin 文件夹下创建 ansi.js 文件，文件中定义如下代码：



```
console.log("\x1B[31mThis text is red\x1B[0m");

```

通过 `node` 执行该 JS 文件，显示的是红色文本，内容为 "This text is red"。如图所示：


![](https://img2024.cnblogs.com/blog/1408181/202411/1408181-20241127210120922-527147714.png)


### 编码解析


即使我们没有借助任何第三方库，仅通过一行文本就能实现命令行中的颜色和样式变化。这一行看似“乱码”的文本实际上是由 `ANSI Escape Codes` 组成的。下面是对这些字符的详细拆解：


* **\\x1B**：这是转义字符，表示 ASCII 值为 27 的字符，也常表示为 ESC。它是所有 ANSI Escape Codes 的前缀。
* **\[**：这是一个分隔符，表示接下来是一个控制序列。
* **31**：这是一个数字代码，表示设置前景色为红色。
* **m**：这是一个终止符，表示控制序列的结束。
* **\\x1B\[0m**：重置所有文本属性，包括颜色和样式。


### 查找期望的样式


要找到期望的样式，可以在 [ansi escape code 官网](https://github.com):[veee加速器](https://youhaochi.com) 查找。例如，31 代表红色前景色，41 代表红色背景色。


![](https://img2024.cnblogs.com/blog/1408181/202411/1408181-20241127210129967-1658440314.png)


虽然可以直接使用 `ANSI Escape Codes` 来实现颜色和样式变化，但在实际开发中这样做会非常繁琐。你需要手动定义转义字符、分隔符，还要查找每个颜色对应的编码。幸运的是，已经有成熟的第三方库可以帮助我们解决这些问题，例如 `chalk` 和 `ora`。


## chalk


`chalk` 是一个用于颜色渲染的库，其语法非常简单，通过方法名就能知道其用途。常见的方法包括：


* `rgb(r, g, b)`：定义自定义颜色。
* `blue`：设置蓝色字体。
* `bold`：设置字体加粗。
* `green`：设置绿色字体。
* `underline`：设置下划线。


这些方法名与 CSS 中的样式名称相似，使得使用起来非常直观。`chalk` 支持多种使用形式，包括直接使用、拼接、链式调用、传入多个参数和嵌套调用。


### 安装


首先通过 npm 安装 `chalk`



```
npm install chalk

```

### 基本使用


`chalk` 是以 `ES module` 方式实现的，需要通过 `import` 方式引入。如果希望在 Node.js 环境中运行，可以将文件后缀名定义为 `.mjs`。


例如以下代码：



```
import chalk from "chalk";

// 直接使用
console.log("hello chalk");
// 定义自定义颜色
console.log(chalk.rgb(255, 0, 0)("hello nodejs"));
// 拼接不同样式
console.log(chalk.blue.bold("hello ") + chalk.green("world"));
// 使用十六进制颜色
console.log(chalk.hex("#ff0000")("it is a nice day"));
// 链式调用和嵌套调用
console.log(
  chalk.green(
    "I am a green line " +
    chalk.blue.underline.bold("with a blue substring") +
    " that becomes green again!"
  )
);

```

执行上述代码后，命令行中的输出效果如下所示：


![](https://img2024.cnblogs.com/blog/1408181/202411/1408181-20241127210142361-762860526.png)


## ora


`ora` 是一个用于显示加载动画的库，非常适合在命令行应用中显示进度和状态。它以 `ES module` 方式导出，需要定义 `.mjs` 文件。


### 安装


首先通过 npm 安装 `ora`：



```
npm install ora

```

### 基本使用


`ora` 在使用时需要手动调用开始和结束方法。以下是一个简单的示例，显示一个加载动画：



```
import ora from "ora";
const spinner = ora({
  text: "loading",
  spinner: "dots",
}).start();

```

执行上述代码后，命令行中会显示一个持续的加载动画，如下图所示：


![](https://img2024.cnblogs.com/blog/1408181/202411/1408181-20241127210152107-1534737066.png)


### 自定义属性


`ora` 还支持定义其他属性，如加载动画效果、颜色、前缀文本等。


属性说明：


* **text**：初始加载文本。
* **spinner**：加载动画效果，可以是一个预定义的字符串（如 dots、line 等）或自定义对象。
* **color**：加载动画的颜色。
* **prefixText**：加载文本的前缀。
* **start()**：启动加载动画。
* **stop()**：停止加载动画。
* **succeed(message)**：停止加载动画并显示成功消息。
* **fail(message)**：停止加载动画并显示失败消息。
* **warn(message)**：停止加载动画并显示警告消息。
* **info(message)**：停止加载动画并显示信息消息。


以下是一个更复杂的示例，展示了如何动态更新加载文本并最终停止加载动画：



```
import ora from "ora";
const spinner = ora({
  text: "loading",
  spinner: "dots",
}).start();

// 设置加载颜色和前缀文本
spinner.color = "red";
spinner.prefixText = "download ora:";

let percent = 0;
let task = setInterval(() => {
  percent += 10;
  spinner.text = "Loading..." + percent + "%";
  if (percent === 100) {
    spinner.stop();
    spinner.succeed("download success");
    clearInterval(task);
  }
}, 1000);

```

按以上逻辑，执行过程的中间态如下所示：


![](https://img2024.cnblogs.com/blog/1408181/202411/1408181-20241127210205962-814217610.png)


# 交互式命令行


在命令行应用中，经常会涉及到一些交互逻辑，例如在 `eslint` 初始化过程中会询问用户当前使用的框架是 `React`、`Vue` 还是其他框架。用户可以通过键盘的上下左右键和回车进行选择。`inquirer` 就是这样一个用于命令行交互的第三方库。


## inquirer


### 安装


`inquirer` 是一个强大的命令行交互库，可以轻松地创建用户友好的命令行界面。



```
npm install inquirer

```

### 基本使用


以下是一个简单的示例，展示了如何使用 `inquirer` 创建一个列表选择：



```
import inquirer from "inquirer";

inquirer
  .prompt([
    {
      type: "list",
      name: "language",
      message: "language",
      choices: [
        {
          value: 1,
          name: "react",
        },
        {
          value: 2,
          name: "vue",
        },
        {
          value: 3,
          name: "angular",
        },
      ],

    },
  ])
  .then((res) => {
    console.log("anwser", res);
  });

```

执行上述代码后，命令行中会出现一个选择列表，如下图所示：


![](https://img2024.cnblogs.com/blog/1408181/202411/1408181-20241127210245479-727142151.png)


### 多种交互类型


`inquirer` 支持多种交互类型，不仅限于列表选择，还可以输入文本、密码、多选等。
以下是一个示例，展示了多种类型的交互问题：



```

import inquirer from "inquirer";
inquirer
  .prompt([
    {
      type: "input",
      name: "yourName",
      message: "Your name",
    },
    {
      type: "list",
      name: "language",
      message: "language",
      choices: [
        {
          value: 1,
          name: "react",
        },
        {
          value: 2,
          name: "vue",
        },
        {
          value: 3,
          name: "angular",
        },
      ],
    },
    {
      type: "expand",
      name: "color",
      message: "color",
      choices: [
        {
          key: "R",
          value: "red",
        },
        {
          key: "G",
          value: "green",
        },
        {
          key: "B",
          value: "blue",
        },
      ],
    },
    {
      type: "checkbox",
      name: "fruits",
      message: "fruits",
      choices: [
        {
          value: 1,
          name: "apple",
        },
        {
          value: 2,
          name: "banana",
        },
        {
          value: 3,
          name: "orange",
        },
      ],
    },
    {
      type: "password",
      name: "password",
      message: "password",
    },
  ])
  .then((res) => {
    console.log("anwser", res);
  });

```

执行上述代码后，命令行中会依次显示多个交互问题，最终，所有的用户输入都会在 `then` 方法中统一获取，如下图所示：


![](https://img2024.cnblogs.com/blog/1408181/202411/1408181-20241127210226197-1032849989.png)


通过 `inquirer`，你可以轻松地在命令行应用中实现各种交互逻辑。其丰富的交互类型和灵活的校验机制使得 `inquirer` 成为一个非常实用的命令行交互库。


通过结合这些工具和库，可以轻松地构建出功能强大、用户体验良好的命令行应用。


如果你对前端工程化有兴趣，或者想了解更多相关的内容，欢迎查看我的其他文章，这些内容将持续更新，希望能给你带来更多的灵感和技术分享\~


