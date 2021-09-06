# Cobra

### A Framework for Modern CLI Apps in Go

![](https://cobra.dev/home/logo.png)

## About

Cobra 是 Go 的 CLI 框架。它包含一个用于创建强大的现代 CLI 应用程序的库和一个用于快速生成基于 Cobra 的应用程序和命令文件的工具。
它由 Go 团队成员 spf13 为 Hugo 创建，并已被最流行的 Go 项目采用。

### Cobra provides:

- 简单的基于子命令的 CLI：应用服务器、应用获取等。
- 完全符合 POSIX 的标志（包括短版和长版）
- 嵌套子命令
- 全局、本地和级联标志
- 使用 cobra init appname 和 cobra add cmdname 轻松生成应用程序和命令
- 智能建议（应用服务器……你是说应用服务器吗？）
- 命令和标志的自动帮助生成`-h`、`--help` 等的自动帮助标志识别。
- 为您的应用程序自动生成 bash 自动完成
- 为您的应用程序自动生成手册页
- 命令别名，这样您就可以在不破坏它们的情况下更改内容
- 定义自己的帮助、用法等的灵活性。
- 可选与 viper 紧密集成，用于 12 要素应用程序

## 安装

Cobra 很容易使用 。首先，使用 `go get` 安装最新版本的库。此命令将安装 cobra 生成器可执行文件以及库及其依赖项：

```shell
go get -u github.com/spf13/cobra/cobra
```

然后，在你的项目中包含Cobra

```go
import "github.com/spf13/cobra"
```

## 概念

Cobra 建立在命令、参数和标志的结构上。

**Command**代表动作，**Args** 是事物，**Flags** 是这些动作的修饰符。

最好的应用程序在使用时会读起来像句子。用户将知道如何使用该应用程序，因为他们本能地了解如何使用它。

## Commands

命令是应用程序的中心点。应用程序支持的每个交互都将包含在一个命令中。一个命令可以有子命令，并可以选择运行一个动作。
[More about cobra.Command](https://godoc.org/github.com/spf13/cobra#Command)

## Flags

Flag是一种修改命令行为的方法。 Cobra 支持完全符合 `POSIX` 的标志以及 Go 标志包。 Cobra 命令可以定义可用于子命令的标志和仅可用于该命令的标志。

Flag功能由 `pflag` 库提供，它是标志标准库的一个分支，它在添加 `POSIX` 合规性的同时保持相同的接口。

## Getting Started

虽然欢迎您提供自己的项目结构，但通常基于 Cobra 的应用程序将遵循以下组织结构：

```
▾ appName/
    ▾ cmd/
        add.go
        your.go
        commands.go
        here.go
      main.go
```

在 Cobra 应用程序中，通常 `main.go` 文件非常简单。它只有一个目的：初始化 Cobra。

```go
package main

import (
  "{pathToYourApp}/cmd"
)

func main() {
  cmd.Execute()
}
```

## 使用Cobra生成器

Cobra 提供了自己的程序，可以创建您的应用程序并添加您想要的任何命令。这是将 Cobra 整合到您的应用程序中的最简单方法。

[在这](https://github.com/spf13/cobra/blob/master/cobra/README.md)你可以找到更多关于Cobra生成器的信息。

## 使用Cobra库

要手动实现 Cobra，您需要创建一个简单的 `main.go` 文件和一个 `rootCmd` 文件。您可以选择提供您认为合适的其他命令。

### 创建 `rootCmd`

Cobra 不需要任何特殊的构造函数。只需创建您的命令。

理想情况下，您将其放在 `app/cmd/root.go` 中：

```go
var rootCmd = &cobra.Command{
  Use:   "hugo",
  Short: "Hugo is a very fast static site generator",
  Long: `A Fast and Flexible Static Site Generator built with
                love by spf13 and friends in Go.
                Complete documentation is available at http://hugo.spf13.com`,
  Run: func(cmd *cobra.Command, args []string) {
    // Do Stuff Here
  },
}

func Execute() {
  if err := rootCmd.Execute(); err != nil {
    fmt.Println(err)
    os.Exit(1)
  }
}
```

标志和处理配置将在 `init()` 函数中定义。

例如 `cmd/root.go`:

```go
import (
  "fmt"
  "os"

  homedir "github.com/mitchellh/go-homedir"
  "github.com/spf13/cobra"
  "github.com/spf13/viper"
)

func init() {
  cobra.OnInitialize(initConfig)
  rootCmd.PersistentFlags().StringVar(&cfgFile, "config", "", "config file (default is $HOME/.cobra.yaml)")
  rootCmd.PersistentFlags().StringVarP(&projectBase, "projectbase", "b", "", "base project directory eg. github.com/spf13/")
  rootCmd.PersistentFlags().StringP("author", "a", "YOUR NAME", "Author name for copyright attribution")
  rootCmd.PersistentFlags().StringVarP(&userLicense, "license", "l", "", "Name of license for the project (can provide `licensetext` in config)")
  rootCmd.PersistentFlags().Bool("viper", true, "Use Viper for configuration")
  viper.BindPFlag("author", rootCmd.PersistentFlags().Lookup("author"))
  viper.BindPFlag("projectbase", rootCmd.PersistentFlags().Lookup("projectbase"))
  viper.BindPFlag("useViper", rootCmd.PersistentFlags().Lookup("viper"))
  viper.SetDefault("author", "NAME HERE <EMAIL ADDRESS>")
  viper.SetDefault("license", "apache")
}

func initConfig() {
  // Don't forget to read config either from cfgFile or from home directory!
  if cfgFile != "" {
    // Use config file from the flag.
    viper.SetConfigFile(cfgFile)
  } else {
    // Find home directory.
    home, err := homedir.Dir()
    if err != nil {
      fmt.Println(err)
      os.Exit(1)
    }

    // Search config in home directory with name ".cobra" (without extension).
    viper.AddConfigPath(home)
    viper.SetConfigName(".cobra")
  }

  if err := viper.ReadInConfig(); err != nil {
    fmt.Println("Can't read config:", err)
    os.Exit(1)
  }
}
```

### 创建`main.go`

使用 root 命令，您需要让 `main` 函数执行它。为清楚起见，`Execute` 应该在`root`上运行，尽管它可以在任何命令上调用。

在 Cobra 应用程序中，通常 `main.go` 文件非常简单。它的目的仅是初始化 Cobra。

```go
package main

import (
  "{pathToYourApp}/cmd"
)

func main() {
  cmd.Execute()
}
```

### 创建子命令

可以定义其他命令，并且通常每个命令在 `cmd/` 目录中都有自己的文件。

如果你想创建一个版本命令，你可以创建 `cmd/version.go` 并用以下内容填充它：

```go
package cmd

import (
  "fmt"

  "github.com/spf13/cobra"
)

func init() {
  rootCmd.AddCommand(versionCmd)
}

var versionCmd = &cobra.Command{
  Use:   "version",
  Short: "Print the version number of Hugo",
  Long:  `All software has versions. This is Hugo's`,
  Run: func(cmd *cobra.Command, args []string) {
    fmt.Println("Hugo Static Site Generator v0.9 -- HEAD")
  },
}
```

## 使用`Flags`

标志提供修饰符来控制操作命令的操作方式。

### 为命令分配标志

由于标志是在不同位置定义和使用的，我们需要在外部定义一个具有正确范围的变量来分配要使用的标志。

```go
var Verbose bool
var Source string
```

有两种不同的方法来分配标志。

### 持久标志

标志可以是“持久的”，这意味着该标志可用于分配给它的命令以及该命令下的每个命令。对于全局标志，在根上分配一个标志作为持久标志。

```go
rootCmd.PersistentFlags().BoolVarP(&Verbose, "verbose", "v", false, "verbose output")
```

### 本地标志

也可以在本地分配一个标志，该标志仅适用于该特定命令。

```go
rootCmd.Flags().StringVarP(&Source, "source", "s", "", "Source directory to read from")
```

### 父命令上的本地标志

默认情况下，Cobra 仅解析目标命令上的本地标志，忽略父命令上的任何本地标志。通过启用 `Command.TraverseChildren` Cobra 将在执行目标命令之前解析每个命令上的本地标志。

```go
command := cobra.Command{
  Use: "print [OPTIONS] [COMMANDS]",
  TraverseChildren: true,
}
```

### 使用配置绑定标志

您还可以使用 `viper` 绑定您的标志：

```go
var author string

func init() {
  rootCmd.PersistentFlags().StringVar(&author, "author", "YOUR NAME", "Author name for copyright attribution")
  viper.BindPFlag("author", rootCmd.PersistentFlags().Lookup("author"))
}
```

在此示例中，持久标志`author`与 `viper` 绑定。请注意，当用户未提供 `--author` 标志时，变量 `author` 不会设置为配置中的值。

More in [viper documentation](https://github.com/spf13/viper#working-with-flags).

### 必需的标志

默认情况下，标志是可选的。相反，如果您希望命令在未设置标志时报告错误，请将其标记为必须的：

```go
rootCmd.Flags().StringVarP(&Region, "region", "r", "", "AWS region (required)")
rootCmd.MarkFlagRequired("region")
```

## 位置和自定义参数

可以使用 `Command` 的 `Args` 字段指定位置参数的验证。

内置了以下验证器：

- `NoArgs` - 如果有任何位置参数，该命令将报告错误。
- `ArbitraryArgs` - 该命令将接受任何参数。
- `OnlyValidArgs` - 如果存在任何不在 `Command` 的 `ValidArgs` 字段中的位置参数，则该命令将报告错误。
- `MinimumNArgs(int)` - 如果没有至少 N 个位置参数，该命令将报告错误。
- `MaximumNArgs(int)` - 如果有超过 N 个位置参数，该命令将报告错误。
- `ExactArgs(int)` - 如果没有正好 N 个位置参数，该命令将报告错误。
- `ExactValidArgs(int)` -如果没有正好 N 个位置参数，或者如果有任何不在命令的 `ValidArgs` 字段中的位置参数，该命令将报告并出错
- `RangeArgs(min, max)` - 如果 `args` 的数量不在预期 `args` 的最小和最大数量之间，该命令将报告错误。

设置自定义验证器的示例：

```go
var cmd = &cobra.Command{
  Short: "hello",
  Args: func(cmd *cobra.Command, args []string) error {
    if len(args) < 1 {
      return errors.New("requires at least one arg")
    }
    if myapp.IsValidColor(args[0]) {
      return nil
    }
    return fmt.Errorf("invalid color specified: %s", args[0])
  },
  Run: func(cmd *cobra.Command, args []string) {
    fmt.Println("Hello, World!")
  },
}
```

## 例子

在下面的示例中，我们定义了三个命令。两个位于顶级命令，一个 (`cmdTimes`) 是其中一个顶级命令的子命令。在这种情况下，`root` 是不可执行的，这意味着需要一个子命令。这是通过不为“`rootCmd`”提供“`Run`”来实现的。

我们只为单个命令定义了一个标志。

有关标志的更多文档，请访问 https://github.com/spf13/pflag。

```go
package main

import (
  "fmt"
  "strings"

  "github.com/spf13/cobra"
)

func main() {
  var echoTimes int

  var cmdPrint = &cobra.Command{
    Use:   "print [string to print]",
    Short: "Print anything to the screen",
    Long: `print is for printing anything back to the screen.
For many years people have printed back to the screen.`,
    Args: cobra.MinimumNArgs(1),
    Run: func(cmd *cobra.Command, args []string) {
      fmt.Println("Print: " + strings.Join(args, " "))
    },
  }

  var cmdEcho = &cobra.Command{
    Use:   "echo [string to echo]",
    Short: "Echo anything to the screen",
    Long: `echo is for echoing anything back.
Echo works a lot like print, except it has a child command.`,
    Args: cobra.MinimumNArgs(1),
    Run: func(cmd *cobra.Command, args []string) {
      fmt.Println("Print: " + strings.Join(args, " "))
    },
  }

  var cmdTimes = &cobra.Command{
    Use:   "times [# times] [string to echo]",
    Short: "Echo anything to the screen more times",
    Long: `echo things multiple times back to the user by providing
a count and a string.`,
    Args: cobra.MinimumNArgs(1),
    Run: func(cmd *cobra.Command, args []string) {
      for i := 0; i < echoTimes; i++ {
        fmt.Println("Echo: " + strings.Join(args, " "))
      }
    },
  }

  cmdTimes.Flags().IntVarP(&echoTimes, "times", "t", 1, "times to echo the input")

  var rootCmd = &cobra.Command{Use: "app"}
  rootCmd.AddCommand(cmdPrint, cmdEcho)
  cmdEcho.AddCommand(cmdTimes)
  rootCmd.Execute()
}
```

有关更大应用程序的更完整示例，请查看[Hugo](http://gohugo.io/)。

## 帮助命令

当您有子命令时，Cobra 会自动向您的应用程序添加帮助命令。这将在用户运行“应用帮助”时调用。此外，帮助还将支持所有其他命令作为输入。比如说，你有一个名为“`create`”的命令，没有任何额外的配置； Cobra 将在调用“`app help create`”时工作。每个命令都会自动添加“`-help`”标志。

### 举例

以下输出由 Cobra 自动生成。除了命令和标志定义之外，什么都不需要。

```shell
$ cobra help

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.

Usage:
  cobra [command]

Available Commands:
  add         Add a command to a Cobra Application
  help        Help about any command
  init        Initialize a Cobra Application

Flags:
  -a, --author string    author name for copyright attribution (default "YOUR NAME")
      --config string    config file (default is $HOME/.cobra.yaml)
  -h, --help             help for cobra
  -l, --license string   name of license for the project
      --viper            use Viper for configuration (default true)

Use "cobra [command] --help" for more information about a command.
```

帮助只是一个命令，就像任何其他命令一样。它没有特殊的逻辑或行为。事实上，如果你愿意，你可以提供你自己的。

### 定义你自己的帮助命令

您可以通过以下方式为默认命令提供您自己的帮助命令或您自己的模板：

```go
cmd.SetHelpCommand(cmd *Command)
cmd.SetHelpFunc(f func(*Command, []string))
cmd.SetHelpTemplate(s string)
```

后两者也适用于任何子命令。

## 用法提示信息

当用户提供无效标志或无效命令时，Cobra 会通过向用户显示“用法”来做出响应。

### 举例

您可以从上面的帮助中认识到这一点。这是因为默认帮助将用法作为其输出的一部分嵌入。

```shell
$ cobra --invalid
Error: unknown flag: --invalid
Usage:
  cobra [command]

Available Commands:
  add         Add a command to a Cobra Application
  help        Help about any command
  init        Initialize a Cobra Application

Flags:
  -a, --author string    author name for copyright attribution (default "YOUR NAME")
      --config string    config file (default is $HOME/.cobra.yaml)
  -h, --help             help for cobra
  -l, --license string   name of license for the project
      --viper            use Viper for configuration (default true)

Use "cobra [command] --help" for more information about a command.
```

### 定制你自己的用法提示

您可以提供自己的使用函数或模板供 Cobra 使用。像帮助一样，函数和模板可以通过公共方法覆盖：

```go
cmd.SetUsageFunc(f func(*Command) error)
cmd.SetUsageTemplate(s string)
```

## Version Flag

如果在 `root` 命令上设置了 `Version` 字段，Cobra 会添加一个顶级“`-version`”标志。运行带有‘`-version`’标志的应用程序将使用版本模板将版本打印到标准输出。可以使用 `cmd.SetVersionTemplate(s string)` 函数自定义模板。

## `PreRun` 和 `PostRun` 钩子

可以在命令的主 `Run` 函数之前或之后运行函数。 `PersistentPreRun` 和 `PreRun` 函数将在 `Run` 之前执行。 `PersistentPostRun` 和 `PostRun` 将在 `Run` 之后执行。 `Persistent*Run` 函数将被子级继承，如果它们不声明它们自己的。这些函数按以下顺序运行：

- `PersistentPreRun`
- `PreRun`
- `Run`
- `PostRun`
- `PersistentPostRun`

下面是使用所有这些函数的两个命令的示例。当子命令被执行时，它会运行 `root` 命令的 `PersistentPreRun` 而不是 `root` 命令的 `PersistentPostRun`：

```go
package main

import (
  "fmt"

  "github.com/spf13/cobra"
)

func main() {

  var rootCmd = &cobra.Command{
    Use:   "root [sub]",
    Short: "My root command",
    PersistentPreRun: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside rootCmd PersistentPreRun with args: %v\n", args)
    },
    PreRun: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside rootCmd PreRun with args: %v\n", args)
    },
    Run: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside rootCmd Run with args: %v\n", args)
    },
    PostRun: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside rootCmd PostRun with args: %v\n", args)
    },
    PersistentPostRun: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside rootCmd PersistentPostRun with args: %v\n", args)
    },
  }

  var subCmd = &cobra.Command{
    Use:   "sub [no options!]",
    Short: "My subcommand",
    PreRun: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside subCmd PreRun with args: %v\n", args)
    },
    Run: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside subCmd Run with args: %v\n", args)
    },
    PostRun: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside subCmd PostRun with args: %v\n", args)
    },
    PersistentPostRun: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside subCmd PersistentPostRun with args: %v\n", args)
    },
  }

  rootCmd.AddCommand(subCmd)

  rootCmd.SetArgs([]string{""})
  rootCmd.Execute()
  fmt.Println()
  rootCmd.SetArgs([]string{"sub", "arg1", "arg2"})
  rootCmd.Execute()
}
```

输出如下：

```
Inside rootCmd PersistentPreRun with args: []
Inside rootCmd PreRun with args: []
Inside rootCmd Run with args: []
Inside rootCmd PostRun with args: []
Inside rootCmd PersistentPostRun with args: []

Inside rootCmd PersistentPreRun with args: [arg1 arg2]
Inside subCmd PreRun with args: [arg1 arg2]
Inside subCmd Run with args: [arg1 arg2]
Inside subCmd PostRun with args: [arg1 arg2]
Inside subCmd PersistentPostRun with args: [arg1 arg2]
```

## 发生“未知命令”时的建议

当“未知命令”错误发生时，Cobra 将打印自动建议。这允许 Cobra 在出现拼写错误时的行为类似于 `git` 命令。例如：

```shell
$ hugo srever
Error: unknown command "srever" for "hugo"

Did you mean this?
        server

Run 'hugo --help' for usage.
```

建议是基于每个注册的子命令自动提出的，并使用 [Levenshtein distance](http://en.wikipedia.org/wiki/Levenshtein_distance) 的实现。每个匹配最小距离 2（忽略大小写）的已注册命令都将显示为建议。

如果您需要在命令中禁用建议或调整字符串距离，请使用：

```go
command.DisableSuggestions = true
```

或

```go
command.SuggestionsMinimumDistance = 1
```

您还可以使用 `SuggestFor` 属性显式设置将建议给定命令的名称。这允许为在字符串距离方面不接近的字符串提供建议，但在您的命令集和一些您不想要别名的命令中有意义。例子：

```shell
$ kubectl remove
Error: unknown command "remove" for "kubectl"

Did you mean this?
        delete

Run 'kubectl help' for usage.
```

## 为您的命令生成文档

Cobra 可以根据子命令、标志等生成以下格式的文档：

- [Markdown](https://cobra.dev/doc/md_docs.md)
- [ReStructured Text](https://cobra.dev/doc/rest_docs.md)
- [Man Page](https://cobra.dev/doc/man_docs.md)

## 生成 bash 完成文件

Cobra 可以生成一个 `bash` 完成文件。如果您向命令中添加更多信息，这些补全会非常强大和灵活。在 [Bash Completions](https://cobra.dev/bash_completions.md)中阅读更多关于它的信息。

