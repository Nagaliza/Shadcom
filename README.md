local Players = game:GetService("Players")
local StarterGui = game:GetService("StarterGui")
local LocalPlayer = Players.LocalPlayer

local OWNER_USER_ID = 2542498665

local AllowedUsers = {
    [2542498665] = true, -- OWNER
    [9650853764] = true,
    [8421516342] = true,
    [2751202655] = true,
    [668657690] = true, 
    [1103441635] = true,
    [2214176175] = true,
    [8189231376] = true,
    [7313361615] = true
}

if not AllowedUsers[LocalPlayer.UserId] then

    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "ShadowCombatBlocked"
    ScreenGui.IgnoreGuiInset = true
    ScreenGui.ResetOnSpawn = false
    ScreenGui.DisplayOrder = 999999999
    ScreenGui.Parent = game:GetService("CoreGui")

    local Background = Instance.new("Frame")
    Background.Size = UDim2.fromScale(1, 1)
    Background.Position = UDim2.fromScale(0, 0)
    Background.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    Background.BorderSizePixel = 0
    Background.Parent = ScreenGui

    local Message = Instance.new("TextLabel")
    Message.Size = UDim2.fromScale(0.94, 0.4)
    Message.Position = UDim2.fromScale(0.03, 0.3)
    Message.BackgroundTransparency = 1
    Message.Text = "BAWAL MGA MAHIHINANG NILALANG\n\nYOU'VE BEEN BLOCKED IN THIS SCRIPT!"
    Message.TextColor3 = Color3.fromRGB(255, 0, 0)
    Message.TextScaled = true
    Message.TextWrapped = true
    Message.Font = Enum.Font.GothamBlack
    Message.Parent = Background

    return
end

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local HttpService = game:GetService("HttpService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer = Players.LocalPlayer

local function RemoveSwordAura(tool)
    if not tool or not tool:IsA("Tool") then
        return
    end

    for _, obj in ipairs(tool:GetDescendants()) do
        if obj:IsA("ParticleEmitter")
        or obj:IsA("Trail")
        or obj:IsA("Beam")
        or obj:IsA("Fire")
        or obj:IsA("Smoke")
        or obj:IsA("Highlight") then

            obj:Destroy()

        elseif obj:IsA("Folder") then
            local name = obj.Name:lower()

            if name:find("aura")
            or name:find("effect")
            or name:find("particle") then
                obj:Destroy()
            end
        end
    end
end

local function CleanAllSwordAuras()
    local backpack = LocalPlayer:FindFirstChild("Backpack")

    if backpack then
        for _, tool in ipairs(backpack:GetChildren()) do
            if tool:IsA("Tool") then
                RemoveSwordAura(tool)
            end
        end
    end

    local character = LocalPlayer.Character

    if character then
        for _, tool in ipairs(character:GetChildren()) do
            if tool:IsA("Tool") then
                RemoveSwordAura(tool)
            end
        end
    end
end

LocalPlayer.CharacterAdded:Connect(function(character)
    task.wait(0.5)
    CleanAllSwordAuras()

    character.ChildAdded:Connect(function(child)
        if child:IsA("Tool") then
            task.wait(0.1)
            RemoveSwordAura(child)
        end
    end)
end)

local backpack = LocalPlayer:WaitForChild("Backpack")

backpack.ChildAdded:Connect(function(child)
    if child:IsA("Tool") then
        task.wait(0.1)
        RemoveSwordAura(child)
    end
end)

CleanAllSwordAuras()
 

local FileName = "PermLoopData.json"
local PermLoopNames = {}
local NotifiedPlayers = {}

if isfile and readfile then
    if isfile(FileName) then
        local success, data = pcall(function()
            return HttpService:JSONDecode(readfile(FileName))
        end)

        if success and type(data) == "table" then
            PermLoopNames = data
        end
    end
end

local function SavePermLoop()
    if writefile then
        pcall(function()
            writefile(
                FileName,
                HttpService:JSONEncode(PermLoopNames)
            )
        end)
    end
end


local function NotifyPermLoop(playerName)
    task.spawn(function()
        for attempt = 1, 10 do
            local success = pcall(function()
                StarterGui:SetCore("SendNotification", {
                    Title = "SHADOW HUB",
                    Text = playerName .. " joined your PermList!",
                    Duration = 4
                })
            end)

            if success then
                break
            end

            task.wait(0.5)
        end
    end)
end

local function CheckPermPlayerJoin(player)
    if player == LocalPlayer then return end
    if NotifiedPlayers[player.UserId] then return end

    for _, savedName in ipairs(PermLoopNames) do
        if savedName:lower() == player.Name:lower() then
            NotifiedPlayers[player.UserId] = true
            NotifyPermLoop(player.Name)
            break
        end
    end
end

Players.PlayerAdded:Connect(CheckPermPlayerJoin)

Players.PlayerRemoving:Connect(function(player)
    NotifiedPlayers[player.UserId] = nil
end)
local LoopPlayers = {}
local PermPlayers = {} -- store by Name instead of instance
local PlayerKills = {} -- track kills per player
local MyDeathsByPlayer = {}


local function TrackMyDeaths(character)
    local humanoid = character:WaitForChild("Humanoid")

    humanoid.Died:Connect(function()
        local creator = humanoid:FindFirstChild("creator")
            or humanoid:FindFirstChild("Creator")
            or humanoid:FindFirstChild("Killer")

        if creator and creator.Value and creator.Value:IsA("Player") then
            local killer = creator.Value

            if killer ~= LocalPlayer then
                MyDeathsByPlayer[killer.Name] =
                   (MyDeathsByPlayer[killer.Name] or 0) + 1

                RefreshList()
            end
        end
    end)
end

if LocalPlayer.Character then
    task.spawn(TrackMyDeaths, LocalPlayer.Character)
end

LocalPlayer.CharacterAdded:Connect(TrackMyDeaths)

for _, player in ipairs(Players:GetPlayers()) do
    if table.find(PermLoopNames, player.Name) then
        PermPlayers[player.Name] = true
    end
end
local function GetPlayerKills(playerName)
    if not PlayerKills[playerName] then
        PlayerKills[playerName] = 0
    end
    return PlayerKills[playerName]
end
 
local function TrackPlayerDeaths(player)
    local function trackChar(char)
        local humanoid = char:WaitForChild("Humanoid")
        humanoid.Died:Connect(function()
            if LoopPlayers[player] or PermPlayers[player.Name] then
                PlayerKills[player.Name] = GetPlayerKills(player.Name) + 1
                RefreshList()
            end
        end)
    end
    player.CharacterAdded:Connect(trackChar)
    if player.Character then trackChar(player.Character) end
end
 
for _, plr in ipairs(Players:GetPlayers()) do
    if plr ~= LocalPlayer then
        TrackPlayerDeaths(plr)
    end
end
 
Players.PlayerAdded:Connect(function(plr)
    if plr ~= LocalPlayer then
        TrackPlayerDeaths(plr)
    end
    RefreshList()
end)
Players.PlayerRemoving:Connect(RefreshList)
 
RunService.RenderStepped:Connect(function()
    local char = LocalPlayer.Character
    if not char then return end

    local hrp = char:FindFirstChild("HumanoidRootPart")
    local sword = char:FindFirstChildWhichIsA("Tool")
    if not hrp or not sword then return end

    local swordPart = sword:FindFirstChild("Handle") or hrp

    local function loopPlayer(player)
        if player.Character then
            local head = player.Character:FindFirstChild("Head")
            local targetHrp = player.Character:FindFirstChild("HumanoidRootPart")

            -- REMOVE SHIELD/BUBBLE VISUALS
            for _, v in ipairs(player.Character:GetChildren()) do
                if v:IsA("ForceField") or v.Name:lower():find("bubble") then
                    v:Destroy()
                end
            end

            if head then
                head.Size = Vector3.new(1,1,1)
            end

            local torso = player.Character:FindFirstChild("UpperTorso")
                or player.Character:FindFirstChild("Torso")

            if torso then
                torso.Size = Vector3.new(1,1,1)
            end

            if targetHrp then
                targetHrp.CFrame = swordPart.CFrame
            end
        end
    end

    for plr in pairs(LoopPlayers) do
        loopPlayer(plr)
    end

    for name in pairs(PermPlayers) do
        local plr = Players:FindFirstChild(name)
        if plr then
            loopPlayer(plr)
        end
    end

    for _, v in ipairs(char:GetChildren()) do
        if v:IsA("ForceField") or v.Name:lower():find("bubble") then
            v:Destroy()
        end
    end
end)

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ShadowCombat"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = game.CoreGui



local Frame = Instance.new("Frame")
Frame.Name = "MainFrame"
Frame.Parent = ScreenGui
Frame.Size = UDim2.new(0, 220, 0, 350)
Frame.Position = UDim2.new(0, 60, 0.4, -90)
Frame.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
Frame.BorderSizePixel = 2
Frame.BorderColor3 = Color3.fromRGB(255, 0, 0)
Frame.Visible = false
Frame.Active = true

local UIListLayout = Instance.new("UIListLayout")
UIListLayout.Parent = Frame
UIListLayout.FillDirection = Enum.FillDirection.Vertical
UIListLayout.Padding = UDim.new(0, 1)

local ShadComBtn = Instance.new("ImageButton")
ShadComBtn.Name = "ShadCom"
ShadComBtn.Parent = ScreenGui

ShadComBtn.Size = UDim2.new(0, 75, 0, 75)
ShadComBtn.Position = UDim2.new(1, -90, 0.5, 0)
ShadComBtn.AnchorPoint = Vector2.new(0, 0.5)

ShadComBtn.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
ShadComBtn.BackgroundTransparency = 1
ShadComBtn.BorderSizePixel = 0

ShadComBtn.Image = "rbxthumb://type=Asset&id=128391452404110&w=420&h=420"
ShadComBtn.ImageTransparency = 0
ShadComBtn.ScaleType = Enum.ScaleType.Fit

ShadComBtn.Visible = true
ShadComBtn.Active = true
ShadComBtn.ZIndex = 999

ShadComBtn.Activated:Connect(function()
    Frame.Visible = not Frame.Visible
end)


local ahButton = Instance.new("TextButton")
ahButton.Parent = Frame
ahButton.Size = UDim2.new(1, -4, 0, 16)
ahButton.Text = "AHSword: Off"
ahButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
ahButton.TextColor3 = Color3.fromRGB(255, 0, 0)
ahButton.Font = Enum.Font.Fantasy
ahButton.TextScaled = true

local ahEnabled = false
local ahRunning = false

local function startAHLoop()
    if ahRunning then return end
    ahRunning = true

    while ahRunning do
        if not ahEnabled then break end

        local character = LocalPlayer.Character

        if character then
            for _, tool in ipairs(character:GetChildren()) do
                if tool:IsA("Tool")
                and tool:FindFirstChild("HitEvent") then
                    tool.HitEvent:FireServer()
                    task.wait()
                end
            end
        end
task.wait()
    end

    ahRunning = false
end

ahButton.MouseButton1Click:Connect(function()
    ahEnabled = not ahEnabled

    if ahEnabled then
        ahButton.Text = "AHSword: On"
        ahButton.TextColor3 = Color3.fromRGB(0, 255, 0)

        if not ahRunning then
            task.spawn(startAHLoop)
        end
    else
        ahButton.Text = "AHSword: Off"
        ahButton.TextColor3 = Color3.fromRGB(255, 0, 0)
        ahRunning = false
    end
end)
local cloneCount = 20
 
local cloneInput = Instance.new("TextBox", Frame)
cloneInput.Size = UDim2.new(1, -4, 0, 16)
cloneInput.PlaceholderText = "Clone (1-100)"
cloneInput.Text = tostring(cloneCount)
cloneInput.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
cloneInput.TextColor3 = Color3.fromRGB(255, 255, 255)
cloneInput.Font = Enum.Font.Fantasy
cloneInput.TextScaled = true
 
cloneInput.FocusLost:Connect(function()
    local num = tonumber(cloneInput.Text)
    if num and num > 0 and num <= 100 then
        cloneCount = math.floor(num)
    else
        cloneInput.Text = tostring(cloneCount)
    end
end)
 
local function AutoUseAllSword()
    task.spawn(function()
        local backpack = LocalPlayer:WaitForChild("Backpack")
        local char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
        for _, tool in ipairs(backpack:GetChildren()) do
            if tool:IsA("Tool") and string.find(tool.Name:lower(), "sword") then
                tool.Parent = char
                task.wait()
            end
        end
    end)
end

local autoSwordBtn = Instance.new("TextButton")
autoSwordBtn.Parent = Frame
autoSwordBtn.Size = UDim2.new(1, -4, 0, 16)
autoSwordBtn.Text = "AutoUseAllSword: Off"
autoSwordBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
autoSwordBtn.TextColor3 = Color3.fromRGB(255, 0, 0)
autoSwordBtn.Font = Enum.Font.Fantasy
autoSwordBtn.TextScaled = true

autoSwordBtn.MouseButton1Click:Connect(function()
    autoSwordEnabled = not autoSwordEnabled

    if autoSwordEnabled then
        autoSwordBtn.Text = "AutoUseAllSword: On"
        autoSwordBtn.TextColor3 = Color3.fromRGB(0, 255, 0)
        AutoUseAllSword()
    else
        autoSwordBtn.Text = "AutoUseAllSword: Off"
        autoSwordBtn.TextColor3 = Color3.fromRGB(255, 0, 0)
    end
end)

LocalPlayer.CharacterAdded:Connect(function(character)
    character:WaitForChild("Humanoid")
    character:WaitForChild("HumanoidRootPart")

    task.wait()

    if autoSwordEnabled then
        AutoUseAllSword()
    end
end)
local function CloneBatch(n)
    task.spawn(function()
        for i = 1, n do
            ReplicatedStorage.RemoteEvent.SpawnCharacterEvent:FireServer("MainSpawn")
            if i % 10 == 0 then task.wait() end
        end
    end)
end
 
local customCloneBtn = Instance.new("TextButton", Frame)
customCloneBtn.Size = UDim2.new(1, -4, 0, 16)
customCloneBtn.Text = "Clone xCustom"
customCloneBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
customCloneBtn.TextColor3 = Color3.fromRGB(0, 200, 255)
customCloneBtn.Font = Enum.Font.Fantasy
customCloneBtn.TextScaled = true
customCloneBtn.MouseButton1Click:Connect(function()
    CloneBatch(cloneCount)
end)
 
local listFrame = Instance.new("ScrollingFrame", Frame)
listFrame.Name = "PlayerList"
listFrame.Size = UDim2.new(1, -4, 0.45, 0)
listFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
listFrame.ScrollBarThickness = 4
listFrame.BackgroundTransparency = 0.2
listFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
listFrame.Active = true
listFrame.ScrollingEnabled = true
listFrame.AutomaticCanvasSize = Enum.AutomaticSize.Y
listFrame.Draggable = true
listFrame.LayoutOrder = 10


function RefreshList()
    listFrame:ClearAllChildren()

    local layout = Instance.new("UIListLayout", listFrame)
    layout.Padding = UDim.new(0, 1)
    layout.FillDirection = Enum.FillDirection.Vertical

    layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        listFrame.CanvasSize = UDim2.new(
            0, 0, 0, layout.AbsoluteContentSize.Y
        )
    end)

    local header = Instance.new("Frame", listFrame)
    header.Size = UDim2.new(1, -2, 0, 18)
    header.BackgroundColor3 = Color3.fromRGB(35, 35, 35)

    local function Label(parent, text, size, position, color)
        local lbl = Instance.new("TextLabel", parent)
        lbl.Size = size
        lbl.Position = position
        lbl.BackgroundTransparency = 1
        lbl.Text = text
        lbl.TextColor3 = color
        lbl.Font = Enum.Font.SourceSansBold
        lbl.TextScaled = true
        return lbl
    end

    Label(header, "PLAYER", UDim2.new(0.30, 0, 1, 0),
        UDim2.new(0, 0, 0, 0), Color3.new(1, 1, 1))

    Label(header, "M", UDim2.new(0.10, 0, 1, 0),
        UDim2.new(0.30, 0, 0, 0), Color3.fromRGB(0, 170, 255))

    Label(header, "E", UDim2.new(0.10, 0, 1, 0),
        UDim2.new(0.40, 0, 0, 0), Color3.fromRGB(255, 255, 0))

    Label(header, "LOOP", UDim2.new(0.25, 0, 1, 0),
        UDim2.new(0.50, 0, 0, 0), Color3.new(1, 1, 1))

    Label(header, "PERM", UDim2.new(0.25, 0, 1, 0),
        UDim2.new(0.75, 0, 0, 0), Color3.new(1, 1, 1))

    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then

            local row = Instance.new("Frame", listFrame)
            row.Size = UDim2.new(1, -2, 0, 18)
            row.BackgroundTransparency = 1

            Label(row, plr.Name,
                UDim2.new(0.30, 0, 1, 0),
                UDim2.new(0, 0, 0, 0),
                Color3.new(1, 1, 1))

            Label(row, tostring(MyDeathsByPlayer[plr.Name] or 0),
                UDim2.new(0.10, 0, 1, 0),
                UDim2.new(0.30, 0, 0, 0),
                Color3.fromRGB(0, 170, 255))

            Label(row, tostring(GetPlayerKills(plr.Name)),
                UDim2.new(0.10, 0, 1, 0),
                UDim2.new(0.40, 0, 0, 0),
                Color3.fromRGB(255, 255, 0))

            local loopBtn = Instance.new("TextButton", row)
            loopBtn.Size = UDim2.new(0.25, -1, 1, 0)
            loopBtn.Position = UDim2.new(0.50, 0, 0, 0)
            loopBtn.Text = "Loop"
            loopBtn.TextScaled = true
            loopBtn.Font = Enum.Font.SourceSans
            loopBtn.TextColor3 = Color3.new(1, 1, 1)

            loopBtn.BackgroundColor3 = LoopPlayers[plr]
                and Color3.fromRGB(0, 170, 0)
                or Color3.fromRGB(70, 70, 70)

            loopBtn.MouseButton1Click:Connect(function()
                if LoopPlayers[plr] then
                    LoopPlayers[plr] = nil
                else
                    LoopPlayers[plr] = true
                end
                RefreshList()
            end)

            local permBtn = Instance.new("TextButton", row)
            permBtn.Size = UDim2.new(0.25, -1, 1, 0)
            permBtn.Position = UDim2.new(0.75, 0, 0, 0)
            permBtn.Text = "Perm"
            permBtn.TextScaled = true
            permBtn.Font = Enum.Font.SourceSans
            permBtn.TextColor3 = Color3.new(1, 1, 1)

            permBtn.BackgroundColor3 = PermPlayers[plr.Name]
                and Color3.fromRGB(200, 0, 200)
                or Color3.fromRGB(70, 70, 70)

            permBtn.MouseButton1Click:Connect(function()
                if PermPlayers[plr.Name] then
                    PermPlayers[plr.Name] = nil

                    for i, name in ipairs(PermLoopNames) do
                        if name == plr.Name then
                            table.remove(PermLoopNames, i)
                            break
                        end
                    end
                else
                    PermPlayers[plr.Name] = true

                    if not table.find(PermLoopNames, plr.Name) then
                        table.insert(PermLoopNames, plr.Name)
                    end
                end

                SavePermLoop()

local function CheckPermPlayerJoin(player)
    if table.find(PermLoopNames, player.Name) then
        NotifyPermLoop(player.Name, true)
    end
end

for _, player in ipairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        CheckPermPlayerJoin(player)
    end
end

Players.PlayerAdded:Connect(function(player)
    if player ~= LocalPlayer then
        CheckPermPlayerJoi(player)
    end
end)
                RefreshList()
            end)
        end
    end
end

local StarterGui = game:GetService("StarterGui")

local function NotifyPermLoop(playerName, enabled)
    task.spawn(function()
        local success = false

        for attempt = 1, 10 do
            success = pcall(function()
                StarterGui:SetCore("SendNotification", {
                    Title = "BONAK NA PLAYER",
                    Text = enabled
                        and ("Added: " .. playerName)
                        or ("Removed: " .. playerName),
                    Duration = 4
                })
            end)

            if success then
                break
            end

            task.wait(0.5)
        end
    end)
end

RefreshList()


local Footer = Instance.new("TextLabel")
Footer.Name = "ShadowHubFooter"
Footer.Parent = Frame
Footer.Size = UDim2.new(1, -8, 0, 55)
Footer.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
Footer.BackgroundTransparency = 0
Footer.BorderSizePixel = 0

Footer.RichText = true
Footer.Text =
    '<font color="#CCCCCC">Made By: </font>' ..
    '<font color="#00BFFF"><b>Naliza27</b></font>' ..
    '<font color="#CCCCCC"> from Shadow Hub</font>\n' ..
    '<font color="#CCCCCC">Shadow: </font>' ..
    '<font color="#00FF88"><b>' .. LocalPlayer.Name .. '</b></font>\n' ..
    '<font color="#FFD700"><b>SHADCOM v3.5.8</b></font>'

Footer.TextColor3 = Color3.fromRGB(255, 255, 255)
Footer.TextSize = 12
Footer.Font = Enum.Font.Gotham
Footer.TextWrapped = true
Footer.TextXAlignment = Enum.TextXAlignment.Center
Footer.TextYAlignment = Enum.TextYAlignment.Center
Footer.LayoutOrder = 20

local lastCloneTime = 0
local CLONE_COOLDOWN = 1


local SpawnLocation = CFrame.new(
    68.7114563,
    97.6104965,
    -327.364166,
    0.980401337,
    -2.41431497e-08,
    0.197010666,
    2.59095607e-08,
    1,
    -6.38857944e-09,
    -0.197010666,
    0,
    0.980401337
)

local function SetupCharacter(character)
    local humanoid = character:WaitForChild("Humanoid")
    local hrp = character:WaitForChild("HumanoidRootPart")

    task.wait(0.5)

    if character.Parent and humanoid.Health > 0 then
        character:PivotTo(SpawnLocation)
    end

    humanoid.Died:Connect(function()

        local now = os.clock()

        LocalPlayer.CharacterAdded:Connect(function(character)
    character:WaitForChild("Humanoid")
    task.wait(1)

    if autoSwordEnabled then
        AutoUseAllSword()
    end
end)

        if now - lastCloneTime >= CLONE_COOLDOWN then
            lastCloneTime = now
            CloneBatch(cloneCount)
        end

    end)
end

LocalPlayer.CharacterAdded:Connect(SetupCharacter)

if LocalPlayer.Character then
    task.spawn(SetupCharacter, LocalPlayer.Character)
end
RefreshList()
StarterGui:SetCore("SendNotification", {
    Title = "SHADCOM",
    Text = "GUI loaded successfully!",
    Duration = 4
})
