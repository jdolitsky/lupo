<img align="right" src="lupo.png" width="140px" />

# lupo

`lupo` is a pre-processor for [Porter](https://porter.sh/) which allows you to build bundles with Lua.

## Install

*Note: system should already have a working Porter installation (see [install instructions](https://porter.sh/install/)). There must also be a `luac` executable in PATH (temporarily required by [Azure/golua](https://github.com/Azure/golua)).*

Just build from source and copy into PATH (requires git, make, and Go 1.11+):
```
(
  set -x && mkdir -p $GOPATH/src/github.com/jdolitsky/ && \
  cd $GOPATH/src/github.com/jdolitsky/ && \
  [ -d lupo/ ] || git clone git@github.com:jdolitsky/lupo.git && \
  cd lupo/ && make build && sudo mv bin/lupo /usr/local/bin/
)
```

## How to use

Simply use the `lupo` command in place of the `porter` command.

Replace your existing `porter.yaml` bundle definition with `porter.lua`.

Here is a simple `porter.lua` example (notice global `bundle` variable):
```lua
local name = "my-bundle"
local version = "0.1.0"
local description = "this application is extremely important"

-- Example of pushing to your personal Docker Hub account,
-- assuming USER env var matches your Docker Hub username
-- (make sure you create the "my-bundle" repo ahead of time)
local registryHost = "docker.io"
local registryRepo = os.getenv("USER") .. "/" .. name

-- Returns valid input for exec mixin
local function execEcho (desc, msg)
    return {description = desc, command = "bash", arguments = {"-c", "echo " .. msg}}
end

bundle = {
    name =  name,
    version = version,
    description = description,
    invocationImage = registryHost .. "/" .. registryRepo .. ":" .. version,
    mixins = {"exec"},
    install = {
        {
            exec = execEcho("Install " .. name, "Hello World")
        }
    },
    uninstall = {
        {
            exec = execEcho("Uninstall " .. name, "Goodbye World")
        }
    }
}
```

Run `lupo` to build the bundle from `porter.lua`:
```
$ lupo build
Copying dependencies ===>
Copying mixins ===>
Copying mixin exec ===>
Copying mixin porter ===>
...
```

If a file named `porter.lua` is detected in the working directory, `lupo` will attempt to use this to generate a `porter.yaml` file in the format expected by Porter, then run Porter itself.

Note: if there is an existing `porter.yaml`, it will be completely overwritten. You may even wish to place `porter.yaml` in your `.gitignore`, as it is dynamically generated each run.
