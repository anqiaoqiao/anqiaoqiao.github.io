---
layout: post
title: 利用Python + PyQt + Qt Designer：
subtitle: 制作带有GUI界面的程序
date: 2021-03-13
author: Ryuu
header-img: img/post-an-plan20191127.jpg
catalog: ture
tags:
    - 编程
---

### 利用Python + PyQt + Qt Designer制作带有GUI界面的程序



#### 导言

初次接触编程，是大学时代折腾单片机时学习的C语言。而后研究生阶段又因为对机器学习和网页设计感兴趣接触了Python和HTML等语言。虽然如此，还没尝试过制作带GUI界面的程序。这段大部分待在家中，遂尝试利用Python+PyQt +Qt Designer 制作带有GUI界面的程序。

<hr>
本次编程环境：win10 + Python 3.9.1 + PyQt5 5.15.2 + pyqt5-tools 5.15.2.3.0.2 + pyinstaller 4.2 + Qt Designer 5.11.1

本次程序源代码：https://github.com/anqiaoqiao/AZR_Pushing_Toolbox

##### 第一步: 构思程序功能，利用Python实现算法

制作程序的第一步便是构思程序的功能，然后试一下算法上是否能够实现，下面就以本次所制作的程序作为例子讲解： 制作某手游的计算工具箱，想要实现的功能如下。

- 计算从当前等级提升至目标等级所需的经验值
- 计算针对当前舰队数据来说的，各副本的难易度系数，通过比较该数值来抉择出击副本
- 计算比较各副本的经验值收益，分为总体收益和单次收益
- 计算通过刷某副本从当前等级提升至目标等级所需要消耗的石油数量

在构思完毕之后，利用Python验证算法是否可行。为实现功能，这边自定义了相应的数学计算模块（selflib/azrmath.py），注意在Pycharm中导入自定义模块时代码格式如下

```python
import selflib.azrmath # selflib为文件夹名，azrmath为自定义模块名
```

在验证好Python的代码可以实现后，进入下一步。

##### 第二步：设计GUI窗口

在第一步中，构思过程序要实现的功能，并且在算法层面验证过后，那么这时候需要导入什么数据，然后输出什么数据，在心里就已经很明白了。所以接下来以此利用Qt Designer设计GUI窗口。

本次程序设计的GUI窗口如下所示：

![](http://picgo.oss-ap-northeast-1.aliyuncs.com/img/GUI_looklike.jpg?x-oss-process=style/mystyle)

第一步中构想的四个功能全部放到了GUI当中。 在这一步当中，GUI中的每个元素最好自己进行命名以方便后续的编程。 各个元素都可以在Qt Designer中通过拖动来添加和编辑。当中值得注意的图片的插入方法为：拖入Label元素后编辑resource创建qrc资源，而后通过属性中的styleSheet选择qrc资源添加图片，具体步骤如下：

![](http://picgo.oss-ap-northeast-1.aliyuncs.com/img/GUI_src.jpg?x-oss-process=style/mystyle)

在完成GUI界面的设计之后，点击：File/Save as 来保存设计好的GUI窗口，保存的文件为"文件名.ui"，这时这个文件还不能直接使用，所以需要通过PyQt来将.ui文件转换为.py文件。在.ui文件所在目录运行cmd，输入下列命令

```
pyuic5 -o azurlane_toolbox.py azurlaneToolbox.ui
# azurlane_toolbox.py 为转换后的.py文件名
# azurlaneToolbox.ui 为转换前的.ui文件名
```

如果在刚才的GUI设计当中添加了图片的，由于加载了qrc资源素材，需要将.qrc文件也转换为.ui文件。 同样的方法在cmd中输入下列命令

```
pyrcc5 -o test_rc.py test.qrc
# test_rc.py 为转换后的.py文件名
# test.qrc 为转换前的.qrc文件名
```

将转换后的.py文件作为自定义模块使用，使用时要稍微修改一下代码来导入，主要修改模块名字以确保其可以导入。这样，GUI的外观设计就基本完成了。

！注意：

1.设计的窗口如果不希望改变大小也必须在属性栏中设置窗口的最大最小大小范围。

2.设计的窗口可以按"Ctrl + R" 进行预览

##### 第三步：为GUI添加功能

在这一步，为GUI添加功能。首先作为主程序，需要先写好如下的固定代码。

```python
import sys # 系统自带的一个模块
from PyQt5.QtWidgets import QApplication, QMainWindow # PyQt5 模块
from ui.ui import Ui_Form # 第二步设计好的模块， Ui_Form是class的名字
import selflib.azrmath as azrmath # 第一步写好的算法模块 <非必须>

class MyMainForm(QMainWindow, Ui_Form):
    def __init__(self, parent=None): # 这一块用于定义事件触发
        super(MyMainForm, self).__init__(parent)
        self.setupUi(self) # 到此为止加载好了设计的GUI：Ui_Form
        self.leftXP_calc_buttom.clicked.connect(self.xpleft)
        # 上面这段代码定义了事件触发：如果那个叫做'leftXP_calc_buttom'的按键被点击了，
        # 那么则执行以后后续叫做'xpleft'的函数
    # 所以接下来，就定义这个函数
    def xpleft(self):
        # 读入那个叫做'currentLV_input'的输入框里面的字符
        current_lv = self.currentLV_input.text()
        # 在那个叫做'leftxp_output'里输出字符
        self.leftxp_output.setText('Error')

# 下面的代码用于运行程序。
if __name__ == "__main__":
    app = QApplication(sys.argv)
    myWin = MyMainForm()
    myWin.show()
    sys.exit(app.exec_())
```

根据上述代码的格式，修改并且添加自己需要的功能后，进行调试排除BUG。确定各方面都没有问题之后保存代码。进入最后的打包环节。

##### 第四步：打包程序

打包程序，顾名思义就是把写好的代码打包，生成一个可以独立使用的EXE程序的过程。EXE程序的话，看不到源代码所以可以在一定程度上确保源代码被别人看到。

这里的打包步骤和之前文件转换步骤类似，首先先新建一个.py文件，本次的叫做'pyinstaller.py'，其内容代码如下

```python
import  os
if __name__ == '__main__':
    from PyInstaller.__main__ import run
    opts=['main.py','-w','--icon=icon.ico'] 
    # main.py 是源代码文件名
    # icon.ico 是exe程序图标的文件名 这里推荐图标大小128px * 128px
    run(opts)
```

准备好程序的图标文件后，找到源代码文件的根目录运行cmd，在cmd下运行pyinstaller.py后即可在根目录打包程序。打包好的程序被放在"dist/.."内。

至此，基于Python + PyQt + Qt Designer的GUI程序制作就完成了。至于程序的修改，只需要修改源代码后再重新打包即可。