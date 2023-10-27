import pygame
import sys
import random
import time

pygame.init()

# make window
width, height = 800, 600
window = pygame.display.set_mode((width, height))
pygame.display.set_caption("Pong")

#color
white = (255, 255, 255)
black = (0, 0, 0)


# Paddle class
class Paddle(pygame.sprite.Sprite):
    def __init__(self, color, width, height, position):
        super().__init__()
        self.image = pygame.Surface((width, height))
        self.image.fill(color)
        self.rect = self.image.get_rect(topleft=position)


# BaLL CLASS
class Ball(pygame.sprite.Sprite):
    def __init__(self, color, size, position, speed):
        super().__init__()
        self.image = pygame.Surface((size, size))
        self.image.fill(color)
        self.rect = self.image.get_rect(center=position)
        self.speed = speed
        self.dx = random.choice([-1, 1])
        self.dy = random.choice([-1, 1])

    def update(self):
        self.rect.x += self.speed * self.dx
        self.rect.y += self.speed * self.dy

        # bouncing
        if self.rect.top <= 0 or self.rect.bottom >= height:
            self.dy *= -1


# make paddle and ball
player_paddle = Paddle(white, 15, 100, (10, height // 2 - 50))
opponent_paddle = Paddle(white, 15, 100, (width - 25, height // 2 - 50))
ball = Ball(white, 15, (width // 2, height // 2), 3)  # Set a slower initial speed

# Sprite Groups
all_sprites = pygame.sprite.Group()
all_sprites.add(player_paddle, opponent_paddle, ball)

# Scores
player_score = 0
opponent_score = 0
font = pygame.font.Font(None, 36)

# Main Game Loop
clock = pygame.time.Clock()
running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    keys = pygame.key.get_pressed()
    if keys[pygame.K_UP] and player_paddle.rect.top > 0:
        player_paddle.rect.y -= 7
    if keys[pygame.K_DOWN] and player_paddle.rect.bottom < height:
        player_paddle.rect.y += 7

    # AI for opponent paddle
    if opponent_paddle.rect.centery < ball.rect.centery:
        opponent_paddle.rect.y += 5
    elif opponent_paddle.rect.centery > ball.rect.centery:
        opponent_paddle.rect.y -= 5

    # Update ball position
    ball.update()

    # Ball and paddle collisions
    if pygame.sprite.collide_rect(ball, player_paddle) or pygame.sprite.collide_rect(ball, opponent_paddle):
        ball.dx *= -1

    # Score points
    if ball.rect.left <= 0:
        opponent_score += 1
        if opponent_score >= 5:
            time.sleep(1)
            print("Opponent Wins!")
            running = False
        else:
            ball.rect.center = (width // 2, height // 2)
            ball.dx *= -1

    if ball.rect.right >= width:
        player_score += 1
        if player_score >= 5:
            time.sleep(1)
            print("Player Wins!")
            running = False
        else:
            ball.rect.center = (width // 2, height // 2)
            ball.dx *= -1

    # Draw everything
    window.fill(black)
    pygame.draw.aaline(window, white, (width // 2, 0), (width // 2, height))

    all_sprites.draw(window)
    all_sprites.update()

    # Display scores
    player_text = font.render(f"Player: {player_score}", True, white)
    opponent_text = font.render(f"Opponent: {opponent_score}", True, white)
    window.blit(player_text, (10, 10))
    window.blit(opponent_text, (width - opponent_text.get_width() - 10, 10))

    # Update the display
    pygame.display.flip()

    # Set the frame rate
    clock.tick(60)

# Quit Pygame
pygame.quit()
sys.exit()
