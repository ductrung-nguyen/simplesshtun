# simplesshtun

[![GoDoc](https://godoc.org/github.com/golang/gddo?status.svg)](https://godoc.org/github.com/ductrung-nguyen/simplesshtun)

simplesshtun is a Go package that provides a SSH tunnel with port forwarding supporting:

- TCP and unix socket connections
- Password authentication
- Un/encrypted key file authentication
- `ssh-agent` based authentication

By default it reads the default linux ssh private key location `$HOME/.ssh/id_rsa` and fallbacks to using `ssh-agent`, but a specific authentication method can be set.

## Installation

`go get github.com/ductrung-nguyen/simplesshtun`

## Example

```go
package main

import (
    "log"
    "time"

    "github.com/ductrung-nguyen/simplesshtun"
)

func main() {
    // We want to connect to port 8080 on our machine to access port 80 on my.super.host.com
    simplesshtun := simplesshtun.New(8080, "my.super.host.com", 80)

    // We enable debug messages to see what happens
    simplesshtun.SetDebug(true)

    // We set a callback to know when the tunnel is ready
    simplesshtun.SetConnState(func(tun *simplesshtun.simplesshtun, state simplesshtun.ConnState) {
        switch state {
        case simplesshtun.StateStarting:
            log.Printf("STATE is Starting")
        case simplesshtun.StateStarted:
            log.Printf("STATE is Started")
        case simplesshtun.StateStopped:
            log.Printf("STATE is Stopped")
        }
    })

    // We start the tunnel (and restart it every time it is stopped)
    go func() {
        for {
            if err := simplesshtun.Start(); err != nil {
                log.Printf("SSH tunnel stopped: %s", err.Error())
                time.Sleep(time.Second) // don't flood if there's a start error :)
            }
        }
    }()

    // We stop the tunnel every 20 seconds (just to see what happens)
    for {
        time.Sleep(time.Second * time.Duration(20))
        log.Println("Lets stop the SSH tunnel...")
        simplesshtun.Stop()
    }
}
```
