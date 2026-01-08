local a
a = hookmetamethod(游戏，“__namecall”，函数(self，...)
method = getnamecallmethod():lower()
如果自我. Name = = " ForceSelfDamage "和method == "fireserver "
返回零
    end
返回一个(自我，...)
结束)

本地b
b = hookmetamethod(游戏，" __namecall "，newcclosure(函数(自我，...)
local method = getnamecallmethod()
本地参数= {...}
如果method = = " SetStateEnabled "且typeof(Self) == "Instance "且自我:IsA(“人形")则
本地状态=参数[1]
如果状态==枚举。类人状态类型.布娃娃或状态==枚举。类人状态类型.坠落下来或状态==枚举。那就自由落体吧
返回乙(自我,状态,假)
结束
结束
返回乙(自我,...)
end))

本地运行服务=游戏:获取服务(“运行服务”)
本地本地玩家=游戏:获取服务(“玩家”).本地播放器

运行服务渲染步进:连接(函数(增量时间)
如果本地播放器.那么性格呢
如果本地播放器。人物:findfirtschild("反摔/本土平板")然后
本地玩家。性格；角色；字母["反投掷/局部平台"]。禁用=真
本地玩家。性格；角色；字母["反投掷/局部平台"]:销毁()
结束
本地玩家。类人的:SetStateEnabled(枚举类人状态类型.FallingDown，false)
本地玩家。类人的:SetStateEnabled(枚举类人状态类型.自由落体,假)
本地玩家。类人的:SetStateEnabled(枚举类人状态类型.布娃娃,假)
结束
结束)
