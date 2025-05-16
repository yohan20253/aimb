--// Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera

local LocalPlayer = Players.LocalPlayer

--// Configurações
local AimbotAtivo = false
local TeclaAimbot = Enum.UserInputType.MouseButton2 -- botão direito do mouse
local ParteParaMirar = "Head" -- ou "HumanoidRootPart"
local ChecarTime = true
local DistanciaMaximaDaTela = 120 -- FOV em pixels

--// Função para encontrar o inimigo mais próximo
local function PegarAlvoMaisProximo()
    local alvoMaisProximo = nil
    local menorDistancia = DistanciaMaximaDaTela

    for _, jogador in pairs(Players:GetPlayers()) do
        if jogador ~= LocalPlayer and jogador.Character and jogador.Character:FindFirstChild(ParteParaMirar) then
            if not ChecarTime or jogador.Team ~= LocalPlayer.Team then
                local parte = jogador.Character[ParteParaMirar]
                local posicaoNaTela, visivel = Camera:WorldToViewportPoint(parte.Position)
                if visivel then
                    local centro = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
                    local distancia = (Vector2.new(posicaoNaTela.X, posicaoNaTela.Y) - centro).Magnitude
                    if distancia < menorDistancia then
                        menorDistancia = distancia
                        alvoMaisProximo = parte
                    end
                end
            end
        end
    end

    return alvoMaisProximo
end

--// Atualiza a mira automaticamente
RunService.RenderStepped:Connect(function()
    if AimbotAtivo then
        local alvo = PegarAlvoMaisProximo()
        if alvo then
            local direcao = (alvo.Position - Camera.CFrame.Position).Unit
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, Camera.CFrame.Position + direcao)
        end
    end
end)

--// Ativa e desativa o aimbot quando segurar o botão direito do mouse
UserInputService.InputBegan:Connect(function(tecla, processado)
    if tecla.UserInputType == TeclaAimbot then
        AimbotAtivo = true
    end
end)

UserInputService.InputEnded:Connect(function(tecla, processado)
    if tecla.UserInputType == TeclaAimbot then
        AimbotAtivo = false
    end
end)
