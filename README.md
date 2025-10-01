-- Delta-ready LocalScript: Aimbot Hub Rei (Full)
-- Colar no executor (Delta/Fluxus/other). Colocar em StarterPlayerScripts behavior emulação.

-- CONFIG (exponha em getgenv se quiser)
getgenv().VALID_KEY = "e1FUGEIGDIGIEUIG53863"
getgenv().AIM_SMOOTH = 0.18
getgenv().DEFAULT_FLY_SPEED = 50
getgenv().DEFAULT_WALKSPEED = 16
getgenv().DEFAULT_JUMPPOWER = 55

-- SERVICES
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

-- HELPERS
local function makeUICorner(parent, radius)
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, radius or 12)
    c.Parent = parent
end

local function safeT(v) if typeof(v)=="Instance" and v.Parent then return true end return false end

local function showTempText(lbl, txt, color, time)
    if not lbl or not lbl.Parent then return end
    lbl.Text = txt
    if color then lbl.TextColor3 = color end
    delay(time or 1.6, function()
        if lbl and lbl.Parent then
            lbl.Text = ""
        end
    end)
end

local function animateButton(btn, msg)
    local originalColor = btn.BackgroundColor3
    local flash = Color3.fromRGB(50,255,50)
    for i = 1,6 do
        btn.BackgroundColor3 = (i % 2 == 0) and originalColor or flash
        if i==1 and msg then showTempText(statusLabel, msg, flash) end
        wait(0.08)
    end
    btn.BackgroundColor3 = originalColor
end

-- GUI: Key System
local keyGui = Instance.new("ScreenGui")
keyGui.Name = "KeySystem"
keyGui.Parent = PlayerGui
keyGui.ResetOnSpawn = false

local overlay = Instance.new("Frame", keyGui)
overlay.Size = UDim2.new(1,0,1,0)
overlay.BackgroundColor3 = Color3.fromRGB(0,0,0)
overlay.BackgroundTransparency = 0.45

local panel = Instance.new("Frame", overlay)
panel.Size = UDim2.new(0,460,0,300)
panel.AnchorPoint = Vector2.new(0.5,0.5)
panel.Position = UDim2.new(0.5,0,0.5,0)
panel.BackgroundColor3 = Color3.fromRGB(18,18,20)
makeUICorner(panel,16)

local title = Instance.new("TextLabel", panel)
title.Size = UDim2.new(1,0,0,48)
title.Position = UDim2.new(0,0,0,8)
title.BackgroundTransparency = 1
title.Font = Enum.Font.GothamBold
title.TextSize = 22
title.Text = "☠️ Aimbot Hub Rei ☠️ (Key)"
title.TextColor3 = Color3.fromRGB(255,120,120)

local info = Instance.new("TextLabel", panel)
info.Size = UDim2.new(1,-40,0,60)
info.Position = UDim2.new(0,20,0,66)
info.BackgroundTransparency = 1
info.Font = Enum.Font.Gotham
info.TextSize = 14
info.TextWrapped = true
info.Text = "Fala com o dono/ajudantes pra pegar a Key.\nCola a key e clique em Verificar."
info.TextColor3 = Color3.fromRGB(220,220,220)

local keyBox = Instance.new("TextBox", panel)
keyBox.Size = UDim2.new(0.9,0,0,44)
keyBox.Position = UDim2.new(0.05,0,0,140)
keyBox.PlaceholderText = "Digite a Key"
keyBox.ClearTextOnFocus = false
keyBox.Font = Enum.Font.Gotham
keyBox.TextSize = 16
keyBox.BackgroundColor3 = Color3.fromRGB(36,36,40)
keyBox.TextColor3 = Color3.fromRGB(255,255,255)
makeUICorner(keyBox,8)

local verifyBtn = Instance.new("TextButton", panel)
verifyBtn.Size = UDim2.new(0.5,0,0,40)
verifyBtn.Position = UDim2.new(0.25,0,0,196)
verifyBtn.BackgroundColor3 = Color3.fromRGB(100,170,255)
verifyBtn.Font = Enum.Font.GothamBold
verifyBtn.Text = "Verificar"
makeUICorner(verifyBtn,8)

statusLabel = Instance.new("TextLabel", panel)
statusLabel.Size = UDim2.new(1,-20,0,30)
statusLabel.Position = UDim2.new(0,10,0,250)
statusLabel.BackgroundTransparency = 1
statusLabel.Font = Enum.Font.Gotham
statusLabel.TextSize = 14
statusLabel.TextColor3 = Color3.fromRGB(255,120,120)
statusLabel.Text = ""

-- MAIN GUI (hidden until key ok)
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AimbotHubRei"
screenGui.Parent = PlayerGui
screenGui.ResetOnSpawn = false
screenGui.Enabled = false

local main = Instance.new("Frame", screenGui)
main.Size = UDim2.new(0,420,0,760)
main.Position = UDim2.new(0.04,0,0.06,0)
main.BackgroundColor3 = Color3.fromRGB(16,16,18)
makeUICorner(main,18)

local topTitle = Instance.new("TextLabel", main)
topTitle.Size = UDim2.new(1,0,0,48)
topTitle.Position = UDim2.new(0,0,0,8)
topTitle.BackgroundTransparency = 1
topTitle.Font = Enum.Font.GothamBold
topTitle.TextSize = 22
topTitle.Text = "☠️ Aimbot Hub Rei ☠️"
topTitle.TextColor3 = Color3.fromHSV(0,1,1) -- will rotate color via tween
topTitle.TextXAlignment = Enum.TextXAlignment.Center

-- animate title color (rainbow)
local grad = Instance.new("UIGradient", topTitle)
grad.Rotation = 0
TweenService:Create(grad, TweenInfo.new(5, Enum.EasingStyle.Linear, Enum.EasingDirection.InOut, -1, true), {Rotation = 360}):Play()

-- button factory
local function createButton(text, posY, color)
    local btn = Instance.new("TextButton", main)
    btn.Size = UDim2.new(0.9,0,0,42)
    btn.Position = UDim2.new(0.05,0,0,posY)
    btn.Text = text
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 16
    btn.BackgroundColor3 = color
    btn.TextColor3 = Color3.fromRGB(255,255,255)
    makeUICorner(btn,10)
    return btn
end

-- create controls
local espBtn     = createButton("ESP: OFF", 70, Color3.fromRGB(180,20,20))
local aimbotBtn  = createButton("Aimbot: OFF", 124, Color3.fromRGB(20,20,160))
local speedBtn   = createButton("Aplicar WalkSpeed", 178, Color3.fromRGB(200,180,0))
local flyBtn     = createButton("Fly: OFF", 232, Color3.fromRGB(20,140,140))
local noclipBtn  = createButton("Noclip: OFF", 286, Color3.fromRGB(140,20,20))
local tpBtn      = createButton("Teleporte: Alvo Próximo", 340, Color3.fromRGB(120,180,60))
local discordBtn = createButton("Discord ☠️ (Copiar)", 394, Color3.fromRGB(20,170,170))

local function makeLabel(text, posY)
    local lbl = Instance.new("TextLabel", main)
    lbl.Size = UDim2.new(0.9,0,0,20)
    lbl.Position = UDim2.new(0.05,0,0,posY)
    lbl.BackgroundTransparency = 1
    lbl.Font = Enum.Font.Gotham
    lbl.TextSize = 14
    lbl.Text = text
    lbl.TextColor3 = Color3.fromRGB(200,200,200)
    return lbl
end

makeLabel("WalkSpeed", 450)
local speedBox = Instance.new("TextBox", main)
speedBox.Size = UDim2.new(0.9,0,0,32)
speedBox.Position = UDim2.new(0.05,0,0,472)
speedBox.PlaceholderText = "Ex: 50"
speedBox.BackgroundColor3 = Color3.fromRGB(45,45,50)
makeUICorner(speedBox,8)

makeLabel("Fly Speed", 516)
local flyBox = Instance.new("TextBox", main)
flyBox.Size = UDim2.new(0.9,0,0,32)
flyBox.Position = UDim2.new(0.05,0,0,538)
flyBox.PlaceholderText = "Ex: 50"
flyBox.BackgroundColor3 = Color3.fromRGB(45,45,50)
makeUICorner(flyBox,8)

makeLabel("JumpPower", 582)
local jumpBox = Instance.new("TextBox", main)
jumpBox.Size = UDim2.new(0.9,0,0,32)
jumpBox.Position = UDim2.new(0.05,0,0,604)
jumpBox.PlaceholderText = "Ex: 55"
jumpBox.BackgroundColor3 = Color3.fromRGB(45,45,50)
makeUICorner(jumpBox,8)

-- STATE
local espOn, aimbotOn, noclipOn, flyOn = false, false, false, false
local wspeed = getgenv().DEFAULT_WALKSPEED
local flySpeed = getgenv().DEFAULT_FLY_SPEED
local jumpPower = getgenv().DEFAULT_JUMPPOWER

-- UTIL: nearest enemy & apply highlight
local highlights = {} -- map player -> Highlight
local function ensureHighlight(p)
    if highlights[p] and safeT(highlights[p]) then return highlights[p] end
    if p.Character then
        local h = Instance.new("Highlight")
        h.Name = "AimbotHighlight"
        h.Adornee = p.Character
        h.Parent = workspace
        h.FillTransparency = 0.6
        h.OutlineTransparency = 1
        h.FillColor = Color3.fromRGB(255,20,20)
        highlights[p] = h
        return h
    end
    return nil
end

local function removeHighlight(p)
    if highlights[p] then
        pcall(function() highlights[p]:Destroy() end)
        highlights[p] = nil
    end
end

local function getClosestEnemy()
    local center = (LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") and LocalPlayer.Character.HumanoidRootPart.Position) or Camera.CFrame.Position
    local best, bd = nil, math.huge
    for _,p in pairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") and p.Character:FindFirstChild("Head") and p.Character:FindFirstChildOfClass("Humanoid") and p.Character:FindFirstChildOfClass("Humanoid").Health>0 then
            local pos = p.Character.HumanoidRootPart.Position
            local d = (pos - center).Magnitude
            if d < bd then bd = d; best = p end
        end
    end
    return best
end

-- BUTTON LOGIC
espBtn.MouseButton1Click:Connect(function()
    espOn = not espOn
    espBtn.Text = "ESP: "..(espOn and "ON" or "OFF")
    if espOn then
        animateButton(espBtn, "🔥 Você ativou o ESP!")
    else
        showTempText(statusLabel, "ESP desativado", Color3.fromRGB(200,200,200))
        -- remove all highlights
        for p,_ in pairs(highlights) do removeHighlight(p) end
    end
end)

aimbotBtn.MouseButton1Click:Connect(function()
    aimbotOn = not aimbotOn
    aimbotBtn.Text = "Aimbot: "..(aimbotOn and "ON" or "OFF")
    if aimbotOn then
        animateButton(aimbotBtn, "🔥 Você ativou o Aimbot!")
    else
        showTempText(statusLabel, "Aimbot desativado", Color3.fromRGB(200,200,200))
    end
end)

speedBtn.MouseButton1Click:Connect(function()
    local n = tonumber(speedBox.Text)
    if n then
        wspeed = n
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
            LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = wspeed
        end
        showTempText(statusLabel, "WalkSpeed: "..wspeed, Color3.fromRGB(220,200,80))
    else
        showTempText(statusLabel, "Valor inválido", Color3.fromRGB(220,60,60))
    end
end)

flyBtn.MouseButton1Click:Connect(function()
    flyOn = not flyOn
    flyBtn.Text = "Fly: "..(flyOn and "ON" or "OFF")
    if flyOn then
        animateButton(flyBtn, "🛫 Fly ativado!")
    else
        showTempText(statusLabel, "Fly desativado", Color3.fromRGB(200,200,200))
    end
end)

noclipBtn.MouseButton1Click:Connect(function()
    noclipOn = not noclipOn
    noclipBtn.Text = "Noclip: "..(noclipOn and "ON" or "OFF")
    showTempText(statusLabel, noclipOn and "Noclip ativado" or "Noclip desativado", Color3.fromRGB(120,255,120))
end)

tpBtn.MouseButton1Click:Connect(function()
    local target = getClosestEnemy()
    if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
        local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        if hrp then
            hrp.CFrame = target.Character.HumanoidRootPart.CFrame * CFrame.new(0,0,1) -- slightly behind
            showTempText(statusLabel, "Teleportado para "..target.Name, Color3.fromRGB(120,255,120))
        end
    else
        showTempText(statusLabel, "Nenhum alvo encontrado", Color3.fromRGB(255,80,80))
    end
end)

discordBtn.MouseButton1Click:Connect(function()
    pcall(function() setclipboard("https://discord.gg/86EWNmKz") end)
    showTempText(statusLabel, "Link do Discord copiado!", Color3.fromRGB(90,200,230))
end)

-- Noclip loop: frequently attempt to disable collisions
RunService.Heartbeat:Connect(function()
    if noclipOn and LocalPlayer.Character then
        for _,part in pairs(LocalPlayer.Character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    end
end)

-- Fly: move in direction camera looks
RunService.Heartbeat:Connect(function()
    if flyOn and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        local hrp = LocalPlayer.Character.HumanoidRootPart
        local bv = hrp:FindFirstChild("__AimbotFlyBV")
        if not bv then
            bv = Instance.new("BodyVelocity")
            bv.Name = "__AimbotFlyBV"
            bv.MaxForce = Vector3.new(9e5,9e5,9e5)
            bv.Parent = hrp
        end
        local speed = tonumber(flyBox.Text) or flySpeed
        local dir = Camera.CFrame.LookVector
        bv.Velocity = dir * speed
    else
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            local hrp = LocalPlayer.Character.HumanoidRootPart
            local bv = hrp:FindFirstChild("__AimbotFlyBV")
            if bv then pcall(function() bv:Destroy() end) end
        end
    end
end)

-- ESP loop: add/remove highlight for enemies
RunService.Heartbeat:Connect(function()
    if espOn then
        for _,p in pairs(Players:GetPlayers()) do
            if p ~= LocalPlayer then
                if p.Character and p.Character:FindFirstChildOfClass("Humanoid") and p.Character:FindFirstChild("Head") and p.Character:FindFirstChild("HumanoidRootPart") and p.Character:FindFirstChildOfClass("Humanoid").Health>0 then
                    ensureHighlight(p)
                else
                    removeHighlight(p)
                end
            end
        end
    end
end)

-- Aimbot loop: smooth camera to head of closest enemy
RunService.RenderStepped:Connect(function(dt)
    if aimbotOn and LocalPlayer.Character and Camera and Camera:IsDescendantOf(game) then
        local target = getClosestEnemy()
        if target and target.Character and target.Character:FindFirstChild("Head") then
            local head = target.Character.Head
            local camPos = Camera.CFrame.Position
            local goal = CFrame.new(camPos, head.Position)
            local smooth = getgenv().AIM_SMOOTH or 0.18
            Camera.CFrame = Camera.CFrame:Lerp(goal, math.clamp(smooth,0,1))
        end
    end
end)

-- CLEANUP highlights when player leaves
Players.PlayerRemoving:Connect(function(plr) removeHighlight(plr) end)

-- KEY VERIFY
local function unlockAll()
    screenGui.Enabled = true
    keyGui:Destroy()
end

verifyBtn.MouseButton1Click:Connect(function()
    if (keyBox.Text or ""):gsub("%s+","") == (getgenv().VALID_KEY or "") then
        unlockAll()
    else
        statusLabel.Text = "❌ Key inválida"
    end
end)

-- FOR TESTAR SEM KEY: descomente a linha abaixo
-- unlockAll()

-- PAINEL ARRASTÁVEL
local dragging = false
local dragInput, dragStart, startPos
local function update(input)
    local delta = input.Position - dragStart
    main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

main.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = main.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then dragging = false end
        end)
    end
end)
main.InputChanged:Connect(function(input) if input.UserInputType==Enum.UserInputType.MouseMovement or input.UserInputType==Enum.UserInputType.Touch then dragInput = input end end)
UserInputService.InputChanged:Connect(function(input) if input==dragInput and dragging then update(input) end end)

print("Aimbot Hub Rei carregado (Delta ready).")
