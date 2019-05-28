1. Install lua on Macos

> brew update
> brew install lua
Current lua version is 5.3

2. Install luarocks on Macos

> brew install luarocks

3. Debug lua in IntellijIdea

3.1 Install mobdebug
> luarocks install mobdebug

3.2 Install luasocket which is required by mobdebug
> luarocks install luasocket

The install path of luasocket is /usr/local/share/lua/5.3/luasocket_3_0rc1_2-socket.
Add symbol link for luasocket_3_0rc1_2-socket
> ln -s /usr/local/share/lua/5.3/luasocket_3_0rc1_2-socket.lua socket.lua
> ln -s /usr/local/share/lua/5.3/luasocket_3_0rc1_2-socket.lua socket

