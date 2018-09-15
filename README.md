# lua-mikrotik

A lightweight lua library for talking to the [Mikrotik RouterOS API](https://wiki.mikrotik.com/wiki/Manual:API).

## Requirements: 

* [luasocket](https://github.com/diegonehab/luasocket)
* Either the [kikito's md5 library](https://github.com/kikito/md5.lua) or the [keplerproject's md5 library](https://github.com/keplerproject/md5)
* On older Lua versions with no `bit32` (`<= 5.1`), either [BitOp](http://luaforge.net/projects/bit/), or [BitLib](https://github.com/LuaDist/bitlib)

## Examples

Starting a RouterOS script from `/system/script` by name:

```lua
local Mikrotik = require 'Mikrotik'

local function runScript(mt, scriptname)
    assert(mt:sendSentence({ '/system/script/print', '?name=' .. scriptname, '=.proplist=.id' }))
    local message = assert(mt:readSentence())
    assert(message.type == '!re')
    assert(mt:readSentence().type == '!done')

    local runTag = mt:nextTag()
    assert(mt:sendSentence({ '/system/script/run', '=number=' .. message['=.id'], '.tag=' .. runTag}))
    local scriptResult = assert(mt:readSentence())
    assert(scriptResult.type == '!done')
    assert(scriptResult['.tag'] == runTag)
end

local mt = assert(Mikrotik:create('192.168.88.1', 8728))
assert(mt:login('login', 'password'), 'Failed login')

runScript(mt, 'routeros-script-name')
```