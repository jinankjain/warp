# README

```
  _      ______ __________ 
 | | /| / / __ `/ ___/ __ \
 | |/ |/ / /_/ / /  / /_/ /
 |__/|__/\__,_/_/  / .___/ 
                  /_/        v0.0.3
```

*Secure and simple terminal sharing*

`warp` lets you securely share your terminal with one simple command: `warp
open`. When connected to your warp, clients can see your terminal exactly as if
they were sitting next to you. You can also grant them write access, the
equivalent of handing them your keyboard.

`warp` distinguishes itself from "tmux/screen over ssh" by its focus and
ease of use as it does not require an SSH access to your machine or a
shared server for others to collaborate with you.

Despite being still quite experimental, `warp` has already proven itself useful
especially in the context of:

- Interaction with remote team-members
- New engineer onboarding (navigating code in group without projection)

## Installation

#### MacOSX (using Homebrew)

```shell
# Requires Homebrew installed. See https://brew.sh/

brew install warp
```

#### From source code

```shell
# Requires Go to be installed on your machine. You can easily install Go from
# https://golang.org/doc/install

go get -u github.com/spolu/warp/client/cmd/warp
```

In case of difficulties, please refer to 
[Troubleshooting your `warp` installation](https://github.com/spolu/warp/wiki/Troubleshooting-your-warp-installation).

## Usage

Instantly start sharing your terminal (read-only) under warp ID **goofy-dev**
with:

```shell
# You can name your warps however you want (here **goofy-dev**). In particular
# a cryptographically secure random ID will be generated for you if you don't
# specifiy a name.

$ warp open goofy-dev
```

This will create a new warp **goofy-dev** and will connect you to it locally
with write-access. From there, anyone can connect (read-only) to your warp
with:

```shell
$ warp connect goofy-dev
```

Creating a new warp spawns a new shell, and closing it is therefore as easy as
killing that shell with `exit` or `CTRL-D`.

#### Granting and revoking write-access

From inside a warp, retrieve the list of connected users with:
```shell
$ warp state
```

Grant write-access to a client (**be extra careful!** see the *Security*
section below):

```shell
$ warp authorize stan
```

Revoke previously granted write-access with:
```shell
$ warp revoke stan
```

## Security

`warp` is a powerful, and therefore, dangerous tool. Its misuse can potentially
enable an attacker to easily gain arbitrary remote code execution priviledges.

#### TLS connections

The connection between your host as well as your warp clients and the `warpd`
server are established over TLS, protecting you from man in the middle attacks.

#### Read-only by default

By default, warps are created read-only. Being protected by TLS does not
protect you from phishing. Be extra careful when running `warp authorize`.

#### IDs are secure and secret

Generated warp IDs are cryptographically secure and not publicized. If you want
to authorize someone to write to your warp, we recommend you use a generated
warp ID (to protect yourself against phishing attacks).

#### Trustless read-only

In particular, when your warp does not authorize anyone to write, it does not
trust the `warpd` daemon to enforce that noone other than you can write to it.
When at least one client is authorized to write, `warp` does trust the `warpd`
daemon it is connected to to enforce the read/write policy of clients.

## Roadmap

- [x] *v0.0.2 "bare"*
  - bare functionalities (see [TODO](TODO))
- [x] *v0.0.3 "safe"*
  - persisted user token/secret
  - graceful host reconnection
- [ ] *v0.0.4 "vt100"*
  - terminal emulation to achieve:
    - full redraw on connection
    - top status bar
    - terminal truncation
    - no resize required anymore
  - web-socket / HTTPS instead of raw sockets
- [ ] *v0.0.5 "cipher"*
  - e2e encryption based on warp ID
- [ ] *future releases*
  - `warp voice :warp` lets you voice-over a warp
  - warp signin and verified usernames

## Notes

#### `warp` is not a fork of tmux

`warp` is not a fork of tmux[0] and is not a terminal emulator (for now). It
really simply multiplexes stdin/stdout to raw ptys between host and clients.
For that reason, if you connect to a warp already running a GUI-like
application (tmux, vim, htop, ...) it might take time or host interactions for
the GUI-like application to visually reconstruct properly client-side.

In particular, since `warp` does not emulate the terminal it cannot reformat or
truncate the output of the host terminal to fit client terminal windows which
may lead to distorted outputs client side if the terminal sizes mismatch. To
mitigate that, `warp` relies on automatic client terminal resizing (pending
*v0.0.4*, see Roadmap).

#### Automatic client terminal resize

Once connected as a client and whenever the host terminal window size changes,
`warp` will attempt to resize your terminal window to the hosting tty size. For
that reason it is recommended to run `warp connect` from a new terminal
window[1].

#### Development of warp

Development of `warp` is generally broadcasted in **warp-dev**. Feel free to
connect at any time.

-- 

[0] You can run a warp from within tmux (or screen) or tmux from within a warp.
It's also fine to run a warp from within a warp.

[1] Terminals supporting window resizes based on the `\033[8;h;wt` ANSI escape
sequence:

| Terminal        | Support |
| --------------- |:-------:|
| MacOSX Terminal | Y       |
| XTerm           | Y       |
| Hyper           | N       |
| iTerm2          | N       |


