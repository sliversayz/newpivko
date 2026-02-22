local sampev = require 'lib.samp.events'
local imgui = require("mimgui")
local encoding = require 'encoding'
local memory = require 'memory'
local bit = require('bit')
local fa = require("fAwesome5")
encoding.default = 'CP1251'
local u8 = encoding.UTF8
local ffi = require 'ffi'
local new = imgui.new
local https = require('ssl.https') -- для загрузки с GitHub
local json = require('dkjson') -- для парсинга JSON (если нужно)
require('lib.moonloader')
require('lib.sampfuncs')

-- Версия скрипта (меняйте при каждом обновлении)
local SCRIPT_VERSION = "1.0.0"
local GITHUB_RAW_URL = "https://raw.githubusercontent.com/ВАШ_АККАУНТ/ВАШ_РЕПОЗИТОРИЙ/main/newchea.lua"
local GITHUB_VERSION_URL = "https://raw.githubusercontent.com/sliversayz/newpivko/refs/heads/main/version.txt"

-- Глобальные переменные
local mainFont = nil
local WinState = imgui.new.bool(false)          -- главное меню
local RouteSelectState = imgui.new.bool(false)   -- окно выбора маршрута

-- Переменные для бота
local botActive = false
local currentRoute = nil   -- "ferma1", "ferma2", "zavod"
local svobodnayaRuka = false

-- Переменные для дополнительных функций
local antiDropObjectActive = imgui.new.bool(false)
local antiBhActive = imgui.new.bool(false)

-- Переменные для автообновления
local updateAvailable = false
local updateChecked = false
local updateProgress = 0
local downloading = false
local downloadSuccess = false
local downloadError = nil

local player = {
    x = 0.0,
    y = 0.0,
    z = 0.0
}

-- Координаты для Ферма 1 (каменщик)
local points_ferma1 = {
    {1658.3875, 692.3926},
    {1611.6863, 684.2416},
    {1591.1506, 683.7051},
    {1564.6937, 650.7100},
    {1555.6661, 651.3547},
    {1555.6661, 651.3547},
    {1591.1506, 683.7051},
    {1611.6863, 684.2416},
    {1658.3875, 692.3926}
}

-- Координаты для Ферма 2 (оригинальная ферма)
local points_ferma2 = {
    {-1074.4659, -1027.3458},
    {-1075.4976, -837.4217},
    {-1097.0354, -836.0347},
    {-1109.6309, -828.8256},
    {-1121.5590, -828.1309},
    {-1109.6309, -828.8256},
    {-1097.0354, -836.0347},
    {-1075.4976, -837.4217},
    {-1074.4659, -1027.3458}
}

-- Координаты для Завода (с паузой 17 сек)
local points_zavod_part1 = {
    {-2916.7700,-1202.0148},
    {-2924.9961,-1204.5250},
    {-2934.0205,-1204.6520},
    {-2940.8232,-1204.4264},
    {-2937.6453,-1198.6371},
    {-2937.8240,-1193.5789}
}
local points_zavod_part2 = {
    {-2907.8328,-1194.8540},
    {-2908.4597,-1181.3795},
    {-2908.6733,-1178.4904},
    {-2911.6284,-1176.1685},
    {-2914.5459,-1195.3065}
}
local ZAVOD_PAUSE = 20000 -- миллисекунд

function sendMessage(msg)
    sampAddChatMessage("[{0395fb}БОТ{FFFFFF}]: "..msg, -1)
end

-- Функция проверки обновлений
function checkForUpdates()
    if updateChecked then return end
    updateChecked = true
    
    lua_thread.create(function()
        local versionFile = io.open("version.txt", "r")
        if versionFile then
            local localVersion = versionFile:read("*all"):gsub("%s+", "")
            versionFile:close()
            
            -- Загружаем версию с GitHub
            local response, status = https.request(GITHUB_VERSION_URL)
            if status == 200 then
                local remoteVersion = response:gsub("%s+", "")
                if remoteVersion ~= localVersion then
                    updateAvailable = true
                    sendMessage("Доступно обновление! Версия: " .. remoteVersion)
                end
            end
        end
    end)
end

-- Функция скачивания обновления
function downloadUpdate()
    if downloading then return end
    downloading = true
    updateProgress = 0
    downloadError = nil
    
    lua_thread.create(function()
        -- Скачиваем новый скрипт
        local response, status = https.request(GITHUB_RAW_URL)
        
        if status == 200 then
            -- Создаём бэкап текущего скрипта
            local scriptPath = debug.getinfo(1).source:gsub("@", "")
            local backupPath = scriptPath:gsub("%.lua$", "_backup.lua")
            
            -- Копируем текущий файл в бэкап
            local currentFile = io.open(scriptPath, "r")
            if currentFile then
                local backupFile = io.open(backupPath, "w")
                if backupFile then
                    backupFile:write(currentFile:read("*all"))
                    backupFile:close()
                end
                currentFile:close()
            end
            
            -- Сохраняем новый скрипт
            local newFile = io.open(scriptPath, "w")
            if newFile then
                newFile:write(response)
                newFile:close()
                downloadSuccess = true
                sendMessage("Обновление загружено! Перезапустите скрипт.")
                
                -- Обновляем файл версии
                local versionFile = io.open("version.txt", "w")
                if versionFile then
                    -- Парсим версию из нового скрипта (можно сохранить отдельно)
                    versionFile:write(SCRIPT_VERSION) -- временно, лучше парсить
                    versionFile:close()
                end
            else
                downloadError = "Не удалось сохранить файл"
            end
        else
            downloadError = "Ошибка загрузки: " .. tostring(status)
        end
        
        downloading = false
        updateProgress = 100
    end)
end

imgui.OnInitialize(function()
    local config = imgui.ImFontConfig()
    config.MergeMode = true
    local glyph_ranges = imgui.GetIO().Fonts:GetGlyphRangesCyrillic()
    local iconRanges = imgui.new.ImWchar[3](fa.min_range, fa.max_range, 0)
    
    mainFont = imgui.GetIO().Fonts:AddFontFromFileTTF('C:\\Windows\\Fonts\\tahoma.ttf', 14.0, nil, glyph_ranges)
    icon = imgui.GetIO().Fonts:AddFontFromFileTTF('moonloader/lib/fa-solid-900.ttf', 14.0, config, iconRanges)
    
    setDuckTrackerTheme()
    
    -- Проверяем обновления при инициализации
    checkForUpdates()
end)

imgui.OnFrame(function() return WinState[0] end, function()
    imgui.SwitchContext()
    
    if mainFont then
        imgui.PushFont(mainFont)
    end
    
    -- Главное окно меню
    imgui.SetNextWindowPos(imgui.ImVec2(900, 500), imgui.Cond.FirstUseEver, imgui.ImVec2(0.5, 0.5))
    imgui.SetNextWindowSize(imgui.ImVec2(500, 550), imgui.Cond.Always)
    imgui.Begin(u8'МЕНЮ БОТА', WinState, imgui.WindowFlags.NoResize)
    
    if imgui.BeginTabBar('MainTabs') then
        -- Вкладка "Главная"
        if imgui.BeginTabItem(u8' Главная') then
            imgui.TextColored(imgui.ImVec4(1.00, 0.08, 0.37, 1.00), u8' Управление ботами')
            imgui.Separator()
            imgui.Spacing()
            
            -- Кнопка открытия окна выбора маршрута
            if imgui.Button(u8' Bot Маршруты', imgui.ImVec2(200, 40)) then
                RouteSelectState[0] = true
            end
            
            -- Отображение статуса активного бота
            if botActive then
                local routeName = ""
                if currentRoute == "ferma1" then routeName = "Ферма 1"
                elseif currentRoute == "ferma2" then routeName = "Ферма 2"
                elseif currentRoute == "zavod" then routeName = "Завод"
                end
                imgui.Spacing()
                imgui.Text(u8'Активный бот: ' .. routeName)
                if imgui.Button(u8' Остановить бота', imgui.ImVec2(200, 30)) then
                    botActive = false
                    currentRoute = nil
                    sendMessage('Бот остановлен')
                end
            else
                imgui.Spacing()
                imgui.Text(u8'Бот не активен')
            end
            
            imgui.EndTabItem()
        end
        
        -- Вкладка "Дополнительно"
        if imgui.BeginTabItem(u8' Дополнительно') then
            imgui.TextColored(imgui.ImVec4(1.00, 0.08, 0.37, 1.00), u8' Дополнительные функции')
            imgui.Separator()
            imgui.Spacing()
            
            imgui.Checkbox(u8'AntiDropObject', antiDropObjectActive)
            imgui.Spacing()
            imgui.Checkbox(u8'AntiBunnyHop (AntiBH)', antiBhActive)
            
            imgui.EndTabItem()
        end
        
        -- Вкладка "Обновления"
        if imgui.BeginTabItem(u8' Обновления') then
            imgui.TextColored(imgui.ImVec4(1.00, 0.08, 0.37, 1.00), u8' Автообновление')
            imgui.Separator()
            imgui.Spacing()
            
            imgui.Text(u8'Текущая версия: ' .. SCRIPT_VERSION)
            imgui.Spacing()
            
            if updateAvailable then
                imgui.TextColored(imgui.ImVec4(0.00, 1.00, 0.00, 1.00), u8'Доступно обновление!')
                imgui.Spacing()
                
                if downloading then
                    imgui.Text(u8'Загрузка: ' .. updateProgress .. '%')
                    imgui.Spacing()
                elseif downloadSuccess then
                    imgui.TextColored(imgui.ImVec4(0.00, 1.00, 0.00, 1.00), u8'Обновление загружено!')
                    imgui.Text(u8'Перезапустите скрипт')
                elseif downloadError then
                    imgui.TextColored(imgui.ImVec4(1.00, 0.00, 0.00, 1.00), u8'Ошибка: ' .. downloadError)
                    imgui.Spacing()
                    if imgui.Button(u8'Повторить', imgui.ImVec2(150, 30)) then
                        downloadUpdate()
                    end
                else
                    if imgui.Button(u8'Скачать обновление', imgui.ImVec2(200, 40)) then
                        downloadUpdate()
                    end
                end
            else
                if updateChecked then
                    imgui.Text(u8'У вас актуальная версия')
                else
                    imgui.Text(u8'Проверка обновлений...')
                end
                
                imgui.Spacing()
                if imgui.Button(u8'Проверить обновления', imgui.ImVec2(200, 30)) then
                    updateChecked = false
                    checkForUpdates()
                end
            end
            
            imgui.EndTabItem()
        end
        
        imgui.EndTabBar()
    end
    imgui.End()
    
    -- Окно выбора маршрута
    if RouteSelectState[0] then
        imgui.SetNextWindowSize(imgui.ImVec2(300, 250), imgui.Cond.Always)
        imgui.SetNextWindowPos(imgui.ImVec2(950, 550), imgui.Cond.FirstUseEver, imgui.ImVec2(0.5, 0.5))
        imgui.Begin(u8'Выбор маршрута', RouteSelectState, imgui.WindowFlags.NoResize)
        
        imgui.Text(u8'Выберите маршрут:')
        imgui.Spacing()
        
        -- Кнопка Ферма 1
        if imgui.Button(u8'Ферма 1', imgui.ImVec2(250, 40)) then
            if botActive and currentRoute ~= "ferma1" then
                botActive = false
                currentRoute = nil
                wait(100)
            end
            if not botActive then
                botActive = true
                currentRoute = "ferma1"
                sendMessage('Ферма 1 - Работаем')
                startBot(points_ferma1)
            elseif currentRoute == "ferma1" then
                botActive = false
                currentRoute = nil
                sendMessage('Ферма 1 - Остановлен')
            end
            RouteSelectState[0] = false
        end
        
        imgui.Spacing()
        
        -- Кнопка Ферма 2
        if imgui.Button(u8'Ферма 2', imgui.ImVec2(250, 40)) then
            if botActive and currentRoute ~= "ferma2" then
                botActive = false
                currentRoute = nil
                wait(100)
            end
            if not botActive then
                botActive = true
                currentRoute = "ferma2"
                sendMessage('Ферма 2 - Работаем')
                startBot(points_ferma2)
            elseif currentRoute == "ferma2" then
                botActive = false
                currentRoute = nil
                sendMessage('Ферма 2 - Остановлен')
            end
            RouteSelectState[0] = false
        end
        
        imgui.Spacing()
        
        -- Кнопка Завод
        if imgui.Button(u8'Завод', imgui.ImVec2(250, 40)) then
            if botActive and currentRoute ~= "zavod" then
                botActive = false
                currentRoute = nil
                wait(100)
            end
            if not botActive then
                botActive = true
                currentRoute = "zavod"
                sendMessage('Завод - Работаем')
                startZavodBot()   -- специальная функция для завода с паузой
            elseif currentRoute == "zavod" then
                botActive = false
                currentRoute = nil
                sendMessage('Завод - Остановлен')
            end
            RouteSelectState[0] = false
        end
        
        imgui.End()
    end
    
    if mainFont then
        imgui.PopFont()
    end
end)

-- Общая функция для обычных маршрутов (без паузы)
function followPath(points)
    local i = 1
    local xAngle = -0.1
    local currentAngle = 0
    local isJump = false

    while botActive and i <= #points do
        local tox, toy = points[i][1], points[i][2]
        local nextPoint = (i < #points) and points[i+1] or nil

        while botActive do
            local x, y, z = getCharCoordinates(PLAYER_PED)
            if not (x and y) then break end
            player.x, player.y, player.z = x, y, z

            local distToCurrent = getDistanceBetweenCoords2d(x, y, tox, toy)

            if distToCurrent < 1.5 then
                break
            end

            local targetX, targetY
            if nextPoint and distToCurrent < 7.0 then
                targetX, targetY = nextPoint[1], nextPoint[2]
            else
                targetX, targetY = tox, toy
            end

            local dx = targetX - x
            local dy = targetY - y
            local yaw = math.atan2(dy, dx) - math.pi/2
            setCameraPositionUnfixed(xAngle, yaw)

            setGameKeyState(1, -255)   -- W
            setGameKeyState(16, -255)  -- Shift

            if currentRoute ~= "zavod" then
                if not isCharInAir(PLAYER_PED) and not isJump and not svobodnayaRuka and distToCurrent > 15 then
                    setGameKeyState(14, 1)
                    isJump = true
                    lua_thread.create(function()
                        wait(700)
                        isJump = false
                    end)
                end
            end

            wait(0)
        end

        i = i + 1
    end
end

function startBot(points)
    lua_thread.create(function()
        pcall(function() writeMemory(7634870, 1, 1, 1) end)
        pcall(function() writeMemory(7635034, 1, 1, 1) end)
        pcall(function() memory.fill(7623723, 144, 8) end)
        pcall(function() memory.fill(5499528, 144, 6) end)
        wait(100)
        followPath(points)
    end)
end

-- Специальная функция для завода с паузой
function startZavodBot()
    lua_thread.create(function()
        pcall(function() writeMemory(7634870, 1, 1, 1) end)
        pcall(function() writeMemory(7635034, 1, 1, 1) end)
        pcall(function() memory.fill(7623723, 144, 8) end)
        pcall(function() memory.fill(5499528, 144, 6) end)
        wait(100)
        
        while botActive do
            -- Первая часть маршрута
            for i = 1, #points_zavod_part1 do
                if not botActive then break end
                local tox, toy = points_zavod_part1[i][1], points_zavod_part1[i][2]
                local x, y, z = getCharCoordinates(PLAYER_PED)
                if x and y then
                    player.x, player.y, player.z = x, y, z
                end
                
                -- Используем followPath для одной точки? Нет, тут нужен runToPoint или свой цикл
                -- Упростим: используем локальный цикл движения для одной точки
                local xAngle = -0.1
                local targetAngle = getHeadingFromVector2d(tox - player.x, toy - player.y)
                setCameraPositionUnfixed(xAngle, math.rad(targetAngle - 90))
                
                local dist = getDistanceBetweenCoords2d(player.x, player.y, tox, toy)
                local timeout = 0
                
                while dist > 1.5 and botActive do
                    wait(0)
                    timeout = timeout + 1
                    if timeout > 5000 then break end
                    
                    x, y, z = getCharCoordinates(PLAYER_PED)
                    if x and y then
                        player.x, player.y, player.z = x, y, z
                    end
                    dist = getDistanceBetweenCoords2d(player.x, player.y, tox, toy)
                    
                    setGameKeyState(1, -255)  -- W
                    setGameKeyState(16, -255) -- Shift
                end
            end
            
            -- Пауза 20 секунд
            if botActive then
                sendMessage('Пауза 20 секунд...')
                local pauseStart = os.clock()
                while botActive and (os.clock() - pauseStart) * 1000 < ZAVOD_PAUSE do
                    wait(0)
                end
                sendMessage('Продолжаем')
            end
            
            -- Вторая часть маршрута
            for i = 1, #points_zavod_part2 do
                if not botActive then break end
                local tox, toy = points_zavod_part2[i][1], points_zavod_part2[i][2]
                local x, y, z = getCharCoordinates(PLAYER_PED)
                if x and y then
                    player.x, player.y, player.z = x, y, z
                end
                
                local xAngle = -0.1
                local targetAngle = getHeadingFromVector2d(tox - player.x, toy - player.y)
                setCameraPositionUnfixed(xAngle, math.rad(targetAngle - 90))
                
                local dist = getDistanceBetweenCoords2d(player.x, player.y, tox, toy)
                local timeout = 0
                
                while dist > 1.5 and botActive do
                    wait(0)
                    timeout = timeout + 1
                    if timeout > 5000 then break end
                    
                    x, y, z = getCharCoordinates(PLAYER_PED)
                    if x and y then
                        player.x, player.y, player.z = x, y, z
                    end
                    dist = getDistanceBetweenCoords2d(player.x, player.y, tox, toy)
                    
                    setGameKeyState(1, -255)  -- W
                    setGameKeyState(16, -255) -- Shift
                end
            end
        end
    end)
end

function sendKey(code, scode, key)
    local _, myId = sampGetPlayerIdByCharHandle(PLAYER_PED)
    local data = allocateMemory(68)
    sampStorePlayerOnfootData(myId, data)
    setStructElement(data, code, scode, key, false)
    sampSendOnfootData(data)
    freeMemory(data)
end

function sampev.onSendPlayerSync(data)
    -- Для бота-маршрута (старая логика)
    if data and botActive then
        if bit.band(data.keysData, 0x28) == 0x28 then
            data.keysData = bit.bxor(data.keysData, 0x20)
        end
    end
    
    -- AntiBunnyHop
    if antiBhActive and antiBhActive[0] and data and data.keysData then
        if bit.band(data.keysData, 0x28) == 0x28 then
            data.keysData = bit.bxor(data.keysData, 0x20)
        end
    end
    
    -- AntiDropObject
    if antiDropObjectActive and antiDropObjectActive[0] and data then
        data.keysData = 0
    end
    
    return data
end

function sampev.onSetPlayerSpecialAction()
    if antiDropObjectActive and antiDropObjectActive[0] then
        return false
    end
    return true
end

function main()
    wait(1000)
    if not isSampLoaded() or not isSampfuncsLoaded() then return end
    while not isSampAvailable() do wait(100) end
    
    sampAddChatMessage('{32CD32}[BotMenu] {E0FFFF}загружен! F5', -1)
    
    while true do
        wait(0)
        if wasKeyPressed(VK_F5) then
            WinState[0] = not WinState[0]
        end
    end
end

function setDuckTrackerTheme()
    local style = imgui.GetStyle()
    local colors = style.Colors

    local accent      = imgui.ImVec4(0.48, 0.41, 0.93, 1.00)
    local accentHover = imgui.ImVec4(0.60, 0.52, 1.00, 1.00)
    local accentActive= imgui.ImVec4(0.38, 0.31, 0.83, 1.00)
    local bg          = imgui.ImVec4(0.13, 0.14, 0.18, 0.40)
    local bg2         = imgui.ImVec4(0.18, 0.19, 0.23, 1.00)
    local bgPopup     = imgui.ImVec4(0.16, 0.17, 0.22, 0.98)
    local border      = imgui.ImVec4(0.32, 0.32, 0.45, 0.60)
    local white       = imgui.ImVec4(1.00, 1.00, 1.00, 1.00)
    local textMuted   = imgui.ImVec4(0.70, 0.72, 0.85, 1.00)
    local separator   = imgui.ImVec4(0.32, 0.32, 0.45, 0.60)

    colors[imgui.Col.WindowBg]             = bg
    colors[imgui.Col.ChildBg]              = bg2
    colors[imgui.Col.PopupBg]              = bgPopup
    colors[imgui.Col.Border]               = border
    colors[imgui.Col.BorderShadow]         = imgui.ImVec4(0,0,0,0)
    colors[imgui.Col.FrameBg]              = bg2
    colors[imgui.Col.FrameBgHovered]       = accentHover
    colors[imgui.Col.FrameBgActive]        = accentActive
    colors[imgui.Col.TitleBg]              = accent
    colors[imgui.Col.TitleBgActive]        = accentActive
    colors[imgui.Col.TitleBgCollapsed]     = accentActive
    colors[imgui.Col.MenuBarBg]            = bg2
    colors[imgui.Col.ScrollbarBg]          = bg2
    colors[imgui.Col.ScrollbarGrab]        = accent
    colors[imgui.Col.ScrollbarGrabHovered] = accentHover
    colors[imgui.Col.ScrollbarGrabActive]  = accentActive
    colors[imgui.Col.CheckMark]            = accent
    colors[imgui.Col.SliderGrab]           = accent
    colors[imgui.Col.SliderGrabActive]     = accentHover
    colors[imgui.Col.Button]               = accent
    colors[imgui.Col.ButtonHovered]        = accentHover
    colors[imgui.Col.ButtonActive]         = accentActive
    colors[imgui.Col.Header]               = accent
    colors[imgui.Col.HeaderHovered]        = accentHover
    colors[imgui.Col.HeaderActive]         = accentActive
    colors[imgui.Col.Separator]            = separator
    colors[imgui.Col.SeparatorHovered]     = accentHover
    colors[imgui.Col.SeparatorActive]      = accentActive
    colors[imgui.Col.Text]                 = white
    colors[imgui.Col.TextDisabled]         = textMuted
    colors[imgui.Col.TextSelectedBg]       = accent
    colors[imgui.Col.DragDropTarget]       = accent
    colors[imgui.Col.NavHighlight]         = accent
    colors[imgui.Col.Tab]                  = accent
    colors[imgui.Col.TabHovered]           = accentHover
    colors[imgui.Col.TabActive]            = accentActive
    colors[imgui.Col.TabUnfocused]         = bg2
    colors[imgui.Col.TabUnfocusedActive]   = bg2

    style.WindowRounding    = 9
    style.ChildRounding     = 7 
    style.FrameRounding     = 7
    style.PopupRounding     = 7
    style.ScrollbarRounding = 7 
    style.GrabRounding      = 7
    style.TabRounding       = 7

    style.WindowBorderSize  = 1.5
    style.FrameBorderSize   = 1.0
    style.PopupBorderSize   = 1.0

    style.WindowPadding     = imgui.ImVec2(15, 11)
    style.FramePadding      = imgui.ImVec2(9, 5)
    style.ItemSpacing       = imgui.ImVec2(9, 7)
    style.ItemInnerSpacing  = imgui.ImVec2(7, 5)
end
