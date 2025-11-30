-- Script Blox Fruits Universal - Mobile & Delta
-- Compatível com todos os executores

local Player = game:GetService("Players").LocalPlayer
local Character = Player.Character or Player.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")

-- Função de segurança anti-ban
local function SafeCall(func, ...)
    local success, result = pcall(func, ...)
    if not success then
        warn("Erro seguro: " .. tostring(result))
    end
    return result
end

-- Auto Farm de Levels
local function AutoFarm()
    while getgenv().AutoFarmEnabled do
        task.wait()
        SafeCall(function()
            -- Encontrar NPCs mais próximos
            local nearestNPC = nil
            local nearestDistance = math.huge
            
            for _, npc in pairs(workspace.Enemies:GetChildren()) do
                if npc:FindFirstChild("Humanoid") and npc.Humanoid.Health > 0 then
                    local distance = (Character.HumanoidRootPart.Position - npc.HumanoidRootPart.Position).Magnitude
                    if distance < nearestDistance then
                        nearestDistance = distance
                        nearestNPC = npc
                    end
                end
            end
            
            if nearestNPC then
                -- Teleportar para o NPC
                Character.HumanoidRootPart.CFrame = nearestNPC.HumanoidRootPart.CFrame * CFrame.new(0, 0, 5)
                
                -- Atacar automaticamente
                game:GetService("VirtualInputManager"):SendKeyEvent(true, "X", false, game)
                task.wait(0.1)
                game:GetService("VirtualInputManager"):SendKeyEvent(false, "X", false, game)
            end
        end)
    end
end

-- Coletar Frutas Automaticamente
local function AutoCollectFruits()
    while getgenv().AutoFruitsEnabled do
        task.wait(2)
        SafeCall(function()
            for _, fruit in pairs(workspace:GetChildren()) do
                if string.find(fruit.Name, "Fruit") and fruit:IsA("Model") then
                    Character.HumanoidRootPart.CFrame = fruit.Handle.CFrame
                    task.wait(0.5)
                end
            end
        end)
    end
end

-- Farm Automático de Beli
local function AutoBeliFarm()
    while getgenv().AutoBeliEnabled do
        task.wait()
        SafeCall(function()
            -- Procurar por chests
            for _, chest in pairs(workspace:GetChildren()) do
                if chest.Name == "Chest" or string.find(chest.Name, "Chest") then
                    Character.HumanoidRootPart.CFrame = chest.CFrame
                    task.wait(1)
                end
            end
        end)
    end
end

-- Teleport para Ilhas
local Islands = {
    ["Starter Island"] = CFrame.new(100, 50, 100),
    ["Jungle"] = CFrame.new(-1500, 60, 300),
    ["Pirate Village"] = CFrame.new(-1100, 70, 3800),
    ["Desert"] = CFrame.new(900, 10, 4100),
    ["Frozen Village"] = CFrame.new(1100, 100, -4200),
    ["Marine Fortress"] = CFrame.new(-4800, 20, 800)
}

local function TeleportToIsland(islandName)
    if Islands[islandName] then
        Character.HumanoidRootPart.CFrame = Islands[islandName]
    end
end

-- Auto Haki
local function AutoHaki()
    while getgenv().AutoHakiEnabled do
        task.wait(10)
        SafeCall(function()
            game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer("Buso")
        end)
    end
end

-- Interface Gráfica
local function CreateGUI()
    local library = loadstring(game:HttpGet("https://raw.githubusercontent.com/wally-rblx/roblox-ui-libs/main/uwuware/source.lua"))()
    
    local window = library:CreateWindow("Blox Fruits Universal") {
        main_color = Color3.fromRGB(41, 74, 122),
        min_size = Vector2.new(370, 380),
        toggle_key = Enum.KeyCode.RightShift,
        callback = function() end
    }
    
    -- Tab Principal
    local mainTab = window:AddTab("Auto Farm")
    
    mainTab:AddToggle("Auto Farm Level", {
        flag = "auto_farm",
        callback = function(value)
            getgenv().AutoFarmEnabled = value
            if value then
                coroutine.wrap(AutoFarm)()
            end
        end
    })
    
    mainTab:AddToggle("Auto Coletar Frutas", {
        flag = "auto_fruits",
        callback = function(value)
            getgenv().AutoFruitsEnabled = value
            if value then
                coroutine.wrap(AutoCollectFruits)()
            end
        end
    })
    
    mainTab:AddToggle("Auto Farm Beli", {
        flag = "auto_beli",
        callback = function(value)
            getgenv().AutoBeliEnabled = value
            if value then
                coroutine.wrap(AutoBeliFarm)()
            end
        end
    })
    
    mainTab:AddToggle("Auto Haki", {
        flag = "auto_haki",
        callback = function(value)
            getgenv().AutoHakiEnabled = value
            if value then
                coroutine.wrap(AutoHaki)()
            end
        end
    })
    
    -- Tab Teleport
    local teleportTab = window:AddTab("Teleport")
    
    for islandName, _ in pairs(Islands) do
        teleportTab:AddButton("Ir para " .. islandName, function()
            TeleportToIsland(islandName)
        end)
    end
    
    -- Tab Player
    local playerTab = window:AddTab("Player")
    
    playerTab:AddSlider("Velocidade de Caminhada", {
        min = 16,
        max = 200,
        default = 16,
        callback = function(value)
            Humanoid.WalkSpeed = value
        end
    })
    
    playerTab:AddSlider("Força do Puloo", {
        min = 50,
        max = 200,
        default = 50,
        callback = function(value)
            Humanoid.JumpPower = value
        end
    })
    
    playerTab:AddToggle("Noclip", {
        flag = "noclip",
        callback = function(value)
            getgenv().NoclipEnabled = value
            if value then
                coroutine.wrap(function()
                    while getgenv().NoclipEnabled do
                        task.wait()
                        for _, part in pairs(Character:GetDescendants()) do
                            if part:IsA("BasePart") then
                                part.CanCollide = false
                            end
                        end
                    end
                end)()
            end
        end
    })
    
    -- Tab Misc
    local miscTab = window:AddTab("Misc")
    
    miscTab:AddButton("Coletar Todas as Frutas", function()
        for _, fruit in pairs(workspace:GetChildren()) do
            if string.find(fruit.Name, "Fruit") then
                Character.HumanoidRootPart.CFrame = fruit.Handle.CFrame
                task.wait(0.5)
            end
        end
    end)
    
    miscTab:AddButton("Reiniciar Personagem", function()
        Character:BreakJoints()
    end)
    
    library:Init()
end

-- Inicialização Segura
local function Initialize()
    -- Verificar se o jogo é Blox Fruits
    if game.PlaceId ~= 2753915549 and game.PlaceId ~= 4442272183 and game.PlaceId ~= 7449423635 then
        warn("Este script é apenas para Blox Fruits!")
        return
    end
    
    -- Configurações padrão
    getgenv().AutoFarmEnabled = false
    getgenv().AutoFruitsEnabled = false
    getgenv().AutoBeliEnabled = false
    getgenv().AutoHakiEnabled = false
    getgenv().NoclipEnabled = false
    
    -- Esperar o personagem carregar
    if not Character then
        Character = Player.CharacterAdded:Wait()
    end
    
    -- Criar interface
    SafeCall(CreateGUI)
    
    -- Mensagem de sucesso
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Blox Fruits Universal",
        Text = "Script carregado com sucesso!",
        Duration = 5
    })
end

-- Iniciar script
SafeCall(Initialize)
