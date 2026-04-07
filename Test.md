local placeId = game.PlaceId

if placeId == 75992362647444 or placeId == 111187356770616 then
-- ================== WAIT GAME LOAD ==================
if not game:IsLoaded() then
    game.Loaded:Wait()
end

-- đợi Player + Character
local Players = game:GetService("Players")
local player = Players.LocalPlayer

while not player do
    task.wait()
    player = Players.LocalPlayer
end

-- đợi character load (tránh lỗi teleport / humanoid)
if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then
    player.CharacterAdded:Wait()
end

-- đợi PlayerGui load
player:WaitForChild("PlayerGui")

task.wait(2) -- buffer thêm cho UI game load hết

print("✅ Game Loaded - Script Starting")

local Players = game:GetService("Players")
local rs = game:GetService("ReplicatedStorage")
local HttpService = game:GetService("HttpService")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer

-- ================== CONFIG ==================
local PET_NAME = "Easter Basket"

local POSITIONS = {
    EGG = Vector3.new(2475.5, 253, -154),
    GOLDEN = Vector3.new(1416, 656, -13443),
    RAINBOW = Vector3.new(1219, 665, -13375)
}

local AutoSystem = {
    enabled = true,
    eggAmount = 8,
    mode = "EGG",
    isRainbowRunning = false,
    autoClick = true
}

-- ================== SAVE ==================
local fileName = "auto_system.json"

pcall(function()
    if isfile(fileName) then
        local data = HttpService:JSONDecode(readfile(fileName))
        AutoSystem.eggAmount = data.eggAmount or 8
        AutoSystem.enabled = data.enabled ~= nil and data.enabled or true
    end
end)

local function saveData()
    writefile(fileName, HttpService:JSONEncode({
        eggAmount = AutoSystem.eggAmount,
        enabled = AutoSystem.enabled
    }))
end

-- ================== REMOTES ==================
local hatchFolder, craftRemote, startRainbow, claimRainbow

for _, v in pairs(rs:GetChildren()) do
    local f = v:FindFirstChild("Functions")
    if f then
        if f:FindFirstChild("OpenEgg") then
            hatchFolder = v
        end
        if f:FindFirstChild("CraftPets") then
            craftRemote = f.CraftPets
        end
        if f:FindFirstChild("StartRainbow") then
            startRainbow = f.StartRainbow
            claimRainbow = f.ClaimRainbow
        end
    end
end

-- ================== INVENTORY ==================
local inventoryUI = player.PlayerGui
    :WaitForChild("Tabs")
    :WaitForChild("Inventory")

local container = inventoryUI
    :WaitForChild("Menu")
    :WaitForChild("Categories")
    :WaitForChild("Pets")
    :WaitForChild("Inner")
    :WaitForChild("List")
    :WaitForChild("Container")

local function setInventory(state)
    inventoryUI.Enabled = state
end

local function waitForInventoryLoad(timeout)
    timeout = timeout or 5
    local start = tick()

    while tick() - start < timeout do
        for _, pet in pairs(container:GetChildren()) do
            if pet.Name == PET_NAME then
                local grad = pet:FindFirstChild("UIGradient")
                if grad and grad.Color then
                    return true
                end
            end
        end
        task.wait(0.2)
    end

    warn("⚠️ Inventory load timeout")
    return false
end

-- ================== GET PET ==================
local function getPets()
    local ids = {}
    for _, pet in pairs(container:GetChildren()) do
        if pet.Name == PET_NAME and pet:FindFirstChild("Mythical") then
            local id = pet:GetAttribute("Id")
            if id then table.insert(ids, id) end
        end
    end
    return ids
end

local function isGolden(pet)
    local grad = pet:FindFirstChild("UIGradient")
    if not grad then return false end

    for _, kp in pairs(grad.Color.Keypoints) do
        if kp.Value.R > 0.95 then
            return true
        end
    end
    return false
end

local function getGoldenPets()
    local list = {}
    for _, pet in pairs(container:GetChildren()) do
        if pet.Name == PET_NAME and not pet:FindFirstChild("Mythical") then
            if isGolden(pet) then
                local id = pet:GetAttribute("Id")
                if id then table.insert(list, id) end
            end
        end
    end
    return list
end

-- ================== TELEPORT ==================
local function tp(pos)
    local char = player.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        char.HumanoidRootPart.CFrame = CFrame.new(pos)
    end
end

-- ================== TRADE ==================
local IDServer = nil
for _, v in pairs(rs:GetChildren()) do
    if v:FindFirstChild("Functions") and v:FindFirstChild("Events") then
        IDServer = v.Name
        break
    end
end

local function getTradePets()
    local petFolder = player.PlayerGui:WaitForChild("Trading")
        :WaitForChild("Trading")
        :WaitForChild("You")
        :WaitForChild("Inner")
        :WaitForChild("MainList")

    local pets = {}

    for _, v in pairs(petFolder:GetChildren()) do
        if v.Name ~= "UIGridLayout" and v.Name ~= "AddPass" then
            table.insert(pets, v.Name)
        end
    end

    return pets
end

local function tradePets(amount)
    local pets = getTradePets()

    if not IDServer then
        warn("Không có ID Server!")
        return
    end

    if #pets == 0 then
        warn("Không có pet!")
        return
    end

    for i = 1, amount do
        local petID = pets[i]
        if not petID then break end

        local args = {
            {
                "Pet",
                petID
            }
        }

        rs:WaitForChild(IDServer)
            :WaitForChild("Functions")
            :WaitForChild("AddTradeItem")
            :InvokeServer(unpack(args))

        task.wait(0.2)
    end
end

-- ================== GUI ==================
local gui = Instance.new("ScreenGui")
gui.Parent = game:GetService("CoreGui")
gui.ResetOnSpawn = false
gui.DisplayOrder = 999999

local main = Instance.new("Frame", gui)
main.Size = UDim2.new(0,380,0,470)
main.AnchorPoint = Vector2.new(1,1)
main.Position = UDim2.new(1, -10, 1, -10)
main.BackgroundColor3 = Color3.fromRGB(20,20,20)
main.Active = true
main.Draggable = true
Instance.new("UICorner", main)

local top = Instance.new("Frame", main)
top.Size = UDim2.new(1,0,0,40)
top.BackgroundColor3 = Color3.fromRGB(15,15,15)
Instance.new("UICorner", top)

local title = Instance.new("TextLabel", top)
title.Size = UDim2.new(1,-40,1,0)
title.Position = UDim2.new(0,10,0,0)
title.Text = "AUTO SYSTEM"
title.BackgroundTransparency = 1
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.GothamBold
title.TextSize = 18
title.TextXAlignment = Enum.TextXAlignment.Left

local minimize = Instance.new("TextButton", top)
minimize.Size = UDim2.new(0,30,0,30)
minimize.Position = UDim2.new(1,-35,0,5)
minimize.Text = "-"
minimize.BackgroundColor3 = Color3.fromRGB(40,40,40)
minimize.TextColor3 = Color3.new(1,1,1)
Instance.new("UICorner", minimize)

local content = Instance.new("Frame", main)
content.Size = UDim2.new(1,0,1,-40)
content.Position = UDim2.new(0,0,0,40)
content.BackgroundTransparency = 1

-- TOGGLE
local toggle = Instance.new("TextButton", content)
toggle.Size = UDim2.new(1,-20,0,50)
toggle.Position = UDim2.new(0,10,0,60)
toggle.Font = Enum.Font.GothamBold
toggle.TextSize = 20
toggle.TextColor3 = Color3.new(1,1,1)
Instance.new("UICorner", toggle)

-- AUTO CLICK TOGGLE
local autoClickBtn = Instance.new("TextButton", content)
autoClickBtn.Size = UDim2.new(1,-20,0,45)
autoClickBtn.Position = UDim2.new(0,10,0,10)
autoClickBtn.Font = Enum.Font.GothamBold
autoClickBtn.TextSize = 18
autoClickBtn.TextColor3 = Color3.new(1,1,1)
Instance.new("UICorner", autoClickBtn)

-- MODE
local modeLabel = Instance.new("TextLabel", content)
modeLabel.Size = UDim2.new(1,-20,0,30)
modeLabel.Position = UDim2.new(0,10,0,120)
modeLabel.BackgroundTransparency = 1
modeLabel.TextColor3 = Color3.new(1,1,1)
modeLabel.Font = Enum.Font.GothamBold
modeLabel.TextSize = 18

-- INPUT EGG
local input = Instance.new("TextBox", content)
input.Size = UDim2.new(1,-20,0,55)
input.Position = UDim2.new(0,10,0,160)
input.BackgroundColor3 = Color3.fromRGB(40,40,40)
input.TextColor3 = Color3.new(1,1,1)
input.Font = Enum.Font.GothamBold
input.TextSize = 24
input.TextXAlignment = Enum.TextXAlignment.Center
input.ClearTextOnFocus = false
Instance.new("UICorner", input)

-- RAINBOW BUTTON
local rainbowBtn = Instance.new("TextButton", content)
rainbowBtn.Size = UDim2.new(1,-20,0,50)
rainbowBtn.Position = UDim2.new(0,10,0,240)
rainbowBtn.Text = "AUTO RAINBOW"
rainbowBtn.BackgroundColor3 = Color3.fromRGB(0,120,200)
rainbowBtn.TextColor3 = Color3.new(1,1,1)
rainbowBtn.Font = Enum.Font.GothamBold
rainbowBtn.TextSize = 24
Instance.new("UICorner", rainbowBtn)

-- TRADE INPUT
local tradeBox = Instance.new("TextBox", content)
tradeBox.Size = UDim2.new(1,-20,0,45)
tradeBox.Position = UDim2.new(0,10,0,300)
tradeBox.PlaceholderText = "Số pet trade..."
tradeBox.Text = "1"
tradeBox.TextSize = 18
tradeBox.BackgroundColor3 = Color3.fromRGB(40,40,40)
tradeBox.TextColor3 = Color3.new(1,1,1)
Instance.new("UICorner", tradeBox)

-- TRADE BUTTON
local tradeBtn = Instance.new("TextButton", content)
tradeBtn.Size = UDim2.new(1,-20,0,45)
tradeBtn.Position = UDim2.new(0,10,0,350)
tradeBtn.Text = "TRADE PET"
tradeBtn.TextSize = 18
tradeBtn.BackgroundColor3 = Color3.fromRGB(200,120,0)
tradeBtn.TextColor3 = Color3.new(1,1,1)
Instance.new("UICorner", tradeBtn)

-- MINIMIZE
local minimized = false
minimize.MouseButton1Click:Connect(function()
    minimized = not minimized
    content.Visible = not minimized
    main.Size = minimized and UDim2.new(0,200,0,40) or UDim2.new(0,380,0,470)
end)

-- UI UPDATE
local function updateUI()
    toggle.Text = AutoSystem.enabled and "ON" or "OFF"
    toggle.BackgroundColor3 = AutoSystem.enabled and Color3.fromRGB(0,170,100) or Color3.fromRGB(170,0,0)

    autoClickBtn.Text = AutoSystem.autoClick and "AUTO CLICK: ON" or "AUTO CLICK: OFF"
    autoClickBtn.BackgroundColor3 = AutoSystem.autoClick and Color3.fromRGB(0,170,100) or Color3.fromRGB(170,0,0)

    if AutoSystem.isRainbowRunning then
        modeLabel.Text = "Mode: RAINBOW"
    else
        modeLabel.Text = "Mode: "..AutoSystem.mode
    end

    input.Text = tostring(AutoSystem.eggAmount)
end

updateUI()

autoClickBtn.MouseButton1Click:Connect(function()
    AutoSystem.autoClick = not AutoSystem.autoClick
    updateUI()
end)

toggle.MouseButton1Click:Connect(function()
    AutoSystem.enabled = not AutoSystem.enabled
    saveData()
    updateUI()
end)

input.FocusLost:Connect(function()
    local num = tonumber(input.Text)
    if num then
        AutoSystem.eggAmount = math.clamp(num,1,99)
        saveData()
    end
    updateUI()
end)

-- TRADE CLICK
tradeBtn.MouseButton1Click:Connect(function()
    local amount = tonumber(tradeBox.Text)
    if not amount then return end
    tradePets(amount)
end)

-- ================== AUTO RAINBOW ==================
rainbowBtn.MouseButton1Click:Connect(function()
    if AutoSystem.isRainbowRunning then return end

    AutoSystem.isRainbowRunning = true
    updateUI()

    task.spawn(function()
        setInventory(true)
        waitForInventoryLoad(6)

        while true do
            local golden = getGoldenPets()
            if #golden < 5 then break end

            tp(POSITIONS.RAINBOW)
            task.wait(1.5)

            setInventory(false)
            task.wait(0.5)
            setInventory(true)
            waitForInventoryLoad(3)

            local queue = {}
            local index = 1

            for slot = 1,3 do
                if golden[index+4] then
                    local ids = {
                        golden[index],
                        golden[index+1],
                        golden[index+2],
                        golden[index+3],
                        golden[index+4]
                    }

                    startRainbow:InvokeServer(ids)
                    table.insert(queue, ids[1])

                    index += 5
                    task.wait(1)
                end
            end

            task.wait(30)

            setInventory(false)

            for _, id in pairs(queue) do
                claimRainbow:InvokeServer(id)
                task.wait(1)
            end

            setInventory(true)
            waitForInventoryLoad(3)

            task.wait(2)
        end

        setInventory(false)

        AutoSystem.isRainbowRunning = false
        AutoSystem.mode = "EGG"
        setInventory(true)
        waitForInventoryLoad(3)
        updateUI()
    end)
end)
-- ================== AUTO INVENTORY LOOP ==================
task.spawn(function()
    while true do
        if AutoSystem.enabled and not AutoSystem.isRainbowRunning then
            setInventory(true)
            waitForInventoryLoad(5)

            -- đợi 10 phút
            local t = 0
            while t < 600 do
                if not AutoSystem.enabled then break end
                task.wait(1)
                t += 1
            end
        else
            task.wait(1)
        end
    end
end)
-- ================== MAIN LOOP ==================
task.spawn(function()
    while true do
        if AutoSystem.enabled and not AutoSystem.isRainbowRunning then
            local ids = getPets()

            if AutoSystem.mode == "EGG" then
                tp(POSITIONS.EGG)

                if #ids >= 6 then
                    AutoSystem.mode = "GOLDEN"
                elseif hatchFolder then
                    hatchFolder.Functions.OpenEgg:InvokeServer("Spring Blossom", AutoSystem.eggAmount)
                end

            elseif AutoSystem.mode == "GOLDEN" then
                tp(POSITIONS.GOLDEN)

                if #ids >= 6 and craftRemote then
                    craftRemote:InvokeServer({
                        ids[1], ids[2], ids[3],
                        ids[4], ids[5], ids[6]
                    })
                else
                    AutoSystem.mode = "EGG"
                end
            end
        end

        task.wait(0.5)
    end
end)

-- ================== AUTO CLICK ==================
task.spawn(function()
    local baseX, baseY = 1050, 100

    while true do
        if AutoSystem.autoClick then
            pcall(function()
                local offsetX = math.random(-5,5)
                local offsetY = math.random(-5,5)

                mousemoveabs(baseX + offsetX, baseY + offsetY)
                mouse1click()
            end)
        end
        task.wait(0.05)
    end
end)

end
