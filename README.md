# by ZZKE
# 修复没有被绘制就被撞死的bug/调整关卡和关卡的加分step/修复发生event下的attributeerror
import pygame
import sys
import pygame.sprite
import pygame.freetype as pf
import random

# pygame各模块初始化
pygame.init()
pygame.mixer.init()
# 图片加载
people = pygame.image.load('people.png')
title_picture = pygame.image.load('滑板1.png')  # 左上图标图片
barrier = pygame.image.load('矩形 47.png')  # 两边矩形
barrier_square = pygame.image.load('实心正方形.png')  # 正方形障碍
bullet_left = pygame.image.load('子弹.png')
bullet_right = pygame.image.load('子弹 (1).png')
bullet_down = pygame.image.load('子弹 (2).png')
# 声音加载
pygame.mixer.music.load('Exoplanet1.wav')
move_fx = pygame.mixer.Sound('Snare.wav')
game_over_fx = pygame.mixer.Sound('Wasted.mp3')
# 音量初始化
pygame.mixer.music.set_volume(1)
move_fx.set_volume(0.8)
game_over_fx.set_volume(0.8)
# 窗口初始化
size = width, height = 1080, 720
windows = pygame.display.set_mode(size, pygame.RESIZABLE)
pygame.display.set_caption('跃动方块')
pygame.display.set_icon(title_picture)
# 背景初始化
windows.fill('white')
# 矩形获取
people_rect = people.get_rect()
windows_rect = windows.get_rect()
# 获得两边的障碍矩形
barrier_left_rect = barrier.get_rect()
barrier_right_rect = barrier_left_rect.copy()
# 获得第一关的障碍正方形矩形对象
barrier_square_rect = barrier_square.get_rect()
barrier_square_rect_two = barrier_square.get_rect()
# 获得第四五关的子弹矩形对象
bullet_left_rect = bullet_left.get_rect()
bullet_right_rect = bullet_right.get_rect()
bullet_down_rect = bullet_down.get_rect()
bullet_down_rect_two = bullet_down.get_rect()
# 屏幕刷新速度控制
clock = pygame.time.Clock()
fps = 60
# people对象和障碍对象的移动开关
move_key = False
#
grade = 0  # 分数初始化
grade_add_step = 1


def check_game_over():  # 检查游戏是否结束
    global grade
    key = False
    if grade < 40:
        if people_rect.colliderect(barrier_left_rect) or people_rect.colliderect(barrier_right_rect) or \
                people_rect.y > windows_rect.bottom or \
                people_rect.colliderect(barrier_square_rect) or people_rect.colliderect(barrier_square_rect_two):  #
            # 游戏结束的标志
            key = True
    elif 40 <= grade < 130:
        if people_rect.colliderect(barrier_left_rect) or people_rect.colliderect(barrier_right_rect) or \
                people_rect.y > windows_rect.bottom or \
                people_rect.colliderect(barrier_square_rect) or \
                people_rect.colliderect(bullet_left_rect) or people_rect.colliderect(bullet_right_rect):  # 游戏结束的标志
            key = True
    elif 130 <= grade:
        if people_rect.colliderect(barrier_left_rect) or people_rect.colliderect(barrier_right_rect) or \
                people_rect.y > windows_rect.bottom or \
                people_rect.colliderect(barrier_square_rect) or people_rect.colliderect(barrier_square_rect_two) or \
                people_rect.colliderect(bullet_left_rect) or people_rect.colliderect(bullet_right_rect) or \
                people_rect.colliderect(bullet_down_rect) or people_rect.colliderect(bullet_down_rect_two):  # 游戏结束的标志
            key = True
    return key


def new_game():
    global grade, grade_add_step, move_key
    # 重置分数
    grade = 0
    grade_add_step = 1
    move_key = False
    # 停止播放game_over_fx
    game_over_fx.stop()
    # 播放背景音乐
    pygame.mixer.music.play()
    # 图像位置初始化
    # 将people移到底部中央
    people_rect.midbottom = windows_rect.midbottom
    people_rect.y = people_rect.y - 500
    windows.blit(people, people_rect)
    # 障碍Rect移到顶部两侧
    barrier_left_rect.bottomleft = (0, 0)
    barrier_right_rect.bottomright = (width, 0)
    windows.blit(barrier, barrier_left_rect)
    windows.blit(barrier, barrier_right_rect)
    # 正方形or子弹障碍初始化
    if grade == 0:
        barrier_square_rect.center = (windows_rect.centerx, windows_rect.centery - 720)
        windows.blit(barrier_square, barrier_square_rect)
        barrier_square_rect_two.center = (windows_rect.centerx, windows_rect.centery - 720)
        # 子弹位置初始化
        bullet_right_rect.topleft = (0, 140)
        bullet_left_rect.topright = (width, 480)
        # 向下子弹位置初始化
        bullet_down_rect.topleft = (200, 0)
        bullet_down_rect_two.topright = (880, 0)


class Manage:  # 建立一个管理类用于游戏结束的动画绘制，及分数计分，关卡等

    def __init__(self):
        # 加载字体C://windows//Fonts//msyh.ttc
        self.font = pf.Font('C://Windows//Fonts//timesbd.ttf', 200)  #
        self.font_surface, self.font_rect = self.font.render('Wasted!', (0, 0, 0), (87, 98, 104))
        # 设置Font_rect速度
        self.speed = [2, 5]

    def game_over_draw(self):  # 检查game over，以及做出game over后的行为
        global people_rect, barrier_left_rect, barrier_right_rect, windows_rect, game_over_fx
        # 绘制结束后的内容
        if check_game_over():  # 游戏结束的标志
            # 游戏结束时对声音的控制
            pygame.mixer.music.stop()  # 停止背景声音播放
            game_over_fx.play()  # 播放游戏结束声音效果
            # 绘制game over提示符
            self.font_rect.move_ip(self.speed)
            if self.font_rect.left <= 0 or self.font_rect.right >= windows_rect.width:
                self.speed[0] = -self.speed[0]
            if self.font_rect.top <= 0 or self.font_rect.bottom >= windows_rect.height:
                self.speed[1] = -self.speed[1]
            windows.blit(self.font_surface, self.font_rect)

    def count_grade_draw(self):  # 计分
        global people_rect, barrier_left_rect, grade, grade_add_step
        font = pf.Font('C://windows//Fonts//msyh.ttc', 40)
        # 设置关卡
        if grade < 5:
            grade_add_step = 1
        elif 5 <= grade < 20:
            grade_add_step = 2
        elif 20 <= grade < 40:
            grade_add_step = 3
        elif 40 <= grade < 80:
            grade_add_step = 4
        elif 80 <= grade < 130:
            grade_add_step = 5
        elif 130 <= grade:
            grade_add_step = 6
        # 判断是否加分
        if barrier_left_rect.top >= windows_rect.bottom:  # 两边矩形障碍越过windows底线
            grade += grade_add_step
        # 绘制分数
        font.render_to(windows, (450, 0), f"你的得分:{grade}")
        # 额外绘制一个游戏帮助字符串
        font.render_to(windows, (0, 0), "按Esc退出游戏", size=20)  # 提示退出游戏
        font.render_to(windows, (0, 25), '按F1重新开始游戏', size=20)  # 提示重新开始游戏
        font.render_to(windows, (0, 50), '按<— —>键左右跳跃', size=20)  # 提示操作
        font.render_to(windows, (0, 75), '按F2关闭游戏声音', size=20)
        font.render_to(windows, (0, 100), '按F3恢复游戏声音', size=20)
        font.render_to(windows, (0, 700), '警告:请勿按下与提示无关的按键,否则可能会导致游戏异常!', fgcolor=(200, 0, 0), size=16)


# 建立管理对象
manage = Manage()


class People:  # 创建一个people类,管理他的行为

    def __init__(self, event):
        self.event = event
        self.speed_left = (-10, -23)
        self.speed_right = (10, -23)
        self.speed_down = 5

    def always_move_down(self):  # 一个让people一直下降的方法
        # 绘制前线判断判断游戏是否结束，结束则people不动
        self.speed_down = 5 + people_rect.y / 60  # 获得新的下降速度
        if check_game_over():  # 检查游戏结束的标志
            self.speed_down = 0
        people_rect.y += self.speed_down  # people_rect.y / 70
        windows.fill('white')
        windows.blit(people, people_rect)

    def event_decide(self):  # 控制people移动的开关方法,顺便用于移动时的声音效果播放。也用于障碍对象的移动
        global move_key, move_fx
        if self.event.type == pygame.KEYDOWN:
            if self.event.key == pygame.K_LEFT or self.event.key == pygame.K_RIGHT:
                move_key = True
                # 播放移动声音效果
                move_fx.play()
        elif self.event.type == pygame.KEYUP:
            move_key = False

    def move_blit_people(self):  # 绘制玩家对people的控制效果（移动）
        global people_rect
        # 绘制前线判断判断游戏是否结束，结束则people不动
        if check_game_over():  # 游戏结束的标志
            self.speed_left = self.speed_right = (0, 0)
        # 绘制people
        # if move_key and self.event.key == pygame.K_LEFT:
        #     people_rect = people_rect.move(self.speed_left)
        #     windows.blit(people, people_rect)
        # elif move_key and self.event.key == pygame.K_RIGHT:
        #     people_rect = people_rect.move(self.speed_right)
        #     windows.blit(people, people_rect)

        # 绘制people_demo 防止出现属性错误导致闪退(跳过if之后的event属性错误)
        try:
            if move_key and self.event.key == pygame.K_LEFT:
                people_rect = people_rect.move(self.speed_left)
                windows.blit(people, people_rect)
        except:
            people_rect = people_rect.move(self.speed_left)
            windows.blit(people, people_rect)
        try:
            if move_key and self.event.key == pygame.K_RIGHT:
                people_rect = people_rect.move(self.speed_right)
                windows.blit(people, people_rect)
        except:
            people_rect = people_rect.move(self.speed_right)
            windows.blit(people, people_rect)

    def check_point_to_speed(self):  # 检查位置并赋予正确的速度
        global people_rect, windows_rect
        self.speed_right = (10, -23)
        self.speed_left = (-10, -23)
        if people_rect.x <= 0:
            self.speed_left = (0, -23)
        elif people_rect.x != 0:
            self.speed_left = (-10, -23)
        if people_rect.right >= windows_rect.right:
            self.speed_right = (0, -23)
        elif people_rect.right != windows_rect.right:
            self.speed_right = (10, -23)
        if people_rect.top <= 50:
            self.speed_left = (-10, 0)
            self.speed_right = (10, 0)
            if people_rect.x <= 0:
                self.speed_left = (0, 0)
            elif people_rect.x != 0:
                self.speed_left = (-10, 0)
            if people_rect.right >= windows_rect.right:
                self.speed_right = (0, 0)
            elif people_rect.right != windows_rect.right:
                self.speed_right = (10, 0)


class Barrier:  # 创建一个障碍物类
    def __init__(self):
        # 速度初值
        self.speed_down = 2
        self.barrier_speed = (0, self.speed_down)
        # 设计关卡

    def first_barrier(self):
        global barrier_square, barrier_square_rect, windows_rect
        if check_game_over():  # 游戏结束的标志
            self.barrier_speed = (0, 0)
        # 检查障碍物的坐标，当其移到底部时，在顶部重新绘制
        if barrier_square_rect.top >= windows_rect.bottom:
            barrier_square_rect.center = (windows_rect.centerx, windows_rect.centery - 720)
        # 正方形一直向下落
        barrier_square_rect.move_ip(self.barrier_speed)
        windows.blit(barrier_square, barrier_square_rect)

    def second_barrier(self):
        global barrier_square, barrier_square_rect, windows_rect
        if check_game_over():  # 游戏结束的标志
            self.barrier_speed = (0, 0)
        # 检查障碍物的坐标，当其移到底部时，在顶部重新绘制
        windows.blit(barrier_square, barrier_square_rect)
        if barrier_square_rect.top >= windows_rect.bottom:
            barrier_square_rect.center = (windows_rect.centerx + random.randrange(-100, 110, 50),
                                          windows_rect.centery - 720)
        # 正方形一直向下落

        barrier_square_rect.move_ip(self.barrier_speed)
        windows.blit(barrier_square, barrier_square_rect)

    def third_barrier(self):
        global barrier_square, barrier_square_rect, windows_rect, barrier_square_rect_two
        if check_game_over():  # 游戏结束的标志
            self.barrier_speed = (0, 0)
        # 检查障碍物的坐标，当其移到底部时，在顶部重新绘制
        if barrier_square_rect.top >= windows_rect.bottom:
            barrier_square_rect.center = (windows_rect.centerx + random.randrange(-100, 110, 50),
                                          windows_rect.centery - 720)
        # 正方形一直向下落
        barrier_square_rect.move_ip(self.barrier_speed)
        windows.blit(barrier_square, barrier_square_rect)
        # 出现第二个正方形
        if barrier_square_rect_two.top >= windows_rect.bottom:
            barrier_square_rect_two.center = (windows_rect.centerx + random.randrange(-100, 110, 50),
                                              windows_rect.centery - 360)
        # 正方形一直向下落
        barrier_square_rect_two.move_ip(self.barrier_speed)
        windows.blit(barrier_square, barrier_square_rect_two)

    def fourth_barrier(self):
        bullet_speed_x = 12
        global windows_rect, barrier_left_rect, barrier_right_rect
        if check_game_over():  # 游戏结束的标志
            bullet_speed_x = 0
            self.speed_down = 0
        # 检查左侧子弹的坐标，当其移到底部时或右侧时，在顶部重新绘制
        if bullet_right_rect.top >= windows_rect.bottom or bullet_right_rect.left >= windows_rect.right:
            bullet_right_rect.topleft = (0, 70)
        # 子弹一直向右下落
        bullet_right_rect.move_ip(bullet_speed_x, self.speed_down)
        windows.blit(bullet_right, bullet_right_rect)
        # 检查右侧子弹的坐标，当其移到底部时或左侧时，在顶部重新绘制
        if bullet_left_rect.top >= windows_rect.bottom or bullet_left_rect.right <= windows_rect.left:
            bullet_left_rect.topright = (width, 250)
        # 子弹一直向右下落
        bullet_left_rect.move_ip(-bullet_speed_x, self.speed_down)
        windows.blit(bullet_left, bullet_left_rect)

    def fifth_barrier(self):
        bullet_speed_x = 7
        if check_game_over():  # 游戏结束的标志
            bullet_speed_x = 0
            self.speed_down = 0
        # 检查左侧子弹的坐标，当其移到底部时或右侧时，在顶部重新绘制
        if bullet_down_rect.top >= windows_rect.bottom or bullet_down_rect.left >= windows_rect.right:
            bullet_down_rect.bottomleft = (100, 0)
        # 子弹一直向右下落
        bullet_down_rect.move_ip(bullet_speed_x, 2 * self.speed_down)
        windows.blit(bullet_down, bullet_down_rect)
        # 检查右侧子弹的坐标，当其移到底部时或左侧时，在顶部重新绘制
        if bullet_down_rect_two.top >= windows_rect.bottom or bullet_down_rect_two.right <= windows_rect.left:
            bullet_down_rect_two.bottomright = (980, 0)
        # 子弹一直向右下落
        bullet_down_rect_two.move_ip(-bullet_speed_x, 2 * self.speed_down)
        windows.blit(bullet_down, bullet_down_rect_two)

    def check_grade_to_speed_guanqia(self):  # 根据游戏分数设置关卡
        global grade
        # 设置关卡(下降速度,关卡障碍)
        if 0 <= grade < 2:
            self.speed_down = 2
            self.barrier_speed = (0, self.speed_down)
        elif 2 <= grade < 5:
            self.speed_down = 4
            self.barrier_speed = (0, self.speed_down)
            self.first_barrier()
        elif 5 <= grade < 20:  # 2
            self.speed_down = 6
            self.barrier_speed = (0, self.speed_down)
            self.second_barrier()
        elif 20 <= grade < 40:  # 3
            self.speed_down = 8
            self.barrier_speed = (0, self.speed_down)
            self.third_barrier()
        elif 40 <= grade < 80:  # 4
            self.speed_down = 10
            self.barrier_speed = (0, self.speed_down)
            self.second_barrier()
            self.fourth_barrier()
        elif 80 <= grade < 130:  # 5
            self.speed_down = 6
            self.barrier_speed = (0, self.speed_down)
            self.fifth_barrier()
            self.fourth_barrier()
        elif 130 <= grade:  # 8
            self.speed_down = 5
            self.barrier_speed = (0, self.speed_down)
            self.second_barrier()
            self.fifth_barrier()

    def blit_barrier(self):  # 移动加绘制两侧全局障碍,障碍一直向下移动(非关卡障碍)
        global people_rect, barrier_left_rect, barrier_right_rect, windows_rect
        # 判断游戏是否结束，结束则barrier不动
        if check_game_over():  # 游戏结束的标志
            self.barrier_speed = (0, 0)  # Rect对象速度归零
        barrier_left_rect.move_ip(self.barrier_speed)
        barrier_right_rect.move_ip(self.barrier_speed)
        windows.blit(barrier, barrier_left_rect)
        windows.blit(barrier, barrier_right_rect)

    def check_point_to_moveup(self):  # 检查两边全局障碍物的坐标，当其移到底部时，在顶部重新绘制
        global barrier_left_rect, barrier_right_rect, windows_rect
        if barrier_left_rect.top >= windows_rect.bottom:
            barrier_left_rect.bottomleft = (0, 0)
            barrier_right_rect.bottomright = (width, 0)


# 建立障碍物对象
barrier_ = Barrier()


def main():  # 管理游戏资源
    global grade, people_object
    # 播放背景音乐
    pygame.mixer.music.set_volume(1)
    pygame.mixer.music.play()
    # 图像位置初始化
    # 将people移到底部中央
    people_rect.midbottom = windows_rect.midbottom
    people_rect.y = people_rect.y - 500
    windows.blit(people, people_rect)
    # 障碍Rect移到顶部两侧
    barrier_left_rect.bottomleft = (0, 0)
    barrier_right_rect.bottomright = (width, 0)
    windows.blit(barrier, barrier_left_rect)
    windows.blit(barrier, barrier_right_rect)
    # 正方形or子弹障碍初始化
    if grade == 0:
        barrier_square_rect.center = (windows_rect.centerx, windows_rect.centery - 720)
        windows.blit(barrier_square, barrier_square_rect)
        barrier_square_rect_two.center = (windows_rect.centerx, windows_rect.centery - 720)
        # 子弹位置初始化 横坐标异常(为负数，在屏幕外)是为了修复游戏开始时子弹没有被绘制人就被撞死的bug
        bullet_right_rect.topleft = (100, 140)
        bullet_left_rect.topright = (width, 480)
        # 向下子弹位置初始化 横坐标异常(为负数，在屏幕外)是为了修复游戏开始时子弹没有被绘制人就被撞死的bug
        bullet_down_rect.topleft = (200, 0)
        bullet_down_rect_two.topright = (880, 0)
    # 开始循环
    while True:
        pygame.event.set_allowed([pygame.KEYDOWN, pygame.KEYUP])  #
        for event in pygame.event.get():
            # 因为要接收event,所以在此建立people对象
            people_object = People(event)
            # 程序退出条件
            if event.type == pygame.QUIT:
                sys.exit()
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_ESCAPE:
                    sys.exit()
                elif event.key == pygame.K_F1:
                    new_game()
                elif event.key == pygame.K_F2:
                    pygame.mixer.music.set_volume(0)
                    move_fx.set_volume(0)
                    game_over_fx.set_volume(0)
                elif event.key == pygame.K_F3:
                    pygame.mixer.music.set_volume(1)
                    move_fx.set_volume(0.8)
                    game_over_fx.set_volume(0.8)
            # 根据event判断是否移动，(move_key)
            people_object.event_decide()
        # people需要一直下降
        people_object.always_move_down()
        # 绘制people
        for i in range(2):  # 这样移动距离会更长，更顺滑
            people_object.check_point_to_speed()  # check坐标,每次绘制之前检查坐标
            people_object.move_blit_people()  # 绘制people
        # 根据分数设置barrier的速度以及设置关卡
        barrier_.check_grade_to_speed_guanqia()
        # 检查障碍物的坐标，当其移到底部时，在顶部重新绘制
        barrier_.check_point_to_moveup()
        barrier_.blit_barrier()  # 绘制barrier
        # 显示分数
        manage.count_grade_draw()  # 继承了manage类
        # 绘制游戏结束的动画
        manage.game_over_draw()  # 继承了manage类
        # 更新屏幕
        pygame.display.update()
        # 控制帧率
        clock.tick(fps)


main()
