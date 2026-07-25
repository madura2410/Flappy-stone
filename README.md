import pygame
import random
import sys

# ---------------------------------------------------------
# 1. INITIALIZATION & SETUP
# ---------------------------------------------------------
pygame.init()
pygame.font.init()

# Game Window Dimensions
SCREEN_WIDTH = 400
SCREEN_HEIGHT = 600
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("Flappy Bird: Stone Cavern")

clock = pygame.time.Clock()
FPS = 60

# Colors
COLOR_SKY = (30, 30, 45)         # Dark cavern sky
COLOR_BIRD = (255, 200, 0)       # Yellow bird
COLOR_STONE = (100, 105, 115)    # Stone grey
COLOR_STONE_BORDER = (60, 65, 75)# Dark stone outline
COLOR_GROUND = (50, 50, 60)      # Cavern floor
COLOR_WHITE = (255, 255, 255)
COLOR_RED = (220, 60, 60)

FONT = pygame.font.SysFont("Arial", 28, bold=True)

# ---------------------------------------------------------
# 2. GAME VARIABLES & PHYSICS
# ---------------------------------------------------------
GRAVITY = 0.45
JUMP_STRENGTH = -8.0

# Bird variables
bird_x = 80
bird_y = 250
bird_vy = 0
bird_radius = 14

# Stone Obstacle variables
stone_width = 65
stone_gap = 160           # Vertical gap size between top & bottom stones
stone_speed = 3.5          # Leftward scroll speed
spawn_timer = 0
spawn_interval = 90        # Frames between spawning new stone pairs (1.5 seconds)

stones = []                # List to hold active stone dicts: {'x': x, 'top_height': h, 'passed': bool}
score = 0
game_over = False

# ---------------------------------------------------------
# 3. HELPER FUNCTIONS
# ---------------------------------------------------------
def create_stone_pair():
    """Generates a random top stone height and adds a new stone pair."""
    min_height = 50
    max_height = SCREEN_HEIGHT - stone_gap - 100
    top_height = random.randint(min_height, max_height)
    return {'x': SCREEN_WIDTH, 'top_height': top_height, 'passed': False}

def reset_game():
    """Resets all game parameters for a restart."""
    global bird_y, bird_vy, stones, score, game_over, spawn_timer
    bird_y = 250
    bird_vy = 0
    stones = []
    score = 0
    spawn_timer = 0
    game_over = False

# ---------------------------------------------------------
# 4. MAIN GAME LOOP
# ---------------------------------------------------------
while True:
    clock.tick(FPS)

    # --- A. EVENT HANDLING ---
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()

        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE:
                if not game_over:
                    bird_vy = JUMP_STRENGTH
                else:
                    reset_game()

    # --- B. GAME STATE UPDATE ---
    if not game_over:
        # Update Bird Position
        bird_vy += GRAVITY
        bird_y += bird_vy

        # Spawn Stone Obstacles
        spawn_timer += 1
        if spawn_timer >= spawn_interval:
            stones.append(create_stone_pair())
            spawn_timer = 0

        # Move and Update Stones
        bird_rect = pygame.Rect(bird_x - bird_radius, bird_y - bird_radius, bird_radius * 2, bird_radius * 2)

        for stone in stones[:]:
            stone['x'] -= stone_speed

            # Rectangles for Collision Checking
            top_stone_rect = pygame.Rect(stone['x'], 0, stone_width, stone['top_height'])
            bottom_stone_y = stone['top_height'] + stone_gap
            bottom_stone_rect = pygame.Rect(stone['x'], bottom_stone_y, stone_width, SCREEN_HEIGHT - bottom_stone_y)

            # Check Collisions
            if bird_rect.colliderect(top_stone_rect) or bird_rect.colliderect(bottom_stone_rect):
                game_over = True

            # Check Score (Passing the Stone)
            if not stone['passed'] and stone['x'] + stone_width < bird_x:
                score += 1
                stone['passed'] = True

            # Remove off-screen stones
            if stone['x'] < -stone_width:
                stones.remove(stone)

        # Check Ground / Ceiling Collision
        if bird_y + bird_radius >= SCREEN_HEIGHT - 30 or bird_y - bird_radius <= 0:
            game_over = True

    # --- C. DRAWING / RENDERING ---
    screen.fill(COLOR_SKY)

    # Draw Stones
    for stone in stones:
        # Top Stone
        top_rect = pygame.Rect(stone['x'], 0, stone_width, stone['top_height'])
        pygame.draw.rect(screen, COLOR_STONE, top_rect)
        pygame.draw.rect(screen, COLOR_STONE_BORDER, top_rect, 4)  # Border

        # Bottom Stone
        bottom_y = stone['top_height'] + stone_gap
        bottom_rect = pygame.Rect(stone['x'], bottom_y, stone_width, SCREEN_HEIGHT - bottom_y)
        pygame.draw.rect(screen, COLOR_STONE, bottom_rect)
        pygame.draw.rect(screen, COLOR_STONE_BORDER, bottom_rect, 4) # Border

    # Draw Floor Ground
    pygame.draw.rect(screen, COLOR_GROUND, (0, SCREEN_HEIGHT - 30, SCREEN_WIDTH, 30))

    # Draw Bird
    pygame.draw.circle(screen, COLOR_BIRD, (int(bird_x), int(bird_y)), bird_radius)

    # Draw Score
    score_surface = FONT.render(f"Score: {score}", True, COLOR_WHITE)
    screen.blit(score_surface, (15, 15))

    # Game Over Overlay Screen
    if game_over:
        go_surface = FONT.render("GAME OVER", True, COLOR_RED)
        restart_surface = FONT.render("Press SPACE to Restart", True, COLOR_WHITE)
        
        screen.blit(go_surface, (SCREEN_WIDTH // 2 - go_surface.get_width() // 2, 220))
        screen.blit(restart_surface, (SCREEN_WIDTH // 2 - restart_surface.get_width() // 2, 270))

    pygame.display.flip()