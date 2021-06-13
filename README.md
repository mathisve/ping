# go-ping
[![PkgGoDev](https://pkg.go.dev/badge/github.com/go-ping/ping)](https://pkg.go.dev/github.com/mathisve/ping)

# Don't use this, this is just a fork because the original repo was broken!

A simple but powerful ICMP echo (ping) library for Go, inspired by
[go-fastping](https://github.com/tatsushid/go-fastping).

Here is a very simple example that sends and receives three packets:

```go
pinger, err := ping.NewPinger("www.google.com")
if err != nil {
	panic(err)
}
pinger.Count = 3
err = pinger.Run() // Blocks until finished.
if err != nil {
	panic(err)
}
stats := pinger.Statistics() // get send/receive/duplicate/rtt stats
```

Here is an example that emulates the traditional UNIX ping command:

```go
pinger, err := ping.NewPinger("www.google.com")
if err != nil {
	panic(err)
}

// Listen for Ctrl-C.
c := make(chan os.Signal, 1)
signal.Notify(c, os.Interrupt)
go func() {
	for _ = range c {
		pinger.Stop()
	}
}()

pinger.OnRecv = func(pkt *ping.Packet) {
	fmt.Printf("%d bytes from %s: icmp_seq=%d time=%v\n",
		pkt.Nbytes, pkt.IPAddr, pkt.Seq, pkt.Rtt)
}

pinger.OnDuplicateRecv = func(pkt *ping.Packet) {
	fmt.Printf("%d bytes from %s: icmp_seq=%d time=%v ttl=%v (DUP!)\n",
		pkt.Nbytes, pkt.IPAddr, pkt.Seq, pkt.Rtt, pkt.Ttl)
}

pinger.OnFinish = func(stats *ping.Statistics) {
	fmt.Printf("\n--- %s ping statistics ---\n", stats.Addr)
	fmt.Printf("%d packets transmitted, %d packets received, %v%% packet loss\n",
		stats.PacketsSent, stats.PacketsRecv, stats.PacketLoss)
	fmt.Printf("round-trip min/avg/max/stddev = %v/%v/%v/%v\n",
		stats.MinRtt, stats.AvgRtt, stats.MaxRtt, stats.StdDevRtt)
}

fmt.Printf("PING %s (%s):\n", pinger.Addr(), pinger.IPAddr())
err = pinger.Run()
if err != nil {
	panic(err)
}
```

It sends ICMP Echo Request packet(s) and waits for an Echo Reply in
response. If it receives a response, it calls the `OnRecv` callback
unless a packet with that sequence number has already been received,
in which case it calls the `OnDuplicateRecv` callback. When it's
finished, it calls the `OnFinish` callback.

For a full ping example, see
[cmd/ping/ping.go](https://github.com/go-ping/ping/blob/master/cmd/ping/ping.go).

## Installation

```
go get -u github.com/go-ping/ping
```

To install the native Go ping executable:

```bash
go get -u github.com/go-ping/ping/...
$GOPATH/bin/ping
```

## Supported Operating Systems

### Linux
This library attempts to send an "unprivileged" ping via UDP. On Linux,
this must be enabled with the following sysctl command:

```
sudo sysctl -w net.ipv4.ping_group_range="0 2147483647"
```

If you do not wish to do this, you can call `pinger.SetPrivileged(true)`
in your code and then use setcap on your binary to allow it to bind to
raw sockets (or just run it as root):

```
setcap cap_net_raw=+ep /path/to/your/compiled/binary
```

See [this blog](https://sturmflut.github.io/linux/ubuntu/2015/01/17/unprivileged-icmp-sockets-on-linux/)
and the Go [x/net/icmp](https://godoc.org/golang.org/x/net/icmp) package
for more details.

## Contributing

Refer to [CONTRIBUTING.md](https://github.com/mathisve/ping/blob/master/CONTRIBUTING.md)
