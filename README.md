# gops

[![GitHub Action Status](https://github.com/google/gops/workflows/Tests/badge.svg)](https://github.com/google/gops/actions?query=workflow%3ATests)
[![Build status](https://circleci.com/gh/google/gops/tree/master.svg?style=shield&circle-token=2637dc1e57d5407ae250480a86a2e553a7d20482)](https://circleci.com/gh/google/gops)
[![GoDoc](https://godoc.org/github.com/google/gops?status.svg)](https://godoc.org/github.com/google/gops)

gops is a command to list and diagnose Go processes currently running on your system.

```sh
$ gops
983   980    uplink-soecks  go1.9   /usr/local/bin/uplink-soecks
52697 52695  gops           go1.10  /Users/jbd/bin/gops
4132  4130   foops        * go1.9   /Users/jbd/bin/foops
51130 51128  gocode         go1.9.2 /Users/jbd/bin/gocode
```

## Installation

To install the latest version of gops:

```sh
$ go get github.com/google/gops
```

or

```sh
$ go install github.com/google/gops@latest
```

To install a specific gops version, for example v0.3.19:

```sh
$ go install github.com/google/gops@v0.3.19
```

## Diagnostics

For processes that start the diagnostics agent, gops can report
additional information such as the current stack trace, Go version, memory
stats, etc.

In order to start the diagnostics agent, see the [hello example](https://github.com/google/gops/blob/master/examples/hello/main.go).

``` go
package main

import (
	"log"
	"time"

	"github.com/google/gops/agent"
)

func main() {
	if err := agent.Listen(agent.Options{}); err != nil {
		log.Fatal(err)
	}
	time.Sleep(time.Hour)
}
```

Otherwise, you could set `GOPS_CONFIG_DIR` environment variables to assign your config dir.
Default, gops will use the current user's home directory(AppData on windows).

### Manual

It is possible to use gops tool both in local and remote mode.

Local mode requires that you start the target binary as the same user that runs gops binary.
To use gops in a remote mode you need to know target's agent address.

In Local mode use process's PID as a target; in Remote mode target is a `host:port` combination.

#### Listing all processes running locally

To print all go processes, run `gops` without arguments:

```sh
$ gops
983   980    uplink-soecks  go1.9   /usr/local/bin/uplink-soecks
52697 52695  gops           go1.10  /Users/jbd/bin/gops
4132  4130   foops        * go1.9   /Users/jbd/bin/foops
51130 51128  gocode         go1.9.2 /Users/jbd/bin/gocode
```

The output displays:
* PID
* PPID
* Name of the program
* Go version used to build the program
* Location of the associated program

Note that processes running the agent are marked with `*` next to the PID (e.g. `4132*`).

#### $ gops \<pid\> [duration]

To report more information about a process, run `gops` followed by a PID:

```sh
$ gops <pid>
parent PID:	5985
threads:	27
memory usage:	0.199%
cpu usage:	0.139%
username:	jbd
cmd+args:	/Applications/Splice.app/Contents/Resources/Splice Helper.app/Contents/MacOS/Splice Helper -pid 5985
local/remote:	127.0.0.1:56765 <-> :0 (LISTEN)
local/remote:	127.0.0.1:56765 <-> 127.0.0.1:50955 (ESTABLISHED)
local/remote:	100.76.175.164:52353 <-> 54.241.191.232:443 (ESTABLISHED)
```

If an optional duration is specified in the format as expected by
[`time.ParseDuration`](https://golang.org/pkg/time/#ParseDuration), the CPU
usage for the given time period is reported in addition:

```sh
$ gops <pid> 2s
parent PID:	5985
threads:	27
memory usage:	0.199%
cpu usage:	0.139%
cpu usage (2s):	0.271%
username:	jbd
cmd+args:	/Applications/Splice.app/Contents/Resources/Splice Helper.app/Contents/MacOS/Splice Helper -pid 5985
local/remote:	127.0.0.1:56765 <-> :0 (LISTEN)
local/remote:	127.0.0.1:56765 <-> 127.0.0.1:50955 (ESTABLISHED)
local/remote:	100.76.175.164:52353 <-> 54.241.191.232:443 (ESTABLISHED)
```

#### $ gops tree

To display a process tree with all the running Go processes, run the following command:

```sh
$ gops tree

...
├── 1
│   └── 13962 [gocode] {go1.9}
├── 557
│   └── 635 [com.docker.supervisor] {go1.9.2}
│       └── 638 [com.docker.driver.amd64-linux] {go1.9.2}
└── 13744
    └── 67243 [gops] {go1.10}
```

#### $ gops stack (\<pid\>|\<addr\>)

In order to print the current stack trace from a target program, run the following command:


```sh
$ gops stack (<pid>|<addr>)
gops stack 85709
goroutine 8 [running]:
runtime/pprof.writeGoroutineStacks(0x13c7bc0, 0xc42000e008, 0xc420ec8520, 0xc420ec8520)
	/Users/jbd/go/src/runtime/pprof/pprof.go:603 +0x79
runtime/pprof.writeGoroutine(0x13c7bc0, 0xc42000e008, 0x2, 0xc428f1c048, 0xc420ec8608)
	/Users/jbd/go/src/runtime/pprof/pprof.go:592 +0x44
runtime/pprof.(*Profile).WriteTo(0x13eeda0, 0x13c7bc0, 0xc42000e008, 0x2, 0xc42000e008, 0x0)
	/Users/jbd/go/src/runtime/pprof/pprof.go:302 +0x3b5
github.com/google/gops/agent.handle(0x13cd560, 0xc42000e008, 0xc420186000, 0x1, 0x1, 0x0, 0x0)
	/Users/jbd/src/github.com/google/gops/agent/agent.go:150 +0x1b3
github.com/google/gops/agent.listen()
	/Users/jbd/src/github.com/google/gops/agent/agent.go:113 +0x2b2
created by github.com/google/gops/agent.Listen
	/Users/jbd/src/github.com/google/gops/agent/agent.go:94 +0x480
# ...
```

#### $ gops memstats (\<pid\>|\<addr\>)

To print the current memory stats, run the following command:

```sh
$ gops memstats (<pid>|<addr>)
```

#### $ gops gc (\<pid\>|\<addr\>)

If you want to force run garbage collection on the target program, run `gc`.
It will block until the GC is completed.

#### $ gops setgc (\<pid\>|\<addr\>) <perc>

Sets the garbage collection target to a certain percentage.
The following command sets it to 10%:

``` sh
$ gops setgc (<pid>|<addr>) 10
```

#### $ gops version (\<pid\>|\<addr\>)

gops reports the Go version the target program is built with, if you run the following:

```sh
$ gops version (<pid>|<addr>)
devel +6a3c6c0 Sat Jan 14 05:57:07 2017 +0000
```

#### $ gops stats (\<pid\>|\<addr\>)

To print the runtime statistics such as number of goroutines and `GOMAXPROCS`.

#### Profiling


##### Pprof

gops supports CPU and heap pprof profiles. After reading either heap or CPU profile,
it shells out to the `go tool pprof` and let you interactively examine the profiles.

To enter the CPU profile, run:

```sh
$ gops pprof-cpu (<pid>|<addr>)
```

To enter the heap profile, run:

```sh
$ gops pprof-heap (<pid>|<addr>)
```

##### Execution trace

gops allows you to start the runtime tracer for 5 seconds and examine the results.

```sh
$ gops trace (<pid>|<addr>)
```
