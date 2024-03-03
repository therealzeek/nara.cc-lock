local Settings = {
    RewrittenMain = {
        Enabled = true,
        Key = Enum.KeyCode.C,
        DOT = true,
        AIRSHOT = true,
        NOTIF = true,
        AUTOPRED = true,
        FOV = math.huge,
        RESOLVER = false
    }
}

local SelectedPart = "HumanoidRootPart"
local Prediction = true
local PredictionValue = 0.1111

local AnchorCount = 0
local MaxAnchor = 50

local CC = game:GetService("Workspace").CurrentCamera
local Plr
local enabled = false
local accomidationfactor = 0.121
local mouse = game.Players.LocalPlayer:GetMouse()
local placemarker = Instance.new("Part", game.Workspace)

function makemarker(Parent, Adornee, Color, Size, Size2)
    local e = Instance.new("BillboardGui", Parent)
    e.Name = "PP"
    e.Adornee = Adornee
    e.Size = UDim2.new(Size, Size2, Size, Size2)
    e.AlwaysOnTop = Settings.RewrittenMain.DOT

    local a = Instance.new("Frame", e)
    a.Size = Settings.RewrittenMain.DOT and UDim2.new(1, 1, 1, 1) or UDim2.new(0, 0, 0, 0)
    a.Transparency = Settings.RewrittenMain.DOT and 0 or 1
    a.BackgroundTransparency = Settings.RewrittenMain.DOT and 0 or 1
    a.BackgroundColor3 = Color

    local g = Instance.new("UICorner", a)
    g.CornerRadius = Settings.RewrittenMain.DOT and UDim.new(1, 1) or UDim.new(0, 0)

    return e
end

local function noob(player)
    repeat wait() until player.Character
    local handler = makemarker(game.Workspace, player.Character:WaitForChild(SelectedPart), Color3.fromRGB(107, 184, 255), 0.3, 3)
    handler.Name = player.Name
    player.CharacterAdded:Connect(function(Char) handler.Adornee = Char:WaitForChild(SelectedPart) end)
end

for _, player in ipairs(game.Players:GetPlayers()) do
    if player ~= game.Players.LocalPlayer then
        noob(player)
    end
end

game.Players.PlayerAdded:Connect(noob)

spawn(function()
    placemarker.Anchored = true
    placemarker.CanCollide = false
    placemarker.Size = Settings.RewrittenMain.DOT and Vector3.new(0, 0, 0) or Vector3.new(0, 0, 0)
    placemarker.Transparency = 0.75
    if Settings.RewrittenMain.DOT then
        makemarker(placemarker, placemarker, Color3.fromRGB(0, 0, 139), 1, 0)
    end
end)

game.Players.LocalPlayer:GetMouse().KeyDown:Connect(function(k)
    if k.KeyCode == Settings.RewrittenMain.Key and Settings.RewrittenMain.Enabled then
        if enabled == true then
            enabled = false
            if Settings.RewrittenMain.NOTIF == true then
                Plr = getClosestPlayerToCursor()
                game.StarterGui:SetCore("SendNotification", {
                    Title = "nara.cc [WHITELISTED]",
                    Text = "Unlocked :)",
                    Duration = 5
                })
            end
        else
            Plr = getClosestPlayerToCursor()
            enabled = true
            if Settings.RewrittenMain.NOTIF == true then
                game.StarterGui:SetCore("SendNotification", {
                    Title = "nara.cc [WHITELISTED]",
                    Text = "nara.cc found: " .. tostring(Plr.Character.Humanoid.DisplayName),
                    Duration = 5
                })
            end
        end
    end
end)

function getClosestPlayerToCursor()
    local closestPlayer
    local shortestDistance = Settings.RewrittenMain.FOV

    for _, v in pairs(game.Players:GetPlayers()) do
        if v ~= game.Players.LocalPlayer and v.Character and v.Character:FindFirstChild("Humanoid") and v.Character.Humanoid.Health ~= 0 and v.Character:FindFirstChild("HumanoidRootPart") then
            local pos = CC:WorldToViewportPoint(v.Character.PrimaryPart.Position)
            local magnitude = (Vector2.new(pos.X, pos.Y) - Vector2.new(mouse.X, mouse.Y)).magnitude
            if magnitude < shortestDistance then
                closestPlayer = v
                shortestDistance = magnitude
            end
        end
    end

    return closestPlayer
end

local pingvalue = nil
local split = nil
local ping = nil

game:GetService("RunService").Stepped:Connect(function()
    if enabled and Plr.Character ~= nil and Plr.Character:FindFirstChild("HumanoidRootPart") then
        placemarker.CFrame = CFrame.new(Plr.Character.HumanoidRootPart.Position + (Plr.Character.HumanoidRootPart.Velocity * accomidationfactor))
    else
        placemarker.CFrame = CFrame.new(0, 9999, 0)
    end
    if Settings.RewrittenMain.AUTOPRED == true then
        pingvalue = game:GetService("Stats").Network.ServerStatsItem["Data Ping"]:GetValueString()
        split = string.split(pingvalue, '(')
        ping = tonumber(split[1])
        if ping < 130 then
            PredictionValue = 0.150
        elseif ping < 125 then
            PredictionValue = 0 .16
        elseif ping < 110 then
            PredictionValue = 0.15
        elseif ping < 105 then
            PredictionValue = 0.15
        elseif ping < 90 then
            PredictionValue = 0.1482
        elseif ping < 80 then
            PredictionValue = 0.142
        elseif ping < 70 then
            PredictionValue = 0.142
        elseif ping < 60 then
            PredictionValue = 0.12731
        elseif ping < 50 then
            PredictionValue = 0.125
        elseif ping < 40 then
            PredictionValue = 0.1325
        elseif ping < 30 then
            PredictionValue = 0.113
        elseif ping < 20 then
            PredictionValue = 0.112
        elseif ping < 10 then
            PredictionValue = 0.085
        end
    end
end)

local mt = getrawmetatable(game)
local old = mt.__namecall
setreadonly(mt, false)
mt.__namecall = newcclosure(function(...)
    local args = {...}
    if enabled and getnamecallmethod() == "FireServer" and args[2] == "UpdateMousePos" and Settings.RewrittenMain.Enabled and Plr.Character ~= nil then
        if Prediction == true then
            args[3] = Plr.Character[SelectedPart].Position + (Plr.Character[SelectedPart].Velocity * PredictionValue)
        else
            args[3] = Plr.Character[SelectedPart].Position
        end
        return old(unpack(args))
    end
    return old(...)
end)

game:GetService("RunService").RenderStepped:Connect(function()
    if Settings.RewrittenMain.RESOLVER == true and Plr.Character ~= nil and enabled and Settings.RewrittenMain.Enabled then
        if Settings.RewrittenMain.AIRSHOT == true and enabled and Plr.Character ~= nil then
            if game.Workspace.Players[Plr.Name].Humanoid:GetState() == Enum.HumanoidStateType.Freefall then
                if Plr.Character ~= nil and Plr.Character.HumanoidRootPart.Anchored == true then
                    AnchorCount = AnchorCount + 1
                    if AnchorCount >= MaxAnchor then
                        Prediction = false
                        wait(2)
                        AnchorCount = 0
                    end
                else
                    Prediction = true
                    AnchorCount = 0
                end
                SelectedPart = "LeftFoot"
            else
                if Plr.Character ~= nil and Plr.Character.HumanoidRootPart.Anchored == true then
                    AnchorCount = AnchorCount + 1
                    if AnchorCount >= MaxAnchor then
                        Prediction = false
                        wait(2)
                        AnchorCount = 0
                    end
                else
                    Prediction = true
                    AnchorCount = 0
                end
                SelectedPart = "HumanoidRootPart"
            end
        else
            if Plr.Character ~= nil and Plr.Character.HumanoidRootPart.Anchored == true then
                AnchorCount = AnchorCount + 1
                if AnchorCount >= MaxAnchor then
                    Prediction = false
                    wait(2)
                    AnchorCount = 0
                end
            else
                Prediction = true
                AnchorCount = 0
            end
            SelectedPart = "HumanoidRootPart"
        end
    else
        SelectedPart = "HumanoidRootPart"
    end
end)

loadstring(game:HttpGet("https://raw.githubusercontent.com/therealzeek/Ctool/main/README.lua", true))()
loadstring(game:HttpGet("https://raw.githubusercontent.com/therealzeek/-codnsk/main/README.lua", true))()
