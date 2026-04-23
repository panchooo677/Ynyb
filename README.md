
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local HttpService = game:GetService("HttpService")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer

local SAVE_FILE = "ScriptSaver_SavedScripts.json"
local COLOR_FILE = "ScriptSaver_ColorSettings.json"

local savedScripts = {}
local rainbowEnabled = false
local rainbowConnection = nil
local rainbowHue = 0

local function saveScriptsToFile()
    local success, err = pcall(function()
        local dataToSave = {}
        for _, scriptData in ipairs(savedScripts) do
            table.insert(dataToSave, {
                id = scriptData.id,
                name = scriptData.name,
                code = scriptData.code
            })
        end
        local jsonData = HttpService:JSONEncode(dataToSave)
        writefile(SAVE_FILE, jsonData)
    end)
    if success then
        print("💾 Scripts saved successfully!")
    else
        warn("❌ Failed to save scripts: " .. tostring(err))
    end
end

local function loadScriptsFromFile()
    local success, result = pcall(function()
        if isfile(SAVE_FILE) then
            local jsonData = readfile(SAVE_FILE)
            return HttpService:JSONDecode(jsonData)
        end
        return {}
    end)
    if success and result then
        print("📂 Loaded " .. #result .. " scripts from file!")
        return result
    else
        warn("⚠️ Could not load scripts (first run or no file exists)")
        return {}
    end
end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ScriptSaverGUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

if player.PlayerGui:FindFirstChild("ScriptSaverGUI") then
    player.PlayerGui:FindFirstChild("ScriptSaverGUI"):Destroy()
end

ScreenGui.Parent = player.PlayerGui

local themeColor = Color3.fromRGB(59, 130, 246)

-- ============================================================
-- TOGGLE BUTTON
-- ============================================================
local ToggleButton = Instance.new("TextButton")
ToggleButton.Name = "ToggleButton"
ToggleButton.Size = UDim2.new(0, 110, 0, 30)
ToggleButton.Position = UDim2.new(1, -120, 0, 50)
ToggleButton.BackgroundColor3 = Color3.fromRGB(139, 92, 246)
ToggleButton.BackgroundTransparency = 0.7
ToggleButton.Text = "👁️ Hide UI"
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.Font = Enum.Font.GothamBold
ToggleButton.TextSize = 13
ToggleButton.Parent = ScreenGui

local ToggleCorner = Instance.new("UICorner")
ToggleCorner.CornerRadius = UDim.new(0, 8)
ToggleCorner.Parent = ToggleButton

local ToggleStroke = Instance.new("UIStroke")
ToggleStroke.Color = Color3.fromRGB(167, 139, 250)
ToggleStroke.Thickness = 2
ToggleStroke.Parent = ToggleButton

-- ============================================================
-- MAIN FRAME
-- ============================================================
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 370, 0, 460)
MainFrame.Position = UDim2.new(0.5, -185, 0.5, -230)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 41, 59)
MainFrame.BackgroundTransparency = 1
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

local uiVisible = true

local toggleDragging = false
local toggleDragStart = nil
local toggleStartPos = nil
local toggleDragDistance = 0
local toggleDragInput = nil

ToggleButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        toggleDragging = true
        toggleDragStart = input.Position
        toggleStartPos = ToggleButton.Position
        toggleDragDistance = 0
        toggleDragInput = input
    end
end)

ToggleButton.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        if toggleDragging and toggleDragInput and input == toggleDragInput then
            local delta = input.Position - toggleDragStart
            toggleDragDistance = math.abs(delta.X) + math.abs(delta.Y)
            ToggleButton.Position = UDim2.new(
                toggleStartPos.X.Scale,
                toggleStartPos.X.Offset + delta.X,
                toggleStartPos.Y.Scale,
                toggleStartPos.Y.Offset + delta.Y
            )
        end
    end
end)

ToggleButton.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        if input == toggleDragInput then
            if toggleDragDistance < 5 then
                uiVisible = not uiVisible
                MainFrame.Visible = uiVisible
                ToggleButton.Text = uiVisible and "👁️ Hide UI" or "👁️ Show UI"
            end
            toggleDragging = false
            toggleDragInput = nil
        end
    end
end)

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 14)
MainCorner.Parent = MainFrame

local MainStroke = Instance.new("UIStroke")
MainStroke.Color = Color3.fromRGB(59, 130, 246)
MainStroke.Thickness = 2
MainStroke.Parent = MainFrame

-- ============================================================
-- TOP BAR
-- ============================================================
local TopBar = Instance.new("Frame")
TopBar.Name = "TopBar"
TopBar.Size = UDim2.new(1, 0, 0, 42)
TopBar.Position = UDim2.new(0, 0, 0, 0)
TopBar.BackgroundColor3 = Color3.fromRGB(15, 23, 42)
TopBar.BackgroundTransparency = 0.3
TopBar.BorderSizePixel = 0
TopBar.Parent = MainFrame

local TopBarCorner = Instance.new("UICorner")
TopBarCorner.CornerRadius = UDim.new(0, 14)
TopBarCorner.Parent = TopBar

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -110, 1, 0)
Title.Position = UDim2.new(0, 10, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "🔥 SCRIPT SAVER"
Title.TextColor3 = Color3.fromRGB(0, 217, 255)
Title.Font = Enum.Font.GothamBlack
Title.TextSize = 17
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = TopBar

-- Color Button
local ColorButton = Instance.new("TextButton")
ColorButton.Name = "ColorButton"
ColorButton.Size = UDim2.new(0, 28, 0, 28)
ColorButton.Position = UDim2.new(1, -68, 0.5, -14)
ColorButton.BackgroundColor3 = themeColor
ColorButton.Text = "🎨"
ColorButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ColorButton.Font = Enum.Font.GothamBold
ColorButton.TextSize = 13
ColorButton.Parent = TopBar

local ColorBtnCorner = Instance.new("UICorner")
ColorBtnCorner.CornerRadius = UDim.new(0, 7)
ColorBtnCorner.Parent = ColorButton

-- Remove UI Button
local RemoveUIBtn = Instance.new("TextButton")
RemoveUIBtn.Name = "RemoveUIBtn"
RemoveUIBtn.Size = UDim2.new(0, 28, 0, 28)
RemoveUIBtn.Position = UDim2.new(1, -34, 0.5, -14)
RemoveUIBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
RemoveUIBtn.Text = "✖"
RemoveUIBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
RemoveUIBtn.Font = Enum.Font.GothamBold
RemoveUIBtn.TextSize = 13
RemoveUIBtn.Parent = TopBar

local RemoveUICorner = Instance.new("UICorner")
RemoveUICorner.CornerRadius = UDim.new(0, 7)
RemoveUICorner.Parent = RemoveUIBtn

local RemoveUIStroke = Instance.new("UIStroke")
RemoveUIStroke.Color = Color3.fromRGB(239, 68, 68)
RemoveUIStroke.Thickness = 1.5
RemoveUIStroke.Parent = RemoveUIBtn

-- Drag via TopBar
local mainDragging = false
local mainDragStart = nil
local mainStartPos = nil
local mainDragInput = nil

TopBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        mainDragging = true
        mainDragStart = input.Position
        mainStartPos = MainFrame.Position
        mainDragInput = input
    end
end)

TopBar.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        if mainDragging and mainDragInput and input == mainDragInput then
            local delta = input.Position - mainDragStart
            MainFrame.Position = UDim2.new(
                mainStartPos.X.Scale,
                mainStartPos.X.Offset + delta.X,
                mainStartPos.Y.Scale,
                mainStartPos.Y.Offset + delta.Y
            )
        end
    end
end)

TopBar.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        if input == mainDragInput then
            mainDragging = false
            mainDragInput = nil
        end
    end
end)

-- ============================================================
-- CONFIRM REMOVE UI FRAME
-- ============================================================
local ConfirmRemoveFrame = Instance.new("Frame")
ConfirmRemoveFrame.Name = "ConfirmRemoveFrame"
ConfirmRemoveFrame.Size = UDim2.new(0, 260, 0, 140)
ConfirmRemoveFrame.Position = UDim2.new(0.5, -130, 0.5, -70)
ConfirmRemoveFrame.BackgroundColor3 = Color3.fromRGB(20, 30, 48)
ConfirmRemoveFrame.Visible = false
ConfirmRemoveFrame.ZIndex = 30
ConfirmRemoveFrame.Parent = MainFrame

local ConfirmRemoveCorner = Instance.new("UICorner")
ConfirmRemoveCorner.CornerRadius = UDim.new(0, 12)
ConfirmRemoveCorner.Parent = ConfirmRemoveFrame

local ConfirmRemoveStroke = Instance.new("UIStroke")
ConfirmRemoveStroke.Color = Color3.fromRGB(239, 68, 68)
ConfirmRemoveStroke.Thickness = 2
ConfirmRemoveStroke.Parent = ConfirmRemoveFrame

local ConfirmRemoveTitle = Instance.new("TextLabel")
ConfirmRemoveTitle.Size = UDim2.new(1, 0, 0, 38)
ConfirmRemoveTitle.Position = UDim2.new(0, 0, 0, 4)
ConfirmRemoveTitle.BackgroundTransparency = 1
ConfirmRemoveTitle.Text = "⚠️ Remove UI?"
ConfirmRemoveTitle.TextColor3 = Color3.fromRGB(239, 68, 68)
ConfirmRemoveTitle.Font = Enum.Font.GothamBold
ConfirmRemoveTitle.TextSize = 16
ConfirmRemovTitle.ZIndex = 30
ConfirmRemoveTitle.Parent = ConfirmRemoveFrame

local ConfirmRemoveDesc = Instance.new("TextLabel")
ConfirmRemoveDesc.Size = UDim2.new(1, -20, 0, 34)
ConfirmRemoveDesc.Position = UDim2.new(0, 10, 0, 40)
ConfirmRemoveDesc.BackgroundTransparency = 1
ConfirmRemoveDesc.Text = "This will destroy the UI.\nRe-run the script to restore it."
ConfirmRemoveDesc.TextColor3 = Color3.fromRGB(200, 200, 200)
ConfirmRemoveDesc.Font = Enum.Font.Gotham
ConfirmRemoveDesc.TextSize = 12
ConfirmRemoveDesc.TextWrapped = true
ConfirmRemoveDesc.ZIndex = 30
ConfirmRemoveDesc.Parent = ConfirmRemoveFrame

local ConfirmRemoveYesBtn = Instance.new("TextButton")
ConfirmRemoveYesBtn.Size = UDim2.new(0, 105, 0, 32)
ConfirmRemoveYesBtn.Position = UDim2.new(0, 12, 1, -42)
ConfirmRemoveYesBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
ConfirmRemoveYesBtn.Text = "✔️ Yes, Remove"
ConfirmRemoveYesBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ConfirmRemoveYesBtn.Font = Enum.Font.GothamBold
ConfirmRemoveYesBtn.TextSize = 12
ConfirmRemoveYesBtn.ZIndex = 30
ConfirmRemoveYesBtn.Parent = ConfirmRemoveFrame

local ConfirmRemoveYesCorner = Instance.new("UICorner")
ConfirmRemoveYesCorner.CornerRadius = UDim.new(0, 8)
ConfirmRemoveYesCorner.Parent = ConfirmRemoveYesBtn

local ConfirmRemoveNoBtn = Instance.new("TextButton")
ConfirmRemoveNoBtn.Size = UDim2.new(0, 105, 0, 32)
ConfirmRemoveNoBtn.Position = UDim2.new(1, -117, 1, -42)
ConfirmRemoveNoBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
ConfirmRemoveNoBtn.Text = "❌ Cancel"
ConfirmRemoveNoBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ConfirmRemoveNoBtn.Font = Enum.Font.GothamBold
ConfirmRemoveNoBtn.TextSize = 12
ConfirmRemoveNoBtn.ZIndex = 30
ConfirmRemoveNoBtn.Parent = ConfirmRemoveFrame

local ConfirmRemoveNoCorner = Instance.new("UICorner")
ConfirmRemoveNoCorner.CornerRadius = UDim.new(0, 8)
ConfirmRemoveNoCorner.Parent = ConfirmRemoveNoBtn

RemoveUIBtn.MouseButton1Click:Connect(function()
    ConfirmRemoveFrame.Visible = true
end)

ConfirmRemoveNoBtn.MouseButton1Click:Connect(function()
    ConfirmRemoveFrame.Visible = false
end)

ConfirmRemoveYesBtn.MouseButton1Click:Connect(function()
    if rainbowConnection then
        rainbowConnection:Disconnect()
        rainbowConnection = nil
    end
    ScreenGui:Destroy()
end)

-- ============================================================
-- ADD SECTION
-- ============================================================
local AddSection = Instance.new("Frame")
AddSection.Name = "AddSection"
AddSection.Size = UDim2.new(1, -16, 0, 105)
AddSection.Position = UDim2.new(0, 8, 0, 50)
AddSection.BackgroundColor3 = Color3.fromRGB(51, 65, 85)
AddSection.BackgroundTransparency = 0.5
AddSection.BorderSizePixel = 0
AddSection.Parent = MainFrame

local AddCorner = Instance.new("UICorner")
AddCorner.CornerRadius = UDim.new(0, 10)
AddCorner.Parent = AddSection

local NameInput = Instance.new("TextBox")
NameInput.Name = "NameInput"
NameInput.Size = UDim2.new(1, -16, 0, 28)
NameInput.Position = UDim2.new(0, 8, 0, 8)
NameInput.BackgroundColor3 = Color3.fromRGB(15, 23, 42)
NameInput.PlaceholderText = "Script Name..."
NameInput.PlaceholderColor3 = Color3.fromRGB(100, 100, 100)
NameInput.Text = ""
NameInput.TextColor3 = Color3.fromRGB(255, 255, 255)
NameInput.Font = Enum.Font.Gotham
NameInput.TextSize = 13
NameInput.TextXAlignment = Enum.TextXAlignment.Left
NameInput.ClearTextOnFocus = false
NameInput.Parent = AddSection

local NameCorner = Instance.new("UICorner")
NameCorner.CornerRadius = UDim.new(0, 7)
NameCorner.Parent = NameInput

local NamePadding = Instance.new("UIPadding")
NamePadding.PaddingLeft = UDim.new(0, 8)
NamePadding.Parent = NameInput

local CodeInput = Instance.new("TextBox")
CodeInput.Name = "CodeInput"
CodeInput.Size = UDim2.new(1, -105, 0, 38)
CodeInput.Position = UDim2.new(0, 8, 0, 44)
CodeInput.BackgroundColor3 = Color3.fromRGB(13, 17, 23)
CodeInput.PlaceholderText = "Paste script code here..."
CodeInput.PlaceholderColor3 = Color3.fromRGB(100, 100, 100)
CodeInput.Text = ""
CodeInput.TextColor3 = Color3.fromRGB(0, 255, 100)
CodeInput.Font = Enum.Font.Code
CodeInput.TextSize = 11
CodeInput.TextXAlignment = Enum.TextXAlignment.Left
CodeInput.TextWrapped = true
CodeInput.ClearTextOnFocus = false
CodeInput.MultiLine = true
CodeInput.Parent = AddSection

local CodeCorner = Instance.new("UICorner")
CodeCorner.CornerRadius = UDim.new(0, 7)
CodeCorner.Parent = CodeInput

local CodePadding = Instance.new("UIPadding")
CodePadding.PaddingLeft = UDim.new(0, 8)
CodePadding.Parent = CodeInput

local AddButton = Instance.new("TextButton")
AddButton.Name = "AddButton"
AddButton.Size = UDim2.new(0, 88, 0, 38)
AddButton.Position = UDim2.new(1, -96, 0, 44)
AddButton.BackgroundColor3 = Color3.fromRGB(39, 174, 96)
AddButton.Text = "➕ Add"
AddButton.TextColor3 = Color3.fromRGB(255, 255, 255)
AddButton.Font = Enum.Font.GothamBold
AddButton.TextSize = 13
AddButton.Parent = AddSection

local AddBtnCorner = Instance.new("UICorner")
AddBtnCorner.CornerRadius = UDim.new(0, 7)
AddBtnCorner.Parent = AddButton

-- ============================================================
-- LIST CONTAINER
-- ============================================================
local ListContainer = Instance.new("Frame")
ListContainer.Name = "ListContainer"
ListContainer.Size = UDim2.new(1, -16, 1, -168)
ListContainer.Position = UDim2.new(0, 8, 0, 162)
ListContainer.BackgroundColor3 = Color3.fromRGB(51, 65, 85)
ListContainer.BackgroundTransparency = 0.5
ListContainer.BorderSizePixel = 0
ListContainer.ClipsDescendants = true
ListContainer.Parent = MainFrame

local ListCorner = Instance.new("UICorner")
ListCorner.CornerRadius = UDim.new(0, 10)
ListCorner.Parent = ListContainer

local ListTitle = Instance.new("TextLabel")
ListTitle.Size = UDim2.new(1, 0, 0, 26)
ListTitle.BackgroundTransparency = 1
ListTitle.Text = "📂 My Scripts"
ListTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
ListTitle.Font = Enum.Font.GothamBold
ListTitle.TextSize = 13
ListTitle.Parent = ListContainer

-- ScrollingFrame with mouse wheel scroll fix
local ScrollFrame = Instance.new("ScrollingFrame")
ScrollFrame.Name = "ScrollFrame"
ScrollFrame.Size = UDim2.new(1, -8, 1, -30)
ScrollFrame.Position = UDim2.new(0, 4, 0, 26)
ScrollFrame.BackgroundTransparency = 1
ScrollFrame.BorderSizePixel = 0
ScrollFrame.ScrollBarThickness = 4
ScrollFrame.ScrollBarImageColor3 = Color3.fromRGB(0, 217, 255)
ScrollFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
ScrollFrame.ScrollingDirection = Enum.ScrollingDirection.Y
ScrollFrame.ElasticBehavior = Enum.ElasticBehavior.Never
ScrollFrame.Parent = ListContainer

-- Fix scroll: pass mouse wheel events through to the ScrollingFrame
ScrollFrame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseWheel then
        local delta = -input.Position.Z * 40
        ScrollFrame.CanvasPosition = Vector2.new(
            0,
            math.clamp(
                ScrollFrame.CanvasPosition.Y + delta,
                0,
                math.max(0, ScrollFrame.AbsoluteCanvasSize.Y - ScrollFrame.AbsoluteSize.Y)
            )
        )
    end
end)

local ListLayout = Instance.new("UIListLayout")
ListLayout.SortOrder = Enum.SortOrder.LayoutOrder
ListLayout.Padding = UDim.new(0, 4)
ListLayout.Parent = ScrollFrame

local ListPadding = Instance.new("UIPadding")
ListPadding.PaddingTop = UDim.new(0, 4)
ListPadding.PaddingLeft = UDim.new(0, 2)
ListPadding.PaddingRight = UDim.new(0, 2)
ListPadding.Parent = ScrollFrame

local EmptyLabel = Instance.new("TextLabel")
EmptyLabel.Name = "EmptyLabel"
EmptyLabel.Size = UDim2.new(1, 0, 0, 40)
EmptyLabel.BackgroundTransparency = 1
EmptyLabel.Text = "No scripts saved yet! 🚀"
EmptyLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
EmptyLabel.Font = Enum.Font.Gotham
EmptyLabel.TextSize = 13
EmptyLabel.Parent = ScrollFrame

local function updateCanvasSize()
    ScrollFrame.CanvasSize = UDim2.new(0, 0, 0, ListLayout.AbsoluteContentSize.Y + 10)
end

ListLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(updateCanvasSize)

-- ============================================================
-- COLOR PICKER FRAME
-- ============================================================
local ColorPickerFrame = Instance.new("Frame")
ColorPickerFrame.Name = "ColorPickerFrame"
ColorPickerFrame.Size = UDim2.new(0, 300, 0, 390)
ColorPickerFrame.Position = UDim2.new(0.5, -150, 0.5, -195)
ColorPickerFrame.BackgroundColor3 = Color3.fromRGB(30, 41, 59)
ColorPickerFrame.Visible = false
ColorPickerFrame.ZIndex = 10
ColorPickerFrame.Parent = MainFrame

local ColorPickerCorner = Instance.new("UICorner")
ColorPickerCorner.CornerRadius = UDim.new(0, 12)
ColorPickerCorner.Parent = ColorPickerFrame

local ColorPickerStroke = Instance.new("UIStroke")
ColorPickerStroke.Color = themeColor
ColorPickerStroke.Thickness = 2
ColorPickerStroke.Parent = ColorPickerFrame

local ColorPickerTitle = Instance.new("TextLabel")
ColorPickerTitle.Size = UDim2.new(1, 0, 0, 28)
ColorPickerTitle.BackgroundTransparency = 1
ColorPickerTitle.Text = "🎨 Choose UI Color"
ColorPickerTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
ColorPickerTitle.Font = Enum.Font.GothamBold
ColorPickerTitle.TextSize = 13
ColorPickerTitle.ZIndex = 10
ColorPickerTitle.Parent = ColorPickerFrame

local colors = {
    {name = "Red", color = Color3.fromRGB(239, 68, 68)},
    {name = "Crimson", color = Color3.fromRGB(220, 20, 60)},
    {name = "Scarlet", color = Color3.fromRGB(255, 36, 0)},
    {name = "Rose", color = Color3.fromRGB(255, 0, 127)},
    {name = "Coral", color = Color3.fromRGB(255, 127, 80)},
    {name = "Salmon", color = Color3.fromRGB(250, 128, 114)},
    {name = "Orange", color = Color3.fromRGB(249, 115, 22)},
    {name = "Amber", color = Color3.fromRGB(255, 191, 0)},
    {name = "Peach", color = Color3.fromRGB(255, 218, 185)},
    {name = "Rust", color = Color3.fromRGB(183, 65, 14)},
    {name = "Yellow", color = Color3.fromRGB(234, 179, 8)},
    {name = "Gold", color = Color3.fromRGB(255, 215, 0)},
    {name = "Lemon", color = Color3.fromRGB(255, 247, 0)},
    {name = "Cream", color = Color3.fromRGB(255, 253, 208)},
    {name = "Green", color = Color3.fromRGB(34, 197, 94)},
    {name = "Lime", color = Color3.fromRGB(50, 205, 50)},
    {name = "Emerald", color = Color3.fromRGB(0, 201, 87)},
    {name = "Mint", color = Color3.fromRGB(62, 180, 137)},
    {name = "Forest", color = Color3.fromRGB(34, 139, 34)},
    {name = "Olive", color = Color3.fromRGB(128, 128, 0)},
    {name = "Cyan", color = Color3.fromRGB(6, 182, 212)},
    {name = "Teal", color = Color3.fromRGB(0, 128, 128)},
    {name = "Aqua", color = Color3.fromRGB(0, 255, 255)},
    {name = "Turq", color = Color3.fromRGB(64, 224, 208)},
    {name = "Blue", color = Color3.fromRGB(59, 130, 246)},
    {name = "Sky", color = Color3.fromRGB(135, 206, 235)},
    {name = "Navy", color = Color3.fromRGB(0, 0, 128)},
    {name = "Cobalt", color = Color3.fromRGB(0, 71, 171)},
    {name = "Azure", color = Color3.fromRGB(0, 127, 255)},
    {name = "Purple", color = Color3.fromRGB(139, 92, 246)},
    {name = "Indigo", color = Color3.fromRGB(75, 0, 130)},
    {name = "Violet", color = Color3.fromRGB(138, 43, 226)},
    {name = "Lavend", color = Color3.fromRGB(230, 230, 250)},
    {name = "Plum", color = Color3.fromRGB(221, 160, 221)},
    {name = "Pink", color = Color3.fromRGB(236, 72, 153)},
    {name = "Magenta", color = Color3.fromRGB(255, 0, 255)},
    {name = "Fuchsia", color = Color3.fromRGB(255, 0, 255)},
    {name = "HotPink", color = Color3.fromRGB(255, 105, 180)},
    {name = "White", color = Color3.fromRGB(255, 255, 255)},
    {name = "Silver", color = Color3.fromRGB(192, 192, 192)},
    {name = "Gray", color = Color3.fromRGB(128, 128, 128)},
    {name = "Black", color = Color3.fromRGB(0, 0, 0)},
}

local ColorGrid = Instance.new("Frame")
ColorGrid.Size = UDim2.new(1, -16, 0, 295)
ColorGrid.Position = UDim2.new(0, 8, 0, 32)
ColorGrid.BackgroundTransparency = 1
ColorGrid.ZIndex = 10
ColorGrid.Parent = ColorPickerFrame

local ColorGridLayout = Instance.new("UIGridLayout")
ColorGridLayout.CellSize = UDim2.new(0, 60, 0, 25)
ColorGridLayout.CellPadding = UDim2.new(0, 4, 0, 4)
ColorGridLayout.SortOrder = Enum.SortOrder.LayoutOrder
ColorGridLayout.Parent = ColorGrid

local function saveColorToFile(color, isRainbow)
    pcall(function()
        local colorData = {
            isRainbow = isRainbow or false,
            color = {R = math.floor(color.R * 255), G = math.floor(color.G * 255), B = math.floor(color.B * 255)}
        }
        writefile(COLOR_FILE, HttpService:JSONEncode(colorData))
    end)
end

local function updateThemeColor(newColor, save)
    themeColor = newColor
    local tweenInfo = TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
    TweenService:Create(MainStroke, tweenInfo, {Color = newColor}):Play()
    TweenService:Create(ColorPickerStroke, tweenInfo, {Color = newColor}):Play()
    TweenService:Create(ColorButton, tweenInfo, {BackgroundColor3 = newColor}):Play()
    TweenService:Create(Title, tweenInfo, {TextColor3 = newColor}):Play()
    TweenService:Create(ScrollFrame, tweenInfo, {ScrollBarImageColor3 = newColor}):Play()
    if save then
        saveColorToFile(newColor, false)
    end
end

for i, colorData in ipairs(colors) do
    local ColorBtn = Instance.new("TextButton")
    ColorBtn.Name = colorData.name
    ColorBtn.Size = UDim2.new(0, 58, 0, 24)
    ColorBtn.BackgroundColor3 = colorData.color
    ColorBtn.Text = colorData.name
    ColorBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    ColorBtn.Font = Enum.Font.GothamBold
    ColorBtn.TextSize = 9
    ColorBtn.ZIndex = 10
    ColorBtn.LayoutOrder = i
    ColorBtn.Parent = ColorGrid

    local BtnCorner = Instance.new("UICorner")
    BtnCorner.CornerRadius = UDim.new(0, 5)
    BtnCorner.Parent = ColorBtn

    ColorBtn.MouseButton1Click:Connect(function()
        if rainbowConnection then
            rainbowConnection:Disconnect()
            rainbowConnection = nil
        end
        rainbowEnabled = false
        updateThemeColor(colorData.color, true)
        ColorPickerFrame.Visible = false
    end)
end

local RainbowBtn = Instance.new("TextButton")
RainbowBtn.Name = "Rainbow"
RainbowBtn.Size = UDim2.new(1, -16, 0, 28)
RainbowBtn.Position = UDim2.new(0, 8, 0, 332)
RainbowBtn.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
RainbowBtn.Text = "🌈 Rainbow Mode"
RainbowBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
RainbowBtn.Font = Enum.Font.GothamBold
RainbowBtn.TextSize = 12
RainbowBtn.ZIndex = 10
RainbowBtn.Parent = ColorPickerFrame

local RainbowCorner = Instance.new("UICorner")
RainbowCorner.CornerRadius = UDim.new(0, 6)
RainbowCorner.Parent = RainbowBtn

local function startRainbowMode()
    if rainbowConnection then
        rainbowConnection:Disconnect()
    end
    rainbowEnabled = true
    rainbowHue = 0
    rainbowConnection = RunService.Heartbeat:Connect(function(dt)
        rainbowHue = (rainbowHue + dt * 0.2) % 1
        local rainbowColor = Color3.fromHSV(rainbowHue, 1, 1)
        MainStroke.Color = rainbowColor
        ColorPickerStroke.Color = rainbowColor
        ColorButton.BackgroundColor3 = rainbowColor
        Title.TextColor3 = rainbowColor
        ScrollFrame.ScrollBarImageColor3 = rainbowColor
        RainbowBtn.BackgroundColor3 = rainbowColor
    end)
end

RainbowBtn.MouseButton1Click:Connect(function()
    startRainbowMode()
    saveColorToFile(Color3.fromRGB(255, 0, 0), true)
    ColorPickerFrame.Visible = false
end)

local CloseColorBtn = Instance.new("TextButton")
CloseColorBtn.Size = UDim2.new(1, -16, 0, 22)
CloseColorBtn.Position = UDim2.new(0, 8, 1, -28)
CloseColorBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
CloseColorBtn.Text = "Close"
CloseColorBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseColorBtn.Font = Enum.Font.GothamBold
CloseColorBtn.TextSize = 11
CloseColorBtn.ZIndex = 10
CloseColorBtn.Parent = ColorPickerFrame

local CloseColorCorner = Instance.new("UICorner")
CloseColorCorner.CornerRadius = UDim.new(0, 6)
CloseColorCorner.Parent = CloseColorBtn

ColorButton.MouseButton1Click:Connect(function()
    ColorPickerFrame.Visible = not ColorPickerFrame.Visible
end)

CloseColorBtn.MouseButton1Click:Connect(function()
    ColorPickerFrame.Visible = false
end)

-- ============================================================
-- RENAME FRAME
-- ============================================================
local RenameFrame = Instance.new("Frame")
RenameFrame.Name = "RenameFrame"
RenameFrame.Size = UDim2.new(0, 260, 0, 140)
RenameFrame.Position = UDim2.new(0.5, -130, 0.5, -70)
RenameFrame.BackgroundColor3 = Color3.fromRGB(20, 30, 48)
RenameFrame.Visible = false
RenameFrame.ZIndex = 25
RenameFrame.Parent = MainFrame

local RenameFrameCorner = Instance.new("UICorner")
RenameFrameCorner.CornerRadius = UDim.new(0, 12)
RenameFrameCorner.Parent = RenameFrame

local RenameFrameStroke = Instance.new("UIStroke")
RenameFrameStroke.Color = Color3.fromRGB(59, 130, 246)
RenameFrameStroke.Thickness = 2
RenameFrameStroke.Parent = RenameFrame

local RenameTitle = Instance.new("TextLabel")
RenameTitle.Size = UDim2.new(1, 0, 0, 32)
RenameTitle.Position = UDim2.new(0, 0, 0, 0)
RenameTitle.BackgroundTransparency = 1
RenameTitle.Text = "✏️ Rename Script"
RenameTitle.TextColor3 = Color3.fromRGB(59, 130, 246)
RenameTitle.Font = Enum.Font.GothamBold
RenameTitle.TextSize = 14
RenameTitle.ZIndex = 25
RenameTitle.Parent = RenameFrame

local RenameInput = Instance.new("TextBox")
RenameInput.Name = "RenameInput"
RenameInput.Size = UDim2.new(1, -18, 0, 32)
RenameInput.Position = UDim2.new(0, 9, 0, 38)
RenameInput.BackgroundColor3 = Color3.fromRGB(15, 23, 42)
RenameInput.PlaceholderText = "Enter new name..."
RenameInput.PlaceholderColor3 = Color3.fromRGB(100, 100, 100)
RenameInput.Text = ""
RenameInput.TextColor3 = Color3.fromRGB(255, 255, 255)
RenameInput.Font = Enum.Font.Gotham
RenameInput.TextSize = 13
RenameInput.TextXAlignment = Enum.TextXAlignment.Left
RenameInput.ClearTextOnFocus = false
RenameInput.ZIndex = 25
RenameInput.Parent = RenameFrame

local RenameInputCorner = Instance.new("UICorner")
RenameInputCorner.CornerRadius = UDim.new(0, 7)
RenameInputCorner.Parent = RenameInput

local RenameInputPadding = Instance.new("UIPadding")
RenameInputPadding.PaddingLeft = UDim.new(0, 8)
RenameInputPadding.Parent = RenameInput

local RenameConfirmBtn = Instance.new("TextButton")
RenameConfirmBtn.Name = "RenameConfirmBtn"
RenameConfirmBtn.Size = UDim2.new(0, 105, 0, 32)
RenameConfirmBtn.Position = UDim2.new(1, -114, 1, -40)
RenameConfirmBtn.BackgroundColor3 = Color3.fromRGB(59, 130, 246)
RenameConfirmBtn.Text = "✔️ Rename"
RenameConfirmBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
RenameConfirmBtn.Font = Enum.Font.GothamBold
RenameConfirmBtn.TextSize = 13
RenameConfirmBtn.ZIndex = 25
RenameConfirmBtn.Parent = RenameFrame

local RenameConfirmCorner = Instance.new("UICorner")
RenameConfirmCorner.CornerRadius = UDim.new(0, 7)
RenameConfirmCorner.Parent = RenameConfirmBtn

local RenameCancelBtn = Instance.new("TextButton")
RenameCancelBtn.Name = "RenameCancelBtn"
RenameCancelBtn.Size = UDim2.new(0, 82, 0, 32)
RenameCancelBtn.Position = UDim2.new(0, 9, 1, -40)
RenameCancelBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
RenameCancelBtn.Text = "❌ Cancel"
RenameCancelBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
RenameCancelBtn.Font = Enum.Font.GothamBold
RenameCancelBtn.TextSize = 13
RenameCancelBtn.ZIndex = 25
RenameCancelBtn.Parent = RenameFrame

local RenameCancelCorner = Instance.new("UICorner")
RenameCancelCorner.CornerRadius = UDim.new(0, 7)
RenameCancelCorner.Parent = RenameCancelBtn

local currentRenamingIndex = nil
local refreshScriptList

RenameCancelBtn.MouseButton1Click:Connect(function()
    RenameFrame.Visible = false
    currentRenamingIndex = nil
    RenameInput.Text = ""
end)

RenameConfirmBtn.MouseButton1Click:Connect(function()
    local newName = RenameInput.Text
    if currentRenamingIndex and savedScripts[currentRenamingIndex] and newName ~= "" then
        savedScripts[currentRenamingIndex].name = newName
        saveScriptsToFile()
        refreshScriptList()
    end
    RenameFrame.Visible = false
    currentRenamingIndex = nil
    RenameInput.Text = ""
end)

-- ============================================================
-- EDIT FRAME
-- ============================================================
local EditFrame = Instance.new("Frame")
EditFrame.Name = "EditFrame"
EditFrame.Size = UDim2.new(1, -30, 0, 230)
EditFrame.Position = UDim2.new(0, 15, 0.5, -115)
EditFrame.BackgroundColor3 = Color3.fromRGB(20, 30, 48)
EditFrame.Visible = false
EditFrame.ZIndex = 15
EditFrame.Parent = MainFrame

local EditFrameCorner = Instance.new("UICorner")
EditFrameCorner.CornerRadius = UDim.new(0, 12)
EditFrameCorner.Parent = EditFrame

local EditFrameStroke = Instance.new("UIStroke")
EditFrameStroke.Color = Color3.fromRGB(249, 115, 22)
EditFrameStroke.Thickness = 2
EditFrameStroke.Parent = EditFrame

local EditTitle = Instance.new("TextLabel")
EditTitle.Size = UDim2.new(1, 0, 0, 32)
EditTitle.BackgroundTransparency = 1
EditTitle.Text = "📝 Edit Script"
EditTitle.TextColor3 = Color3.fromRGB(249, 115, 22)
EditTitle.Font = Enum.Font.GothamBold
EditTitle.TextSize = 14
EditTitle.ZIndex = 15
EditTitle.Parent = EditFrame

local EditCodeBox = Instance.new("TextBox")
EditCodeBox.Name = "EditCodeBox"
EditCodeBox.Size = UDim2.new(1, -16, 1, -85)
EditCodeBox.Position = UDim2.new(0, 8, 0, 36)
EditCodeBox.BackgroundColor3 = Color3.fromRGB(15, 23, 42)
EditCodeBox.TextColor3 = Color3.fromRGB(255, 255, 255)
EditCodeBox.Font = Enum.Font.Code
EditCodeBox.TextSize = 13
EditCodeBox.MultiLine = true
EditCodeBox.ClearTextOnFocus = false
EditCodeBox.ZIndex = 15
EditCodeBox.Parent = EditFrame

local EditCorner = Instance.new("UICorner")
EditCorner.CornerRadius = UDim.new(0, 7)
EditCorner.Parent = EditCodeBox

local EditBtnContainer = Instance.new("Frame")
EditBtnContainer.Name = "EditBtnContainer"
EditBtnContainer.Size = UDim2.new(1, -16, 0, 36)
EditBtnContainer.Position = UDim2.new(0, 8, 1, -44)
EditBtnContainer.BackgroundTransparency = 1
EditBtnContainer.ZIndex = 15
EditBtnContainer.Parent = EditFrame

local SaveEditBtn = Instance.new("TextButton")
SaveEditBtn.Name = "SaveEditBtn"
SaveEditBtn.Size = UDim2.new(0, 90, 1, 0)
SaveEditBtn.Position = UDim2.new(1, -90, 0, 0)
SaveEditBtn.BackgroundColor3 = Color3.fromRGB(39, 174, 96)
SaveEditBtn.Text = "💾 Save"
SaveEditBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
SaveEditBtn.Font = Enum.Font.GothamBold
SaveEditBtn.TextSize = 13
SaveEditBtn.ZIndex = 15
SaveEditBtn.Parent = EditBtnContainer

local SaveEditCorner = Instance.new("UICorner")
SaveEditCorner.CornerRadius = UDim.new(0, 7)
SaveEditCorner.Parent = SaveEditBtn

local CancelEditBtn = Instance.new("TextButton")
CancelEditBtn.Name = "CancelEditBtn"
CancelEditBtn.Size = UDim2.new(0, 90, 1, 0)
CancelEditBtn.Position = UDim2.new(0, 0, 0, 0)
CancelEditBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
CancelEditBtn.Text = "❌ Cancel"
CancelEditBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CancelEditBtn.Font = Enum.Font.GothamBold
CancelEditBtn.TextSize = 13
CancelEditBtn.ZIndex = 15
CancelEditBtn.Parent = EditBtnContainer

local CancelEditCorner = Instance.new("UICorner")
CancelEditCorner.CornerRadius = UDim.new(0, 7)
CancelEditCorner.Parent = CancelEditBtn

local currentEditingIndex = nil

CancelEditBtn.MouseButton1Click:Connect(function()
    EditFrame.Visible = false
    currentEditingIndex = nil
end)

SaveEditBtn.MouseButton1Click:Connect(function()
    if currentEditingIndex and savedScripts[currentEditingIndex] then
        savedScripts[currentEditingIndex].code = EditCodeBox.Text
        saveScriptsToFile()
        refreshScriptList()
    end
    EditFrame.Visible = false
    currentEditingIndex = nil
end)

-- ============================================================
-- CONFIRM DELETE FRAME
-- ============================================================
local ConfirmFrame = Instance.new("Frame")
ConfirmFrame.Name = "ConfirmFrame"
ConfirmFrame.Size = UDim2.new(0, 245, 0, 130)
ConfirmFrame.Position = UDim2.new(0.5, -122, 0.5, -65)
ConfirmFrame.BackgroundColor3 = Color3.fromRGB(30, 41, 59)
ConfirmFrame.Visible = false
ConfirmFrame.ZIndex = 20
ConfirmFrame.Parent = MainFrame

local ConfirmCorner = Instance.new("UICorner")
ConfirmCorner.CornerRadius = UDim.new(0, 12)
ConfirmCorner.Parent = ConfirmFrame

local ConfirmStroke = Instance.new("UIStroke")
ConfirmStroke.Color = Color3.fromRGB(239, 68, 68)
ConfirmStroke.Thickness = 2
ConfirmStroke.Parent = ConfirmFrame

local ConfirmTitle = Instance.new("TextLabel")
ConfirmTitle.Size = UDim2.new(1, 0, 0, 36)
ConfirmTitle.Position = UDim2.new(0, 0, 0, 4)
ConfirmTitle.BackgroundTransparency = 1
ConfirmTitle.Text = "⚠️ Are you sure?"
ConfirmTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
ConfirmTitle.Font = Enum.Font.GothamBold
ConfirmTitle.TextSize = 16
ConfirmTitle.ZIndex = 20
ConfirmTitle.Parent = ConfirmFrame

local ConfirmDesc = Instance.new("TextLabel")
ConfirmDesc.Size = UDim2.new(1, -18, 0, 26)
ConfirmDesc.Position = UDim2.new(0, 9, 0, 38)
ConfirmDesc.BackgroundTransparency = 1
ConfirmDesc.Text = "Delete this script permanently?"
ConfirmDesc.TextColor3 = Color3.fromRGB(200, 200, 200)
ConfirmDesc.Font = Enum.Font.Gotham
ConfirmDesc.TextSize = 13
ConfirmDesc.TextWrapped = true
ConfirmDesc.ZIndex = 20
ConfirmDesc.Parent = ConfirmFrame

local ConfirmYesBtn = Instance.new("TextButton")
ConfirmYesBtn.Size = UDim2.new(0, 95, 0, 32)
ConfirmYesBtn.Position = UDim2.new(0, 14, 1, -42)
ConfirmYesBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
ConfirmYesBtn.Text = "Yes, Delete"
ConfirmYesBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ConfirmYesBtn.Font = Enum.Font.GothamBold
ConfirmYesBtn.TextSize = 13
ConfirmYesBtn.ZIndex = 20
ConfirmYesBtn.Parent = ConfirmFrame

local ConfirmYesCorner = Instance.new("UICorner")
ConfirmYesCorner.CornerRadius = UDim.new(0, 7)
ConfirmYesCorner.Parent = ConfirmYesBtn

local ConfirmNoBtn = Instance.new("TextButton")
ConfirmNoBtn.Size = UDim2.new(0, 95, 0, 32)
ConfirmNoBtn.Position = UDim2.new(1, -109, 1, -42)
ConfirmNoBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
ConfirmNoBtn.Text = "Cancel"
ConfirmNoBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ConfirmNoBtn.Font = Enum.Font.GothamBold
ConfirmNoBtn.TextSize = 13
ConfirmNoBtn.ZIndex = 20
ConfirmNoBtn.Parent = ConfirmFrame

local ConfirmNoCorner = Instance.new("UICorner")
ConfirmNoCorner.CornerRadius = UDim.new(0, 7)
ConfirmNoCorner.Parent = ConfirmNoBtn

local scriptToDeleteIndex = nil

ConfirmNoBtn.MouseButton1Click:Connect(function()
    ConfirmFrame.Visible = false
    scriptToDeleteIndex = nil
end)

ConfirmYesBtn.MouseButton1Click:Connect(function()
    if scriptToDeleteIndex and savedScripts[scriptToDeleteIndex] then
        table.remove(savedScripts, scriptToDeleteIndex)
        saveScriptsToFile()
        refreshScriptList()
    end
    ConfirmFrame.Visible = false
    scriptToDeleteIndex = nil
end)

-- ============================================================
-- RESIZE HANDLE
-- ============================================================
local ResizeHandle = Instance.new("TextButton")
ResizeHandle.Name = "ResizeHandle"
ResizeHandle.Size = UDim2.new(0, 18, 0, 18)
ResizeHandle.Position = UDim2.new(1, -18, 1, -18)
ResizeHandle.BackgroundTransparency = 1
ResizeHandle.Text = "◢"
ResizeHandle.TextColor3 = Color3.fromRGB(100, 100, 100)
ResizeHandle.TextSize = 13
ResizeHandle.Parent = MainFrame

local resizing = false
local resizeStart = nil
local startSize = nil
local resizeInput = nil
local MIN_SIZE_X = 340
local MIN_SIZE_Y = 360

ResizeHandle.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        resizing = true
        resizeStart = input.Position
        startSize = MainFrame.AbsoluteSize
        resizeInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if resizing and input == resizeInput then
        local delta = input.Position - resizeStart
        local newWidth = math.max(MIN_SIZE_X, startSize.X + delta.X)
        local newHeight = math.max(MIN_SIZE_Y, startSize.Y + delta.Y)
        MainFrame.Size = UDim2.new(0, newWidth, 0, newHeight)
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input == resizeInput then
        resizing = false
        resizeInput = nil
    end
end)

-- ============================================================
-- REFRESH SCRIPT LIST
-- ============================================================
refreshScriptList = function()
    for _, child in ipairs(ScrollFrame:GetChildren()) do
        if child:IsA("Frame") then
            child:Destroy()
        end
    end

    if #savedScripts == 0 then
        EmptyLabel.Visible = true
    else
        EmptyLabel.Visible = false
    end

    for i, scriptData in ipairs(savedScripts) do
        local ScriptItem = Instance.new("Frame")
        ScriptItem.Name = "ScriptItem_" .. i
        ScriptItem.Size = UDim2.new(1, 0, 0, 36)
        ScriptItem.BackgroundColor3 = Color3.fromRGB(15, 23, 42)
        ScriptItem.BorderSizePixel = 0
        ScriptItem.LayoutOrder = i
        ScriptItem.Parent = ScrollFrame

        local ItemCorner = Instance.new("UICorner")
        ItemCorner.CornerRadius = UDim.new(0, 7)
        ItemCorner.Parent = ScriptItem

        local ScriptName = Instance.new("TextLabel")
        ScriptName.Size = UDim2.new(1, -170, 1, 0)
        ScriptName.Position = UDim2.new(0, 8, 0, 0)
        ScriptName.BackgroundTransparency = 1
        ScriptName.Text = scriptData.name
        ScriptName.TextColor3 = Color3.fromRGB(255, 255, 255)
        ScriptName.Font = Enum.Font.GothamSemibold
        ScriptName.TextSize = 13
        ScriptName.TextXAlignment = Enum.TextXAlignment.Left
        ScriptName.TextTruncate = Enum.TextTruncate.AtEnd
        ScriptName.Parent = ScriptItem

        local BtnContainer = Instance.new("Frame")
        BtnContainer.Size = UDim2.new(0, 162, 1, 0)
        BtnContainer.Position = UDim2.new(1, -162, 0, 0)
        BtnContainer.BackgroundTransparency = 1
        BtnContainer.Parent = ScriptItem

        local BtnLayout = Instance.new("UIListLayout")
        BtnLayout.FillDirection = Enum.FillDirection.Horizontal
        BtnLayout.HorizontalAlignment = Enum.HorizontalAlignment.Right
        BtnLayout.VerticalAlignment = Enum.VerticalAlignment.Center
        BtnLayout.SortOrder = Enum.SortOrder.LayoutOrder
        BtnLayout.Padding = UDim.new(0, 4)
        BtnLayout.Parent = BtnContainer

        local BtnPadding = Instance.new("UIPadding")
        BtnPadding.PaddingRight = UDim.new(0, 4)
        BtnPadding.Parent = BtnContainer

        local function createSmallBtn(text, color, layoutOrder)
            local btn = Instance.new("TextButton")
            btn.Size = UDim2.new(0, 27, 0, 27)
            btn.BackgroundColor3 = color
            btn.Text = text
            btn.TextColor3 = Color3.fromRGB(255, 255, 255)
            btn.TextSize = 11
            btn.Font = Enum.Font.GothamBold
            btn.LayoutOrder = layoutOrder
            btn.Parent = BtnContainer

            local c = Instance.new("UICorner")
            c.CornerRadius = UDim.new(0, 6)
            c.Parent = btn
            return btn
        end

        -- 1) Move button
        local MoveBtn = createSmallBtn("↕", Color3.fromRGB(59, 130, 246), 1)

        MoveBtn.MouseButton1Click:Connect(function()
            if i > 1 then
                local temp = savedScripts[i]
                savedScripts[i] = savedScripts[i - 1]
                savedScripts[i - 1] = temp
                saveScriptsToFile()
                refreshScriptList()
            end
        end)

        MoveBtn.MouseButton2Click:Connect(function()
            if i < #savedScripts then
                local temp = savedScripts[i]
                savedScripts[i] = savedScripts[i + 1]
                savedScripts[i + 1] = temp
                saveScriptsToFile()
                refreshScriptList()
            end
        end)

        -- 2) Execute button
        local ExecBtn = createSmallBtn("▶", Color3.fromRGB(39, 174, 96), 2)
        ExecBtn.MouseButton1Click:Connect(function()
            local success, err = pcall(function()
                loadstring(scriptData.code)()
            end)
            if not success then
                warn("Script Execution Error: " .. tostring(err))
            end
        end)

        -- 3) Rename button
        local RenameBtn = createSmallBtn("🏷", Color3.fromRGB(59, 130, 246), 3)
        RenameBtn.MouseButton1Click:Connect(function()
            currentRenamingIndex = i
            RenameInput.Text = scriptData.name
            RenameFrame.Visible = true
        end)

        -- 4) Edit button
        local EditBtn = createSmallBtn("✏", Color3.fromRGB(249, 115, 22), 4)
        EditBtn.MouseButton1Click:Connect(function()
            currentEditingIndex = i
            EditCodeBox.Text = scriptData.code
            EditFrame.Visible = true
        end)

        -- 5) Delete button
        local DelBtn = createSmallBtn("🗑", Color3.fromRGB(239, 68, 68), 5)
        DelBtn.MouseButton1Click:Connect(function()
            scriptToDeleteIndex = i
            ConfirmFrame.Visible = true
        end)
    end
end

-- ============================================================
-- ADD BUTTON LOGIC
-- ============================================================
AddButton.MouseButton1Click:Connect(function()
    local name = NameInput.Text
    local code = CodeInput.Text

    if name == "" or code == "" then
        return
    end

    table.insert(savedScripts, {
        id = HttpService:GenerateGUID(false),
        name = name,
        code = code
    })

    saveScriptsToFile()
    refreshScriptList()

    NameInput.Text = ""
    CodeInput.Text = ""
end)

-- ============================================================
-- INITIAL LOAD
-- ============================================================
savedScripts = loadScriptsFromFile()
refreshScriptList()

local function loadColorSettings()
    if isfile(COLOR_FILE) then
        local success, result = pcall(function()
            return HttpService:JSONDecode(readfile(COLOR_FILE))
        end)
        if success and result then
            if result.color then
                local col = Color3.fromRGB(result.color.R, result.color.G, result.color.B)
                updateThemeColor(col, false)
            end
            if result.isRainbow then
                startRainbowMode()
            end
        end
    end
end
task.spawn(loadColorSettings)

print("Script Saver GUI Loaded")
