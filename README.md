# trenerslov3
import pygame
import sys
import sqlite3
import random
import pygame_gui

conn = sqlite3.connect('word.db')
c = conn.cursor()

c.execute('''CREATE TABLE IF NOT EXISTS words (words text, translation text)''')

word = [('hello', 'привет'), ('world', 'мир'), ('pear', 'груша'),('cherry','вишня'),('lemon','лимон'),('strawberry','клубника'),('i like pear','я люблю грушу'),('fruits','фрукты'),('i like fruits','я люблю фрукты')]
c.executemany('INSERT INTO words VALUES (?,?)', word)
conn.commit()

def get_random_word():
    c.execute('SELECT * FROM words ORDER BY RANDOM() LIMIT 1')
    return c.fetchone()

def check_translation(word, user_translation):
    c.execute('SELECT * FROM words WHERE words=?', (word,))
    row = c.fetchone()
    if row:
        correct_translation = row[1]
        return correct_translation == user_translation
    else:
        return False

def main():
    pygame.init()

    size = (700, 500)
    screen = pygame.display.set_mode(size)

    pygame.display.set_caption("Тренер слов")

    manager = pygame_gui.UIManager(size)

    button_sound = pygame.mixer.Sound(r'C:\Users\Ayaulym\Downloads\rclick-13693.WAV')

    green_icon = pygame.transform.scale(pygame.image.load(r'C:\Users\Ayaulym\Downloads\green2.png'), (30, 30))
    red_icon = pygame.transform.scale(pygame.image.load(r'C:\Users\Ayaulym\Downloads\red3.png'), (30, 30))

    icon_margin = 10
    green_icon_rect = green_icon.get_rect(topleft=(icon_margin, icon_margin))
    red_icon_rect = red_icon.get_rect(topleft=(icon_margin, green_icon_rect.bottom + icon_margin))

    background_image = pygame.image.load(r'C:\Users\Ayaulym\Downloads\image071-44.jpg')
    background_image = pygame.transform.scale(background_image, (950, 800))

    background_rect = background_image.get_rect()
    background_rect.center = (size[0] // 2, size[1] // 2)

    word = get_random_word()[0]

    button_color = (100, 100, 255)
    text_color = (255, 255, 255)
    background_color = (50, 50, 50)

    button = pygame_gui.elements.UIButton(relative_rect=pygame.Rect((300, 400), (100, 50)), text='Проверить', manager=manager)
    text_entry = pygame_gui.elements.UITextEntryLine(relative_rect=pygame.Rect((225, 350), (250, 50)), manager=manager)
    word_label = pygame_gui.elements.UILabel(relative_rect=pygame.Rect((220, 300), (250, 50)), text=word, manager=manager)
    result_label = pygame_gui.elements.UILabel(relative_rect=pygame.Rect((275, 450), (250, 50)), text='', manager=manager)

    correct_count_label = pygame_gui.elements.UILabel(relative_rect=pygame.Rect((10, 10), (200, 30)), text=f"Правильные: 0", manager=manager)
    incorrect_count_label = pygame_gui.elements.UILabel(relative_rect=pygame.Rect((10, 50), (200, 30)), text=f"Неправильные: 0", manager=manager)

    clock = pygame.time.Clock()

    green_count = 0
    red_count = 0

    while True:
        time_delta = clock.tick(60) / 1000.0
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()

            manager.process_events(event)

            if event.type != pygame.USEREVENT:
                continue
            if event.user_type == pygame_gui.UI_BUTTON_PRESSED:
                if event.ui_element == button:
                    button_sound.play()
                    if check_translation(word, text_entry.get_text()):

                        green_count += 1
                        correct_count_label.set_text(f"Правильные: {green_count}")
                        word = get_random_word()[0]
                        word_label.set_text(word)
                    else:

                        red_count += 1
                        incorrect_count_label.set_text(f"Неправильные: {red_count}")

        manager.update(time_delta)

        screen.blit(background_image, background_rect)

        screen.blit(green_icon, green_icon_rect)
        screen.blit(red_icon, red_icon_rect)

        manager.draw_ui(screen)

        pygame.display.flip()

if __name__ == "__main__":
    main()
