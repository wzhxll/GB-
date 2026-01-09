local Players = game:GetService("Players")
 local RunService = game:GetService("RunService")
 local UserInputService = game:GetService("UserInputService")
 local LocalPlayer = Players.LocalPlayer
 local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
 -- UI颜色配置（匹配图片风格）
 local UI_BG = Color3.fromRGB(46, 46, 46) -- 深黑背景
 local BTN_OFF = Color3.fromRGB(220, 0, 0) -- 功能关-红色
 local BTN_ON = Color3.fromRGB(0, 200, 0) -- 功能开-绿色
 local DESTROY_BTN = Color3.fromRGB(140, 0, 200) -- 销毁UI-紫色
 local TEXT_COLOR = Color3.fromRGB(255, 255, 255) -- 白色文字
 local BORDER_COLOR = Color3.fromRGB(0, 0, 0) -- 黑色边框
 local SPEED_BTN_COLOR = Color3.fromRGB(60, 60, 60) -- 速度按钮背景色
 -- 飞行功能变量
 local speeds = 1
 local nowe = false
 local tis
 local dis
 local tpwalking = false
 local SPEED_LIMIT = 10
 local speedLabel -- 速度标签（显示当前速度）
 -- 防护功能变量 拆分为独立开关
 local isAntiSelfDamage = false -- 防自伤
 local isAntiKnockback = false -- 防击飞
 local isAntiRagdoll = false -- 防布娃娃
 local namecallHook1 = nil
 local namecallHook2 = nil
 local renderConnection = nil
 -- 主UI引用
 local mainUI
 -- 构建整合UI界面
 local function buildUI()
     mainUI = Instance.new("ScreenGui")
     mainUI.Name = "FlyAndProtectUI"
     mainUI.Parent = PlayerGui
     mainUI.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
     mainUI.ResetOnSpawn = false
     -- 主框架（高度调整为280，刚好包裹所有控件，消除下方空白）
     local Frame = Instance.new("Frame")
     Frame.Parent = mainUI
     Frame.BackgroundColor3 = UI_BG
     Frame.BackgroundTransparency = 0
     Frame.BorderColor3 = BORDER_COLOR
     Frame.BorderSizePixel = 2
     Frame.Position = UDim2.new(0.1, 0, 0.3, 0)
     Frame.Size = UDim2.new(0, 180, 0, 280) -- 核心修改：高度从320→280，贴合控件
     Frame.Active = true
     Frame.Draggable = true
     -- 主框架圆角
     local UICorner = Instance.new("UICorner")
     UICorner.CornerRadius = UDim.new(0.08, 0)
     UICorner.Parent = Frame
     -- 标题标签（缩小高度，紧凑排列）
     local TitleLabel = Instance.new("TextLabel")
     TitleLabel.Parent = Frame
     TitleLabel.BackgroundTransparency = 1
     TitleLabel.Position = UDim2.new(0, 0, 0.01, 0)
     TitleLabel.Size = UDim2.new(1, 0, 0.12, 0)
     TitleLabel.Font = Enum.Font.SourceSansBold
     TitleLabel.Text = "飞行&防护控制"
     TitleLabel.TextColor3 = TEXT_COLOR
     TitleLabel.TextScaled = true
     -- 飞行开关按钮（缩小高度+上移）
     local FlyBtn = Instance.new("TextButton")
     FlyBtn.Name = "FlyToggle"
     FlyBtn.Parent = Frame
     FlyBtn.BackgroundColor3 = BTN_OFF
     FlyBtn.BorderColor3 = BORDER_COLOR
     FlyBtn.BorderSizePixel = 1
     FlyBtn.Position = UDim2.new(0.05, 0, 0.14, 0)
     FlyBtn.Size = UDim2.new(0.9, 0, 0.12, 0)
     FlyBtn.Font = Enum.Font.SourceSansBold
     FlyBtn.Text = "飞行关"
     FlyBtn.TextColor3 = TEXT_COLOR
     FlyBtn.TextScaled = true
     Instance.new("UICorner", FlyBtn).CornerRadius = UDim.new(0.05, 0)
     -- 飞行提示文字（上移+缩小高度）
     local FlyTipLabel = Instance.new("TextLabel")
     FlyTipLabel.Parent = Frame
     FlyTipLabel.BackgroundTransparency = 1
     FlyTipLabel.Position = UDim2.new(0.05, 0, 0.27, 0)
     FlyTipLabel.Size = UDim2.new(0.9, 0, 0.04, 0)
     FlyTipLabel.Font = Enum.Font.SourceSansBold
     FlyTipLabel.Text = "可调节飞行速度"
     FlyTipLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
     FlyTipLabel.TextScaled = true
     -- 飞行速度调节容器（上移+缩小高度）
     local SpeedControlFrame = Instance.new("Frame")
     SpeedControlFrame.Parent = Frame
     SpeedControlFrame.BackgroundColor3 = UI_BG
     SpeedControlFrame.BackgroundTransparency = 0
     SpeedControlFrame.BorderColor3 = BORDER_COLOR
     SpeedControlFrame.BorderSizePixel = 1
     SpeedControlFrame.Position = UDim2.new(0.05, 0, 0.32, 0)
     SpeedControlFrame.Size = UDim2.new(0.9, 0, 0.12, 0)
     Instance.new("UICorner", SpeedControlFrame).CornerRadius = UDim.new(0.05, 0)
     -- 速度-按钮（缩小高度）
     local SpeedMinusBtn = Instance.new("TextButton")
     SpeedMinusBtn.Name = "SpeedMinus"
     SpeedMinusBtn.Parent = SpeedControlFrame
     SpeedMinusBtn.BackgroundColor3 = SPEED_BTN_COLOR
     SpeedMinusBtn.BorderColor3 = BORDER_COLOR
     SpeedMinusBtn.BorderSizePixel = 1
     SpeedMinusBtn.Position = UDim2.new(0.02, 0, 0.05, 0)
     SpeedMinusBtn.Size = UDim2.new(0.25, 0, 0.9, 0)
     SpeedMinusBtn.Font = Enum.Font.SourceSansBold
     SpeedMinusBtn.Text = "-"
     SpeedMinusBtn.TextColor3 = TEXT_COLOR
     SpeedMinusBtn.TextScaled = true
     Instance.new("UICorner", SpeedMinusBtn).CornerRadius = UDim.new(0.1, 0)
     -- 速度显示标签
     speedLabel = Instance.new("TextLabel")
     speedLabel.Name = "SpeedDisplay"
     speedLabel.Parent = SpeedControlFrame
     speedLabel.BackgroundTransparency = 1
     speedLabel.Position = UDim2.new(0.3, 0, 0, 0)
     speedLabel.Size = UDim2.new(0.4, 0, 1, 0)
     speedLabel.Font = Enum.Font.SourceSansBold
     speedLabel.Text = "速度: 1"
     speedLabel.TextColor3 = TEXT_COLOR
     speedLabel.TextScaled = true
     -- 速度+按钮（缩小高度）
     local SpeedPlusBtn = Instance.new("TextButton")
     SpeedPlusBtn.Name = "SpeedPlus"
     SpeedPlusBtn.Parent = SpeedControlFrame
     SpeedPlusBtn.BackgroundColor3 = SPEED_BTN_COLOR
     SpeedPlusBtn.BorderColor3 = BORDER_COLOR
     SpeedPlusBtn.BorderSizePixel = 1
     SpeedPlusBtn.Position = UDim2.new(0.73, 0, 0.05, 0)
     SpeedPlusBtn.Size = UDim2.new(0.25, 0, 0.9, 0)
     SpeedPlusBtn.Font = Enum.Font.SourceSansBold
     SpeedPlusBtn.Text = "+"
     SpeedPlusBtn.TextColor3 = TEXT_COLOR
     SpeedPlusBtn.TextScaled = true
     Instance.new("UICorner", SpeedPlusBtn).CornerRadius = UDim.new(0.1, 0)
     -- 防自伤按钮（上移+缩小高度）
     local AntiSelfDamageBtn = Instance.new("TextButton")
     AntiSelfDamageBtn.Name = "AntiSelfDamageBtn"
     AntiSelfDamageBtn.Parent = Frame
     AntiSelfDamageBtn.BackgroundColor3 = BTN_OFF
     AntiSelfDamageBtn.BorderColor3 = BORDER_COLOR
     AntiSelfDamageBtn.BorderSizePixel = 1
     AntiSelfDamageBtn.Position = UDim2.new(0.05, 0, 0.45, 0)
     AntiSelfDamageBtn.Size = UDim2.new(0.9, 0, 0.1, 0)
     AntiSelfDamageBtn.Font = Enum.Font.SourceSansBold
     AntiSelfDamageBtn.Text = "防自伤 [关]"
     AntiSelfDamageBtn.TextColor3 = TEXT_COLOR
     AntiSelfDamageBtn.TextScaled = true
     Instance.new("UICorner", AntiSelfDamageBtn).CornerRadius = UDim.new(0.05, 0)
     -- 防击飞按钮（上移+缩小高度）
     local AntiKnockbackBtn = Instance.new("TextButton")
     AntiKnockbackBtn.Name = "AntiKnockbackBtn"
     AntiKnockbackBtn.Parent = Frame
     AntiKnockbackBtn.BackgroundColor3 = BTN_OFF
     AntiKnockbackBtn.BorderColor3 = BORDER_COLOR
     AntiKnockbackBtn.BorderSizePixel = 1
     AntiKnockbackBtn.Position = UDim2.new(0.05, 0, 0.56, 0)
     AntiKnockbackBtn.Size = UDim2.new(0.9, 0, 0.1, 0)
     AntiKnockbackBtn.Font = Enum.Font.SourceSansBold
     AntiKnockbackBtn.Text = "防击飞 [关]"
     AntiKnockbackBtn.TextColor3 = TEXT_COLOR
     AntiKnockbackBtn.TextScaled = true
     Instance.new("UICorner", AntiKnockbackBtn).CornerRadius = UDim.new(0.05, 0)
     -- 防布娃娃按钮（上移+缩小高度）
     local AntiRagdollBtn = Instance.new("TextButton")
     AntiRagdollBtn.Name = "AntiRagdollBtn"
     AntiRagdollBtn.Parent = Frame
     AntiRagdollBtn.BackgroundColor3 = BTN_OFF
     AntiRagdollBtn.BorderColor3 = BORDER_COLOR
     AntiRagdollBtn.BorderSizePixel = 1
     AntiRagdollBtn.Position = UDim2.new(0.05, 0, 0.67, 0)
     AntiRagdollBtn.Size = UDim2.new(0.9, 0, 0.1, 0)
     AntiRagdollBtn.Font = Enum.Font.SourceSansBold
     AntiRagdollBtn.Text = "防布娃娃 [关]"
     AntiRagdollBtn.TextColor3 = TEXT_COLOR
     AntiRagdollBtn.TextScaled = true
     Instance.new("UICorner", AntiRagdollBtn).CornerRadius = UDim.new(0.05, 0)
     -- 销毁UI按钮（位置适配新高度，无遮挡）
     local DestroyBtn = Instance.new("TextButton")
     DestroyBtn.Name = "DestroyUI"
     DestroyBtn.Parent = Frame
     DestroyBtn.BackgroundColor3 = DESTROY_BTN
     DestroyBtn.BorderColor3 = BORDER_COLOR
     DestroyBtn.BorderSizePixel = 1
     DestroyBtn.Position = UDim2.new(0.05, 0, 0.78, 0) -- 微调位置适配280高度
     DestroyBtn.Size = UDim2.new(0.9, 0, 0.08, 0)
     DestroyBtn.Font = Enum.Font.SourceSansBold
     DestroyBtn.Text = "销毁UI"
     DestroyBtn.TextColor3 = TEXT_COLOR
     DestroyBtn.TextScaled = true
     Instance.new("UICorner", DestroyBtn).CornerRadius = UDim.new(0.05, 0)
     -- 初始化提示
     game:GetService("StarterGui"):SetCore("SendNotification", {
         Title = "整合控制工具";
         Text = "UI加载成功！新增速度调节按钮";
         Icon = "rbxthumb://type=Asset&id=5107182114&w=150&h=150"
     })
     -- 飞行速度更新函数
     local function updateSpeedDisplay()
         speedLabel.Text = "速度: " .. tostring(speeds)
         -- 速度达到上限时文字变红
         if speeds >= SPEED_LIMIT then
             speedLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
         else
             speedLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
         end
         -- 飞行中更新移动逻辑
         if nowe then
             tpwalking = false
             for i = 1, speeds do
                 spawn(function()
                     local hb = RunService.Heartbeat
                     while tpwalking and hb:Wait() do
                         local chr = LocalPlayer.Character
                         local hum = chr and chr:FindFirstChildWhichIsA("Humanoid")
                         if chr and hum and hum.Parent then
                             if hum.MoveDirection.Magnitude > 0 then
                                 chr:TranslateBy(hum.MoveDirection)
                             end
                         end
                     end
                 end)
             end
         end
     end
     -- 飞行功能核心逻辑
     local function toggleFly()
         nowe = not nowe
         local chr = LocalPlayer.Character
         local hum = chr and chr:FindFirstChildWhichIsA("Humanoid")
         local humRootPart = chr and chr:FindFirstChild("HumanoidRootPart")
         if not hum or not humRootPart then return end
         -- 更新飞行按钮状态
         if nowe then
             FlyBtn.Text = "飞行开"
             FlyBtn.BackgroundColor3 = BTN_ON
             humRootPart.CanCollide = false
             tpwalking = true
             -- 初始化飞行速度对应的移动逻辑
             for i = 1, speeds do
                 spawn(function()
                     local hb = RunService.Heartbeat
                     while tpwalking and hb:Wait() do
                         local chr = LocalPlayer.Character
                         local hum = chr and chr:FindFirstChildWhichIsA("Humanoid")
                         if chr and hum and hum.Parent then
                             if hum.MoveDirection.Magnitude > 0 then
                                 chr:TranslateBy(hum.MoveDirection)
                             end
                         end
                     end
                 end)
             end
         else
             FlyBtn.Text = "飞行关"
             FlyBtn.BackgroundColor3 = BTN_OFF
             humRootPart.CanCollide = true
             tpwalking = false
         end
         -- 原有飞行状态控制逻辑
         if nowe then
             hum:SetStateEnabled(Enum.HumanoidStateType.Climbing,false)
             hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown,false)
             hum:SetStateEnabled(Enum.HumanoidStateType.Flying,false)
             hum:SetStateEnabled(Enum.HumanoidStateType.Freefall,false)
             hum:SetStateEnabled(Enum.HumanoidStateType.GettingUp,false)
             hum:SetStateEnabled(Enum.HumanoidStateType.Jumping,false)
             hum:SetStateEnabled(Enum.HumanoidStateType.Landed,false)
             hum:SetStateEnabled(Enum.HumanoidStateType.Physics,false)
             hum:SetStateEnabled(Enum.HumanoidStateType.PlatformStanding,false)
             hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll,false)
             hum:SetStateEnabled(Enum.HumanoidStateType.Running,false)
             hum:SetStateEnabled(Enum.HumanoidStateType.RunningNoPhysics,false)
             hum:SetStateEnabled(Enum.HumanoidStateType.Seated,false)
             hum:SetStateEnabled(Enum.HumanoidStateType.StrafingNoPhysics,false)
             hum:SetStateEnabled(Enum.HumanoidStateType.Swimming,false)
             hum:ChangeState(Enum.HumanoidStateType.Swimming)
             chr.Animate.Disabled = true
             for i,v in next, hum:GetPlayingAnimationTracks() do
                 v:AdjustSpeed(0)
             end
             local rootPart = hum.RigType == Enum.HumanoidRigType.R6 and chr.Torso or chr.UpperTorso
             local bg = Instance.new("BodyGyro", rootPart)
             bg.P = 9e4
             bg.maxTorque = Vector3.new(9e9, 9e9, 9e9)
             bg.cframe = rootPart.CFrame
             local bv = Instance.new("BodyVelocity", rootPart)
             bv.velocity = Vector3.new(0,0.1,0)
             bv.maxForce = Vector3.new(9e9, 9e9, 9e9)
             hum.PlatformStand = true
             local ctrl = {f = 0, b = 0, l = 0, r = 0}
             local maxspeed = 50
             local flySpeed = 0
             spawn(function()
                 while nowe and hum.Health > 0 do
                     RunService.RenderStepped:Wait()
                     ctrl.f = UserInputService:IsKeyDown(Enum.KeyCode.W) and 1 or 0
                     ctrl.b = UserInputService:IsKeyDown(Enum.KeyCode.S) and 1 or 0
                     ctrl.l = UserInputService:IsKeyDown(Enum.KeyCode.A) and 1 or 0
                     ctrl.r = UserInputService:IsKeyDown(Enum.KeyCode.D) and 1 or 0
                     if ctrl.l + ctrl.r ~= 0 or ctrl.f + ctrl.b ~= 0 then
                         flySpeed = flySpeed+.5+(flySpeed/maxspeed)
                         flySpeed = math.clamp(flySpeed, 0, maxspeed)
                     else
                         flySpeed = math.clamp(flySpeed - 1, 0, maxspeed)
                     end
                     if flySpeed > 0 then
                         local dir = (workspace.CurrentCamera.CoordinateFrame.lookVector * (ctrl.f+ctrl.b)) + ((workspace.CurrentCamera.CoordinateFrame * CFrame.new(ctrl.l+ctrl.r,(ctrl.f+ctrl.b)*.2,0).p) - workspace.CurrentCamera.CoordinateFrame.p)
                         bv.velocity = dir * flySpeed
                         bg.cframe = workspace.CurrentCamera.CoordinateFrame * CFrame.Angles(-math.rad((ctrl.f+ctrl.b)*50*flySpeed/maxspeed),0,0)
                     else
                         bv.velocity = Vector3.new(0,0.1,0)
                     end
                 end
                 bg:Destroy()
                 bv:Destroy()
                 hum.PlatformStand = false
                 chr.Animate.Disabled = false
             end)
         else
             hum:SetStateEnabled(Enum.HumanoidStateType.Climbing,true)
             hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown,true)
             hum:SetStateEnabled(Enum.HumanoidStateType.Flying,true)
             hum:SetStateEnabled(Enum.HumanoidStateType.Freefall,true)
             hum:SetStateEnabled(Enum.HumanoidStateType.GettingUp,true)
             hum:SetStateEnabled(Enum.HumanoidStateType.Jumping,true)
             hum:SetStateEnabled(Enum.HumanoidStateType.Landed,true)
             hum:SetStateEnabled(Enum.HumanoidStateType.Physics,true)
             hum:SetStateEnabled(Enum.HumanoidStateType.PlatformStanding,true)
             hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll,true)
             hum:SetStateEnabled(Enum.HumanoidStateType.Running,true)
             hum:SetStateEnabled(Enum.HumanoidStateType.RunningNoPhysics,true)
             hum:SetStateEnabled(Enum.HumanoidStateType.Seated,true)
             hum:SetStateEnabled(Enum.HumanoidStateType.StrafingNoPhysics,true)
             hum:SetStateEnabled(Enum.HumanoidStateType.Swimming,true)
             hum:ChangeState(Enum.HumanoidStateType.RunningNoPhysics)
             chr.Animate.Disabled = false
         end
     end
     -- 速度按钮点击事件
     SpeedMinusBtn.MouseButton1Click:Connect(function()
         if speeds > 1 then
             speeds = speeds - 1
             updateSpeedDisplay()
         end
     end)
     SpeedPlusBtn.MouseButton1Click:Connect(function()
         if speeds < SPEED_LIMIT then
             speeds = speeds + 1
             updateSpeedDisplay()
         end
     end)
     -- 飞行上下移动快捷键（保留原有）
     UserInputService.InputBegan:Connect(function(input)
         if input.UserInputType == Enum.UserInputType.Keyboard and nowe then
             if input.KeyCode == Enum.KeyCode.Space then
                 tis = RunService.RenderStepped:Connect(function()
                     local humRootPart = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                     if humRootPart then
                         humRootPart.CFrame = humRootPart.CFrame * CFrame.new(0,1,0)
                     end
                 end)
             elseif input.KeyCode == Enum.KeyCode.LeftShift then
                 dis = RunService.RenderStepped:Connect(function()
                     local humRootPart = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                     if humRootPart then
                         humRootPart.CFrame = humRootPart.CFrame * CFrame.new(0,-1,0)
                     end
                 end)
             end
         end
     end)
     UserInputService.InputEnded:Connect(function(input)
         if input.UserInputType == Enum.UserInputType.Keyboard then
             if input.KeyCode == Enum.KeyCode.Space and tis then
                 tis:Disconnect()
                 tis = nil
             elseif input.KeyCode == Enum.KeyCode.LeftShift and dis then
                 dis:Disconnect()
                 dis = nil
             end
         end
     end)
     -- 防护功能拆分为三个独立函数
     -- 防自伤 开关函数
     local function toggleAntiSelfDamage()
         isAntiSelfDamage = not isAntiSelfDamage
         if isAntiSelfDamage then
             -- 开启防自伤
             namecallHook1 = hookmetamethod(game, "__namecall", newcclosure(function(self, ...)
                 local method = getnamecallmethod():lower()
                 if self.Name == "ForceSelfDamage" and method == "fireserver" then
                     return nil
                 end
                 return namecallHook1(self, ...)
             end))
             AntiSelfDamageBtn.Text = "防自伤 [开]"
             AntiSelfDamageBtn.BackgroundColor3 = BTN_ON
         else
             -- 关闭防自伤
             if namecallHook1 then
                 hookmetamethod(game, "__namecall", namecallHook1)
                 namecallHook1 = nil
             end
             AntiSelfDamageBtn.Text = "防自伤 [关]"
             AntiSelfDamageBtn.BackgroundColor3 = BTN_OFF
         end
     end
     -- 防击飞 开关函数
     local function toggleAntiKnockback()
         isAntiKnockback = not isAntiKnockback
         if isAntiKnockback then
             -- 开启防击飞
             renderConnection = RunService.RenderStepped:Connect(function(deltaTime)
                 local char = LocalPlayer.Character
                 if not char then return end
                 local hum = char:FindFirstChildWhichIsA("Humanoid")
                 if not hum then return end
                 local antiFlingObj = char:FindFirstChild("Anti-fling/LocalPlat")
                 if antiFlingObj then
                     antiFlingObj:Destroy()
                 end
                 hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
                 hum:SetStateEnabled(Enum.HumanoidStateType.Freefall, false)
             end)
             AntiKnockbackBtn.Text = "防击飞 [开]"
             AntiKnockbackBtn.BackgroundColor3 = BTN_ON
         else
             -- 关闭防击飞
             if renderConnection then
                 renderConnection:Disconnect()
                 renderConnection = nil
             end
             local char = LocalPlayer.Character
             if char then
                 local hum = char:FindFirstChildWhichIsA("Humanoid")
                 if hum then
                     hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, true)
                     hum:SetStateEnabled(Enum.HumanoidStateType.Freefall, true)
                 end
             end
             AntiKnockbackBtn.Text = "防击飞 [关]"
             AntiKnockbackBtn.BackgroundColor3 = BTN_OFF
         end
     end
     -- 防布娃娃 开关函数
     local function toggleAntiRagdoll()
         isAntiRagdoll = not isAntiRagdoll
         if isAntiRagdoll then
             -- 开启防布娃娃
             namecallHook2 = hookmetamethod(game, "__namecall", newcclosure(function(self, ...)
                 local method = getnamecallmethod()
                 local args = {...}
                 if method == "SetStateEnabled" and typeof(self) == "Instance" and self:IsA("Humanoid") then
                     local state = args[1]
                     if state == Enum.HumanoidStateType.Ragdoll then
                         return namecallHook2(self, state, false)
                     end
                 end
                 return namecallHook2(self, ...)
             end))
             AntiRagdollBtn.Text = "防布娃娃 [开]"
             AntiRagdollBtn.BackgroundColor3 = BTN_ON
         else
             -- 关闭防布娃娃
             if namecallHook2 then
                 hookmetamethod(game, "__namecall", namecallHook2)
                 namecallHook2 = nil
             end
             local char = LocalPlayer.Character
             if char then
                 local hum = char:FindFirstChildWhichIsA("Humanoid")
                 if hum then
                     hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, true)
                 end
             end
             AntiRagdollBtn.Text = "防布娃娃 [关]"
             AntiRagdollBtn.BackgroundColor3 = BTN_OFF
         end
     end
     -- 按钮点击事件绑定
     FlyBtn.MouseButton1Click:Connect(toggleFly)
     AntiSelfDamageBtn.MouseButton1Click:Connect(toggleAntiSelfDamage)
     AntiKnockbackBtn.MouseButton1Click:Connect(toggleAntiKnockback)
     AntiRagdollBtn.MouseButton1Click:Connect(toggleAntiRagdoll)
     DestroyBtn.MouseButton1Click:Connect(function()
         -- 关闭所有功能后销毁UI
         if nowe then
             local chr = LocalPlayer.Character
             local hum = chr and chr:FindFirstChildWhichIsA("Humanoid")
             local humRootPart = chr and chr:FindFirstChild("HumanoidRootPart")
             if hum then
                 hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll,true)
                 hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown,true)
                 hum:SetStateEnabled(Enum.HumanoidStateType.Freefall,true)
                 hum.PlatformStand = false
             end
             if humRootPart then
                 humRootPart.CanCollide = true
             end
         end
         -- 关闭所有防护功能
         if isAntiSelfDamage then toggleAntiSelfDamage() end
         if isAntiKnockback then toggleAntiKnockback() end
         if isAntiRagdoll then toggleAntiRagdoll() end
         mainUI:Destroy()
     end)
     -- 角色重生重置
     LocalPlayer.CharacterAdded:Connect(function(char)
         wait(0.7)
         local hum = char:FindFirstChildWhichIsA("Humanoid")
         local humRootPart = char:FindFirstChild("HumanoidRootPart")
         if hum then
             hum.PlatformStand = false
         end
         if humRootPart then
             humRootPart.CanCollide = true
         end
         if char:FindFirstChild("Animate") then
             char.Animate.Disabled = false
         end
         -- 重置飞行状态
         nowe = false
         tpwalking = false
         speeds = 1
         updateSpeedDisplay()
         FlyBtn.Text = "飞行关"
         FlyBtn.BackgroundColor3 = BTN_OFF
         -- 重置防护状态（重生后保持开启）
         if isAntiSelfDamage then
             if namecallHook1 then hookmetamethod(game, "__namecall", namecallHook1) end
             namecallHook1 = hookmetamethod(game, "__namecall", newcclosure(function(self, ...)
                 local method = getnamecallmethod():lower()
                 if self.Name == "ForceSelfDamage" and method == "fireserver" then
                     return nil
                 end
                 return namecallHook1(self, ...)
             end))
         end
         if isAntiKnockback then
             if renderConnection then renderConnection:Disconnect() end
             renderConnection = RunService.RenderStepped:Connect(function(deltaTime)
                 local char = LocalPlayer.Character
                 if not char then return end
                 local hum = char:FindFirstChildWhichIsA("Humanoid")
                 if not hum then return end
                 local antiFlingObj = char:FindFirstChild("Anti-fling/LocalPlat")
                 if antiFlingObj then
                     antiFlingObj:Destroy()
                 end
                 hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
                 hum:SetStateEnabled(Enum.HumanoidStateType.Freefall, false)
             end)
         end
         if isAntiRagdoll then
             if namecallHook2 then hookmetamethod(game, "__namecall", namecallHook2) end
             namecallHook2 = hookmetamethod(game, "__namecall", newcclosure(function(self, ...)
                 local method = getnamecallmethod()
                 local args = {...}
                 if method == "SetStateEnabled" and typeof(self) == "Instance" and self:IsA("Humanoid") then
                     local state = args[1]
                     if state == Enum.HumanoidStateType.Ragdoll then
                         return namecallHook2(self, state, false)
                     end
                 end
                 return namecallHook2(self, ...)
             end))
         end
     end)
     -- 初始化速度显示
     updateSpeedDisplay()
 end
 -- 初始化整合UI
 buildUI()
