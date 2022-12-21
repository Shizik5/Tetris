## Тетрис
Вводим модули и библиотеки для дальнейшей работы:
```python3
import pygame as pg
import random, time, sys
from pygame.locals import *
```
Для работы игры определяем частоту кадров и размеры окна, будущих фигур, игровой области.
```python3
fps = 25
window_w, window_h = 600, 500
block, cup_h, cup_w = 20, 20, 10
```
Задаём время, за которое фигура должна будет переместится в любую сторону или вниз при нажатии игроком на клавиши.
```python3
side_freq, down_freq = 0.15, 0.1
```
Отделяем игровую область cup на определённое расстояние от основного окна.
```python3
side_margin = int((window_w - cup_w * block) / 2)
top_margin = window_h - (cup_h * block) - 5
```
Теперь определяем цвета, что будут использоваться в дальнейшем.
```python3
colors = ((0, 0, 225), (0, 225, 0), (225, 0, 0), (225, 225, 0))
lightcolors = ((30, 30, 255), (50, 255, 50), (255, 30, 30), (255, 255, 30))

white, gray, black  = (255, 255, 255), (185, 185, 185), (0, 0, 0)
brd_color, bg_color, txt_color, title_color, info_color = white, black, white, colors[3], colors[0]
```
Сейчас нужно задать сами фигуры. Как в оригинальной игре они должны двигаться и поворачиваться на 90 градусов. Для этого сделаем область поворота 5*5, где "o" будет обозначать пустую ечейку. Так же мы должны задать форму данной фигуры после поворота. Поэтому пропишем все варианты каждого объекта(S, Z, J, L, I, O, T)
```python3
fig_w, fig_h = 5, 5
empty = 'o'

figures = {'S': [['ooooo',
                  'ooooo',
                  'ooxxo',
                  'oxxoo',
                  'ooooo'],
                 ['ooooo',
                  'ooxoo',
                  'ooxxo',
                  'oooxo',
                  'ooooo']],
           'Z': [['ooooo',
                  'ooooo',
                  'oxxoo',
                  'ooxxo',
                  'ooooo'],
                 ['ooooo',
                  'ooxoo',
                  'oxxoo',
                  'oxooo',
                  'ooooo']],
           'J': [['ooooo',
                  'oxooo',
                  'oxxxo',
                  'ooooo',
                  'ooooo'],
                 ['ooooo',
                  'ooxxo',
                  'ooxoo',
                  'ooxoo',
                  'ooooo'],
                 ['ooooo',
                  'ooooo',
                  'oxxxo',
                  'oooxo',
                  'ooooo'],
                 ['ooooo',
                  'ooxoo',
                  'ooxoo',
                  'oxxoo',
                  'ooooo']],
           'L': [['ooooo',
                  'oooxo',
                  'oxxxo',
                  'ooooo',
                  'ooooo'],
                 ['ooooo',
                  'ooxoo',
                  'ooxoo',
                  'ooxxo',
                  'ooooo'],
                 ['ooooo',
                  'ooooo',
                  'oxxxo',
                  'oxooo',
                  'ooooo'],
                 ['ooooo',
                  'oxxoo',
                  'ooxoo',
                  'ooxoo',
                  'ooooo']],
           'I': [['ooxoo',
                  'ooxoo',
                  'ooxoo',
                  'ooxoo',
                  'ooooo'],
                 ['ooooo',
                  'ooooo',
                  'xxxxo',
                  'ooooo',
                  'ooooo']],
           'O': [['ooooo',
                  'ooooo',
                  'oxxoo',
                  'oxxoo',
                  'ooooo']],
           'T': [['ooooo',
                  'ooxoo',
                  'oxxxo',
                  'ooooo',
                  'ooooo'],
                 ['ooooo',
                  'ooxoo',
                  'ooxxo',
                  'ooxoo',
                  'ooooo'],
                 ['ooooo',
                  'ooooo',
                  'oxxxo',
                  'ooxoo',s
                  'ooooo'],
                 ['ooooo',
                  'ooxoo',
                  'oxxoo',
                  'ooxoo',
                  'ooooo']]}
```
Для паузы мы используем метод, который создаёт доп. поверхность с и последующую заливку экрана паузы цветом с наложением на поверхность окна игры:
```python3
def pauseScreen():
        pause = pg.Surface((600, 500), pg.SRCALPHA)   
        pause.fill((0, 0, 255, 127))                        
        display_surf.blit(pause, (0, 0))
```
### Main часть
Здесь вводим несколько доп констант, начало игры, её запуск и другие глобальные части.
```python3
def main():
    global fps_clock, display_surf, basic_font, big_font
    pg.init()
    fps_clock = pg.time.Clock()
    display_surf = pg.display.set_mode((window_w, window_h))
    basic_font = pg.font.SysFont('arial', 20)
    big_font = pg.font.SysFont('verdana', 45)
    pg.display.set_caption('Тетрис Lite')
    showText('Тетрис Lite')
    while True:
        runTetris()
        pauseScreen()
        showText('Игра закончена')
```

Сам процесс игры(сперва задаётся пустое окно emptycup и движения устанавливаютс на False. Значения меняется по мере отклика клавиатуры и возможности действия):
```python3
def runTetris():
    cup = emptycup()
    last_move_down = time.time()
    last_side_move = time.time()
    last_fall = time.time()
    going_down = False 
    going_left = False
    going_right = False
    points = 0
    level, fall_speed = calcSpeed(points)
    fallingFig = getNewFig()
    nextFig = getNewFig()
```
Нужно отобразить события, фигуры и их движение.
```python3
    while True: 
        if fallingFig == None:
            fallingFig = nextFig
            nextFig = getNewFig()
            last_fall = time.time()
            if not checkPos(cup, fallingFig):
                return
        quitGame()
```
Когда фигура приземляется, она принимает значчение None и nextFig становится fallingFig. Если поле будет заполнено, то сработает функция checkPos()

Сейчас расписываем команды для взаимодействия с клавиатурой. В обычном состоянии все клавишии принимают значение False.
```python3
        for event in pg.event.get(): 
            if event.type == KEYUP:
                if event.key == K_SPACE:
                    pauseScreen()
                    showText('Пауза')
                    last_fall = time.time()
                    last_move_down = time.time()
                    last_side_move = time.time()
                elif event.key == K_LEFT:
                    going_left = False
                elif event.key == K_RIGHT:
                    going_right = False
                elif event.key == K_DOWN:
                    going_down = False

            elif event.type == KEYDOWN:
                if event.key == K_LEFT and checkPos(cup, fallingFig, adjX=-1):
                    fallingFig['x'] -= 1
                    going_left = True
                    going_right = False
                    last_side_move = time.time()

                elif event.key == K_RIGHT and checkPos(cup, fallingFig, adjX=1):
                    fallingFig['x'] += 1
                    going_right = True
                    going_left = False
                    last_side_move = time.time()

                elif event.key == K_UP:
                    fallingFig['rotation'] = (fallingFig['rotation'] + 1) % len(figures[fallingFig['shape']])
                    if not checkPos(cup, fallingFig):
                        fallingFig['rotation'] = (fallingFig['rotation'] - 1) % len(figures[fallingFig['shape']])

                elif event.key == K_DOWN:
                    going_down = True
                    if checkPos(cup, fallingFig, adjY=1):
                        fallingFig['y'] += 1
                    last_move_down = time.time()
                    
                elif event.key == K_RETURN:
                    going_down = False
                    going_left = False
                    going_right = False
                    for i in range(1, cup_h):
                        if not checkPos(cup, fallingFig, adjY=i):
                            break
                    fallingFig['y'] += i - 1
```
Надо так же задать, если игрок будет удерживать клавиши:
```python3
        if (going_left or going_right) and time.time() - last_side_move > side_freq:
            if going_left and checkPos(cup, fallingFig, adjX=-1):
                fallingFig['x'] -= 1
            elif going_right and checkPos(cup, fallingFig, adjX=1):
                fallingFig['x'] += 1
            last_side_move = time.time()

        if going_down and time.time() - last_move_down > down_freq and checkPos(cup, fallingFig, adjY=1):
            fallingFig['y'] += 1
            last_move_down = time.time()
```
Возможен вариант, когда Игрок ничего не трогает. Это тоже надо расписать.
```python3
        if time.time() - last_fall > fall_speed:       
            if not checkPos(cup, fallingFig, adjY=1):
                addToCup(cup, fallingFig)
                points += clearCompleted(cup)
                level, fall_speed = calcSpeed(points)
                fallingFig = None
            else:
                fallingFig['y'] += 1
                last_fall = time.time()
```
Экран должен обновлятся во время игры:
```python3
        display_surf.fill(bg_color)
        drawTitle()
        gamecup(cup)
        drawInfo(points, level)
        drawnextFig(nextFig)
        if fallingFig != None:
            drawFig(fallingFig)
        pg.display.update()
        fps_clock.tick(fps)
```
Функция txtObjects() принимает текст, шрифт и цвет, и с помощью метода render() возвращает готовые объекты Surface (поверхность) и Rect (прямоугольник). Эти объекты в дальнейшем обрабатываются методом blitв функции showText(), выводящей информационные надписи и название игры.

Выход из игры обеспечивает функция stopGame(), в которой используется sys.exit() из импортированного в начале кода модуля sys.

```python3
def txtObjects(text, font, color):
    surf = font.render(text, True, color)
    return surf, surf.get_rect()


def stopGame():
    pg.quit()
    sys.exit()


def checkKeys():
    quitGame()

    for event in pg.event.get([KEYDOWN, KEYUP]):
        if event.type == KEYDOWN:
            continue
        return event.key
    return None


def showText(text):
    titleSurf, titleRect = txtObjects(text, big_font, title_color)
    titleRect.center = (int(window_w / 2) - 3, int(window_h / 2) - 3)
    display_surf.blit(titleSurf, titleRect)
   
    pressKeySurf, pressKeyRect = txtObjects('Нажмите любую клавишу для продолжения', basic_font, title_color)
    pressKeyRect.center = (int(window_w / 2), int(window_h / 2) + 100)
    display_surf.blit(pressKeySurf, pressKeyRect)

    while checkKeys() == None:
        pg.display.update()
        fps_clock.tick()


def quitGame():
    for event in pg.event.get(QUIT):
        stopGame() 
    for event in pg.event.get(KEYUP): 
        if event.key == K_ESCAPE:
            stopGame() 
        pg.event.post(event) 


def calcSpeed(points):
    level = int(points / 10) + 1
    fall_speed = 0.27 - (level * 0.02)
    return level, fall_speed

def getNewFig():
    shape = random.choice(list(figures.keys()))
    newFigure = {'shape': shape,
                'rotation': random.randint(0, len(figures[shape]) - 1),
                'x': int(cup_w / 2) - int(fig_w / 2),
                'y': -2, 
                'color': random.randint(0, len(colors)-1)}
    return newFigure


def addToCup(cup, fig):
    for x in range(fig_w):
        for y in range(fig_h):
            if figures[fig['shape']][fig['rotation']][y][x] != empty:
                cup[x + fig['x']][y + fig['y']] = fig['color']

```
Пустая область создается функцией emptycup():
```python3
def emptycup():
    cup = []
    for i in range(cup_w):
        cup.append([empty] * cup_h)
    return cup
```
В игре заполненные ряды должны удалятся и с этим работает clearCompleted() и isCompleted(). После удаления переносим все ряды и нулевой заполняем значением empty.
```python3
def incup(x, y):
    return x >= 0 and x < cup_w and y < cup_h


def checkPos(cup, fig, adjX=0, adjY=0):
    for x in range(fig_w):
        for y in range(fig_h):
            abovecup = y + fig['y'] + adjY < 0
            if abovecup or figures[fig['shape']][fig['rotation']][y][x] == empty:
                continue
            if not incup(x + fig['x'] + adjX, y + fig['y'] + adjY):
                return False
            if cup[x + fig['x'] + adjX][y + fig['y'] + adjY] != empty:
                return False
    return True

def isCompleted(cup, y):
    for x in range(cup_w):
        if cup[x][y] == empty:
            return False
    return True


def clearCompleted(cup):
    removed_lines = 0
    y = cup_h - 1 
    while y >= 0:
        if isCompleted(cup, y):
           for pushDownY in range(y, 0, -1):
                for x in range(cup_w):
                    cup[x][pushDownY] = cup[x][pushDownY-1]
           for x in range(cup_w):
                cup[x][0] = empty
           removed_lines += 1
        else:
            y -= 1 
    return removed_lines
```

Теперь созаём фигуры, по 4 ячейки каждая.
Для рисования блоков используются примитивы rect (прямоугольник) и circle (круг). При желании верхний квадрат можно конвертировать в поверхность (Surface), после чего наложить на эту поверхность изображение или текстовый символ. Функция drawBlock()также используется в drawnextFig() для вывода следующей фигуры справа от игрового поля.
```python3
def convertCoords(block_x, block_y):
    return (side_margin + (block_x * block)), (top_margin + (block_y * block))


def drawBlock(block_x, block_y, color, pixelx=None, pixely=None):
    if color == empty:
        return
    if pixelx == None and pixely == None:
        pixelx, pixely = convertCoords(block_x, block_y)
    pg.draw.rect(display_surf, colors[color], (pixelx + 1, pixely + 1, block - 1, block - 1), 0, 3)
    pg.draw.rect(display_surf, lightcolors[color], (pixelx + 1, pixely + 1, block - 4, block - 4), 0, 3)
    pg.draw.circle(display_surf, colors[color], (pixelx + block / 2, pixely + block / 2), 5)
    
def gamecup(cup):
    pg.draw.rect(display_surf, brd_color, (side_margin - 4, top_margin - 4, (cup_w * block) + 8, (cup_h * block) + 8), 5)

    pg.draw.rect(display_surf, bg_color, (side_margin, top_margin, block * cup_w, block * cup_h))
    for x in range(cup_w):
        for y in range(cup_h):
            drawBlock(x, y, cup[x][y])

def drawTitle():
    titleSurf = big_font.render('Тетрис Lite', True, title_color)
    titleRect = titleSurf.get_rect()
    titleRect.topleft = (window_w - 425, 30)
    display_surf.blit(titleSurf, titleRect)
    

def drawInfo(points, level):

    pointsSurf = basic_font.render(f'Баллы: {points}', True, txt_color)
    pointsRect = pointsSurf.get_rect()
    pointsRect.topleft = (window_w - 550, 180)
    display_surf.blit(pointsSurf, pointsRect)

    levelSurf = basic_font.render(f'Уровень: {level}', True, txt_color)
    levelRect = levelSurf.get_rect()
    levelRect.topleft = (window_w - 550, 250)
    display_surf.blit(levelSurf, levelRect)

    pausebSurf = basic_font.render('Пауза: пробел', True, info_color)
    pausebRect = pausebSurf.get_rect()
    pausebRect.topleft = (window_w - 550, 420)
    display_surf.blit(pausebSurf, pausebRect)
    
    escbSurf = basic_font.render('Выход: Esc', True, info_color)
    escbRect = escbSurf.get_rect()
    escbRect.topleft = (window_w - 550, 450)
    display_surf.blit(escbSurf, escbRect)

def drawFig(fig, pixelx=None, pixely=None):
    figToDraw = figures[fig['shape']][fig['rotation']]
    if pixelx == None and pixely == None:    
        pixelx, pixely = convertCoords(fig['x'], fig['y'])

    for x in range(fig_w):
        for y in range(fig_h):
            if figToDraw[y][x] != empty:
                drawBlock(None, None, fig['color'], pixelx + (x * block), pixely + (y * block))
```
Для удобства создаём картинку следующей фигуры:
```python3
def drawnextFig(fig):
    nextSurf = basic_font.render('Следующая:', True, txt_color)
    nextRect = nextSurf.get_rect()
    nextRect.topleft = (window_w - 150, 180)
    display_surf.blit(nextSurf, nextRect)
    drawFig(fig, pixelx=window_w-150, pixely=230)
```
Осталось лишь задать конец кода
```python3
if __name__ == '__main__':
    main()
```
