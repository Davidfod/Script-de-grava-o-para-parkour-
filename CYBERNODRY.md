--[[
    CYBERNODRY HUB - MOTION RECORDER PRO (FINAL STABLE)
    - Correções de segurança (Character/RootPart checks)
    - Anti-corrupção de arquivos JSON
    - Sistema de Fila de Execução Limpa
    - By: CyberNoDry
]]

local Player = game:GetService("Players").LocalPlayer
local RunService = game:GetService("RunService")
local HttpService = game:GetService("HttpService")
local CoreGui = game:GetService("CoreGui")

local Recording = false
local Replaying = false
local CurrentData = {}
local FILE_NAME = "CyberNoDry_Macros.json"

-- Funções de Banco de Dados Protegidas
local function LoadStorage()
    local success, content = pcall(function()
        if isfile and isfile(FILE_NAME) then
            local data = readfile(FILE_NAME)
            return HttpService:JSONDecode(data)
        end
    end)
    return (success and type(content) == "table") and content or {}
end

local function SaveToDisk(name, data)
    if #data == 0 then return end
    local allSaves = LoadStorage()
    allSaves[name] = data
    pcall(function() 
        writefile(FILE_NAME, HttpService:JSONEncode(allSaves)) 
    end)
end

local function DeleteSave(name)
    local allSaves = LoadStorage()
    if name == "ALL" then allSaves = {} else allSaves[name] = nil end
    pcall(function() 
        writefile(FILE_NAME, HttpService:JSONEncode(allSaves)) 
    end)
end

-- Interface Profissional
local function Create(cls, props)
    local inst = Instance.new(cls)
    for i, v in pairs(props) do inst[i] = v end
    return inst
end

local ScreenGui = Create("ScreenGui", { Name = "CyberRecorder_Final", Parent = CoreGui, ResetOnSpawn = false })
local Main = Create("Frame", {
    Parent = ScreenGui, BackgroundColor3 = Color3.fromRGB(12, 12, 12),
    Position = UDim2.new(0.5, -115, 0.3, 0), Size = UDim2.new(0, 230, 0, 320),
    Active = true, Draggable = true 
})
Create("UICorner", { Parent = Main })
Create("UIStroke", { Parent = Main, Thickness = 2, Color = Color3.fromRGB(255, 215, 0) })

local Title = Create("TextLabel", { Parent = Main, Text = "CYBER RECORDER PRO", Size = UDim2.new(1, 0, 0, 35), BackgroundTransparency = 1, TextColor3 = Color3.fromRGB(255, 215, 0), Font = Enum.Font.GothamBold, TextSize = 14 })

local NameInput = Create("TextBox", {
    Parent = Main, PlaceholderText = "Nome da Gravação...",
    Text = "", Position = UDim2.new(0.1, 0, 0.12, 0), Size = UDim2.new(0.8, 0, 0, 30),
    BackgroundColor3 = Color3.fromRGB(30, 30, 30), TextColor3 = Color3.new(1,1,1),
    Font = Enum.Font.Gotham, TextSize = 11
})
Create("UICorner", { Parent = NameInput })

local function StyledBtn(txt, pos, color)
    local b = Create("TextButton", { Parent = Main, Text = txt, Position = pos, Size = UDim2.new(0.8, 0, 0, 30), BackgroundColor3 = color, TextColor3 = Color3.new(1,1,1), Font = Enum.Font.GothamBold, TextSize = 11 })
    Create("UICorner", { Parent = b })
    return b
end

local RecBtn = StyledBtn("GRAVAR MOVIMENTOS", UDim2.new(0.1, 0, 0.25, 0), Color3.fromRGB(80, 0, 0))
local StopBtn = StyledBtn("PARAR / FINALIZAR", UDim2.new(0.1, 0, 0.37, 0), Color3.fromRGB(40, 40, 40))
local SaveBtn = StyledBtn("SALVAR NO BANCO", UDim2.new(0.1, 0, 0.49, 0), Color3.fromRGB(0, 80, 0))
local TableBtn = StyledBtn("MEUS SAVES 📂", UDim2.new(0.1, 0, 0.61, 0), Color3.fromRGB(20, 20, 20))
local ClearBtn = StyledBtn("LIMPAR ARQUIVOS 🧹", UDim2.new(0.1, 0, 0.73, 0), Color3.fromRGB(130, 80, 0))

local ByLabel = Create("TextLabel", { Parent = Main, Text = "By: CyberNoDry", Position = UDim2.new(0, 10, 1, -20), Size = UDim2.new(0, 100, 0, 20), BackgroundTransparency = 1, TextColor3 = Color3.fromRGB(100, 100, 100), Font = Enum.Font.Code, TextSize = 10, TextXAlignment = Enum.TextXAlignment.Left })

local ListFrame = Create("ScrollingFrame", { Parent = Main, Visible = false, BackgroundColor3 = Color3.fromRGB(15, 15, 15), Position = UDim2.new(1, 10, 0, 0), Size = UDim2.new(0, 200, 1, 0), CanvasSize = UDim2.new(0, 0, 0, 0), AutomaticCanvasSize = Enum.AutomaticSize.Y })
Create("UIStroke", { Parent = ListFrame, Color = Color3.fromRGB(255, 215, 0) })
Create("UIListLayout", { Parent = ListFrame, Padding = UDim.new(0, 5) })

-- Gravação de Frames
RunService.Heartbeat:Connect(function()
    if Recording then
        local Char = Player.Character
        if Char and Char:FindFirstChild("HumanoidRootPart") then
            table.insert(CurrentData, {
                Pos = {Char.HumanoidRootPart.CFrame:GetComponents()},
                Jump = Char.Humanoid.Jump
            })
        end
    end
end)

-- Lógica de Execução Estável
local function PlayMacro(data)
    if Replaying or #data == 0 then return end
    Replaying = true
    
    task.spawn(function()
        local Char = Player.Character
        local Hum = Char and Char:FindFirstChild("Humanoid")
        local Root = Char and Char:FindFirstChild("HumanoidRootPart")
        
        if not Hum or not Root then Replaying = false return end
        
        -- Caminhada inicial para evitar teleporte bruto
        Hum:MoveTo(CFrame.new(unpack(data[1].Pos)).Position)
        local arrived = false
        local connection = Hum.MoveToFinished:Connect(function() arrived = true end)
        task.delay(5, function() arrived = true end) -- Timeout de segurança
        repeat task.wait() until arrived
        connection:Disconnect()

        -- Replay dos frames
        for _, frame in ipairs(data) do
            if not Replaying or not Root then break end
            Root.CFrame = CFrame.new(unpack(frame.Pos))
            if frame.Jump then Hum.Jump = true end
            RunService.Heartbeat:Wait()
        end
        Replaying = false
    end)
end

-- Handlers dos Botões
RecBtn.MouseButton1Click:Connect(function()
    if not Replaying then
        Recording, CurrentData = true, {}
        RecBtn.Text = "GRAVANDO... 🔴"
    end
end)

StopBtn.MouseButton1Click:Connect(function()
    Recording, Replaying = false, false
    RecBtn.Text = "GRAVAR MOVIMENTOS"
end)

SaveBtn.MouseButton1Click:Connect(function()
    local name = NameInput.Text ~= "" and NameInput.Text or "Macro_"..os.time()
    SaveToDisk(name, CurrentData)
    NameInput.Text = ""
end)

local function UpdateTable()
    for _, c in pairs(ListFrame:GetChildren()) do if c:IsA("Frame") then c:Destroy() end end
    local saves = LoadStorage()
    for name, data in pairs(saves) do
        local Item = Create("Frame", { Parent = ListFrame, Size = UDim2.new(1, -10, 0, 35), BackgroundTransparency = 1 })
        local b = Create("TextButton", { Parent = Item, Text = name, Size = UDim2.new(0.75, 0, 1, 0), BackgroundColor3 = Color3.fromRGB(30, 30, 30), TextColor3 = Color3.new(1,1,1), Font = Enum.Font.Gotham })
        local del = Create("TextButton", { Parent = Item, Text = "🗑️", Position = UDim2.new(0.78, 0, 0, 0), Size = UDim2.new(0.2, 0, 1, 0), BackgroundColor3 = Color3.fromRGB(100, 0, 0), TextColor3 = Color3.new(1,1,1) })
        Create("UICorner", { Parent = b }) Create("UICorner", { Parent = del })
        b.MouseButton1Click:Connect(function() PlayMacro(data) end)
        del.MouseButton1Click:Connect(function() DeleteSave(name) UpdateTable() end)
    end
end

TableBtn.MouseButton1Click:Connect(function() ListFrame.Visible = not ListFrame.Visible if ListFrame.Visible then UpdateTable() end end)
ClearBtn.MouseButton1Click:Connect(function() DeleteSave("ALL") UpdateTable() end)

-- Minimizar e Fechar
local isMin = false
local MinBtn = Create("TextButton", { Parent = Main, Text = "_", Position = UDim2.new(1, -50, 0, 5), Size = UDim2.new(0, 20, 0, 20), BackgroundTransparency = 1, TextColor3 = Color3.new(1,1,1), Font = Enum.Font.GothamBold, TextSize = 18 })
local CloseBtn = Create("TextButton", { Parent = Main, Text = "X", Position = UDim2.new(1, -25, 0, 5), Size = UDim2.new(0, 20, 0, 20), BackgroundTransparency = 1, TextColor3 = Color3.fromRGB(255, 50, 50), Font = Enum.Font.GothamBold, TextSize = 16 })

MinBtn.MouseButton1Click:Connect(function()
    isMin = not isMin
    Main:TweenSize(isMin and UDim2.new(0, 230, 0, 35) or UDim2.new(0, 230, 0, 320), "Out", "Quad", 0.3, true)
    for _, v in pairs(Main:GetChildren()) do
        if v:IsA("GuiObject") and v ~= Title and v ~= MinBtn and v ~= CloseBtn then v.Visible = not isMin end
    end
end)
CloseBtn.MouseButton1Click:Connect(function() ScreenGui:Destroy() end)
