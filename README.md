-- StarterGui/GengarHubComplete.local.lua
-- Hub Unificado LocalScript para Roblox Studio (somente para desenvolvimento/teste)
-- Atualizado: Interface do usuário renovada com layout aprimorado, tema moderno em roxo e preto, gradientes, ícones e janela arrastável.
-- Botões mais amigáveis, alternâncias, uma fila, ScrollingFrame e painel de status. Funcionalidades (ESP, FruitESP, Fazenda, Voo, Telemetria)
-- mantido da versão anterior. Coloque no StarterGui como um LocalScript em um local que você controla.

-- AVISO: Utilize apenas em ambientes que você possui/controla. Desative os recursos de depuração (Fly) antes da produção.

-- =========================
-- CONFIG
-- =========================
Configuração local = {
    ENV = "desenvolvimento", -- defina como "produção" antes de publicar
    GengarImage = "rbxassetid://INSIRA_SEU_ASSET_ID_AQUI",
    Cores = {
        Fundo = Color3.fromRGB(12, 12, 15),
        Painel = Color3.fromRGB(30, 12, 40),
        Primário = Color3.fromRGB(85, 31, 121),
        Secundário = Color3.fromRGB(45, 18, 55),
        Accent = Color3.fromRGB(160, 100, 255),
        Texto = Color3.fromRGB(240, 240, 245),
        Silencioso = Color3.fromRGB(165, 160, 170),
    },
    Unidades = { pinos_por_metro = 3,571 },
    Logs = { enabled = true },
    Segmentação = {
        tagNames = { "farmável" },
        nomeIgual = { "FarmNPC" },
        filtros = { nível_mínimo = 1, nível_máximo = 999 },
        prioridade = { "raridade", "distância", "saúde" },
        maxQueueSize = 30,
        distânciamáxima = 200
    },
    Movimento = { tempo_limite_de_busca_de_caminho = 8, distância_de_aproximação = 3 },
    Combate = { intervalo_de_ataque = 1.0, tempo_limite_de_ataque = 12 },
    Transições = { auto_transição = true, níveis_limite = { 10, 20, 30 }, tempo_de_espera_entre_transições = 30 },
    Voar = { habilitado = verdadeiro, velocidade_máxima = 60, altura_máxima = 200 },
    Simulação = { teste_seco = verdadeiro, simular_npcs = 20 }
}

-- =========================
-- DETECÇÃO AMBIENTAL (Guarda do Roblox)
-- =========================
local InRoblox = (type(game) == "Instance" and typeof(game) == "Instance")
Jogadores locais, RunService, Workspace, StarterGui, TweenService, HttpService, Jogador local
se for no Roblox então
    Jogadores = jogo:ObterServiço("Jogadores")
    RunService = jogo:GetService("RunService")
    Espaço de trabalho = jogo:ObterServiço("Espaço de trabalho")
    StarterGui = jogo:GetService("StarterGui")
    TweenService = game:GetService("TweenService")
    local ok, svc = pcall(function() return game:GetService("HttpService") end)
    HttpService = ok e svc ou nulo
    JogadorLocal = Jogadores.JogadorLocal
fim

-- =========================
-- SERVIÇOS PÚBLICOS
-- =========================
local Utils = {}
function Utils.now_iso() return os.date("!%Y-%m-%dT%H:%M:%SZ") end
function Utils.format_float(n, decimals) decimals = decimals or 1; return string.format("%..tostring(decimals).."f", n) end
function Utils.sleep(s) if InRoblox then task.wait(s) else local t0=os.clock(); while os.clock()-t0 < s do end end end
função Utils.table_contains(tbl, val) se não tbl então retorne falso fim para _,v em ipairs(tbl) faça se v==val então retorne verdadeiro fim fim retorne falso fim
função Utils.json_encode(obj)
    se HttpService então local ok,enc = pcall(function() return HttpService:JSONEncode(obj) end) se ok então retorne enc fim fim
    -- fallback mínimo de tabela para string (não JSON completo)
    função local enc_val(v)
        local t=tipo(v)
        se t=="string" então retorne ("%q"):format(v) fim
        se t=="número" ou t=="booleano" então retorne tostring(v) fim
        se t=="tabela" então
            local isArr=true; local max=0
            para k,_ em pares(v) faça se tipo(k)~="número" então isArr=falso; interrompa fim; se k>máx então máximo=k fim fim
            se isArr então local parts={}; para i=1,max faça parts[#parts+1]=enc_val(v[i]) fim retorne "["..table.concat(parts,"").."]" fim
            local parts={}; for k,val in pairs(v) do parts[#parts+1]=("%q:%s"):format(tostring(k), enc_val(val)) end; return "{"..table.concat(parts,"").."}"
        fim
        retornar "\"<não suportado>\""
    fim
    retornar enc_val(obj)
fim

-- =========================
-- TELEMETRIA (na memória)
-- =========================
local Telemetry = { _session = nil }
função local makeSessionId() retorna tostring(os.time()).."_"..tostring(math.random(1000,9999)) fim
função Telemetria.iniciarSessão(meta)
    meta = meta ou {}
    Telemetry._session = { id = makeSessionId(), start_time = Utils.now_iso(), meta = meta, events = {} }
    Telemetry.logEvent("session.started", { meta = meta })
    retornar Telemetry._session.id
fim
função Telemetry.logEvent(evType, data)
    se não Telemetry._session então Telemetry.startSession() fim
    local entry = { ts = Utils.now_iso(), type = evType, data = data or {} }
    tabela.inserir(Telemetria._sessão.eventos, entrada)
    Se Config.Logs e Config.Logs.enabled então
        local ok,enc = pcall(function() return Utils.json_encode(entry) end)
        Se estiver tudo bem, então
            se InRoblox então print("[Telemetria]", enc) senão print(enc) fim
        fim
    fim
fim
função Telemetria.fimSessão()
    Se não for Telemetry._session, retorne o fim.
    Telemetry._session.end_time = Utils.now_iso()
    Telemetry.logEvent("session.ended", {})
    local s = Telemetry._session; Telemetry._session = nil; return s
fim
função Telemetria.exportarParaConsole()
    Se não houver Telemetry._session, retorne falso, "sem_sessão".
    local ok,enc = pcall(function() return Utils.json_encode(Telemetry._session) end)
    se estiver tudo bem, então imprima("[Sessão Completa de Telemetria]", enc); retorne verdadeiro fim
    retornar falso, "encode_failed"
fim

-- =========================
-- AMBIENTE SIMULADO (adaptador de teste)
-- =========================
local SimEnv = {}
fazer
    SimEnv._npcs = {}
    SimEnv._player = { pos = { x=0,y=0,z=0 }, level = 1, inventory = {} }
    função local spawnNpcs(contagem)
        SimEnv._npcs = {}
        para i=1,contagem faça
            SimEnv._npcs[#SimEnv._npcs+1] = {
                id = "npc_"..i, nome = "FarmNPC_"..i,
                pos = { x = math.random(-120,120), y = 0, z = math.random(10,380) },
                raridade = math.random(1,5), saúde = math.random(50,250), saúdemáxima = 250,
                nível = math.random(1,40), tipo = "mob", zona = "starter", éFarmável = true, morto = false, saqueável = false
            }
        fim
    fim
    function SimEnv.init() spawnNpcs(Config.Simulation.simulate_npcs or 20); Telemetry.logEvent("simulate.init",{count=#SimEnv._npcs}) end
    função SimEnv.getNPCs() retorna SimEnv._npcs fim
    função SimEnv.getNPC(id) para _,n em ipairs(SimEnv._npcs) faça se n.id==id então retorne n fim fim retorne nil fim
    função SimEnv.getPlayerPos() retorna SimEnv._player.pos fim
    função SimEnv.setPlayerPos(p) SimEnv._player.pos = p fim
    função SimEnv.getPlayerLevel() retorna SimEnv._player.level fim
    função SimEnv.getInventory() retorna SimEnv._player.inventory fim
    função SimEnv.getActiveQuests() retorna {} fim
    function SimEnv.moveTo(pos,opts) opts=opts or {}; local p=SimEnv.getPlayerPos(); local dx,dy,dz=pos.xp.x,pos.yp.y,pos.zp.z; local dist=math.sqrt(dx*dx+dy*dy+dz*dz); local tt=math.max(0.05,dist/30); if tt>(opts.timeout or 8) then return false,"timeout" end; local steps=math.max(1,math.floor(tt/0.03)); for i=1,steps do local frac=i/steps; SimEnv.setPlayerPos({x=p.x+dx*frac,y=p.y+dy*frac,z=p.z+dz*frac}); Utils.sleep(0.03) end; retornar verdadeiro,"chegou" fim
    function SimEnv.attack(id) local npc=SimEnv.getNPC(id); if not npc then return false,"no_npc" end; npc.health=npc.health-math.random(10,28); if npc.health<=0 then npc.dead=true; npc.lootable=true; Telemetry.logEvent("simulate.npc_dead",{id=id}) end; return true end
    function SimEnv.loot(id) local npc=SimEnv.getNPC(id); if not npc then return false end; if npc.lootable then table.insert(SimEnv._player.inventory,{item="bone",qty=1}); npc.lootable=false; Telemetry.logEvent("simulate.loot",{id=id}); return true end; return false end
    function SimEnv.simulateDamage(id,amount) local npc=SimEnv.getNPC(id); if npc and not npc.dead then npc.health=npc.health-amount; if npc.health<=0 then npc.dead=true; npc.lootable=true end end end
    function SimEnv.teleportToHub() SimEnv.setPlayerPos({x=0,y=0,z=0}); Telemetry.logEvent("simulate.teleport_hub",{}); return true end
    função SimEnv.wait(t) Utils.sleep(t) fim
    function SimEnv.addXP(xp) SimEnv._player.level = SimEnv._player.level + math.floor(xp/100); Telemetry.logEvent("simulate.xp",{xp=xp,level=SimEnv._player.level}) end
    SimEnv.init()
fim

-- Escolha o adaptador
local EnvAdapter = (Config.Simulation.dry_run and SimEnv) or (InRoblox and (function()
    retornar {
        obterNPCs = função()
            resultados locais = {}
            para _,desc em ipairs(Workspace:GetDescendants()) faça
                local okPart
                se desc:IsA("BasePart") então okPart = desc fim
                se desc:IsA("Model") e desc:FindFirstChild("Humanoid") então okPart = desc fim
                se okPart então
                    nome local = desc.Nome
                    se Utils.table_contains(Config.Targeting.nameEquals, name) ou Utils.table_contains(Config.Targeting.tagNames, (desc.GetAttribute e desc:GetAttribute(desc,"tag")) ou "") então
                        correio local
                        se desc:IsA("BasePart") então pos = desc.Position senão local part = desc:FindFirstChild("HumanoidRootPart") ou desc.PrimaryPart; se part e part:IsA("BasePart") então pos = part.Position fim fim
                        resultados[#resultados+1] = { id = tostring(desc:GetDebugId and desc:GetDebugId() or name), name = name, pos = pos, rarity = 1, health = 100, maxHealth = 100, level = 1, type = "mob", zone = "unknown", isFarmable = true }
                    fim
                fim
            fim
            retornar resultados
        fim,
        obterPosiçãoDoJogador = função()
            local char = LocalPlayer e LocalPlayer.Character
            local hrp = char e (char:FindFirstChild("HumanoidRootPart") ou char.PrimaryPart)
            Se hrp então retorne { x = hrp.Position.X, y = hrp.Position.Y, z = hrp.Position.Z } fim
            retornar { x=0,y=0,z=0 }
        fim,
        moveTo = function(pos,opts) local char = LocalPlayer and LocalPlayer.Character; local hrp = char and (char:FindFirstChild("HumanoidRootPart") or char.PrimaryPart); if hrp and pos then hrp.CFrame = CFrame.new(pos.x,pos.y,pos.z); return true,"teleported" end; return false,"no_hrp" end,
        ataque = função(id) retorna falso,"não_implementado" fim,
        obterNPC = função(id) retorna nulo fim,
        loot = função(id) retorna falso fim,
        teleportToHub = function() if LocalPlayer and LocalPlayer.Character then local hrp = LocalPlayer.Character:FindFirstChild("HumanoidRootPart") or LocalPlayer.Character.PrimaryPart; if hrp then hrp.CFrame = CFrame.new(0,5,0) end end end,
        aguardar = função(t) tarefa.aguardar(t) fim
    }
fim)() ou SimEnv)

-- =========================
-- ALVO
-- =========================
Direcionamento local = {}
fazer
    função local buildList()
        local raw = EnvAdapter.getNPCs()
        local playerPos = EnvAdapter.getPlayerPos()
        lista local = {}
        para _, n em ipairs(raw) faça
            se n e (n.isFarmable ou falso) então
                se n.level e (n.level < Config.Targeting.filters.level_min ou n.level > Config.Targeting.filters.level_max) então
                outro
                    local pos = n.pos
                    distância local = matemática.enorme
                    se positivo então
                        se typeof e typeof(pos) == "Vector3" então
                            local dx,dy,dz = pos.X - (playerPos.x ou 0), pos.Y - (playerPos.y ou 0), pos.Z - (playerPos.z ou 0)
                            dist = math.sqrt(dx*dx + dy*dy + dz*dz)
                        senão se type(pos) == "table" e pos.x então
                            local dx,dy,dz = pos.x - (playerPos.x ou 0), pos.y - (playerPos.y ou 0), pos.z - (playerPos.z ou 0)
                            dist = math.sqrt(dx*dx + dy*dy + dz*dz)
                        fim
                    fim
                    n._distância = dist
                    lista[#lista+1] = n
                fim
            fim
        fim
        tabela.ordenar(lista, função(a,b)
            para _, chave em ipairs(Config.Targeting.priority) faça
                se a chave for igual a "raridade", então
                    local av,bv=(a.raridade ou 0),(b.raridade ou 0); se av~=bv então retorne av>bv fim
                senão se chave == "distância" então
                    se (a._distance ou math.huge) ~= (b._distance ou math.huge) então retorne (a._distance ou math.huge) < (b._distance ou math.huge) fim
                senão se a chave for igual a "saúde" então
                    Se (a.saúde ou 0) ~= (b.saúde ou 0) então retorne (a.saúde ou 0) < (b.saúde ou 0) fim
                fim
            fim
            retornar falso
        fim)
        enquanto #lista > Config.Targeting.maxQueueSize faça tabela.remova(lista) fim
        lista de retorno
    fim
    function Targeting.buildQueue() local q=buildList(); Telemetry.logEvent("targeting.queue_built",{size=#q}); return q end
fim

-- =========================
-- MOVIMENTO E COMBATE
-- =========================
Movimento local = {}
function Movement.moveTo(pos,opts) opts=opts or {}; local timeout=opts.timeout or Config.Movement.pathfinding_timeout; Telemetry.logEvent("movement.move_start",{pos=pos,timeout=timeout}); if EnvAdapter.moveTo then local ok,reason=EnvAdapter.moveTo(pos,{timeout=timeout,approach_dist=opts.approach_distance}); Telemetry.logEvent("movement.move_end",{pos=pos,ok=ok,reason=reason}); return ok,reason end; return false,"no_move_adapter" end

local Combat = {}
função Combat.attackTarget(alvo,opts)
    opts = opts ou {}
    local maxTimeout = opts.timeout ou Config.Combat.attack_timeout
    local attackInterval = Config.Combat.attack_interval ou 1.0
    local startT = os.time()
    Telemetry.logEvent("combat.attack_start", { id = target.id, name = target.name })
    local pos = alvo.pos
    local ok, motivo = Movement.moveTo(pos, { timeout = Config.Movement.pathfinding_timeout })
    Se não estiver tudo bem, então Telemetry.logEvent("combat.attack_failed_approach",{id=target.id,reason=reason}); retorne { success=false, reason="approach_failed" } fim
    enquanto (os.time() - startT) < maxTimeout faça
        local npc = EnvAdapter.getNPC e EnvAdapter.getNPC(target.id) ou target
        se não for npc ou npc.morto então
            Telemetry.logEvent("combat.target_dead",{id=target.id})
            Se EnvAdapter.loot e npc e npc.lootable então pcall(EnvAdapter.loot, target.id) fim
            retornar { sucesso = verdadeiro, motivo = "alvo_morto" }
        fim
        if EnvAdapter.attack then pcall(function() EnvAdapter.attack(target.id) end); Telemetry.logEvent("combat.attack_tick",{id=target.id})
        senão se EnvAdapter.simulateDamage então pcall(EnvAdapter.simulateDamage, target.id, 18) fim; Telemetry.logEvent("combat.attack_simulated",{id=target.id}) fim
        EnvAdapter.wait(intervaloDeAtaque)
    fim
    Telemetry.logEvent("combat.attack_timeout",{id=target.id})
    retornar { sucesso=falso, motivo="tempo limite" }
fim

-- =========================
-- FLY DEBUG (restrito)
-- =========================
local Fly = { ativo = falso }
function Fly.isAllowed() return (Config.ENV == "development") and Config.Fly.enabled end
function Fly.start() if not Fly.isAllowed() then return false,"not_allowed" end; if Fly.active then return true,"already" end; Fly.active=true; Telemetry.logEvent("fly.start",{user=(InRoblox and LocalPlayer and LocalPlayer.Name) or "local", ts=Utils.now_iso()}); return true end
function Fly.stop() if not Fly.active then return false,"not_active" end; Fly.active=false; Telemetry.logEvent("fly.stop",{ts=Utils.now_iso()}); return true end
function Fly.toggle() if Fly.active then return Fly.stop() else return Fly.start() end end
função Fly.isActive() retorna Fly.active fim

-- =========================
-- ESP
-- =========================
local ESP = {}
fazer
    ESP.humanoidHighlights = {}
    ESP.fruitTracked = {}
    ESP._fruitHeartbeat = nulo

    função local makeHighlightForAdornee(adornee, fillColor, fillTransparency, outlineColor)
        Se não for InRoblox ou não for adornee, retorne nulo.
        local playerGui = LocalPlayer:WaitForChild("PlayerGui")
        local hl = Instance.new("Highlight")
        hl.Name = "GengarHub_Highlight"
        hl.Adornee = adornee
        hl.FillColor = fillColor ou Color3.fromRGB(255,80,80)
        hl.FillTransparency = fillTransparency ou 0,65
        hl.OutlineColor = outlineColor ou Color3.fromRGB(180,20,20)
        hl.OutlineTransparency = 0
        hl.Parent = playerGui
        retornar hl
    fim

    função local adicionarDestaqueHumanoide(modelo)
        Se não for InRoblox ou não for modelo ou ESP.humanoidHighlights[modelo], então retorne.
        local adornee = model:FindFirstChild("HumanoidRootPart") ou model.PrimaryPart
        se não adornee então para _,c em ipairs(model:GetChildren()) faça se c:IsA("BasePart") então adornee = c; interrompa fim fim fim
        se não for adornado, retorne ao fim
        local hl = makeHighlightForAdornee(adornee, Color3.fromRGB(255,80,80), 0.65, Color3.fromRGB(180,20,20))
        ESP.humanoidHighlights[model] = hl
    fim

    função ESP.enableHumanoidESP()
        Se não for InRoblox, retorne ao fim.
        Telemetry.logEvent("esp.humanoid_enable", {})
        para _,p em ipairs(Players:GetPlayers()) faça se p.Character então adicioneHumanoidHighlight(p.Character) fim; p.CharacterAdded:Connect(function(chr) adicioneHumanoidHighlight(chr) fim) fim
        para _,desc em ipairs(Workspace:GetDescendants()) faça se desc:IsA("Model") e desc:FindFirstChildOfClass("Humanoid") então adicioneHumanoidHighlight(desc) fim fim
        Workspace.DescendantAdded:Connect(function(desc) if desc:IsA("Model") and desc:FindFirstChildOfClass("Humanoid") then addHumanoidHighlight(desc) end end)
    fim

    função ESP.desativarESPHumanoide()
        Telemetry.logEvent("esp.humanoid_disable", {})
        para cada modelo e hl em pares(ESP.humanoidHighlights) faça se hl e hl.Parent então hl:Destroy() fim; ESP.humanoidHighlights[modelo] = nil fim
    fim

    -- Fruta ESP
    função local findAdornee(obj)
        Se não for obj, retorne nulo.
        se obj:IsA e obj:IsA("BasePart") então retorne obj fim
        se obj:IsA e obj:IsA("Modelo") então
            Se obj.PrimaryPart e obj.PrimaryPart:IsA("BasePart") então retorne obj.PrimaryPart fim
            local hrp = obj:FindFirstChild("HumanoidRootPart")
            se hrp e hrp:IsA("BasePart") então retorne hrp fim
            para _,c em ipairs(obj:GetChildren()) faça se c:IsA("BasePart") então retorne c fim fim
        fim
        retornar nulo
    fim

    função local criarVisuaisDeFrutas(adornee, displayName)
        Se não for InRoblox ou não for adornee, retorne nulo.
        local playerGui = LocalPlayer:WaitForChild("PlayerGui")
        local hl = Instance.new("Highlight")
        hl.Name = "FruitESP_Highlight"
        hl.Adornee = adornee
        hl.FillColor = Color3.fromRGB(165,95,255)
        hl.FillTransparency = 0,5
        hl.OutlineColor = Color3.fromRGB(110,40,200)
        hl.OutlineTransparency = 0
        hl.Parent = playerGui

        outdoor local = Instance.new("BillboardGui")
        billboard.Name = "FruitESP_Billboard"
        outdoor.Adornee = adornee
        billboard.Size = UDim2.new(0,180,0,54)
        billboard.AlwaysOnTop = true
        billboard.StudsOffset = Vector3.new(0,2.8,0)
        billboard.Parent = playerGui

        local frame = Instance.new("Frame")
        frame.Size = UDim2.new(1,0,1,0)
        frame.BackgroundTransparency = 1
        quadro.Pai = outdoor

        rótulo local = Instance.new("TextLabel")
        label.Name = "FruitESP_Label"
        label.Size = UDim2.new(1,0,1,0)
        label.BackgroundTransparency = 1
        label.TextColor3 = Color3.fromRGB(230.210.255)
        rótulo.TextStrokeTransparency = 0,6
        label.TextScaled = true
        label.Font = Enum.Font.GothamSemibold
        label.Text = tostring(displayName ou "")
        rótulo.Pai = quadro

        retornar { destaque = hl, outdoor = outdoor, rótulo = rótulo }
    fim

    função ESP.enableFruitESP()
        Se não for InRoblox, retorne ao fim.
        Telemetry.logEvent("esp.fruit_enable", {})
        para _,desc em ipairs(Workspace:GetDescendants()) faça
            se desc.Name == "Fruit" ou desc.Name == "Fruta" então
                local adornee = findAdornee(desc)
                se adornee e não ESP.fruitTracked[desc] então
                    local vs = createFruitVisuals(adornee, desc.Name)
                    ESP.fruitTracked[desc] = { adornee = adornee, visuais = vs }
                fim
            fim
        fim
        Workspace.DescendantAdded:Connect(function(desc)
            se desc.Name == "Fruit" ou desc.Name == "Fruta" então
                Utils.sleep(0.03)
                local adornee = findAdornee(desc)
                se adornee e não ESP.fruitTracked[desc] então
                    local vs = createFruitVisuals(adornee, desc.Name)
                    ESP.fruitTracked[desc] = { adornee = adornee, visuais = vs }
                fim
            fim
        fim)
        se não for ESP._fruitHeartbeat e RunService então
            ESP._fruitHeartbeat = RunService.RenderStepped:Connect(função()
                local hrp = LocalPlayer.Character e (LocalPlayer.Character:FindFirstChild("HumanoidRootPart") ou LocalPlayer.Character.PrimaryPart)
                local hrpPos = hrp e hrp.Position
                para obj,info em pares(ESP.fruitTracked) faça
                    se não obj.Parent ou não info.adornee ou não info.adornee.Parent então
                        se info.visuals então
                            se info.visuals.highlight então info.visuals.highlight:Destroy() fim
                            se info.visuals.billboard então info.visuals.billboard:Destroy() fim
                        fim
                        ESP.fruitTracked[obj] = nil
                    outro
                        local ok,pos = pcall(function() return info.adornee.Position end)
                        se ok e pos e hrpPos então
                            local distStuds = (pos - hrpPos).Magnitude
                            local distMeters = distStuds / Config.Units.studs_per_meter
                            se info.visuals e info.visuals.label então
                                info.visuals.label.Text = tostring(obj.Name) .. "\n" .. Utils.format_float(distMeters,1) .. " m"
                            fim
                        fim
                    fim
                fim
            fim)
        fim
    fim

    função ESP.desativarFrutasESP()
        Telemetry.logEvent("esp.fruit_disable", {})
        para obj,info em pares(ESP.fruitTracked) faça
            se info.visuals então
                se info.visuals.highlight e info.visuals.highlight.Parent então info.visuals.highlight:Destroy() fim
                se info.visuals.billboard e info.visuals.billboard.Parent então info.visuals.billboard:Destroy() fim
            fim
            ESP.fruitTracked[obj] = nil
        fim
        Se ESP._fruitHeartbeat então ESP._fruitHeartbeat:Desconectar(); ESP._fruitHeartbeat = nulo fim
    fim
fim

-- =========================
-- ORQUESTRADOR (simples)
-- =========================
local Orchestrator = { running = false, paused = false, queue = {}, currentTarget = nil, stats = { defeat = 0, xp = 0 }, _thread = nil }

função Orchestrator.refreshQueue()
    local q = Targeting.buildQueue()
    Orchestrator.queue = q
    Telemetry.logEvent("orchestrator.queue_refreshed",{size=#q})
fim

função Orchestrator.start()
    Se Orchestrator estiver em execução, retorne o fim.
    Orchestrator.running = true; Orchestrator.paused = false
    Telemetry.logEvent("orchestrator.start",{mode = Config.Simulation.dry_run and "dry" or "real"})
    Orchestrator._thread = corrotina.create(function()
        enquanto Orchestrator.running faça
            Se Orchestrator.paused então EnvAdapter.wait(0.5)
            outro
                se #Orchestrator.queue == 0 então Orchestrator.refreshQueue() fim
                alvo local = Orchestrator.queue[1]
                se não for alvo, então EnvAdapter.wait(1)
                outro
                    Orchestrator.currentTarget = alvo
                    Telemetry.logEvent("orchestrator.target_acquired",{id=target.id})
                    local res = Combat.attackTarget(alvo)
                    se res e res.success então
                        Orchestrator.stats.defeated = Orchestrator.stats.defeated + 1
                        se Config.Simulation.dry_run e EnvAdapter.addXP então EnvAdapter.addXP(120) fim
                        tabela.remover(Orchestrator.queue,1)
                        Telemetry.logEvent("orchestrator.target_defeated",{id=target.id})
                    outro
                        Telemetry.logEvent("orchestrator.target_failed",{id=target.id,reason=res e res.reason})
                        tabela.remover(Orchestrator.queue,1)
                    fim
                    se Config.Transitions.auto_transition então
                        nível local = EnvAdapter.getPlayerLevel e EnvAdapter.getPlayerLevel() ou 0
                        para _,th em ipairs(Config.Transitions.level_thresholds) faça
                            se nível >= th então
                                Telemetry.logEvent("orchestrator.transition_trigger",{level=lvl,threshold=th})
                                se EnvAdapter.teleportToHub então pcall(EnvAdapter.teleportToHub) fim
                                quebrar
                            fim
                        fim
                    fim
                fim
            fim
            EnvAdapter.wait(0.1)
        fim
        Telemetry.logEvent("orchestrator.stopped",{})
    fim)
    corrotina.resume(Orchestrator._thread)
fim

function Orchestrator.pause() Orchestrator.paused = true; Telemetry.logEvent("orchestrator.pause",{}) end
function Orchestrator.stop() Orchestrator.running = false; Telemetry.logEvent("orchestrator.stop",{}) end

-- =========================
-- INTERFACE DE USUÁRIO BONITA (aprimorada)
-- =========================
se for no Roblox então
    local playerGui = LocalPlayer:WaitForChild("PlayerGui")
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "GengarHubGui"
    screenGui.ResetOnSpawn = false
    screenGui.Parent = playerGui

    função local nova(nomeDaClasse, propriedades)
        local obj = Instance.new(className)
        se props então para k,v em pares(props) faça obj[k] = v fim fim
        retornar obj
    fim

    -- Janela principal
    local main = new("Frame", {
        Nome = "JanelaPrincipal",
        Pai = screenGui,
        PontoAncora = Vector2.new(0.5,0.5),
        Posição = UDim2.new(0.5,0,0.45,0),
        Tamanho = UDim2.new(0,720,0,420),
        BackgroundColor3 = Config.Colors.Panel,
        BackgroundTransparency = 0,
        Ativo = verdadeiro
    })
    novo("UICorner", { Parent = main, CornerRadius = UDim.new(0,14) })
    novo("UIStroke", { Parent = main, Color = Config.Colors.Primary, Thickness = 2, Transparency = 0 })

    -- Sobreposição de gradiente sutil
    local grad = new("UIGradient", { Parent = main, Color = ColorSequence.new({ ColorSequenceKeypoint.new(0, Config.Colors.Panel), ColorSequenceKeypoint.new(1, Color3.fromRGB(20,10,40)) }), Rotation = 0, Transparency = NumberSequence.new(0.05) })

    -- Barra superior (arrastável)
    local topBar = new("Frame", { Parent = main, Name = "TopBar", Size = UDim2.new(1,0,0,52), BackgroundTransparency = 1 })
    local title = new("TextLabel", { Parent = topBar, Text = "GENGAR HUB — Dev", TextColor3 = Config.Colors.Text, Font = Enum.Font.GothamBold, TextSize = 20, BackgroundTransparency = 1, Position = UDim2.new(0,16,0,10), Size = UDim2.new(0.6,0,0,32), TextXAlignment = Enum.TextXAlignment.Left })
    local subtitle = new("TextLabel", { Parent = topBar, Text = "Ferramentas de Desenvolvimento", TextColor3 = Config.Colors.Muted, Font = Enum.Font.Gotham, TextSize = 14, BackgroundTransparency = 1, Position = UDim2.new(0,16,0,30), Size = UDim2.new(0.6,0,0,16), TextXAligment = Enum.TextXAligment.Left })

    -- Fechar / Minimizar
    local btnClose = new("ImageButton", { Parent = topBar, Name = "BtnClose", Size = UDim2.new(0,36,0,28), Position = UDim2.new(1,-46,0,12), BackgroundTransparency = 0.6, BackgroundColor3 = Config.Colors.Secondary, Image = "", AutoButtonColor = true })
    novo("UICorner", { Parent = btnClose, CornerRadius = UDim.new(0,8) })
    local closeX = new("TextLabel", { Parent = btnClose, Text = "âœ•", TextColor3 = Color3.fromRGB(240,240,240), BackgroundTransparency = 1, Font = Enum.Font.GothamBold, TextSize = 18 })
    local btnMin = new("ImageButton", { Parent = topBar, Name = "BtnMin", Size = UDim2.new(0,36,0,28), Position = UDim2.new(1,-92,0,12), BackgroundTransparency = 0.6, BackgroundColor3 = Config.Colors.Secondary, Image = "", AutoButtonColor = true })
    novo("UICorner", { Parent = btnMin, CornerRadius = UDim.new(0,8) })
    novo("TextLabel", { Parent = btnMin, Text = "â--", TextColor3 = Color3.fromRGB(240,240,240), BackgroundTransparency = 1, Font = Enum.Font.GothamBold, TextSize = 18 })

    -- Coluna da esquerda: Imagem e estatísticas do Gengar
    local leftCol = new("Frame", { Parent = main, Position = UDim2.new(0,18,0,72), Size = UDim2.new(0,240,1,-88), BackgroundColor3 = Config.Colors.Secondary })
    novo("UICorner", { Parent = leftCol, CornerRadius = UDim.new(0,12) })
    local imgFrame = new("Frame", { Parent = leftCol, Position = UDim2.new(0.5,-100,0,12), Size = UDim2.new(0,200,0,200), BackgroundTransparency = 1 })
    local gengarImage = new("ImageLabel", { Parent = imgFrame, AnchorPoint = Vector2.new(0.5,0.5), Position = UDim2.new(0.5,0,0.5,0), Size = UDim2.new(1,0,1,0), BackgroundTransparency = 1, Image = Config.GengarImage, ScaleType = Enum.ScaleType.Fit })
    novo("UICorner", { Parent = gengarImage, CornerRadius = UDim.new(0,12) })
    -- brilho por trás da imagem
    local glow = new("ImageLabel", { Parent = leftCol, Position = UDim2.new(0.5,-130,0,26), Size = UDim2.new(0,260,0,260), BackgroundTransparency = 1, Image = "", ZIndex = 0 })
    glow.ImageTransparency = 1 -- marcador de posição, mantenha invisível se não houver recurso

    -- Caixa de estatísticas
    local statsBox = new("Frame", { Parent = leftCol, Position = UDim2.new(0,12,0,220), Size = UDim2.new(1,-24,0,88), BackgroundColor3 = Color3.fromRGB(18,12,28) })
    novo("UICorner", { Parent = statsBox, CornerRadius = UDim.new(0,10) })
    local defeatdLabel = new("TextLabel", { Parent = statsBox, Position = UDim2.new(0,12,0,8), Size = UDim2.new(1,-24,0,28), BackgroundTransparency = 1, Text = "Derrotados: 0", TextColor3 = Config.Colors.Text, Font = Enum.Font.GothamSemibold, TextSize = 16, TextXAlignment = Enum.TextXAlignment.Left })
    local xpLabel = new("TextLabel", { Parent = statsBox, Position = UDim2.new(0,12,0,38), Size = UDim2.new(1,-24,0,28), BackgroundTransparency = 1, Text = "XP: 0", TextColor3 = Config.Colors.Muted, Font = Enum.Font.Gotham, TextSize = 14, TextXAligment = Enum.TextXAligment.Left })

    -- Coluna da direita: Controles, botões de alternância, configurações, fila
    local rightCol = new("Frame", { Parent = main, Position = UDim2.new(0,276,0,72), Size = UDim2.new(1,-294,1,-88), BackgroundTransparency = 1 })
    -- Painel de controles
    controles locais = novo("Frame", { Pai = rightCol, Posição = UDim2.new(0,0,0,0), Tamanho = UDim2.new(1,0,0,124), CorDeFundo3 = Color3.fromRGB(18,12,28) })
    novo("UICorner", { Parent = controls, CornerRadius = UDim.new(0,10) })
    local ctrlLayout = new("UIListLayout", { Parent = controls, FillDirection = Enum.FillDirection.Horizontal, HorizontalAlignment = Enum.HorizontalAlignment.Left, Padding = UDim.new(0,8) })
    local ctrlPadding = new("UIPadding", { Parent = controls, PaddingLeft = UDim.new(0,12), PaddingTop = UDim.new(0,12), PaddingBottom = UDim.new(0,12) })

    função local criarBotãoCtrl(texto, largura)
        local btn = new("TextButton", { Parent = controls, Size = UDim2.new(0,width or 120,0,40), BackgroundColor3 = Config.Colors.Primary, Text = text, TextColor3 = Color3.fromRGB(255,255,255), Font = Enum.Font.GothamSemibold, TextSize = 15, AutoButtonColor = false })
        novo("UICorner", { Parent = btn, CornerRadius = UDim.new(0,8) })
        local grad = new("UIGradient", { Parent = btn, Color = ColorSequence.new({ ColorSequenceKeypoint.new(0, Config.Colors.Primary), ColorSequenceKeypoint.new(1, Config.Colors.Accent) }) })
        novo("UIStroke", { Parent = btn, Color = Color3.fromRGB(0,0,0), Thickness = 1, Transparency = 0.8 })
        botão de retorno
    fim

    local startBtn = createCtrlButton("Iniciar", 110)
    local pauseBtn = createCtrlButton("Pause", 110)
    local stopBtn = createCtrlButton("Parar", 110)
    local spacer = new("Frame", { Parent = controls, Size = UDim2.new(1, -360, 1, 0), BackgroundTransparency = 1 })

    -- Painel de alternância (abaixo dos controles)
    local toggles = new("Frame", { Parent = rightCol, Position = UDim2.new(0,0,0,136), Size = UDim2.new(1,0,0,86), BackgroundColor3 = Color3.fromRGB(18,12,28) })
    novo("UICorner", { Parent = toggles, CornerRadius = UDim.new(0,10) })
    local tLayout = new("UIListLayout", { Parent = toggles, FillDirection = Enum.FillDirection.Horizontal, Padding = UDim.new(0,8) })
    novo("UIPadding", { Parent = toggles, PaddingLeft = UDim.new(0,12), PaddingTop = UDim.new(0,12), PaddingBottom = UDim.new(0,12) })

    função local criarAlternância(texto)
        local wrap = new("Frame", { Parent = toggles, Size = UDim2.new(0,160,0,56), BackgroundTransparency = 1 })
        local btn = new("TextButton", { Parent = wrap, Size = UDim2.new(1,0,0,44), BackgroundColor3 = Config.Colors.Secondary, Text = text, TextColor3 = Config.Colors.Text, Font = Enum.Font.GothamSemibold, TextSize = 14, AutoButtonColor = false })
        novo("UICorner", { Parent = btn, CornerRadius = UDim.new(0,8) })
        indicador local = novo("Frame", { Pai = btn, Tamanho = UDim2.new(0,18,0,18), Posição = UDim2.new(1,-28,0.5,-9), CorDeFundo3 = Color3.fromRGB(120,120,120) })
        novo("UICorner", { Parent = indicator, CornerRadius = UDim.new(0,9) })
        botão de retorno, indicador
    fim

    local espBtn, espInd = createToggle("ESP Humanoids")
    local fruitBtn, fruitInd = createToggle("Fruit ESP")
    local flyBtn, flyInd = createToggle("Depurar voo")

    -- Painel de configurações + fila
    local lowerPanel = new("Frame", { Parent = rightCol, Position = UDim2.new(0,0,0,236), Size = UDim2.new(1,0,1,-236), BackgroundColor3 = Color3.fromRGB(10,8,14) })
    novo("UICorner", { Parent = lowerPanel, CornerRadius = UDim.new(0,10) })
    -- Lado esquerdo: configurações
    configurações locais = novo("Frame", { Pai = painelInferior, Posição = UDim2.novo(0,12,0,12), Tamanho = UDim2.novo(0,320,1,-24), TransparênciaDeFundo = 1 })
    local sTitle = new("TextLabel", { Parent = settings, Text = "Configurações", BackgroundTransparency = 1, TextColor3 = Config.Colors.Muted, Font = Enum.Font.GothamBold, TextSize = 14, Position = UDim2.new(0,0,0,0) })
    local pLabel = new("TextLabel", { Parent = settings, Text = "Prioridade: raridade / distância / saúde", BackgroundTransparency = 1, TextColor3 = Config.Colors.Text, Font = Enum.Font.Gotham, TextSize = 14, Position = UDim2.new(0,0,0,28) })
    local lvlLabel = new("TextLabel", { Parent = settings, Text = "Nível mín./máx.: "..tostring(Config.Targeting.filters.level_min).."/"..tostring(Config.Targeting.filters.level_max), BackgroundTransparency = 1, TextColor3 = Config.Colors.Text, Font = Enum.Font.Gotham, TextSize = 14, Position = UDim2.new(0,0,0,56) })
    local refreshBtn = createCtrlButton("Atualizar fila", 140); refreshBtn.Parent = settings; refreshBtn.Position = UDim2.new(0,0,0,88)

    -- Lado direito: fila e detalhes
    local queueFrame = new("Frame", { Parent = lowerPanel, Position = UDim2.new(0,344,0,12), Size = UDim2.new(1,-356,1,-24), BackgroundTransparency = 1 })
    local qTitle = new("TextLabel", { Parent = queueFrame, Text = "Fila", BackgroundTransparency = 1, TextColor3 = Config.Colors.Muted, Font = Enum.Font.GothamBold, TextSize = 14, Position = UDim2.new(0,0,0,0) })
    local scroll = new("ScrollingFrame", { Parent = queueFrame, Position = UDim2.new(0,0,0,28), Size = UDim2.new(1,0,1,-28), BackgroundColor3 = Color3.fromRGB(18,12,22), CanvasSize = UDim2.new(0,0,0,0), ScrollBarImageColor3 = Config.Colors.Primary })
    novo("UICorner", { Parent = scroll, CornerRadius = UDim.new(0,8) })
    local listLayout = new("UIListLayout", { Parent = scroll, Padding = UDim.new(0,6), SortOrder = Enum.SortOrder.LayoutOrder })
    local listPadding = new("UIPadding", { Parent = scroll, PaddingTop = UDim.new(0,8), PaddingLeft = UDim.new(0,8), PaddingRight = UDim.new(0,8), PaddingBottom = UDim.new(0,8) })

    -- auxiliar para preencher a interface do usuário da fila
    função local refreshQueueUI()
        para _, filho em ipairs(scroll:GetChildren()) faça se filho:IsA("Frame") então filho:Destroy() fim fim
        local q = Orchestrator.queue
        para i, alvo em ipairs(q ou {}) faça
            local item = new("Frame", { Parent = scroll, Size = UDim2.new(1,-12,0,44), BackgroundColor3 = Color3.fromRGB(28,18,36) })
            novo("UICorner", { Parent = item, CornerRadius = UDim.new(0,8) })
            local name = new("TextLabel", { Parent = item, Position = UDim2.new(0,8,0,6), Size = UDim2.new(0.6,0,0,20), BackgroundTransparency = 1, Text = (target.name or "unknown"), TextColor3 = Config.Colors.Text, Font = Enum.Font.GothamSemibold, TextSize = 14, TextXAlignment = Enum.TextXAlignment.Left })
            local distTxt = new("TextLabel", { Parent = item, Position = UDim2.new(0.62,0,0,6), Size = UDim2.new(0.36,-8,0,20), BackgroundTransparency = 1, Text = string.format("%.1f m", (target._distance or 0)/Config.Units.studs_per_meter), TextColor3 = Config.Colors.Muted, Font = Enum.Font.Gotham, TextSize = 12, TextXAlignment = Enum.TextXAlignment.Right })
            local meta = new("TextLabel", { Parent = item, Position = UDim2.new(0,8,0,24), Size = UDim2.new(1,-16,0,16), BackgroundTransparency = 1, Text = ("raridade:"..tostring(target.rarity or "-").." nível:"..tostring(target.level or "-")), TextColor3 = Config.Colors.Muted, Font = Enum.Font.Gotham, TextSize = 12, TextXAlignment = Enum.TextXAlignment.Left })
        fim
        -- atualizar tamanho da tela
        contentSize local = listLayout.AbsoluteContentSize.Y
        scroll.CanvasSize = UDim2.new(0,0,0,contentSize + 12)
    fim

    -- botão de atualização do gancho
    refreshBtn.MouseButton1Click:Connect(function()
        Orchestrator.refreshQueue()
        refreshQueueUI()
        notify("Fila", "Fila atualizada")
    fim)

    -- conectar atualizações da fila do orquestrador por meio de sondagem periódica (leve)
    se RunService então
        RunService.Heartbeat:Connect(function()
            -- atualizar estatísticas
            defeatedLabel.Text = "Derrotados: " .. tostring(Orchestrator.stats.defeated or 0)
            xpLabel.Text = "XP: " .. tostring(Orchestrator.stats.xp or 0)
            -- Atualizar a interface do usuário da fila ocasionalmente
        fim)
        -- Atualizar a interface do usuário a cada 0,8s quando a janela estiver aberta
        spawn(função()
            enquanto screenGui.Parent faça
                refreshQueueUI()
                Utils.sleep(0.8)
            fim
        fim)
    fim

    -- Comportamentos dos botões (conectar ao orquestrador, ESP, Fly)
    startBtn.MouseButton1Click:Connect(function() Orchestrator.start(); notify("Farm", "Iniciado") end)
    pauseBtn.MouseButton1Click:Connect(function() Orchestrator.pause(); notify("Farm", "Pausado") end)
    stopBtn.MouseButton1Click:Connect(function() Orchestrator.stop(); notify("Farm", "Parado") end)

    humanoide local ESPOn = falso
    espBtn.MouseButton1Click:Connect(function()
        humanoideESPOn = não humanoideESPOn
        espInd.BackgroundColor3 = humanoidESPOn e Config.Colors.Accent ou Color3.fromRGB(120,120,120)
        Se humanoidESPOn então ESP.enableHumanoidESP() senão ESP.disableHumanoidESP() fim
    fim)
    local fruitESPOn = falso
    frutaBtn.MouseButton1Click:Conectar(função()
        fruitESPOn = não fruitESPOn
        fruitInd.BackgroundColor3 = fruitESPOn e Config.Colors.Accent ou Color3.fromRGB(120,120,120)
        Se fruitESPOn então ESP.enableFruitESP() senão ESP.disableFruitESP() fim
    fim)
    local flyOn = falso
    flyBtn.MouseButton1Click:Connect(function()
        Se Fly.isAllowed() então
            flyOn = não flyOn
            flyInd.BackgroundColor3 = flyOn e Config.Colors.Accent ou Color3.fromRGB(120,120,120)
            Voar.alternar()
        outro
            notify("Fly", "Não é permitido neste ambiente")
        fim
    fim)

    -- Botão de telemetria
    btnTelemetry.MouseButton1Click:Connect(function()
        Telemetria.exportarParaConsole()
        notify("Telemetria", "Sessão impressa no Output")
    fim)

    -- Ações de fechamento/minimização
    btnClose.MouseButton1Click:Connect(function() screenGui:Destroy() end)
    btnMin.MouseButton1Click:Connect(function()
        setOpen = (function()
            -- alternar minimizar
            se main.Size.Y.Offset > 100 então
                main.Size = UDim2.new(main.Size.X.Scale, main.Size.X.Offset, 0, 56)
            outro
                main.Size = UDim2.new(0,720,0,420)
            fim
        fim)()
    fim)

    -- Implementação de arrastar (barra superior)
    fazer
        arrastar local = falso
        local dragStart, startPos
        barraSuperior.InputBegan:Conectar(função(entrada)
            se input.UserInputType == Enum.UserInputType.MouseButton1 então
                arrastar = verdadeiro
                dragStart = input.Position
                startPos = main.Position
                input.Changed:Connect(function()
                    Se input.UserInputState == Enum.UserInputState.End então arrastar = falso fim
                fim)
            fim
        fim)
        topBar.InputChanged:Connect(function(input)
            Se o arrastar ocorrer e input.UserInputType == Enum.UserInputType.MouseMovement, então
                delta local = input.Position - dragStart
                local newPos = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
                main.Position = newPos
            fim
        fim)
    fim

fim -- fim da interface do usuário do InRoblox

-- =========================
-- COMECE
-- =========================
Telemetry.startSession({ mode = Config.Simulation.dry_run and "dry_run" or "real", user = (InRoblox and LocalPlayer and LocalPlayer.Name) or "local" })
Telemetry.logEvent("system.init", { ENV = Config.ENV })

se Config.Simulation.dry_run então
    Se InRoblox então task.defer(function() Utils.sleep(0.2); Orchestrator.start() end) senão Orchestrator.start() end
fim

se for no Roblox então
    print("[GengarHub] Hub carregado — interface atualizada. Substitua Config.GengarImage por seu ID de ativo.")
    se LocalPlayer então LocalPlayer.AncestryChanged:Connect(function(_,parent) se não parent então Telemetry.endSession() fim fim) fim
outro
    print("[GengarHub] Rodando em ambiente não-Roblox (simulação).")
fim

-- Fim do arquivo
