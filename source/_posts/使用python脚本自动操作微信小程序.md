---
title: 使用python脚本自动操作微信小程序
date: 2024-03-18 02:17:18
tags: 
    - python
    - 脚本
categories: python
---
# 使用python脚本自动操作小程序

## 引子

今天群里有人分享了一个微信小游戏———“开局托儿所”，制作方是和“羊了个羊”同一个公司，可想而知这个游戏的尿性。第一关极其简单，第二关难度飙升，玩到后面无解也是常态。我玩了一局，果不其然，只有托儿所的水平。这怎么能忍呢？python，启动！

## 游戏规则

在开始写代码之前，我先介绍一下这个游戏的规则。

游戏会先给出一个随机的数字方阵，玩家可以在屏幕上绘制矩形，被矩形覆盖到的数字相加若等于10则会被消除，消除的数字越多，分数越多。当然，你也可以自己搜索小程序玩上一局。
![开局托儿所游戏图片](https://s2.loli.net/2024/03/18/fKegp7mCJMFWYEU.png)
## 需要用到的库

1. `pyautogui`：用于在屏幕上自动执行鼠标和键盘操作。它可以帮助你编写自动化脚本，模拟用户在计算机上的交互行为，例如点击、拖动、输入文本等。

2. `pygetwindow`：用于获取和管理窗口相关信息，例如窗口标题、位置、大小等。它提供了一些方便的方法来操作桌面上的窗口，使得对窗口的管理更加简单和灵活。

3. `time`：Python 标准库中的一个模块，用于处理与时间相关的操作。

4. `copy`：Python 标准库中的一个模块，用于实现对象的浅拷贝（shallow copy）和深拷贝（deep copy）。

5. `numpy`： Python 中用于科学计算的重要库之一。它提供了高性能的多维数组对象（称为 ndarray），以及对这些数组进行操作的各种函数。

6. `os`： Python 标准库中的一个模块，用于与操作系统进行交互，提供了许多对文件和目录进行操作的功能。

7. `cv2`：OpenCV（Open Source Computer Vision Library，开源计算机视觉库）的 Python 接口库，用于图像处理、计算机视觉和图像识别等任务。

8. `pytesseract`：用于将图像中的文本转换为可识别的文本字符串。它是对 Google 的 Tesseract OCR（Optical Character Recognition，光学字符识别）引擎的封装，通过调用 Tesseract 引擎实现文本识别功能。

本脚本中pygetwindow别名gw，numpy别名np

## main.py

首先创建一个eliminater类以及其构造函数
```
def __init__(self, window_title="开局托儿所"):
    self.window = gw.getWindowsWithTitle(window_title)[0]
    self.width = self.window.width
    self.height = self.window.height
    self.s1list = []
    self.runtime = 0
    self.thread = 3
    self.thd = 80
```
action方法用于模拟鼠标绘制矩形
```
def action(self, begin_x, end_x, begin_y, end_y,duration=0.1):
    x1, y1, x2, y2 = self.digit_squares[begin_x * 10 + begin_y]
    x1, y1 = ((x1 + x2) / 2, (y1 + y2) / 2)
    x3, y3, x4, y4 = self.digit_squares[(end_x - 1) * 10 + end_y - 1]
    x2, y2 = ((x3 + x4) / 2, (y3 + y4) / 2)
    pyautogui.moveTo(self.window.left + x1, self.window.top+self.height//7 + y1)
    pyautogui.mouseDown()
    pyautogui.moveTo(self.window.left + x2, self.window.top+self.height//7 + y2, duration=duration)
    pyautogui.mouseUp()
```
restart方法用于重开一局
```
def restart(self):
    # 设置
    x = self.window.left + 40
    y = self.window.top + 70
    pyautogui.click(x, y)
    time.sleep(1)
    # 放弃
    x = self.window.left+ (self.width // 2)
    y = self.window.top + 500
    pyautogui.click(x, y)
    time.sleep(1)
    # 确定
    y = self.window.top + 520
    pyautogui.click(x, y)
    time.sleep(1)
    # 开始游戏
    y = self.window.top + 780
    pyautogui.click(x, y)
    time.sleep(2)
```
获取score
```
def score(self):
    if hasattr(self, 'cal_matrix'):
        return 160 - np.sum(self.cal_matrix.astype(bool))
    else:
        print('未初始化')
        return 0
```
record方法记录历史分数
```
def record(self, x):
    with open('历史分数.txt', 'a') as file:
        if x[1]==0:
            file.write(f'\n')
        else:
            file.write(f'\t策略{x[0]}{x[1]}: {self.score},')
```
截图并识别
```
def init_game(self):
    time.sleep(1)
    print('\t截图中……')
    self.capture_window()
    if self.frame is not None:
        print('\t识别图像中，请耐心等待……')
        matrix, self.digit_squares = recognize_matrix(self.frame, self.thread)
        try:
            self.matrix = np.array(matrix).astype(int) 
            assert self.matrix.shape == (16,10)
            return True
        except Exception as e:
            print('\t识别错误，尝试重启')
            self.trys += 1
            return False
        time.sleep(3)
    else:
        print("截图失败！")
        return False
```
执行策略
```
def run_strategy(self, strategy, action=False):
    self.cal_matrix = self.matrix.copy()
    if strategy[0] == 1:
        self.cal_two_x(action=action)
    elif strategy[0] == 2:
        self.cal_two_y(action=action)
    if strategy[1] == 1:
        self.cal_all_x(action=action)
    elif strategy[1] == 2:
        self.cal_all_y(action=action)
```
捕捉窗口
```
def capture_window(self, record=False):
    try:
        try:
            self.window.activate()
        except:
            pass
        time.sleep(1)
        screenshot = pyautogui.screenshot(region=(self.window.left, self.window.top,
                                                    self.window.width, self.window.height))
        frame = cv2.cvtColor(np.array(screenshot), cv2.COLOR_RGB2BGR)
        if record:
            cv2.imwrite(f'result_{int(time.time())}.png', frame)
        else:
            self.frame = frame[self.height//7:-self.height//25,:,:]
            cv2.imwrite('shot.png', self.frame)
    except IndexError:
        print("窗口未找到")
        return None
```
各种策略
```
def cal_all_x(self, End=False, action=False):
    if End:
        return
    else:
        End=True
        for x_len in range(1, 16):
            for y_len in range(1, 10):
                for begin_x in range(0, 16-x_len+1):
                    for begin_y in range(0, 10-y_len+1):
                        _sum = np.sum(self.cal_matrix[begin_x:begin_x+x_len,begin_y: begin_y + y_len])
                        if _sum == 10:
                            self.cal_matrix[begin_x:begin_x+x_len,begin_y: begin_y + y_len] = 0
                            if action:
                                self.action(begin_x, begin_x+x_len, begin_y, begin_y + y_len)
                            End = False
        self.cal_all_x(End=End, action=action)
        
def cal_all_y(self, End=False, action=False):
    if End:
        return
    else:
        End=True
        for y_len in range(1, 10):
            for x_len in range(1, 16):
                for begin_x in range(0, 16-x_len+1):
                    for begin_y in range(0, 10-y_len+1):
                        _sum = np.sum(self.cal_matrix[begin_x:begin_x+x_len,begin_y: begin_y + y_len])
                        if _sum == 10:
                            self.cal_matrix[begin_x:begin_x+x_len,begin_y: begin_y + y_len] = 0
                            if action:
                                self.action(begin_x, begin_x+x_len, begin_y, begin_y + y_len)
                            End = False
        self.cal_all_y(End=End, action=action)

def cal_two_x(self, End=False, action=False):
    if End:
        return
    else:
        End=True
        for begin_x in range(0, 16):
            for begin_y in range(0, 10):
                # 搜索右边
                if self.cal_matrix[begin_x, begin_y] ==0:
                    continue
                if len(self.s1list) >0 and self.cal_matrix[begin_x, begin_y] not in self.s1list:
                    continue
                for x in range(begin_x+1, 16):
                    if self.cal_matrix[x, begin_y] ==0:
                        continue
                    elif self.cal_matrix[begin_x, begin_y]+self.cal_matrix[x, begin_y] ==10:
                        self.cal_matrix[x, begin_y] = 0
                        self.cal_matrix[begin_x, begin_y] = 0
                        if action:
                            self.action(begin_x, x+1, begin_y, begin_y+1)
                        End = False
                        break
                    else:
                        break
                # 搜索左边
                for x in range(begin_x-1, -1, -1):
                    if x < 0:
                        break
                    if self.cal_matrix[x, begin_y] ==0:
                        continue
                    elif self.cal_matrix[begin_x, begin_y]+self.cal_matrix[x, begin_y] ==10:
                        self.cal_matrix[x, begin_y] = 0
                        self.cal_matrix[begin_x, begin_y] = 0
                        if action:
                            self.action(x, begin_x+1, begin_y, begin_y+1)
                        End = False
                        break
                    else:
                        break
                # 搜索下面
                for y in range(begin_y+1, 10):
                    if self.cal_matrix[begin_x, y] ==0:
                        continue
                    elif self.cal_matrix[begin_x, begin_y]+self.cal_matrix[begin_x, y] ==10:
                        self.cal_matrix[begin_x, begin_y] = 0
                        self.cal_matrix[begin_x, y] = 0
                        if action:
                            self.action(begin_x, begin_x+1, begin_y, y+1)
                        End = False
                        break
                    else:
                        break
                # 搜索上面
                for y in range(begin_y-1, -1, -1):
                    if y < 0:
                        break
                    if self.cal_matrix[begin_x, y] ==0:
                        continue
                    elif self.cal_matrix[begin_x, begin_y]+self.cal_matrix[begin_x, y] ==10:
                        self.cal_matrix[begin_x, begin_y] = 0
                        self.cal_matrix[begin_x, y] = 0
                        if action:
                            self.action(begin_x, begin_x+1, y, begin_y+1)
                        End = False
                        break
                    else:
                        break
        self.cal_two_x(End=End, action=action)
        
def cal_two_y(self, End=False, action=False):
    if End:
        return
    else:
        End=True
        for begin_y in range(0, 10):
            for begin_x in range(0, 16):
                # 搜索右边
                if self.cal_matrix[begin_x, begin_y] ==0:
                    continue
                if len(self.s1list) >0 and self.cal_matrix[begin_x, begin_y] not in self.s1list:
                    continue
                for x in range(begin_x+1, 16):
                    if self.cal_matrix[x, begin_y] ==0:
                        continue
                    elif self.cal_matrix[begin_x, begin_y]+self.cal_matrix[x, begin_y] ==10:
                        self.cal_matrix[x, begin_y] = 0
                        self.cal_matrix[begin_x, begin_y] = 0
                        if action:
                            self.action(begin_x, x+1, begin_y, begin_y+1)
                        End = False
                        break
                    else:
                        break
                # 搜索左边
                for x in range(begin_x-1, -1, -1):
                    if x < 0:
                        break
                    if self.cal_matrix[x, begin_y] ==0:
                        continue
                    elif self.cal_matrix[begin_x, begin_y]+self.cal_matrix[x, begin_y] ==10:
                        self.cal_matrix[x, begin_y] = 0
                        self.cal_matrix[begin_x, begin_y] = 0
                        if action:
                            self.action(x, begin_x+1, begin_y, begin_y+1)
                        End = False
                        break
                    else:
                        break
                # 搜索下面
                for y in range(begin_y+1, 10):
                    if self.cal_matrix[begin_x, y] ==0:
                        continue
                    elif self.cal_matrix[begin_x, begin_y]+self.cal_matrix[begin_x, y] ==10:
                        self.cal_matrix[begin_x, begin_y] = 0
                        self.cal_matrix[begin_x, y] = 0
                        if action:
                            self.action(begin_x, begin_x+1, begin_y, y+1)
                        End = False
                        break
                    else:
                        break
                # 搜索上面
                for y in range(begin_y-1, -1, -1):
                    if y < 0:
                        break
                    if self.cal_matrix[begin_x, y] ==0:
                        continue
                    elif self.cal_matrix[begin_x, begin_y]+self.cal_matrix[begin_x, y] ==10:
                        self.cal_matrix[begin_x, begin_y] = 0
                        self.cal_matrix[begin_x, y] = 0
                        if action:
                            self.action(begin_x, begin_x+1, y, begin_y+1)
                        End = False
                        break
                    else:
                        break
        self.cal_two_y(End=End, action=action)
```
运行脚本
```
def run(self, continues=True):
    self.thread = int(input('OCR线程数（推荐3）：'))
    self.thd = int(input('请输入分数阈值（低于该值将自动放弃重开）：'))
    print(f"开始运行...")
    self.trys = 0
    while continues:
        if self.trys > 5:
            print('错误次数过多，终止运行')
            break
        if not self.init_game():
            self.restart()
            continue
        self.runtime += 1
        print('\t识别完毕，执行策略……')
        maxscore = 0
        go = None
        for strategy in [[0,1], [0,2], [1,1], [1,2], [2,1], [2,2]]:
            self.run_strategy(strategy)
            self.record(strategy)
            if self.score > maxscore:
                maxscore = self.score
                go = strategy
            print(f'\t策略{strategy}分数:{self.score}')

        self.record([0,0])
        self.trys = 0
        if maxscore < self.thd:
            print(f'\t均小于目标{self.thd}，放弃本次')
            self.restart()
        else:
            print('\t执行最优策略', go)
            self.run_strategy(go, action=True)
            self.capture_window(record=True)
            time.sleep(100)
        print(f"游戏{self.runtime}结束, 开始下一次...")
        # 点击再次挑战
        x = self.window.left + (self.width // 2)
        y = self.window.top + 620
        pyautogui.click(x, y)
        time.sleep(1)
```

## recognizeNum.py
```
import pytesseract
import os
import cv2
import numpy as np
from functools import partial
from multiprocessing import Pool


os.environ['TESSDATA_PREFIX'] = r''  # 替换为您的实际路径
pytesseract.pytesseract.tesseract_cmd = r''  # 替换为您的实际路径


def get_score(img):
    res = pytesseract.image_to_string(nmg, lang='eng', config='--psm 6 --oem 3 -c tessedit_char_whitelist=0123456789')
    return int(res.strip())

def get_intersection(h_line, v_line):
    rho_h, theta_h = h_line
    rho_v, theta_v = v_line
    # 计算交点坐标
    x, y = np.linalg.solve(np.array([[np.cos(theta_h), np.sin(theta_h)],
                                     [np.cos(theta_v), np.sin(theta_v)]]).astype(float),
                           np.array([rho_h, rho_v]).astype(float))

    # 将交点坐标转为整数
    x, y = int(x), int(y)

    return x, y


def find_all_squares(image):
    gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
    blurred = cv2.GaussianBlur(gray, (9, 9), 0)
    sharpened = cv2.filter2D(blurred, -1, np.array([[0, -2, 0], [-2, 9, -2], [0, -2, 0]]))  # 强化锐化处理
    edges = cv2.Canny(sharpened, 200, 500)

    # 使用霍夫线变换检测直线
    lines = cv2.HoughLines(edges, 1, np.pi / 180, threshold=175)

    horizontal_lines = []
    vertical_lines = []

    if lines is not None:
        for line in lines:
            rho, theta = line[0]
            a = np.cos(theta)
            b = np.sin(theta)
            x0 = a * rho
            y0 = b * rho

            # 转换为图像上的坐标
            x1 = int(x0 + 1000 * (-b))
            y1 = int(y0 + 1000 * (a))
            x2 = int(x0 - 1000 * (-b))
            y2 = int(y0 - 1000 * (a))

            # 计算直线的角度
            angle = np.degrees(np.arctan2(y2 - y1, x2 - x1))

            # 根据角度进行分类，阈值可以根据实际情况调整
            if 0 <= abs(angle) <= 2 or 178 <= abs(angle) <= 175:
                horizontal_lines.append((rho, theta))
            elif 88 <= abs(angle) <= 92:
                vertical_lines.append((rho, theta))

    # 对横线按照从上到下的顺序排序
    horizontal_lines.sort(key=lambda line: line[0])
    merged_horizontal_lines = []
    merged_vertical_lines = []
    merge_threshold = 3
    previous_line = None

    for current_line in horizontal_lines:
        if previous_line is None or current_line[0] - previous_line[0] > merge_threshold:
            merged_horizontal_lines.append((current_line[0], current_line[1]))
        previous_line = current_line

    # 对竖线按照从左到右的顺序排序
    vertical_lines.sort(key=lambda line: line[0])
    previous_line = None
    for current_line in vertical_lines:
        if previous_line is not None and current_line[0] - previous_line[0] <= merge_threshold:
            # 合并相邻的水平线
            merged_vertical_lines[-1] = (current_line[0], current_line[1])
        else:
            merged_vertical_lines.append((current_line[0], current_line[1]))
        previous_line = current_line

    found_squares = []
    threshold = 3

    # 寻找正方形
    for i, h_line in enumerate(merged_horizontal_lines):
        if i >= len(merged_horizontal_lines)-1:
            break
        next_h_line = merged_horizontal_lines[i+1]
        for j, v_line in enumerate(merged_vertical_lines):
            if j >= len(merged_vertical_lines) - 1:
                break
            next_v_line = merged_vertical_lines[j+1]

            p_x1, p_y1 = get_intersection(h_line, v_line)
            p_x2, p_y2 = get_intersection(next_h_line, next_v_line)

            is_square = abs(abs(p_x2-p_x1) - abs(p_y2-p_y1)) <= threshold and abs(p_x2-p_x1) > 15
            if is_square:
                found_squares.append((p_x1, p_y1, p_x2, p_y2))

    return found_squares


def crop_region(image, square):
    (x1, y1, x2, y2) = square

    # 通过切片提取矩形区域
    cropped_region = image[y1:y2, x1:x2]

    return cropped_region


def recognize_digit(image):
    # 预处理图像，例如二值化处理
    gray_image = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
    _, thresholded = cv2.threshold(gray_image, 127, 255, cv2.THRESH_BINARY)

    # 使用 pytesseract 进行数字识别
    digit = pytesseract.image_to_string(thresholded, lang='eng', config='--psm 6 digits -c tessedit_char_whitelist=123456789')  # --psm 6 表示按行识别

    return digit.strip()


def recognize_matrix(image, thread):
    squares = find_all_squares(image)
    get_crop = partial(crop_region, image)
    crop_images = list(map(get_crop, squares))
    worker = Pool(thread)
    recognized_digits = worker.map(recognize_digit, crop_images)
    worker.close()
    worker.join()

    digits_matrix = []
    for i in range(16):
        digits_matrix.append((recognized_digits[i * 10:i * 10 + 10]))

    return digits_matrix, squares
```
## 结尾的话

我最终用这个脚本打了135分，成功上榜。快在评论区晒晒你的分数吧~