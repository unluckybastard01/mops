import pygame
from pygame import *
from random import randint
    
pygame.init()
pygame.mixer.init()
pygame.font.init() 

WINDOW_WIDTH = 700
WINDOW_HEIGHT = 500
screen = pygame.display.set_mode((WINDOW_WIDTH, WINDOW_HEIGHT))
pygame.display.set_caption("Космическая игра")

background = pygame.image.load("galaxy.jpg").convert()
background = pygame.transform.scale(background, (WINDOW_WIDTH, WINDOW_HEIGHT))

pygame.mixer.music.load("space.ogg")
pygame.mixer.music.play(-1)
pygame.mixer.music.set_volume(0.38)
fire_sound = pygame.mixer.Sound("fire.ogg")

class GameSprite(sprite.Sprite):
    def __init__(self, player_image, player_x, player_y, size_x, size_y, player_speed):
        super().__init__()
        self.image = transform.scale(image.load(player_image), (size_x, size_y))
        self.speed = player_speed
        self.rect = self.image.get_rect()
        self.rect.x = player_x
        self.rect.y = player_y
    def reset(self):
        screen.blit(self.image, (self.rect.x, self.rect.y))

class Player(GameSprite):
    def update(self):
        keys = key.get_pressed()
        if keys[K_LEFT] and self.rect.x > 5:
            self.rect.x -= self.speed
        if keys[K_RIGHT] and self.rect.x < WINDOW_WIDTH - 85:
            self.rect.x += self.speed            
    def fire(self):
        bullet = Bullet("bullet.png", self.rect.centerx - 7, self.rect.top, 15, 20, 15)
        bullets.add(bullet)

class Enemy(GameSprite):
    def update(self):
        global lost 
        self.rect.y += self.speed  
        if self.rect.y > WINDOW_HEIGHT:  
            self.rect.y = 0
            self.rect.x = randint(0, WINDOW_WIDTH - 80)
            lost += 1 


class Asteroid(GameSprite):
    def update(self):
        self.rect.y += self.speed
        if self.rect.y > WINDOW_HEIGHT:
            self.rect.y = -40
            self.rect.x = randint(0, WINDOW_WIDTH - 50)

class Bullet(GameSprite):
    def update(self):
        self.rect.y -= self.speed
        if self.rect.y < 0:
            self.kill() 

player = Player("rocket.png", 300, 400, 80, 80, 5)
bullets = sprite.Group()
enemies = sprite.Group()
asteroids = sprite.Group() 


for _ in range(5):    
    enemy = Enemy("ufo.png", randint(0, WINDOW_WIDTH - 80), -80, 80, 50, randint(1, 3))
    enemies.add(enemy)


for _ in range(3):
    asteroid = Asteroid("asteroid.png", randint(0, WINDOW_WIDTH - 50), -40, 50, 50, randint(1, 2))
    asteroids.add(asteroid)

clock = pygame.time.Clock()
FPS = 60

lost = 0
score = 0


num_fire = 0          
rel_time = False    
reload_start = 0      

stat_font = font.Font(None, 36)
finish_font = font.Font(None, 80)
stat_text_color = (255, 255, 255) 
warning_color = (255, 50, 50) 

win_text = finish_font.render('ПОБЕДА!', True, (0, 255, 0))
lose_text = finish_font.render('ПРОИГРЫШ!', True, (255, 0, 0))

finish = False
running = True

end_time = 0
game_result = ""

while running:
    clock.tick(FPS)

    for event in pygame.event.get():
        if event.type == QUIT:
            running = False
        elif event.type == KEYDOWN:
            if event.key == K_SPACE:
                
                if num_fire < 5 and not rel_time and not finish:
                    num_fire += 1
                    fire_sound.play()
                    player.fire()
                    
                
                if num_fire >= 5 and not rel_time:
                    rel_time = True
                    reload_start = pygame.time.get_ticks() 

    if not finish:
        screen.blit(background, (0, 0))

        lost_text = stat_font.render(f'Пропущено: {lost}', True, stat_text_color)
        screen.blit(lost_text, (10, 50))
        score_text = stat_font.render(f'Счет: {score}', True, stat_text_color)
        screen.blit(score_text, (10, 20))

     
        if rel_time:
            now_time = pygame.time.get_ticks()
            if now_time - reload_start < 3000:
                reload_text = stat_font.render('Перезарядка...', True, warning_color)
                screen.blit(reload_text, (260, 460))
            else:
                num_fire = 0 
                rel_time = False 

       
        ammo_text = stat_font.render(f'Патроны: {5 - num_fire}/5', True, stat_text_color)
        screen.blit(ammo_text, (530, 20))

        player.update()
        enemies.update()
        asteroids.update() 
        bullets.update()

        player.reset()
        enemies.draw(screen)
        asteroids.draw(screen) 
        bullets.draw(screen) 

        
        collides = sprite.groupcollide(enemies, bullets, True, True)
        for c in collides:
            score += 1 
            enemy = Enemy("ufo.png", randint(0, WINDOW_WIDTH - 80), -80, 80, 50, randint(1, 3))
            enemies.add(enemy)

       
        sprite.groupcollide(asteroids, bullets, False, True)

        
        if sprite.spritecollide(player, enemies, False) or sprite.spritecollide(player, asteroids, False) or lost >= 10:
            finish = True
            game_result = "lose"
            end_time = pygame.time.get_ticks()
            
        if score >= 50:
            finish = True
            game_result = "win"
            end_time = pygame.time.get_ticks() 
        
    else:
        if game_result == "win":
            screen.blit(win_text, (240, 200))
        else:
            screen.blit(lose_text, (200, 200))
            
        if pygame.time.get_ticks() - end_time > 3000:
            lost = 0
            score = 0
            num_fire = 0
            rel_time = False
            finish = False
            game_result = ""
            
            enemies.empty()
            asteroids.empty()
            bullets.empty()
            
            player.rect.x = 300
            
            
            for _ in range(5):    
                enemy = Enemy("ufo.png", randint(0, WINDOW_WIDTH - 80), -80, 80, 50, randint(1, 3))
                enemies.add(enemy)
                
            for _ in range(3):
                asteroid = Asteroid("asteroid.png", randint(0, WINDOW_WIDTH - 50), -40, 50, 50, randint(1, 2))
                asteroids.add(asteroid)

    pygame.display.flip()

pygame.quit()
