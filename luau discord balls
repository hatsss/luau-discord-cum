local HttpService = game:GetService('HttpService')
local websocket = syn and syn.websocket or WebSocket
local request = syn and syn.request or request

local Discord = {
    Token = nil,
    WebSocket = nil,
    Events = {},
}

function Discord:Start(Token, Intents)
    Discord.Token = Token

    local Payload = {
        ['op'] = 2,
        ['d'] = {
            ['token'] = Token,
            ['properties'] = {
                ['$os'] = 'linux',
                ['$browser'] = 'chrome',
                ['$device'] = 'chrome'
            }
        }
    }
    if Intents then
        Payload['intents'] = Intents
    end
    Payload = HttpService:JSONEncode(Payload)

    Discord.WebSocket = syn.websocket.connect('ws://gateway.discord.gg/?v=10&encoding=json')
    Discord.WebSocket:Send(Payload)
    coroutine.wrap(function()
        while true do
            task.wait(30)
            Discord.WebSocket:Send(HttpService:JSONEncode({
                ['op'] = 1,
                ['d'] = 'null'
            }))
        end
    end)()

    Discord.WebSocket.OnMessage:Connect(function(Response)
        Response = HttpService:JSONDecode(Response)
        local Event, Data, Opcode = Response.t, Response.d, Response.op
        if Opcode == 0 then
            if Discord.Events[Event] then
                Discord.Events[Event](Data)
            end
        end
    end)
end

function Discord:End()
    Connected = false
    Discord.WebSocket:Close()
end

function Discord:OnEvent(Event, Callback)
    Discord.Events[Event] = Callback
end

function Discord:Send(Endpoint, Method, Body)
    local Data = {
        Url = 'https://discord.com/api/'..Endpoint,
        Method = Method or 'GET',
        Headers = {
            ['User-Agent'] = 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) discord/1.0.9006 Chrome/91.0.4472.164 Electron/13.6.6 Safari/537.36',
            ['authorization'] = Discord.Token
        }
    }
    if Method == 'POST' then
        Data.Headers['Content-Type'] = 'application/json'
        Data.Body = HttpService:JSONEncode(Body)
    end
    local Body = syn.request(Data).Body
    pcall(function()
        Body = HttpService:JSONDecode(Body)
    end)
    return Body
end

return Discord
