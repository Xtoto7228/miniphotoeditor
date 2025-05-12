# miniphotoeditor
#I create mini photo editor on python

from PyQt5.QtCore import Qt
from PyQt5.QtWidgets import (QApplication, QWidget, QHBoxLayout, QVBoxLayout, QGroupBox, QRadioButton, QPushButton, QLabel, QButtonGroup, QTextEdit, QLineEdit, QListWidget, QInputDialog, QFormLayout, QInputDialog, QFileDialog)
from PyQt5.QtCore import QPropertyAnimation, QRect
from PyQt5.QtGui import QPixmap
import os
from PIL import Image, ImageFilter, ImageEnhance


#--Palette--

c_dark1 = '#303030'
c_dark2 = '#3d3d3d'
c_dark3 = '#404040'
c_dark4 = '#828282'




#----------INTERFACE----------
app = QApplication([])

window = QWidget()
window.setWindowTitle('Easy Editor')
window.setStyleSheet("background-color: #303030; color: white; font-size: 12px;")
window.resize(1200, 800)
lb_image = QLabel('Картинка')
listt = QListWidget()

#other

# drawing buttons 
btn_papka = QPushButton('Папка')
btn_papka.setStyleSheet("background-color: #404040; color: white; border-top-left-radius: 5px; border-bottom-left-radius: 5px; font-size: 15px; border-top-right-radius: 5px; border-bottom-right-radius: 5px;")
btn_left = QPushButton('Лево')
btn_left.setStyleSheet("background-color: #404040; color: white; border-top-left-radius: 5px; border-bottom-left-radius: 5px; font-size: 15px; border-top-right-radius: 5px; border-bottom-right-radius: 5px;")
btn_right = QPushButton('Право')
btn_right.setStyleSheet("background-color: #404040; color: white; border-top-left-radius: 5px; border-bottom-left-radius: 5px; font-size: 15px; border-top-right-radius: 5px; border-bottom-right-radius: 5px;")
btn_mirror = QPushButton('Зеркало')
btn_mirror.setStyleSheet("background-color: #404040; color: white; border-top-left-radius: 5px; border-bottom-left-radius: 5px; font-size: 15px; border-top-right-radius: 5px; border-bottom-right-radius: 5px;")
btn_contrast = QPushButton('Резкость')
btn_contrast.setStyleSheet("background-color: #404040; color: white; border-top-left-radius: 5px; border-bottom-left-radius: 5px; font-size: 15px; border-top-right-radius: 5px; border-bottom-right-radius: 5px;")
btn_black_white = QPushButton('Ч/Б')
btn_black_white.setStyleSheet("background-color: #404040; color: white; border-top-left-radius: 5px; border-bottom-left-radius: 5px; font-size: 15px; border-top-right-radius: 5px; border-bottom-right-radius: 5px;")
btn_original = QPushButton('Отменить')
btn_original.setStyleSheet("background-color: #404040; color: white; border-top-left-radius: 5px; border-bottom-left-radius: 5px; font-size: 15px; border-top-right-radius: 5px; border-bottom-right-radius: 5px;")

# layouts

row = QHBoxLayout()

vertical1 = QVBoxLayout()

vertical2 = QVBoxLayout()

horizontal1 = QHBoxLayout()
horizontal1.addWidget(btn_papka)

horizontal2 = QHBoxLayout()
horizontal2.addWidget(lb_image)

horizontal3 = QHBoxLayout()
horizontal3.addWidget(btn_left)
horizontal3.addWidget(btn_right)
horizontal3.addWidget(btn_mirror)
horizontal3.addWidget(btn_contrast)
horizontal3.addWidget(btn_black_white)
horizontal3.addWidget(btn_original)

vertical1.addLayout(horizontal1)
vertical1.addWidget(listt)

vertical2.addLayout(horizontal2)
vertical2.addLayout(horizontal3)

# final layouts

row.addLayout(vertical1, stretch=1)
row.addLayout(vertical2, stretch=4)
window.setLayout(row)


# functions

class ImageProcessor():
    def __init__(self):
        self.image = None
        self.dir = None
        self.filename = None
        self.save_dir = '/Modified'
        self.undo_stack = []  #list of modifications

    def loadImage(self, filename):
        self.filename = filename
        file_path = os.path.join(workdir, filename)
        self.image = Image.open(file_path)
        self.undo_stack.clear()  
        self.undo_stack.append(self.image.copy())  

    def showImage(self, file_path):
        lb_image.hide()
        pixmapimage = QPixmap(file_path)
        w, h = lb_image.width(), lb_image.height()
        pixmapimage = pixmapimage.scaled(w, h, Qt.KeepAspectRatio)
        lb_image.setPixmap(pixmapimage)
        lb_image.show()
    
    def saveImage(self):
        global workdir
        path = os.path.join(workdir, self.save_dir)
        if not (os.path.exists(path) or os.path.isdir(path)):
            os.mkdir(path)
        image_path = os.path.join(path, self.filename)
        self.image.save(image_path)
    
    def do_bw(self):
        self.undo_stack.append(self.image.copy())  
        self.image = self.image.convert('L')
        self.saveImage()
        global workdir
        image_path = os.path.join(workdir, self.save_dir, self.filename)
        self.showImage(image_path)
    
    def do_mir(self):
        self.undo_stack.append(self.image.copy())  
        self.image = self.image.transpose(Image.FLIP_LEFT_RIGHT)
        self.saveImage()
        global workdir
        image_path = os.path.join(workdir, self.save_dir, self.filename)
        self.showImage(image_path)

    def do_right(self):
        self.undo_stack.append(self.image.copy())  
        self.image = self.image.transpose(Image.ROTATE_90)
        self.saveImage()
        global workdir
        image_path = os.path.join(workdir, self.save_dir, self.filename)
        self.showImage(image_path)
    
    def do_left(self):
        self.undo_stack.append(self.image.copy()) 
        self.image = self.image.transpose(Image.ROTATE_270)
        self.saveImage()
        global workdir
        image_path = os.path.join(workdir, self.save_dir, self.filename)
        self.showImage(image_path)
    
    def do_sharp(self):
        self.undo_stack.append(self.image.copy()) 
        self.image = self.image.filter(ImageFilter.SHARPEN)
        self.saveImage()
        global workdir
        image_path = os.path.join(workdir, self.save_dir, self.filename)
        self.showImage(image_path)

    def undo(self):
        if len(self.undo_stack) > 1:  
            self.undo_stack.pop()  
            self.image = self.undo_stack[-1] 
            global workdir
            image_path = os.path.join(workdir, self.save_dir, self.filename)
            self.image.save(image_path)  
            self.showImage(image_path)  
        else:
            print("Невозможно отменить действие.")


    




workimage = ImageProcessor()


def filter(files, extensions):
    result = []
    print(files)
    for filename in files:
        for ext in extensions:
            if filename.endswith(ext):
                result.append(filename)

    return result


def chooseWorkdir():
    global workdir
    workdir = QFileDialog.getExistingDirectory()
    files = os.listdir(workdir)

def showFilenamesList():
    extensions = ['.jpg', '.jpeg', '.png', '.gif', '.bmp']
    chooseWorkdir()
    filenames = filter(os.listdir(workdir), extensions)
    print(filenames)
    listt.clear()
    for filename in filenames:
        listt.addItem(filename)


def showChoosenImage():
    if listt.currentRow() >= 0:
        filename = listt.currentItem().text()
        workimage.loadImage(filename)
        file_path = os.path.join(workdir, workimage.filename)
        workimage.showImage(file_path)








# event handlers

btn_papka.clicked.connect(showFilenamesList)
listt.currentRowChanged.connect(showChoosenImage)
btn_black_white.clicked.connect(workimage.do_bw)
btn_mirror.clicked.connect(workimage.do_mir)
btn_right.clicked.connect(workimage.do_right)
btn_left.clicked.connect(workimage.do_left)
btn_contrast.clicked.connect(workimage.do_sharp)
btn_original.clicked.connect(workimage.undo)


window.show()
app.exec()
