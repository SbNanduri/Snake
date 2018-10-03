import pygame
import os
import random

os.environ['SDL_VIDEO_CENTERED'] = '1'

pygame.init()

game_exit = False

mouse_1 = False
mouse_1_states = []  # Tracks what state mouse_1 was in in the previous loop
mouse_2 = False
mouse_2_states = []
letter_keys = {'w': False,
               'a': False,
               's': False,
               'd': False,
               'up': False,
               'left': False,
               'down': False,
               'right': False,
               'shift': False}

# Colours

off_white = (230, 230, 230)
white = (255, 255, 255)
black = (0, 0, 0)
grey = (60, 60, 60)
light_grey = (135, 135, 135)
red = (245, 0, 0)
green = (0, 215, 0)
blue = (0, 0, 245)
light_blue = (0, 191, 255)
purple = (150, 0, 175)
pink = (245, 105, 180)
turquoise = (0, 206, 209)
yellow = (255, 255, 0)
ochre = (204, 119, 34)
gold = (255, 215, 0)
brown = (155, 83, 19)
light_brown = (255, 218, 185)

# Official Resolution is display_width = 800 display_height = 600

display_width = 800
display_height = 600
resolution = (display_width, display_height)

Display = pygame.display.set_mode(resolution)

pygame.display.set_caption('Snake')

clock = pygame.time.Clock()

# Fonts
small_font_size = display_height // 25
med_font_size = display_height // 12
large_font_size = display_height // 5

small_font = pygame.font.SysFont('Roboto', small_font_size)
med_font = pygame.font.SysFont('Roboto', med_font_size)
large_font = pygame.font.SysFont('Roboto', large_font_size)

# Images

original_snake_imgs = {'Head': pygame.image.load('Snek.png'),
                       'Body': pygame.image.load('Snek Body.png'),
                       'Tail': pygame.image.load('Snek Tail.png'),
                       'Down Left': pygame.image.load('Snek Body Down Left.png'),
                       'Down Right': pygame.image.load('Snek Body Down Right.png'),
                       'Left Down': pygame.image.load('Snek Body Left Down.png'),
                       'Left Up': pygame.image.load('Snek Body Left Up.png'),
                       'Right Down': pygame.image.load('Snek Body Right Down.png'),
                       'Right Up': pygame.image.load('Snek Body Right Up.png'),
                       'Up Left': pygame.image.load('Snek Body Up Left.png'),
                       'Up Right': pygame.image.load('Snek Body Up Right.png')
                       }

snake_imgs = {'Head': pygame.image.load('Snek.png'),
                       'Body': pygame.image.load('Snek Body.png'),
                       'Tail': pygame.image.load('Snek Tail.png'),
                       'downleft': pygame.image.load('Snek Body Down Left.png'),
                       'downright': pygame.image.load('Snek Body Down Right.png'),
                       'leftdown': pygame.image.load('Snek Body Left Down.png'),
                       'leftup': pygame.image.load('Snek Body Left Up.png'),
                       'rightdown': pygame.image.load('Snek Body Right Down.png'),
                       'rightup': pygame.image.load('Snek Body Right Up.png'),
                       'upleft': pygame.image.load('Snek Body Up Left.png'),
                       'upright': pygame.image.load('Snek Body Up Right.png')
                       }

food_imgs = {'Blueberries': pygame.image.load('blueberries.png'),
             'Bagel': pygame.image.load('Bagel.png')}

snake_width = 20

for key, value in food_imgs.items():
	food_imgs[key] = pygame.transform.scale(value, (snake_width, snake_width))


class Entities:
	def __init__(self):
		self._x = snake_width
		self._y = snake_width
		self._coords = (self._x, self._y)

		self._part = 'Body'
		self.img = original_snake_imgs[self.part]

		self.direction = 'right'

		self.corner = False

	@property
	def x(self):
		return self._x

	@x.setter
	def x(self, new_x):
		self._x = new_x
		self._coords = (new_x, self._y)

	@property
	def y(self):
		return self._y

	@y.setter
	def y(self, new_y):
		self._y = new_y
		self._coords = (self._x, new_y)

	@property
	def coords(self):
		return self._coords

	@coords.setter
	def coords(self, new_coords):
		self._coords = new_coords
		self._x = new_coords[0]
		self._y = new_coords[1]

	@property
	def part(self):
		return self._part

	@part.setter
	def part(self, new_part):
		self.img = original_snake_imgs[new_part]
		self._part = new_part

	def draw(self):
		Display.blit(self.img, (self.x, self.y))

	def __repr__(self):
		return str(self.coords)


class SnakeHead(Entities):
	def __init__(self):
		super().__init__()
		self._x = 2 * snake_width
		self._coords = (self._x, self._y)

		self.part = 'Head'
		self.img = original_snake_imgs[self.part]


class SnakeBody(Entities):
	pass


class Food(Entities):
	"""
	Make sure when increasing snake's length you add the new body part
	to the beginning of the snake_bodies list so don't append it, insert it
	"""

	def __init__(self):
		super().__init__()
		self.type = 'Blueberries'  # Change this when needed
		self.img = food_imgs[self.type]

		food_x = random.randint(0, (display_width - snake_width) // snake_width) * snake_width
		food_y = random.randint(0, (display_height - snake_width) // snake_width) * snake_width
		self._coords = (food_x, food_y)
		self._x = food_x
		self._y = food_y

		self._coords = (self._x, self._y)

	def __repr__(self):
		return self.type + ' ' + str(self.coords)


class SuperFood(Food):
	def __init__(self):
		super().__init__()
		self.type = 'Bagel'
		self.img = food_imgs[self.type]
		self.original_frames = 100
		self.frames = self.original_frames  # How many frames the super food lasts for
		# Note the frames is in terms of the snake's movement frames and not the
		# actual fps of the entire game

		self.original_cooldown = 200
		self.cooldown = self.original_cooldown


def text_object(text, colour, size):
	if size == 'small':
		text_surface = small_font.render(text, True, colour)
	elif size == 'medium':
		text_surface = med_font.render(text, True, colour)
	elif size == 'large':
		text_surface = large_font.render(text, True, colour)
	else:
		raise Exception('Incorrect size in def of text_object()')
	return text_surface, text_surface.get_rect()


def message_to_screen(msg,
                      colour,
                      y_displace=0,
                      x_displace=0,
                      size='small',
                      side='center'
                      ):
	"""
	Each 'side' differs in their starting positions,
	point of the text box being controlled and
	in the direction of the displacements

	The name od the 'side' indicates the starting position as well as
	the point of the text box being controlled

	Center: Pygame directions
	Top: Pygame directions
	Bottom Left: Negative y
	Bottom Right: Negative y and Negative x
	:param msg:
	:param colour:
	:param y_displace:
	:param x_displace:
	:param size:
	:param side:
	:return:
	"""
	text_surf, text_rect = text_object(msg, colour, size)
	if side == 'center':
		text_rect.center = (display_width / 2) + x_displace, \
		                   (display_height / 2) + y_displace
	elif side == 'top':
		text_rect.midtop = (display_width / 2) + x_displace, y_displace
	elif side == 'bottom_left':
		text_rect.bottomleft = (x_displace,
		                        display_height - y_displace)
	elif side == 'bottom_right':
		text_rect.bottomright = (display_width - x_displace,
		                         display_height - y_displace)
	elif side == 'custom_center':
		text_rect.center = x_displace, y_displace
	elif side == 'custom_top':
		text_rect.midtop = x_displace, y_displace
	elif side == 'custom_top_left':
		text_rect.topleft = x_displace, y_displace
	elif side == 'custom_bottom':
		text_rect.midbottom = x_displace, y_displace
	elif side == 'custom_bot_left':
		text_rect.bottomleft = x_displace, y_displace
	elif side == 'custom_bot_right':
		text_rect.bottomright = x_displace, y_displace
	elif side == 'custom_mid_right':
		text_rect.midright = x_displace, y_displace
	elif side == 'custom_mid_left':
		text_rect.midleft = x_displace, y_displace
	else:
		text_rect.center = (display_width / 2) + x_displace, \
		                   (display_height / 2) + y_displace
	Display.blit(text_surf, text_rect)


def draw_to_screen():
	"""
	Draws everything
	:return:
	"""
	global score
	global snake_imgs

	for food in foods:
		food.draw()
	# berry.draw()

	snake_head.draw()

	for body in snake_bodies:
		body.draw()

	message_to_screen('Score: ' + str(score), ochre, y_displace=0, side='custom_top_left')


def rotation():
	global snake_imgs

	snake_bodies[0].direction = snake_bodies[1].direction

	for index in range(len(snake_bodies[1:])):
		if snake_bodies[index].direction != snake_bodies[index + 1].direction:
			snake_bodies[index].corner = True
			snake_bodies[index].img = snake_imgs[snake_bodies[index].direction + snake_bodies[index + 1].direction]
		else:
			snake_bodies[index].corner = False

	for part in snake_bodies:
		if not part.corner:

			if part.direction == 'right':
				snake_imgs[part.part] = original_snake_imgs[part.part]
			if part.direction == 'down':
				snake_imgs[part.part] = pygame.transform.rotate(original_snake_imgs[part.part], 270)
			if part.direction == 'left':
				snake_imgs[part.part] = pygame.transform.rotate(original_snake_imgs[part.part], 180)
			if part.direction == 'up':
				snake_imgs[part.part] = pygame.transform.rotate(original_snake_imgs[part.part], 90)

			part.img = snake_imgs[part.part]


def movement():
	"""
	Moves the elements in the game
	:return:
	"""

	global letter_keys
	global x_pos
	global y_pos
	global snake_head
	global snake_bodies
	global last_direction
	global last_tail_pos
	global speed_control
	global score

	if direction == 'right' and last_direction != 'left':
		x_pos += snake_width
		last_direction = 'right'
	elif direction == 'left' and last_direction != 'right':
		x_pos -= snake_width
		last_direction = 'left'
	elif direction == 'up' and last_direction != 'down':
		y_pos -= snake_width
		last_direction = 'up'
	elif direction == 'down' and last_direction != 'up':
		y_pos += snake_width
		last_direction = 'down'
	else:
		print(direction, last_direction)

	last_tail_pos = snake_bodies[0].x, snake_bodies[0].y

	for snake in range(len(snake_bodies[:-1])):
		snake_bodies[snake].x = snake_bodies[snake + 1].x
		snake_bodies[snake].y = snake_bodies[snake + 1].y
		snake_bodies[snake].direction = snake_bodies[snake + 1].direction

	snake_head.x = x_pos
	snake_head.y = y_pos
	snake_head.direction = last_direction

	if len(foods) == 1:
		# a = random.randint(0, len(snake_bodies))
		# if a > 4:
		if len(snake_bodies) >= 4:
			bagel.cooldown -= 1
		print(bagel.cooldown)
		if bagel.cooldown <= 0:
			foods.append(bagel)
			bagel.frames = bagel.original_frames - random.randint(len(snake_bodies) - 10, 2 * len(snake_bodies))
			bagel.cooldown = bagel.original_cooldown + random.randint(- len(snake_bodies), 10 - len(snake_bodies))

	for food_index in range(len(foods)):
		if foods[food_index].coords == snake_head.coords:
			food_x = random.randint(0, (display_width - snake_width) // snake_width) * snake_width
			food_y = random.randint(0, (display_height - snake_width) // snake_width) * snake_width
			foods[food_index].coords = (food_x, food_y)

			new_body = SnakeBody()
			new_body.coords = last_tail_pos
			new_body.part = 'Tail'
			new_body.direction = snake_bodies[0].direction
			snake_bodies.insert(0, new_body)
			snake_bodies[1].part = 'Body'

			if (not len(snake_bodies) % 5) and speed_control > 4:
				speed_control -= 1
				print(speed_control)

			score += 1

			if getattr(foods[food_index], 'frames', False):
				foods[food_index].frames = 0
				score += 3
				speed_control += 2
				if speed_control > 12:
					speed_control = 12

		if getattr(foods[food_index], 'frames', 'not found') != 'not found':
			foods[food_index].frames -= 1
			if foods[food_index].frames <= 0:
				food_x = random.randint(0, (display_width - snake_width) // snake_width) * snake_width
				food_y = random.randint(0, (display_height - snake_width) // snake_width) * snake_width
				foods[food_index].coords = (food_x, food_y)
				del foods[food_index]

	rotation()


def control():
	"""
	This function manages the user's input
	:return:
	"""
	global letter_keys
	global direction

	if (letter_keys['right'] or letter_keys['d']) and last_direction != 'left':
		direction = 'right'
	elif (letter_keys['left'] or letter_keys['a']) and last_direction != 'right':
		direction = 'left'
	elif (letter_keys['up'] or letter_keys['w']) and last_direction != 'down':
		direction = 'up'
	elif (letter_keys['down'] or letter_keys['s']) and last_direction != 'up':
		direction = 'down'


def game_loop():
	global mouse_1
	global mouse_1_states
	global mouse_2
	global mouse_2_states
	global game_exit
	global letter_keys
	global frame_count

	global snake_head
	global snake_bodies

	global x_pos
	global y_pos

	game_exit = False

	while not game_exit:

		mouse_pos = pygame.mouse.get_pos()

		for event in pygame.event.get():
			if event.type == pygame.QUIT:
				game_exit = True

			if event.type == pygame.KEYDOWN:
				if event.key == pygame.K_p:
					pass
				if event.key == pygame.K_w:
					letter_keys['w'] = True
				if event.key == pygame.K_a:
					letter_keys['a'] = True
				if event.key == pygame.K_s:
					letter_keys['s'] = True
				if event.key == pygame.K_d:
					letter_keys['d'] = True
				if event.key == pygame.K_UP:
					letter_keys['up'] = True
				if event.key == pygame.K_LEFT:
					letter_keys['left'] = True
				if event.key == pygame.K_DOWN:
					letter_keys['down'] = True
				if event.key == pygame.K_RIGHT:
					letter_keys['right'] = True

				if event.key == pygame.K_LSHIFT:
					letter_keys['shift'] = True

			if event.type == pygame.KEYUP:
				if event.key == pygame.K_w:
					letter_keys['w'] = False
				if event.key == pygame.K_a:
					letter_keys['a'] = False
				if event.key == pygame.K_s:
					letter_keys['s'] = False
				if event.key == pygame.K_d:
					letter_keys['d'] = False
				if event.key == pygame.K_UP:
					letter_keys['up'] = False
				if event.key == pygame.K_LEFT:
					letter_keys['left'] = False
				if event.key == pygame.K_DOWN:
					letter_keys['down'] = False
				if event.key == pygame.K_RIGHT:
					letter_keys['right'] = False

				if event.key == pygame.K_LSHIFT:
					letter_keys['shift'] = False

			if event.type == pygame.MOUSEBUTTONDOWN:
				if event.button == 1:
					mouse_1 = True
				if event.button == 3:
					mouse_2 = True
				if event.button == 4:
					pass
				if event.button == 5:
					pass

			if event.type == pygame.MOUSEBUTTONUP:
				if event.button == 1:
					mouse_1 = False
				if event.button == 3:
					mouse_2 = False

		frame_count.append(1)

		mouse_1_states.insert(0, mouse_1)
		mouse_1_states = mouse_1_states[:15]

		mouse_2_states.insert(0, mouse_2)
		mouse_2_states = mouse_2_states[:2]

		Display.fill(grey)

		control()

		if len(frame_count) > speed_control:
			frame_count = list()
			movement()

		if x_pos < 0 or x_pos >= display_width or y_pos < 0 or y_pos >= display_height:
			game_over()

		# ----------------------------------------------------------------- #

		# Makes sure that the snake body parts never touch
		attr_x = (body_part.x for body_part in snake_bodies[:-1])
		attr_y = (body_part.y for body_part in snake_bodies[:-1])

		if (snake_head.x, snake_head.y) in zip(attr_x, attr_y):
			game_over()

		# ----------------------------------------------------------------- #

		if not game_exit:
			draw_to_screen()

		pygame.display.update()
		clock.tick(120)

	pygame.quit()
	quit()


def game_over():
	global game_exit
	global score
	Display.fill(off_white)

	message_to_screen('GAME OVER', red, y_displace=-150, size='large')
	message_to_screen('Your Score Was: ' + str(score), ochre, y_displace=-50, size='medium')

	pygame.display.update()

	while not game_exit:
		for event in pygame.event.get():
			if event.type == pygame.QUIT:
				game_exit = True

	return game_exit


snake_head = SnakeHead()
first_tail = SnakeBody()
first_tail.part = 'Tail'
snake_bodies = [first_tail, snake_head]

# last_tail_pos is the last position of the tail of the snake used
# for determining the location of a new snake body part
last_tail_pos = (snake_bodies[0].x, snake_bodies[0].y)

berry = Food()
bagel = SuperFood()
foods = [berry]

direction = 'right'
last_direction = direction

x_pos = snake_head.x
y_pos = snake_head.y

# 15 is a good starting speed for 60 fps
speed_control = 10  # The smaller the value, the faster the snake moves

frame_count = list()

score = 0

game_loop()
