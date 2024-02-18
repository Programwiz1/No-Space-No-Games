# No-Space-No-Games
import sys
import pygame
import random
import time
import math

# Initialize Pygame
pygame.init()

# Constants
WIDTH, HEIGHT = 800, 600
BACKGROUND_COLOR = (0, 0, 0)
BIRD_SPEED = 5
FIREBALL_SPEED = 3
LAVA_SPEED = 1
GRAVITY = 0.15
JUMP_FORCE = 5
HORIZONTAL_SPEED = 5  # Horizontal speed for left and right movement
COIN_INITIAL_COUNT = 10  # Initial number of coins
COLLECT_DISTANCE = 30  # Distance within which the bird collects the coin
HEALING_POTION_TIME = 20  # Duration of healing potion effect in seconds
FPS = 60

screen = pygame.display.set_mode((WIDTH, HEIGHT))

# Load background wallpaper
background_image = pygame.image.load('background_image.gif')
background_image = pygame.transform.scale(background_image, (WIDTH, HEIGHT))

# Load sprites
bird_sprite = pygame.image.load('bird.png')
bird_sprite = pygame.transform.scale(bird_sprite, (50, 50))

fireball_sprite = pygame.image.load('fireball.png')
fireball_sprite = pygame.transform.scale(fireball_sprite, (30, 30))

lava_sprite = pygame.image.load('lava.png')
lava_sprite = pygame.transform.scale(lava_sprite, (WIDTH, 50))

heart_sprite = pygame.image.load('heart.png')
heart_sprite = pygame.transform.scale(heart_sprite, (30, 30))

coin_sprite = pygame.image.load('coin.png')
coin_sprite = pygame.transform.scale(coin_sprite, (30, 30))

healing_potion_sprite = pygame.image.load('healing_potion.png')
healing_potion_sprite = pygame.transform.scale(healing_potion_sprite, (30, 30))

# Function to create fireballs
def create_fireball():
    fireball = {
        'pos': [random.randint(0, WIDTH), 0],
        'sprite': fireball_sprite
    }
    fireballs.append(fireball)

# Function to create coins
def create_coins():
    coins.clear()
    for _ in range(COIN_INITIAL_COUNT):
        coin = {
            'pos': [random.randint(0, WIDTH), random.randint(0, HEIGHT)],
            'sprite': coin_sprite
        }
        coins.append(coin)

# Function to display game over message
def display_game_over():
    font = pygame.font.Font(None, 72)
    text = font.render("GAME OVER", True, (255, 0, 0))
    text_rect = text.get_rect(center=(WIDTH // 2, HEIGHT // 2))
    screen.blit(text, text_rect)

    replay_button = pygame.Rect(WIDTH // 2 - 100, HEIGHT // 2 + 100, 200, 50)
    pygame.draw.rect(screen, (0, 255, 0), replay_button)
    font = pygame.font.Font(None, 36)
    text = font.render("REPLAY", True, (0, 0, 0))
    text_rect = text.get_rect(center=replay_button.center)
    screen.blit(text, text_rect)

    return replay_button

# Function to display high score
def display_high_score(high_score):
    rounded_score = round(high_score, 2)  # Round to two decimal places
    font = pygame.font.Font(None, 36)
    text = font.render(f'High Score: {rounded_score}', True, (255, 255, 255))
    screen.blit(text, (10, 50))

# Main menu screen
def main_menu():

    start_button = pygame.Rect(WIDTH // 2 - 100, HEIGHT // 2 - 50, 200, 50)
    store_button = pygame.Rect(WIDTH // 2 - 100, HEIGHT // 2 + 50, 200, 50)

    font = pygame.font.Font(None, 36)
    text = font.render("START GAME", True, (255, 255, 255))
    text_rect = text.get_rect(center=start_button.center)
    screen.blit(text, text_rect)

    text = font.render("STORE", True, (255, 255, 255))
    text_rect = text.get_rect(center=store_button.center)
    screen.blit(text, text_rect)

    return start_button, store_button

# Main game loop
def main_loop():
    global bird_lives, bird_pos, bird_vel, fireballs, lava_height, coins, high_score, healing_potion_start_time, lava_stopped

    clock = pygame.time.Clock()
    start_time = time.time()
    high_score = 0
    healing_potion_start_time = 0
    lava_stopped = False

    running = True
    game_over = False
    replay_button = None
    in_menu = True

    while running:
        if in_menu:
            start_button, store_button = main_menu()

        # Event handling
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()

            if in_menu:
                if event.type == pygame.MOUSEBUTTONDOWN:
                    mouse_pos = pygame.mouse.get_pos()
                    if start_button.collidepoint(mouse_pos):
                        in_menu = False
                        bird_lives = 3
                        bird_pos = [WIDTH // 2, HEIGHT // 2]
                        bird_vel = [0, 0]
                        fireballs = []
                        lava_height = 0
                        coins = []
                        create_coins()
                        start_time = time.time()
                    elif store_button.collidepoint(mouse_pos):
                        # Implement store functionality here
                        pass

            # Key press events
            if not in_menu and event.type == pygame.KEYDOWN:
                if event.key == pygame.K_UP:
                    bird_jump()
                elif event.key == pygame.K_LEFT:
                    bird_vel[0] = -HORIZONTAL_SPEED
                elif event.key == pygame.K_RIGHT:
                    bird_vel[0] = HORIZONTAL_SPEED
            elif not in_menu and event.type == pygame.KEYUP:
                if event.key == pygame.K_LEFT or event.key == pygame.K_RIGHT:
                    bird_vel[0] = 0

        if not in_menu:
            if not game_over:
                # Apply gravity to bird's velocity
                bird_vel[1] += GRAVITY

                # Update bird position based on velocity
                bird_pos[0] += bird_vel[0]
                bird_pos[1] += bird_vel[1]

                # Keep the bird within the screen bounds
                bird_pos[0] = max(0, min(WIDTH - bird_sprite.get_width(), bird_pos[0]))
                bird_pos[1] = max(0, min(HEIGHT - bird_sprite.get_height(), bird_pos[1]))

                # Check for collision with lava
                if bird_pos[1] + bird_sprite.get_height() > HEIGHT - lava_height:
                    game_over = True

                # Check if bird goes out of screen
                if bird_pos[1] > HEIGHT:
                    game_over = True

                # Update game state
                for fireball in fireballs:
                    fireball['pos'][1] += FIREBALL_SPEED
                    if bird_pos[0] < fireball['pos'][0] < bird_pos[0] + bird_sprite.get_width() and \
                    bird_pos[1] < fireball['pos'][1] < bird_pos[1] + bird_sprite.get_height():
                        bird_lives -= 1
                        fireballs.remove(fireball)

                for coin in coins:
                    if math.dist(coin['pos'], bird_pos) < COLLECT_DISTANCE:
                        bird_lives += 1
                        coins.remove(coin)
                        create_coins()  # Spawn a new coin when one is collected

                # Update lava height
                lava_height += LAVA_SPEED

                # Spawn fireballs at the end of each level
                if bird_pos[1] - bird_sprite.get_height() <= 0:
                    create_fireball()
                    lava_height = 0

                # Check if all lives are lost
                if bird_lives <= 0:
                    game_over = True
                    current_score = time.time() - start_time
                    if current_score > high_score:
                        high_score = current_score

                # Check if healing potion effect has ended
                if time.time() - healing_potion_start_time >= HEALING_POTION_TIME:
                    lava_stopped = False

            # Draw everything
            screen.blit(background_image, (0, 0))
            screen.blit(lava_sprite, (0, HEIGHT - lava_height))
            for fireball in fireballs:
                screen.blit(fireball['sprite'], fireball['pos'])
            for coin in coins:
                screen.blit(coin['sprite'], coin['pos'])
            screen.blit(bird_sprite, bird_pos)

            # Draw hearts for bird lives
            for i in range(bird_lives):
                heart_x = WIDTH - 40 - i * 40
                heart_y = 10
                screen.blit(heart_sprite, (heart_x, heart_y))

            # Draw healing potion
            if bird_lives > 5:
                screen.blit(healing_potion_sprite, (WIDTH - 40, 50))

            # Display high score
            display_high_score(high_score)

            # If game over, display game over message and replay button
            if game_over:
                replay_button = display_game_over()

        pygame.display.flip()
        clock.tick(FPS)

# Function to make the bird jump
def bird_jump():
    bird_vel[1] = -JUMP_FORCE

# Call the main loop function to start the game
if __name__ == "__main__":
    bird_lives = 3
    bird_pos = [WIDTH // 2, HEIGHT // 2]
    bird_vel = [0, 0]  # Velocity of the bird [x, y]
    fireballs = []
    lava_height = 0
    coins = []
    create_coins()  # Initial coin creation
    main_loop()
