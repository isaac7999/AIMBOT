--[[ 📌 Aimbot + Hitbox Expandida Temporária na Cabeça do Alvo - Arceus X Mobile ]]
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer
local Workspace = game:GetService("Workspace")

-- Círculo Visual Roxo
local circle = Drawing.new("Circle")
circle.Color = Color3.fromRGB(170, 0, 255)
circle.Thickness = 2
circle.Radius = 75 -- Círculo um pouco menor
circle.Filled = false
circle.Visible = true
circle.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)

local function IsInCircle(position)
    local screenPos, onScreen = Camera:WorldToViewportPoint(position)
    if onScreen then
        local dist = (Vector2.new(screenPos.X, screenPos.Y) - circle.Position).Magnitude
        return dist <= circle.Radius
    end
    return false
end

local function HasLineOfSight(targetPos)
    local origin = Camera.CFrame.Position
    local direction = (targetPos - origin).Unit * (targetPos - origin).Magnitude
    local rayParams = RaycastParams.new()
    rayParams.FilterDescendants = {LocalPlayer.Character}
    rayParams.FilterType = Enum.RaycastFilterType.Blacklist

    local rayResult = Workspace:Raycast(origin, direction, rayParams)
    if rayResult then
        -- Se o Ray bater antes da cabeça do alvo, significa que tem parede na frente
        if (rayResult.Position - origin).Magnitude < (targetPos - origin).Magnitude then
            return false
        end
    end
    return true
end

local function GetClosestPlayer()
    local closestPlayer = nil
    local closestDistance = math.huge
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
            if humanoid and humanoid.Health > 0 then
                local headPos = player.Character.Head.Position
                if IsInCircle(headPos) and HasLineOfSight(headPos) then
                    local screenPos, onScreen = Camera:WorldToViewportPoint(headPos)
                    if onScreen then
                        local distance = (Vector2.new(screenPos.X, screenPos.Y) - circle.Position).Magnitude
                        if distance < closestDistance then
                            closestDistance = distance
                            closestPlayer = player
                        end
                    end
                end
            end
        end
    end
    return closestPlayer
end

local currentTarget = nil
local originalSize = nil

local function ExpandHead(player)
    if player and player.Character and player.Character:FindFirstChild("Head") then
        local head = player.Character.Head
        if not originalSize then
            originalSize = head.Size
        end
        head.Size = Vector3.new(5, 5, 5)
        head.CanCollide = false
    end
end

local function RestoreHead(player)
    if player and player.Character and player.Character:FindFirstChild("Head") and originalSize then
        local head = player.Character.Head
        head.Size = originalSize
        head.CanCollide = false
    end
    originalSize = nil
end

-- Loop principal
RunService.RenderStepped:Connect(function()
    local target = GetClosestPlayer()

    if target ~= currentTarget then
        if currentTarget then
            RestoreHead(currentTarget)
        end
        currentTarget = target
        if currentTarget then
            ExpandHead(currentTarget)
        end
    end

    if currentTarget and currentTarget.Character and currentTarget.Character:FindFirstChild("Head") then
        local headPos = currentTarget.Character.Head.Position
        Camera.CFrame = CFrame.new(Camera.CFrame.Position, headPos)
    end
end)
