# Glorytun

Glorytun is a small, simple and secure VPN over [mud](https://github.com/angt/mud).

## Compatibility

Glorytun only depends on [libsodium](https://github.com/jedisct1/libsodium) version >= 1.0.4.
Which can be installed on a wide variety of systems.
Linux is the platform of choice but the code is standard so it should be easily ported on other posix systems.
It was successfully tested on OpenBSD, FreeBSD and MacOS.

## Stability

The master branch is very unstable because it is used for dev and testing.
In any case, wait for a 1.0 if you want to use it in production.

## Features

The key features of Glorytun come directly from mud:

 * **Fast and highly secure**

   The use of UDP and [libsodium](https://github.com/jedisct1/libsodium) allows you to secure
   your communications without impacting performance.
   Glorytun uses AES only if AES-NI is available otherwise ChaCha20 is used.
   You can force the use of ChaCha20 for higher security.
   All messages are encrypted, authenticated and marked with a timestamp.
   Perfect forward secrecy is also implemented with ECDH over Curve25519.

 * **Multipath and active failover**

   This is the main feature of Glorytun that allows to build an SD-WAN like service.
   This allows a TCP connection to explore and exploit multiple links without being disconnected.

 * **Traffic shaping**

   Shaping is very important in network, it allows to keep a low latency without sacrificing the bandwidth.
   It also helps the multipath scheduler to make better decisions.
   Currently it must be configured by hand, but soon Glorytun will do it for you.

 * **Path MTU discovery without ICMP**

   Bad MTU configuration is a very common problem in the world of VPN.
   As it is critical, Glorytun will try to setup it correctly by guessing its value.
   It doesn't rely on ICMP Next-hop MTU to avoid black holes.
   In asymmetric situations the minimum MTU is selected.

## Build and Install

We recommend the use of [meson](http://mesonbuild.com) for building instead of
the more classical autotools suite (also available for old systems).

On Ubuntu, the following command should be sufficient to get all the necessary build dependencies:

    $ sudo apt-get install meson libsodium-dev pkg-config

To build and install the latest release from github:

    $ git clone https://github.com/angt/glorytun --recursive
    $ meson glorytun glorytun/build
    $ sudo ninja -C glorytun/build install

This will install all binaries in `/usr/local/bin` by default.

You can easily customize your setup with meson (see `meson help`).

## Usage

Just run `glorytun` with no arguments to view the list of available commands:

```
$ glorytun
available commands:

  show     show all running tunnels
  bench    start a crypto bench
  bind     start a new tunnel
  set      change tunnel properties
  sync     re-sync tunnels
  keygen   generate a new secret key
  path     manage paths
  version  show version

```

Use the keyword `help` after a command to show its usage.

## Mini HowTo

Glorytun does not touch the configuration of its network interface (except for the MTU),
It is up to the user to do it according to the tools available
on his system (systemd-networkd, netifd, ...).
This also allows a wide variety of configurations.

To start a server:

    # (umask 066; glorytun keygen > my_secret_key)
    # glorytun bind 0.0.0.0 keyfile my_secret_key &

You should now have an unconfigured network interface (let's say `tun0`).
For example, the simplest setup with `ifconfig`:

    # ifconfig tun0 10.0.1.1 pointopoint 10.0.1.2 up

To check if the server is running, simply call `glorytun show`.
It will show you all of the running tunnels.

To start a new client, you need to get the secret key generated for the server.
Then simply call:

    # glorytun bind 0.0.0.0 to SERVER_IP keyfile my_secret_key &
    # ifconfig tun0 10.0.1.2 pointopoint 10.0.1.1 up

Now you have to setup your path, let's say you have an ADSL link that can do 1Mbit upload and 20Mbit download then call:

    # glorytun path up LOCAL_IPADDR rate tx 125000 rx 2500000

Again, to check if your path is working, you can watch its status with `glorytun path`.
You should now be able to ping your server with `ping 10.0.1.1`.

If you use systemd-networkd, you can easily setup your tunnels with the helper program `glorytun-setup`.

## Thanks

 * @jedisct1 for all his help and the code for MacOS/BSD.
 * The team OTB (@bessa, @gregdel, @pouulet, @sduponch and @simon) for all tests and discussions.
 * OVH to support this soft :)

---

For feature requests and bug reports, please create an [issue](https://github.com/angt/glorytun/issues).
