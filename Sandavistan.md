-- ===================================================================
-- GTML: SANDEVISTAN V2 — VERSÃO OTIMIZADA PARA ALTO FPS (SEM TRAVAMENTOS)
-- ===================================================================

if _G.GTML_Stop then
    pcall(_G.GTML_Stop)
end

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")

local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local Mouse = LocalPlayer:GetMouse()

local active = false
local returning = false
local military = false
local destroyed = false
local speed = 50
local stepCount = 0
local elapsed = 0

-- Contadores de uso para as penalidades de dano
local normalUseCount = 0
local lastMilitaryColorIdx = 0

local startCFrame = nil
local path = {}
local ghosts = {}
local connections = {}
local saved = { walkSpeed = 16, jumpPower = 50, autoRotate = true }
local colorEffect = nil
local originalFOV = Camera.FieldOfView

local shadowClone = nil
local smasherMode = false

-- Cores do Modo Militar
local militaryColors = {
    Color3.fromRGB(75, 255, 33),   -- 1: Verde-limão
    Color3.fromRGB(0, 240, 255),   -- 2: Ciano
    Color3.fromRGB(0, 100, 255),   -- 3: Azul intenso
    Color3.fromRGB(150, 40, 255),  -- 4: Roxo intenso
    Color3.fromRGB(255, 40, 40),   -- 5: Vermelho intenso
    Color3.fromRGB(255, 145, 0),   -- 6: Laranja
    Color3.fromRGB(255, 235, 30),  -- 7: Amarelo
    Color3.fromRGB(255, 255, 255)  -- 8: Branco
}

-- ===================================================================
-- INTERFACE GRÁFICA (FAIXA VERDE, PAINEL PRETO/BRANCO)
-- ===================================================================
local gui = Instance.new("ScreenGui")
gui.Name = "GTML_Sandevistan_GUI"
gui.ResetOnSpawn = false
gui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local panel = Instance.new("Frame")
panel.Size = UDim2.new(0, 260, 0, 325)
panel.Position = UDim2.new(0.05, 0, 0.25, 0)
panel.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
panel.BorderSizePixel = 2
panel.BorderColor3 = Color3.fromRGB(0, 0, 0)
panel.Parent = gui

local header = Instance.new("Frame")
header.Size = UDim2.new(1, 0, 0, 25)
header.BackgroundColor3 = Color3.fromRGB(0, 180, 0)
header.BorderSizePixel = 0
header.Parent = panel

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -10, 1, 0)
title.Position = UDim2.new(0, 5, 0, 0)
title.BackgroundTransparency = 1
title.Text = "GTML | SANDEVISTAN"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.Legacy
title.TextSize = 14
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = header

-- Arrastável Mobile
local dragging, dragStart, startPos = false, nil, nil
header.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = panel.Position
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        panel.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

local normalButton = Instance.new("TextButton")
normalButton.Size = UDim2.new(0.9, 0, 0, 30)
normalButton.Position = UDim2.new(0.05, 0, 0.10, 0)
normalButton.BackgroundColor3 = Color3.fromRGB(240, 240, 240)
normalButton.BorderColor3 = Color3.fromRGB(0, 0, 0)
normalButton.Text = "SANDEVISTAN NORMAL"
normalButton.Font = Enum.Font.Legacy
normalButton.TextColor3 = Color3.fromRGB(0, 0, 0)
normalButton.TextSize = 11
normalButton.Parent = panel

local militaryButton = Instance.new("TextButton")
militaryButton.Size = UDim2.new(0.9, 0, 0, 30)
militaryButton.Position = UDim2.new(0.05, 0, 0.21, 0)
militaryButton.BackgroundColor3 = Color3.fromRGB(240, 240, 240)
militaryButton.BorderColor3 = Color3.fromRGB(0, 0, 0)
militaryButton.Text = "SANDEVISTAN MILITAR"
militaryButton.Font = Enum.Font.Legacy
militaryButton.TextColor3 = Color3.fromRGB(0, 0, 0)
militaryButton.TextSize = 11
militaryButton.Parent = panel

local smasherButton = Instance.new("TextButton")
smasherButton.Size = UDim2.new(0.9, 0, 0, 30)
smasherButton.Position = UDim2.new(0.05, 0, 0.32, 0)
smasherButton.BackgroundColor3 = Color3.fromRGB(240, 240, 240)
smasherButton.BorderColor3 = Color3.fromRGB(0, 0, 0)
smasherButton.Text = "MILITAR ADAM SMASHER"
smasherButton.Font = Enum.Font.Legacy
smasherButton.TextColor3 = Color3.fromRGB(0, 0, 0)
smasherButton.TextSize = 10
smasherButton.Parent = panel

local speedLabel = Instance.new("TextLabel")
speedLabel.Size = UDim2.new(0.9, 0, 0, 20)
speedLabel.Position = UDim2.new(0.05, 0, 0.44, 0)
speedLabel.BackgroundTransparency = 1
speedLabel.Text = "Velocidade militar:"
speedLabel.Font = Enum.Font.Legacy
speedLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
speedLabel.TextSize = 12
speedLabel.TextXAlignment = Enum.TextXAlignment.Left
speedLabel.Parent = panel

local speedBox = Instance.new("TextBox")
speedBox.Size = UDim2.new(0.9, 0, 0, 25)
speedBox.Position = UDim2.new(0.05, 0, 0.52, 0)
speedBox.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
speedBox.BorderColor3 = Color3.fromRGB(0, 0, 0)
speedBox.Text = "500"
speedBox.Font = Enum.Font.Legacy
speedBox.TextColor3 = Color3.fromRGB(0, 0, 0)
speedBox.TextSize = 14
speedBox.Parent = panel

local status = Instance.new("TextLabel")
status.Size = UDim2.new(0.9, 0, 0, 20)
status.Position = UDim2.new(0.05, 0, 0.68, 0)
status.BackgroundTransparency = 1
status.Text = "Status: aguardando ativação"
status.Font = Enum.Font.Legacy
status.TextColor3 = Color3.fromRGB(255, 255, 255)
status.TextSize = 12
status.TextXAlignment = Enum.TextXAlignment.Left
status.Parent = panel

local stepsLabel = Instance.new("TextLabel")
stepsLabel.Size = UDim2.new(0.9, 0, 0, 20)
stepsLabel.Position = UDim2.new(0.05, 0, 0.78, 0)
stepsLabel.BackgroundTransparency = 1
stepsLabel.Text = "Passos registrados: 0"
stepsLabel.Font = Enum.Font.Legacy
stepsLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
stepsLabel.TextSize = 12
stepsLabel.TextXAlignment = Enum.TextXAlignment.Left
stepsLabel.Parent = panel

-- ===================================================================
-- FUNÇÕES DE SUPORTE OTIMIZADAS
-- ===================================================================
local function getCharParts()
    local char = LocalPlayer.Character
    if not char then return nil, nil end
    return char:FindFirstChildOfClass("Humanoid"), char:FindFirstChild("HumanoidRootPart")
end

local function enableGreen()
    if not colorEffect then
        colorEffect = Instance.new("ColorCorrectionEffect")
        colorEffect.TintColor = Color3.fromRGB(150, 255, 145)
        colorEffect.Contrast = 0.15
        colorEffect.Saturation = -0.1
        colorEffect.Parent = Lighting
    end
    Camera.FieldOfView = 100
end

local function disableGreen()
    if colorEffect then colorEffect:Destroy(); colorEffect = nil end
    Camera.FieldOfView = originalFOV
end

local function clearGhosts()
    for _, ghost in ipairs(ghosts) do 
        if ghost then ghost:Destroy() end 
    end
    ghosts = {}
end

local function removeShadow()
    if shadowClone then
        shadowClone:Destroy()
        shadowClone = nil
    end
end

local function createShadow(cf)
    removeShadow()
    local char = LocalPlayer.Character
    if not char then return end
    
    char.Archivable = true
    shadowClone = char:Clone()
    char.Archivable = false
    
    -- Otimização: Remove componentes desnecessários do clone para poupar memória
    for _, part in ipairs(shadowClone:GetDescendants()) do
        if part:IsA("Script") or part:IsA("LocalScript") or part:IsA("Decal") then
            part:Destroy()
        elseif part:IsA("BasePart") then
            part.Color = Color3.fromRGB(15, 15, 15)
            part.Material = Enum.Material.SmoothPlastic
            part.Transparency = 0.2
            part.CanCollide = false
        end
    end
    
    shadowClone.Parent = workspace
    local sRoot = shadowClone:FindFirstChild("HumanoidRootPart")
    if sRoot then
        sRoot.CFrame = cf
    end
    
    local shadowHum = shadowClone:FindFirstChildOfClass("Humanoid")
    if shadowHum then
        shadowHum.WalkSpeed = math.min(speed, 1000)
        Camera.CameraSubject = shadowHum
        LocalPlayer.Character = shadowClone
    end
end

-- Rastro Otimizado (Limita o número máximo de peças na tela para não travar o FPS)
local function createTrail(color, cf)
    local charRef = shadowClone or LocalPlayer.Character
    if not charRef then return end
    local rootRef = charRef:FindFirstChild("HumanoidRootPart")
    if not rootRef then return end
    
    local p = Instance.new("Part")
    p.Size = Vector3.new(2, 5, 2)
    p.CFrame = cf
    p.Anchored = true
    p.CanCollide = false
    p.Material = Enum.Material.Neon
    p.Color = color
    p.Transparency = 0.4
    p.Parent = workspace
    
    table.insert(ghosts, p)
    if #ghosts > 40 then -- Limite estrito de 40 peças para manter o FPS fluido no mobile
        local oldest = table.remove(ghosts, 1)
        if oldest then oldest:Destroy() end
    end
end

-- ===================================================================
-- REPLAY & PENALIDADES DE DANO OTIMIZADAS
-- ===================================================================
local function executeReplay(isSmasher, targetCF)
    returning = true
    status.Text = "Status: Executando percurso..."
    
    removeShadow()
    local realChar = workspace:FindFirstChild(LocalPlayer.Name)
    if realChar then
        LocalPlayer.Character = realChar
        local hum = realChar:FindFirstChildOfClass("Humanoid")
        local root = realChar:FindFirstChild("HumanoidRootPart")
        if hum then Camera.CameraSubject = hum end
        
        if startCFrame and root then
            root.CFrame = startCFrame
        end
        
        -- Loop de replay otimizado com passo adaptativo (Fluido e sem travamentos)
        local skip = math.max(1, math.floor(#path / 35))
        local i = 1
        while i <= #path do
            if destroyed then break end
            if root then
                root.CFrame = path[i]
            end
            i = i + skip
            RunService.RenderStepped:Wait()
        end
        
        if isSmasher and targetCF and root then
            root.CFrame = targetCF
        elseif #path > 0 and root then
            root.CFrame = path[#path]
        end
    end
    
    disableGreen()
    clearGhosts()
    returning = false
end

local function stopAbility(isSmasher, targetCF)
    if not active then return end
    active = false
    
    executeReplay(isSmasher, targetCF)
    
    local hum, _ = getCharParts()
    if hum then
        hum.WalkSpeed = saved.walkSpeed
        pcall(function() hum.JumpPower = saved.jumpPower end)
        hum.AutoRotate = saved.autoRotate
    end
    
    path = {}
    stepCount = 0
    elapsed = 0
    smasherMode = false
    lastMilitaryColorIdx = 0
    
    status.Text = "Status: aguardando ativação"
    stepsLabel.Text = "Passos registrados: 0"
    normalButton.Text = "SANDEVISTAN NORMAL"
    militaryButton.Text = "SANDEVISTAN MILITAR"
    smasherButton.Text = "MILITAR ADAM SMASHER"
end

local function startAbility(isMilitary, isSmasher)
    if active or returning then return end
    local hum, root = getCharParts()
    if not hum or not root then return end
    
    -- Penalidades de dano
    if isSmasher then
        pcall(function() hum:TakeDamage(10) end)
    elseif not isMilitary then
        normalUseCount = normalUseCount + 1
        if normalUseCount >= 3 then
            normalUseCount = 0
            pcall(function() hum:TakeDamage(5) end)
        end
    end
    
    saved.walkSpeed = hum.WalkSpeed
    pcall(function() saved.jumpPower = hum.JumpPower end)
    saved.autoRotate = hum.AutoRotate
    
    military = isMilitary
    smasherMode = isSmasher or false
    active = true
    stepCount = 0
    elapsed = 0
    path = {}
    
    startCFrame = root.CFrame
    table.insert(path, root.CFrame)
    
    if smasherMode then
        speed = 999999
        smasherButton.Text = "CLIQUE NO MAPA..."
        status.Text = "Status: Clique em um lugar no mapa!"
    elseif military then
        local rawSpeed = tonumber(speedBox.Text) or 500
        speed = math.clamp(rawSpeed, 1, 9999)
        militaryButton.Text = "DESATIVAR MILITAR"
        status.Text = "Status: Modo Militar Ativo"
    else
        speed = 50
        normalButton.Text = "DESATIVAR NORMAL"
        status.Text = "Status: Modo Normal Ativo"
    end
    
    enableGreen()
    createShadow(startCFrame)
end

speedBox.FocusLost:Connect(function()
    local val = tonumber(speedBox.Text)
    if not val then speedBox.Text = "500"
    else speedBox.Text = tostring(math.clamp(math.floor(val), 1, 9999)) end
end)

normalButton.MouseButton1Click:Connect(function()
    if active then stopAbility(false) else startAbility(false, false) end
end)

militaryButton.MouseButton1Click:Connect(function()
    if active then stopAbility(false) else startAbility(true, false) end
end)

smasherButton.MouseButton1Click:Connect(function()
    if active then 
        stopAbility(false) 
    else 
        startAbility(true, true) 
    end
end)

-- Clique no mapa otimizado para o Adam Smasher
local mouseClickConn = UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if active and smasherMode and not returning then
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            local targetPos = Mouse.Hit
            if targetPos and shadowClone then
                local sRoot = shadowClone:FindFirstChild("HumanoidRootPart")
                if sRoot then
                    local startPos = sRoot.Position
                    local endPos = targetPos.Position + Vector3.new(0, 3, 0)
                    local dist = (endPos - startPos).Magnitude
                    local steps = math.clamp(math.floor(dist / 8), 10, 100)
                    
                    for i = 1, steps do
                        local alpha = i / steps
                        local interpolatedCF = CFrame.new(startPos:Lerp(endPos, alpha))
                        table.insert(path, interpolatedCF)
                        
                        local colorIdx = (math.floor(i / 3) % #militaryColors) + 1
                        createTrail(militaryColors[colorIdx], interpolatedCF)
                    end
                    
                    smasherMode = false
                    local finalDestinationCF = CFrame.new(endPos)
                    stopAbility(true, finalDestinationCF)
                end
            end
        end
    end
end)
table.insert(connections, mouseClickConn)

local hb = RunService.Heartbeat:Connect(function(dt)
    if not active or returning or smasherMode then return end
    
    local sHum = shadowClone and shadowClone:FindFirstChildOfClass("Humanoid")
    local sRoot = shadowClone and shadowClone:FindFirstChild("HumanoidRootPart")
    
    if not sHum or not sRoot then return end
    
    elapsed = elapsed + dt
    
    -- Intervalo de registro otimizado para não pesar o processador
    if elapsed >= 0.1 then
        elapsed = 0
        stepCount = stepCount + 1
        stepsLabel.Text = "Passos registrados: " .. tostring(stepCount)
        table.insert(path, sRoot.CFrame)
        
        local colorIdx = (math.floor(stepCount / 2) % #militaryColors) + 1
        createTrail(militaryColors[colorIdx], sRoot.CFrame)
        
        if military then
            if colorIdx ~= lastMilitaryColorIdx and lastMilitaryColorIdx ~= 0 then
                local realHum, _ = getCharParts()
                if realHum then
                    pcall(function() realHum:TakeDamage(5) end)
                end
            end
            lastMilitaryColorIdx = colorIdx
        end
        
        local realHum, _ = getCharParts()
        if not military then
            if stepCount >= 40 then stopAbility(false); return end
        else
            if stepCount >= 250 then
                if realHum then pcall(function() realHum:TakeDamage(80) end) end
                stopAbility(false)
                return
            end
        end
    end
end)
table.insert(connections, hb)

_G.GTML_Stop = function()
    destroyed = true
    if active then stopAbility(false) end
    removeShadow()
    local realChar = workspace:FindFirstChild(LocalPlayer.Name)
    if realChar then LocalPlayer.Character = realChar end
    for _, c in ipairs(connections) do c:Disconnect() end
    disableGreen()
    clearGhosts()
    gui:Destroy()
    _G.GTML_Stop = nil
end
