--[[
    ⚡ ESP MENU v5.0 - Interface Premium + Todas Funções
    ✅ Interface estilo Keyless (sidebar + cards)
    ✅ Todas as funções anteriores mantidas
    ✅ Bounding Box, Tracer, Highlight (chams) funcionais
    ✅ Auto-target, search funcional, status dot
    ✅ Save/Load, presets, notificações
    ✅ Loop único otimizado + pcall
--]]

-----------------------------------------------------------
--// SERVIÇOS
-----------------------------------------------------------
local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local HttpService = game:GetService("HttpService")
local TweenService = game:GetService("TweenService")
local LocalPlayer = Players.LocalPlayer

for _, nome in ipairs({"ESP_GUI", "LocalizadorGUI", "NotifGUI", "TracerGUI"}) do
    local a = CoreGui:FindFirstChild(nome)
    if a then a:Destroy() end
end

-----------------------------------------------------------
--// TEMA
-----------------------------------------------------------
local Tema = {
    fundoPrincipal = Color3.fromRGB(15, 15, 20),
    fundoSidebar = Color3.fromRGB(10, 10, 14),
    fundoCard = Color3.fromRGB(22, 22, 28),
    fundoCardAlt = Color3.fromRGB(28, 28, 35),
    fundoHover = Color3.fromRGB(35, 35, 45),
    fundoInput = Color3.fromRGB(18, 18, 24),
    borda = Color3.fromRGB(38, 38, 48),
    texto = Color3.fromRGB(240, 240, 245),
    textoSub = Color3.fromRGB(130, 130, 145),
    textoFraco = Color3.fromRGB(80, 80, 95),
    accent = Color3.fromRGB(130, 90, 255),
    accentAlt = Color3.fromRGB(90, 60, 200),
    verde = Color3.fromRGB(80, 220, 130),
    vermelho = Color3.fromRGB(240, 80, 90),
    amarelo = Color3.fromRGB(255, 200, 90),
}

-----------------------------------------------------------
--// CONFIGURAÇÃO
-----------------------------------------------------------
local ConfigPadrao = {
    espAtivo = false,
    mostrarNome = true,
    mostrarVida = true,
    mostrarDistancia = true,
    mostrarHPnumero = true,
    mostrarFundo = true,
    mostrarBorda = false,
    mostrarIcone = true,
    mostrarSombra = true,
    mostrarCaixa = false,
    mostrarTracer = false,
    mostrarHighlight = false,
    mostrarMortos = true,
    mostrarLocal = false,
    sempreVisivel = true,
    
    corFundo = Color3.fromRGB(0, 0, 0),
    corNome = Color3.fromRGB(255, 255, 255),
    corDistancia = Color3.fromRGB(255, 220, 100),
    corBorda = Color3.fromRGB(130, 90, 255),
    corCaixa = Color3.fromRGB(80, 220, 130),
    corTracer = Color3.fromRGB(240, 80, 90),
    corHighlight = Color3.fromRGB(255, 100, 100),
    corVidaCheia = Color3.fromRGB(80, 220, 130),
    corVidaMedia = Color3.fromRGB(255, 200, 90),
    corVidaBaixa = Color3.fromRGB(240, 80, 90),
    corVidaBG = Color3.fromRGB(50, 15, 20),
    
    transparenciaFundo = 0.4,
    tamanhoLargura = 200,
    tamanhoAltura = 70,
    offsetY = 3,
    tamanhoFonteNome = 14,
    tamanhoFonteDist = 12,
    espessuraBarra = 10,
    distanciaMaxima = 500,
    
    localizadorAtivo = false,
    corLocalizador = Color3.fromRGB(255, 0, 100),
    espessuraLinhaLoc = 2,
    atalhoAtivo = true,
}

local Config = {}
for k, v in pairs(ConfigPadrao) do Config[k] = v end

local ArquivoConfig = "esp_config_v5.json"

-----------------------------------------------------------
--// STORE
-----------------------------------------------------------
local Store = {subs = {}}
function Store:set(chave, valor)
    Config[chave] = valor
    for _, fn in ipairs(self.subs[chave] or {}) do task.spawn(fn, valor) end
    for _, fn in ipairs(self.subs["*"] or {}) do task.spawn(fn, chave, valor) end
end
function Store:get(chave) return Config[chave] end
function Store:subscribe(chave, fn)
    self.subs[chave] = self.subs[chave] or {}
    table.insert(self.subs[chave], fn)
end

-----------------------------------------------------------
--// SAVE/LOAD
-----------------------------------------------------------
local function podeSalvar()
    return typeof(writefile) == "function" and typeof(readfile) == "function"
end
local function ficheiroExiste()
    if typeof(isfile) == "function" then return isfile(ArquivoConfig) end
    return false
end
local function salvarConfig()
    if not podeSalvar() then return end
    local dados = {}
    for k, v in pairs(Config) do
        if typeof(v) == "Color3" then
            dados[k] = {__t = "Color3", r = v.R, g = v.G, b = v.B}
        elseif typeof(v) ~= "Instance" then
            dados[k] = v
        end
    end
    pcall(function() writefile(ArquivoConfig, HttpService:JSONEncode(dados)) end)
end
local function carregarConfig()
    if not podeSalvar() then return false end
    local ok, conteudo = pcall(readfile, ArquivoConfig)
    if not ok or not conteudo then return false end
    local ok2, dados = pcall(HttpService.JSONDecode, HttpService, conteudo)
    if not ok2 or type(dados) ~= "table" then return false end
    for k, v in pairs(dados) do
        if type(v) == "table" and v.__t == "Color3" then
            Config[k] = Color3.new(v.r, v.g, v.b)
        elseif k ~= "localizadorAtivo" then
            Config[k] = v
        end
    end
    return true
end
local carregouConfig = carregarConfig()

-----------------------------------------------------------
--// NOTIFICAÇÕES
-----------------------------------------------------------
local NotifGui = Instance.new("ScreenGui")
NotifGui.Name = "NotifGUI"
NotifGui.ResetOnSpawn = false
NotifGui.Parent = CoreGui

local NotifContainer = Instance.new("Frame")
NotifContainer.Size = UDim2.new(0, 320, 1, -40)
NotifContainer.Position = UDim2.new(1, -340, 0, 20)
NotifContainer.BackgroundTransparency = 1
NotifContainer.Parent = NotifGui

local NotifLayout = Instance.new("UIListLayout")
NotifLayout.Padding = UDim.new(0, 8)
NotifLayout.VerticalAlignment = Enum.VerticalAlignment.Bottom
NotifLayout.HorizontalAlignment = Enum.HorizontalAlignment.Right
NotifLayout.SortOrder = Enum.SortOrder.LayoutOrder
NotifLayout.Parent = NotifContainer

local function notificar(texto, cor)
    cor = cor or Tema.accent
    local notif = Instance.new("Frame")
    notif.Size = UDim2.new(0, 300, 0, 46)
    notif.BackgroundColor3 = Tema.fundoCard
    notif.BorderSizePixel = 0
    notif.Parent = NotifContainer
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = notif
    
    local stroke = Instance.new("UIStroke")
    stroke.Color = Tema.borda
    stroke.Thickness = 1
    stroke.Parent = notif
    
    local barra = Instance.new("Frame")
    barra.Size = UDim2.new(0, 3, 1, -12)
    barra.Position = UDim2.new(0, 8, 0, 6)
    barra.BackgroundColor3 = cor
    barra.BorderSizePixel = 0
    barra.Parent = notif
    
    local bc = Instance.new("UICorner")
    bc.CornerRadius = UDim.new(0, 2)
    bc.Parent = barra
    
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -30, 1, 0)
    label.Position = UDim2.new(0, 22, 0, 0)
    label.BackgroundTransparency = 1
    label.Text = texto
    label.TextColor3 = Tema.texto
    label.TextSize = 12
    label.Font = Enum.Font.Gotham
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.TextWrapped = true
    label.Parent = notif
    
    notif.Position = UDim2.new(1, 60, 0, 0)
    notif.BackgroundTransparency = 1
    TweenService:Create(notif, TweenInfo.new(0.25, Enum.EasingStyle.Quart), {
        Position = UDim2.new(1, -320, 0, 0),
        BackgroundTransparency = 0
    }):Play()
    
    task.delay(3, function()
        if notif and notif.Parent then
            TweenService:Create(notif, TweenInfo.new(0.25), {
                Position = UDim2.new(1, 60, 0, 0),
                BackgroundTransparency = 1
            }):Play()
            task.wait(0.3)
            if notif then notif:Destroy() end
        end
    end)
end

-----------------------------------------------------------
--// HELPERS UI
-----------------------------------------------------------
local function corner(p, r)
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, r or 8)
    c.Parent = p
    return c
end

local function stroke(p, cor, esp)
    local s = Instance.new("UIStroke")
    s.Color = cor or Tema.borda
    s.Thickness = esp or 1
    s.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    s.Parent = p
    return s
end

-----------------------------------------------------------
--// JANELA PRINCIPAL
-----------------------------------------------------------
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ESP_GUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = CoreGui
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 740, 0, 480)
MainFrame.Position = UDim2.new(0.5, -370, 0.5, -240)
MainFrame.BackgroundColor3 = Tema.fundoPrincipal
MainFrame.BorderSizePixel = 0
MainFrame.Active = false
MainFrame.Parent = ScreenGui
corner(MainFrame, 12)
stroke(MainFrame, Tema.borda, 1)

-- Header
local Header = Instance.new("Frame")
Header.Size = UDim2.new(1, 0, 0, 40)
Header.BackgroundColor3 = Tema.fundoPrincipal
Header.BorderSizePixel = 0
Header.Active = true
Header.Parent = MainFrame
corner(Header, 12)

local HeaderFix = Instance.new("Frame")
HeaderFix.Size = UDim2.new(1, 0, 0, 12)
HeaderFix.Position = UDim2.new(0, 0, 1, -12)
HeaderFix.BackgroundColor3 = Tema.fundoPrincipal
HeaderFix.BorderSizePixel = 0
HeaderFix.Parent = Header

local HeaderSep = Instance.new("Frame")
HeaderSep.Size = UDim2.new(1, -20, 0, 1)
HeaderSep.Position = UDim2.new(0, 10, 1, -1)
HeaderSep.BackgroundColor3 = Tema.borda
HeaderSep.BorderSizePixel = 0
HeaderSep.ZIndex = 2
HeaderSep.Parent = Header

local HeaderTitle = Instance.new("TextLabel")
HeaderTitle.Size = UDim2.new(0, 300, 1, 0)
HeaderTitle.Position = UDim2.new(0, 16, 0, 0)
HeaderTitle.BackgroundTransparency = 1
HeaderTitle.Text = "◈  ESP MENU"
HeaderTitle.TextColor3 = Tema.texto
HeaderTitle.TextSize = 13
HeaderTitle.Font = Enum.Font.GothamBold
HeaderTitle.TextXAlignment = Enum.TextXAlignment.Left
HeaderTitle.ZIndex = 3
HeaderTitle.Parent = Header

-- Status dot
local StatusDot = Instance.new("Frame")
StatusDot.Size = UDim2.new(0, 8, 0, 8)
StatusDot.Position = UDim2.new(1, -290, 0.5, -4)
StatusDot.BackgroundColor3 = Tema.vermelho
StatusDot.BorderSizePixel = 0
StatusDot.ZIndex = 3
StatusDot.Parent = Header
corner(StatusDot, 4)

-- Badge
local Badge = Instance.new("TextLabel")
Badge.Size = UDim2.new(0, 100, 0, 22)
Badge.Position = UDim2.new(1, -220, 0.5, -11)
Badge.BackgroundColor3 = Tema.fundoCardAlt
Badge.Text = "v5.0  premium"
Badge.TextColor3 = Tema.textoSub
Badge.TextSize = 10
Badge.Font = Enum.Font.Gotham
Badge.BorderSizePixel = 0
Badge.ZIndex = 3
Badge.Parent = Header
corner(Badge, 6)
stroke(Badge, Tema.borda, 1)

local MinBtn = Instance.new("TextButton")
MinBtn.Size = UDim2.new(0, 26, 0, 26)
MinBtn.Position = UDim2.new(1, -72, 0.5, -13)
MinBtn.BackgroundColor3 = Tema.fundoCardAlt
MinBtn.Text = "—"
MinBtn.TextColor3 = Tema.textoSub
MinBtn.TextSize = 14
MinBtn.Font = Enum.Font.GothamBold
MinBtn.AutoButtonColor = false
MinBtn.BorderSizePixel = 0
MinBtn.ZIndex = 3
MinBtn.Parent = Header
corner(MinBtn, 6)

local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 26, 0, 26)
CloseBtn.Position = UDim2.new(1, -40, 0.5, -13)
CloseBtn.BackgroundColor3 = Tema.fundoCardAlt
CloseBtn.Text = "✕"
CloseBtn.TextColor3 = Tema.textoSub
CloseBtn.TextSize = 12
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.AutoButtonColor = false
CloseBtn.BorderSizePixel = 0
CloseBtn.ZIndex = 3
CloseBtn.Parent = Header
corner(CloseBtn, 6)

MinBtn.MouseEnter:Connect(function() TweenService:Create(MinBtn, TweenInfo.new(0.15), {BackgroundColor3 = Tema.fundoHover}):Play() end)
MinBtn.MouseLeave:Connect(function() TweenService:Create(MinBtn, TweenInfo.new(0.15), {BackgroundColor3 = Tema.fundoCardAlt}):Play() end)
CloseBtn.MouseEnter:Connect(function()
    TweenService:Create(CloseBtn, TweenInfo.new(0.15), {BackgroundColor3 = Tema.vermelho, TextColor3 = Tema.texto}):Play()
end)
CloseBtn.MouseLeave:Connect(function()
    TweenService:Create(CloseBtn, TweenInfo.new(0.15), {BackgroundColor3 = Tema.fundoCardAlt, TextColor3 = Tema.textoSub}):Play()
end)

-- Arrastar
do
    local dragging, inicio, inicioPos
    Header.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            inicio = input.Position
            inicioPos = MainFrame.Position
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            local d = input.Position - inicio
            MainFrame.Position = UDim2.new(
                inicioPos.X.Scale, inicioPos.X.Offset + d.X,
                inicioPos.Y.Scale, inicioPos.Y.Offset + d.Y
            )
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)
end

-----------------------------------------------------------
--// SIDEBAR
-----------------------------------------------------------
local Sidebar = Instance.new("Frame")
Sidebar.Size = UDim2.new(0, 170, 1, -40)
Sidebar.Position = UDim2.new(0, 0, 0, 40)
Sidebar.BackgroundColor3 = Tema.fundoSidebar
Sidebar.BorderSizePixel = 0
Sidebar.Parent = MainFrame

local SideSep = Instance.new("Frame")
SideSep.Size = UDim2.new(0, 1, 1, 0)
SideSep.Position = UDim2.new(1, 0, 0, 0)
SideSep.BackgroundColor3 = Tema.borda
SideSep.BorderSizePixel = 0
SideSep.Parent = Sidebar

local SideHeader = Instance.new("Frame")
SideHeader.Size = UDim2.new(1, 0, 0, 46)
SideHeader.BackgroundTransparency = 1
SideHeader.Parent = Sidebar

local LogoIcone = Instance.new("Frame")
LogoIcone.Size = UDim2.new(0, 26, 0, 26)
LogoIcone.Position = UDim2.new(0, 14, 0.5, -13)
LogoIcone.BackgroundColor3 = Tema.accent
LogoIcone.BorderSizePixel = 0
LogoIcone.Parent = SideHeader
corner(LogoIcone, 8)

local LogoTexto = Instance.new("TextLabel")
LogoTexto.Size = UDim2.new(0, 110, 1, 0)
LogoTexto.Position = UDim2.new(0, 48, 0, 0)
LogoTexto.BackgroundTransparency = 1
LogoTexto.Text = "ESP MENU"
LogoTexto.TextColor3 = Tema.texto
LogoTexto.TextSize = 13
LogoTexto.Font = Enum.Font.GothamBold
LogoTexto.TextXAlignment = Enum.TextXAlignment.Left
LogoTexto.Parent = SideHeader

-- Search global
local SearchFrame = Instance.new("Frame")
SearchFrame.Size = UDim2.new(1, -20, 0, 30)
SearchFrame.Position = UDim2.new(0, 10, 0, 54)
SearchFrame.BackgroundColor3 = Tema.fundoInput
SearchFrame.BorderSizePixel = 0
SearchFrame.Parent = Sidebar
corner(SearchFrame, 7)
stroke(SearchFrame, Tema.borda, 1)

local SearchIcon = Instance.new("TextLabel")
SearchIcon.Size = UDim2.new(0, 22, 1, 0)
SearchIcon.Position = UDim2.new(0, 6, 0, 0)
SearchIcon.BackgroundTransparency = 1
SearchIcon.Text = "🔍"
SearchIcon.TextColor3 = Tema.textoFraco
SearchIcon.TextSize = 11
SearchIcon.Parent = SearchFrame

local SearchBox = Instance.new("TextBox")
SearchBox.Size = UDim2.new(1, -30, 1, 0)
SearchBox.Position = UDim2.new(0, 28, 0, 0)
SearchBox.BackgroundTransparency = 1
SearchBox.Text = ""
SearchBox.PlaceholderText = "Search..."
SearchBox.PlaceholderColor3 = Tema.textoFraco
SearchBox.TextColor3 = Tema.texto
SearchBox.TextSize = 11
SearchBox.Font = Enum.Font.Gotham
SearchBox.TextXAlignment = Enum.TextXAlignment.Left
SearchBox.ClearTextOnFocus = false
SearchBox.Parent = SearchFrame

-- Nav
local SideNav = Instance.new("Frame")
SideNav.Size = UDim2.new(1, 0, 1, -200)
SideNav.Position = UDim2.new(0, 10, 0, 96)
SideNav.BackgroundTransparency = 1
SideNav.Parent = Sidebar

local SideNavLayout = Instance.new("UIListLayout")
SideNavLayout.Padding = UDim.new(0, 3)
SideNavLayout.SortOrder = Enum.SortOrder.LayoutOrder
SideNavLayout.Parent = SideNav

-- Perfil
local UserFrame = Instance.new("Frame")
UserFrame.Size = UDim2.new(1, -20, 0, 52)
UserFrame.Position = UDim2.new(0, 10, 1, -62)
UserFrame.BackgroundColor3 = Tema.fundoCardAlt
UserFrame.BorderSizePixel = 0
UserFrame.Parent = Sidebar
corner(UserFrame, 8)
stroke(UserFrame, Tema.borda, 1)

local UserAvatar = Instance.new("Frame")
UserAvatar.Size = UDim2.new(0, 32, 0, 32)
UserAvatar.Position = UDim2.new(0, 10, 0.5, -16)
UserAvatar.BackgroundColor3 = Tema.accent
UserAvatar.BorderSizePixel = 0
UserAvatar.Parent = UserFrame
corner(UserAvatar, 16)

local UserAvatarTxt = Instance.new("TextLabel")
UserAvatarTxt.Size = UDim2.new(1, 0, 1, 0)
UserAvatarTxt.BackgroundTransparency = 1
UserAvatarTxt.Text = string.sub(LocalPlayer.Name, 1, 1):upper()
UserAvatarTxt.TextColor3 = Tema.texto
UserAvatarTxt.TextSize = 14
UserAvatarTxt.Font = Enum.Font.GothamBold
UserAvatarTxt.Parent = UserAvatar

local UserNome = Instance.new("TextLabel")
UserNome.Size = UDim2.new(1, -50, 0, 16)
UserNome.Position = UDim2.new(0, 50, 0, 8)
UserNome.BackgroundTransparency = 1
UserNome.Text = LocalPlayer.Name
UserNome.TextColor3 = Tema.texto
UserNome.TextSize = 11
UserNome.Font = Enum.Font.GothamBold
UserNome.TextXAlignment = Enum.TextXAlignment.Left
UserNome.TextTruncate = Enum.TextTruncate.AtEnd
UserNome.Parent = UserFrame

local UserTag = Instance.new("TextLabel")
UserTag.Size = UDim2.new(1, -50, 0, 14)
UserTag.Position = UDim2.new(0, 50, 0, 24)
UserTag.BackgroundTransparency = 1
UserTag.Text = "@" .. string.lower(LocalPlayer.Name)
UserTag.TextColor3 = Tema.textoFraco
UserTag.TextSize = 10
UserTag.Font = Enum.Font.Gotham
UserTag.TextXAlignment = Enum.TextXAlignment.Left
UserTag.TextTruncate = Enum.TextTruncate.AtEnd
UserTag.Parent = UserFrame

-----------------------------------------------------------
--// COMPONENTES
-----------------------------------------------------------
local function criarSidebarBtn(parent, nome, icone, ordem, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, -20, 0, 38)
    btn.BackgroundColor3 = Tema.fundoSidebar
    btn.BackgroundTransparency = 1
    btn.Text = ""
    btn.AutoButtonColor = false
    btn.BorderSizePixel = 0
    btn.LayoutOrder = ordem
    btn.Parent = parent
    corner(btn, 8)
    
    local iconeLbl = Instance.new("TextLabel")
    iconeLbl.Size = UDim2.new(0, 24, 1, 0)
    iconeLbl.Position = UDim2.new(0, 10, 0, 0)
    iconeLbl.BackgroundTransparency = 1
    iconeLbl.Text = icone
    iconeLbl.TextColor3 = Tema.textoSub
    iconeLbl.TextSize = 16
    iconeLbl.Font = Enum.Font.GothamBold
    iconeLbl.Parent = btn
    
    local nomeLbl = Instance.new("TextLabel")
    nomeLbl.Size = UDim2.new(1, -45, 1, 0)
    nomeLbl.Position = UDim2.new(0, 40, 0, 0)
    nomeLbl.BackgroundTransparency = 1
    nomeLbl.Text = nome
    nomeLbl.TextColor3 = Tema.textoSub
    nomeLbl.TextSize = 12
    nomeLbl.Font = Enum.Font.GothamMedium
    nomeLbl.TextXAlignment = Enum.TextXAlignment.Left
    nomeLbl.Parent = btn
    
    local indicador = Instance.new("Frame")
    indicador.Size = UDim2.new(0, 3, 0, 0)
    indicador.Position = UDim2.new(0, 0, 0.5, 0)
    indicador.AnchorPoint = Vector2.new(0, 0.5)
    indicador.BackgroundColor3 = Tema.accent
    indicador.BorderSizePixel = 0
    indicador.Parent = btn
    corner(indicador, 2)
    
    btn.MouseEnter:Connect(function()
        if not btn:GetAttribute("ativo") then
            TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundTransparency = 0.6}):Play()
            btn.BackgroundColor3 = Tema.fundoHover
        end
    end)
    btn.MouseLeave:Connect(function()
        if not btn:GetAttribute("ativo") then
            TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundTransparency = 1}):Play()
        end
    end)
    btn.MouseButton1Click:Connect(callback)
    
    return {btn = btn, icone = iconeLbl, nome = nomeLbl, indicador = indicador}
end

local function criarCard(parent, titulo, icone, ordem, abertoInicial)
    abertoInicial = abertoInicial ~= false
    local card = Instance.new("Frame")
    card.Size = UDim2.new(1, 0, 0, abertoInicial and 60 or 44)
    card.BackgroundColor3 = Tema.fundoCard
    card.BorderSizePixel = 0
    card.LayoutOrder = ordem or 1
    card.ClipsDescendants = true
    card.Parent = parent
    corner(card, 10)
    stroke(card, Tema.borda, 1)
    
    local header = Instance.new("TextButton")
    header.Size = UDim2.new(1, 0, 0, 44)
    header.BackgroundTransparency = 1
    header.Text = ""
    header.AutoButtonColor = false
    header.BorderSizePixel = 0
    header.Parent = card
    
    local headerIcone = Instance.new("TextLabel")
    headerIcone.Size = UDim2.new(0, 20, 1, 0)
    headerIcone.Position = UDim2.new(0, 14, 0, 0)
    headerIcone.BackgroundTransparency = 1
    headerIcone.Text = icone or "◆"
    headerIcone.TextColor3 = Tema.accent
    headerIcone.TextSize = 14
    headerIcone.Font = Enum.Font.GothamBold
    headerIcone.Parent = header
    
    local headerTitulo = Instance.new("TextLabel")
    headerTitulo.Size = UDim2.new(1, -70, 1, 0)
    headerTitulo.Position = UDim2.new(0, 38, 0, 0)
    headerTitulo.BackgroundTransparency = 1
    headerTitulo.Text = string.upper(titulo)
    headerTitulo.TextColor3 = Tema.texto
    headerTitulo.TextSize = 12
    headerTitulo.Font = Enum.Font.GothamBold
    headerTitulo.TextXAlignment = Enum.TextXAlignment.Left
    headerTitulo.Parent = header
    
    local chevron = Instance.new("TextLabel")
    chevron.Size = UDim2.new(0, 20, 1, 0)
    chevron.Position = UDim2.new(1, -30, 0, 0)
    chevron.BackgroundTransparency = 1
    chevron.Text = "▾"
    chevron.TextColor3 = Tema.textoSub
    chevron.TextSize = 14
    chevron.Font = Enum.Font.GothamBold
    chevron.Rotation = abertoInicial and 0 or -90
    chevron.Parent = header
    
    local content = Instance.new("Frame")
    content.Size = UDim2.new(1, -24, 0, 0)
    content.Position = UDim2.new(0, 12, 0, 44)
    content.BackgroundTransparency = 1
    content.AutomaticSize = Enum.AutomaticSize.Y
    content.Parent = card
    
    local layout = Instance.new("UIListLayout")
    layout.Padding = UDim.new(0, 6)
    layout.SortOrder = Enum.SortOrder.LayoutOrder
    layout.Parent = content
    
    local pad = Instance.new("UIPadding")
    pad.PaddingBottom = UDim.new(0, 12)
    pad.Parent = content
    
    content:GetPropertyChangedSignal("AbsoluteSize"):Connect(function()
        if card:GetAttribute("aberto") then
            card.Size = UDim2.new(1, 0, 0, 44 + content.AbsoluteSize.Y + 12)
        end
    end)
    
    card:SetAttribute("aberto", abertoInicial)
    
    header.MouseButton1Click:Connect(function()
        local aberto = not card:GetAttribute("aberto")
        card:SetAttribute("aberto", aberto)
        chevron.Rotation = aberto and 0 or -90
        if aberto then
            card.Size = UDim2.new(1, 0, 0, 44 + content.AbsoluteSize.Y + 12)
        else
            card.Size = UDim2.new(1, 0, 0, 44)
        end
    end)
    
    return content
end

local function criarToggle(parent, texto, chaveConfig, ordem)
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1, 0, 0, 34)
    row.BackgroundTransparency = 1
    row.LayoutOrder = ordem or 1
    row.Parent = parent
    
    local lbl = Instance.new("TextLabel")
    lbl.Size = UDim2.new(1, -70, 1, 0)
    lbl.Position = UDim2.new(0, 4, 0, 0)
    lbl.BackgroundTransparency = 1
    lbl.Text = texto
    lbl.TextColor3 = Tema.texto
    lbl.TextSize = 12
    lbl.Font = Enum.Font.Gotham
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Parent = row
    
    local pill = Instance.new("TextButton")
    pill.Size = UDim2.new(0, 40, 0, 22)
    pill.Position = UDim2.new(1, -40, 0.5, -11)
    pill.BackgroundColor3 = Tema.fundoInput
    pill.Text = ""
    pill.AutoButtonColor = false
    pill.BorderSizePixel = 0
    pill.Parent = row
    corner(pill, 11)
    stroke(pill, Tema.borda, 1)
    
    local bolinha = Instance.new("Frame")
    bolinha.Size = UDim2.new(0, 16, 0, 16)
    bolinha.Position = UDim2.new(0, 3, 0.5, -8)
    bolinha.BackgroundColor3 = Tema.textoSub
    bolinha.BorderSizePixel = 0
    bolinha.Parent = pill
    corner(bolinha, 8)
    
    local estado = Store:get(chaveConfig)
    
    local function atualizar(animar)
        local pos = estado and UDim2.new(1, -19, 0.5, -8) or UDim2.new(0, 3, 0.5, -8)
        local corPill = estado and Tema.accent or Tema.fundoInput
        local corBol = estado and Tema.texto or Tema.textoSub
        
        if animar then
            TweenService:Create(bolinha, TweenInfo.new(0.2, Enum.EasingStyle.Quart), {Position = pos}):Play()
            TweenService:Create(pill, TweenInfo.new(0.2), {BackgroundColor3 = corPill}):Play()
            TweenService:Create(bolinha, TweenInfo.new(0.2), {BackgroundColor3 = corBol}):Play()
        else
            bolinha.Position = pos
            pill.BackgroundColor3 = corPill
            bolinha.BackgroundColor3 = corBol
        end
    end
    atualizar(false)
    
    pill.MouseButton1Click:Connect(function()
        estado = not estado
        Store:set(chaveConfig, estado)
        atualizar(true)
    end)
    
    Store:subscribe(chaveConfig, function(v)
        if estado ~= v then
            estado = v
            atualizar(true)
        end
    end)
    
    return row
end

local function criarSlider(parent, texto, chaveConfig, min, max, passo, ordem, sufixo)
    passo = passo or 1
    sufixo = sufixo or ""
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1, 0, 0, 46)
    row.BackgroundTransparency = 1
    row.LayoutOrder = ordem or 1
    row.Parent = parent
    
    local topRow = Instance.new("Frame")
    topRow.Size = UDim2.new(1, 0, 0, 18)
    topRow.BackgroundTransparency = 1
    topRow.Parent = row
    
    local lbl = Instance.new("TextLabel")
    lbl.Size = UDim2.new(0.6, 0, 1, 0)
    lbl.Position = UDim2.new(0, 4, 0, 0)
    lbl.BackgroundTransparency = 1
    lbl.Text = texto
    lbl.TextColor3 = Tema.texto
    lbl.TextSize = 12
    lbl.Font = Enum.Font.Gotham
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Parent = topRow
    
    local valorLbl = Instance.new("TextLabel")
    valorLbl.Size = UDim2.new(0.4, 0, 1, 0)
    valorLbl.Position = UDim2.new(0.6, 0, 0, 0)
    valorLbl.BackgroundTransparency = 1
    valorLbl.TextColor3 = Tema.textoSub
    valorLbl.TextSize = 11
    valorLbl.Font = Enum.Font.Gotham
    valorLbl.TextXAlignment = Enum.TextXAlignment.Right
    valorLbl.Parent = topRow
    
    local barraBg = Instance.new("Frame")
    barraBg.Size = UDim2.new(1, -8, 0, 6)
    barraBg.Position = UDim2.new(0, 4, 0, 30)
    barraBg.BackgroundColor3 = Tema.fundoInput
    barraBg.BorderSizePixel = 0
    barraBg.Parent = row
    corner(barraBg, 3)
    
    local fill = Instance.new("Frame")
    fill.Size = UDim2.new(0, 0, 1, 0)
    fill.BackgroundColor3 = Tema.accent
    fill.BorderSizePixel = 0
    fill.Parent = barraBg
    corner(fill, 3)
    
    local thumb = Instance.new("Frame")
    thumb.Size = UDim2.new(0, 14, 0, 14)
    thumb.Position = UDim2.new(0, 0, 0.5, -7)
    thumb.BackgroundColor3 = Tema.texto
    thumb.BorderSizePixel = 0
    thumb.AnchorPoint = Vector2.new(0.5, 0.5)
    thumb.Parent = barraBg
    corner(thumb, 7)
    
    local hitbox = Instance.new("TextButton")
    hitbox.Size = UDim2.new(1, 0, 3, 0)
    hitbox.Position = UDim2.new(0, 0, 0.5, 0)
    hitbox.AnchorPoint = Vector2.new(0, 0.5)
    hitbox.BackgroundTransparency = 1
    hitbox.Text = ""
    hitbox.Parent = barraBg
    
    local valor = Store:get(chaveConfig) or min
    
    local function fmt(v)
        if passo >= 1 then return string.format("%d%s", math.floor(v + 0.5), sufixo)
        else return string.format("%.2f%s", v, sufixo) end
    end
    
    local function atualizar()
        local pct = math.clamp((valor - min) / (max - min), 0, 1)
        fill.Size = UDim2.new(pct, 0, 1, 0)
        thumb.Position = UDim2.new(pct, 0, 0.5, 0)
        valorLbl.Text = fmt(valor)
    end
    atualizar()
    
    local dragging = false
    local connM, connE
    
    local function parar()
        dragging = false
        if connM then connM:Disconnect() connM = nil end
        if connE then connE:Disconnect() connE = nil end
    end
    
    local function aplicar(input)
        local relX = math.clamp((input.Position.X - barraBg.AbsolutePosition.X) / barraBg.AbsoluteSize.X, 0, 1)
        local novo = min + (max - min) * relX
        novo = math.floor(novo / passo + 0.5) * passo
        novo = math.clamp(novo, min, max)
        valor = novo
        Store:set(chaveConfig, valor)
        atualizar()
    end
    
    hitbox.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            aplicar(input)
            connM = UserInputService.InputChanged:Connect(function(inp)
                if dragging and (inp.UserInputType == Enum.UserInputType.MouseMovement or inp.UserInputType == Enum.UserInputType.Touch) then
                    aplicar(inp)
                end
            end)
            connE = UserInputService.InputEnded:Connect(function(inp)
                if inp.UserInputType == Enum.UserInputType.MouseButton1 or inp.UserInputType == Enum.UserInputType.Touch then
                    parar()
                end
            end)
        end
    end)
    
    Store:subscribe(chaveConfig, function(v)
        valor = v
        atualizar()
    end)
    
    return row
end

local function criarBotaoAcao(parent, texto, corFundo, corTexto, ordem, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 34)
    btn.BackgroundColor3 = corFundo
    btn.Text = texto
    btn.TextColor3 = corTexto or Tema.texto
    btn.TextSize = 12
    btn.Font = Enum.Font.GothamMedium
    btn.AutoButtonColor = false
    btn.BorderSizePixel = 0
    btn.LayoutOrder = ordem or 1
    btn.Parent = parent
    corner(btn, 7)
    stroke(btn, corFundo, 1)
    
    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundTransparency = 0.15}):Play()
    end)
    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundTransparency = 0}):Play()
    end)
    btn.MouseButton1Click:Connect(function()
        local ok, err = pcall(callback)
        if not ok then warn("[ESP] erro:", err) end
    end)
    return btn
end

local function criarInput(parent, placeholder, ordem, callback)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 0, 34)
    frame.BackgroundColor3 = Tema.fundoInput
    frame.BorderSizePixel = 0
    frame.LayoutOrder = ordem or 1
    frame.Parent = parent
    corner(frame, 7)
    stroke(frame, Tema.borda, 1)
    
    local iconL = Instance.new("TextLabel")
    iconL.Size = UDim2.new(0, 20, 1, 0)
    iconL.Position = UDim2.new(0, 8, 0, 0)
    iconL.BackgroundTransparency = 1
    iconL.Text = "○"
    iconL.TextColor3 = Tema.textoFraco
    iconL.TextSize = 10
    iconL.Font = Enum.Font.GothamBold
    iconL.Parent = frame
    
    local box = Instance.new("TextBox")
    box.Size = UDim2.new(1, -40, 1, 0)
    box.Position = UDim2.new(0, 32, 0, 0)
    box.BackgroundTransparency = 1
    box.Text = ""
    box.PlaceholderText = placeholder
    box.PlaceholderColor3 = Tema.textoFraco
    box.TextColor3 = Tema.texto
    box.TextSize = 12
    box.Font = Enum.Font.Gotham
    box.ClearTextOnFocus = false
    box.TextXAlignment = Enum.TextXAlignment.Left
    box.Parent = frame
    
    box:GetPropertyChangedSignal("Text"):Connect(function()
        if callback then pcall(callback, box.Text) end
    end)
    return frame, box
end

local function criarColorPicker(parent, texto, chaveConfig, ordem)
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1, 0, 0, 32)
    row.BackgroundTransparency = 1
    row.LayoutOrder = ordem or 1
    row.Parent = parent
    
    local lbl = Instance.new("TextLabel")
    lbl.Size = UDim2.new(1, -50, 1, 0)
    lbl.Position = UDim2.new(0, 4, 0, 0)
    lbl.BackgroundTransparency = 1
    lbl.Text = texto
    lbl.TextColor3 = Tema.texto
    lbl.TextSize = 12
    lbl.Font = Enum.Font.Gotham
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Parent = row
    
    local preview = Instance.new("TextButton")
    preview.Size = UDim2.new(0, 36, 0, 22)
    preview.Position = UDim2.new(1, -36, 0.5, -11)
    preview.Text = ""
    preview.AutoButtonColor = false
    preview.BorderSizePixel = 0
    preview.Parent = row
    corner(preview, 6)
    stroke(preview, Tema.borda, 1)
    
    local paleta = {
        Color3.fromRGB(255, 255, 255),
        Color3.fromRGB(80, 220, 130),
        Color3.fromRGB(255, 220, 100),
        Color3.fromRGB(240, 80, 90),
        Color3.fromRGB(130, 90, 255),
        Color3.fromRGB(80, 180, 255),
        Color3.fromRGB(255, 130, 200),
        Color3.fromRGB(255, 130, 0),
        Color3.fromRGB(0, 0, 0),
    }
    
    local cor = Store:get(chaveConfig)
    local idx = 1
    for i, c in ipairs(paleta) do
        if (c.R - cor.R)^2 + (c.G - cor.G)^2 + (c.B - cor.B)^2 < 0.01 then
            idx = i
            break
        end
    end
    
    local function atualizar() preview.BackgroundColor3 = cor end
    atualizar()
    
    preview.MouseButton1Click:Connect(function()
        idx = idx + 1
        if idx > #paleta then idx = 1 end
        cor = paleta[idx]
        Store:set(chaveConfig, cor)
        atualizar()
    end)
    
    Store:subscribe(chaveConfig, function(v)
        cor = v
        atualizar()
    end)
    return row
end

-----------------------------------------------------------
--// ESTADO ESP
-----------------------------------------------------------
local espGuiElements = {}
local caixasESP = {}
local highlights = {}
local tracersESP = {}
local jogadoresSelecionados = {}

local function jogadorVisivel(p)
    if p == LocalPlayer and not Config.mostrarLocal then return false end
    if jogadoresSelecionados[p] == false then return false end
    if not Config.espAtivo and jogadoresSelecionados[p] ~= true then return false end
    return true
end

local function limparPlayer(p)
    if espGuiElements[p] then espGuiElements[p]:Destroy() espGuiElements[p] = nil end
    if caixasESP[p] then caixasESP[p]:Destroy() caixasESP[p] = nil end
    if highlights[p] then highlights[p]:Destroy() highlights[p] = nil end
    if tracersESP[p] then tracersESP[p]:Destroy() tracersESP[p] = nil end
end

local function limparTodos()
    for p, _ in pairs(espGuiElements) do limparPlayer(p) end
    for p, _ in pairs(caixasESP) do limparPlayer(p) end
    for p, _ in pairs(highlights) do limparPlayer(p) end
    for p, _ in pairs(tracersESP) do limparPlayer(p) end
end

local function criarESPBasico(p)
    if espGuiElements[p] then return end
    if p == LocalPlayer and not Config.mostrarLocal then return end
    if jogadoresSelecionados[p] == false then return end
    
    local char = p.Character
    if not char then return end
    local head = char:FindFirstChild("Head")
    if not head then return end
    if not char:FindFirstChildOfClass("Humanoid") then return end
    
    local bb = Instance.new("BillboardGui")
    bb.Name = "ESP_BB"
    bb.Size = UDim2.new(0, Config.tamanhoLargura, 0, Config.tamanhoAltura)
    bb.StudsOffset = Vector3.new(0, Config.offsetY, 0)
    bb.AlwaysOnTop = Config.sempreVisivel
    bb.Adornee = head
    bb.Parent = head
    
    local bg = Instance.new("Frame")
    bg.Name = "BG"
    bg.Size = UDim2.new(1, 0, 1, 0)
    bg.BackgroundColor3 = Config.corFundo
    bg.BackgroundTransparency = Config.transparenciaFundo
    bg.BorderSizePixel = 0
    bg.Visible = Config.mostrarFundo
    bg.Parent = bb
    corner(bg, 6)
    
    local borda = Instance.new("UIStroke")
    borda.Name = "Borda"
    borda.Color = Config.corBorda
    borda.Thickness = 1.5
    borda.Transparency = Config.mostrarBorda and 0 or 1
    borda.Parent = bg
    
    local nome = Instance.new("TextLabel")
    nome.Name = "Nome"
    nome.Size = UDim2.new(1, -10, 0, 22)
    nome.Position = UDim2.new(0, 5, 0, 4)
    nome.BackgroundTransparency = 1
    nome.Text = (Config.mostrarIcone and "● " or "") .. p.Name
    nome.TextColor3 = Config.corNome
    nome.TextSize = Config.tamanhoFonteNome
    nome.Font = Enum.Font.GothamBold
    nome.TextStrokeTransparency = Config.mostrarSombra and 0 or 1
    nome.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    nome.Visible = Config.mostrarNome
    nome.Parent = bb
    
    local vBg = Instance.new("Frame")
    vBg.Name = "VidaBG"
    vBg.Size = UDim2.new(1, -10, 0, Config.espessuraBarra)
    vBg.Position = UDim2.new(0, 5, 0, 28)
    vBg.BackgroundColor3 = Config.corVidaBG
    vBg.BorderSizePixel = 0
    vBg.Visible = Config.mostrarVida
    vBg.Parent = bb
    corner(vBg, 3)
    
    local vFill = Instance.new("Frame")
    vFill.Name = "VidaFill"
    vFill.Size = UDim2.new(1, 0, 1, 0)
    vFill.BackgroundColor3 = Config.corVidaCheia
    vFill.BorderSizePixel = 0
    vFill.Parent = vBg
    corner(vFill, 3)
    
    local vTxt = Instance.new("TextLabel")
    vTxt.Name = "VidaTxt"
    vTxt.Size = UDim2.new(1, 0, 1, 0)
    vTxt.BackgroundTransparency = 1
    vTxt.Text = "100"
    vTxt.TextColor3 = Color3.fromRGB(255, 255, 255)
    vTxt.TextSize = 10
    vTxt.Font = Enum.Font.GothamBold
    vTxt.TextStrokeTransparency = 0
    vTxt.ZIndex = 2
    vTxt.Visible = Config.mostrarHPnumero
    vTxt.Parent = vBg
    
    local dLbl = Instance.new("TextLabel")
    dLbl.Name = "Dist"
    dLbl.Size = UDim2.new(1, -10, 0, 18)
    dLbl.Position = UDim2.new(0, 5, 0, 42)
    dLbl.BackgroundTransparency = 1
    dLbl.Text = "0m"
    dLbl.TextColor3 = Config.corDistancia
    dLbl.TextSize = Config.tamanhoFonteDist
    dLbl.Font = Enum.Font.GothamBold
    dLbl.TextStrokeTransparency = Config.mostrarSombra and 0 or 1
    dLbl.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    dLbl.Visible = Config.mostrarDistancia
    dLbl.Parent = bb
    
    espGuiElements[p] = bb
end

local function criarCaixa(p)
    if caixasESP[p] then return end
    if not Config.mostrarCaixa then return end
    local char = p.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    local box = Instance.new("BoxHandleAdornment")
    box.Name = "ESP_Box"
    box.Size = Vector3.new(2, 5, 1)
    box.Adornee = hrp
    box.AlwaysOnTop = true
    box.ZIndex = 5
    box.Transparency = 0.5
    box.Color3 = Config.corCaixa
    box.Parent = hrp
    caixasESP[p] = box
end

local function criarHighlight(p)
    if highlights[p] then return end
    if not Config.mostrarHighlight then return end
    local char = p.Character
    if not char then return end
    
    local hl = Instance.new("Highlight")
    hl.Name = "ESP_HL"
    hl.FillColor = Config.corHighlight
    hl.FillTransparency = 0.6
    hl.OutlineColor = Config.corBorda
    hl.OutlineTransparency = 0.3
    hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    hl.Adornee = char
    hl.Parent = char
    highlights[p] = hl
end

local function criarTracer(p)
    if tracersESP[p] then return end
    if not Config.mostrarTracer then return end
    if p == LocalPlayer and not Config.mostrarLocal then return end
    if jogadoresSelecionados[p] == false then return end
    
    local gui = ScreenGui:FindFirstChild("TracerGUI")
    if not gui then
        gui = Instance.new("Frame")
        gui.Name = "TracerGUI"
        gui.Size = UDim2.new(1, 0, 1, 0)
        gui.BackgroundTransparency = 1
        gui.ZIndex = 0
        gui.Parent = ScreenGui
    end
    
    local linha = Instance.new("Frame")
    linha.Name = p.Name .. "_Tracer"
    linha.BackgroundColor3 = Config.corTracer
    linha.BorderSizePixel = 0
    linha.AnchorPoint = Vector2.new(0.5, 0.5)
    linha.ZIndex = 0
    linha.Parent = gui
    
    tracersESP[p] = linha
end

-----------------------------------------------------------
--// LOCALIZADOR
-----------------------------------------------------------
local localizadorGui, setaLocalizador, linhaLocalizador, infoLocalizador, alvoLocalizador

local function criarLocalizadorGui()
    if localizadorGui then return end
    localizadorGui = Instance.new("ScreenGui")
    localizadorGui.Name = "LocalizadorGUI"
    localizadorGui.ResetOnSpawn = false
    localizadorGui.IgnoreGuiInset = true
    localizadorGui.Parent = CoreGui
    
    setaLocalizador = Instance.new("ImageLabel")
    setaLocalizador.Size = UDim2.new(0, 60, 0, 60)
    setaLocalizador.Position = UDim2.new(0.5, -30, 0.5, -70)
    setaLocalizador.BackgroundTransparency = 1
    setaLocalizador.Image = "rbxassetid://6208090560"
    setaLocalizador.ImageColor3 = Config.corLocalizador
    setaLocalizador.Parent = localizadorGui
    
    linhaLocalizador = Instance.new("Frame")
    linhaLocalizador.BackgroundColor3 = Config.corLocalizador
    linhaLocalizador.BackgroundTransparency = 0.3
    linhaLocalizador.BorderSizePixel = 0
    linhaLocalizador.AnchorPoint = Vector2.new(0.5, 0.5)
    linhaLocalizador.ZIndex = 0
    linhaLocalizador.Parent = localizadorGui
    
    infoLocalizador = Instance.new("TextLabel")
    infoLocalizador.Size = UDim2.new(0, 320, 0, 34)
    infoLocalizador.Position = UDim2.new(0.5, -160, 0.5, 120)
    infoLocalizador.BackgroundColor3 = Tema.fundoCard
    infoLocalizador.BackgroundTransparency = 0.2
    infoLocalizador.Text = ""
    infoLocalizador.TextColor3 = Config.corLocalizador
    infoLocalizador.TextSize = 13
    infoLocalizador.Font = Enum.Font.GothamBold
    infoLocalizador.BorderSizePixel = 0
    infoLocalizador.Parent = localizadorGui
    corner(infoLocalizador, 8)
    stroke(infoLocalizador, Config.corLocalizador, 1)
end

local function pararLocalizador()
    Config.localizadorAtivo = false
    Store:set("localizadorAtivo", false)
    alvoLocalizador = nil
    if localizadorGui then localizadorGui:Destroy() localizadorGui = nil end
    setaLocalizador = nil
    linhaLocalizador = nil
    infoLocalizador = nil
end

local function iniciarLocalizador(p)
    if not p then return end
    pararLocalizador()
    alvoLocalizador = p
    Config.localizadorAtivo = true
    criarLocalizadorGui()
    notificar("◎ A localizar: " .. p.Name, Tema.accent)
end

-----------------------------------------------------------
--// LOOP PRINCIPAL
-----------------------------------------------------------
local acc = 0
local INTERVALO = 1 / 30

RunService.Heartbeat:Connect(function(dt)
    acc = acc + dt
    if acc < INTERVALO then return end
    acc = 0
    
    pcall(function()
        local myChar = LocalPlayer.Character
        local myHrp = myChar and myChar:FindFirstChild("HumanoidRootPart")
        local cam = workspace.CurrentCamera
        if not cam then return end
        
        for _, p in ipairs(Players:GetPlayers()) do
            if jogadorVisivel(p) then
                if not espGuiElements[p] then criarESPBasico(p) end
                if Config.mostrarCaixa and not caixasESP[p] then criarCaixa(p) end
                if Config.mostrarHighlight and not highlights[p] then criarHighlight(p) end
                if Config.mostrarTracer and not tracersESP[p] then criarTracer(p) end
            else
                if espGuiElements[p] or caixasESP[p] or highlights[p] or tracersESP[p] then limparPlayer(p) end
            end
        end
        
        -- Atualizar ESPs
        for p, bb in pairs(espGuiElements) do
            local char = p.Character
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            local head = char and char:FindFirstChild("Head")
            if not hum or not head or not bb.Parent then
                if not char then limparPlayer(p) end
            else
                local esconderMorto = not Config.mostrarMortos and hum.Health <= 0
                local dist = 0
                if myHrp and head then
                    dist = (myHrp.Position - head.Position).Magnitude
                end
                local foraAlcance = dist > Config.distanciaMaxima
                bb.Enabled = not esconderMorto and not foraAlcance
                
                if bb.Enabled then
                    local vida = math.max(0, hum.Health)
                    local vMax = hum.MaxHealth > 0 and hum.MaxHealth or 100
                    local pct = vida / vMax
                    
                    local vFill = bb:FindFirstChild("VidaBG") and bb.VidaBG:FindFirstChild("VidaFill")
                    local vTxt = bb:FindFirstChild("VidaBG") and bb.VidaBG:FindFirstChild("VidaTxt")
                    local dLbl = bb:FindFirstChild("Dist")
                    
                    if vFill then
                        vFill.Size = UDim2.new(pct, 0, 1, 0)
                        if pct > 0.6 then vFill.BackgroundColor3 = Config.corVidaCheia
                        elseif pct > 0.3 then vFill.BackgroundColor3 = Config.corVidaMedia
                        else vFill.BackgroundColor3 = Config.corVidaBaixa end
                    end
                    if vTxt then vTxt.Text = string.format("%d", math.floor(vida)) end
                    if dLbl then dLbl.Text = string.format("%.0fm", dist) end
                end
            end
        end
        
        -- Caixas
        for p, box in pairs(caixasESP) do
            if not Config.mostrarCaixa or not box.Parent then
                if box then box:Destroy() end
                caixasESP[p] = nil
            else
                box.Color3 = Config.corCaixa
                local char = p.Character
                local hum = char and char:FindFirstChildOfClass("Humanoid")
                if hum then
                    box.Transparency = (not Config.mostrarMortos and hum.Health <= 0) and 1 or 0.5
                end
            end
        end
        
        -- Highlights
        for p, hl in pairs(highlights) do
            if not Config.mostrarHighlight or not hl.Parent then
                if hl then hl:Destroy() end
                highlights[p] = nil
            else
                hl.FillColor = Config.corHighlight
                hl.OutlineColor = Config.corBorda
            end
        end
        
        -- Tracers
        if myHrp and cam then
            local vp = cam.ViewportSize
            for p, linha in pairs(tracersESP) do
                local char = p.Character
                local head = char and char:FindFirstChild("Head")
                local hum = char and char:FindFirstChildOfClass("Humanoid")
                if head and hum and linha.Parent then
                    if not Config.mostrarMortos and hum.Health <= 0 then
                        linha.Visible = false
                    else
                        linha.Visible = true
                        local screenPos = cam:WorldToViewportPoint(head.Position)
                        local ox, oy = vp.X / 2, vp.Y
                        local dx, dy = screenPos.X - ox, screenPos.Y - oy
                        local comp = math.sqrt(dx*dx + dy*dy)
                        local ang = math.deg(math.atan2(dy, dx))
                        linha.Size = UDim2.new(0, comp, 0, 1.5)
                        linha.Position = UDim2.new(0, ox + dx/2, 0, oy + dy/2)
                        linha.Rotation = ang
                        linha.BackgroundColor3 = Config.corTracer
                    end
                elseif not char then
                    if linha then linha:Destroy() end
                    tracersESP[p] = nil
                end
            end
        end
        
        -- Localizador
        if Config.localizadorAtivo and alvoLocalizador and localizadorGui and setaLocalizador and myHrp then
            local char = alvoLocalizador.Character
            local hrp = char and char:FindFirstChild("HumanoidRootPart")
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            
            if hrp then
                local screenPos, onScreen = cam:WorldToViewportPoint(hrp.Position)
                local rel = cam.CFrame:PointToObjectSpace(hrp.Position)
                local ang = math.deg(math.atan2(rel.X, -rel.Z))
                ang = (ang + 360) % 360
                setaLocalizador.Rotation = ang
                setaLocalizador.ImageColor3 = Config.corLocalizador
                
                if linhaLocalizador then
                    local vp = cam.ViewportSize
                    local cx, cy = vp.X / 2, vp.Y / 2
                    local tx, ty
                    if onScreen then tx, ty = screenPos.X, screenPos.Y
                    else
                        local dir = Vector2.new(screenPos.X - cx, screenPos.Y - cy)
                        if dir.Magnitude > 0 then dir = dir.Unit end
                        tx = cx + dir.X * 250
                        ty = cy + dir.Y * 250
                    end
                    local dx, dy = tx - cx, ty - cy
                    local comp = math.sqrt(dx*dx + dy*dy)
                    local a2 = math.deg(math.atan2(dy, dx))
                    linhaLocalizador.Size = UDim2.new(0, comp, 0, Config.espessuraLinhaLoc)
                    linhaLocalizador.Position = UDim2.new(0, cx + dx/2, 0, cy + dy/2)
                    linhaLocalizador.Rotation = a2
                    linhaLocalizador.BackgroundColor3 = Config.corLocalizador
                end
                
                if infoLocalizador and hum then
                    local dist = (myHrp.Position - hrp.Position).Magnitude
                    infoLocalizador.Text = string.format("◎ %s   ❤ %d/%d   %dm",
                        alvoLocalizador.Name, math.floor(hum.Health), math.floor(hum.MaxHealth), math.floor(dist))
                    infoLocalizador.TextColor3 = Config.corLocalizador
                end
            end
        end
    end)
end)

-- Subscribers
Store:subscribe("espAtivo", function(v)
    TweenService:Create(StatusDot, TweenInfo.new(0.3), {
        BackgroundColor3 = v and Tema.verde or Tema.vermelho
    }):Play()
    if not v then
        for p, _ in pairs(espGuiElements) do
            if jogadoresSelecionados[p] ~= true then limparPlayer(p) end
        end
    end
end)

Store:subscribe("mostrarTracer", function(v)
    if v then
        for _, p in ipairs(Players:GetPlayers()) do
            if jogadorVisivel(p) then criarTracer(p) end
        end
    else
        for p, linha in pairs(tracersESP) do
            if linha then linha:Destroy() end
            tracersESP[p] = nil
        end
    end
end)

Store:subscribe("mostrarCaixa", function(v)
    if v then
        for _, p in ipairs(Players:GetPlayers()) do
            if jogadorVisivel(p) then criarCaixa(p) end
        end
    else
        for p, box in pairs(caixasESP) do
            if box then box:Destroy() end
            caixasESP[p] = nil
        end
    end
end)

Store:subscribe("mostrarHighlight", function(v)
    if v then
        for _, p in ipairs(Players:GetPlayers()) do
            if jogadorVisivel(p) then criarHighlight(p) end
        end
    else
        for p, hl in pairs(highlights) do
            if hl then hl:Destroy() end
            highlights[p] = nil
        end
    end
end)

Store:subscribe("*", function()
    for p, bb in pairs(espGuiElements) do
        if not bb.Parent then continue end
        bb.Size = UDim2.new(0, Config.tamanhoLargura, 0, Config.tamanhoAltura)
        bb.StudsOffset = Vector3.new(0, Config.offsetY, 0)
        bb.AlwaysOnTop = Config.sempreVisivel
        
        local bg = bb:FindFirstChild("BG")
        if bg then
            bg.BackgroundColor3 = Config.corFundo
            bg.BackgroundTransparency = Config.transparenciaFundo
            bg.Visible = Config.mostrarFundo
            local borda = bg:FindFirstChildOfClass("UIStroke")
            if borda then
                borda.Color = Config.corBorda
                borda.Transparency = Config.mostrarBorda and 0 or 1
            end
        end
        
        local nome = bb:FindFirstChild("Nome")
        if nome then
            nome.Visible = Config.mostrarNome
            nome.TextColor3 = Config.corNome
            nome.TextSize = Config.tamanhoFonteNome
            nome.TextStrokeTransparency = Config.mostrarSombra and 0 or 1
            nome.Text = (Config.mostrarIcone and "● " or "") .. p.Name
        end
        
        local vBg = bb:FindFirstChild("VidaBG")
        if vBg then
            vBg.Visible = Config.mostrarVida
            vBg.BackgroundColor3 = Config.corVidaBG
            vBg.Size = UDim2.new(1, -10, 0, Config.espessuraBarra)
            local vTxt = vBg:FindFirstChild("VidaTxt")
            if vTxt then vTxt.Visible = Config.mostrarHPnumero end
        end
        
        local dLbl = bb:FindFirstChild("Dist")
        if dLbl then
            dLbl.Visible = Config.mostrarDistancia
            dLbl.TextColor3 = Config.corDistancia
            dLbl.TextSize = Config.tamanhoFonteDist
            dLbl.TextStrokeTransparency = Config.mostrarSombra and 0 or 1
        end
    end
    
    for _, box in pairs(caixasESP) do
        if box.Parent then box.Color3 = Config.corCaixa end
    end
    for _, hl in pairs(highlights) do
        if hl.Parent then
            hl.FillColor = Config.corHighlight
            hl.OutlineColor = Config.corBorda
        end
    end
    for _, linha in pairs(tracersESP) do
        if linha.Parent then linha.BackgroundColor3 = Config.corTracer end
    end
    if setaLocalizador then setaLocalizador.ImageColor3 = Config.corLocalizador end
    if linhaLocalizador then linhaLocalizador.BackgroundColor3 = Config.corLocalizador end
    
    salvarConfig()
end)

-----------------------------------------------------------
--// PÁGINAS
-----------------------------------------------------------
local PaginasFrame = Instance.new("Frame")
PaginasFrame.Size = UDim2.new(1, -170, 1, -40)
PaginasFrame.Position = UDim2.new(0, 170, 0, 40)
PaginasFrame.BackgroundTransparency = 1
PaginasFrame.Parent = MainFrame

local paginas = {}
local sidebarBtns = {}

local function criarPagina(nome, titulo, subtitulo)
    local pag = Instance.new("Frame")
    pag.Size = UDim2.new(1, 0, 1, 0)
    pag.BackgroundTransparency = 1
    pag.Visible = false
    pag.Parent = PaginasFrame
    
    local topBar = Instance.new("Frame")
    topBar.Size = UDim2.new(1, 0, 0, 60)
    topBar.BackgroundTransparency = 1
    topBar.Parent = pag
    
    local topTitulo = Instance.new("TextLabel")
    topTitulo.Size = UDim2.new(1, -30, 0, 24)
    topTitulo.Position = UDim2.new(0, 20, 0, 12)
    topTitulo.BackgroundTransparency = 1
    topTitulo.Text = titulo
    topTitulo.TextColor3 = Tema.texto
    topTitulo.TextSize = 18
    topTitulo.Font = Enum.Font.GothamBold
    topTitulo.TextXAlignment = Enum.TextXAlignment.Left
    topTitulo.Parent = topBar
    
    local topSub = Instance.new("TextLabel")
    topSub.Size = UDim2.new(1, -30, 0, 16)
    topSub.Position = UDim2.new(0, 20, 0, 36)
    topSub.BackgroundTransparency = 1
    topSub.Text = subtitulo or ""
    topSub.TextColor3 = Tema.textoSub
    topSub.TextSize = 11
    topSub.Font = Enum.Font.Gotham
    topSub.TextXAlignment = Enum.TextXAlignment.Left
    topSub.Parent = topBar
    
    local scroll = Instance.new("ScrollingFrame")
    scroll.Size = UDim2.new(1, -20, 1, -70)
    scroll.Position = UDim2.new(0, 10, 0, 60)
    scroll.BackgroundTransparency = 1
    scroll.BorderSizePixel = 0
    scroll.ScrollBarThickness = 3
    scroll.ScrollBarImageColor3 = Tema.accent
    scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
    scroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
    scroll.Parent = pag
    
    local sp = Instance.new("UIPadding")
    sp.PaddingBottom = UDim.new(0, 20)
    sp.PaddingRight = UDim.new(0, 10)
    sp.Parent = scroll
    
    local layout = Instance.new("UIListLayout")
    layout.Padding = UDim.new(0, 10)
    layout.SortOrder = Enum.SortOrder.LayoutOrder
    layout.Parent = scroll
    
    paginas[nome] = {frame = pag, content = scroll}
    return scroll
end

local function trocarPagina(nome)
    for n, p in pairs(paginas) do
        p.frame.Visible = (n == nome)
    end
    for n, b in pairs(sidebarBtns) do
        local ativo = (n == nome)
        b.btn:SetAttribute("ativo", ativo)
        if ativo then
            b.btn.BackgroundColor3 = Tema.fundoHover
            b.btn.BackgroundTransparency = 0
            b.icone.TextColor3 = Tema.accent
            b.nome.TextColor3 = Tema.texto
            TweenService:Create(b.indicador, TweenInfo.new(0.2), {Size = UDim2.new(0, 3, 0, 20)}):Play()
        else
            b.btn.BackgroundTransparency = 1
            b.icone.TextColor3 = Tema.textoSub
            b.nome.TextColor3 = Tema.textoSub
            TweenService:Create(b.indicador, TweenInfo.new(0.2), {Size = UDim2.new(0, 3, 0, 0)}):Play()
        end
    end
end

local function addSidebarBtn(nome, label, icone, ordem)
    local b = criarSidebarBtn(SideNav, label, icone, ordem, function()
        trocarPagina(nome)
    end)
    sidebarBtns[nome] = b
end

-----------------------------------------------------------
--// ABA ESP
-----------------------------------------------------------
local pagESP = criarPagina("esp", "ESP", "ESP geral e individual por jogador")

local cGlobal = criarCard(pagESP, "ESP Global", "◈", 1, true)
criarToggle(cGlobal, "Ativar ESP (Todos)", "espAtivo", 1)

local cIndividual = criarCard(pagESP, "ESP Individual", "●", 2, true)
criarInput(cIndividual, "Pesquisar jogador...", 1, function(txt)
    _G.ESP_Filtro = txt
    if _G.ESP_RefreshLista then _G.ESP_RefreshLista() end
end)

local listaHolder = Instance.new("Frame")
listaHolder.Size = UDim2.new(1, 0, 0, 220)
listaHolder.BackgroundColor3 = Tema.fundoInput
listaHolder.BorderSizePixel = 0
listaHolder.LayoutOrder = 2
listaHolder.Parent = cIndividual
corner(listaHolder, 8)
stroke(listaHolder, Tema.borda, 1)

local listaScroll = Instance.new("ScrollingFrame")
listaScroll.Size = UDim2.new(1, 0, 1, 0)
listaScroll.BackgroundTransparency = 1
listaScroll.BorderSizePixel = 0
listaScroll.ScrollBarThickness = 3
listaScroll.ScrollBarImageColor3 = Tema.accent
listaScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
listaScroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
listaScroll.Parent = listaHolder

local lp = Instance.new("UIPadding")
lp.PaddingTop = UDim.new(0, 4)
lp.PaddingBottom = UDim.new(0, 4)
lp.PaddingLeft = UDim.new(0, 4)
lp.PaddingRight = UDim.new(0, 4)
lp.Parent = listaScroll

local lLayout = Instance.new("UIListLayout")
lLayout.Padding = UDim.new(0, 3)
lLayout.SortOrder = Enum.SortOrder.LayoutOrder
lLayout.Parent = listaScroll

local linhasJogadores = {}

local function refreshLista()
    for _, l in ipairs(linhasJogadores) do l:Destroy() end
    linhasJogadores = {}
    
    local filtro = _G.ESP_Filtro or ""
    
    for _, p in ipairs(Players:GetPlayers()) do
        if p == LocalPlayer and not Config.mostrarLocal then continue end
        if filtro ~= "" and not string.find(string.lower(p.Name), string.lower(filtro), 1, true) then continue end
        
        local linha = Instance.new("Frame")
        linha.Size = UDim2.new(1, 0, 0, 30)
        linha.BackgroundColor3 = Tema.fundoCardAlt
        linha.BorderSizePixel = 0
        linha.Parent = listaScroll
        corner(linha, 6)
        
        local lbl = Instance.new("TextLabel")
        lbl.Size = UDim2.new(1, -70, 1, 0)
        lbl.Position = UDim2.new(0, 10, 0, 0)
        lbl.BackgroundTransparency = 1
        lbl.Text = p.Name
        lbl.TextColor3 = Tema.texto
        lbl.TextSize = 11
        lbl.Font = Enum.Font.Gotham
        lbl.TextXAlignment = Enum.TextXAlignment.Left
        lbl.TextTruncate = Enum.TextTruncate.AtEnd
        lbl.Parent = linha
        
        local ativo = (jogadoresSelecionados[p] == true) or (Config.espAtivo and jogadoresSelecionados[p] ~= false)
        
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0, 32, 0, 20)
        btn.Position = UDim2.new(1, -40, 0.5, -10)
        btn.BackgroundColor3 = ativo and Tema.accent or Tema.fundoInput
        btn.Text = ativo and "ON" or "OFF"
        btn.TextColor3 = ativo and Tema.texto or Tema.textoSub
        btn.TextSize = 9
        btn.Font = Enum.Font.GothamBold
        btn.AutoButtonColor = false
        btn.BorderSizePixel = 0
        btn.Parent = linha
        corner(btn, 5)
        
        btn.MouseButton1Click:Connect(function()
            if ativo then
                jogadoresSelecionados[p] = false
                limparPlayer(p)
                btn.Text = "OFF"
                btn.BackgroundColor3 = Tema.fundoInput
                btn.TextColor3 = Tema.textoSub
                ativo = false
            else
                jogadoresSelecionados[p] = true
                criarESPBasico(p)
                if Config.mostrarCaixa then criarCaixa(p) end
                if Config.mostrarHighlight then criarHighlight(p) end
                if Config.mostrarTracer then criarTracer(p) end
                btn.Text = "ON"
                btn.BackgroundColor3 = Tema.accent
                btn.TextColor3 = Tema.texto
                ativo = true
            end
        end)
        
        table.insert(linhasJogadores, linha)
    end
end

_G.ESP_RefreshLista = refreshLista
refreshLista()

criarBotaoAcao(cIndividual, "🔄  Atualizar Lista", Tema.fundoCardAlt, Tema.texto, 3, function()
    refreshLista()
    notificar("Lista atualizada", Tema.accent)
end)

-----------------------------------------------------------
--// ABA LOCALIZAR
-----------------------------------------------------------
local pagLoc = criarPagina("localizar", "Localizador", "Encontra e persegue um alvo")

local cLocInput = criarCard(pagLoc, "Procurar Alvo", "◎", 1, true)
local inputFrameLoc, inputBoxLoc = criarInput(cLocInput, "Nome do jogador...", 1, nil)

local lblStatus = Instance.new("TextLabel")
lblStatus.Size = UDim2.new(1, 0, 0, 28)
lblStatus.BackgroundColor3 = Tema.fundoInput
lblStatus.Text = "Nenhum alvo selecionado"
lblStatus.TextColor3 = Tema.textoSub
lblStatus.TextSize = 11
lblStatus.Font = Enum.Font.Gotham
lblStatus.BorderSizePixel = 0
lblStatus.LayoutOrder = 2
lblStatus.Parent = cLocInput
corner(lblStatus, 6)
stroke(lblStatus, Tema.borda, 1)

criarBotaoAcao(cLocInput, "◎  Iniciar Localizador", Tema.accent, Tema.texto, 3, function()
    local nome = inputBoxLoc.Text
    if nome == "" then
        notificar("Escreve um nome!", Tema.amarelo)
        return
    end
    local encontrado
    for _, p in ipairs(Players:GetPlayers()) do
        if string.find(string.lower(p.Name), string.lower(nome), 1, true) then
            encontrado = p
            break
        end
    end
    if encontrado then
        iniciarLocalizador(encontrado)
        lblStatus.Text = "◎  A localizar: " .. encontrado.Name
        lblStatus.TextColor3 = Tema.verde
    else
        notificar("Não encontrado: " .. nome, Tema.vermelho)
    end
end)

criarBotaoAcao(cLocInput, "⚡  Auto-Alvo (mais próximo)", Tema.fundoCardAlt, Tema.accent, 4, function()
    local myChar = LocalPlayer.Character
    local myHrp = myChar and myChar:FindFirstChild("HumanoidRootPart")
    if not myHrp then return end
    
    local maisProximo, menorDist = nil, math.huge
    for _, p in ipairs(Players:GetPlayers()) do
        if p == LocalPlayer then continue end
        local char = p.Character
        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        if hrp then
            local d = (myHrp.Position - hrp.Position).Magnitude
            if d < menorDist then menorDist = d; maisProximo = p end
        end
    end
    
    if maisProximo then
        iniciarLocalizador(maisProximo)
        lblStatus.Text = "◎  Auto-alvo: " .. maisProximo.Name
        lblStatus.TextColor3 = Tema.verde
        inputBoxLoc.Text = maisProximo.Name
    else
        notificar("Nenhum alvo disponível", Tema.amarelo)
    end
end)

criarBotaoAcao(cLocInput, "■  Parar Localizador", Tema.fundoCardAlt, Tema.vermelho, 5, function()
    pararLocalizador()
    lblStatus.Text = "Nenhum alvo selecionado"
    lblStatus.TextColor3 = Tema.textoSub
    notificar("Localizador parado", Tema.vermelho)
end)

local cLocLista = criarCard(pagLoc, "Lista Rápida", "●", 2, true)
local listaLocalHolder = Instance.new("Frame")
listaLocalHolder.Size = UDim2.new(1, 0, 0, 200)
listaLocalHolder.BackgroundColor3 = Tema.fundoInput
listaLocalHolder.BorderSizePixel = 0
listaLocalHolder.LayoutOrder = 1
listaLocalHolder.Parent = cLocLista
corner(listaLocalHolder, 8)
stroke(listaLocalHolder, Tema.borda, 1)

local lLocScroll = Instance.new("ScrollingFrame")
lLocScroll.Size = UDim2.new(1, 0, 1, 0)
lLocScroll.BackgroundTransparency = 1
lLocScroll.BorderSizePixel = 0
lLocScroll.ScrollBarThickness = 3
lLocScroll.ScrollBarImageColor3 = Tema.accent
lLocScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
lLocScroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
lLocScroll.Parent = listaLocalHolder

local llp = Instance.new("UIPadding")
llp.PaddingTop = UDim.new(0, 4)
llp.PaddingBottom = UDim.new(0, 4)
llp.PaddingLeft = UDim.new(0, 4)
llp.PaddingRight = UDim.new(0, 4)
llp.Parent = lLocScroll

local lll = Instance.new("UIListLayout")
lll.Padding = UDim.new(0, 3)
lll.SortOrder = Enum.SortOrder.LayoutOrder
lll.Parent = lLocScroll

local linhasLocal = {}

local function refreshListaLocal()
    for _, l in ipairs(linhasLocal) do l:Destroy() end
    linhasLocal = {}
    
    for _, p in ipairs(Players:GetPlayers()) do
        if p == LocalPlayer and not Config.mostrarLocal then continue end
        
        local linha = Instance.new("Frame")
        linha.Size = UDim2.new(1, 0, 0, 30)
        linha.BackgroundColor3 = Tema.fundoCardAlt
        linha.BorderSizePixel = 0
        linha.Parent = lLocScroll
        corner(linha, 6)
        
        local lbl = Instance.new("TextLabel")
        lbl.Size = UDim2.new(1, -70, 1, 0)
        lbl.Position = UDim2.new(0, 10, 0, 0)
        lbl.BackgroundTransparency = 1
        lbl.Text = p.Name
        lbl.TextColor3 = Tema.texto
        lbl.TextSize = 11
        lbl.Font = Enum.Font.Gotham
        lbl.TextXAlignment = Enum.TextXAlignment.Left
        lbl.TextTruncate = Enum.TextTruncate.AtEnd
        lbl.Parent = linha
        
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0, 32, 0, 20)
        btn.Position = UDim2.new(1, -40, 0.5, -10)
        btn.BackgroundColor3 = Tema.accent
        btn.Text = "◎"
        btn.TextColor3 = Tema.texto
        btn.TextSize = 12
        btn.Font = Enum.Font.GothamBold
        btn.AutoButtonColor = false
        btn.BorderSizePixel = 0
        btn.Parent = linha
        corner(btn, 5)
        
        btn.MouseButton1Click:Connect(function()
            iniciarLocalizador(p)
            lblStatus.Text = "◎  A localizar: " .. p.Name
            lblStatus.TextColor3 = Tema.verde
            inputBoxLoc.Text = p.Name
        end)
        table.insert(linhasLocal, linha)
    end
end

_G.ESP_RefreshLocal = refreshListaLocal
refreshListaLocal()

local cLocOpt = criarCard(pagLoc, "Aparência do Localizador", "◐", 3, true)
criarColorPicker(cLocOpt, "Cor da Seta/Linha", "corLocalizador", 1)
criarSlider(cLocOpt, "Espessura da Linha", "espessuraLinhaLoc", 1, 8, 1, 2, "px")

-----------------------------------------------------------
--// ABA VISUAL
-----------------------------------------------------------
local pagVis = criarPagina("visual", "Visuais", "Personaliza os elementos do ESP")

local cElementos = criarCard(pagVis, "Elementos", "◐", 1, true)
criarToggle(cElementos, "Nome", "mostrarNome", 1)
criarToggle(cElementos, "Barra de Vida", "mostrarVida", 2)
criarToggle(cElementos, "Número da Vida", "mostrarHPnumero", 3)
criarToggle(cElementos, "Distância", "mostrarDistancia", 4)
criarToggle(cElementos, "Sempre Visível (atrás de paredes)", "sempreVisivel", 5)

local cEstilo = criarCard(pagVis, "Estilo", "◉", 2, true)
criarToggle(cEstilo, "Fundo (parte preta)", "mostrarFundo", 1)
criarToggle(cEstilo, "Borda", "mostrarBorda", 2)
criarToggle(cEstilo, "Ícone no Nome", "mostrarIcone", 3)
criarToggle(cEstilo, "Sombra no Texto", "mostrarSombra", 4)
criarToggle(cEstilo, "Bounding Box (caixa 3D)", "mostrarCaixa", 5)
criarToggle(cEstilo, "Highlight / Chams", "mostrarHighlight", 6)
criarToggle(cEstilo, "Tracer (linha do fundo)", "mostrarTracer", 7)

local cFiltros = criarCard(pagVis, "Filtros", "⚙", 3, true)
criarToggle(cFiltros, "Mostrar Mortos", "mostrarMortos", 1)
criarToggle(cFiltros, "Mostrar o Próprio Jogador", "mostrarLocal", 2)
criarSlider(cFiltros, "Distância Máxima", "distanciaMaxima", 50, 2000, 10, 3, "m")

local cTam = criarCard(pagVis, "Tamanho e Posição", "◐", 4, false)
criarSlider(cTam, "Largura do ESP", "tamanhoLargura", 100, 400, 1, 1, "px")
criarSlider(cTam, "Altura do ESP", "tamanhoAltura", 40, 150, 1, 2, "px")
criarSlider(cTam, "Offset Vertical", "offsetY", -10, 20, 0.5, 3, "")
criarSlider(cTam, "Tamanho Fonte Nome", "tamanhoFonteNome", 8, 24, 1, 4, "px")
criarSlider(cTam, "Tamanho Fonte Distância", "tamanhoFonteDist", 8, 24, 1, 5, "px")
criarSlider(cTam, "Espessura Barra Vida", "espessuraBarra", 4, 20, 1, 6, "px")
criarSlider(cTam, "Transparência do Fundo", "transparenciaFundo", 0, 1, 0.05, 7, "")

-----------------------------------------------------------
--// ABA CORES
-----------------------------------------------------------
local pagCores = criarPagina("cores", "Cores", "Ajusta as cores do ESP")

local cCorElem = criarCard(pagCores, "Elementos", "◉", 1, true)
criarColorPicker(cCorElem, "Nome", "corNome", 1)
criarColorPicker(cCorElem, "Distância", "corDistancia", 2)
criarColorPicker(cCorElem, "Fundo", "corFundo", 3)
criarColorPicker(cCorElem, "Borda", "corBorda", 4)
criarColorPicker(cCorElem, "Bounding Box", "corCaixa", 5)
criarColorPicker(cCorElem, "Highlight / Chams", "corHighlight", 6)
criarColorPicker(cCorElem, "Tracer", "corTracer", 7)

local cCorVida = criarCard(pagCores, "Barra de Vida", "◐", 2, true)
criarColorPicker(cCorVida, "Vida Cheia (>60%)", "corVidaCheia", 1)
criarColorPicker(cCorVida, "Vida Média (30-60%)", "corVidaMedia", 2)
criarColorPicker(cCorVida, "Vida Baixa (<30%)", "corVidaBaixa", 3)
criarColorPicker(cCorVida, "Fundo da Barra", "corVidaBG", 4)

-----------------------------------------------------------
--// ABA SETTINGS
-----------------------------------------------------------
local pagSet = criarPagina("settings", "Settings", "Presets, guardar e reset")

local cPresets = criarCard(pagSet, "Presets", "⚙", 1, true)

local function aplicarPreset(nome)
    if nome == "padrao" then
        Store:set("corFundo", Color3.fromRGB(0, 0, 0))
        Store:set("corNome", Color3.fromRGB(255, 255, 255))
        Store:set("corDistancia", Color3.fromRGB(255, 220, 100))
        Store:set("corVidaCheia", Color3.fromRGB(80, 220, 130))
        Store:set("corVidaMedia", Color3.fromRGB(255, 200, 90))
        Store:set("corVidaBaixa", Color3.fromRGB(240, 80, 90))
        Store:set("corVidaBG", Color3.fromRGB(50, 15, 20))
        Store:set("transparenciaFundo", 0.4)
        Store:set("tamanhoLargura", 200)
        Store:set("tamanhoAltura", 70)
        Store:set("tamanhoFonteNome", 14)
        Store:set("tamanhoFonteDist", 12)
        Store:set("mostrarFundo", true)
        Store:set("mostrarSombra", true)
        Store:set("mostrarIcone", true)
        Store:set("mostrarBorda", false)
    elseif nome == "neon" then
        Store:set("corFundo", Color3.fromRGB(20, 0, 40))
        Store:set("corNome", Color3.fromRGB(0, 255, 200))
        Store:set("corDistancia", Color3.fromRGB(255, 0, 200))
        Store:set("corVidaCheia", Color3.fromRGB(0, 255, 180))
        Store:set("corVidaMedia", Color3.fromRGB(255, 255, 0))
        Store:set("corVidaBaixa", Color3.fromRGB(255, 0, 100))
        Store:set("transparenciaFundo", 0.2)
        Store:set("mostrarBorda", true)
        Store:set("corBorda", Color3.fromRGB(255, 0, 200))
    elseif nome == "minimal" then
        Store:set("corNome", Color3.fromRGB(220, 220, 220))
        Store:set("corDistancia", Color3.fromRGB(180, 180, 180))
        Store:set("tamanhoFonteNome", 12)
        Store:set("tamanhoFonteDist", 11)
        Store:set("mostrarFundo", false)
        Store:set("mostrarSombra", true)
        Store:set("mostrarIcone", false)
        Store:set("mostrarBorda", false)
    elseif nome == "hacker" then
        Store:set("corFundo", Color3.fromRGB(0, 10, 0))
        Store:set("corNome", Color3.fromRGB(0, 255, 0))
        Store:set("corDistancia", Color3.fromRGB(0, 200, 0))
        Store:set("corVidaCheia", Color3.fromRGB(0, 255, 0))
        Store:set("corVidaMedia", Color3.fromRGB(200, 255, 0))
        Store:set("corVidaBaixa", Color3.fromRGB(255, 100, 0))
        Store:set("corVidaBG", Color3.fromRGB(0, 40, 0))
        Store:set("mostrarBorda", true)
        Store:set("corBorda", Color3.fromRGB(0, 255, 0))
        Store:set("mostrarIcone", false)
    end
    notificar("Preset aplicado: " .. nome, Tema.accent)
end

local gridPresets = Instance.new("Frame")
gridPresets.Size = UDim2.new(1, 0, 0, 76)
gridPresets.BackgroundTransparency = 1
gridPresets.LayoutOrder = 1
gridPresets.Parent = cPresets

local gridLayout = Instance.new("UIGridLayout")
gridLayout.CellSize = UDim2.new(0.5, -3, 0, 34)
gridLayout.CellPadding = UDim2.new(0, 6, 0, 6)
gridLayout.SortOrder = Enum.SortOrder.LayoutOrder
gridLayout.Parent = gridPresets

local function miniPresetBtn(texto, cor, ordem, callback)
    local b = Instance.new("TextButton")
    b.BackgroundColor3 = cor
    b.Text = texto
    b.TextColor3 = Tema.texto
    b.TextSize = 11
    b.Font = Enum.Font.GothamMedium
    b.AutoButtonColor = false
    b.BorderSizePixel = 0
    b.LayoutOrder = ordem
    b.Parent = gridPresets
    corner(b, 7)
    b.MouseButton1Click:Connect(callback)
end

miniPresetBtn("🎨  Padrão", Tema.fundoCardAlt, 1, function() aplicarPreset("padrao") end)
miniPresetBtn("🌈  Neon", Color3.fromRGB(60, 20, 90), 2, function() aplicarPreset("neon") end)
miniPresetBtn("⚫  Minimal", Tema.fundoCardAlt, 3, function() aplicarPreset("minimal") end)
miniPresetBtn("🩸  Hacker", Color3.fromRGB(10, 50, 20), 4, function() aplicarPreset("hacker") end)

local cAcoes = criarCard(pagSet, "Ações", "⚙", 2, true)
criarBotaoAcao(cAcoes, "❌  Desligar Tudo", Tema.fundoCardAlt, Tema.vermelho, 1, function()
    Store:set("espAtivo", false)
    pararLocalizador()
    limparTodos()
    jogadoresSelecionados = {}
    refreshLista()
    notificar("Tudo desligado", Tema.vermelho)
end)

criarBotaoAcao(cAcoes, "💾  Guardar Config", Tema.accent, Tema.texto, 2, function()
    if podeSalvar() then
        salvarConfig()
        notificar("Config guardada!", Tema.verde)
    else
        notificar("Executor sem suporte a save", Tema.vermelho)
    end
end)

criarBotaoAcao(cAcoes, "🔄  Reset Completo", Tema.fundoCardAlt, Tema.texto, 3, function()
    for k, v in pairs(ConfigPadrao) do Store:set(k, v) end
    Store:set("espAtivo", false)
    pararLocalizador()
    limparTodos()
    jogadoresSelecionados = {}
    refreshLista()
    notificar("Reset completo", Tema.amarelo)
end)

local cAtalho = criarCard(pagSet, "Atalho de Teclado", "⚙", 3, true)
criarToggle(cAtalho, "RightShift para abrir/fechar", "atalhoAtivo", 1)

local cSobre = criarCard(pagSet, "Sobre", "⚙", 4, false)
local sobreLbl = Instance.new("TextLabel")
sobreLbl.Size = UDim2.new(1, 0, 0, 70)
sobreLbl.BackgroundTransparency = 1
sobreLbl.Text = "ESP MENU v5.0\nDesign Premium\n\nConstruído com ❤ e Lua"
sobreLbl.TextColor3 = Tema.textoSub
sobreLbl.TextSize = 11
sobreLbl.Font = Enum.Font.Gotham
sobreLbl.TextWrapped = true
sobreLbl.LayoutOrder = 1
sobreLbl.Parent = cSobre

-----------------------------------------------------------
--// SIDEBAR BOTÕES + INICIALIZAÇÃO
-----------------------------------------------------------
addSidebarBtn("esp", "ESP", "◈", 1)
addSidebarBtn("localizar", "Localizar", "◎", 2)
addSidebarBtn("visual", "Visuais", "◐", 3)
addSidebarBtn("cores", "Cores", "◉", 4)
addSidebarBtn("settings", "Settings", "⚙", 5)

trocarPagina("esp")

-----------------------------------------------------------
--// SEARCH DA SIDEBAR (funcional — filtra lista de jogadores)
-----------------------------------------------------------
SearchBox:GetPropertyChangedSignal("Text"):Connect(function()
    _G.ESP_Filtro = SearchBox.Text
    if _G.ESP_RefreshLista then _G.ESP_RefreshLista() end
end)

-----------------------------------------------------------
--// PLAYER EVENTS
-----------------------------------------------------------
Players.PlayerAdded:Connect(function()
    task.wait(0.3)
    if _G.ESP_RefreshLista then _G.ESP_RefreshLista() end
    if _G.ESP_RefreshLocal then _G.ESP_RefreshLocal() end
end)

Players.PlayerRemoving:Connect(function(p)
    limparPlayer(p)
    jogadoresSelecionados[p] = nil
    if alvoLocalizador == p then pararLocalizador() end
    task.wait(0.1)
    if _G.ESP_RefreshLista then _G.ESP_RefreshLista() end
    if _G.ESP_RefreshLocal then _G.ESP_RefreshLocal() end
end)

-----------------------------------------------------------
--// BOTÕES TOPO + ATALHO
-----------------------------------------------------------
CloseBtn.MouseButton1Click:Connect(function()
    Store:set("espAtivo", false)
    pararLocalizador()
    limparTodos()
    salvarConfig()
    ScreenGui:Destroy()
    NotifGui:Destroy()
end)

MinBtn.MouseButton1Click:Connect(function()
    MainFrame.Visible = false
end)

UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.RightShift and Config.atalhoAtivo then
        MainFrame.Visible = not MainFrame.Visible
    end
end)

-----------------------------------------------------------
--// NOTIFICAÇÃO INICIAL
-----------------------------------------------------------
task.wait(0.4)
notificar("ESP MENU v5.0 pronto!", Tema.accent)
if carregouConfig then
    task.wait(0.2)
    notificar("Config carregada automaticamente", Tema.verde)
end
