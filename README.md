local cloneref = cloneref ou função(o) retorna o fim

função local svc(nome)
local ok, s = pcall(function() return game:GetService(name) end)
Se não estiver ok ou não for s, retorne nil.
local ok2, r = pcall(cloneref, s)
se ok2 e r então retorne r fim
retornar s
fim

local Players = svc("Players")
local ReplicatedStorage = svc("ReplicatedStorage")
local RunService = svc("RunService")
local UserInputService = svc("UserInputService")
local HttpService = svc("HttpService")
local TweenService = svc("TweenService")
local CoreGui = svc("CoreGui")

jogador local = Players.LocalPlayer
se não for jogador então
Jogadores:GetPropertyChangedSignal("LocalPlayer"):Wait()
jogador = Players.LocalPlayer
fim
local playerGui = player:WaitForChild("PlayerGui")

local env = (typeof(getgenv) == "function" and getgenv()) or _G

se env.ReviveStop então pcall(env.ReviveStop) fim
se env.ExeStop então pcall(env.ExeStop) fim
se env.Skyr0Stop então pcall(env.Skyr0Stop) fim
se env.StopAura então pcall(env.StopAura) fim

local getupvalues ​​= (debug e debug.getupvalues) ou getupvalues
local getconns = getconnections ou (debug e debug.getconnections)
local setupv = (debug e debug.setupvalue) ou setupvalue

-- CONFIG
local CONFIG_FILE = "revive_vs_sniper.json"
cfg local = {
atirador = verdadeiro,
autoSubmit = true,
submitAfter = 3,
retypeInvalid = falso,
}

pcall(função()
se type(isfile) == "function" e type(readfile) == "function" e isfile(CONFIG_FILE) então
local decodificado = HttpService:JSONDecode(readfile(CONFIG_FILE))
se type(decoded) == "table" então
para chave, valor em pares(cfg) faça
local got = decodificado[chave]
se type(got) == type(value) então cfg[key] = got fim
fim
cfg.submitAfter = math.max(1, math.floor(tonumber(cfg.submitAfter) or 3))
fim
fim
fim)

função local salvarConfig()
se type(writefile) ~= "function" então retorne fim
pcall(function() writefile(CONFIG_FILE, HttpService:JSONEncode(cfg)) end)
fim

-- ESTADO + DECLARAÇÕES DE FUTURO
local UI_NAME = "ReviverVsSniper"
local _visto = {}
local _capturedParts = {}
local _lastBox = nil
local _focused = nulo
local _lastWatchedBox = nil
local _lastNonBlankText = ""
local _boxTextConn, _boxAncestryConn
local _pendingText, _pendingBox, _pendingUntil, _pendingToken = nil, nil, 0, 0

GUI local
log localRich, limparLog
definir status local
local typeAndSubmitCode, appendToBox, clearCapture
local rememberPending, clearPending, handleFeedback
local bumpSolvedLabel, applyPowerVisual

-- PEQUENAS EMPRESAS DE SERVIÇOS PÚBLICOS
aparar função local
retornar (tostring(s ou ""):gsub("^%s+", ""):gsub("%s+$", ""))
fim

função local stripRich(s)
Se o(s) tipo(s) for(em) ~= "string", então retorne para a(s) string(s) fim
retornar (s:gsub("<[^>]->", ""))
fim

-- INTERAÇÃO DA INTERFACE DO JOGO
função local isOurGui(instância)
nó local = instância
para _ = 1, 12 faça
se não for um nó, então interrompa o fim
Se node.Name == UI_NAME, retorne verdadeiro.
nó = nó.Pai
fim
retornar falso
fim

função local isVisibleChain(inst)
nó local = inst
enquanto o nó faz
Se node:IsA("GuiObject") e não node.Visible então retorne falso fim
Se o nó for "ScreenGui", retorne node.Enabled.
nó = nó.Pai
fim
retornar verdadeiro
fim

função local findAllTextBoxes(pg)
caixas locais = {}
para _, gui em ipairs(pg:GetChildren()) faça
se gui:IsA("ScreenGui") e gui.Enabled e não isOurGui(gui) então
para _, d em ipairs(gui:GetDescendants()) faça
se d:IsA("TextBox") e não isOurGui(d) então
caixas[#caixas + 1] = d
fim
fim
fim
fim
caixas de devolução
fim

função local findCodeButtons(pg)
local btns = {}
para _, gui em ipairs(pg:GetChildren()) faça
se gui:IsA("ScreenGui") e gui.Enabled e não isOurGui(gui) então
para _, d em ipairs(gui:GetDescendants()) faça
if (d:IsA("TextButton") ou d:IsA("ImageButton")) e não isOurGui(d) então
local n = d.Name:lower()
local pn = (d.Parent and d.Parent.Name or ""):lower()
se (n:find("código") ou n:find("resgatar") ou pn:find("código") ou pn:find("resgatar"))
e isVisibleChain(d) então
btns[#btns + 1] = d
fim
fim
fim
fim
fim
botões de retorno
fim

função local fireConnections(sinal, ...)
se typeof(getconns) ~= "function" então retorne falso fim
local ok, conexões = pcall(getconns, sinal)
se não estiver ok ou type(connections) ~= "table" então retorne falso fim
argumentos locais = tabela.pack(...)
local disparou = falso
para _, conexão em ipairs(conexões) faça
local fireOk = pcall(function()
se connection.Enabled ~= false então
se args.n > 0 então connection:Fire(table.unpack(args, 1, args.n))
senão conexão:Disparar() fim
fim
fim)
demitido = demitido ou fogo ok
fim
revidar disparou
fim

função local clickButton(btn)
se não btn então retorne falso fim
local any = falso
qualquer = pcall(function() btn.MouseButton1Click:Fire() end) ou qualquer
qualquer = pcall(function() btn.Activated:Fire() end) ou qualquer
se typeof(firesignal) == "function" então
qualquer = pcall(firesignal, btn.MouseButton1Click) ou qualquer
qualquer = pcall(firesignal, btn.Activated) ou qualquer
fim
qualquer = fireConnections(btn.MouseButton1Click) ou qualquer
qualquer = fireConnections(btn.Activated) ou qualquer
se typeof(fireclick) == "function" então
qualquer = pcall(fireclick, btn) ou qualquer
fim
devolva qualquer
fim

função local fireBoxFocusLost(box)
Se não for uma caixa, retorne falso.
local any = falso
se typeof(firesignal) == "function" então
qualquer = pcall(firesignal, box.FocusLost, true) ou qualquer
fim
se typeof(getconns) == "function" então
local ok, conexões = pcall(getconns, box.FocusLost)
se ok e type(connections) == "table" então
para _, conexão em ipairs(conexões) faça
função local
pcall(function() fn = connection.Function end)
se fn e typeof(getupvalues) == "function" e typeof(setupv) == "function" então
local upsOk, ups = pcall(getupvalues, fn)
se upsOk e type(ups) == "table" então
para índice, valor em pares(para cima) faça
se type(value) == "boolean" e value == true então
pcall(setupv, fn, index, false)
fim
fim
fim
fim
qualquer = pcall(função()
se connection.Enabled ~= false então connection:Fire(true) fim
fim) ou qualquer
fim
fim
fim
devolva qualquer
fim

função local liberarFoco(caixa)
pcall(function() box:ReleaseFocus(false) end)
pcall(função()
local vim = game:GetService("VirtualInputManager")
se vim então
vim:SendMouseButtonEvent(0, 0, 0, true, game, 0)
tarefa.esperar(0.03)
vim:SendMouseButtonEvent(0, 0, 0, false, game, 0)
fim
fim)
pcall(function() box:ReleaseFocus(false) end)
fim

função local currentCodeBox()
local pg = playerGui ou player:FindFirstChildOfClass("PlayerGui")
Se não for pg, retorne nil.
local codesGui = pg:FindFirstChild("Códigos")
se codesGui então
raiz local = codesGui:FindFirstChild("Códigos") ou codesGui
resgate local = root:FindFirstChild("CodeRedeem")
local box = resgatar e resgatar:FindFirstChild("TextBox")
Se a caixa e a caixa:ÉUma("TextBox") então retorne a caixa.
para _, d em ipairs(codesGui:GetDescendants()) faça
se d:IsA("TextBox") então retorne d fim
fim
fim
para _, caixa em ipairs(findAllTextBoxes(pg)) faça
Se isVisibleChain(box) então retorne box fim
fim
retornar nulo
fim

função local findSubmitButton(box)
nomes locais = { "enviar","resgatar","reivindicar","confirmar","inserir","enviar","aplicar","ok","usar","ir","verificar" }
nó local = caixa.Pai
para _ = 1, 8 faça
se não for um nó, então interrompa o fim
para _, d em ipairs(node:GetDescendants()) faça
if (d:IsA("TextButton") ou d:IsA("ImageButton")) e não isOurGui(d) ed ~= box então
local n = d.Name:lower()
texto local = ""
pcall(function() txt = tostring(d.Text):lower() end)
para _, chave em ipairs(nomes) faça
se (n:find(key) ou txt:find(key)) e isVisibleChain(d) então
retornar d
fim
fim
fim
fim
nó = nó.Pai
fim
retornar nulo
fim

typeAndSubmitCode = function(code, captureFocus)
código = tostring(código ou "")
Se o código for igual a "" então retorne falso, "código vazio" fim

local pg = playerGui ou player:FindFirstChildOfClass("PlayerGui")
Se não for pg, retorne falso, "sem PlayerGui".

local codesGui = pg:FindFirstChild("Códigos")
se codesGui então
pcall(função()
se codesGui:IsA("ScreenGui") então codesGui.Enabled = true fim
nó local = codesGui:FindFirstChild("Códigos") ou codesGui
enquanto nó e nó ~= códigosGui faça
se node:IsA("GuiObject") então node.Visible = true fim
nó = nó.Pai
fim
fim)
outro
para _, btn em ipairs(findCodeButtons(pg)) faça
clickButton(btn)
tarefa.esperar(0.05)
fim
tarefa.esperar(0.2)
fim

caixa local
prazo local = tick() + 2
repita
caixa = caixaCódigoAtual()
se for uma caixa, então interrompa o fim.
tarefa.esperar(0.1)
até que tick() > prazo
Se não houver caixa, retorne falso, "caixa de código não encontrada".

se captureFocus então
pcall(function() box:CaptureFocus() end)
tarefa.esperar(0.05)
fim

pcall(function() box.Text = "" end)
tarefa.esperar(0.03)
pcall(function() box.Text = código fim)
para _ = 1, 10 faça
Se box.Text == código então interrompa o loop
tarefa.esperar(0.04)
pcall(function() box.Text = código fim)
fim
tarefa.esperar(0.08)

local enviado = fireBoxFocusLost(caixa)
local btn = findSubmitButton(box)
se btn então
clickButton(btn)
submetido = verdadeiro
fim
pcall(função()
se typeof(keypress) == "function" então
tecla pressionada(0x0D)
tarefa.esperar(0.04)
se typeof(keyrelease) == "function" então keyrelease(0x0D) fim
submetido = verdadeiro
fim
fim)
se captureFocus então
tarefa.esperar(0.05)
liberarFoco(caixa)
fim
retornar verdadeiro, enviado e "enviado" ou "digitado"
fim

-- =====================================================================
-- GUI - Revive.vs Sniper
-- Paleta PRETO / BRANCO / CINZA
-- =====================================================================

local BRAND_MAIN = "Reviver"
local BRAND_TAIL = ".vs Sniper"
local BRAND_TAG = "Revive.vs Sniper"
local BRAND_AUTHOR = "vazado em https://discord.gg/ZvERp8sHq"

Tema local = {
MainBackground = Color3.fromRGB(10, 10, 10),
Fundo = Color3.fromRGB(18, 18, 18),
Painel = Color3.fromRGB(26, 26, 26),
Linha = Color3.fromRGB(36, 36, 36),
RowHover = Color3.fromRGB(50, 50, 50),
Accent = Color3.fromRGB(235, 235, 235),
AccentLight = Color3.fromRGB(160, 160, 160),
Verde = Color3.fromRGB(235, 235, 235),
Vermelho = Color3.fromRGB(210, 70, 70),
Vermelho2 = Color3.fromRGB(170, 50, 50),
Texto = Color3.fromRGB(245, 245, 245),
Dim = Cor3.fromRGB(140, 140, 140),
Traço = Color3.fromRGB(55, 55, 55),
SoftButton = Color3.fromRGB(36, 36, 36),
SoftButtonHover = Color3.fromRGB(50, 50, 50),
SoftAccent = Color3.fromRGB(26, 26, 26),
AlternarDesligar = Color3.fromRGB(60, 60, 60),
ToggleOff2 = Color3.fromRGB(30, 30, 30),
InputBg = Color3.fromRGB(12, 12, 12),
SliderBg = Color3.fromRGB(45, 45, 45),
TextoNoAccentro = Color3.fromRGB(15, 15, 15),
}

local T = {
bg = Tema.MainBackground,
bgDeep = Tema.InputBg,
painel = Tema.Painel,
painel2 = Tema.Linha,
linha = Tema.Traço,
acento = Tema.Acento,
accent2 = Tema.AccentLight,
accentHi = Tema.Accent,
texto = Tema.Texto,
sub = Tema.Dim,
ok = Tema.Verde,
err = Tema.Vermelho,
aviso = Color3.fromRGB(220, 190, 90),
}

local LOG = {
dim = "rgb(140,140,140)",
branco = "rgb(245,245,245)",
ok = "rgb(235,235,235)",
err = "rgb(210,70,70)",
aviso = "rgb(220,190,90)",
acc = "rgb(220,220,220)",
ciano = "rgb(170,170,170)",
}

função local nova(classe, propriedades, pai)
local inst = Instance.new(class)
para chave, valor em pares(props ou {}) faça
inst[chave] = valor
fim
se pai então inst.Parent = pai fim
retornar inst
fim

local IS_TOUCH = falso
pcall(função()
IS_TOUCH = UserInputService.TouchEnabled e não UserInputService.MouseEnabled
fim)

local MIN_TAP = IS_TOUCH e 40 ou 0
função local tapH(h) retorna math.max(h, MIN_TAP) fim

função local viewportSize()
câmera local = espaço de trabalho.CâmeraAtual
retornar (cam e cam.ViewportSize) ou Vector2.new(1280, 720)
fim

função local canto(o, r)
local c = Instance.new("UICorner"); c.CornerRadius = UDim.new(0, r); c.Parent = o; return c
fim

função local stroke(o, col, th, tr)
local s = Instance.new("UIStroke")
s.Color = col ou Theme.Stroke; s.Thickness = th ou 1; s.Transparency = tr ou 0
s.ApplyStrokeMode = Enum.ApplyStrokeMode.Border; s.Parent = o
retornar s
fim

função local tw(o, p, t)
TweenService:Create(o, TweenInfo.new(t ou 0.14, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), p):Play()
fim

função local addOutline(f)
local o = Instance.new("UIStroke")
o.Color = Theme.AccentLight; o.Thickness = 1.25; o.Transparency = 0.15
o.ApplyStrokeMode = Enum.ApplyStrokeMode.Border; o.Parent = f
retornar o
fim

função local decoratePanel(f)
local g = Instance.new("UIGradient", f); g.Rotation = 90
g.Color = ColorSequence.new{
ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 255, 255)),
ColorSequenceKeypoint.new(1, Color3.fromRGB(170, 170, 170)),
}
local hl = Instance.new("Frame")
hl.Size = UDim2.new(1, -16, 0, 1); hl.Position = UDim2.new(0, 8, 0, 1)
hl.BackgroundColor3 = Color3.fromRGB(220, 220, 220); hl.BackgroundTransparency = 0.55
hl.BorderSizePixel = 0; hl.ZIndex = 6; hl.Parent = f
local hlg = Instance.new("UIGradient", hl)
hlg.Transparência = NumberSequence.new{
NumberSequenceKeypoint.new(0, 1),
NumberSequenceKeypoint.new(0.5, 0),
NumberSequenceKeypoint.new(1, 1),
}
retornar g
fim

função local label(pai, texto, tamanho, cor, fonte, alinhamento)
retornar novo("TextLabel", {
BackgroundTransparency = 1,
Texto = texto,
Tamanho do texto = tamanho ou 11,
TextColor3 = cor ou Tema.Texto,
Fonte = fonte ou Enum.Font.GothamMedium,
TextXAligment = align ou Enum.TextXAligment.Left,
TextYAligment = Enum.TextYAligment.Center,
}, pai)
fim

função local bindClick(botão, callback)
local para baixo = falso
base local = botão.CorDeFundo3
botão.InputBegan:Conectar(função(entrada)
se input.UserInputType == Enum.UserInputType.MouseButton1
ou input.UserInputType == Enum.UserInputType.Touch então
para baixo = verdadeiro
tw(button, { BackgroundTransparency = math.min((button.BackgroundTransparency or 0) + 0.15, 1) }, 0.08)
fim
fim)
botão.EntradaEncerrada:Conectar(função(entrada)
se input.UserInputType == Enum.UserInputType.MouseButton1
ou input.UserInputType == Enum.UserInputType.Touch então
incêndio local = desativado
para baixo = falso
tw(button, { BackgroundColor3 = base }, 0.10)
se disparar então tarefa.spawn(callback) fim
fim
fim)
fim

pcall(função()
para _, nome em ipairs({
UI_NAME, "ExeCodeSniper", "Skyr0WtfUI", "ACECodeSniperUI",
"AutoTypeCodesUI", "ACEPaste", "GuiznxRiddle"
}) fazer
local antigo = playerGui:FindFirstChild(nome)
se antigo então antigo:Destruir() fim
local oldCore = CoreGui e CoreGui:FindFirstChild(name)
se oldCore então oldCore:Destroy() fim
fim
fim)

GUI = novo("ScreenGui", {
Nome = UI_NAME,
ResetOnSpawn = falso,
IgnorarGuiInset = verdadeiro,
Ordem de exibição = 9999999,
ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
}, playerGui)

local Master = novo("Frame", {
Nome = "Reviver_MasterFrame",
BackgroundTransparency = 1,
TamanhoBordaPixel = 0,
Tamanho = UDim2.new(1, 0, 1, 0),
}, GUI)

local GlobalScale = new("UIScale", { Name = "Revive_GlobalScale", Scale = 1 }, Master)

viewport localConn
função local recalcularEscala()
câmera local = espaço de trabalho.CâmeraAtual
Se não for câmera, retorne ao fim.
local h = camera.ViewportSize.Y
escala local
se UserInputService.TouchEnabled então
escala = math.clamp(h / 1000, 0.55, 0.82)
outro
escala = math.clamp(h / 800, 0.72, 1.05)
fim
Escala Global.Escala = escala
Master.Size = UDim2.new(1 / scale, 0, 1 / scale, 0)
fim

função local setupCameraListener()
se viewportConn então pcall(function() viewportConn:Disconnect() fim) fim
câmera local = espaço de trabalho.CâmeraAtual
se a câmera então
viewportConn = camera:GetPropertyChangedSignal("ViewportSize"):Connect(recalculateScale)
fim
recalcularEscala()
fim
workspace:GetPropertyChangedSignal("CurrentCamera"):Connect(setupCameraListener)
tarefa.spawn(setupCameraListener)

local WIN_W, WIN_H = 380, 660
se IS_TOUCH então
local vp = viewportSize()
WIN_W = math.floor(math.clamp(vp.X * 0.92, 320, 440))
WIN_H = math.floor(math.clamp(vp.Y * 0.88, 460, 720))
fim

local Window = new("Frame", {
Nome = "CodeRedeemer",
Tamanho = UDim2.fromOffset(WIN_W, WIN_H),
Posição = UDim2.new(1, -400, 0, 60),
BackgroundColor3 = Tema.Fundo,
BackgroundTransparency = 0.25,
TamanhoBordaPixel = 0,
Ativo = verdadeiro,
ClipsDescendants = verdadeiro,
}, Mestre)
canto(Janela, 14)
local WindowOutline = addOutline(Window)
decorarPainel(Janela)

local Header = new("Frame", {
Nome = "Cabeçalho",
Tamanho = UDim2.new(1, 0, 0, 54),
BackgroundTransparency = 1,
Ativo = verdadeiro,
}, Janela)

local Brand = label(Header, BRAND_MAIN .. BRAND_TAIL, 15, Theme.Text, Enum.Font.GothamBlack, Enum.TextXAlinment.Center)
Marca.RichText = verdadeiro
Brand.Text = BRAND_MAIN .. '<font color="rgb(160,160,160)">' .. BRAND_TAIL .. "</font>"
Marca.Tamanho = UDim2.new(1, -70, 0, 20)
Marca.Posição = UDim2.new(0, 14, 0, 8)

local SubTitle = label(Header, "Sniper - ouvindo", 11, Theme.Dim, Enum.Font.GothamMedium, Enum.TextXAlignment.Center)
SubTitle.Size = UDim2.new(1, -70, 0, 15)
SubTitle.Position = UDim2.new(0, 14, 0, 27)

local LiveDot = novo("Frame", {
Tamanho = UDim2.fromOffset(7, 7),
Posição = UDim2.fromOffset(14, 17),
BackgroundColor3 = Tema.AlternarDesativado,
TamanhoBordaPixel = 0,
}, Cabeçalho)
canto(LiveDot, 4)

função local headerButton(texto, xOffset, xOffsetTouch)
local off = (IS_TOUCH e xOffsetTouch) ou xOffset
tamanho local = IS_TOUCH e 28 ou 20
local b = novo("TextButton", {
Tamanho = UDim2.fromOffset(tamanho, tamanho),
Posição = UDim2.new(1, off, 0, IS_TOUCH e 13 ou 8),
BackgroundColor3 = Tema.Linha,
BackgroundTransparency = 0.3,
TamanhoBordaPixel = 0,
AutoButtonColor = false,
Ativo = verdadeiro,
Texto = texto,
Tamanho do texto = 13,
TextColor3 = Tema.Texto,
Fonte = Enum.Font.GothamBold,
}, Cabeçalho)
canto(b, 6)
traço(b, Tema.AccentLight, 1, 0.35)
b.MouseEnter:Connect(function() tw(b, { BackgroundColor3 = Theme.RowHover }, 0.12) end)
b.MouseLeave:Connect(function() tw(b, { BackgroundColor3 = Theme.Row }, 0.12) end)
retornar b
fim

local MinBtn = headerButton("-", -54, -70)
local CloseBtn = headerButton("x", -24, -34)

-- pílula de energia principal (único controle liga/desliga restante)
local Power = novo("TextButton", {
Nome = "Poder",
Tamanho = UDim2.fromOffset(44, 24),
Posição = UDim2.new(1, -54, 0, 28),
BackgroundColor3 = cfg.sniper e Theme.Green ou Theme.ToggleOff,
TamanhoBordaPixel = 0,
AutoButtonColor = false,
Ativo = verdadeiro,
Texto = "",
}, Cabeçalho)
canto (Power, 22)

local PowerDot = novo("Frame", {
Tamanho = UDim2.fromOffset(18, 18),
Posição = cfg.sniper e UDim2.new(1, -21, 0.5, -9) ou UDim2.new(0, 3, 0.5, -9),
BackgroundColor3 = Tema.TextOnAccent,
TamanhoBordaPixel = 0,
}, Poder)
canto(PowerDot, 22)

local HeaderDivider = novo("Frame", {
Tamanho = UDim2.new(1, -28, 0, 1),
Posição = UDim2.new(0, 14, 0, 52),
BackgroundColor3 = Tema.AccentLight,
BackgroundTransparency = 0.15,
TamanhoBordaPixel = 0,
}, Janela)
novo("UIGradient", {
Transparência = NumberSequence.new{
NumberSequenceKeypoint.new(0, 1),
NumberSequenceKeypoint.new(0.5, 0),
NumberSequenceKeypoint.new(1, 1),
},
}, HeaderDivider)

-- corpo
local Body = novo("Frame", {
Nome = "Corpo",
Tamanho = UDim2.new(1, -16, 1, -112),
Posição = UDim2.fromOffset(8, 60),
BackgroundTransparency = 1,
}, Janela)
novo("UIListLayout", {
FillDirection = Enum.FillDirection.Vertical,
OrdemDeClassificação = Enum.OrdemDeClassificação.OrdemDeLayout,
Preenchimento = UDim.new(0, 7),
}, Corpo)

-- linha de alternância de pílulas
função local makeToggleRow(order, text, initial, onChange)
linha local = novo("Quadro", {
Tamanho = UDim2.new(1, -4, 0, tapH(38)),
BackgroundColor3 = Tema.Painel,
BackgroundTransparency = 0.3,
TamanhoBordaPixel = 0,
LayoutOrder = ordem,
}, Corpo)
canto(linha, 8)

local l = label(linha, texto, 12, Tema.Texto, Enum.Fonte.GothamMedium)
l.Size = UDim2.new(1, -64, 1, 0)
l.Position = UDim2.new(0, 12, 0, 0)
l.TextTruncate = Enum.TextTruncate.AtEnd

local toggle = new("TextButton", {
Tamanho = UDim2.fromOffset(44, 24),
Posição = UDim2.new(1, -52, 0.5, -12),
BackgroundColor3 = inicial e Theme.Green ou Theme.ToggleOff,
TamanhoBordaPixel = 0,
AutoButtonColor = false,
Ativo = verdadeiro,
Texto = "",
}, linha)
canto(alternar, 22)

local dot = new("Frame", {
Tamanho = UDim2.fromOffset(18, 18),
Posição = inicial e UDim2.new(1, -21, 0.5, -9) ou UDim2.new(0, 3, 0.5, -9),
BackgroundColor3 = Tema.TextOnAccent,
TamanhoBordaPixel = 0,
}, alternar)
canto(ponto, 22)

estado local = inicial
bindClick(toggle, function()
estado = não estado
tw(toggle, { BackgroundColor3 = state and Theme.Green or Theme.ToggleOff }, 0.12)
tw(dot, { Position = state and UDim2.new(1, -21, 0.5, -9) or UDim2.new(0, 3, 0.5, -9) }, 0.12)
onChange(estado)
fim)
retornar alternar
fim

makeToggleRow(1, "Auto Submit", cfg.autoSubmit, function(state)
cfg.autoSubmit = estado
salvarConfig()
logRich('<font color="' .. LOG.dim .. '">auto submit </font><font color="' .. (state and LOG.ok or LOG.err) .. '">' .. (state and "on" or "off") .. "</font>")
fim)

makeToggleRow(2, "Redigitar inválido", cfg.retypeInvalid, function(state)
cfg.retypeInvalid = estado
salvarConfig()
logRich('<font color="' .. LOG.dim .. '">redigitação inválida </font><font color="' .. (estado e LOG.ok ou LOG.err) .. '">' .. (estado e "ligado" ou "desligado") .. "</font>")
fim)

-- controle deslizante
fazer
local MIN, MAX = 1, 10
local holder = novo("Frame", {
Tamanho = UDim2.new(1, -4, 0, 54),
BackgroundTransparency = 1,
LayoutOrder = 3,
}, Corpo)

local sliderLabel = label(holder, "Enviar após: " .. cfg.submitAfter .. " parts", 12, Theme.Text, Enum.Font.GothamMedium)
sliderLabel.Size = UDim2.new(1, 0, 0, 18)
sliderLabel.Position = UDim2.new(0, 4, 0, 0)

barra local = novo("Frame", {
Tamanho = UDim2.new(1, -12, 0, 8),
Posição = UDim2.fromOffset(6, 30),
BackgroundColor3 = Tema.SliderBg,
TamanhoBordaPixel = 0,
}, detentor)
canto(barra, 10)

proporção local = (cfg.submitAfter - MIN) / (MAX - MIN)
preenchimento local = novo("Quadro", {
Tamanho = UDim2.new(math.clamp(ratio, 0, 1), 0, 1, 0),
BackgroundColor3 = Tema.Accent,
TamanhoBordaPixel = 0,
}, bar)
canto(preenchimento, 10)

local knob = novo("Frame", {
Nome = "WhiteSliderKnob",
Tamanho = UDim2.fromOffset(16, 16),
PontoAncora = Vector2.new(0.5, 0.5),
Posição = UDim2.new(math.clamp(ratio, 0, 1), 0, 0.5, 0),
BackgroundColor3 = Tema.Texto,
TamanhoBordaPixel = 0,
}, bar)
canto(puxador, 22)

arrastar local = falso
atualização de função local(x)
local rel = math.clamp((x - bar.AbsolutePosition.X) / bar.AbsoluteSize.X, 0, 1)
valor local = math.floor(MIN + (MAX - MIN) * rel + 0.5)
rel = (valor - MÍN) / (MÁX - MÍN)
fill.Size = UDim2.new(rel, 0, 1, 0)
knob.Position = UDim2.new(rel, 0, 0.5, 0)
sliderLabel.Text = "Enviar após: " .. valor .. (valor == 1 e " parte" ou " partes")
se o valor for ~= cfg.submitAfter então
cfg.submitAfter = valor
limparCaptura()
salvarConfig()
fim
fim

bar.InputBegan:Connect(function(input)
se input.UserInputType == Enum.UserInputType.MouseButton1
ou input.UserInputType == Enum.UserInputType.Touch então
arrastar = verdadeiro
atualizar(entrada.Posição.X)
fim
fim)
UserInputService.InputEnded:Connect(function(input)
se input.UserInputType == Enum.UserInputType.MouseButton1
ou input.UserInputType == Enum.UserInputType.Touch então
arrastar = falso
fim
fim)
UserInputService.InputChanged:Connect(function(input)
se arrastar e (input.UserInputType == Enum.UserInputType.MouseMovement
ou input.UserInputType == Enum.UserInputType.Touch) então
atualizar(entrada.Posição.X)
fim
fim)
fim

-- barra de console
local ConsoleBar = new("Frame", {
Tamanho = UDim2.new(1, -4, 0, 18),
BackgroundTransparency = 1,
LayoutOrder = 4,
}, Corpo)

local ConsoleTitle = label(ConsoleBar, "CONSOLE", 10, Theme.Dim, Enum.Font.GothamBlack)
ConsoleTitle.Size = UDim2.fromOffset(80, 18)
ConsoleTitle.Position = UDim2.fromOffset(4, 0)

local ClearBtn = new("TextButton", {
Tamanho = UDim2.fromOffset(56, 18),
Posição = UDim2.new(1, -56, 0, 0),
BackgroundColor3 = Tema.Linha,
BackgroundTransparency = 0.3,
TamanhoBordaPixel = 0,
AutoButtonColor = false,
Ativo = verdadeiro,
Texto = "LIMPO",
Tamanho do texto = 10,
TextColor3 = Tema.Texto,
Fonte = Enum.Font.GothamBold,
}, ConsoleBar)
canto(ClearBtn, 6)
stroke(ClearBtn, Theme.AccentLight, 1, 0.35)
ClearBtn.MouseEnter:Connect(function() tw(ClearBtn, { BackgroundColor3 = Theme.RowHover }, 0.12) end)
ClearBtn.MouseLeave:Connect(function() tw(ClearBtn, { BackgroundColor3 = Theme.Row }, 0.12) end)

-- o console preenche o espaço corporal restante
-- deslocamento de altura = Linhas(1,2) + slider + ConsoleBar + 4 espaços de 7px
local CONSOLE_OFFSET = tapH(38) + tapH(38) + 54 + 18 + 28

local Console = new("ScrollingFrame", {
Nome = "Console",
Tamanho = UDim2.new(1, -4, 1, -CONSOLE_OFFSET),
BackgroundColor3 = Tema.InputBg,
BackgroundTransparency = 0.1,
TamanhoBordaPixel = 0,
Ativo = verdadeiro,
ClipsDescendants = verdadeiro,
DireçãoDeRolagem = Enum.ScrollingDirection.Y,
Espessura da barra de rolagem = 4,
ScrollBarImageColor3 = Tema.AccentLight,
CanvasSize = UDim2.new(0, 0, 0, 0),
AutomaticCanvasSize = Enum.AutomaticSize.None,
ElasticBehavior = Enum.ElasticBehavior.WhenScrollable,
LayoutOrder = 5,
}, Corpo)
canto(Console, 8)
traço(Console, Tema.AccentLight, 1, 0.35)

local ConsoleText = new("TextLabel", {
Nome = "Saída",
Tamanho = UDim2.new(1, -18, 0, 0),
AutomaticSize = Enum.AutomaticSize.Y,
Posição = UDim2.fromOffset(9, 7),
BackgroundTransparency = 1,
RichText = verdadeiro,
Texto = "",
Tamanho do texto = 12,
Fonte = Enum.Font.Code,
TextColor3 = Tema.Texto,
TextXAlinhamento = Enum.TextXAlinhamento.Esquerda,
TextYAlignment = Enum.TextYAlignment.Top,
TextoEnvolvido = verdadeiro,
}, Console)

-- barra inferior
local BottomBar = novo("Frame", {
Nome = "Barra Inferior",
Tamanho = UDim2.new(1, -16, 0, 44),
Posição = UDim2.new(0, 8, 1, -52),
BackgroundColor3 = Tema.Fundo,
BackgroundTransparency = 0.3,
TamanhoBordaPixel = 0,
}, Janela)
canto(BarraInferior, 12)
traço(BarraInferior, Tema.LuzDeDestaque, 1, 0.35)

local LogoTile = novo("Frame", {
Tamanho = UDim2.fromOffset(30, 30),
Posição = UDim2.new(0, 9, 0.5, -15),
BackgroundColor3 = Tema.SoftAccent,
TamanhoBordaPixel = 0,
}, BarraInferior)
canto(LogoTile, 8)
traço(LogoTile, Tema.AccentLight, 1, 0.45)

local WordMark = label(BottomBar, BRAND_MAIN, 17, Theme.Text, Enum.Font.GothamBlack)
WordMark.Size = UDim2.fromOffset(70, 22)
WordMark.Position = UDim2.fromOffset(46, 5)

local BarDivider = label(BottomBar, "|", 15, Theme.Dim, Enum.Font.GothamBlack)
BarDivider.Size = UDim2.fromOffset(12, 22)
BarDivider.Position = UDim2.fromOffset(110, 5)

local FullMark = label(BottomBar, "vs Sniper", 14, Theme.AccentLight, Enum.Font.GothamBold)
FullMark.Size = UDim2.fromOffset(140, 22)
FullMark.Position = UDim2.fromOffset(124, 5)

local Author = label(BottomBar, BRAND_AUTHOR, 9, Theme.Dim, Enum.Font.GothamSemibold)
Author.Size = UDim2.fromOffset(230, 12)
Author.Position = UDim2.fromOffset(47, 26)
Autor.TextTruncate = Enum.TextTruncate.AtEnd

-- escritores de console
local logLines = {}
local MAX_LOG = 200

função local scrollToBottom()
tarefa.adiar(função()
tarefa.esperar()
Se não for Console ou não for ConsoleText, retorne.
altura local = ConsoleText.AbsoluteSize.Y + 14
Console.CanvasSize = UDim2.new(0, 0, 0, altura)
Console.CanvasPosition = Vector2.new(0, math.max(0, height - Console.AbsoluteSize.Y))
fim)
fim

ConsoleText:GetPropertyChangedSignal("AbsoluteSize"):Connect(function()
Console.CanvasSize = UDim2.new(0, 0, 0, ConsoleText.AbsoluteSize.Y + 14)
fim)

logRich = função(linha)
logLines[#logLines + 1] = linha
Se #logLines > MAX_LOG então table.remove(logLines, 1) fim
ConsoleText.Text = table.concat(logLines, "\n")
scrollToBottom()
fim

clearLog = função()
logLines = {}
ConsoleText.Text = '<font color="' .. LOG.dim .. '">limpo</font>'
Console.CanvasPosition = Vector2.new(0, 0)
fim

setStatus = função(msg, cor)
logRich('<font color="' .. (color or LOG.dim) .. '">' .. tostring(msg) .. "</font>")
fim

bumpSolvedLabel = função() fim

bindClick(ClearBtn, function()
limparLog()
fim)

-- Visualização de poder (setPowerRow foi removido; o único controle é o cabeçalho)
aplicarPowerVisual = função()
local on = cfg.sniper
tw(Power, { BackgroundColor3 = on and Theme.Green or Theme.ToggleOff }, 0.12)
tw(PowerDot, { Position = on and UDim2.new(1, -21, 0.5, -9) or UDim2.new(0, 3, 0.5, -9) }, 0.12)
tw(LiveDot, { BackgroundColor3 = on and Theme.Green or Theme.ToggleOff }, 0.12)
Subtítulo.Texto = ligado e "Atirador - ouvindo" ou "Atirador - em pausa"
SubTitle.TextColor3 = on e Theme.Text ou Theme.Dim
tw(WindowOutline, { Transparência = ativada e 0,15 ou 0,55 }, 0,12)
fim

bindClick(Power, function()
cfg.sniper = não cfg.sniper
salvarConfig()
aplicarPowerVisual()
se cfg.sniper então
limparCaptura()
logRich('<font color="' .. LOG.ok .. '">sniper ligado</font>')
outro
logRich('<font color="' .. LOG.err .. '">sniper off</font>')
fim
fim)

minimizado local = falso
bindClick(MinBtn, function()
minimizado = não minimizado
Corpo.Visível = não minimizado
BottomBar.Visible = não minimizado
tw(Janela, { Tamanho = UDim2.fromOffset(WIN_W, minimizado e 56 ou WIN_H) }, 0.18)
MinBtn.Text = minimizado e "+" ou "-"
fim)

bindClick(CloseBtn, function()
se env.ReviveStop então env.ReviveStop() senão GUI:Destroy() fim
fim)

-- LANÇADOR MÓVEL
se IS_TOUCH então
local fab = novo("TextButton", {
Nome = "Reviver_Launcher",
Tamanho = UDim2.fromOffset(58, 58),
Posição = UDim2.new(0, 14, 0.5, -29),
BackgroundColor3 = Tema.Accent,
TamanhoBordaPixel = 0,
AutoButtonColor = false,
Ativo = verdadeiro,
Texto = "R",
Tamanho do texto = 18,
TextColor3 = Tema.TextOnAccent,
Fonte = Enum.Font.GothamBlack,
ZIndex = 50,
}, Mestre)
canto(fab, 29); traço(fab, Tema.Texto, 1.5, 0.5)

arrastar local, movido, iniciarEm, posiçãoInicial = falso, falso, nulo, nulo
fab.InputBegan:Connect(function(i)
Se i.UserInputType == Enum.UserInputType.Touch ou i.UserInputType == Enum.UserInputType.MouseButton1 então
arrastando, movido = verdadeiro, falso
startIn, startPos = i.Position, fab.Position
tw(fab, { Size = UDim2.fromOffset(52, 52) }, 0.08)
fim
fim)
fab.InputEnded:Connect(function(i)
Se i.UserInputType == Enum.UserInputType.Touch ou i.UserInputType == Enum.UserInputType.MouseButton1 então
arrastar = falso
tw(fab, { Size = UDim2.fromOffset(58, 58) }, 0.10)
Se não for movido, então Window.Visible = not Window.Visible.
fim
fim)
UserInputService.InputChanged:Connect(function(i)
Se não estiver arrastando, retorne ao fim.
se i.UserInputType ~= Enum.UserInputType.Touch
e i.UserInputType ~= Enum.UserInputType.MouseMovement então retorne fim
local d = i.Position - startIn
se math.abs(dX) > 6 ou math.abs(dY) > 6 então movido = verdadeiro fim
local vp = viewportSize()
fab.Position = UDim2.new(
0, math.clamp(startPos.X.Offset + dX, 0, vp.X - 58),
0, math.clamp(startPos.Y.Offset + dY, 0, vp.Y - 58))
fim)
fim

-- arrastar (com reconhecimento de escala)
fazer
arrastar local, dragInput, dragStart, startPos = false, nil, nil, nil
local THRESHOLD = UserInputService.TouchEnabled e 8 ou 2
local movido = falso

função local overControl(posição)
para _, controle em ipairs({ Power, MinBtn, CloseBtn }) faça
local pos, size = control.AbsolutePosition, control.AbsoluteSize
se position.X >= pos.X - 8 e position.X <= pos.X + size.X + 8
e posição.Y >= pos.Y - 8 e posição.Y <= pos.Y + tamanho.Y + 8 então
retornar verdadeiro
fim
fim
retornar falso
fim

Cabeçalho.InputBegan:Conectar(função(entrada)
se input.UserInputType ~= Enum.UserInputType.MouseButton1
e input.UserInputType ~= Enum.UserInputType.Touch então retorne fim
Se arrastar ou sobreControlar(input.Position), retorne o fim.
arrastar, dragInput = verdadeiro, entrada
dragStart = Vector2.new(input.Position.X, input.Position.Y)
startPos = Window.Position
movido = falso
input.Changed:Connect(function()
se input.UserInputState == Enum.UserInputState.End
ou input.UserInputState == Enum.UserInputState.Cancel então
Se input == dragInput então arrastando, dragInput = false, nil fim
fim
fim)
fim)

UserInputService.InputChanged:Connect(function(input)
Se não estiver arrastando ou não houver entrada de arrastar, retorne ao fim.
local trackedTouch = dragInput.UserInputType == Enum.UserInputType.Touch e input == dragInput
local trackedMouse = dragInput.UserInputType == Enum.UserInputType.MouseButton1
e input.UserInputType == Enum.UserInputType.MouseMovement
Se não houver rastreamento de toque nem de mouse, retorne ao fim.
local delta = Vector2.new(input.Position.X, input.Position.Y) - dragStart
se não for movido então
Se delta.Magnitude < LIMITE então retorne fim
movido = verdadeiro
fim
Escala local = Escala Global.Escala
se escala <= 0 então escala = 1 fim
Window.Position = UDim2.new(
startPos.X.Scale, startPos.X.Offset + (delta.X / escala),
startPos.Y.Scale, startPos.Y.Offset + (delta.Y / escala))
fim)
fim

UserInputService.InputBegan:Connect(function(input, processed)
Se processado, retorne ao fim.
Se input.KeyCode == Enum.KeyCode.RightControl então
GUI.Enabled = não GUI.Enabled
fim
fim)

fazer
alvo local = Window.Position
local openScale = new("UIScale", { Name = "ReviveScale", Scale = 0.92 }, Window)
Window.Position = UDim2.new(target.X.Scale, target.X.Offset, target.Y.Scale, target.Y.Offset + 18)
tw(openScale, { Scale = 1 }, 0.20)
tw(Janela, { Posição = alvo }, 0.20)
fim

aplicarPowerVisual()

logRich('<font color="' .. LOG.acc .. '">Revive.vs Sniper carregado</font>')
logRich('<font color="' .. LOG.dim .. '">códigos digitados automaticamente quando anunciados</font>')

-- FLUXO DE CAPTURA E ENVIO DE CÓDIGO
clearCapture = função()
_capturedParts = {}
fim

função local clearBoxWatchers()
se _boxTextConn então pcall(function() _boxTextConn:Disconnect() fim) fim
se _boxAncestryConn então pcall(function() _boxAncestryConn:Disconnect() fim) fim
_boxTextConn, _boxAncestryConn, _lastWatchedBox = zero, zero, zero
fim

função local watchBox(box)
Se não for box ou _lastWatchedBox == box, então retorne.
clearBoxWatchers()
_lastWatchedBox = caixa
Se box.Text ~= "" então _lastNonBlankText = box.Text fim
_boxTextConn = box:GetPropertyChangedSignal("Text"):Connect(function()
se box.Text == "" então
limparCaptura()
outro
_lastNonBlankText = box.Text
fim
fim)
boxAncestryConn = box.AncestryChanged:Connect(function(), parent)
se não for pai então
limparCaptura()
clearBoxWatchers()
fim
fim)
fim

limparPendente = função()
_pendingToken += 1
_pendingText, _pendingBox, _pendingUntil = nil, nil, 0
fim

lembrarPendente = função(caixa, texto, substituirExistente)
Se não cfg.retypeInvalid ou não text ou text == "" então retorne fim
se não replaceExisting e _pendingText e os.clock() <= _pendingUntil então retorne fim
_pendingToken += 1
token local = _pendingToken
_pendingText, _pendingBox = texto, caixa
_pendingUntil = os.clock() + 8
tarefa.atraso(8, função()
se token == _pendingToken então clearPending() fim
fim)
fim

função local restaurarRejeitado(caixa, texto)
Se não cfg.retypeInvalid ou não text ou text == "" então retorne falso.
RunService.Heartbeat:Wait()
alvo local = currentCodeBox() ou caixa
Se não for alvo ou não for isVisibleChain(alvo), retorne falso.
local ok = pcall(function() target.Text = texto end)
Se estiver tudo bem, então
_lastBox = alvo
watchBox(alvo)
fim
retornar ok
fim

handleFeedback = função(texto, objetoFonte)
Se não cfg.retypeInvalid ou não _pendingText, retorne.
se os.clock() > _pendingUntil então clearPending(); retorne fim
Se sourceObject e sourceObject:IsDescendantOf(GUI), então retorne end.

local lower = tostring(texto ou ""):lower()
local rejected = lower:find("código inválido", 1, true)
ou inferior:find("código inválido", 1, true)
ou inferior: encontrar("expirado", 1, verdadeiro)
ou inferior: encontrar("já resgatado", 1, verdadeiro)
ou inferior: encontrar("já usado", 1, verdadeiro)
ou inferior:find("não existe", 1, true)
ou inferior:find("não existe", 1, true)
ou inferior: encontrar("não encontrado", 1, verdadeiro)
ou inferior: encontrar("rejeitado", 1, verdadeiro)
Se não for rejeitado, retorne ao fim.

local previousText, previousBox = _pendingText, _pendingBox
local restaurado = restaurarRejeitado(caixaAnterior, textoAnterior)
limparPendente()
se restaurado então
setStatus("inválido, redigitado: " .. previousText, LOG.warn)
fim
fim

appendToBox = function(text)
Se não houver texto ou se o texto for igual a "" então retorne fim
se _lastWatchedBox e não isVisibleChain(_lastWatchedBox) então
limparCaptura()
clearBoxWatchers()
fim

caixa local = caixaCódigoAtual()
_capturedParts[#_capturedParts + 1] = texto
local combinado = tabela.concat(_capturedParts)
contagem local = #_partescapturadas

se caixa então
_lastBox = caixa
watchBox(caixa)
local estavaFocado = UserInputService:GetFocusedTextBox() == caixa
pcall(function() box.Text = combinado fim)
se estava em foco então
pcall(função()
local caret = #combinado + 1
box.CursorPosition = caret
box.SelectionStart = caret
fim)
fim
fim

logRich('<font color="' .. LOG.dim .. '">código ' .. contagem .. "/" .. cfg.submitAfter .. ': </font><font color="' .. LOG.ok .. '">' .. combinado .. "</font>")

se count >= cfg.submitAfter então
_capturedParts = {}
se cfg.autoSubmit então
lembrarPendente(caixa, combinado, verdadeiro)
local ok, mensagem = typeAndSubmitCode(combined, false)
Se estiver tudo bem, então
setStatus("redestinado: " .. combinado, LOG.ok)
outro
local restaurado = restaurarRejeitado(caixa, combinado)
limparPendente()
se restaurado então
setStatus("inválido, redigitado: " .. combinado, LOG.warn)
outro
setStatus("falhou: " .. tostring(mensagem), LOG.err)
fim
fim
fim
fim
fim

-- ANÚNCIO AO OUVINTE
função local tokenize(texto)
palavras locais = {}
para cada palavra em texto:gmatch("[%w_]+") faça
palavras[#palavras + 1] = palavra
fim
palavras de retorno
fim

função local onAnnouncement(...)
texto local = aparar(retirarRich(tostring((...) ou "")))
se o texto for igual a "" então retorne fim
se texto:encontrar("%s") então retornar fim

para _, palavra em ipairs(tokenize(texto)) faça
se palavra ~= "" e não _visto[palavra] então
_visto[palavra] = verdadeiro
tarefa.delay(1.25, função() _seen[palavra] = nil fim)
adicionarÀCaixa(palavra)
fim
fim
fim

função local resolveNotifyRemote()
se _G.PhiNotifyRemote então retorne _G.PhiNotifyRemote fim

local ok, controlador = pcall(função()
Se não for ReplicatedStorage, retorne nulo.
controladores locais = ReplicatedStorage:FindFirstChild("Controladores")
notificação local = controladores e controladores:FindFirstChild("NotificationController", true)
se houver notificação, retorne require(notificação) fim
retornar nulo
fim)
se ok e type(controller) == "table" e type(controller.Start) == "function"
e typeof(getupvalues) == "function" então
valores locais Ok, valores = pcall(getupvalues, controller.Start)
se valuesOk e type(values) == "table" então
para _, valor em pares(valores) faça
se typeof(value) == "Instance"
e (valor:IsA("RemoteEvent") ou valor:IsA("UnreliableRemoteEvent")) então
valor de retorno
fim
fim
fim
fim

local getinfo = debug e (debug.getinfo ou debug.info)
pacotes locais = ReplicatedStorage e ReplicatedStorage:FindFirstChild("Packages")
rede local = pacotes e pacotes:FindFirstChild("Net")
se net e getinfo e getconns então
para _, d em ipairs(net:GetDescendants()) faça
se d:IsA("RemoteEvent") então
local okConns, conexões = pcall(getconns, d.OnClientEvent)
se okConns e type(connections) == "table" então
para _, conexão em ipairs(conexões) faça
local fnOk, fn = pcall(function() return connection.Function end)
se fnOk e type(fn) == "function" então
local infoOk, info = pcall(getinfo, fn)
se infoOk e info
e tostring(info.short_src ou info.source ou ""):find("NotificationController", 1, true) então
retornar d
fim
fim
fim
fim
fim
fim
fim
retornar nulo
fim

local listenConn
tarefa.spawn(função()
local remoto = resolverNotificarRemoto()
se remoto e (remoto:IsA("RemoteEvent") ou remoto:IsA("UnreliableRemoteEvent")) então
listenConn = remote.OnClientEvent:Connect(function(...)
se não cfg.sniper então retorne fim
pcall(onAnnouncement, ...)
fim)
logRich('<font color="' .. LOG.dim .. '">ouvinte conectado: ' .. remote.Name .. "</font>")
outro
LiveDot.BackgroundColor3 = T.err
logRich('<font color="' .. LOG.err .. '">nenhum notify remoto encontrado</font>')
fim
fim)

-- OBSERVADORES DE FEEDBACK + RASTREAMENTO DE FOCO
função local watchFeedbackObject(obj)
se não (obj:IsA("TextLabel") ou obj:IsA("TextButton")) então retorne fim
se isOurGui(obj) então retorne fim
handleFeedback(obj.Text ou "", obj)
obj:GetPropertyChangedSignal("Text"):Connect(function()
handleFeedback(obj.Text ou "", obj)
fim)
fim

para _, obj em ipairs(playerGui:GetDescendants()) faça
pcall(watchFeedbackObject, obj)
fim
playerGui.DescendantAdded:Connect(function(obj)
tarefa.esperar(0.04)
pcall(watchFeedbackObject, obj)
fim)

UserInputService.TextBoxFocused:Connect(function(box)
Se box:IsDescendantOf(GUI) então retorne fim
se box ~= currentCodeBox() então retorne fim
_focused, _lastBox = caixa, caixa
watchBox(caixa)
fim)

UserInputService.TextBoxFocusReleased:Connect(function(box)
Se box:IsDescendantOf(GUI) então retorne fim
local codeBox = currentCodeBox()
Se box ~= codeBox e box ~= _lastBox então retorne fim
se cfg.retypeInvalid então
local submitted = box.Text ~= "" and box.Text or _lastNonBlankText
lembrarPendente(caixa, enviado, falso)
fim
se _focused == box então _focused = nil fim
fim)

-- GANCHO DE LIMPEZA
função local pararTudo()
se listenConn então
pcall(function() listenConn:Disconnect() end)
listenConn = nulo
fim
se viewportConn então pcall(function() viewportConn:Disconnect() fim) fim
clearBoxWatchers()
se GUI então pcall(function() GUI:Destroy() fim) fim
fim

env.ReviverStop = pararTudo
env.ExeStop = pararTudo
env.Skyr0Stop = pararTudo
env.StopAura = pararTudo

