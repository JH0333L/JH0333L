import pygame as pg
import random as rd
import sys

pg.init()
pg.mixer.init()

win = pg.display.set_mode((400, 400), pg.SCALED | pg.FULLSCREEN)
delay = 200

eat_sound = pg.mixer.Sound('snake pro/powerup.wav')
crash_sound = pg.mixer.Sound('snake pro/colicion.wav')

pg.mixer.music.load('snake pro/vamonosdefiesta.wav')
pg.mixer.music.set_volume(0.5)
pg.mixer.music.play(-1, 0.0)

class CommonFeatures:
    def __init__(self, x=0, y=0):
        self._x = x
        self._y = y

    def get_x(self):
        return self._x

    def set_x(self, value):
        self._x = value

    def get_y(self):
        return self._y

    def set_y(self, value):
        self._y = value

class Rect(CommonFeatures):
    def __init__(self, x=0, y=0):
        super().__init__(x, y)
        self.vel = 20
        self.dir = 0
        self.next_dir = 0

    def movement(self):
        keys = pg.key.get_pressed()

        if keys[pg.K_LEFT] and self.dir != 0:
            self.next_dir = 1
        elif keys[pg.K_RIGHT] and self.dir != 1:
            self.next_dir = 0
        elif keys[pg.K_UP] and self.dir != 3:
            self.next_dir = 2
        elif keys[pg.K_DOWN] and self.dir != 2:
            self.next_dir = 3

        self.dir = self.next_dir

        if self.dir == 0:
            self._x += self.vel
        elif self.dir == 1:
            self._x -= self.vel
        elif self.dir == 2:
            self._y -= self.vel
        elif self.dir == 3:
            self._y += self.vel

    def draw(self, color='white'):
        pg.draw.rect(win, color, (self._x, self._y, 20, 20))

class Snake:
    def __init__(self):
        self.body = [Rect(100, 100), Rect(80, 100), Rect(60, 100)]

    def update_body(self):
        for i in range(len(self.body) - 1, 0, -1):
            self.body[i].set_x(self.body[i - 1].get_x())
            self.body[i].set_y(self.body[i - 1].get_y())
        self.body[0].movement()

    def grow(self):
        last_segment = self.body[-1]
        new_segment = Rect(last_segment.get_x(), last_segment.get_y())
        self.body.append(new_segment)

    def draw(self):
        for segment in self.body:
            segment.draw()

    def check_self_collision(self):
        for segment in self.body[1:]:
            if self.body[0].get_x() == segment.get_x() and self.body[0].get_y() == segment.get_y():
                return True
        return False

class Apple(CommonFeatures):
    def __init__(self):
        super().__init__(rd.randrange(20) * 20, rd.randrange(20) * 20)

    def draw(self):
        pg.draw.rect(win, 'red', (self._x, self._y, 20, 20))

class Game:
    def __init__(self):
        self.snake = Snake()
        self.apple = Apple()
        self.delay = 200

    def check_collision(self):
        self.handle_boundaries()

        if self.snake.body[0].get_x() == self.apple.get_x() and self.snake.body[0].get_y() == self.apple.get_y():
            eat_sound.play()
            self.apple = Apple()
            self.snake.grow()
            if self.delay > 80:
                self.delay -= 3

        if self.snake.check_self_collision():
            crash_sound.play()
            self.show_game_over()

    def handle_boundaries(self):
        head = self.snake.body[0]
        if head.get_x() > 380:
            head.set_x(0)
        elif head.get_x() < 0:
            head.set_x(380)

        if head.get_y() < 0:
            head.set_y(380)
        elif head.get_y() > 380:
            head.set_y(0)

    def reset_game(self):
        self.snake = Snake()
        self.apple = Apple()
        self.delay = 200

    def show_game_over(self):
        pg.mixer.music.stop()
        font = pg.font.SysFont("Times New Roman", 40)
        text = font.render("Game Over", True, "red")
        win.fill("black")
        win.blit(text, (120, 150))

        options_font = pg.font.SysFont("Times New Roman", 25)
        options_text = options_font.render("Press R to Restart or Q to Quit", True, "white")
        win.blit(options_text, (60, 200))

        pg.display.flip()

        waiting = True
        while waiting:
            for event in pg.event.get():
                if event.type == pg.QUIT:
                    pg.quit()
                    sys.exit()
                if event.type == pg.KEYDOWN:
                    if event.key == pg.K_r:
                        pg.mixer.music.play(-1, 0.0)
                        self.reset_game()
                        waiting = False
                    elif event.key == pg.K_q:
                        pg.quit()
                        sys.exit()

    def draw_game(self):
        win.fill('darkgray')
        self.apple.draw()
        self.snake.draw()
        pg.display.flip()

    def game_loop(self):
        while True:
            for event in pg.event.get():
                if event.type == pg.QUIT or (event.type == pg.KEYDOWN and event.key == pg.K_ESCAPE):
                    sys.exit()

            self.snake.update_body()
            self.check_collision()
            self.draw_game()
            pg.time.wait(self.delay)

game = Game()
game.game_loop()
