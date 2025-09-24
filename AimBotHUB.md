--[[
- Painel customizado com imagem de fundo, borda arredondada e preta, arrastável, com posição persistente ao morrer/reaparecer
- Botão de minimizar ("-") no canto superior direito, SEM borda preta
- Ao minimizar, aparece um botão simples "Reabrir" na tela, sempre visível, para reabrir o painel
- Botões Ativar Aimbot, + e - SEM borda preta
- Círculo de FOV permanece mesmo minimizado se o aimbot estiver ativado
- Funciona após morrer/reaparecer, sempre na última posição deixada
]]

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local backgroundImageId = "rbxassetid://123597786948714"
local lastPanelPosition = UDim2.new(0.5, -150, 0.5, -150)
local minimized = false
local guiObjects
local reopenBtn

-- Remove GUI utilitário
local function removeGuiByName(name)
    local g = LocalPlayer.PlayerGui:FindFirstChild(name)
    if g then g:Destroy() end
end

-- Cria botão "Reabrir" sempre no canto inferior esquerdo (ou mude a posição se quiser)
function CreateReopenBtn()
    removeGuiByName("ReabrirBtn")
    reopenBtn = Instance.new("TextButton")
    reopenBtn.Name = "ReabrirBtn"
    reopenBtn.Parent = LocalPlayer.PlayerGui
    reopenBtn.Size = UDim2.new(0, 100, 0, 40)
    reopenBtn.Position = UDim2.new(0, 10, 1, -50)
    reopenBtn.AnchorPoint = Vector2.new(0,0)
    reopenBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    reopenBtn.Text = "Reabrir"
    reopenBtn.TextColor3 = Color3.new(1,1,1)
    reopenBtn.TextSize = 22
    reopenBtn.Font = Enum.Font.SourceSansBold
    reopenBtn.AutoButtonColor = true

    local hubCorner = Instance.new("UICorner")
    hubCorner.CornerRadius = UDim.new(0, 12)
    hubCorner.Parent = reopenBtn

    local hubStroke = Instance.new("UIStroke")
    hubStroke.Color = Color3.new(0,0,0)
    hubStroke.Thickness = 3
    hubStroke.Parent = reopenBtn

    reopenBtn.MouseButton1Click:Connect(function()
        minimized = false
        removeGuiByName("ReabrirBtn")
        guiObjects = CreateAimbotPanel()
        updateConnections()
        updateFovLabel()
    end)
end

-- Cria o painel principal
function CreateAimbotPanel()
    removeGuiByName("AimbotGUI")
    removeGuiByName("ReabrirBtn")

    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "AimbotGUI"
    screenGui.ResetOnSpawn = false
    screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

    local panel = Instance.new("Frame")
    panel.Size = UDim2.new(0, 300, 0, 300)
    panel.Position = lastPanelPosition
    panel.BackgroundTransparency = 1
    panel.Active = true
    panel.Parent = screenGui

    -- Borda preta arredondada
    local container = Instance.new("Frame")
    container.Size = UDim2.new(1, 0, 1, 0)
    container.Position = UDim2.new(0, 0, 0, 0)
    container.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    container.BorderSizePixel = 0
    container.Parent = panel

    local uicornerContainer = Instance.new("UICorner")
    uicornerContainer.CornerRadius = UDim.new(0, 30)
    uicornerContainer.Parent = container

    -- Imagem de fundo
    local bg = Instance.new("ImageLabel")
    bg.Size = UDim2.new(1, -8, 1, -8)
    bg.Position = UDim2.new(0, 4, 0, 4)
    bg.Image = backgroundImageId
    bg.BackgroundTransparency = 1
    bg.Parent = container

    local uicornerBG = Instance.new("UICorner")
    uicornerBG.CornerRadius = UDim.new(0, 24)
    uicornerBG.Parent = bg

    -- Botão de minimizar no topo direito (SEM borda preta)
    local minimizeBtn = Instance.new("TextButton")
    minimizeBtn.Size = UDim2.new(0, 36, 0, 36)
    minimizeBtn.Position = UDim2.new(1, -38, 0, 2)
    minimizeBtn.AnchorPoint = Vector2.new(0, 0)
    minimizeBtn.Text = "-"
    minimizeBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    minimizeBtn.TextColor3 = Color3.fromRGB(255,255,255)
    minimizeBtn.TextSize = 28
    minimizeBtn.Font = Enum.Font.SourceSansBold
    minimizeBtn.AutoButtonColor = true
    minimizeBtn.ZIndex = 2
    minimizeBtn.BorderSizePixel = 0
    minimizeBtn.Parent = panel
    local uicornerMinimize = Instance.new("UICorner")
    uicornerMinimize.CornerRadius = UDim.new(1, 0)
    uicornerMinimize.Parent = minimizeBtn

    -- Botão centralizado - Ativar Aimbot (sem borda)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0.8, 0, 0, 70)
    button.Position = UDim2.new(0.5, 0, 0.45, 0)
    button.AnchorPoint = Vector2.new(0.5, 0.5)
    button.Text = "Ativar Aimbot"
    button.BackgroundColor3 = Color3.fromRGB(30, 200, 40)
    button.TextColor3 = Color3.fromRGB(255,255,255)
    button.TextSize = 32
    button.Font = Enum.Font.SourceSansBold
    button.AutoButtonColor = true
    button.ZIndex = 2
    button.BorderSizePixel = 0
    button.Parent = panel
    local uicornerButton = Instance.new("UICorner")
    uicornerButton.CornerRadius = UDim.new(0, 20)
    uicornerButton.Parent = button

    -- Botão "-" para diminuir FOV (sem borda)
    local minusBtn = Instance.new("TextButton")
    minusBtn.Size = UDim2.new(0, 40, 0, 40)
    minusBtn.Position = UDim2.new(0.17, 0, 0.80, 0)
    minusBtn.AnchorPoint = Vector2.new(0.5, 0.5)
    minusBtn.Text = "-"
    minusBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    minusBtn.TextColor3 = Color3.fromRGB(255,255,255)
    minusBtn.TextSize = 28
    minusBtn.Font = Enum.Font.SourceSansBold
    minusBtn.AutoButtonColor = true
    minusBtn.ZIndex = 2
    minusBtn.BorderSizePixel = 0
    minusBtn.Parent = panel
    local uicornerMinus = Instance.new("UICorner")
    uicornerMinus.CornerRadius = UDim.new(1, 0)
    uicornerMinus.Parent = minusBtn

    -- Botão "+" para aumentar FOV (sem borda)
    local plusBtn = Instance.new("TextButton")
    plusBtn.Size = UDim2.new(0, 40, 0, 40)
    plusBtn.Position = UDim2.new(0.83, 0, 0.80, 0)
    plusBtn.AnchorPoint = Vector2.new(0.5, 0.5)
    plusBtn.Text = "+"
    plusBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    plusBtn.TextColor3 = Color3.fromRGB(255,255,255)
    plusBtn.TextSize = 28
    plusBtn.Font = Enum.Font.SourceSansBold
    plusBtn.AutoButtonColor = true
    plusBtn.ZIndex = 2
    plusBtn.BorderSizePixel = 0
    plusBtn.Parent = panel
    local uicornerPlus = Instance.new("UICorner")
    uicornerPlus.CornerRadius = UDim.new(1, 0)
    uicornerPlus.Parent = plusBtn

    -- Texto mostrando o valor atual do FOV (com borda preta)
    local fovLabel = Instance.new("TextLabel")
    fovLabel.Size = UDim2.new(0, 80, 0, 32)
    fovLabel.Position = UDim2.new(0.5, -40, 0.80, 0)
    fovLabel.BackgroundTransparency = 1
    fovLabel.TextColor3 = Color3.fromRGB(255,255,255)
    fovLabel.TextSize = 22
    fovLabel.Font = Enum.Font.SourceSansBold
    fovLabel.Text = "FOV: 150"
    fovLabel.ZIndex = 2
    fovLabel.Parent = panel
    local fovLabelStroke = Instance.new("UIStroke")
    fovLabelStroke.Color = Color3.new(0,0,0)
    fovLabelStroke.Thickness = 2
    fovLabelStroke.Parent = fovLabel

    -- Lógica de arrastar painel
    local dragging, dragInput, dragStart, startPos
    panel.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = panel.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)
    panel.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)
    UIS.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            local delta = input.Position - dragStart
            panel.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
            lastPanelPosition = panel.Position
        end
    end)

    -- Minimizar painel
    minimizeBtn.MouseButton1Click:Connect(function()
        minimized = true
        lastPanelPosition = panel.Position
        if screenGui then screenGui:Destroy() end
        RunService.RenderStepped:Wait()
        CreateReopenBtn()
    end)

    return {
        button = button,
        minusBtn = minusBtn,
        plusBtn = plusBtn,
        fovLabel = fovLabel,
        panel = panel
    }
end

-- Aimbot / FOV
local aimbotEnabled = false
local aimCircle
local circleRadius = 150
local minRadius = 50
local maxRadius = 400
guiObjects = CreateAimbotPanel()

function updateFovLabel()
    if guiObjects and guiObjects.fovLabel then
        guiObjects.fovLabel.Text = "FOV: "..circleRadius
    end
end

function drawCircle()
    if aimCircle then aimCircle:Remove() end
    aimCircle = Drawing.new("Circle")
    aimCircle.Position = Vector2.new(workspace.CurrentCamera.ViewportSize.X/2, workspace.CurrentCamera.ViewportSize.Y/2)
    aimCircle.Radius = circleRadius
    aimCircle.Thickness = 2
    aimCircle.Color = Color3.fromRGB(100, 200, 255)
    aimCircle.Transparency = 1
    aimCircle.Visible = true
end

function getTarget()
    local camera = workspace.CurrentCamera
    local center = Vector2.new(camera.ViewportSize.X/2, workspace.CurrentCamera.ViewportSize.Y/2)
    local closestPlayer, closestDist = nil, circleRadius

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local headPos = camera:WorldToViewportPoint(player.Character.Head.Position)
            local screenPos = Vector2.new(headPos.X, headPos.Y)
            local dist = (screenPos - center).Magnitude
            if dist < closestDist then
                closestDist = dist
                closestPlayer = player
            end
        end
    end
    return closestPlayer
end

function updateConnections()
    if not guiObjects then return end
    guiObjects.plusBtn.MouseButton1Click:Connect(function()
        circleRadius = math.min(circleRadius + 10, maxRadius)
        updateFovLabel()
        if aimbotEnabled then drawCircle() end
    end)
    guiObjects.minusBtn.MouseButton1Click:Connect(function()
        circleRadius = math.max(circleRadius - 10, minRadius)
        updateFovLabel()
        if aimbotEnabled then drawCircle() end
    end)
    guiObjects.button.MouseButton1Click:Connect(function()
        aimbotEnabled = not aimbotEnabled
        guiObjects.button.Text = aimbotEnabled and "Desativar Aimbot" or "Ativar Aimbot"
        guiObjects.button.BackgroundColor3 = aimbotEnabled and Color3.fromRGB(200,30,30) or Color3.fromRGB(30,200,40)
        if aimbotEnabled then
            drawCircle()
        else
            if aimCircle then aimCircle:Remove() aimCircle = nil end
        end
    end)
end

updateConnections()

local ShootRemote = nil
for _,v in pairs(getgc(true)) do
    if typeof(v) == "Instance" and v:IsA("RemoteEvent") and v.Name:lower():find("shoot") then
        ShootRemote = v
        break
    end
end

UIS.InputBegan:Connect(function(input, gp)
    if not aimbotEnabled then return end
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        local target = getTarget()
        if target and target.Character and target.Character:FindFirstChild("Head") then
            if ShootRemote then
                ShootRemote:FireServer(target.Character.Head.Position)
            else
                local camera = workspace.CurrentCamera
                camera.CFrame = CFrame.new(camera.CFrame.Position, target.Character.Head.Position)
            end
        end
    end
end)

RunService.RenderStepped:Connect(function()
    -- O círculo fica visível quando aimbot está ativado, independentemente do painel estar aberto ou minimizado
    if aimCircle and aimbotEnabled then
        aimCircle.Position = Vector2.new(workspace.CurrentCamera.ViewportSize.X/2, workspace.CurrentCamera.ViewportSize.Y/2)
        aimCircle.Radius = circleRadius
        aimCircle.Visible = true
    elseif aimCircle then
        aimCircle.Visible = false
    end
end)

-- Mantém GUI ao morrer/reaparecer e respeita minimizado e posição
Players.LocalPlayer.CharacterAdded:Connect(function()
    while not LocalPlayer:FindFirstChild("PlayerGui") do task.wait() end
    if minimized then
        removeGuiByName("AimbotGUI")
        removeGuiByName("ReabrirBtn")
        CreateReopenBtn()
    else
        guiObjects = CreateAimbotPanel()
        updateConnections()
        updateFovLabel()
    end
end)

updateFovLabel()
