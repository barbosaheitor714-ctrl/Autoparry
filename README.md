-- [[ AUTO PARRY + SPAM UNIVERSAL COM INTERFACE ]] --

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local LocalPlayer = Players.LocalPlayer

-- Variáveis de Controle (Ativadas pelos Botões)
local AutoParryEnabled = false
local SpamEnabled = false

-- Configurações Ajustáveis
local PARRY_DISTANCE = 20 -- Distância para o Auto Parry ativar
local SPAM_SPEED = 0.01    -- Velocidade do Spam (quanto menor, mais rápido)

-- --- FUNÇÕES DE SIMULAÇÃO DE INPUT ---
local function pressParryKey()
    -- Simula a tecla F (comum em jogos de PC/Mobile emulados)
    VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.F, false, game)
    task.wait(0.01)
    VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.F, false, game)
end

-- --- LÓGICA DO AUTO PARRY ---
local function findBall()
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return nil end
    local myPos = char.HumanoidRootPart.Position
    
    -- Nomes comuns que desenvolvedores usam para bolas/projéteis
    local commonNames = {"Ball", "Bola", "Projectile", "EnergyBall", "BladeBall"}
    
    -- 1. Tenta achar por nome comum na Workspace
    for _, name in ipairs(commonNames) do
        local obj = workspace:FindFirstChild(name, true) -- busca profunda
        if obj and obj:IsA("BasePart") then
            return obj
        end
    end
    
    -- 2. Se não achar por nome, procura o objeto físico mais próximo se movendo rápido
    for _, obj in ipairs(workspace:GetChildren()) do
        if obj:IsA("BasePart") and obj.Name ~= char.Name then
            local dist = (obj.Position - myPos).Magnitude
            if dist < 60 and obj.AssemblyLinearVelocity.Magnitude > 10 then
                return obj
            end
        end
    end
    return nil
end

-- Loop do Auto Parry
RunService.RenderStepped:Connect(function()
    if not AutoParryEnabled then return end
    
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    
    local ball = findBall()
    if ball then
        local myPos = char.HumanoidRootPart.Position
        local ballPos = ball.Position
        local distance = (ballPos - myPos).Magnitude
        
        -- Se a bola estiver dentro do raio de perigo, defende
        if distance <= PARRY_DISTANCE then
            pressParryKey()
        end
    end
end)

-- Loop do Spam (Clona cliques rápidos quando ativado)
task.spawn(function()
    while true do
        task.wait(SPAM_SPEED)
        if SpamEnabled then
            pressParryKey()
        end
    end
end)

-- --- CRIAÇÃO DA INTERFACE VISUAL (GUI) ---
-- Criando a tela
local ScreenGui = Instance.new("ScreenGui")
local MainFrame = Instance.new("Frame")
local Title = Instance.new("TextLabel")
local ParryBtn = Instance.new("TextButton")
local SpamBtn = Instance.new("TextButton")

ScreenGui.Parent = game:GetService("CoreGui")
ScreenGui.ResetOnSpawn = false

-- Estilização do Menu
MainFrame.Name = "DeltaScriptMenu"
MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
MainFrame.Position = UDim2.new(0.1, 0, 0.3, 0)
MainFrame.Size = UDim2.new(0, 180, 0, 160)
MainFrame.Active = true
MainFrame.Draggable = true -- Permite arrastar o menu pela tela do celular

Title.Parent = MainFrame
Title.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
Title.Size = UDim2.new(1, 0, 0, 30)
Title.Font = Enum.Font.SourceSansBold
Title.Text = "PARRY / SPAM"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = 16

-- Botão 1: Auto Parry
ParryBtn.Parent = MainFrame
ParryBtn.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
ParryBtn.Position = UDim2.new(0.05, 0, 0.25, 0)
ParryBtn.Size = UDim2.new(0.9, 0, 0, 35)
ParryBtn.Font = Enum.Font.SourceSans
ParryBtn.Text = "Auto Parry: DESLIGADO"
ParryBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ParryBtn.TextSize = 14

ParryBtn.MouseButton1Click:Connect(function()
    AutoParryEnabled = not AutoParryEnabled
    if AutoParryEnabled then
        ParryBtn.Text = "Auto Parry: LIGADO"
        ParryBtn.BackgroundColor3 = Color3.fromRGB(50, 180, 50)
    else
        ParryBtn.Text = "Auto Parry: DESLIGADO"
        ParryBtn.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
    end
end)

-- Botão 2: Spam Parry
SpamBtn.Parent = MainFrame
SpamBtn.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
SpamBtn.Position = UDim2.new(0.05, 0, 0.6, 0)
SpamBtn.Size = UDim2.new(0.9, 0, 0, 35)
SpamBtn.Font = Enum.Font.SourceSans
SpamBtn.Text = "Spam: DESLIGADO"
SpamBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
SpamBtn.TextSize = 14

SpamBtn.MouseButton1Click:Connect(function()
    SpamEnabled = not SpamEnabled
    if SpamEnabled then
        SpamBtn.Text = "Spam: LIGADO"
        SpamBtn.BackgroundColor3 = Color3.fromRGB(50, 180, 50)
    else
        SpamBtn.Text = "Spam: DESLIGADO"
        SpamBtn.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
    end
end)

print("[Delta Menu] Carregado com sucesso!")
