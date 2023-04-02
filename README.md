# lllllllll
from pygame import *
from random import randint
 
#фоновая музыка
mixer.init()
mixer.music.load('space.ogg')
mixer.music.play()

#нам нужны такие картинки:
img_back = "galaxy.jpg" #фон игры
img_hero = "rocket.png" #герой
img_enemy = "ufo.png" # враг
img_bullet = "bullet.png"
#Создаем окошко
win_width = 700
win_height = 500
display.set_caption("Shooter")
window = display.set_mode((win_width, win_height))
background = transform.scale(image.load(img_back), (win_width, win_height))
lost = 0 
score = 0
font.init()
font2 = font.Font(None,70)
c = 0 


#класс-родитель для других спрайтов
class GameSprite(sprite.Sprite):
 #конструктор класса
    def __init__(self, player_image, player_x, player_y, size_x, size_y, player_speed):
        #Вызываем конструктор класса (Sprite):
        super().__init__()
 
        #каждый спрайт должен хранить свойство image - изображение
        self.image = transform.scale(image.load(player_image), (size_x, size_y))
        self.speed = player_speed
 
        #каждый спрайт должен хранить свойство rect - прямоугольник, в который он вписан
        self.rect = self.image.get_rect()
        self.rect.x = player_x
        self.rect.y = player_y
    #метод, отрисовывающий героя на окне
    def reset(self):
        window.blit(self.image, (self.rect.x, self.rect.y))

g =0
#класс главного игрока
class Player(GameSprite):
   #метод для управления спрайтом стрелками клавиатуры
    def update(self):
        keys = key.get_pressed()
        if keys[K_LEFT] and self.rect.x > 5:
            self.rect.x -= self.speed
        if keys[K_RIGHT] and self.rect.x < win_width - 80:
            self.rect.x += self.speed
 #метод "выстрел" (используем место игрока, чтобы создать там пулю)
    def fire(self):
        global g 
        if g <= 5:
            bullet = Bullet(img_bullet, self.rect.centerx, self.rect.top,15,20,15)
            bullets.add(bullet)
            g = g + 1 

#класс спрайта-врага  
class Enemy(GameSprite):
   #движение врага
    def update(self):
        global lost
        self.rect.y += self.speed

        if self.rect.y > win_height:
            self.rect.x = randint(80,win_width - 80 )
            self.rect.y = -40 
            self.speed = randint(1,5)
            lost = lost +1
            print(lost)

class Bullet(GameSprite):
    def update(self):
        self.rect.y -= self.speed

        if self.rect.y < 0:
            self.kill() 
        
        

        
win = font2.render('YOU WINE',True,(215,215,0))
lose = font2.render('YOU LOSE',True,(215,0,0))





font.init()
font2 = font.Font(None,70)
#создаем спрайты
ship = Player(img_hero, 5, win_height - 100, 80, 100, 10)

monsters = sprite.Group()
bullets = sprite.Group()

for i in range(1, 6):
    monster = Enemy(img_enemy, randint(80, win_width - 80), -40, 80, 50, randint(1, 5))
    monsters.add(monster)
    



#переменная "игра закончилась": как только там True, в основном цикле перестают работать спрайты
finish = False
#Основной цикл игры:
run = True #флаг сбрасывается кнопкой закрытия окна

while run:
   #событие нажатия на кнопку Закрыть
    for e in event.get():
        if e.type == QUIT:
            run = False
    
        elif e.type == KEYDOWN:
            if e.key == K_SPACE:
                ship.fire()
    
    
    if not finish:
       #обновляем фон
        window.blit(background,(0,0))
        if sprite.spritecollide(ship,monsters,False):
            finish = True
     

        collides = sprite.groupcollide(monsters,bullets,True,True)
        
        for c in collides:
            score = score +1 
            monster = Enemy(img_enemy, randint(80, win_width - 80), -40, 80, 50, randint(1, 5))
            monsters.add(monster)
            monster = Enemy(img_enemy, randint(80, win_width - 80), -40, 80, 50, randint(1, 5))
            monsters.add(monster)

        if score >= 10:
            finish = True
            window.blit(win,(200,200))

        if sprite.spritecollide(ship,monsters,False) or lost >= 3:
            finish = True
            window.blit(lose,(200,200)) 

     
       #производим движения спрайтов
        ship.update()
        monsters.update()
        
        text_score = font2.render('счет:'+str(score),True,(255,255,255))
        window.blit(text_score,(10,20))
        
        text_lose = font2.render('пропущено:'+str(lost),True,(255,255,255))
        window.blit(text_lose,(10,50))
       #обновляем их в новом местоположении при каждой итерации цикла
        ship.reset()
        bullets.update()
        monsters.draw(window)
        bullets.draw(window)
        display.update()
        
   #цикл срабатывает каждые 0.05 секунд
    time.delay(50)
