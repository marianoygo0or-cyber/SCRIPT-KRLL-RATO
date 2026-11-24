-- K R L L X V14.1 (RATO) — INTERFACE PREMIUM APLICADA
-- Interface premium (splash, barra fina, janela com abas, FPS arrastável)
-- + lógica do script RATO (speed/pulo, fullbright, ESP do rato, TP flutuante).

-- ==========================================
-- SERVIÇOS E CONFIGURAÇÃO BASE
-- ==========================================
local P,R,U,T,L,MarketplaceService = game:GetService("Players"),game:GetService("RunService"),game:GetService("UserInputService"),game:GetService("TweenService"),game:GetService("Lighting"),game:GetService("MarketplaceService")
local lp=P.LocalPlayer
local pg=lp:WaitForChild("PlayerGui")

local Conexoes={}
local rodando=true

local function Conectar(evento,funcao)
    local conn=evento:Connect(funcao)
    table.insert(Conexoes,conn)
    return conn
end

local function LimparConexoes()
    rodando=false
    for _,c in ipairs(Conexoes) do
        if c and typeof(c)=="RBXScriptConnection" then c:Disconnect() end
    end
    Conexoes={}
end

pcall(function()
    if pg:FindFirstChild("KRLLX_UI") then pg.KRLLX_UI:Destroy() end
    if pg:FindFirstChild("KRLLX_FloatingTP") then pg.KRLLX_FloatingTP:Destroy() end
    local h=gethui and gethui()
    if h then
        if h:FindFirstChild("KRLLX_UI") then h.KRLLX_UI:Destroy() end
        if h:FindFirstChild("KRLLX_FloatingTP") then h.KRLLX_FloatingTP:Destroy() end
    end
end)

local sg=Instance.new("ScreenGui")
sg.Name="KRLLX_UI";sg.IgnoreGuiInset=true;sg.ResetOnSpawn=false;sg.DisplayOrder=999
local hui=nil
pcall(function() hui=gethui() end)
if hui and typeof(hui)=="Instance" then sg.Parent=hui else sg.Parent=pg end

local function C(c,p,f) local o=Instance.new(c);for k,v in pairs(p) do o[k]=v end;o.Parent=f;return o end
local TC=Color3.fromRGB(255,210,0);local W=Color3.new(1,1,1);local DB=Color3.fromRGB(20,20,20)

-- Estado simples do RATO
local S={Speed=16,Jump=50}
local op=false
local savedMbPos=UDim2.new(0.5,-175,0,50)
local exName=identifyexecutor and identifyexecutor() or "Mobile"

-- Nome do jogo (real via Marketplace, caindo para "RATO")
local mapName="RATO"
local gameId=game.PlaceId
if gameId==0 then gameId=game.GameId end
local pcallSuccess,gameInfo=pcall(function() return MarketplaceService:GetProductInfo(gameId) end)
if pcallSuccess and gameInfo and gameInfo.Name and gameInfo.Name~="" then
    mapName=gameInfo.Name
elseif game.Name and game.Name~="" then
    mapName=game.Name
end

local AI="rbxthumb://type=Avatar&id="..lp.UserId.."&w=420&h=420"
task.wait(0.1)

-- ==========================================
-- SPLASH SCREEN (INTERFACE PREMIUM)
-- ==========================================
local ig=C("Frame",{Size=UDim2.fromScale(1,1),BackgroundColor3=Color3.new(0,0,0),BackgroundTransparency=1,ZIndex=10000},sg)

local function makeFullGlow(rot,thickness)
    local f=C("Frame",{Size=UDim2.fromScale(1,1),Position=UDim2.fromScale(0,0),BackgroundColor3=TC,BackgroundTransparency=1,ZIndex=10001,BorderSizePixel=0},ig)
    local g=Instance.new("UIGradient")
    g.Rotation=rot
    g.Transparency=NumberSequence.new({
        NumberSequenceKeypoint.new(0,0),
        NumberSequenceKeypoint.new(thickness*0.4,0.8),
        NumberSequenceKeypoint.new(thickness,1),
        NumberSequenceKeypoint.new(1,1)
    })
    g.Parent=f
    return f
end

local gTop=makeFullGlow(90,0.04)
local gBot=makeFullGlow(270,0.04)

local lc=C("Frame",{Size=UDim2.fromOffset(300,120),Position=UDim2.new(0.5,0,1.5,0),AnchorPoint=Vector2.new(0.5,0.5),BackgroundTransparency=1,ZIndex=10002},ig)
local tv=C("TextLabel",{Size=UDim2.new(1,0,0,75),BackgroundTransparency=1,Text="KRLL X",TextColor3=TC,Font=Enum.Font.GothamBlack,TextSize=70,TextXAlignment=Enum.TextXAlignment.Center,ZIndex=10003},lc)
local tv5=C("TextLabel",{Size=UDim2.new(1,0,0,50),Position=UDim2.new(0,0,0,70),BackgroundTransparency=1,Text="V14.1 RATO",TextColor3=W,Font=Enum.Font.GothamBlack,TextSize=34,TextXAlignment=Enum.TextXAlignment.Center,ZIndex=10003},lc)

local rc=C("Frame",{Size=UDim2.fromOffset(250,50),Position=UDim2.new(0.75,0,0.5,0),AnchorPoint=Vector2.new(0.5,0.5),BackgroundTransparency=1,ZIndex=10002},ig)
local dt=C("TextLabel",{Size=UDim2.new(1,0,1,0),BackgroundTransparency=1,Text="",TextColor3=W,Font=Enum.Font.GothamBold,TextSize=20,TextXAlignment=Enum.TextXAlignment.Center,ZIndex=10003},rc)
-- ==========================================
-- FPS + JOGADORES (QUADRADO ARRASTÁVEL)
-- ==========================================
local fpsBox=C("Frame",{Size=UDim2.fromOffset(115,50),Position=UDim2.new(1,-130,0,10),BackgroundColor3=DB,BackgroundTransparency=0.3,ZIndex=10000},sg)
C("UICorner",{CornerRadius=UDim.new(0,10)},fpsBox)
C("UIStroke",{Color=TC,Thickness=1,Transparency=0.5},fpsBox)
local fpsLabel=C("TextLabel",{Size=UDim2.new(1,-10,1,0),Position=UDim2.fromOffset(10,0),BackgroundTransparency=1,Text="FPS: --\nJOGADORES: --",TextColor3=TC,Font=Enum.Font.GothamBlack,TextSize=12,TextXAlignment=Enum.TextXAlignment.Left,ZIndex=10001},fpsBox)

local fpsDrag,fpsStart,fpsOrig=false,nil,nil
Conectar(fpsBox.InputBegan,function(i)
    if i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.Touch then
        fpsDrag=true;fpsStart=i.Position;fpsOrig=fpsBox.Position
    end
end)
Conectar(U.InputChanged,function(i)
    if fpsDrag and (i.UserInputType==Enum.UserInputType.MouseMovement or i.UserInputType==Enum.UserInputType.Touch) then
        local delta=i.Position-fpsStart
        local cam=workspace.CurrentCamera;if not cam then return end
        local vps=cam.ViewportSize
        local scaleX,scaleY=fpsOrig.X.Scale,fpsOrig.Y.Scale
        local absX=(vps.X*scaleX)+fpsOrig.X.Offset+delta.X
        local absY=(vps.Y*scaleY)+fpsOrig.Y.Offset+delta.Y
        local sizeX,sizeY=fpsBox.AbsoluteSize.X,fpsBox.AbsoluteSize.Y
        if vps.X>sizeX then absX=math.clamp(absX,0,vps.X-sizeX) else absX=0 end
        if vps.Y>sizeY then absY=math.clamp(absY,0,vps.Y-sizeY) else absY=0 end
        fpsBox.Position=UDim2.new(scaleX,absX-(vps.X*scaleX),scaleY,absY-(vps.Y*scaleY))
    end
end)
Conectar(U.InputEnded,function(i)
    if i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.Touch then fpsDrag=false end
end)

local frames=0;local lastTime=os.clock()
Conectar(R.RenderStepped,function()
    if not rodando then return end
    frames=frames+1
    if os.clock()-lastTime>=1 then
        fpsLabel.Text="FPS: "..frames.."\nJOGADORES: "..#P:GetPlayers()
        frames=0;lastTime=os.clock()
    end
end)

-- ==========================================
-- MENU BAR (FINA E ELEGANTE)
-- ==========================================
local mb=C("Frame",{Size=UDim2.fromOffset(350,20),Position=savedMbPos,BackgroundColor3=DB,BackgroundTransparency=0.4,Visible=false,ZIndex=999,ClipsDescendants=true},sg)
C("UICorner",{CornerRadius=UDim.new(0,10)},mb)

local shine=C("Frame",{Size=UDim2.new(0,50,1,0),Position=UDim2.new(-1,0,0,0),BackgroundColor3=Color3.fromRGB(255,255,220),BackgroundTransparency=0.5,ZIndex=998},mb)
local shGrad=Instance.new("UIGradient")
shGrad.Transparency=NumberSequence.new({NumberSequenceKeypoint.new(0,1),NumberSequenceKeypoint.new(0.5,0),NumberSequenceKeypoint.new(1,1)})
shGrad.Parent=shine

task.spawn(function()
    while rodando and mb.Parent do
        shine.Position=UDim2.new(-1,0,0,0)
        local tw=T:Create(shine,TweenInfo.new(1.5,Enum.EasingStyle.Linear),{Position=UDim2.new(1.5,0,0,0)})
        tw:Play();tw.Completed:Wait();task.wait(2.5)
    end
end)

local mv=C("TextLabel",{Size=UDim2.fromOffset(50,20),Position=UDim2.fromOffset(15,0),BackgroundTransparency=1,Text="V14.1",TextColor3=W,Font=Enum.Font.GothamBlack,TextSize=11,TextXAlignment=Enum.TextXAlignment.Left,ZIndex=1000},mb)
local mbl=C("TextLabel",{Size=UDim2.fromScale(1,1),BackgroundTransparency=1,Text="K R L L X",TextColor3=TC,Font=Enum.Font.GothamBlack,TextSize=14,ZIndex=1000},mb)
local mp=C("TextButton",{Size=UDim2.fromOffset(40,20),Position=UDim2.new(1,-45,0,0),BackgroundTransparency=1,Text="≡",TextColor3=TC,Font=Enum.Font.GothamBlack,TextSize=18,TextXAlignment=Enum.TextXAlignment.Center,TextYAlignment=Enum.TextYAlignment.Center,ZIndex=1001},mb)

-- Arrastar Barra
local dM,sM,oM,isDr=false,nil,nil,false
Conectar(mb.InputBegan,function(i)
    if i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.Touch then
        local mouse=i.Position;local mpPos=mp.AbsolutePosition;local mpSize=mp.AbsoluteSize
        if mouse.X>=mpPos.X and mouse.X<=mpPos.X+mpSize.X and mouse.Y>=mpPos.Y and mouse.Y<=mpPos.Y+mpSize.Y then return end
        dM=true;isDr=false;sM=i.Position;oM=mb.Position
    end
end)
Conectar(U.InputChanged,function(i)
    if dM and (i.UserInputType==Enum.UserInputType.MouseMovement or i.UserInputType==Enum.UserInputType.Touch) then
        local delta=i.Position-sM
        if delta.Magnitude>5 then isDr=true end
        local cam=workspace.CurrentCamera;if not cam then return end
        local vps=cam.ViewportSize;local scaleX,scaleY=oM.X.Scale,oM.Y.Scale
        local absX=(vps.X*scaleX)+oM.X.Offset+delta.X;local absY=(vps.Y*scaleY)+oM.Y.Offset+delta.Y
        local sizeX,sizeY=mb.AbsoluteSize.X,mb.AbsoluteSize.Y
        if vps.X>sizeX then absX=math.clamp(absX,0,vps.X-sizeX) else absX=0 end
        if vps.Y>sizeY then absY=math.clamp(absY,36,vps.Y-sizeY) else absY=36 end
        mb.Position=UDim2.new(scaleX,absX-(vps.X*scaleX),scaleY,absY-(vps.Y*scaleY))
        savedMbPos=mb.Position
    end
end)
Conectar(U.InputEnded,function(i) if i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.Touch then dM=false end end)

-- ==========================================
-- JANELA PRINCIPAL
-- ==========================================
local mf=C("Frame",{Size=UDim2.fromOffset(500,280),Position=UDim2.new(.5,-250,.5,-140),BackgroundColor3=DB,BackgroundTransparency=0.35,Visible=false,ClipsDescendants=true,ZIndex=10},sg)
C("UICorner",{CornerRadius=UDim.new(0,20)},mf)
local tb=C("Frame",{Size=UDim2.new(1,0,0,50),BackgroundTransparency=1,ZIndex=11},mf)
C("TextLabel",{Size=UDim2.fromOffset(150,20),Position=UDim2.fromOffset(20,5),BackgroundTransparency=1,Text="K R L L X",TextColor3=TC,Font=Enum.Font.GothamBlack,TextSize=18,TextXAlignment=Enum.TextXAlignment.Left,ZIndex=12},tb)
C("TextLabel",{Size=UDim2.fromOffset(250,15),Position=UDim2.fromOffset(20,25),BackgroundTransparency=1,Text="MAPA: "..mapName,TextColor3=W,Font=Enum.Font.GothamBold,TextSize=10,TextXAlignment=Enum.TextXAlignment.Left,TextTruncate=Enum.TextTruncate.AtEnd,ZIndex=12},tb)
local mBtn=C("TextButton",{Size=UDim2.fromOffset(35,35),Position=UDim2.new(1,-75,0,7.5),BackgroundTransparency=1,Text="-",TextColor3=W,Font=Enum.Font.GothamBold,TextSize=24,ZIndex=12},tb)
local cBtn=C("TextButton",{Size=UDim2.fromOffset(35,35),Position=UDim2.new(1,-40,0,7.5),BackgroundTransparency=1,Text="X",TextColor3=TC,Font=Enum.Font.GothamBold,TextSize=22,ZIndex=12},tb)
local sb=C("Frame",{Size=UDim2.new(0,130,1,-75),Position=UDim2.fromOffset(15,60),BackgroundColor3=DB,BackgroundTransparency=0.5,ZIndex=11},mf)
C("UICorner",{CornerRadius=UDim.new(0,16)},sb)
local ca=C("Frame",{Size=UDim2.new(1,-165,1,-75),Position=UDim2.fromOffset(150,60),BackgroundColor3=Color3.fromRGB(15,15,15),BackgroundTransparency=0.6,ClipsDescendants=true,ZIndex=11},mf)
C("UICorner",{CornerRadius=UDim.new(0,16)},ca)

-- ==========================================
-- HELPERS UI (TOGGLES E SLIDERS)
-- ==========================================
local aS=nil
Conectar(U.InputChanged,function(i) if aS and (i.UserInputType==Enum.UserInputType.MouseMovement or i.UserInputType==Enum.UserInputType.Touch) then local sx=math.clamp((i.Position.X-aS.b.AbsolutePosition.X)/aS.b.AbsoluteSize.X,0,1);aS.f.Size=UDim2.new(sx,0,1,0);local v=math.floor(aS.m+(aS.x-aS.m)*sx);aS.l.Text=aS.t..": "..v;aS.c(v) end end)
Conectar(U.InputEnded,function(i) if i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.Touch then aS=nil end end)

local function Ctg(p,t,d,y,cb,defOn)
    local f=C("Frame",{Size=UDim2.new(1,-10,0,38),Position=UDim2.fromOffset(5,y),BackgroundTransparency=1,ZIndex=12},p)
    C("TextLabel",{Size=UDim2.new(1,-40,0,18),BackgroundTransparency=1,Text=t,TextColor3=W,Font=Enum.Font.GothamBold,TextSize=12,TextXAlignment=Enum.TextXAlignment.Left,TextTruncate=Enum.TextTruncate.AtEnd,ZIndex=12},f)
    C("TextLabel",{Size=UDim2.new(1,-40,0,15),Position=UDim2.fromOffset(0,16),BackgroundTransparency=1,Text=d,TextColor3=Color3.fromRGB(150,150,150),Font=Enum.Font.GothamMedium,TextSize=10,TextXAlignment=Enum.TextXAlignment.Left,TextTruncate=Enum.TextTruncate.AtEnd,ZIndex=12},f)
    local s=C("TextButton",{Size=UDim2.fromOffset(34,16),Position=UDim2.new(1,-34,.5,-8),BackgroundColor3=Color3.fromRGB(60,60,60),BackgroundTransparency=0.2,Text="",ZIndex=13},f)
    C("UICorner",{CornerRadius=UDim.new(1,0)},s)
    local c=C("Frame",{Size=UDim2.fromOffset(12,12),Position=UDim2.new(0,2,.5,-6),BackgroundColor3=W,ZIndex=14},s)
    C("UICorner",{CornerRadius=UDim.new(1,0)},c)
    local o=defOn or false
    if o then s.BackgroundColor3=TC;c.Position=UDim2.new(1,-14,.5,-6) end
    cb(o)
    Conectar(s.MouseButton1Click,function()
        o=not o
        T:Create(s,TweenInfo.new(.2,Enum.EasingStyle.Quart),{BackgroundColor3=o and TC or Color3.fromRGB(60,60,60)}):Play()
        T:Create(c,TweenInfo.new(.2,Enum.EasingStyle.Quart),{Position=o and UDim2.new(1,-14,.5,-6) or UDim2.new(0,2,.5,-6)}):Play()
        cb(o)
    end)
end

local function Csl(p,t,y,mn,mx,df,cb)
    local f=C("Frame",{Size=UDim2.new(1,-10,0,42),Position=UDim2.fromOffset(5,y),BackgroundTransparency=1,ZIndex=12},p)
    local l=C("TextLabel",{Size=UDim2.new(1,0,0,15),BackgroundTransparency=1,Text=t..": "..df,TextColor3=W,Font=Enum.Font.GothamBold,TextSize=12,TextXAlignment=Enum.TextXAlignment.Left,ZIndex=12},f)
    local b=C("Frame",{Size=UDim2.new(1,0,0,12),Position=UDim2.fromOffset(0,22),BackgroundColor3=Color3.fromRGB(60,60,60),BackgroundTransparency=0.2,ZIndex=13},f)
    C("UICorner",{CornerRadius=UDim.new(1,0)},b)
    local fl=C("Frame",{Size=UDim2.new((df-mn)/(mx-mn),0,1,0),BackgroundColor3=TC,ZIndex=14},b)
    C("UICorner",{CornerRadius=UDim.new(1,0)},fl)
    Conectar(b.InputBegan,function(i)
        if i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.Touch then
            aS={b=b,f=fl,l=l,m=mn,x=mx,t=t,c=cb}
            local sx=math.clamp((i.Position.X-b.AbsolutePosition.X)/b.AbsoluteSize.X,0,1)
            fl.Size=UDim2.new(sx,0,1,0)
            local v=math.floor(mn+(mx-mn)*sx)
            l.Text=t..": "..v
            cb(v)
        end
    end)
end
-- ==========================================
-- ABAS
-- ==========================================
local tabs,tw={},{}
local function Ct(t,y,isA)
    local b=C("TextButton",{Size=UDim2.new(1,-20,0,28),Position=UDim2.fromOffset(10,y),BackgroundColor3=isA and TC or Color3.fromRGB(50,50,50),BackgroundTransparency=isA and 0 or 0.4,Text=t,TextColor3=isA and DB or W,Font=Enum.Font.GothamBlack,TextSize=12,ZIndex=13},sb)
    C("UICorner",{CornerRadius=UDim.new(0,8)},b)
    return b
end
local t0=Ct("CRÉDITOS",10,true)
local t1=Ct("PRINCIPAL",42,false)
local t2=Ct("ESP",74,false)
local t3=Ct("TELEPORTE",106,false)
local t4=Ct("CONFIG",138,false)
tabs={t0,t1,t2,t3,t4}
Conectar(t0.MouseButton1Down,function() T:Create(t0,TweenInfo.new(0.1),{BackgroundTransparency=0.5}):Play() end)
Conectar(t0.MouseButton1Up,function() T:Create(t0,TweenInfo.new(0.1),{BackgroundTransparency=0}):Play() end)

local function SetT(t) for _,v in ipairs(tabs) do if tw[v] then tw[v]:Cancel() end;local a=(v==t);tw[v]=T:Create(v,TweenInfo.new(0.2,Enum.EasingStyle.Quart),{BackgroundColor3=a and TC or Color3.fromRGB(50,50,50),BackgroundTransparency=a and 0 or 0.4,TextColor3=a and DB or W});tw[v]:Play() end end

-- ==========================================
-- PÁGINAS
-- ==========================================
local p0=C("ScrollingFrame",{Size=UDim2.new(1,0,1,0),BackgroundTransparency=1,ZIndex=12,ScrollBarThickness=4,ScrollBarImageColor3=TC,CanvasSize=UDim2.new(0,0,0,430)},ca)
local p1=C("Frame",{Size=UDim2.new(1,0,1,0),BackgroundTransparency=1,Visible=false,ZIndex=12},ca)
local p2=C("Frame",{Size=UDim2.new(1,0,1,0),BackgroundTransparency=1,Visible=false,ZIndex=12},ca)
local p3=C("Frame",{Size=UDim2.new(1,0,1,0),BackgroundTransparency=1,Visible=false,ZIndex=12},ca)
local p4=C("Frame",{Size=UDim2.new(1,0,1,0),BackgroundTransparency=1,Visible=false,ZIndex=12},ca)

-- ==========================================
-- ABA 0: CRÉDITOS (PREMIUM)
-- ==========================================
local avBox=C("Frame",{Size=UDim2.fromOffset(80,80),Position=UDim2.fromOffset(15,15),BackgroundColor3=W,ZIndex=13},p0)
C("UICorner",{CornerRadius=UDim.new(0,10)},avBox)
C("ImageLabel",{Size=UDim2.new(1,-4,1,-4),Position=UDim2.fromOffset(2,2),BackgroundTransparency=1,Image=AI,ZIndex=14},avBox)

local crdBox=C("Frame",{Size=UDim2.new(1,-115,0,80),Position=UDim2.fromOffset(105,15),BackgroundColor3=Color3.fromRGB(50,50,50),BackgroundTransparency=0.2,ZIndex=13},p0)
C("UICorner",{CornerRadius=UDim.new(0,10)},crdBox)
C("TextLabel",{Size=UDim2.new(1,0,0,20),Position=UDim2.fromOffset(0,8),BackgroundTransparency=1,Text="CRÉDITOS",TextColor3=TC,Font=Enum.Font.GothamBlack,TextSize=14,ZIndex=14},crdBox)
C("TextLabel",{Size=UDim2.new(1,0,0,20),Position=UDim2.fromOffset(0,32),BackgroundTransparency=1,Text="criador: YGOR",TextColor3=W,Font=Enum.Font.GothamBold,TextSize=12,ZIndex=14},crdBox)
C("TextLabel",{Size=UDim2.new(1,0,0,20),Position=UDim2.fromOffset(0,52),BackgroundTransparency=1,Text="ajudante: Vinícius",TextColor3=W,Font=Enum.Font.GothamBold,TextSize=12,ZIndex=14},crdBox)

local mapContainer=C("Frame",{Size=UDim2.new(1,-30,0,195),Position=UDim2.fromOffset(15,105),BackgroundColor3=Color3.fromRGB(50,50,50),BackgroundTransparency=0.2,ZIndex=13},p0)
C("UICorner",{CornerRadius=UDim.new(0,10)},mapContainer)
local mapPhotoBox=C("Frame",{Size=UDim2.new(1,-20,0,130),Position=UDim2.fromOffset(10,10),BackgroundTransparency=1,ClipsDescendants=true,ZIndex=14},mapContainer)
C("UICorner",{CornerRadius=UDim.new(0,8)},mapPhotoBox)
local loadingTxt=C("TextLabel",{Size=UDim2.new(1,0,1,0),BackgroundTransparency=1,Text="Carregando...",TextColor3=Color3.fromRGB(150,150,150),Font=Enum.Font.GothamBold,TextSize=12,ZIndex=14},mapPhotoBox)
local mapImg=C("ImageLabel",{Size=UDim2.new(1,0,1,0),BackgroundTransparency=1,ScaleType=Enum.ScaleType.Crop,ZIndex=15},mapPhotoBox)
task.spawn(function()
    if pcallSuccess and gameInfo and gameInfo.IconImageAssetId and gameInfo.IconImageAssetId~=0 then
        mapImg.Image="rbxthumb://type=Asset&id="..gameInfo.IconImageAssetId.."&w=420&h=420"
    else
        mapImg.Image="rbxthumb://type=GameIcon&id="..gameId.."&w=420&h=420"
    end
    task.wait(1)
    if loadingTxt and loadingTxt.Parent then loadingTxt:Destroy() end
end)
task.delay(4,function() if loadingTxt and loadingTxt.Parent then loadingTxt:Destroy() end end)
C("TextLabel",{Size=UDim2.new(1,-20,0,25),Position=UDim2.fromOffset(10,145),BackgroundTransparency=1,Text=mapName,TextColor3=W,Font=Enum.Font.GothamBold,TextSize=14,TextXAlignment=Enum.TextXAlignment.Left,TextTruncate=Enum.TextTruncate.AtEnd,ZIndex=14},mapContainer)
local jogadoresLabel=C("TextLabel",{Size=UDim2.new(1,-20,0,20),Position=UDim2.fromOffset(10,165),BackgroundTransparency=1,Text=#P:GetPlayers().." Jogadores online",TextColor3=TC,Font=Enum.Font.GothamMedium,TextSize=12,TextXAlignment=Enum.TextXAlignment.Left,ZIndex=14},mapContainer)
local function AtualizarJogadores() jogadoresLabel.Text=#P:GetPlayers().." Jogadores online" end
Conectar(P.PlayerAdded,AtualizarJogadores)
Conectar(P.PlayerRemoving,AtualizarJogadores)

local infoB=C("Frame",{Size=UDim2.fromOffset(80,100),Position=UDim2.fromOffset(15,310),BackgroundColor3=Color3.fromRGB(50,50,50),BackgroundTransparency=0.2,ZIndex=13},p0)
C("UICorner",{CornerRadius=UDim.new(0,10)},infoB)
C("TextLabel",{Size=UDim2.new(1,-8,0,20),Position=UDim2.fromOffset(4,8),BackgroundTransparency=1,Text="Nome: "..lp.Name,TextColor3=W,Font=Enum.Font.GothamBold,TextSize=10,TextWrapped=true,ZIndex=14},infoB)
C("TextLabel",{Size=UDim2.new(1,-8,0,20),Position=UDim2.fromOffset(4,40),BackgroundTransparency=1,Text="Mapa: "..mapName,TextColor3=W,Font=Enum.Font.GothamBold,TextSize=10,TextWrapped=true,ZIndex=14},infoB)
C("TextLabel",{Size=UDim2.new(1,-8,0,20),Position=UDim2.fromOffset(4,70),BackgroundTransparency=1,Text="Exec: "..exName,TextColor3=W,Font=Enum.Font.GothamBold,TextSize=10,TextWrapped=true,ZIndex=14},infoB)

local tmpBox=C("Frame",{Size=UDim2.new(1,-115,0,100),Position=UDim2.fromOffset(105,310),BackgroundColor3=Color3.fromRGB(50,50,50),BackgroundTransparency=0.2,ZIndex=13},p0)
C("UICorner",{CornerRadius=UDim.new(0,10)},tmpBox)
C("TextLabel",{Size=UDim2.new(1,0,0,20),Position=UDim2.fromOffset(0,15),BackgroundTransparency=1,Text="TEMPO DE CRIAÇÃO",TextColor3=TC,Font=Enum.Font.GothamBlack,TextSize=12,ZIndex=14},tmpBox)
C("TextLabel",{Size=UDim2.new(1,-10,0,40),Position=UDim2.fromOffset(5,40),BackgroundTransparency=1,Text="Esse Script foi desenvolvido em 7 ou 8 meses de muito trabalho e dedicação.",TextColor3=W,Font=Enum.Font.GothamBold,TextSize=10,TextWrapped=true,ZIndex=14},tmpBox)
-- ==========================================
-- ABA 1: PRINCIPAL (SPEED / PULO / FULLBRIGHT)
-- ==========================================
local sc1=C("ScrollingFrame",{Size=UDim2.new(1,-10,1,-10),Position=UDim2.fromOffset(5,5),BackgroundTransparency=1,ScrollBarThickness=4,ScrollBarImageColor3=TC,CanvasSize=UDim2.fromOffset(0,300)},p1)
Csl(sc1,"Velocidade",0,16,150,16,function(v)
    S.Speed=v
    local c=lp.Character;local h=c and c:FindFirstChildOfClass("Humanoid")
    if h then h.WalkSpeed=v end
end)
Csl(sc1,"Pulo",50,50,150,50,function(v)
    S.Jump=v
    local c=lp.Character;local h=c and c:FindFirstChildOfClass("Humanoid")
    if h then h.UseJumpPower=true;h.JumpPower=v end
end)
local oldLight=nil
Ctg(sc1,"FULLBRIGHT","Iluminação total no mapa",110,function(v)
    if v then
        if not oldLight then oldLight={Brightness=L.Brightness,ClockTime=L.ClockTime,Ambient=L.Ambient,OutdoorAmbient=L.OutdoorAmbient} end
        L.Brightness=3;L.ClockTime=12;L.Ambient=Color3.fromRGB(255,255,255);L.OutdoorAmbient=Color3.fromRGB(255,255,255)
    elseif oldLight then
        L.Brightness=oldLight.Brightness;L.ClockTime=oldLight.ClockTime;L.Ambient=oldLight.Ambient;L.OutdoorAmbient=oldLight.OutdoorAmbient
        oldLight=nil
    end
end)

-- Reaplica speed/pulo a cada respawn
Conectar(R.Heartbeat,function()
    if not rodando then return end
    local c=lp.Character;local h=c and c:FindFirstChildOfClass("Humanoid")
    if h and h.Health>0 then
        h.WalkSpeed=S.Speed
        h.UseJumpPower=true
        h.JumpPower=S.Jump
    end
end)

-- ==========================================
-- ABA 2: ESP RATO (MICKEY)
-- ==========================================
local sc2=C("ScrollingFrame",{Size=UDim2.new(1,-10,1,-10),Position=UDim2.fromOffset(5,5),BackgroundTransparency=1,ScrollBarThickness=0,CanvasSize=UDim2.fromOffset(0,220)},p2)
local espEnabled=false
local espObjects={}
local espConnection=nil

local function clearESP()
    for _,item in ipairs(espObjects) do
        if item.highlight then item.highlight:Destroy() end
        if item.billboard then item.billboard:Destroy() end
    end
    espObjects={}
end

local function createESPForObject(obj,targetPart)
    if not obj or not obj.Parent or not targetPart then return end
    local highlight=Instance.new("Highlight")
    highlight.Name="RatoESP_Highlight";highlight.Adornee=obj;highlight.FillColor=Color3.fromRGB(255,0,0);highlight.OutlineColor=Color3.fromRGB(255,255,255);highlight.FillTransparency=0.5;highlight.Parent=obj
    local billboard=Instance.new("BillboardGui")
    billboard.Name="RatoESP_Billboard";billboard.Adornee=targetPart;billboard.Size=UDim2.fromOffset(120,40);billboard.StudsOffset=Vector3.new(0,2,0);billboard.AlwaysOnTop=true;billboard.Parent=targetPart
    local textLabel=Instance.new("TextLabel")
    textLabel.Size=UDim2.new(1,0,1,0);textLabel.BackgroundTransparency=1;textLabel.RichText=true;textLabel.Text="<font color='rgb(255,0,0)'>RATO</font>\n<font color='rgb(255,255,255)'>0 STUDS</font>";textLabel.TextStrokeTransparency=0;textLabel.TextStrokeColor3=Color3.fromRGB(0,0,0);textLabel.Font=Enum.Font.GothamBlack;textLabel.TextSize=18;textLabel.Parent=billboard
    table.insert(espObjects,{obj=obj,highlight=highlight,billboard=billboard,text=textLabel,part=targetPart})
end

local function updateESP()
    if not espEnabled then return end
    for i=#espObjects,1,-1 do
        if not espObjects[i].obj or not espObjects[i].obj.Parent or not espObjects[i].part or not espObjects[i].part.Parent then
            if espObjects[i].highlight then espObjects[i].highlight:Destroy() end
            if espObjects[i].billboard then espObjects[i].billboard:Destroy() end
            table.remove(espObjects,i)
        end
    end
    local keywords={"mickey","rato"}
    local ignoreKeywords={"ship","boat","deck","door","wall","crate","window","rope","anchor","steam","wheel","floor","ceiling","prop","decoration","railing"}
    for _,obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("Model") then
            local nameLower=string.lower(obj.Name)
            local isTarget=false;local isProp=false
            for _,kw in ipairs(keywords) do if string.find(nameLower,kw) then isTarget=true;break end end
            if isTarget then
                for _,kw in ipairs(ignoreKeywords) do if string.find(nameLower,kw) then isProp=true;break end end
                if not isProp and not P:GetPlayerFromCharacter(obj) then
                    local already=false
                    for _,e in ipairs(espObjects) do if e.obj==obj then already=true;break end end
                    if not already then
                        local targetPart=obj:FindFirstChild("HumanoidRootPart") or obj:FindFirstChild("Head") or obj:FindFirstChild("Torso") or obj.PrimaryPart or obj:FindFirstChildWhichIsA("BasePart")
                        if targetPart then createESPForObject(obj,targetPart) end
                    end
                end
            end
        end
    end
end

Ctg(sc2,"ESP RATO (MICKEY)","Marca o monstro com nome e distância",10,function(v)
    espEnabled=v
    if v then
        if not espConnection then espConnection=task.spawn(function() while espEnabled and rodando do updateESP();task.wait(1) end end) end
    else
        clearESP()
        if espConnection then task.cancel(espConnection);espConnection=nil end
    end
end)

Conectar(R.RenderStepped,function()
    if not espEnabled or not rodando then return end
    local char=lp.Character;local myHRP=char and char:FindFirstChild("HumanoidRootPart")
    if not myHRP then return end
    for _,item in ipairs(espObjects) do
        if item.part and item.part.Parent and item.text then
            local dist=math.floor((myHRP.Position-item.part.Position).Magnitude)
            item.text.Text=string.format("<font color='rgb(255,0,0)'>RATO</font>\n<font color='rgb(255,255,255)'>%d STUDS</font>",dist)
        end
    end
end)

-- ==========================================
-- ABA 3: TELEPORTE FLUTUANTE
-- ==========================================
local sc3=C("ScrollingFrame",{Size=UDim2.new(1,-10,1,-10),Position=UDim2.fromOffset(5,5),BackgroundTransparency=1,ScrollBarThickness=0,CanvasSize=UDim2.fromOffset(0,220)},p3)
local savedPos=nil
local floatingTPGui=Instance.new("ScreenGui")
floatingTPGui.Name="KRLLX_FloatingTP";floatingTPGui.ResetOnSpawn=false;floatingTPGui.Enabled=false;floatingTPGui.DisplayOrder=1000
floatingTPGui.Parent=(hui and typeof(hui)=="Instance") and hui or pg

local floatingFrame=Instance.new("Frame")
floatingFrame.Name="FloatingTPFrame";floatingFrame.Size=UDim2.fromOffset(150,130);floatingFrame.Position=UDim2.new(0,50,0.5,-50);floatingFrame.BackgroundColor3=DB;floatingFrame.BackgroundTransparency=0.2;floatingFrame.BorderSizePixel=0;floatingFrame.Active=true;floatingFrame.Parent=floatingTPGui
Instance.new("UICorner",floatingFrame).CornerRadius=UDim.new(0,10)

local btnSaveFloat=Instance.new("TextButton")
btnSaveFloat.Size=UDim2.new(1,-10,0,35);btnSaveFloat.Position=UDim2.fromOffset(5,5);btnSaveFloat.BackgroundColor3=Color3.fromRGB(50,150,255);btnSaveFloat.Text="Salvar Posição";btnSaveFloat.TextColor3=W;btnSaveFloat.Font=Enum.Font.GothamBold;btnSaveFloat.TextSize=12;btnSaveFloat.Parent=floatingFrame
Instance.new("UICorner",btnSaveFloat).CornerRadius=UDim.new(0,8)

local btnTPFloat=Instance.new("TextButton")
btnTPFloat.Size=UDim2.new(1,-10,0,35);btnTPFloat.Position=UDim2.fromOffset(5,50);btnTPFloat.BackgroundColor3=TC;btnTPFloat.Text="Teleportar";btnTPFloat.TextColor3=DB;btnTPFloat.Font=Enum.Font.GothamBold;btnTPFloat.TextSize=12;btnTPFloat.Parent=floatingFrame
Instance.new("UICorner",btnTPFloat).CornerRadius=UDim.new(0,8)

local feedbackLabel=Instance.new("TextLabel")
feedbackLabel.Size=UDim2.new(1,-10,0,30);feedbackLabel.Position=UDim2.fromOffset(5,95);feedbackLabel.BackgroundTransparency=1;feedbackLabel.Text="";feedbackLabel.TextColor3=Color3.fromRGB(100,255,100);feedbackLabel.Font=Enum.Font.GothamBold;feedbackLabel.TextSize=12;feedbackLabel.TextTransparency=1;feedbackLabel.Parent=floatingFrame

local fdrag,fstart,forigin=false,nil,nil
Conectar(floatingFrame.InputBegan,function(i) if i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.Touch then fdrag=true;fstart=i.Position;forigin=floatingFrame.Position end end)
Conectar(U.InputChanged,function(i) if fdrag and (i.UserInputType==Enum.UserInputType.MouseMovement or i.UserInputType==Enum.UserInputType.Touch) then local delta=i.Position-fstart;floatingFrame.Position=UDim2.new(forigin.X.Scale,forigin.X.Offset+delta.X,forigin.Y.Scale,forigin.Y.Offset+delta.Y) end end)
Conectar(U.InputEnded,function(i) if i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.Touch then fdrag=false end end)

local feedbackTween=nil
local function showFeedback(text,color)
    if feedbackTween then feedbackTween:Cancel() end
    feedbackLabel.Text=text;feedbackLabel.TextColor3=color;feedbackLabel.TextTransparency=0
    feedbackTween=T:Create(feedbackLabel,TweenInfo.new(1.5,Enum.EasingStyle.Quart,Enum.EasingDirection.In),{TextTransparency=1});feedbackTween:Play()
end

local function savePosition()
    local char=lp.Character
    if char and char:FindFirstChild("HumanoidRootPart") then savedPos=char.HumanoidRootPart.CFrame;showFeedback("SALVO SUA POSIÇÃO",Color3.fromRGB(100,255,100)) end
end

local function teleportToSaved()
    if savedPos then
        local char=lp.Character
        if char and char:FindFirstChild("HumanoidRootPart") then char.HumanoidRootPart.CFrame=savedPos;showFeedback("TELEPORTADO",TC) end
    else
        showFeedback("SALVE UMA POSIÇÃO",Color3.fromRGB(255,100,100))
    end
end
Conectar(btnSaveFloat.MouseButton1Click,savePosition)
Conectar(btnTPFloat.MouseButton1Click,teleportToSaved)

Ctg(sc3,"INTERFACE FLUTUANTE","Cria os botões de TP fora do painel",10,function(v) floatingTPGui.Enabled=v end)
-- ==========================================
-- ABA 4: CONFIG
-- ==========================================
local sc4=C("ScrollingFrame",{Size=UDim2.new(1,-10,1,-10),Position=UDim2.fromOffset(5,5),BackgroundTransparency=1,ScrollBarThickness=4,ScrollBarImageColor3=TC,CanvasSize=UDim2.fromOffset(0,300)},p4)
Csl(sc4,"Transparência do Menu",50,0,90,0,function(v) mf.BackgroundTransparency=v/100 end)
Ctg(sc4,"MOSTRAR FPS","Liga/desliga contador",100,function(v) fpsBox.Visible=v end,true)

-- ==========================================
-- LÓGICA DE UI (TROCA DE ABAS, TUI, DRAG)
-- ==========================================
SetT(t0)
Conectar(t0.MouseButton1Click,function() SetT(t0);p0.Visible=true;p1.Visible=false;p2.Visible=false;p3.Visible=false;p4.Visible=false end)
Conectar(t1.MouseButton1Click,function() SetT(t1);p0.Visible=false;p1.Visible=true;p2.Visible=false;p3.Visible=false;p4.Visible=false end)
Conectar(t2.MouseButton1Click,function() SetT(t2);p0.Visible=false;p1.Visible=false;p2.Visible=true;p3.Visible=false;p4.Visible=false end)
Conectar(t3.MouseButton1Click,function() SetT(t3);p0.Visible=false;p1.Visible=false;p2.Visible=false;p3.Visible=true;p4.Visible=false end)
Conectar(t4.MouseButton1Click,function() SetT(t4);p0.Visible=false;p1.Visible=false;p2.Visible=false;p3.Visible=false;p4.Visible=true end)

local function TUI()
    op=not op
    if op then
        mb.Visible=false
        mf.Visible=true
        mf.Position=UDim2.new(0.5,-250,1.2,0)
        T:Create(mf,TweenInfo.new(0.5,Enum.EasingStyle.Back,Enum.EasingDirection.Out),{Position=UDim2.new(0.5,-250,0.5,-140)}):Play()
    else
        local cT=T:Create(mf,TweenInfo.new(0.4,Enum.EasingStyle.Quart,Enum.EasingDirection.In),{Position=UDim2.new(0.5,-250,1.2,0)})
        cT:Play()
        cT.Completed:Connect(function()
            mf.Visible=false
            mb.Visible=true
            mb.Position=UDim2.new(savedMbPos.X.Scale,savedMbPos.X.Offset,savedMbPos.Y.Scale-0.05,savedMbPos.Y.Offset)
            T:Create(mb,TweenInfo.new(.4,Enum.EasingStyle.Back,Enum.EasingDirection.Out),{Position=savedMbPos}):Play()
        end)
    end
end

Conectar(mp.MouseButton1Click,function() if not isDr then TUI() end end)
Conectar(mBtn.MouseButton1Click,TUI)

-- Arrastar Janela Principal
local dr,ds2,dp=false,nil,nil
Conectar(tb.InputBegan,function(i) if i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.Touch then local pos=i.Position;if pos.X>cBtn.AbsolutePosition.X then return end;dr=true;ds2=i.Position;dp=mf.Position end end)
Conectar(U.InputChanged,function(i) if dr and (i.UserInputType==Enum.UserInputType.MouseMovement or i.UserInputType==Enum.UserInputType.Touch) then local cam=workspace.CurrentCamera;if not cam then return end;local vps=cam.ViewportSize;local scaleX,scaleY=dp.X.Scale,dp.Y.Scale;local absX=(vps.X*scaleX)+dp.X.Offset+(i.Position.X-ds2.X);local absY=(vps.Y*scaleY)+dp.Y.Offset+(i.Position.Y-ds2.Y);local sizeX,sizeY=mf.AbsoluteSize.X,mf.AbsoluteSize.Y;if vps.X>sizeX then absX=math.clamp(absX,0,vps.X-sizeX) else absX=0 end;if vps.Y>sizeY then absY=math.clamp(absY,36,vps.Y-sizeY) else absY=36 end;mf.Position=UDim2.new(scaleX,absX-(vps.X*scaleX),scaleY,absY-(vps.Y*scaleY)) end end)
Conectar(U.InputEnded,function(i) if i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.Touch then dr=false end end)

-- CONFIRM CLOSE
local cG2=C("Frame",{Size=UDim2.fromOffset(300,180),Position=UDim2.new(.5,-150,.5,-90),BackgroundColor3=DB,BackgroundTransparency=0.1,Visible=false,ZIndex=10000},sg)
C("UICorner",{CornerRadius=UDim.new(0,16)},cG2);C("UIStroke",{Color=TC,Thickness=1.5},cG2)
C("TextLabel",{Size=UDim2.new(1,-20,0,60),Position=UDim2.fromOffset(10,10),BackgroundTransparency=1,Text="FECHAR O K R L L X?",TextColor3=W,Font=Enum.Font.GothamBlack,TextSize=15,TextWrapped=true,ZIndex=10001,TextXAlignment=Enum.TextXAlignment.Center},cG2)
local yB=C("TextButton",{Size=UDim2.fromOffset(110,35),Position=UDim2.new(0,20,1,-50),BackgroundTransparency=1,Text="SIM",TextColor3=TC,Font=Enum.Font.GothamBlack,TextSize=16,ZIndex=10001},cG2)
local nB=C("TextButton",{Size=UDim2.fromOffset(110,35),Position=UDim2.new(1,-130,1,-50),BackgroundTransparency=1,Text="NÃO",TextColor3=W,Font=Enum.Font.GothamBlack,TextSize=16,ZIndex=10001},cG2)
Conectar(cBtn.MouseButton1Click,function()
    cG2.Visible=true
    cG2.Position=UDim2.new(0.5,-150,1.2,0)
    T:Create(cG2,TweenInfo.new(0.4,Enum.EasingStyle.Back,Enum.EasingDirection.Out),{Position=UDim2.new(0.5,-150,0.5,-90)}):Play()
end)
Conectar(nB.MouseButton1Click,function()
    local cT=T:Create(cG2,TweenInfo.new(0.2,Enum.EasingStyle.Quart,Enum.EasingDirection.In),{Position=UDim2.new(0.5,-150,1.2,0)})
    cT:Play();cT.Completed:Connect(function() cG2.Visible=false end)
end)
Conectar(yB.MouseButton1Click,function()
    LimparConexoes()
    pcall(function() local c=lp.Character;local h=c and c:FindFirstChildOfClass("Humanoid");if h then h.WalkSpeed=16;h.UseJumpPower=true;h.JumpPower=50 end end)
    pcall(function() if floatingTPGui then floatingTPGui:Destroy() end end)
    sg:Destroy()
end)

-- ==========================================
-- SPLASH PREMIUM + INICIALIZAÇÃO
-- ==========================================
task.spawn(function()
    local splashActive=true
    local sucesso=pcall(function()
        T:Create(ig,TweenInfo.new(1.5),{BackgroundTransparency=0.3}):Play()
        task.wait(1.5)
        T:Create(gTop,TweenInfo.new(1.5,Enum.EasingStyle.Sine),{BackgroundTransparency=0.2}):Play()
        T:Create(gBot,TweenInfo.new(1.5,Enum.EasingStyle.Sine),{BackgroundTransparency=0.2}):Play()
        task.wait(2.0)

        task.spawn(function()
            while rodando and ig.Parent and splashActive do
                T:Create(gTop,TweenInfo.new(1.5,Enum.EasingStyle.Sine),{BackgroundTransparency=0.6}):Play()
                T:Create(gBot,TweenInfo.new(1.5,Enum.EasingStyle.Sine),{BackgroundTransparency=0.6}):Play()
                task.wait(1.5)
                if not splashActive then break end
                T:Create(gTop,TweenInfo.new(1.5,Enum.EasingStyle.Sine),{BackgroundTransparency=0.2}):Play()
                T:Create(gBot,TweenInfo.new(1.5,Enum.EasingStyle.Sine),{BackgroundTransparency=0.2}):Play()
                task.wait(1.5)
            end
        end)

        local tw1=T:Create(lc,TweenInfo.new(1,Enum.EasingStyle.Quart,Enum.EasingDirection.Out),{Position=UDim2.new(0.5,0,0.5,0)})
        tw1:Play();tw1.Completed:Wait();task.wait(1.0)

        local tw2=T:Create(lc,TweenInfo.new(1.5,Enum.EasingStyle.Back,Enum.EasingDirection.Out),{Position=UDim2.new(0.25,0,0.5,0)})
        tw2:Play();task.wait(1.0)

        local msgs={mapName,"EXECUTOR: "..exName,"OBRIGADO POR USAR","BEM VINDO"}
        for _,msg in ipairs(msgs) do
            dt.Text=msg
            dt.Position=UDim2.new(0,0,1.5,0)
            dt.TextTransparency=1
            local twM=T:Create(dt,TweenInfo.new(0.8,Enum.EasingStyle.Quart,Enum.EasingDirection.Out),{Position=UDim2.new(0,0,0,0),TextTransparency=0})
            twM:Play();task.wait(2.0)
            local twM2=T:Create(dt,TweenInfo.new(0.8,Enum.EasingStyle.Quart,Enum.EasingDirection.In),{Position=UDim2.new(0,0,-0.5,0),TextTransparency=1})
            twM2:Play();task.wait(1.0)
        end

        local tw3=T:Create(lc,TweenInfo.new(1.5,Enum.EasingStyle.Back,Enum.EasingDirection.Out),{Position=UDim2.new(0.5,0,0.5,0)})
        tw3:Play();task.wait(1.5)

        splashActive=false
        task.wait(0.1)
        local fadeOutTime=1.5
        T:Create(gTop,TweenInfo.new(fadeOutTime),{BackgroundTransparency=1}):Play()
        T:Create(gBot,TweenInfo.new(fadeOutTime),{BackgroundTransparency=1}):Play()
        T:Create(ig,TweenInfo.new(fadeOutTime),{BackgroundTransparency=1}):Play()
        local tw4=T:Create(lc,TweenInfo.new(fadeOutTime,Enum.EasingStyle.Quart,Enum.EasingDirection.In),{Position=UDim2.new(0.5,0,-0.5,0)})
        tw4:Play()
        task.wait(fadeOutTime)
        ig:Destroy()
    end)
    if not sucesso then pcall(function() ig:Destroy() end) end
    TUI()
end)

print("K R L L X V14.1 (RATO) Executando 100%!")
