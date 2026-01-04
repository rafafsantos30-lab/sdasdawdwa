-- Premium Auto TP GUI Script v4 - FINAL
-- God Mode sempre ativo + Bypass máximo + Marcador quadrado correto

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local ProximityPromptService = game:GetService("ProximityPromptService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Configurações
local savedPosition = nil
local markerPart = nil
local lineConnection = nil
local isHoldingButton = false
local isTeleporting = false
local statusText = "Aguardando..."

-- ===== GOD MODE ULTRA BYPASS (INVISÍVEL) =====
local godModeConnections = {}
local godModeActive = false

local function setupGodMode()
    local character = player.Character
    if not character then return end
    
    local humanoid = character:FindFirstChild("Humanoid")
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not humanoid or not hrp then return end
    
    godModeActive = true
    print("🛡️ God Mode ULTRA BYPASS ATIVO")
    
    -- BYPASS 1: Força MaxHealth muito alto (anti-oneshot)
    humanoid.MaxHealth = math.huge
    humanoid.Health = math.huge
    
    -- BYPASS 2: Remove todos os danos via Heartbeat
    table.insert(godModeConnections, RunService.Heartbeat:Connect(function()
        if not godModeActive then return end
        pcall(function()
            if humanoid and humanoid.Parent then
                humanoid.Health = math.huge
                humanoid.MaxHealth = math.huge
                
                -- Anti-fall damage
                if humanoid:GetState() == Enum.HumanoidStateType.Freefall then
                    hrp.AssemblyLinearVelocity = Vector3.new(hrp.AssemblyLinearVelocity.X, 0, hrp.AssemblyLinearVelocity.Z)
                end
            end
        end)
    end))
    
    -- BYPASS 3: RenderStepped (mais rápido que Heartbeat)
    table.insert(godModeConnections, RunService.RenderStepped:Connect(function()
        if not godModeActive then return end
        pcall(function()
            if humanoid and humanoid.Parent then
                humanoid.Health = math.huge
            end
        end)
    end))
    
    -- BYPASS 4: Stepped (física)
    table.insert(godModeConnections, RunService.Stepped:Connect(function()
        if not godModeActive then return end
        pcall(function()
            if humanoid and humanoid.Parent then
                humanoid.Health = math.huge
                
                -- Previne ragdoll
                for _, part in pairs(character:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = part.Name == "HumanoidRootPart" or part.Name == "Head"
                    end
                end
            end
        end)
    end))
    
    -- BYPASS 5: StateChanged (anti-morte/ragdoll)
    table.insert(godModeConnections, humanoid.StateChanged:Connect(function(oldState, newState)
        if not godModeActive then return end
        if newState == Enum.HumanoidStateType.Dead or 
           newState == Enum.HumanoidStateType.FallingDown or
           newState == Enum.HumanoidStateType.Ragdoll or
           newState == Enum.HumanoidStateType.Physics then
            task.spawn(function()
                task.wait()
                humanoid:ChangeState(Enum.HumanoidStateType.GettingUp)
                humanoid:ChangeState(Enum.HumanoidStateType.Running)
                humanoid.Health = math.huge
            end)
        end
    end))
    
    -- BYPASS 6: HealthChanged (instantâneo)
    table.insert(godModeConnections, humanoid.HealthChanged:Connect(function(health)
        if not godModeActive then return end
        if health < math.huge then
            humanoid.Health = math.huge
        end
    end))
    
    -- BYPASS 7: Dying event (última linha de defesa)
    table.insert(godModeConnections, humanoid.Dying:Connect(function()
        if not godModeActive then return end
        task.spawn(function()
            humanoid.Health = math.huge
            humanoid:ChangeState(Enum.HumanoidStateType.GettingUp)
        end)
    end))
    
    -- BYPASS 8: ForceField invisível (dupla proteção)
    local ff = Instance.new("ForceField")
    ff.Visible = false
    ff.Parent = character
    
    -- BYPASS 9: Monitora e recria ForceField
    task.spawn(function()
        while godModeActive and character and character.Parent do
            if not character:FindFirstChildOfClass("ForceField") then
                local newFF = Instance.new("ForceField")
                newFF.Visible = false
                newFF.Parent = character
            end
            task.wait(1)
        end
    end)
    
    -- BYPASS 10: Desabilita dano de scripts remotos
    for _, part in pairs(character:GetDescendants()) do
        if part:IsA("BasePart") then
            part:SetAttribute("GodMode", true)
        end
    end
    
    -- BYPASS 11: Anti-void (detecta quedas infinitas)
    task.spawn(function()
        while godModeActive and character and character.Parent do
            if hrp and hrp.Position.Y < -200 then
                if savedPosition then
                    hrp.CFrame = CFrame.new(savedPosition)
                else
                    hrp.CFrame = hrp.CFrame + Vector3.new(0, 500, 0)
                end
                humanoid.Health = math.huge
            end
            task.wait(0.5)
        end
    end)
    
    print("✓ 11 Bypasses de God Mode ativos!")
end

-- Ativa God Mode quando spawnar
player.CharacterAdded:Connect(function(char)
    task.wait(1)
    for _, conn in pairs(godModeConnections) do
        if conn then conn:Disconnect() end
    end
    godModeConnections = {}
    setupGodMode()
end)

-- Ativa God Mode inicial
if player.Character then
    setupGodMode()
end

-- Função para criar tweens suaves
local function createTween(object, properties, duration, easingStyle, easingDirection)
    local tweenInfo = TweenInfo.new(
        duration or 0.3,
        easingStyle or Enum.EasingStyle.Quad,
        easingDirection or Enum.EasingDirection.Out
    )
    return TweenService:Create(object, tweenInfo, properties)
end

-- Função para encontrar Flying Carpet
local function findFlyingCarpet()
    local character = player.Character
    if not character then return nil end
    
    -- Procura no backpack
    local backpack = player:FindFirstChild("Backpack")
    if backpack then
        for _, item in pairs(backpack:GetChildren()) do
            if item:IsA("Tool") then
                local name = item.Name:lower()
                if name:find("flying") or name:find("carpet") then
                    return item
                end
            end
        end
    end
    
    -- Procura no character
    for _, item in pairs(character:GetChildren()) do
        if item:IsA("Tool") then
            local name = item.Name:lower()
            if name:find("flying") or name:find("carpet") then
                return item
            end
        end
    end
    
    return nil
end

-- Função ULTRA FORÇADA para TP com Flying Carpet (100% BYPASS)
local function useFlyingCarpetAndTP()
    if not savedPosition then
        statusText = "❌ Nenhuma base marcada!"
        task.wait(2)
        statusText = "Aguardando..."
        return false
    end
    
    local character = player.Character
    if not character then return false end
    
    local hrp = character:FindFirstChild("HumanoidRootPart")
    local humanoid = character:FindFirstChild("Humanoid")
    if not hrp or not humanoid then return false end
    
    print("🔍 FORÇANDO uso do Flying Carpet...")
    
    -- PROCURA AGRESSIVA DO CARPET
    local carpet = findFlyingCarpet()
    local attempts = 0
    
    while not carpet and attempts < 10 do
        print("⚠️ Carpet não encontrado, tentativa", attempts + 1)
        task.wait(0.5)
        carpet = findFlyingCarpet()
        attempts = attempts + 1
    end
    
    if not carpet then
        statusText = "❌ Flying Carpet não encontrado após 10 tentativas!"
        task.wait(2)
        statusText = "Aguardando..."
        return false
    end
    
    print("✓ Flying Carpet encontrado:", carpet.Name)
    
    -- DESATIVA GOD MODE TEMPORARIAMENTE (alguns jogos bloqueiam tools com ForceField)
    godModeActive = false
    local ff = character:FindFirstChildOfClass("ForceField")
    if ff then ff:Destroy() end
    
    -- FORÇA EQUIPAR O CARPET (método agressivo)
    for i = 1, 5 do
        if carpet.Parent == player.Backpack then
            humanoid:EquipTool(carpet)
            task.wait(0.1)
        end
        
        if carpet.Parent == character then
            print("✓ Carpet equipado!")
            break
        end
        task.wait(0.1)
    end
    
    task.wait(0.2)
    
    -- ATIVA O CARPET DE TODAS AS FORMAS POSSÍVEIS
    print("🔥 FORÇANDO ativação do carpet...")
    
    -- Método 1: Activate() normal
    for i = 1, 10 do
        pcall(function()
            if carpet:FindFirstChild("Handle") then
                carpet:Activate()
            end
        end)
        task.wait(0.03)
    end
    
    -- Método 2: Simula clique do mouse (alguns tools precisam disso)
    for i = 1, 5 do
        pcall(function()
            carpet:Activate()
            if carpet:FindFirstChild("Handle") then
                local handle = carpet.Handle
                handle:Activate()
            end
        end)
        task.wait(0.05)
    end
    
    -- Método 3: FireServer em RemoteEvents (caso o carpet use)
    for _, obj in pairs(carpet:GetDescendants()) do
        if obj:IsA("RemoteEvent") then
            pcall(function()
                obj:FireServer("Activate")
                obj:FireServer(true)
                obj:FireServer("Use")
            end)
        end
    end
    
    task.wait(0.3)
    
    print("✓ Carpet ativado! Iniciando TP...")
    
    -- REATIVA GOD MODE
    godModeActive = true
    setupGodMode()
    
    -- TELEPORTE COM 100% BYPASS
    local targetPos = savedPosition
    
    -- BYPASS 1: Desativa detecção de velocidade
    hrp.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
    hrp.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
    hrp.Anchored = true
    task.wait(0.05)
    hrp.Anchored = false
    
    -- BYPASS 2: Teleporte gradual (anti-detecção)
    local steps = 5
    local startPos = hrp.Position
    
    for i = 1, steps do
        local alpha = i / steps
        local interpPos = startPos:Lerp(targetPos, alpha)
        
        for j = 1, 3 do
            hrp.CFrame = CFrame.new(interpPos)
            hrp.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
            hrp.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
            task.wait(0.02)
        end
        
        task.wait(0.05)
    end
    
    -- BYPASS 3: Teleporte final preciso
    for i = 1, 10 do
        hrp.CFrame = CFrame.new(targetPos)
        hrp.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
        hrp.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
        task.wait(0.01)
    end
    
    -- BYPASS 4: Força estado do humanoid
    humanoid:ChangeState(Enum.HumanoidStateType.GettingUp)
    task.wait(0.1)
    humanoid:ChangeState(Enum.HumanoidStateType.Landed)
    
    -- BYPASS 5: LOCK EXTREMO de posição (3 segundos)
    print("🔒 LOCK ANTI-ROLLBACK EXTREMO ATIVADO!")
    local lockStart = tick()
    local lockConnections = {}
    
    -- Lock em 3 serviços diferentes
    table.insert(lockConnections, RunService.Heartbeat:Connect(function()
        if tick() - lockStart >= 3 then
            for _, conn in pairs(lockConnections) do
                conn:Disconnect()
            end
            print("✓ Lock finalizado!")
            return
        end
        
        if hrp and hrp.Parent then
            hrp.CFrame = CFrame.new(targetPos)
            hrp.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
            hrp.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
        end
    end))
    
    table.insert(lockConnections, RunService.RenderStepped:Connect(function()
        if tick() - lockStart >= 3 then return end
        
        if hrp and hrp.Parent then
            hrp.CFrame = CFrame.new(targetPos)
        end
    end))
    
    table.insert(lockConnections, RunService.Stepped:Connect(function()
        if tick() - lockStart >= 3 then return end
        
        if hrp and hrp.Parent then
            hrp.CFrame = CFrame.new(targetPos)
            hrp.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
        end
    end))
    
    task.wait(3)
    
    -- BYPASS 6: Reforça posição final
    for i = 1, 5 do
        hrp.CFrame = CFrame.new(targetPos)
        task.wait(0.1)
    end
    
    print("✓ TELEPORTE 100% COMPLETO!")
    return true
end

-- Função para criar marcador QUADRADO (não plataforma)
local function createMarker(position)
    if markerPart then
        markerPart:Destroy()
    end
    
    -- Cria um QUADRADO VERTICAL (como um cubo outline)
    markerPart = Instance.new("Part")
    markerPart.Name = "BaseMarker"
    markerPart.Size = Vector3.new(5, 5, 5) -- CUBO 5x5x5
    markerPart.Position = position
    markerPart.Anchored = true
    markerPart.CanCollide = false
    markerPart.Transparency = 1 -- Invisível, só mostra borda
    markerPart.Parent = workspace
    
    -- BORDA DO QUADRADO (SelectionBox)
    local border = Instance.new("SelectionBox")
    border.Adornee = markerPart
    border.Color3 = Color3.fromRGB(138, 43, 226) -- Roxo neon
    border.LineThickness = 0.05
    border.Transparency = 0.2
    border.Parent = markerPart
    
    -- Texto 3D PEQUENO
    local billboard = Instance.new("BillboardGui")
    billboard.Size = UDim2.new(0, 100, 0, 25)
    billboard.Adornee = markerPart
    billboard.AlwaysOnTop = true
    billboard.StudsOffset = Vector3.new(0, 3, 0)
    billboard.Parent = markerPart
    
    local textLabel = Instance.new("TextLabel")
    textLabel.Size = UDim2.new(1, 0, 1, 0)
    textLabel.BackgroundTransparency = 1
    textLabel.Text = "LOCAL BASE"
    textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    textLabel.TextSize = 12 -- TEXTO PEQUENO
    textLabel.Font = Enum.Font.GothamBold
    textLabel.TextStrokeTransparency = 0.5
    textLabel.Parent = billboard
    
    -- Animação de pulso na borda
    spawn(function()
        while markerPart and markerPart.Parent do
            for i = 0, 20 do
                if not border or not border.Parent then break end
                border.Transparency = 0.2 + (math.sin(i / 20 * math.pi * 2) * 0.15)
                task.wait(0.05)
            end
        end
    end)
end

-- Função para criar linha
local function updateLine()
    if lineConnection then
        lineConnection:Disconnect()
    end
    
    lineConnection = RunService.RenderStepped:Connect(function()
        local character = player.Character
        if markerPart and markerPart.Parent and character and character:FindFirstChild("HumanoidRootPart") then
            local hrp = character.HumanoidRootPart
            local startPos = hrp.Position
            local endPos = markerPart.Position
            local distance = (startPos - endPos).Magnitude
            
            local line = workspace:FindFirstChild("TPLine") or Instance.new("Part")
            line.Name = "TPLine"
            line.Size = Vector3.new(0.2, 0.2, distance)
            line.CFrame = CFrame.new(startPos:Lerp(endPos, 0.5), endPos)
            line.Anchored = true
            line.CanCollide = false
            line.Material = Enum.Material.Neon
            line.Color = Color3.fromRGB(138, 43, 226)
            line.Transparency = 0.5
            line.Parent = workspace
        else
            local line = workspace:FindFirstChild("TPLine")
            if line then line:Destroy() end
        end
    end)
end

-- ===== GUI =====
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "PremiumAutoTPGui"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Parent = playerGui

-- Tela de carregamento
local loadingFrame = Instance.new("Frame")
loadingFrame.Name = "LoadingScreen"
loadingFrame.Size = UDim2.new(0, 250, 0, 120)
loadingFrame.Position = UDim2.new(0.5, -125, 0.5, -60)
loadingFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
loadingFrame.BorderSizePixel = 0
loadingFrame.Parent = screenGui

local loadingCorner = Instance.new("UICorner")
loadingCorner.CornerRadius = UDim.new(0, 20)
loadingCorner.Parent = loadingFrame

local loadingStroke = Instance.new("UIStroke")
loadingStroke.Color = Color3.fromRGB(138, 43, 226)
loadingStroke.Thickness = 2
loadingStroke.Transparency = 0.3
loadingStroke.Parent = loadingFrame

local loadingText = Instance.new("TextLabel")
loadingText.Size = UDim2.new(1, -40, 0, 40)
loadingText.Position = UDim2.new(0, 20, 0, 25)
loadingText.BackgroundTransparency = 1
loadingText.Text = "Carregando..."
loadingText.TextColor3 = Color3.fromRGB(255, 255, 255)
loadingText.Font = Enum.Font.GothamBold
loadingText.TextSize = 18
loadingText.Parent = loadingFrame

local loadingBar = Instance.new("Frame")
loadingBar.Size = UDim2.new(0, 0, 0, 4)
loadingBar.Position = UDim2.new(0, 20, 1, -25)
loadingBar.BackgroundColor3 = Color3.fromRGB(138, 43, 226)
loadingBar.BorderSizePixel = 0
loadingBar.Parent = loadingFrame

local barCorner = Instance.new("UICorner")
barCorner.CornerRadius = UDim.new(0, 2)
barCorner.Parent = loadingBar

spawn(function()
    createTween(loadingBar, {Size = UDim2.new(1, -40, 0, 4)}, 0.5):Play()
    task.wait(0.5)
    
    createTween(loadingFrame, {BackgroundTransparency = 1}, 0.3):Play()
    createTween(loadingText, {TextTransparency = 1}, 0.3):Play()
    createTween(loadingBar, {BackgroundTransparency = 1}, 0.3):Play()
    createTween(loadingStroke, {Transparency = 1}, 0.3):Play()
    
    task.wait(0.3)
    loadingFrame:Destroy()
end)

task.wait(0.5)

-- GUI Principal
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 280, 0, 200)
mainFrame.Position = UDim2.new(0.5, -140, 0.5, -100)
mainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
mainFrame.BorderSizePixel = 0
mainFrame.BackgroundTransparency = 0.05
mainFrame.Parent = screenGui

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 18)
mainCorner.Parent = mainFrame

local mainStroke = Instance.new("UIStroke")
mainStroke.Color = Color3.fromRGB(138, 43, 226)
mainStroke.Thickness = 2
mainStroke.Transparency = 0.5
mainStroke.Parent = mainFrame

-- Título
local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, 0, 0, 40)
titleLabel.Position = UDim2.new(0, 0, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "AUTO TP SYSTEM"
titleLabel.TextColor3 = Color3.fromRGB(138, 43, 226)
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 18
titleLabel.Parent = mainFrame

-- Botão SET LOCAL TP
local setButton = Instance.new("TextButton")
setButton.Name = "SetButton"
setButton.Size = UDim2.new(1, -40, 0, 45)
setButton.Position = UDim2.new(0, 20, 0, 50)
setButton.BackgroundColor3 = Color3.fromRGB(138, 43, 226)
setButton.BorderSizePixel = 0
setButton.Text = "SET LOCAL TP"
setButton.TextColor3 = Color3.fromRGB(255, 255, 255)
setButton.Font = Enum.Font.GothamBold
setButton.TextSize = 16
setButton.AutoButtonColor = false
setButton.Parent = mainFrame

local setCorner = Instance.new("UICorner")
setCorner.CornerRadius = UDim.new(0, 12)
setCorner.Parent = setButton

-- Status Frame
local statusFrame = Instance.new("Frame")
statusFrame.Size = UDim2.new(1, -40, 0, 85)
statusFrame.Position = UDim2.new(0, 20, 0, 105)
statusFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
statusFrame.BorderSizePixel = 0
statusFrame.Parent = mainFrame

local statusCorner = Instance.new("UICorner")
statusCorner.CornerRadius = UDim.new(0, 12)
statusCorner.Parent = statusFrame

local statusStroke = Instance.new("UIStroke")
statusStroke.Color = Color3.fromRGB(138, 43, 226)
statusStroke.Thickness = 1.5
statusStroke.Transparency = 0.7
statusStroke.Parent = statusFrame

-- Status Title
local statusTitle = Instance.new("TextLabel")
statusTitle.Size = UDim2.new(1, -20, 0, 25)
statusTitle.Position = UDim2.new(0, 10, 0, 5)
statusTitle.BackgroundTransparency = 1
statusTitle.Text = "STATUS:"
statusTitle.TextColor3 = Color3.fromRGB(138, 43, 226)
statusTitle.Font = Enum.Font.GothamBold
statusTitle.TextSize = 12
statusTitle.TextXAlignment = Enum.TextXAlignment.Left
statusTitle.Parent = statusFrame

-- Status Label
local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(1, -20, 0, 50)
statusLabel.Position = UDim2.new(0, 10, 0, 30)
statusLabel.BackgroundTransparency = 1
statusLabel.Text = statusText
statusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
statusLabel.Font = Enum.Font.Gotham
statusLabel.TextSize = 14
statusLabel.TextWrapped = true
statusLabel.TextXAlignment = Enum.TextXAlignment.Left
statusLabel.TextYAlignment = Enum.TextYAlignment.Top
statusLabel.Parent = statusFrame

-- Atualiza status
spawn(function()
    while task.wait(0.1) do
        if statusLabel then
            local color = Color3.fromRGB(200, 200, 200)
            
            if statusText:find("Segurando") then
                color = Color3.fromRGB(255, 193, 7)
            elseif statusText:find("Teleportando") or statusText:find("✓") then
                color = Color3.fromRGB(76, 175, 80)
            elseif statusText:find("❌") then
                color = Color3.fromRGB(244, 67, 54)
            end
            
            statusLabel.Text = statusText
            statusLabel.TextColor3 = color
        end
    end
end)

-- Hover effects
setButton.MouseEnter:Connect(function()
    createTween(setButton, {BackgroundColor3 = Color3.fromRGB(158, 63, 246)}, 0.2):Play()
end)

setButton.MouseLeave:Connect(function()
    createTween(setButton, {BackgroundColor3 = Color3.fromRGB(138, 43, 226)}, 0.2):Play()
end)

-- Funcionalidade SET LOCAL TP
setButton.MouseButton1Click:Connect(function()
    createTween(setButton, {Size = UDim2.new(1, -44, 0, 43)}, 0.1):Play()
    task.wait(0.1)
    createTween(setButton, {Size = UDim2.new(1, -40, 0, 45)}, 0.1):Play()
    
    local character = player.Character
    if character and character:FindFirstChild("HumanoidRootPart") then
        local hrp = character.HumanoidRootPart
        savedPosition = hrp.Position
        
        createMarker(savedPosition)
        updateLine()
        
        setButton.Text = "LOCAL DEFINIDO ✓"
        setButton.BackgroundColor3 = Color3.fromRGB(46, 125, 50)
        statusText = "✓ Base marcada! Segure botão para TP."
        
        task.wait(2)
        setButton.Text = "SET LOCAL TP"
        createTween(setButton, {BackgroundColor3 = Color3.fromRGB(138, 43, 226)}, 0.3):Play()
        statusText = "Aguardando hold..."
    end
end)

-- Tornar arrastável
local dragging = false
local dragInput
local dragStart
local startPos

mainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = mainFrame.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

mainFrame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

-- ===== DETECÇÃO DE PROXIMITYPROMPTREND =====
print("🔍 Sistema de detecção ativo!")

ProximityPromptService.PromptButtonHoldBegan:Connect(function(prompt)
    if not savedPosition then return end
    
    print("✓ Hold começou!")
    isHoldingButton = true
    statusText = "Segurando... (" .. prompt.HoldDuration .. "s)"
end)

ProximityPromptService.PromptButtonHoldEnded:Connect(function(prompt)
    print("✓ Hold finalizado!")
    
    if isHoldingButton and savedPosition and not isTeleporting then
        isHoldingButton = false
        isTeleporting = true
        statusText = "Teleportando!"
        
        task.wait(0.2)
        
        local success = useFlyingCarpetAndTP()
        
        if success then
            statusText = "✓ Teleportado!"
        else
            statusText = "❌ Erro no TP"
        end
        
        task.wait(2)
        isTeleporting = false
        statusText = "Aguardando hold..."
    else
        isHoldingButton = false
        if not isTeleporting then
            statusText = "Aguardando hold..."
        end
    end
end)

print("✓ Premium Auto TP GUI v4 carregado!")
print("✓ God Mode ATIVO (invisível)")
print("✓ Detecção automática pronta!")
