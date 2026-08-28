---
title: 'Cobra for generating CLI tools'
date: '2026-03-30'
tags: ['golang']
---

This is my featured project for this month, perhaps even for this year.
I'm surprised I hadn't seen it before; it's
[spf13/cobra](https://github.com/spf13/cobra). I say it's surprising that I
didn't know about it sooner because it appears among the contributors to
[gohugoio/hugo](https://github.com/gohugoio/hugo), one of the best projects in
the `golang` world in my opinion, which I use daily, for example, to create
this very blog, and which also uses several libraries created by
[spf13](https://github.com/spf13).

## What is Cobra?

As its GitHub description says:

> A Commander for modern Go CLI interactions

Perhaps this doesn't say much until we start looking for solutions to make
handling the "tasks" of the command-line applications we write easier, so
I'll start by giving an example. Instead of starting a Go project as usual
and racking our brains over the entry point, error handling, documentation,
shell-dependent autocompletion, configuration files, and everything else
separately, we can delegate all these tasks to Cobra and Viper. For example,

I want to create an application for managing package managers, basically so
that I can configure them through a `.yaml` file and then run

- `pm sync` to synchronize the repositories.
- `pm update` to update the packages.

This would give me a way to update my entire system, toolchains, standalone
packages, and so on, all with a single command. Ideally, it would have flags
to choose a particular package manager, for example:

- `pm update -m pacman` to update the packages installed by pacman.
- `pm update -r flathub` or even packages installed from a "remote".

### Creating a project with `cobra` and `viper`

Let's see what this looks like in practice:

```go
go mod init github.com/tavocg/pm

// install cobra-cli with:
// go install github.com/spf13/cobra-cli@latest
cobra-cli init --author "Gustavo Calvo <tavo@tavo.cr>" --license GPLv3 --viper

// Add the options we want (they can also be added manually,
// but for now, to become familiar with the recommended structure, we'll do it
// this way).
cobra-cli add sync
cobra-cli add update
```

### Result

```
 .
├──  cmd
│   ├──  root.go
│   ├──  sync.go
│   └──  update.go
├──  go.mod
├──  go.sum
├──  LICENSE
└──  main.go
```

```sh
go run main.go
# A longer description that spans multiple lines and likely contains
# examples and usage of using your application. For example:
#
# Cobra is a CLI library for Go that empowers applications.
# This application is a tool to generate the needed files
# to quickly create a Cobra application.
#
# Usage:
#   pm [command]
#
# Available Commands:
#   completion  Generate the autocompletion script for the specified shell
#   help        Help about any command
#   sync        A brief description of your command
#   update      A brief description of your command
#
# Flags:
#       --config string   config file (default is $HOME/.pm.yaml)
#   -h, --help            help for pm
#   -t, --toggle          Help message for toggle
#
# Use "pm [command] --help" for more information about a command.

go run main.go sync
# sync called

go run main.go update
# update called
```

We now have a project structure where we can edit the files in `cmd/` to
customize them for each application, a help screen, the option to generate
an autocompletion script for any supported shell, and logic that looks for
our configuration files in whichever directories we want.

Let's make one modification. Personally, for configuration I prefer to look
in `${XDG_CONFIG_HOME:-$HOME/.config}/pm`, so:

```diff
diff --git a/cmd/root.go b/cmd/root.go
index b0bcc46..39b885d 100644
--- a/cmd/root.go
+++ b/cmd/root.go
@@ -17,8 +17,9 @@ along with this program. If not, see <http://www.gnu.org/licenses/>.
 package cmd
 
 import (
-	"fmt"
 	"os"
+	"path"
+	"slices"
 
 	"github.com/spf13/cobra"
 	"github.com/spf13/viper"
@@ -70,20 +71,36 @@ func initConfig() {
 		// Use config file from the flag.
 		viper.SetConfigFile(cfgFile)
 	} else {
-		// Find home directory.
-		home, err := os.UserHomeDir()
-		cobra.CheckErr(err)
+		// Find config directories.
+		for _, d := range findConfigDirs() {
+			viper.AddConfigPath(d)
+		}
 
-		// Search config in home directory with name ".pm" (without extension).
-		viper.AddConfigPath(home)
 		viper.SetConfigType("yaml")
-		viper.SetConfigName(".pm")
+		viper.SetConfigName("pm")
 	}
 
 	viper.AutomaticEnv() // read in environment variables that match
 
 	// If a config file is found, read it in.
-	if err := viper.ReadInConfig(); err == nil {
-		fmt.Fprintln(os.Stderr, "Using config file:", viper.ConfigFileUsed())
+	err := viper.ReadInConfig()
+	cobra.CheckErr(err)
+}
+
+func findConfigDirs() []string {
+	dirs := []string{}
+
+	if xdgCfgDir := os.Getenv("XDG_CONFIG_HOME"); xdgCfgDir != "" {
+		dirs = append(dirs, xdgCfgDir)
 	}
+
+	if home, err := os.UserHomeDir(); err == nil && home != "" {
+		if defCfgDir := path.Join(home, ".config"); !slices.Contains(dirs, defCfgDir) {
+			dirs = append(dirs, defCfgDir)
+		}
+	}
+
+	dirs = append(dirs, "/usr/local/etc", "/etc")
+
+	return dirs
 }
```

In short, we add:

```go
// ...
func findConfigDirs() []string {
	dirs := []string{}

	if xdgCfgDir := os.Getenv("XDG_CONFIG_HOME"); xdgCfgDir != "" {
		dirs = append(dirs, xdgCfgDir)
	}

	if home, err := os.UserHomeDir(); err == nil && home != "" {
		if defCfgDir := path.Join(home, ".config"); !slices.Contains(dirs, defCfgDir) {
			dirs = append(dirs, defCfgDir)
		}
	}

	dirs = append(dirs, "/usr/local/etc", "/etc")

	return dirs
}
// ...
```

And we slightly modify it so that it doesn't search only in our user
directory, but instead searches the most common configuration directories.

```go
// ...
for _, d := range findConfigDirs() {
  viper.AddConfigPath(d)
}
// ...
```


Very well, now we can define our program's logic in each of the files
generated in `cmd/`, each specifically for its purpose. Let's start with
`cmd/sync.go`.

```go
// ...
var syncCmd = &cobra.Command{
	Use:          "sync",
	Aliases:      []string{"s"},
	Short:        "Sync package managers",
	Long:         `sync fetches latest remote data.`,
	Run:          sync,
	SilenceUsage: true,
}

func init() {
	rootCmd.AddCommand(syncCmd)
}

func sync(cmd *cobra.Command, args []string) {
  // orch is an instance of a system orchestrator for carrying out
  // package-management tasks. To see the complete source code:
  // github.com/tavocg/pm
	if err := orch.SyncRemotes(); err != nil {
		os.Exit(1)
	}
}
// ...
```

This structure makes it possible to customize the terminal client in detail,
while keeping it very simple, even allowing us to generate autocompletion
scripts and man pages, all with a logic that makes operating the utility as
easy as writing prose.
