---
title: 'Cobra para generar herramientas de CLI'
date: '2026-03-30'
tags: ['golang']
---

Este es mi proyecto destacado de este mes, tal vez de este año.
Me sorprende no haberlo visto antes, se trata de
[spf13/cobra](https://github.com/spf13/cobra). Digo que me sorprende no
conocerlo desde antes porque aparece entre los colaboradores de
[gohugoio/hugo](https://github.com/gohugoio/hugo), uno de los mejores proyectos
en mi opinión en el mundo de `golang` y que utilizo a diario, por ejemplo para
crear este mismo blog, y que además utilizan varias librerías creadas por
[spf13](https://github.com/spf13).

## ¿Qué es cobra?

Como lo dice su descripción en GitHub:

> A Commander for modern Go CLI interactions

Tal vez esto no dice mucho hasta que empezamos a buscar soluciones para hacer
más sencillo el manejo de "tareas" de las aplicaciones de línea de comando que
escribimos, empezaré ejemplificando eso. En lugar de empezar un proyecto de Go
como siempre, y romperse la cabeza pensando en el punto de entrada, manejo de
errores, documentación, autocompletado dependiendo de la shell, archivos de
configuración, todo por aparte, podemos delegar todas estas tareas a cobra y
viper. Por ejemplo,

Quiero crear una aplicación para manejar gestores de paquetes, básicamente
para poder configurarlos a través de un archivo .yaml y luego ejecutar

- `pm sync` para sincronizar los repositorios.
- `pm update` para actualizar los paquetes.

Entonces tener una forma de actualizar todo mi sistema, toolchains, paquetes
aislados, etc, todo en un comando. Idealmente tendría modificadores para poder
escoger un gestor de paquetes en particular, por ejemplo:

- `pm update -m pacman` para actualizar los paquetes instalados por pacman.
- `pm update -r flathub` o incluso instalados por un "remote".

### Creando un proyecto con `cobra` y `viper`

Veámos cómo se ve en la práctica:

```go
go mod init github.com/tavocg/pm

// instalar cobra-cli con:
// go install github.com/spf13/cobra-cli@latest
cobra-cli init --author "Gustavo Calvo <tavo@tavo.cr>" --license GPLv3 --viper

// Agregamos las opciones que queremos (se pueden agregar manualmente también,
// pero por ahora para familiarizarnos con la estructura recomendada lo hacemos
// de esta forma).
cobra-cli add sync
cobra-cli add update
```

### Resultado

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

Tenemos una estructura de proyecto en la que podemos editar los archivos en
`cmd/` para customizarlos dependiendo de cada aplicación, una pantalla de ayuda,
podemos opcionalmente generar un script de autocompletado para cualquiera de las
shells con soporte, y la lógica que va a buscar nuestros archivos de
configuración en los directorios que gustemos.

Vamos a hacer una modificación, para la configuración personalmente prefiero
buscar en `${XDG_CONFIG_HOME:-$HOME/.config}/pm`, entonces:

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

En resumen, agregamos:

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

Y modificamos ligeramente, para no buscar únicamente en nuestro directorio de
usuario, sino en los directorios de configuración más comunes.

```go
// ...
for _, d := range findConfigDirs() {
  viper.AddConfigPath(d)
}
// ...
```


Muy bien, ahora, podemos defininr la lógica de nuestro programa en cada uno de
los archivos generados en `cmd/` específicamente para su propósito. Empecemos
por `cmd/sync.go`.

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
  // orch es una instancia de un orquestador del sistema para llevar a cabo
  // tareas de gestión de paquetes. Para ver el código fuente completo:
  // github.com/tavocg/pm
	if err := orch.SyncRemotes(); err != nil {
		os.Exit(1)
	}
}
// ...
```

Esta estructura permite customizar a detalle, pero de manera muy sencilla al
cliente de la terminal, incluso permitiendo generar scripts de autocompletado,
manpages, todo bajo una lógica que hace la operación de la utilidad tan fácil
como escribir prosa.
