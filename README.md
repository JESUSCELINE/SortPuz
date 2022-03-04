import pygame, sys, random, time
from pygame.locals import *


def genNumberOfContainer(lev) :
		
	if lev <= 3 :
		total_cont = 4
	elif lev <= 6 :
		total_cont = 5
	elif lev <= 10 :
		total_cont = 6
	elif lev <= 15 :
		total_cont = 7
	elif lev <= 21 :
		total_cont = 8
	elif lev <= 28 :
		total_cont = 9
	elif lev <= 36 :
		total_cont = 10
	elif lev <= 45 :
		total_cont = 11
	else :
		total_cont = 12
	
	return total_cont


def genNumberOfContainerInSets(total_cont) :
	
	if total_cont % 2 == 0 :
		num = (int(total_cont / 2), int(total_cont / 2))
	else :
		num = (int(total_cont / 2) + 1, int(total_cont / 2))
	
	return num


def genPaddings(wind_wid, cont_wid, num_in_sets, cont_spa) :
	
	pad1 = int((wind_wid - ((cont_wid * num_in_sets[0]) + (cont_spa * (num_in_sets[0] - 1)))) / 2)
	pad2 = int((wind_wid - ((cont_wid * num_in_sets[1]) + (cont_spa * (num_in_sets[1] - 1)))) / 2)
	pad = (pad1, pad2)
	
	return pad


def genContainerRectData(pad, cont_spa, cont_wid, cont_hei, nis, ts, sas, set_spa) :
	
	y = sas
	data = []
	for i in range(ts) :
		x = pad[i]
		for j in range(nis[i]) :
			data.append(pygame.Rect(x, y, cont_wid, cont_hei))
			x += cont_wid + cont_spa
		y += cont_hei + set_spa
	
	return data


def genContainerData(crd, total_unemp_cont, total_emp_cont, cont_vol, total_cont, liq_colors) :
	
	liq_data = genLiquidData(total_unemp_cont, cont_vol, liq_colors)
	liq = []
	for i in range(total_unemp_cont) :
		liq.append(liq_data[i])
	for i in range(total_emp_cont) :
		liq.append([])

	hl = [False for i in range(total_cont)]
	emp = [False for i in range(total_unemp_cont)] + [True for i in range(total_emp_cont)]
	fil = [True for i in range(total_unemp_cont)] + [False for i in range(total_emp_cont)]
	con = [True for i in range(total_unemp_cont)] + [False for i in range(total_emp_cont)]
	fwsl = [False for i in range(total_cont)]
	cov = [False for i in range(total_cont)]
	data = {}
	for i in range(total_cont) :
		data[i + 1] = {'rect': crd[i], 'liq': liq[i], 'emp': emp[i], 'fil': fil[i], 'con': con[i], 'hl': hl[i], 'fwsl': fwsl[i], 'cov': cov[i]}
	
	return data
	
	
def genLevelData(lev, pad, cont_spa, cont_wid, cont_hei, total_set, sas, set_spa, msv, cv, liq_colors) :
	
	total_cont = genNumberOfContainer(lev)
	nis = genNumberOfContainerInSets(total_cont)
	nec = 2
	win_count = 0
	select = False
	first1 = True
	first2 = True
	refresh = False
	skip = False
	introducing = lev == 1
	nuc = total_cont - nec
	cont_rect_data = genContainerRectData(pad, cont_spa, cont_wid, cont_hei, nis, total_set, sas, set_spa)
	rand_num = random.randrange(msv)
	random.seed(rand_num)
	cont_data = genContainerData(cont_rect_data, nuc, nec, cv, total_cont, liq_colors)
	for i in cont_data :
		if isContainerFilledWithSameLiquid(cont_data[i]) :
			rand_num = random.randrange(msv)
			random.seed(rand_num)
			cont_data = genContainerData(cont_rect_data, nuc, nec, cv, total_cont, liq_colors)
			break
	
	data = {'tc': total_cont, 'nis': nis, 'nec': nec, 'wc': win_count, 'sel': select, 'fir1': first1, 'fir2': first2, 'nuc': nuc, 'crd': cont_rect_data, 'cd': cont_data, 'rn': rand_num, 'intro': introducing, 'ref': refresh, 'ski': skip}
	
	return data


def divIntoGroupOf(data, num) :
	
	grouped_data = []
	for i in range(0, len(data), num) :
		grouped_data.append([
data[i + j] for j in range(num)
])
	
	return grouped_data


def genLiquidData(total_unemp_cont, cont_vol, liq_colors) :
	
	data = []
	for i in range(total_unemp_cont) :
		for j in range(cont_vol) :
			data.append(liq_colors[i])
	random.shuffle(data)
	data = divIntoGroupOf(data, cont_vol)
	
	return data


def drawContainers(surf, cont_rect, cont_color, thi) :

		x, y = cont_rect.left - 2, cont_rect.top
		wid, hei = cont_rect.width + 4, cont_rect.height + 2
		pygame.draw.line(surf, cont_color, (x, y), (x, y + hei), thi)
		pygame.draw.line(surf, cont_color, (x, y + hei), (x + wid, y + hei), thi)
		pygame.draw.line(surf, cont_color, (x + wid, y + hei), (x + wid, y), thi)
				
		
def drawLiquids(surf, cont, thi, liq_hei) :
		
		x = cont['rect'].left
		y = cont['rect'].bottom - liq_hei
		wid = cont['rect'].width
		hei = liq_hei
		for i in cont['liq'] :
			rect = pygame.Rect(x, y, wid, hei)
			pygame.draw.rect(surf, i, rect)
			y -= liq_hei

def drawHighlight(surf, hlc, cont, hlt) :
		
		x, y = cont['rect'].left, cont['rect'].top
		x -= hlt * 2
		y -= hlt * 2
		wid = cont['rect'].width + (hlt * 4)
		hei = cont['rect'].height + (hlt * 4)
		init = x, y
		for i in range(2) :
			pygame.draw.line(surf, hlc, (x, y), (x + wid, y), hlt)
			y += hei
		x, y = init
		for i in range(2) :
			pygame.draw.line(surf, hlc, (x, y), (x, y + hei), hlt)
			x += wid


def eraseHighlight(surf, bgc, cont, hlt) :
	
	drawHighlight(surf, bgc, cont, hlt)


def isPourable(cont1, cont2) :
	
	if cont2['emp'] or cont1['liq'][-1] == cont2['liq'][-1] :
		return True


def genVolumeToPour(cont1, cont2, cv) :
	
	vol2 = len(cont2['liq'])
	spa2 = cv - vol2
	vp = 1
	for i in range(-1, -(len(cont1['liq'])), -1) :
		if cont1['liq'][i] == cont1['liq'][i - 1] :
			vp += 1
		else :
			break
	if vp <= spa2 :
		vtp = vp
	else :
		vtp = spa2
	
	return vtp
	

def Pour(cont1, cont2, cv) :
	
	vtp = genVolumeToPour(cont1, cont2, cv)
	for i in range(vtp) :
			temp = [cont1['liq'][-1]]
			cont2['liq'] += temp
			top_liq = cont1['liq'][-1]
			del cont1['liq'][-1]


def isContainerFilledWithSameLiquid(cont) :
	
	if cont['fil'] :
		for i in range(len(cont['liq']) - 1) :
			if cont['liq'][i] == cont['liq'][i + 1] :
				retr_val = True
			else :
				retr_val = False
				break
	else :
		retr_val = False
	
	return retr_val


def drawCover(surf, cont, cov_color) :
	
	hei = 30
	wid = cont['rect'].width + 20
	x, y = cont['rect'].left - 10, cont['rect'].top - hei
	rect1 = pygame.Rect(x, y, wid, hei)
	rect2 = pygame.Rect(x, y, wid, hei)
	rect2.top += hei
	rect2.left += 10
	rect2.width -= 20
	pygame.draw.rect(surf, cov_color, rect1)
	pygame.draw.rect(surf, cov_color, rect2)

		
FPS = 60
MAXIMUMSEEDINGVALUE = 100
WINDOWWIDTH = 720
WINDOWHEIGHT = 1344
TOTALSET = 2
CONTAINERVOLUME = 4
CONTAINERWIDTH = 80
CONTAINERHEIGHT = 240
CONTAINERSPACING = 40
CONTAINERTHICKNESS = 5
HIGHLIGHTTHICKNESS = 5
SETSPACING = 75
LIQUIDHEIGHT = (CONTAINERHEIGHT - 40) / CONTAINERVOLUME
SPACEABOVESET = WINDOWHEIGHT * (3 / 8)
coin = 0
gem = 0
level = 1
gem_skip_price = 10
coin_skip_price = 50

WHITE   = (225, 225, 225)
BLACK   = (  0,   0,   0)
YELLOW  = (225, 225,   0)
RED     = (225,   0,   0)
BLUE    = (  0,   0, 225)
LIME    = (  0, 225,   0)
FUCHSIA = (225,   0, 225)
CYAN    = (  0, 225, 225)
GREEN   = (  0, 128,   0)
OLIVE   = (128, 128,   0)
TEAL    = (  0, 128, 128)
GREY    = (128, 128, 128)
ORANGE  = (255, 147,  17)
PURPLE  = (166,  47, 255)
SILVER  = (193, 192, 165)


BACKGROUNDCOLOR = {'intro': YELLOW, 'play': WHITE}
CONTAINERCOLOR = BLACK
TEXT1COLOR = RED
TEXT2COLOR = WHITE
TEXT2RECTCOLOR = BLUE
TEXT3COLOR = WHITE
TEXT3RECTCOLOR = RED
TEXT4COLOR = BLUE
TEXT5COLOR = BLUE
TEXT6COLOR = OLIVE
TEXT7COLOR = GREEN
TEXT8COLOR = WHITE
TEXT8RECTCOLOR = OLIVE
TEXT9COLOR = WHITE
TEXT9RECTCOLOR = GREEN
INSTRUCTIONCOLOR = WHITE
HIGHLIGHTCOLOR  = GREY
COVERCOLOR = OLIVE
LIQUIDCOLORS = [RED, BLUE, YELLOW, LIME, PURPLE, CYAN, GREEN, TEAL, ORANGE, SILVER]

pygame.init()

WINDOWSURFACE = pygame.display.set_mode((WINDOWWIDTH, WINDOWHEIGHT), 0, 32)
FPSCLOCK = pygame.time.Clock()

FONT1 = pygame.font.Font(None, 150)
FONTOBJ1 = FONT1.render('SortPuz', 1, TEXT1COLOR)
FONTRECT1 = FONTOBJ1.get_rect()
FONTRECT1.center = WINDOWWIDTH / 2, WINDOWHEIGHT / 2
FONT2 = pygame.font.Font(None, 50)
FONTOBJ2 = FONT2.render(' Refresh ', 1, TEXT2COLOR, TEXT2RECTCOLOR)
FONTRECT2 = FONTOBJ2.get_rect()
FONT3 = pygame.font.Font(None, 50)
FONTOBJ3 = FONT3.render(' X ', 1, TEXT3COLOR, TEXT3RECTCOLOR)
FONTRECT3 = FONTOBJ3.get_rect()
FONTRECT3.right = WINDOWWIDTH
FONT4 = pygame.font.Font(None, 40)
FONTOBJ4 = FONT4.render('Congratulations, level completed.', 1, TEXT4COLOR)
FONTRECT4 = FONTOBJ4.get_rect()
FONTRECT4.center = WINDOWWIDTH / 2, WINDOWHEIGHT * (1 / 4)
FONT8 = pygame.font.Font(None, 50)
FONTOBJ8 = FONT8.render(' Skip ', 1, TEXT8COLOR, TEXT8RECTCOLOR)
FONTRECT8 = FONTOBJ8.get_rect()
FONTRECT8.topleft = FONTRECT2.topright
FONT9 = pygame.font.Font(None, 50)
FONTOBJ9 = FONT9.render(' Skip ', 1, TEXT9COLOR, TEXT9RECTCOLOR)
FONTRECT9 = FONTOBJ9.get_rect()
FONTRECT9.topleft = FONTRECT8.topright
RECT1 = pygame.Rect(0, 0, WINDOWWIDTH, 50)
RECT2 = pygame.Rect(0, 0, WINDOWWIDTH, 50)
RECT3 = pygame.Rect(0, 0, WINDOWWIDTH, 50)
count = 0
for i in RECT3, RECT2, RECT1 :
	i.top = WINDOWHEIGHT - (RECT1.height * count)
	count += 1
INSTRUCTION1 = pygame.font.Font(None, 20)
INSTRUCTIONOBJ1 = INSTRUCTION1.render("Click the 'Refresh' button to refresh a level.", 1, INSTRUCTIONCOLOR)
INSTRUCTIONRECT1 = INSTRUCTIONOBJ1.get_rect()
INSTRUCTIONRECT1.center = RECT1.center
INSTRUCTION2 = pygame.font.Font(None, 20)
INSTRUCTIONOBJ2 = INSTRUCTION2.render("Click the olive 'Skip' button to skip a level with 50 coins.", 1, INSTRUCTIONCOLOR)
INSTRUCTIONRECT2 = INSTRUCTIONOBJ2.get_rect()
INSTRUCTIONRECT2.center = RECT2.center
INSTRUCTION3 = pygame.font.Font(None, 20)
INSTRUCTIONOBJ3 = INSTRUCTION3.render("Click the green 'Skip' button to skip a level with 10 gems.", 1, INSTRUCTIONCOLOR)
INSTRUCTIONRECT3 = INSTRUCTIONOBJ3.get_rect()
INSTRUCTIONRECT3.center = RECT3.center
FONTS = {'intro': {'obj': FONTOBJ1, 'rect': FONTRECT1}, 'ref': {'obj': FONTOBJ2, 'rect': FONTRECT2}, 'exit': {'obj': FONTOBJ3, 'rect': FONTRECT3}, 'cong': {'obj': FONTOBJ4, 'rect': FONTRECT4}, 'skip1': {'obj': FONTOBJ8, 'rect': FONTRECT8}, 'skip2': {'obj': FONTOBJ9, 'rect': FONTRECT9}, 'ref_inst': {'obj': INSTRUCTIONOBJ1, 'rect': INSTRUCTIONRECT1}, 'skip1_inst': {'obj': INSTRUCTIONOBJ2, 'rect': INSTRUCTIONRECT2}, 'skip2_inst': {'obj': INSTRUCTIONOBJ3, 'rect': INSTRUCTIONRECT3}}

SOUNDS = {'hl': pygame.mixer.Sound('/storage/emulated/0/Android/data/ru.iiec.pydroid3/files/Fullscreen scaling/assets/audio/wing.ogg'), 'cong': pygame.mixer.Sound('/storage/emulated/0/Android/media/com.google.android.gm/Notifications/Treasure/Treasure.ogg'), 'pour': pygame.mixer.Sound('/storage/emulated/0/Android/media/com.google.android.gm/Notifications/Music_box/Music Box.ogg'), 'ref': pygame.mixer.Sound('/storage/emulated/0/Android/data/ru.iiec.pydroid3/files/Fullscreen scaling/assets/audio/die.ogg'), 'fwsl': pygame.mixer.Sound('/storage/emulated/0/Android/data/ru.iiec.pydroid3/files/Fullscreen scaling/assets/audio/point.ogg'), 'new_game': pygame.mixer.Sound('/storage/emulated/0/Android/media/com.google.android.gm/Notifications/Piggyback/Piggyback.ogg')}

while True :
	
	FONT5 = pygame.font.Font(None, 50)
	FONTOBJ5 = FONT5.render('Level: ' + str(level), 1, TEXT5COLOR)
	FONTRECT5 = FONTOBJ5.get_rect()
	FONTRECT5.top = FONTS['ref']['rect'].bottom
	FONTS['lev'] = {'obj': FONTOBJ5, 'rect': FONTRECT5}
	FONT6 = pygame.font.Font(None, 50)
	FONTOBJ6 = FONT6.render('Coin: ' + str(coin), 1, TEXT6COLOR)
	FONTRECT6 = FONTOBJ6.get_rect()
	FONTRECT6.top = FONTS['lev']['rect'].bottom
	FONTS['coin'] = {'obj': FONTOBJ6, 'rect': FONTRECT6}
	FONT7 = pygame.font.Font(None, 50)
	FONTOBJ7 = FONT7.render('Gem: ' + str(gem), 1, TEXT7COLOR)
	FONTRECT7 = FONTOBJ7.get_rect()
	FONTRECT7.top = FONTS['coin']['rect'].bottom
	FONTS['gem'] = {'obj': FONTOBJ7, 'rect': FONTRECT7}
	
	random.shuffle(LIQUIDCOLORS)
	end_level = False
	total_container = genNumberOfContainer(level)
	number_in_sets = genNumberOfContainerInSets(total_container)
	padding = genPaddings(WINDOWWIDTH, CONTAINERWIDTH, number_in_sets, CONTAINERSPACING)
	level_data = genLevelData(level, padding, CONTAINERSPACING, CONTAINERWIDTH, CONTAINERHEIGHT, TOTALSET, SPACEABOVESET, SETSPACING, MAXIMUMSEEDINGVALUE, CONTAINERVOLUME, LIQUIDCOLORS)
	introducing = level_data['intro']
	select = level_data['sel']
	first1 = level_data['fir1']
	first2 = level_data['fir2']
	total_empty_container = level_data['nec']
	total_unempty_container = level_data['nuc']
	win_count = level_data['wc']
	container_rect_data = level_data['crd']
	container_data = level_data['cd']
	random_number = level_data['rn']

	while not end_level :
		
		if introducing :
			WINDOWSURFACE.fill(BACKGROUNDCOLOR['intro'])
			WINDOWSURFACE.blit(FONTS['intro']['obj'], FONTS['intro']['rect'])
	
		else :
			WINDOWSURFACE.fill(BACKGROUNDCOLOR['play'])
			WINDOWSURFACE.blit(FONTS['ref']['obj'], FONTS['ref']['rect'])
			WINDOWSURFACE.blit(FONTS['exit']['obj'], FONTS['exit']['rect'])
			WINDOWSURFACE.blit(FONTS['lev']['obj'], FONTS['lev']['rect'])
			WINDOWSURFACE.blit(FONTS['coin']['obj'], FONTS['coin']['rect'])
			WINDOWSURFACE.blit(FONTS['gem']['obj'], FONTS['gem']['rect'])
			WINDOWSURFACE.blit(FONTS['skip1']['obj'], FONTS['skip1']['rect'])
			WINDOWSURFACE.blit(FONTS['skip2']['obj'], FONTS['skip2']['rect'])
			pygame.draw.rect(WINDOWSURFACE, TEXT2RECTCOLOR, RECT1)
			pygame.draw.rect(WINDOWSURFACE, TEXT8RECTCOLOR, RECT2)
			pygame.draw.rect(WINDOWSURFACE, TEXT9RECTCOLOR, RECT3)
			WINDOWSURFACE.blit(FONTS['ref_inst']['obj'], FONTS['ref_inst']['rect'])
			WINDOWSURFACE.blit(FONTS['skip1_inst']['obj'], FONTS['skip1_inst']['rect'])
			WINDOWSURFACE.blit(FONTS['skip2_inst']['obj'], FONTS['skip2_inst']['rect'])
			
			for i in range(total_container) :
				drawContainers(WINDOWSURFACE, container_data[i + 1]['rect'], CONTAINERCOLOR, CONTAINERTHICKNESS)
				drawLiquids(WINDOWSURFACE, container_data[i + 1], CONTAINERTHICKNESS, LIQUIDHEIGHT)
				if container_data[i + 1]['cov'] :
					drawCover(WINDOWSURFACE, container_data[i + 1], COVERCOLOR)
			for i in container_data :
				if container_data[i]['hl'] :
					drawHighlight(WINDOWSURFACE, HIGHLIGHTCOLOR, container_data[i], HIGHLIGHTTHICKNESS)
					break
				
			if first1 :
				SOUNDS['new_game'].play()
				first1 = False
			win = win_count == total_unempty_container
			if win :
				WINDOWSURFACE.blit(FONTS['cong']['obj'], FONTS['cong']['rect'])
				if first2:
					SOUNDS['cong'].play()
					first2 = False
				end_level = True
		
		for event in pygame.event.get() :
			if event.type == QUIT :
				pygame.quit()
				sys.exit()
			
			elif event.type == MOUSEBUTTONDOWN :
				for i in container_data :
					if container_data[i]['rect'].collidepoint(event.pos) :
						if not select :
							if container_data[i]['con'] and not isContainerFilledWithSameLiquid(container_data[i]) :
								SOUNDS['hl'].play()
								drawHighlight(WINDOWSURFACE, HIGHLIGHTCOLOR, container_data[i], HIGHLIGHTTHICKNESS)
								container_data[i]['hl'] = True
								select = True
						else :
							for j in container_data :
								if container_data[j]['hl'] :
									if len(container_data[i]['liq']) != CONTAINERVOLUME :
										if isPourable(container_data[j], container_data[i]) :
											if not container_data[i]['hl'] :
												SOUNDS['pour'].play()
												Pour(container_data[j], container_data[i], CONTAINERVOLUME)
							for j in container_data :
								if len(container_data[j]['liq']) == 0 :
									container_data[j]['con'] = False
									container_data[j]['emp'] = True
									container_data[j]['fil'] = False
									container_data[j]['fwsl'] == False
								elif len(container_data[j]['liq']) > 0 :
									container_data[j]['con'] = True
									container_data[j]['emp'] = False
									if len(container_data[j]['liq']) == CONTAINERVOLUME :
										container_data[j]['fil'] = True
										if isContainerFilledWithSameLiquid(container_data[i]) :
											container_data[j]['fwsl'] == True
										else :
											container_data[j]['fwsl'] == False
									else :
										container_data[j]['fil'] = False
										container_data[j]['fwsl'] == False
								elif len(container_data[j]['liq']) == CONTAINERVOLUME :
									container_data[j]['con'] = True
									container_data[j]['emp'] = False
									container_data[j]['fil'] = True
									if isContainerFilledWithSameLiquid(container_data[i]) :
										container_data[j]['fwsl'] == True
									else :
										container_data[j]['fwsl'] == False
							for j in container_data :
								if container_data[j]['hl'] :
										container_data[j]['hl'] = False
										select = False
										break
							
						if not container_data[i]['cov'] and isContainerFilledWithSameLiquid(container_data[i]) :
							win_count += 1
							container_data[i]['cov'] = True
							SOUNDS['fwsl'].play()
	
				if not end_level and FONTS['ref']['rect'].collidepoint(event.pos) :
					level_data['ref'] = True
					SOUNDS['ref'].play()
					select = False
					win_count = 0
					random.seed(random_number)
					container_data = genContainerData(container_rect_data, total_unempty_container, total_empty_container, CONTAINERVOLUME, total_container, LIQUIDCOLORS)
				
				if FONTS['skip2']['rect'].collidepoint(event.pos) :
					if gem >= gem_skip_price :
						end_level = True
						gem -= gem_skip_price
						level_data['ski'] = True
				if FONTS['exit']['rect'].collidepoint(event.pos) :
					pygame.quit()
					sys.exit()
				if FONTS['skip1']['rect'].collidepoint(event.pos) :
					if coin >= coin_skip_price :
						end_level = True
						coin -= coin_skip_price
						level_data['ski'] = True

		pygame.display.update()
		FPSCLOCK.tick(FPS)
		if introducing  :
			time.sleep(2)
			introducing = False
		if end_level :
			time.sleep(2)
			end_level = True

	level += 1
	if not level_data['ski'] :
		coin += 10
	if not level_data['ref'] and not level_data['ski'] :
		gem += 1
