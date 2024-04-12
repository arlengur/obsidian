Lessons https://www.youtube.com/c/TimesaverVFX/playlists

Ниламбара:
- Лого: 846x-291x0.17
- Youtube thumbnail: 1280x720
# Settings (Системные настройки)
Cmd+, - settings
- System
	- Memory and GPU - Memory Configuration - на максимум
	- Memory and GPU - GPU processing mode - использовать ресурсы видеокарты
	- Media Storage - где хранить файлы Давинчи
- User
	- Project Save and Load - Project backups (every 15min)
	- Project Save and Load - Project backup location - где хранятся бэкапы
	- Editing - Start timecode (00:00:00:00)

# Project settings (Настройки проекта)
Shift+9 - project settings
Project Settings - Master Settings:
- Timeline format
	- Timeline resolution
	- Timeline frame rate: 25
- Optimized Media and render cache (разрешение и качество прокси и оптимизированных медиа)
	- Proxy media resolution: Quarter
	- Proxy media format: DNxHR HQ
	- Optimized media resolution: Quarter
	- Optimized media format: DNxHR HQ
	- Render cache format: DNxHR HQ
- Working folders (где эти файлы хранятся)
	- Proxy generation location: same as project (их часто пользуем, лучше на ssd диске)
	- Gallery stills location: same as project (screenshots during color editing)
	- Retime process: optical flow (когда делаешь замедление видео очень помогает)
- General options
	- Use timelines bin (использовать отдельную папку для таймлайнов)

# Shortcuts
opt+cmd+K - keyboard customization
- Timeline shortcuts
	- Shift+1 - projects library
	- Shift+2 - media pull
	- Shift+3 - cut
	- Shift+4 - edit
	- Shift+5 - fusion
	- Shift+6 - color
	- Shift+7 - fairlight
	- Shift+8 - deliver
	- F9 - insert
	- F10 - overwrite
	- F11 - replace
	- F12 - replace on top
	- Shift+F10 - append at end
	- Shift+F12 - ripple overwrite

	- c - split clip
	- \[ - trim - ripple - start to playhead
	- ] - trim - ripple - end to playhead
	- , - trim - nudge - one frame left
	- . - trim - nudge - one frame right
	- shift+0 - view - zoom - zoom to fit
	- \+ - view - zoom - zoom in
	- \- - view - zoom - zoom out
	- cmd+z - view - zoom viewer to fit
	- opt+/ - playback - play around/to - play in to out (works in loop is activated)
	- -/+ - Application - Timecode - Dec/Increment 
	- cmd+= - Timecode - Go To

r - change clip speed
- ripple sequence - сдвинет другие секвенции соответственно
- freeze frame - все что останется превратиться в скриншот

shift+r - make all as screenshot
cmd+r - показать дорожку скорости
cmd+t - сделать переход на видео и аудио
opt+t - сделать переход только на видео 
shift+t - сделать переход только на аудио

Performance optimization:

shift+d - toggle effects
Playback - timeline Proxy resolution 
ПКМ - generate proxy media
ПКМ - render cache color/fusion output (playback - render cache - user)
ПКМ - render in place

Inspector

Stabilization 
Mode - Perspective (произвольное вращение) - Similarity (оси х и у и z) - Translation (оси х и у)
Strength - 0.5 to 1

Retime and scaling - как просчитываются кадры при условии их нехватки
- Retime process: Nearest - Frame Blend - Optical flow (от простого к сложному)
- Scaling - как ведут себя файлы, которые не попадают по разрешению

Deliver
Advanced settings:
-bypass re-encode when possible
-use render cached images
-force sizing to highest quality
-force debayer to highest quality

Generate optimized media - создает прокси

Fusion

Effects-templates-edit-colorer border - цветная рамка
Open FX-blanking fill - размывает туже картинку на фоне
drop shadow - добавить тень
color compressor - убрать цвет
lens blur - размытие
ResolverFX Warp - добавить волны

cmd+c - copy style
opt+v - paste attributes

edge defect overlay
glow

Shift+9 (настройки проекта) - color management - lookup tables - open LUT folder - load folder and click - Update lists
Opt+s - создать новую ноду в цветокоре

ПКМ+A - включить альфа режим в LumaKey

Slow motion
- 24|25 fps - don't slow down (or use twixtor plugin)
- 30 fps - 80% normal speed
- 50 fps - 50% normal speed
- 60 fps - 40% normal speed
- 120 fps - 20% normal speed

Shutter speed:
- 24|25 fps - shutter speed 1/50
- 30 fps - shutter speed 1/60
- 50 fps - shutter speed 1/100
- 60 fps - shutter speed 1/125
- 120 fps - shutter speed 1/250

Сохранить скриншот
- перейти на вкладку color
- ПКМ "grab still
- Media Pool - Export as jpeg

4x+3y=114
3x+4y=110

x-y=4
y=x-4

3x+4x-16=110=126