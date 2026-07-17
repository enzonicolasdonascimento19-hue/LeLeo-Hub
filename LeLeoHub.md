# LeLeo-Hub-- LeLeo Hub - Universal Script for Delta Executor (Mobile)
-- Interface: Fluent UI - Pequena e Estática

repeat task.wait() until game:IsLoaded()

local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()

local Window = Fluent:CreateWindow({
    Title = "LeLeo Hub",
    SubTitle = "Universal • Mobile",
    TabWidth = 140,
    Size = UDim2.fromOffset(420, 380),
    Acrylic = false,
    Theme = "Dark",
    MinimizeKey = Enum.KeyCode.RightControl
})

local MainTab = Window:AddTab({ Title = "Main", Icon = "home" })
local VisualsTab = Window:AddTab({ Title = "Visuals", Icon = "eye" })

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

-- Variáveis
local wsValue = 16
local jpValue = 50
local noclipEnabled = false
local invisibleEnabled = false
local espEnabled = false

-- Funções
local function setWalkSpeed(speed)
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        LocalPlayer.Character.Humanoid.WalkSpeed = speed
    end
end

local function setJumpPower(power)
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        LocalPlayer.Character.Humanoid.JumpPower = power
    end
end

local noclipConnection
local function toggleNoclip(state)
    noclipEnabled = state
    if noclipEnabled then
        noclipConnection = RunService.Stepped:Connect(function()
            if LocalPlayer.Character then
                for _, part in pairs(LocalPlayer.Character:GetDescendants()) do
                    if part:IsA("BasePart") and part.CanCollide then
                        part.CanCollide = false
                    end
                end
            end
        end)
    else
        if noclipConnection then noclipConnection:Disconnect() end
        if LocalPlayer.Character then
            for _, part in pairs(LocalPlayer.Character:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = true
                end
            end
        end
    end
end

local invisibleConnection
local originalTransparency = {}
local function toggleInvisible(state)
    invisibleEnabled = state
    if not LocalPlayer.Character then return end
    
    if invisibleEnabled then
        for _, part in pairs(LocalPlayer.Character:GetDescendants()) do
            if part:IsA("BasePart") or part:IsA("MeshPart") or part:IsA("Decal") then
                originalTransparency[part] = part.Transparency
                part.Transparency = 1
            end
        end
        invisibleConnection = RunService.RenderStepped:Connect(function()
            if LocalPlayer.Character then
                for _, part in pairs(LocalPlayer.Character:GetDescendants()) do
                    if (part:IsA("BasePart") or part:IsA("MeshPart")) and part.Transparency ~= 1 then
                        part.Transparency = 1
                    end
                end
            end
        end)
    else
        if invisibleConnection then invisibleConnection:Disconnect() end
        for part, trans in pairs(originalTransparency) do
            if part and part.Parent then
                part.Transparency = trans
            end
        end
        originalTransparency = {}
    end
end

local espObjects = {}
local function toggleESP(state)
    espEnabled = state
    if not espEnabled then
        for _, obj in pairs(espObjects) do
            if obj then obj:Destroy() end
        end
        espObjects = {}
        return
    end
    
    local function createESP(plr)
        if plr == LocalPlayer then return end
        local char = plr.Character
        if not char then return end
        
        local box = Instance.new("BoxHandleAdornment")
        box.Size = char:GetExtentsSize() + Vector3.new(0.5, 0.5, 0.5)
        box.Adornee = char:FindFirstChild("HumanoidRootPart") or char.PrimaryPart
        box.AlwaysOnTop = true
        box.ZIndex = 10
        box.Transparency = 0.7
        box.Color3 = Color3.fromRGB(255, 0, 0)
        box.Parent = Workspace.Terrain
        
        local nameTag = Instance.new("BillboardGui")
        nameTag.Adornee = char:FindFirstChild("Head")
        nameTag.Size = UDim2.new(0, 200, 0, 50)
        nameTag.StudsOffset = Vector3.new(0, 3, 0)
        nameTag.AlwaysOnTop = true
        nameTag.Parent = Workspace.Terrain
        
        local text = Instance.new("TextLabel")
        text.Size = UDim2.new(1, 0, 1, 0)
        text.BackgroundTransparency = 1
        text.Text = plr.Name .. " | " .. (char:FindFirstChild("Humanoid") and math.floor(char.Humanoid.Health) or "?")
        text.TextColor3 = Color3.new(1, 1, 1)
        text.TextScaled = true
        text.Font = Enum.Font.GothamBold
        text.Parent = nameTag
        
        espObjects[plr] = {box = box, tag = nameTag}
    end
    
    for _, plr in pairs(Players:GetPlayers()) do
        createESP(plr)
    end
    
    Players.PlayerAdded:Connect(function(plr)
        plr.CharacterAdded:Connect(function() task.wait(1) createESP(plr) end)
    end)
end

-- ==================== INTERFACE ====================

MainTab:AddSection("Movement")

MainTab:AddSlider("WalkSpeed", {
    Title = "WalkSpeed",
    Description = "Máximo 100",
    Default = 16,
    Min = 16,
    Max = 100,
    Rounding = 0,
    Callback = function(value)
        wsValue = value
        setWalkSpeed(value)
    end
})

MainTab:AddSlider("JumpPower", {
    Title = "JumpPower",
    Description = "Máximo 150",
    Default = 50,
    Min = 50,
    Max = 150,
    Rounding = 0,
    Callback = function(value)
        jpValue = value
        setJumpPower(value)
    end
})

MainTab:AddToggle("Noclip", {
    Title = "Noclip",
    Default = false,
    Callback = function(state)
        toggleNoclip(state)
    end
})

MainTab:AddToggle("Invisible", {
    Title = "Invisible (Client)",
    Default = false,
    Callback = function(state)
        toggleInvisible(state)
    end
})

VisualsTab:AddSection("Visuals")

VisualsTab:AddToggle("ESP", {
    Title = "Player ESP",
    Default = false,
    Callback = function(state)
        toggleESP(state)
    end
})

-- Reset ao respawn
LocalPlayer.CharacterAdded:Connect(function()
    task.wait(1)
    if wsValue and wsValue ~= 16 then setWalkSpeed(wsValue) end
    if jpValue and jpValue ~= 50 then setJumpPower(jpValue) end
    if noclipEnabled then toggleNoclip(true) end
    if invisibleEnabled then toggleInvisible(true) end
end)

Fluent:Notify({
    Title = "LeLeo Hub",
    Content = "Carregado com sucesso! (Sem aba Settings)",
    Duration = 5
})

print("✅ LeLeo Hub carregado - Versão sem Settings")
