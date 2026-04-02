local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer

-- ================== GET ID SERVER ==================
local IDServer = nil

for _, v in pairs(ReplicatedStorage:GetChildren()) do
    if v:FindFirstChild("Functions") and v:FindFirstChild("Events") then
        IDServer = v.Name
        break
    end
end

print("ID Server:", IDServer)

-- ================== GET PET (FIXED) ==================
local function getPets()
    local petFolder = player.PlayerGui:WaitForChild("Trading")
        :WaitForChild("Trading")
        :WaitForChild("You")
        :WaitForChild("Inner")
        :WaitForChild("MainList")

    local pets = {}

    print("===== DANH SÁCH PET =====")

    for _, v in pairs(petFolder:GetChildren()) do
        if v.Name ~= "UIGridLayout" and v.Name ~= "AddPass" then
            table.insert(pets, v.Name)
            print("Pet ID:", v.Name, "| Class:", v.ClassName)
        end
    end

    print("Tổng pet:", #pets)
    print("=========================")

    return pets
end

-- ================== TRADE ==================
local function tradePets(amount)
    local pets = getPets()

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

        ReplicatedStorage:WaitForChild(IDServer)
            :WaitForChild("Functions")
            :WaitForChild("AddTradeItem")
            :InvokeServer(unpack(args))

        task.wait(0.2)
    end
end

-- ================== UI ==================
local gui = Instance.new("ScreenGui")
gui.Name = "AutoTradeUI"
gui.Parent = game:GetService("CoreGui")
gui.ResetOnSpawn = false

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 260, 0, 160)
frame.Position = UDim2.new(0.5, -130, 0.5, -80)
frame.BackgroundColor3 = Color3.fromRGB(30,30,30)
frame.Active = true

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1,0,0,30)
title.Text = "AUTO TRADE PET"
title.BackgroundColor3 = Color3.fromRGB(20,20,20)
title.TextColor3 = Color3.new(1,1,1)

local textbox = Instance.new("TextBox", frame)
textbox.Size = UDim2.new(0.8,0,0,30)
textbox.Position = UDim2.new(0.1,0,0.35,0)
textbox.PlaceholderText = "Nhập số lượng pet..."
textbox.Text = ""
textbox.BackgroundColor3 = Color3.fromRGB(50,50,50)
textbox.TextColor3 = Color3.new(1,1,1)

local button = Instance.new("TextButton", frame)
button.Size = UDim2.new(0.8,0,0,40)
button.Position = UDim2.new(0.1,0,0.65,0)
button.Text = "Trade Pet"
button.BackgroundColor3 = Color3.fromRGB(0,170,0)
button.TextColor3 = Color3.new(1,1,1)

-- ================== DRAG ==================
local dragging = false
local dragInput, dragStart, startPos

local function update(input)
    local delta = input.Position - dragStart
    frame.Position = UDim2.new(
        startPos.X.Scale,
        startPos.X.Offset + delta.X,
        startPos.Y.Scale,
        startPos.Y.Offset + delta.Y
    )
end

title.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = frame.Position

        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

title.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        update(input)
    end
end)

-- ================== CLICK ==================
button.MouseButton1Click:Connect(function()
    local amount = tonumber(textbox.Text)

    if not amount then
        warn("Nhập số hợp lệ!")
        return
    end

    tradePets(amount)
end)
