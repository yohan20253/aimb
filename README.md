--// Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

--// Variáveis
local ESPEnabled = false
local ESPBoxes = {}

--// Criar GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ESPGui"
ScreenGui.Parent = PlayerGui

local ToggleButton = Instance.new("TextButton")
ToggleButton.Size = UDim2.new(0, 120, 0, 40)
ToggleButton.Position = UDim2.new(0, 10, 0, 10)
ToggleButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
ToggleButton.TextColor3 = Color3.new(1, 1, 1)
ToggleButton.Text = "Ativar ESP"
ToggleButton.Parent = ScreenGui

--// Função para criar a box de ESP em cada personagem
local function CreateESPBox(character)
    local box = Instance.new("BoxHandleAdornment")
    box.Adornee = character:FindFirstChild("HumanoidRootPart")
    box.AlwaysOnTop = true
    box.ZIndex = 10
    box.Size = Vector3.new(4, 6, 1)
    box.Transparency = 0.5
    box.Color3 = Color3.new(1, 0, 0) -- Vermelho por padrão
    box.Parent = character:FindFirstChild("HumanoidRootPart")
    return box
end

--// Atualizar as caixas ESP para todos os jogadores inimigos
local function UpdateESP()
    -- Limpar ESPBoxes para jogadores que saíram ou estão no mesmo time
    for player, box in pairs(ESPBoxes) do
        if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") or player.Team == LocalPlayer.Team then
            if box and box.Parent then
                box:Destroy()
            end
            ESPBoxes[player] = nil
        end
    end

    if ESPEnabled then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Team ~= LocalPlayer.Team then
                if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                    if not ESPBoxes[player] then
                        ESPBoxes[player] = CreateESPBox(player.Character)
                    end
                end
            end
        end
    else
        -- Se ESP desligado, destruir todas as boxes
        for player, box in pairs(ESPBoxes) do
            if box and box.Parent then
                box:Destroy()
            end
        end
        ESPBoxes = {}
    end
end

--// Toggle ESP
ToggleButton.MouseButton1Click:Connect(function()
    ESPEnabled = not ESPEnabled
    ToggleButton.Text = ESPEnabled and "Desativar ESP" or "Ativar ESP"
    UpdateESP()
end)

--// Atualiza constantemente enquanto ESP estiver ativo (para acompanhar respawns e time changes)
RunService.Heartbeat:Connect(function()
    if ESPEnabled then
        UpdateESP()
    end
end)

--// Limpar ESP ao sair do jogo
Players.PlayerRemoving:Connect(function(player)
    if ESPBoxes[player] then
        ESPBoxes[player]:Destroy()
        ESPBoxes[player] = nil
    end
end)
