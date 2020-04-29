---
title: Lua print_r json
categories: lua
tag: hide
date: 2020-03-16 19:38:49
tags:
---

Lua print 模块，用于格式化打印json

```
require("print_r")
function tabprint(tab)
	function pairsBySort(_t, func)
	    local a = {}
	    for n in pairs(_t) do a[#a + 1] = n end
	    table.sort(a, func)
	    local i = 0
	    return function()
	        i = i + 1
	        return a[i], _t[a[i]]
	    end
	end
	
	function sortFunc(a, b)
	    return restable[a] > restable[b]
	end
	local restable_ret = {}
	for i,v in pairsBySort(restable, sortFunc) do
	    restable_ret[#restable_ret + 1] = v
	end
	for k,v in ipairs(restable_ret) do
		for i,n in pairs(restable) do
			if n == v then
				print(i,'->',v)
			end
		end
	end
end

function is_include(key, tab)
    --print('key:',key)
    if tab[key] == nil then
      return false
    else
      return true
    end
end

function Split(szFullString, szSeparator)  
local nFindStartIndex = 1  
local nSplitIndex = 1  
local nSplitArray = {}  
while true do  
   local nFindLastIndex = string.find(szFullString, szSeparator, nFindStartIndex)  
   if not nFindLastIndex then  
    nSplitArray[nSplitIndex] = string.sub(szFullString, nFindStartIndex, string.len(szFullString))  
    break  
   end  
   nSplitArray[nSplitIndex] = string.sub(szFullString, nFindStartIndex, nFindLastIndex - 1)  
   nFindStartIndex = nFindLastIndex + string.len(szSeparator)  
   nSplitIndex = nSplitIndex + 1  
end  
return nSplitArray  
end

function isempty(s)
  return s == nil or s == ''
end

function readFile(file1)--创建读取文件函数 
file1 = io.open(file1)
assert(file1,"file1 open failed-文件打开失败")--如果文件不存在，则提示：文件打开失败 
    local fileTab = {}--创建一个局部变量表 

 
    local line = file1:read()--读取文件中的单行内容存为另一个变量 
     
    while line do   --当读取一行内容为真时 
        --print(string.format("%s",line))
	local tmplist = Split(line,'|')
	local tmpcutcul = tmplist[8]
	if not isempty(tmpcutcul) then
		--print('aaaaaaaaaaaaaaaa',tmpcutcul)
		--cutcul = tring.match(cutcul,"[0-9]")
		cutcul = string.gsub(tmpcutcul,"(.*):(.*):(.*)",'%1')
		--print(cutcul)
		--os.exit()
	end
	--print(cutcul)
	if (is_include(cutcul,fileTab)) then
		fileTab[cutcul] = fileTab[cutcul] + 1
	else
		fileTab[cutcul] = 1

	end	
	--for col=1,#tmplist do
	--   print(tmplist[col])
	--end
        --table.insert(fileTab,line)--在fileTab表末尾插入读取line内容 
        line = file1:read()--读取下一行内容 
        --notifyMessage(string.format("%s",line))             
    end     
    return fileTab--内容读取完毕，返回表     
end     
restable = readFile(arg[1])
--restable = readFile('pc.log')
tabprint(restable)
```
