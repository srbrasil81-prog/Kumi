-- main.lua
-- Template GUI em Lua (Love2D)
-- AVISO: Apenas interface local / gerenciamento de configurações.
-- Não contém nem realiza cheats, injeções, ou manipulação de processos externos.

local config = {
    esp_players = false,
    esp_items = false,
    show_names = true,
    speed = 50,   -- 1..100
    fov = 90,     -- 60..180
    extra_option = false
}

local ui = {}
ui.buttons = {}
ui.dragging = nil
ui.status = "Pronto"
ui.font = nil

local function serialize_table(t, indent)
    indent = indent or ""
    local next_indent = indent .. "  "
    local parts = {"{\n"}
    for k, v in pairs(t) do
        local key
        if type(k) == "string" and k:match("^%a[%w_]*$") then
            key = k
        else
            key = "[" .. tostring(k) .. "]"
        end
        local val
        if type(v) == "table" then
            val = serialize_table(v, next_indent)
        elseif type(v) == "string" then
            val = string.format("%q", v)
        elseif type(v) == "number" or type(v) == "boolean" then
            val = tostring(v)
        else
            val = "nil"
        end
        table.insert(parts, next_indent .. key .. " = " .. val .. ",\n")
    end
    table.insert(parts, indent .. "}")
    return table.concat(parts)
end

local function save_config(filename)
    local chunk = "return " .. serialize_table(config)
    local ok, err = love.filesystem.write(filename, chunk)
    if ok then
        ui.status = "Config salva em " .. filename
    else
        ui.status = "Erro ao salvar: " .. tostring(err)
    end
end

local function load_config(filename)
    if not love.filesystem.getInfo(filename) then
        ui.status = "Arquivo não encontrado: " .. filename
        return
    end
    local chunk, loadErr = love.filesystem.load(filename)
    if not chunk then
        ui.status = "Erro ao carregar: " .. tostring(loadErr)
        return
    end
    local ok, result = pcall(chunk)
    if not ok then
        ui.status = "Erro ao executar config: " .. tostring(result)
        return
    end
    if type(result) == "table" then
        -- apenas copia campos esperados
        for k, _ in pairs(config) do
            if result[k] ~= nil then
                config[k] = result[k]
            end
        end
        ui.status = "Config carregada de " .. filename
    else
        ui.status = "Arquivo inválido"
    end
end

local function pointInRect(x, y, rx, ry, rw, rh)
    return x >= rx and x <= rx + rw and y >= ry and y <= ry + rh
end

local function drawCheckbox(x, y, label, value)
    local size = 18
    love.graphics.rectangle("line", x, y, size, size)
    if value then
        love.graphics.line(x+3, y+size/2, x+size/2, y+size-4)
        love.graphics.line(x+size/2, y+size-4, x+size-3, y+3)
    end
    love.graphics.print(label, x + size + 8, y - 2)
    return x, y, size, size
end

local function drawSlider(x, y, w, minv, maxv, val)
    local h = 16
    local barY = y + (h - 6) / 2
    love.graphics.rectangle("line", x, barY, w, 6)
    local t = (val - minv) / (maxv - minv)
    local knobX = x + t * w
    love.graphics.circle("fill", knobX, y + h/2, 7)
    return x, y, w, h, knobX
end

local function addButton(id, x, y, w, h, label, onClick)
    ui.buttons[id] = {x=x, y=y, w=w, h=h, label=label, onClick=onClick}
end

function love.load()
    love.window.setTitle("Template GUI - Opções (ESP, Speed, etc.)")
    love.window.setMode(520, 360, {resizable=false})
    ui.font = love.graphics.newFont(12)
    love.graphics.setFont(ui.font)

    -- Botões: Apply, Save, Load
    addButton("apply", 30, 280, 100, 32, "Aplicar", function()
        -- Apenas demonstrativo: imprime no console
        print("Config aplicada:")
        for k, v in pairs(config) do
            print("  ", k, "=", tostring(v))
        end
        ui.status = "Config aplicada (veja console)"
    end)
    addButton("save", 150, 280, 120, 32, "Salvar Config", function()
        save_config("config.lua")
    end)
    addButton("load", 290, 280, 120, 32, "Carregar Config", function()
        load_config("config.lua")
    end)
end

function love.draw()
    love.graphics.clear(0.95, 0.95, 0.95)
    love.graphics.setColor(0, 0, 0)

    -- Título
    love.graphics.setColor(0.1, 0.1, 0.1)
    love.graphics.printf("Template GUI - Opções (ESP, Speed, etc.)", 10, 8, 500, "center")

    love.graphics.setColor(0, 0, 0)
    -- ESP frame
    love.graphics.rectangle("line", 20, 40, 480, 100)
    love.graphics.print("ESP / Overlay", 26, 42)

    -- Checkboxes
    local cx, cy = 40, 70
    local _,_,_,_ = drawCheckbox(cx, cy, "ESP Players", config.esp_players)
    cx = cx + 240
    drawCheckbox(cx, cy, "ESP Items", config.esp_items)

    cy = cy + 30
    cx = 40
    drawCheckbox(cx, cy, "Mostrar nomes", config.show_names)
    cx = cx + 240
    drawCheckbox(cx, cy, "Opção extra (placeholder)", config.extra_option)

    -- Movement / Speed frame
    love.graphics.rectangle("line", 20, 150, 480, 110)
    love.graphics.print("Movement / Speed", 26, 152)

    -- Speed slider
    local sx, sy = 40, 180
    love.graphics.print("Speed: " .. tostring(config.speed), sx, sy - 20)
    local bx, by, bw, bh, knobX = drawSlider(sx, sy, 420, 1, 100, config.speed)

    -- FOV slider
    local fx, fy = 40, 220
    love.graphics.print("FOV: " .. tostring(config.fov), fx, fy - 20)
    drawSlider(fx, fy, 420, 60, 180, config.fov)

    -- Botões
    for id, b in pairs(ui.buttons) do
        love.graphics.setColor(0.85, 0.85, 0.85)
        love.graphics.rectangle("fill", b.x, b.y, b.w, b.h)
        love.graphics.setColor(0,0,0)
        love.graphics.rectangle("line", b.x, b.y, b.w, b.h)
        love.graphics.printf(b.label, b.x, b.y + 8, b.w, "center")
    end

    -- Status
    love.graphics.setColor(0.2, 0.2, 0.6)
    love.graphics.print("Status: " .. ui.status, 20, 330)
    love.graphics.setColor(1,1,1)
end

function love.mousepressed(x, y, button)
    if button ~= 1 then return end

    -- Click em checkboxes
    local checks = {
        {x=40, y=70, key="esp_players"},
        {x=280, y=70, key="esp_items"},
        {x=40, y=100, key="show_names"},
        {x=280, y=100, key="extra_option"}
    }
    for _, c in ipairs(checks) do
        if pointInRect(x, y, c.x, c.y, 18, 18) then
            config[c.key] = not config[c.key]
            ui.status = c.key .. " = " .. tostring(config[c.key])
            return
        end
    end

    -- Sliders: speed and fov
    -- speed slider geometry
    local sx, sy, sw, sh = 40, 180, 420, 16
    local fx, fy, fw, fh = 40, 220, 420, 16
    if pointInRect(x, y, sx, sy, sw, sh) then
        ui.dragging = {type="speed", x0 = sx, w = sw, min = 1, max = 100}
        love.mousemoved(x, y, 0, 0)
        return
    elseif pointInRect(x, y, fx, fy, fw, fh) then
        ui.dragging = {type="fov", x0 = fx, w = fw, min = 60, max = 180}
        love.mousemoved(x, y, 0, 0)
        return
    end

    -- Botões
    for id, b in pairs(ui.buttons) do
        if pointInRect(x, y, b.x, b.y, b.w, b.h) then
            if b.onClick then b.onClick() end
            return
        end
    end
end

function love.mousereleased(x, y, button)
    if button ~= 1 then return end
    ui.dragging = nil
end

function love.mousemoved(x, y, dx, dy)
    if ui.dragging then
        local d = ui.dragging
        local t = (x - d.x0) / d.w
        if t < 0 then t = 0 elseif t > 1 then t = 1 end
        local value = math.floor(d.min + t * (d.max - d.min) + 0.5)
        if d.type == "speed" then
            config.speed = value
            ui.status = "speed = " .. tostring(config.speed)
        elseif d.type == "fov" then
            config.fov = value
            ui.status = "fov = " .. tostring(config.fov)
        end
    end
end

function love.keypressed(key)
    if key == "escape" then
        love.event.quit()
    end
end
