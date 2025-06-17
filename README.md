local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local VirtualInputManager = game:GetService("VirtualInputManager")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

local speed = 1.5
local plantas = {
    Vector3.new(307.29, 33.93, 1786.47), Vector3.new(323.36, 33.97, 1786.84),
    Vector3.new(337.36, 33.96, 1788.14), Vector3.new(352.48, 34.03, 1787.68),
    Vector3.new(369.08, 34.03, 1789.15), Vector3.new(385.24, 34.06, 1789.45),
    Vector3.new(305.71, 33.25, 1805.89), Vector3.new(321.20, 33.29, 1806.03),
    Vector3.new(336.98, 33.32, 1806.60), Vector3.new(352.18, 33.37, 1806.64),
}

local npcPos = Vector3.new(4320.91, 13.68, -2644.16)
local preEntregaPos = Vector3.new(570.66, -5.99, 1567.76)
local retornoPos = Vector3.new(601.92, -6.95, 1518.75)

local playerGui = player:WaitForChild("PlayerGui")
local screenGui = Instance.new("ScreenGui", playerGui)

local startButton = Instance.new("TextButton")
startButton.Size = UDim2.new(0, 100, 0, 40)
startButton.Position = UDim2.new(1, -110, 0, 10)
startButton.BackgroundColor3 = Color3.fromRGB(50, 150, 50)
startButton.TextColor3 = Color3.new(1, 1, 1)
startButton.Font = Enum.Font.GothamBold
startButton.TextSize = 20
startButton.Text = "Start"
startButton.Parent = screenGui

local dropdownLabel = Instance.new("TextLabel", screenGui)
dropdownLabel.Size = UDim2.new(0, 120, 0, 30)
dropdownLabel.Position = UDim2.new(1, -230, 0, 15)
dropdownLabel.BackgroundTransparency = 1
dropdownLabel.TextColor3 = Color3.new(1, 1, 1)
dropdownLabel.Font = Enum.Font.GothamBold
dropdownLabel.TextSize = 18
dropdownLabel.Text = "Ciclos:"

local dropdown = Instance.new("TextButton", screenGui)
dropdown.Size = UDim2.new(0, 50, 0, 30)
dropdown.Position = UDim2.new(1, -170, 0, 15)
dropdown.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
dropdown.TextColor3 = Color3.new(1, 1, 1)
dropdown.Font = Enum.Font.GothamBold
dropdown.TextSize = 18
dropdown.Text = "1"

local dropdownOpen = false
local optionsFrame = Instance.new("Frame", screenGui)
optionsFrame.Size = UDim2.new(0, 50, 0, 90)
optionsFrame.Position = UDim2.new(1, -170, 0, 45)
optionsFrame.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
optionsFrame.Visible = false

for i = 1, 3 do
    local option = Instance.new("TextButton", optionsFrame)
    option.Size = UDim2.new(1, 0, 0, 30)
    option.Position = UDim2.new(0, 0, 0, (i - 1) * 30)
    option.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
    option.TextColor3 = Color3.new(1, 1, 1)
    option.Font = Enum.Font.GothamBold
    option.TextSize = 18
    option.Text = tostring(i)

    option.MouseButton1Click:Connect(function()
        dropdown.Text = option.Text
        optionsFrame.Visible = false
        dropdownOpen = false
    end)
end

dropdown.MouseButton1Click:Connect(function()
    dropdownOpen = not dropdownOpen
    optionsFrame.Visible = dropdownOpen
end)

local function cflyTo(targetPos, speed)
    local reached = false
    local conn
    conn = RunService.Heartbeat:Connect(function()
        local pos = humanoidRootPart.Position
        local dir = (targetPos - pos).Unit
        local dist = (targetPos - pos).Magnitude
        if dist < speed then
            humanoidRootPart.CFrame = CFrame.new(targetPos)
            reached = true
            conn:Disconnect()
        else
            humanoidRootPart.CFrame = humanoidRootPart.CFrame + dir * speed
        end
    end)
    repeat task.wait() until reached
end

local function holdKey(keyCode, duration)
    VirtualInputManager:SendKeyEvent(true, keyCode, false, game)
    task.wait(duration)
    VirtualInputManager:SendKeyEvent(false, keyCode, false, game)
end

local function pressKey(keyCode)
    VirtualInputManager:SendKeyEvent(true, keyCode, false, game)
    task.wait(0.1)
    VirtualInputManager:SendKeyEvent(false, keyCode, false, game)
end

local running = false
local function startAutoFarm()
    running = true
    startButton.Text = "Stop"
    startButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)

    local maxCycles = tonumber(dropdown.Text)

    while running do
        for cycle = 1, maxCycles do
            for i, pos in ipairs(plantas) do
                if not running then return end
                cflyTo(pos, speed)
                task.wait(0.2)

                if i == 1 then
                    if cycle == 1 then pressKey(Enum.KeyCode.One)
                    elseif cycle == 2 then pressKey(Enum.KeyCode.Two)
                    elseif cycle == 3 then pressKey(Enum.KeyCode.Three) end
                    task.wait(0.2)
                end

                holdKey(Enum.KeyCode.E, 6.3)
            end
        end

        if maxCycles == 1 then pressKey(Enum.KeyCode.One)
        elseif maxCycles == 2 then pressKey(Enum.KeyCode.Two)
        elseif maxCycles == 3 then pressKey(Enum.KeyCode.Three) end

        cflyTo(preEntregaPos, speed)
        task.wait(0.3)

        cflyTo(npcPos, speed)
        task.wait(0.3)

        for i = 1, maxCycles do
            local key = (i == 1 and Enum.KeyCode.One) or (i == 2 and Enum.KeyCode.Two) or Enum.KeyCode.Three
            pressKey(key)
            holdKey(Enum.KeyCode.E, 5)
            task.wait(0.3)
        end

        cflyTo(retornoPos, speed)
        task.wait(0.3)
        cflyTo(plantas[1], speed)
    end

    startButton.Text = "Start"
    startButton.BackgroundColor3 = Color3.fromRGB(50, 150, 50)
end

startButton.MouseButton1Click:Connect(function()
    if running then
        running = false
    else
        coroutine.wrap(startAutoFarm)()
    end
end)

player.Idled:Connect(function()
    VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.Space, false, game)
    task.wait(0.1)
    VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.Space, false, game)
end)

if not _G.KickHooked then
    _G.KickHooked = true
    local mt = getrawmetatable(game)
    setreadonly(mt, false)
    local old = mt.__namecall
    mt.__namecall = newcclosure(function(self, ...)
        if getnamecallmethod() == "Kick" then
            warn("Tentativa de Kick bloqueada.")
            return
        end
        return old(self, ...)
    end)
end
