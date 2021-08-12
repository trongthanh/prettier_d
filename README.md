# \@trongthanh/prettier_d  [![npm package](https://img.shields.io/npm/v/@trongthanh/prettier_d?color=yellow)](https://www.npmjs.com/package/@trongthanh/prettier_d)

Makes [prettier][] fast.

**Forked and improved from `prettier_d_slim`** but with a slight change to default params.

## "But prettier is pretty fast already, right?"

Yes, it's really fast. But the node.js startup time and loading all the
required modules slows down linting times for a single file to ~700
milliseconds. `prettier_d` reduces this overhead by running a server in the
background. It brings the formatting time down to ~130 milliseconds. If you want
to format from within your editor whenever you save a file, `prettier_d` is for
you.

## Install

This will install the `@trongthanh/prettier_d` command globally:

```bash
$ npm install -g @trongthanh/prettier_d
```

This will create a global command `prettier_d`

## Usage

To start the server and lint a file, just run:

```bash
# Prettier needs to know the file name to do its thing (input default from stdin)
$ cat file.js | prettier_d --stdin-filepath file.js
# Or if you already have the content of the file:
$ prettier_d --stdin-filepath file.js --text 'const foo = {}'
```

On the initial call, the `prettier_d` server is launched and then the given file
is formatted. Subsequent invocations are super fast.

## HOWTO: Integrate with Sublime Text

The whole reason I forked and improved this package is because I need it to work with Sublime Text

- Install `@trongthanh/prettier_d`.
- Check where the `prettier_d` executable is with `$ which prettier_d`
  + For macOS with Home Brew Node.js & npm, it locates at `/usr/local/bin/prettier_d`
  + For Ubuntu* with apt-get Node.js & npm, it locates at `/usr/bin/prettier_d`
- In Sublime Text, install `JsPrettier` via Package Control
- Open JsPrettier Settings - User, scroll down to (or add to if not exist) setting entry `"prettier_cli_path"`.
- Set the absolute path to `prettier_d` executable discovered from above step.
- Sublime Text should now use `prettier_d` to format prettier's supported file types and it's much faster than default `prettier` cli.

## How does this work?

The first time you use `prettier_d`, a little server is started in the background
and bound to a random port. The port number is stored along with [a
token][change401] in `~/.prettier_d`. You can then run `prettier_d` commands the
same way you would use `prettier` and it will delegate to the background server.
It will load a [separate instance][change220] of prettier for each working
directory to make sure settings are kept local. If prettier is found in the
current working directories `node_modules` folder, then this version of prettier
is going to be used. Otherwise, the version of prettier that ships with
`prettier_d` is used as a fallback.

To keep the memory footprint low, `prettier_d` keeps only the last 10 used
instances in the internal [nanolru][] cache.

## Which versions of prettier are supported?

As far as I'm aware, all of them.

## Commands

Control the server like this:

```bash
$ prettier_d <command>
```

Available commands:

- `start`: start the server
- `stop`: stop the server
- `status`: print out whether the server is currently running
- `restart`: restart the server
- `[options] file.js [file.js] [dir]`: invoke `prettier` with the given options.
  The `prettier` engine will be created in the current directory. If the server
  is not yet running, it is started.

Type `prettier_d --help` to see the supported `prettier` options.

`prettier_d` will select a free port automatically and store the port number
along with an access token in `~/.prettier_d`.

## Moar speed

If you're really into performance and want the lowest possible latency, talk to
the `prettier_d` server with netcat. This will also eliminate the node.js startup
time.

```bash
$ PORT=`cat ~/.prettier_d | cut -d" " -f1`
$ TOKEN=`cat ~/.prettier_d | cut -d" " -f2`
$ echo "$TOKEN $PWD file.js" | nc localhost $PORT
```

Or if you want to work with stdin:

```bash
$ echo "$TOKEN $PWD --stdin" | cat - file.js | nc localhost $PORT
```

This runs `prettier` in under `50ms`!

## References

If you're interested in building something similar to this: Most of the logic
was extracted to [core_d][], a library that manages the background server.

## Compatibility

- `1.1.0`: prettier ^2.3.2
- `1.0.0`: prettier ^1.19.1

## License

MIT

[prettier]: https://prettier.io/
[nanolru]: https://github.com/s3ththompson/nanolru
[core_d]: https://github.com/mantoni/core_d.js
