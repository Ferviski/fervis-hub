local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local Mouse = LocalPlayer:GetMouse()

-- CONFIGURAÇÕES
local AimbotFOV = 150
local AimbotEnabled = true
local TeamCheck = true
local EnemyColor = Color3.fromRGB(255, 0, 0)
local AllyColor = Color3.fromRGB(0, 170, 255)

-- CÍRCULO DO FOV
local FOVCircle = Drawing.new("Circle")
FOVCircle.Radius = AimbotFOV
FOVCircle.Thickness = 1
FOVCircle.Position = Vector2.new(Mouse.X, Mouse.Y + 36)
FOVCircle.Color = Color3.fromRGB(255, 255, 255)
FOVCircle.Visible = true
FOVCircle.Filled = false

RunService.RenderStepped:Connect(function()
    FOVCircle.Position = Vector2.new(Mouse.X, Mouse.Y + 36)
end)

-- FUNÇÃO PARA ESP (CAIXA E NOME)
function createESP(player)
    local box = Drawing.new("Square")
    box.Thickness = 1
    box.Filled = false
    box.Visible = false

    local name = Drawing.new("Text")
    name.Size = 16
    name.Center = true
    name.Outline = true
    name.Font = 2
    name.Visible = false

    RunService.RenderStepped:Connect(function()
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChild("Humanoid") then
            local isEnemy = player.Team ~= LocalPlayer.Team
            local hum = player.Character.Humanoid
            local root = player.Character.HumanoidRootPart

            local pos, onScreen = Camera:WorldToViewportPoint(root.Position)
            if onScreen then
                local scale = 1 / (root.Position - Camera.CFrame.Position).Magnitude * 100
                local sizeX, sizeY = 30 * scale, 60 * scale
                box.Size = Vector2.new(sizeX, sizeY)
                box.Position = Vector2.new(pos.X - sizeX / 2, pos.Y - sizeY / 2)
                box.Color = EnemyColor
                box.Visible = isEnemy -- mostrar apenas se for inimigo

                name.Position = Vector2.new(pos.X, pos.Y - sizeY / 2 - 12)
                name.Text = player.Name
                name.Color = isEnemy and EnemyColor or AllyColor
                name.Visible = true
            else
                box.Visible = false
                name.Visible = false
            end
        else
            box.Visible = false
            name.Visible = false
        end
    end)
end

-- FUNÇÃO PARA PEGAR O INIMIGO MAIS PRÓXIMO DENTRO DO FOV
function getClosestPlayer()
    local closestPlayer = nil
    local shortestDistance = AimbotFOV

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            if not TeamCheck or player.Team ~= LocalPlayer.Team then
                local pos, onScreen = Camera:WorldToViewportPoint(player.Character.HumanoidRootPart.Position)
                if onScreen then
                    local distance = (Vector2.new(Mouse.X, Mouse.Y + 36) - Vector2.new(pos.X, pos.Y)).Magnitude
                    if distance < shortestDistance then
                        shortestDistance = distance
                        closestPlayer = player
                    end
                end
            end
        end
    end

    return closestPlayer
end

-- AIMBOT: TRAVAR NA CABEÇA AO SEGURAR MOUSE2
RunService.RenderStepped:Connect(function()
    if AimbotEnabled and UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
        local target = getClosestPlayer()
        if target and target.Character then
            local head = target.Character:FindFirstChild("Head")
            if head then
                Camera.CFrame = CFrame.new(Camera.CFrame.Position, head.Position)
            end
        end
    end
end)

-- APLICAR ESP A TODOS OS JOGADORES EXISTENTES
for _, player in pairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        createESP(player)
    end
end

-- APLICAR ESP A NOVOS JOGADORES
Players.PlayerAdded:Connect(function(player)
    if player ~= LocalPlayer then
        createESP(player)
    end
end)
