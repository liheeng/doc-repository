1. Install lua on Macos

> brew update
> brew install lua
Current lua version is 5.3

2. Install luarocks on Macos

> brew install luarocks

3. Debug lua in IntellijIdea

3.1 Install mobdebug
> luarocks install mobdebug
> luarocks --lua-dir=/usr/local/opt/lua@5.1 install modebug

3.2 Install luasocket which is required by mobdebug
> luarocks install luasocket
> luarocks --lua-dir=/usr/local/opt/lua@5.1 install luasocket

4. Debug with Openresty
   Since Openresty uses Luajit, but Luajit2.0 doesn't compatible with Lua5.3, it is just compatible with Lua5.1, follow statemetns should be added in top of lua file to make sure luastocket can be load.

package.path = "/Users/mac/.luarocks/share/lua/5.1/?.lua;/usr/local/opt/luarocks/share/lua/5.3/?.lua;" .. package.path
package.cpath = "/Users/mac/.luarocks/lib/lua/5.1/?.so;/usr/local/lib/lua/5.3/?.so;" .. package.cpath

5. Add follow statements in lua file to make sure mobdebug is started.
  --- 启动调试
  local mobdebug = require("mobdebug");
  mobdebug.start();


