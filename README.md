if not VIP_USER then return end
_G.LevelSpell = function(id)
	local offsets = {
		[_Q] = 0xAA,
		[_W] = 0xAB,
		[_E] = 0xAC,
		[_R] = 0xAD,
	}
	local p = CLoLPacket(0x009C)
	p.vTable = 0xF0A57C
	p:EncodeF(myHero.networkID)
	p:Encode1(0x46)
	p:Encode1(0xD5D5D5D5)
	p:Encode1(ofssets[ID])
	p:Encode1(0xF8)
	p:Encode1(0xEA)
	p:Encode1(0x0C)
	p:Encode1(0x0000)
	SendPacket(p)
end

local abilitySequence
local ini=true
local _autoLevel = { spellsSlots = { SPELL_1, SPELL_2, SPELL_3, SPELL_4 }, levelSequence = {}, nextUpdate = 0, tickUpdate = 5 }
local __autoLevel__OnTick
local rOFF=0
--update func--
local version = "2.14"
local AUTOUPDATE = true
local UPDATE_HOST = "raw.github.com"
local UPDATE_PATH = "/prado1506/Bol1/master/AutoLevelSkillTyler1.lua".."?rand="..math.random(1,10000)
local UPDATE_FILE_PATH = SCRIPT_PATH..GetCurrentEnv().FILE_NAME
local UPDATE_URL = "https://"..UPDATE_HOST..UPDATE_PATH

function _AutoupdaterMsg(msg) print("<b><font color=\"#FF0000\">Giovani Auto Level Spells:</font></b> <font color=\"#FFFFFF\">"..msg.."</font>") end
if AUTOUPDATE then
	local ServerData = GetWebResult(UPDATE_HOST, "/prado1506/Bol1/master/AutoLevelSkillTyler1.version")
	if ServerData then
		ServerVersion = type(tonumber(ServerData)) == "number" and tonumber(ServerData) or nil
		if ServerVersion then
			if tonumber(version) < ServerVersion then
				_AutoupdaterMsg("New version available "..ServerVersion)
				_AutoupdaterMsg("Updating, please don't press F9")
				DelayAction(function() DownloadFile(UPDATE_URL, UPDATE_FILE_PATH, function () _AutoupdaterMsg("Successfully updated. ("..version.." => "..ServerVersion.."), press F9 twice to load the updated version.") end) end, 3)
			else
				_AutoupdaterMsg("You have got the latest version ("..ServerVersion..")")
			end
		end
	else
		_AutoupdaterMsg("Error downloading version info")
	end
end
--end updt

local function autoLevel__OnLoad()
    if not __autoLevel__OnTick then
        function __autoLevel__OnTick()
            local tick = os.clock()
            if _autoLevel.nextUpdate > tick then return end
            _autoLevel.nextUpdate = tick + Menu.DelayTime
            local realLevel = rOFF + GetHeroLeveled()
            if player.level > realLevel and _autoLevel.levelSequence[realLevel + 1] ~= nil then
                local splell = _autoLevel.levelSequence[realLevel + 1]
                if splell == 0 and type(_autoLevel.onChoiceFunction) == "function" then splell = _autoLevel.onChoiceFunction() end
                if type(splell) == "number" and splell >= 1 and splell <= 4 then LevelSpell(_autoLevel.spellsSlots[splell]) end
            end
        end

        AddTickCallback(__autoLevel__OnTick)
    end
end

function autoLevelSetSequenceCustom(sequence)
    assert(sequence == nil or type(sequence) == "table", "autoLevelSetSequence : wrong argument types (<table> or nil expected)")
    autoLevel__OnLoad()
    local sequence = sequence or {}
    for i = 1, 18 do
        local spell = sequence[i]
        if type(spell) == "number" and spell >= 0 and spell <= 4 then
            _autoLevel.levelSequence[i] = spell
       -- else
     --       _autoLevel.levelSequence[i] = nil
        end
    end
end

function OnLoad()
	print("<b><font color=\"#FF0000\">Thize Auto Level:</font></b> <font color=\"#FFFFFF\">Loading...</font>"..version)
	Menu = scriptConfig("["..myHero.charName.." - AutoLevel]", player.charName.."AutoLevel")
	Menu:addParam("sequenceSpells", "Sequence Spells", SCRIPT_PARAM_LIST, 1, { 'R-Q-W-E',})
	print("<b><font color=\"#FF0000\">Thize Auto Level:</font></b> <font color=\"#FFFFFF\">Sucessfully Loaded..!</font>")
end
