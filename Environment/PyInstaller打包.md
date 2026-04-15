最好是在虚拟环境里操作，不然python版本太多容易出错。

```python
# 创建虚拟环境
python -m venv venv

# 激活虚拟环境
venv\Scripts\activate

# 安装 PyInstaller
pip install pyinstaller

# 运行打包命令
pyinstaller --onefile --windowed --clean --icon=app.ico your_script.py
```

注意：

1. 打包前先在虚拟环境里用python运行下，确保所有依赖已经安装

2. 图标必须是ico格式。可以 用如下工具做转换：

```python
import sys
from PyQt5.QtWidgets import (QApplication, QMainWindow, QVBoxLayout, QHBoxLayout, 
                            QLabel, QLineEdit, QPushButton, QComboBox, QWidget, 
                            QFileDialog, QMessageBox)
from PyQt5.QtGui import QPixmap, QDragEnterEvent, QDropEvent
from PyQt5.QtCore import Qt, QMimeData
from PIL import Image
import os
from datetime import datetime

class ImageToICOConverter(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("图片转ICO工具")
        self.setGeometry(100, 100, 600, 500)
        self.setAcceptDrops(True)
        self.initUI()
        
    def initUI(self):
        # 主布局
        main_widget = QWidget()
        main_layout = QVBoxLayout()
        
        # 图片预览区域
        self.preview_label = QLabel("拖放图片到这里或点击选择图片", self)
        self.preview_label.setAlignment(Qt.AlignCenter)
        self.preview_label.setStyleSheet("QLabel { border: 2px dashed #aaa; }")
        self.preview_label.setMinimumSize(300, 300)
        
        # 图片信息显示
        self.info_label = QLabel("图片信息将显示在这里", self)
        self.info_label.setWordWrap(True)
        
        # 图片路径输入
        path_layout = QHBoxLayout()
        self.image_path_input = QLineEdit(self)
        self.image_path_input.setPlaceholderText("图片路径")
        browse_btn = QPushButton("浏览...", self)
        browse_btn.clicked.connect(self.browse_image)
        path_layout.addWidget(self.image_path_input)
        path_layout.addWidget(browse_btn)
        
        # ICO尺寸选择
        size_layout = QHBoxLayout()
        size_label = QLabel("ICO尺寸:", self)
        self.size_combo = QComboBox(self)
        self.size_combo.addItems(["16x16", "32x32", "48x48", "64x64", "128x128", "256x256"])
        self.size_combo.setCurrentIndex(2)  # 默认48x48
        size_layout.addWidget(size_label)
        size_layout.addWidget(self.size_combo)
        
        # 保存按钮
        save_layout = QHBoxLayout()
        self.save_path_input = QLineEdit(self)
        self.save_path_input.setPlaceholderText("保存路径")
        save_browse_btn = QPushButton("浏览...", self)
        save_browse_btn.clicked.connect(self.browse_save_path)
        convert_btn = QPushButton("转换为ICO", self)
        convert_btn.clicked.connect(self.convert_to_ico)
        save_layout.addWidget(self.save_path_input)
        save_layout.addWidget(save_browse_btn)
        save_layout.addWidget(convert_btn)
        
        # 添加到主布局
        main_layout.addWidget(self.preview_label)
        main_layout.addWidget(self.info_label)
        main_layout.addLayout(path_layout)
        main_layout.addLayout(size_layout)
        main_layout.addLayout(save_layout)
        
        main_widget.setLayout(main_layout)
        self.setCentralWidget(main_widget)
        
        # 初始化变量
        self.current_image_path = ""
        self.current_image = None
    
    def browse_image(self):
        file_path, _ = QFileDialog.getOpenFileName(
            self, "选择图片", "", 
            "图片文件 (*.png *.jpg *.jpeg *.bmp *.gif)"
        )
        if file_path:
            self.load_image(file_path)
    
    def browse_save_path(self):
        save_path, _ = QFileDialog.getSaveFileName(
            self, "保存ICO文件", "", 
            "ICO文件 (*.ico)"
        )
        if save_path:
            self.save_path_input.setText(save_path)
    
    def load_image(self, file_path):
        try:
            self.current_image_path = file_path
            self.image_path_input.setText(file_path)
            
            # 显示图片
            pixmap = QPixmap(file_path)
            if not pixmap.isNull():
                scaled_pixmap = pixmap.scaled(
                    self.preview_label.width(), 
                    self.preview_label.height(),
                    Qt.KeepAspectRatio
                )
                self.preview_label.setPixmap(scaled_pixmap)
                self.preview_label.setText("")
            
            # 显示图片信息
            with Image.open(file_path) as img:
                info = f"""
                <b>图片信息:</b><br>
                文件名: {os.path.basename(file_path)}<br>
                格式: {img.format}<br>
                尺寸: {img.size[0]}x{img.size[1]}<br>
                模式: {img.mode}<br>
                大小: {os.path.getsize(file_path)/1024:.2f} KB
                """
                self.info_label.setText(info)
                self.current_image = img.copy()
                
        except Exception as e:
            QMessageBox.warning(self, "错误", f"无法加载图片: {str(e)}")
    
    def convert_to_ico(self):
        if not self.current_image_path:
            QMessageBox.warning(self, "警告", "请先选择图片")
            return
            
        save_path = self.save_path_input.text()
        if not save_path:
            QMessageBox.warning(self, "警告", "请指定保存路径")
            return
            
        try:
            size_str = self.size_combo.currentText()
            size = tuple(map(int, size_str.split('x')))
            
            # 确保保存路径有.ico扩展名
            if not save_path.lower().endswith('.ico'):
                save_path += '.ico'
            
            # 转换并保存为ICO
            icon_sizes = [size]
            self.current_image.save(save_path, sizes=icon_sizes)
            
            # 记录转换日志
            log_entry = f"{datetime.now().strftime('%Y-%m-%d %H:%M:%S')} | {os.path.basename(self.current_image_path)} -> {os.path.basename(save_path)} | {size_str}\n"
            with open("conversion_log.txt", "a") as f:
                f.write(log_entry)
            
            QMessageBox.information(self, "成功", "图片已成功转换为ICO格式!")
            
        except Exception as e:
            QMessageBox.warning(self, "错误", f"转换失败: {str(e)}")
    
    # 拖放功能实现
    def dragEnterEvent(self, event: QDragEnterEvent):
        if event.mimeData().hasUrls():
            event.acceptProposedAction()
    
    def dropEvent(self, event: QDropEvent):
        urls = event.mimeData().urls()
        if urls and urls[0].isLocalFile():
            file_path = urls[0].toLocalFile()
            if file_path.lower().endswith(('.png', '.jpg', '.jpeg', '.bmp', '.gif')):
                self.load_image(file_path)
            else:
                QMessageBox.warning(self, "错误", "仅支持图片文件(png/jpg/bmp/gif)")

if __name__ == "__main__":
    app = QApplication(sys.argv)
    converter = ImageToICOConverter()
    converter.show()
    sys.exit(app.exec_())
```

3.如果依赖本地环境变量配置的比如exiftool，要麻烦很多，直接打包会报错：
Traceback (most recent call last):

  File "motionPhoto.py", line 36, in <module>

ModuleNotFoundError: No module named 'exiftool'

##### 终极打包方案：使用 .spec 文件

如果上面的命令还是不行，说明 PyInstaller 的缓存乱了：

1. 删除工程目录下的 `build`、`dist` 文件夹和 `motionPhoto.spec`。

2. 在终端输入 `pip install pyexiftool`（确保库最新）。

3. 运行命令：
   
   ```
   pyi-makespec --onefile --windowed --add-data "exiftool_bin/exiftool.exe;exiftool_bin" --icon=app.ico motionPhoto.py
   ```

4. 这会生成一个 `motionPhoto.spec` 文件。用记事本打开它，修改成
   
   ```python
   # -*- mode: python ; coding: utf-8 -*-
   import ffpyplayer
   ff_path = ffpyplayer.__path__[0]
   
   a = Analysis(
       ['motionPhoto.py'],
       pathex=[],
       binaries=[],
       datas=[
           ('tools/exiftool.exe', 'tools'),
           (ff_path, 'ffpyplayer'),  # 强制将整个 ffpyplayer 文件夹存入 exe 根目录
           ('app.ico', '.'),         # 窗口图标，供 resource_path('app.ico') 使用
       ],
       hiddenimports=['exiftool','ffpyplayer', 'ffpyplayer.player', 'ffpyplayer.threading',
                      'PyQt5.QtWidgets', 'PyQt5.QtGui', 'PyQt5.QtCore', 'PyQt5.QtMultimedia', 'PyQt5.QtMultimediaWidgets'],
       hookspath=[],
       hooksconfig={},
       runtime_hooks=[],
       excludes=[],
       noarchive=False,
       optimize=0,
   )
   pyz = PYZ(a.pure)
   
   exe = EXE(
       pyz,
       a.scripts,
       a.binaries,
       a.datas,
       [],
       name='motionPhoto',
       debug=False,
       bootloader_ignore_signals=False,
       strip=False,
       upx=True,
       upx_exclude=[],
       runtime_tmpdir=None,
       console=False,
       disable_windowed_traceback=False,
       argv_emulation=False,
       target_arch=None,
       codesign_identity=None,
       entitlements_file=None,
       icon=['app.ico'],
   )
   
   ```
   
   

5. 最后运行：python -m PyInstaller motionPhoto.spec


