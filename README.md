local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local mouse = player:GetMouse()

print("[IWUANK] Script iniciando...")

-- Aguardar o player entrar no jogo
if not player.Character then
	player.CharacterAdded:Wait()
end

local character = player.Character
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

print("[IWUANK] Character encontrado!")

-- Criar ScreenGui
local gui = Instance.new("ScreenGui")
gui.Name = "IwuankGui"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

-- Criar Botão
local btn = Instance.new("TextButton")
btn.Name = "AimBtn"
btn.Size = UDim2.new(0, 50, 0, 50)
btn.Position = UDim2.new(0.5, -25, 0.85, 0) -- CORRIGIDO: -25 para 50x50
btn.BackgroundColor3 = Color3.new(0, 0, 0)
btn.TextColor3 = Color3.new(1, 1, 1)
btn.Text = "tech iwuank"
btn.TextSize = 8
btn.Font = Enum.Font.GothamBold
btn.Parent = gui

print("[IWUANK] Botão criado!")

-- Variáveis
local AIMANDO = false
local SMOOTHING = 0.35
local DISTANCIA = 150

-- Movimento do botão
local clicando = false

btn.InputBegan:Connect(function(input, gameProcessed)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		clicando = true
	end
end)

btn.InputEnded:Connect(function(input, gameProcessed)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		clicando = false
	end
end)

RunService.RenderStepped:Connect(function()
	if clicando then
		btn.Position = UDim2.new(0, mouse.X - 25, 0, mouse.Y - 25) -- CORRIGIDO: -25 para 50x50
	end
end)

-- Função para encontrar inimigo mais perto
local function acharInimigo()
	local maisPerto = nil
	local distancia = DISTANCIA
	
	for _, p in pairs(Players:GetPlayers()) do
		if p ~= player and p.Character then
			local root = p.Character:FindFirstChild("HumanoidRootPart")
			local hum = p.Character:FindFirstChild("Humanoid")
			
			if root and hum and hum.Health > 0 then
				local dist = (humanoidRootPart.Position - root.Position).Magnitude
				if dist < distancia then
					distancia = dist
					maisPerto = root
				end
			end
		end
	end
	
	return maisPerto
end

-- Cores rainbow
local cores = {
	Color3.new(1, 0, 0),       -- Vermelho
	Color3.new(1, 0.5, 0),     -- Laranja
	Color3.new(1, 1, 0),       -- Amarelo
	Color3.new(0, 1, 0),       -- Verde
	Color3.new(0, 0, 1),       -- Azul
	Color3.new(0.5, 0, 1),     -- Índigo
	Color3.new(1, 0, 1),       -- Violeta
}
local indice = 1

-- Função criar aura
local function criarAura(inimigo)
	local aura = inimigo:FindFirstChild("AuraRainbow")
	if aura then
		aura:Destroy()
	end
	
	local nova = Instance.new("Part")
	nova.Name = "AuraRainbow"
	nova.Shape = Enum.PartType.Ball
	nova.Size = Vector3.new(6, 6, 6)
	nova.CanCollide = false
	nova.CFrame = inimigo.CFrame
	nova.Transparency = 0.4
	nova.Parent = inimigo
	
	local w = Instance.new("WeldConstraint")
	w.Part0 = inimigo
	w.Part1 = nova
	w.Parent = nova
	
	return nova
end

-- Função remover auras
local function removerAuras()
	for _, p in pairs(Players:GetPlayers()) do
		if p.Character then
			local root = p.Character:FindFirstChild("HumanoidRootPart")
			if root then
				local aura = root:FindFirstChild("AuraRainbow")
				if aura then
					aura:Destroy()
				end
			end
		end
	end
end

-- Clique no botão para TOGGLE
local ultimoClique = 0
btn.MouseButton1Up:Connect(function()
	local tempoAgora = tick()
	if tempoAgora - ultimoClique > 0.2 and not clicando then
		AIMANDO = not AIMANDO
		ultimoClique = tempoAgora
		
		if AIMANDO then
			btn.BackgroundColor3 = Color3.new(0, 1, 0)
			print("[IWUANK] ✓ AIM LIGADO")
		else
			btn.BackgroundColor3 = Color3.new(0, 0, 0)
			removerAuras()
			print("[IWUANK] ✗ AIM DESLIGADO")
		end
	end
end)

-- Loop principal
local alvoAtual = nil

RunService.RenderStepped:Connect(function()
	if not character or not humanoidRootPart or humanoidRootPart.Parent == nil then
		return
	end
	
	if AIMANDO then
		local inimigo = acharInimigo()
		
		if inimigo then
			-- Trocar alvo?
			if alvoAtual ~= inimigo then
				removerAuras()
				alvoAtual = inimigo
				criarAura(inimigo)
			end
			
			-- Animar cores da aura
			local aura = inimigo:FindFirstChild("AuraRainbow")
			if aura then
				aura.Color = cores[indice]
			end
			indice = indice + 1
			if indice > #cores then
				indice = 1
			end
			
			-- MIRAR NAS COSTAS
			local direcao = inimigo.CFrame.LookVector
			local posCostas = inimigo.Position - (direcao * 3)
			
			local vetorAim = (posCostas - humanoidRootPart.Position).Unit
			local cframeMira = CFrame.new(humanoidRootPart.Position, humanoidRootPart.Position + vetorAim)
			
			humanoidRootPart.CFrame = humanoidRootPart.CFrame:Lerp(cframeMira, SMOOTHING)
		else
			removerAuras()
			alvoAtual = nil
		end
	end
end)

-- Respawn
player.CharacterAdded:Connect(function(novoChar)
	character = novoChar
	humanoidRootPart = character:WaitForChild("HumanoidRootPart")
	AIMANDO = false
	btn.BackgroundColor3 = Color3.new(0, 0, 0)
	removerAuras()
	alvoAtual = nil
	print("[IWUANK] Respawnado!")
end)

print("[IWUANK] ✓ TUDO PRONTO!")
