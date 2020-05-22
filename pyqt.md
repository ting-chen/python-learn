一、pycharm 使用技巧
Ctrl+N : 通过class名称快速查找class定位
Ctrl+G : 快速跳转到指定行列
Shift+Ctrl+Backspace : 跳转到最后一次编辑的行
Ctrl+Shift+T : 快捷生成测试代码
Ctrl+B : 对于调用的查找定义处. 对于定义的查找调用处
Ctrl+Alt+B : 快速跳转到声明处
Ctrl+F12 : 列出当前文件的全部变量以及函数声明
Ctrl+Shift+Alt+2 : 列出当前文件的路径结构
Ctrl+Alt+H : 对函数快速编辑
Shift+F6 : 快速重命名（文件名，类名，成员变量，函数，局部变量）
Ctrl+Q : 查找注释
Ctrl+Tab : 切换鼠标焦点到不同的页面中
Alt+Shift+Ctrl+7 : 全局查找函数被调用被引用的地方
Alt+F7 : 查找被引用的地方

二、Code

1.通过QFileDialog.getOpenFile快速选择文件路径，已经指定正确parent对象，但是还是show不出来？
file_name = QFileDialog.getOpenFileName(parent, "Open File", '', "Config (*.yml *.xpm *.jpg)")  # 调用后无法显示
#解决方案
  指定 options=QFileDialog.DontUseNativeDialog ；有的系统调用原始的路径选择款无法显示，所以改用非系统原始框。并且使用非系统原生框还能将弹框使用qss效果渲染

2.怎么通过Delegate 设置item的custom Widget一直显示？

class ServerAndIstanceDelegate(QtWidgets.QStyledItemDelegate):
    # 用来传递widget处理事件的信号
    sigCreateICloudTaskTrigger = QtCore.pyqtSignal()
    sigDeleteInstanceTrigger = QtCore.pyqtSignal()
    
    # 创建custom Widget
    def createEditor(self, parent, option, index):
        try:
            data = index.data(QtCore.Qt.UserRole)
            widget = None
            if isinstance(data, DeviceManager):
            #custom widget
                widget = ServerItemWidget(self.parent())
                widget.setLabelText(data.getDeviceName())
                widget.setConnectState(data.getConnectState)
                if isinstance(data, LocalDeviceManager):
                       widget.sigCreateTaskTrigger.connect(
                       self.sigCreateICloudTaskTrigger)    
                       widget.sigCloseTigger.connect(
                       self.sigDeleteInstanceTrigger）
        except Exception as e:
            print('create server item dele')
        return widget

    # 改变item 的显示高度
    def sizeHint(self, QStyleOptionViewItem, QModelIndex):
        QtWidgets.QStyledItemDelegate().sizeHint(
        QStyleOptionViewItem, QModelIndex)
        widget = ServerItemWidget()
        return QtCore.QSize(60, widget.height())

    # 开启处于编辑状态
    def paint(self, painter, option, index):
        if isinstance(self.parent(), QtWidgets.QAbstractItemView):
            self.parent().openPersistentEditor(index)
    
    # 设置widget的显示位置
    def updateEditorGeometry(self, QWidget,   
    QStyleOptionViewItem,QModelIndex): 
        rect = QtCore.QRect(QStyleOptionViewItem.rect.x(),   
                            QStyleOptionViewItem.rect.y(),
                            QStyleOptionViewItem.rect.width(),     
                            QWidget.height())
        QWidget.setGeometry(rect)

  
3.改变控件property 后样式效果不会根据qss中指定的效果改变，怎么办？
qss 样式效果指定：
QPushButton {
  background-color: #505F69;
  border: 1px solid #32414B;
  color: #F0F0F0;
  border-radius: 4px;
  outline: none;
  min-width: 80px;
}

QPushButton[ConnectState="Connected"] {
  background-color: #26A000; /*148CD2 006D4F*/
  border: 1px solid #32414B;
  color: #F0F0F0;
  border-radius: 4px;
  outline: none;
}

改变控件属性代码：
def setConnectState(self, connectState):
    """
    :param connectState: bool  change state
    :return:
    """
    self._connectState = connectState
    if self._connectState:
        self.setText(self._connectText)
        self.setProperty('ConnectState', 
                        'Connected')
    else:
        self.setText(self._disconnectText)
        self.setProperty('ConnectState', 
                        'DisConnected')
        
    self.style().unpolish(self)
    self.style().polish(self)


解决方案：
虽然元系统中控件属性已经更新，但是没有显式调用update 或者repaint的话效果不会被渲染到界面上
    self.update() # repaint()

4.怎么利用装饰器在widget show出来之前做一些动作，例如让widget显示在屏幕中央？
需要让弹框居中显示，并且在显示之前不同弹框类有各自的处理动作，基于此场景：
# 定义装饰器
def initWidgetBeforeShow(function):  # Decorator
    def init(self):
        self.initBeforeShow()
        center_point =   \                  
        QtWidgets.QDesktopWidget().availableGeometry().center()
        self.move(center_point)
        function(self)
    return init

声明widget基类
class BaseCenterWidget(QtWidgets.QDialog):
    def __init__(self, parent=None):
        super(BaseCenterWidget, self).__init__(parent)
        self._is_close = True

    def initBeforeShow(self):
        pass

    @initWidgetBeforeShow
    def show(self):
        QtWidgets.QDialog.show(self) # or super().show()

    @initWidgetBeforeShow
    def exec(self):
        QtWidgets.QDialog.exec(self) # or super().exec()

    @initWidgetBeforeShow
    def exec_(self):
        QtWidgets.QDialog.exec_(self) # or super().exec_()
 所有继承自BaseCenterWidget的类都能居中显示，并且可以重载initBeforeShow，来实现各自的显示前的初始化动作

5.怎么在readonly的QLineEdit 开启使用clearButton？
对于QLineEdit设置为readonly状态后，点击clearButton是没有效果的，但是可以通过代码解决这个问题
首先，开始clear功能开启
self.configLineEdit.setClearButtonEnabled(True)
其次，把button的enable状态单独打开
self.configLineEdit.findChild(QtWidgets.QToolButton).setEnabled(True)
效果
图片: https://uploader.shimo.im/f/tobtGpwFXm0amuDF.png

6.怎么实现遮罩效果
首先来个装饰器
def maskWidget(function):
    # 设置遮罩窗体
    def mask(self):
        self.setProperty('WidgetType', 'mask')  # TODO:hardcode
        self.setWindowFlags(Qt.FramelessWindowHint) #设置为无标题栏
        if self.parent:
            parent_rect = self.parent().geometry()
            # 设置为父窗体对象大小
            self.setGeometry(0, 0, parent_rect.width(), 
                             parent_rect.height())
        # self.setWindowOpacity(100)
        # self.setAttribute(Qt.WA_TranslucentBackground)
        function(self)
    return mask
坑点：mask初始show出来的时候大小和需要遮罩窗体大小一致，但是需要遮罩的窗体大小改变的时候mask还是初始化的大小，咋办呢？
解决方案：
# 重载widget的painteEvent事件跟随被遮罩窗体geometry变化
def paintEvent(self, QPaintEvent):
    if self.parent():
        parent_rect = self.parent().geometry()
        self.setGeometry(0, 0, parent_rect.width(),   
                         parent_rect.height())
    super().paintEvent(QPaintEvent)
带定时消失的遮罩example
from Widget.NormalWidget import maskWidget, initWidgetBeforeShow
from Widget.UI_ATPMaskForm import Ui_maskForm
from PyQt5.QtWidgets import QDialog
from PyQt5.QtCore import QTimer, pyqtSignal, Qt
DEFAULT_TIMMER_INT = 20000  # 10s


class MaskWidget(QDialog, Ui_maskForm):
    sigTimeOut = pyqtSignal()

    def __init__(self, parent):
        super(MaskWidget, self).__init__(parent)
        self.setupUi(self)
        self.iconPushButton.setProperty('CusIcon', 'load')
        self.setWindowFlag(Qt.WindowStaysOnTopHint)
        self._is_time_out = False
        self._simple_timer = QTimer(self)
        self._simple_timer.setInterval(DEFAULT_TIMMER_INT)
        self.initConnect()

    def initConnect(self):
        self._simple_timer.timeout.connect(self.slotTimeout)
    
    # triger before show or exec
    def initBeforeShow(self):
        self._is_time_out = False
        self._simple_timer.start()

    def actionAfterShow(self):
        self.setWindowFlag(Qt.WindowStaysOnTopHint)

    @maskWidget
    @initWidgetBeforeShow
    def show(self):
        super().show()

    @maskWidget
    @initWidgetBeforeShow
    def exec(self):
        super().exec()

    @maskWidget
    @initWidgetBeforeShow
    def exec_(self):
        super().exec_()

    def paintEvent(self, QPaintEvent):
        if self.parent():
            parent_rect = self.parent().geometry()
            self.setGeometry(0, 0, parent_rect.width(), parent_rect.height())
        super().paintEvent(QPaintEvent)

    def closeEvent(self, QCloseEvent):
        self._simple_timer.stop()
        super().closeEvent(QCloseEvent)

    def slotTimeout(self):
        self._is_time_out = True
        self.sigTimeOut.emit()
        self.close()


7.怎么指定Wizard.show() 出现的是第page2？
首先设置每个page的id page1:0， page2:1
wizard.setStartId(1)
其次，这里面有坑：
例如，将wizard的option设置如下：
图片: https://uploader.shimo.im/f/YDhC2pBCAWfinki4.png
noBackButtonOnStartPage
本来wizard从page1跳转到page2的时候框架逻辑会自动显示上一步，但是目前将page2放在首页显示，结合勾选的option，导致上一步show不出来。如果需要显示上一步可以建议用一个CustomButton 结合wizard.back() slot来完成上一步功能


8.怎么指定Wizard提供的button显示顺序？
设置按钮的文字，并指定顺序
self.setButtonText(self.BackButton, "上一步")
self.setButtonText(self.CustomButton2, "上一步")  # TODO： 用框架无法实现show为第二个页面的时候上一步能够显示
self.setButtonText(self.NextButton, "下一步")
self.setButtonText(self.CancelButton, "取消")
self.setButtonText(self.FinishButton, "完成")
self.setButtonText(self.CustomButton1, "创建")
# 指定显示顺序
custom_btn_layout = [QWizard.CancelButton, QWizard.NextButton, 
                     QWizard.BackButton,
                     QWizard.CustomButton2,
                     QWizard.CustomButton1, QWizard.FinishButton]
self.setButtonLayout(custom_btn_layout)

9.有一个弹框A， A中有两个subWidget b,c  当b，c指定父对象为A的后，将A.show()会发生什么？
弹框A中不会显示b和c 
解决方案：
需要指定b，c的parent为A。parent 或者不指定

10.怎么指定Delegate custom 的显示位置以及大小
可以通过Delegate给view设置custom的widget
效果如下
图片: https://uploader.shimo.im/f/msmpS69bzZAv8OIj.jpg
但是,只重载createEditor（），那么custom widget的起始位置在view 的header处，item的高度使用原始item字体大小
解决方案：
重载sizeHint事件，设置为customWidget的高度
def sizeHint(self, QStyleOptionViewItem, QModelIndex):

    QtWidgets.QStyledItemDelegate().sizeHint(QStyleOptionViewItem,   
                                             QModelIndex)
    widget = ServerItemWidget()
    return QtCore.QSize(60, widget.height())
重载updateEditorGeometry,添加上hear的高度
def updateEditorGeometry(self, QWidget, QStyleOptionViewItem,QModelIndex): #TODO 这里有一个viewHeader的高度30
    rect = QtCore.QRect(QStyleOptionViewItem.rect.x(), QStyleOptionViewItem.rect.y()+35,
                        QStyleOptionViewItem.rect.width(), QWidget.height())
    QWidget.setGeometry(rect)

11.直接调用yaml.load()会发生什么
会被认为不安全的调用，需要指定
yaml.load(secrect_context, Loader=yaml.FullLoader)

12.加密写入文件和读取加密文件
import base64
import os
from Core.constant import CURRENT_MAIN_FILE_PATH
import yaml


class SecretFileWritter:
    @staticmethod
    def writeToFile(file_context, file_name):
        file_context_byte = file_context
        if isinstance(file_context, dict):
            file_context_byte = str(file_context)
        encode_str =base64.encodebytes(file_context_byte.encode('utf-8'))
        try:
            with open(file_name, "wb+") as fd:
                fd.write(encode_str)
                fd.close()
                return True
        except Exception as e:
            print('write encode context fail', e.args)
            return False
        return False

    @staticmethod
    def readFromFile(file_name):
        try:
            with open(file_name, "rb") as fd:
                file_context = fd.read()
                print('read context', file_context)
                decode_str = base64.decodebytes(file_context)
                print('read context', decode_str)
                return True, decode_str
        except Exception as e:
            print('read secret context fail', e.args)
            return False, ''
        return False, ''


from Core.constant import LOCAL_SERVER_ACCESS_CONFIG_FILE
from Core.constant import ICLOUD_ACCESS_CONFIG_FILE

if __name__ == '__main__':
    # write test
    # yaml_str = dict()
    # yaml_str['user_name'] = 'xt'
    # yaml_str['password'] = 'asd'
    # contect = yaml.dump(yaml_str)
    # SecretFileWritter.writeToFile(file_context=contect, file_name=LOCAL_SERVER_ACCESS_CONFIG_FILE)


    # read test
    retult, secrect_context = SecretFileWritter.readFromFile(file_name=LOCAL_SERVER_ACCESS_CONFIG_FILE)
    print(secrect_context)
    print(secrect_context.decode('utf-8'))
    dect = yaml.load(secrect_context, Loader=yaml.FullLoader)
    print(dect)
    print(yaml.dump(dect))


13.读取‘wb+’加密后的文件是乱码，怎么办？
通过下列方式读取的文件内容为乱码
with open(file_name, "r") as fd:

file_context = fd.read()
解决方案：
因为写入的时候是以字节方式写入加密的，所以也要先以‘rb’的模式读取

14.相对通用的使用异步功能
下列方法可以简单的将耗时操作移出主线程
import time
from PyQt5 import QtCore
from PyQt5 import QtWidgets
from numpy import unicode
import threading


def async(method, args, uid, readycb, errorcb=None, sigshutdown=None):
    """
    Asynchronously runs a task

    :param func method: the method to run in a thread
    :param object uid: a unique identifier for this task (used for verification)
    :param slot updatecb: the callback when data is receieved cb(uid, data)
    :param slot errorcb: the callback when there is an error cb(uid, errmsg)

    The uid option is useful when the calling code makes multiple async calls
    and the callbacks need some context about what was sent to the async method.
    For example, if you use this method to thread a long running database call
    and the user decides they want to cancel it and start a different one, the
    first one may complete before you have a chance to cancel the task.  In that
    case, the "readycb" will be called with the cancelled task's data.  The uid
    can be used to differentiate those two calls (ie. using the sql query).

    :returns: Request instance
    """
    request = Request(method, args, uid, readycb, errorcb, sigshutdown)
    QtCore.QThreadPool.globalInstance().start(request)
    return request


class Request(QtCore.QRunnable):
    """
    A Qt object that represents an asynchronous task

    :param func method: the method to call
    :param list args: list of arguments to pass to method
    :param object uid: a unique identifier (used for verification)
    :param slot readycb: the callback used when data is receieved
    :param slot errorcb: the callback used when there is an error
    :param signal sigshutdown: stop action
    The uid param is sent to your error and update callbacks as the
    first argument. It's there to verify the data you're returning

    After created it should be used by invoking:

    .. code-block:: python

       task = Request(...)
       QtCore.QThreadPool.globalInstance().start(task)

    """
    INSTANCES = []
    FINISHED = []

    def __init__(self, method, args, uid, readycb, errorcb=None, sigshutdown=None):
        super(Request, self).__init__()
        self.setAutoDelete(True)
        self.cancelled = False

        self.method = method
        self.args = args
        self.uid = uid
        self.dataReady = readycb
        self.dataError = errorcb
        self.stopaction = sigshutdown
        # connect shutdowm signal
        if self.stopaction:
            print('connect successful')
            self.stopaction.connect(self.slotShutDown, QtCore.Qt.QueuedConnection)

        Request.INSTANCES.append(self)

        # release all of the finished tasks
        Request.FINISHED = []

    def slotShutDown(self):
        print('slot short down', threading.currentThread())
        self.shutdown()

    def run(self):
        """
        Method automatically called by Qt when the runnable is ready to run.
        This will run in a separate thread.
        """
        # this allows us to "cancel" queued tasks if needed, should be done
        # on shutdown to prevent the app from hanging
        if self.cancelled:
            self.cleanup()
            return

        # runs in a separate thread, for proper async signal/slot behavior
        # the object that emits the signals must be created in this thread.
        # Its not possible to run grabber.moveToThread(QThread.currentThread())
        # so to get this QObject to properly exhibit asynchronous
        # signal and slot behavior it needs to live in the thread that
        # we're running in, creating the object from within this thread
        # is an easy way to do that.
        grabber = Requester()
        grabber.Loaded.connect(self.dataReady, QtCore.Qt.QueuedConnection)
        if self.dataError is not None:
            grabber.Error.connect(self.dataError, QtCore.Qt.QueuedConnection)

        try:
            result = self.method(*self.args)
            if self.cancelled:
                # cleanup happens in 'finally' statement
                return
            grabber.Loaded.emit(self.uid, result)
        except Exception as error:
            if self.cancelled:
                # cleanup happens in 'finally' statement
                return
            grabber.Error.emit(self.uid, unicode(error))
        finally:
            # this will run even if one of the above return statements
            # is executed inside of the try/except statement see:
            # https://docs.python.org/2.7/tutorial/errors.html#defining-clean-up-actions
            self.cleanup(grabber)

    def cleanup(self, grabber=None):
        # remove references to any object or method for proper ref counting
        self.method = None
        self.args = None
        self.uid = None
        self.dataReady = None
        self.dataError = None

        if grabber is not None:
            grabber.deleteLater()

        # make sure this python obj gets cleaned up
        self.remove()

    def remove(self):
        try:
            Request.INSTANCES.remove(self)

            # when the next request is created, it will clean this one up
            # this will help us avoid this object being cleaned up
            # when it's still being used
            Request.FINISHED.append(self)
        except ValueError:
            # there might be a race condition on shutdown, when shutdown()
            # is called while the thread is still running and the instance
            # has already been removed from the list
            return

    @staticmethod
    def shutdown():
        print('enter shutdown')
        for inst in Request.INSTANCES:
            inst.cancelled = True
        Request.INSTANCES = []
        Request.FINISHED = []


class Requester(QtCore.QObject):
    """
    A simple object designed to be used in a separate thread to allow
    for asynchronous data fetching
    """

    #
    # Signals
    #

    Error = QtCore.pyqtSignal(object, unicode)
    """
    Emitted if the fetch fails for any reason

    :param unicode uid: an id to identify this request
    :param unicode error: the error message
    """

    Loaded = QtCore.pyqtSignal(object, object)
    """
    Emitted whenever data comes back successfully

    :param unicode uid: an id to identify this request
    :param list data: the json list returned from the GET
    """

    NetworkConnectionError = QtCore.pyqtSignal(unicode)
    """
    Emitted when the task fails due to a network connection error

    :param unicode message: network connection error message
    """

    def __init__(self, parent=None):
        super(Requester, self).__init__(parent)


class ExampleObject(QtCore.QObject):
    sigStop = QtCore.pyqtSignal()

    def __init__(self, parent=None):
        super(ExampleObject, self).__init__(parent)
        self.uid = 0
        self.request = None

    def ready_callback(self, uid, result):
        if uid != self.uid:
            return
        print("Data ready from %s: %s" % (uid, result))

    def error_callback(self, uid, error):
        if uid != self.uid:
            return
        print("Data error from %s: %s" % (uid, error))

    def fetch(self):
        if self.request is not None:
            # cancel any pending requests
            self.request.cancelled = True
            self.request = None

        self.uid += 1
        self.request = async(slow_method, ["arg1", "arg2"], self.uid,
                             self.ready_callback,
                             self.error_callback,
                             self.sigStop)

    def stop(self):
        print('emit stop signal', threading.currentThread())
        self.sigStop.emit()


def slow_method(arg1, arg2):
    print("Starting slow method")
    # for i in range(0, 1000000000000000000):
    #     pass
    time.sleep(10)
    return arg1 + arg2


if __name__ == "__main__":
    import sys
    app = QtWidgets.QApplication(sys.argv)

    obj = ExampleObject()

    dialog = QtWidgets.QDialog()
    layout = QtWidgets.QVBoxLayout(dialog)
    button = QtWidgets.QPushButton("Generate", dialog)
    button2 = QtWidgets.QPushButton("stop", dialog)
    progress = QtWidgets.QProgressBar(dialog)
    progress.setRange(0, 0)
    layout.addWidget(button)
    layout.addWidget(progress)
    layout.addWidget(button2)
    button.clicked.connect(obj.fetch)
    button2.clicked.connect(obj.stop)
    dialog.show()

    app.exec_()
    app.deleteLater() # avoids some QThread messages in the shell on exit
    # cancel all running tasks avoid QThread/QTimer error messages
    # on exit
    Request.shutdown()

15.setIndexWidget 在pyqt中的坑
setIndexWidget 不用关系delegate中的显示等各种问题，还能直接获取到widget的对象，但是这个时候的model 一旦reset后，widget会消失。字面意义上的，连内存都被释放掉

16.关于成员变量property 使用的一种场景必要性
震惊：Python的protect类型对象可以直接通过类实例访问！！！
这就导致会出现数据绑定的现象，具体看下列example
class FakeMutableString(object):
    def __init__(self, s=""):
        self.s = [s]

    def __str__(self):
        return self.s[0]

    def get(self):
        return self.s[0]

    def set(self, new):
        self.s[0] = new


class Example(object):
    def __init__(self):
        self._name = FakeMutableString()
        self._can = "看得到吗！！！"

    @property
    def name(self):
        return self._name.get()

    @name.setter
    def name(self, new):
        self._name.set(new)


if __name__ == '__main__':
    # Create Example instances
    a = Example()
    b = Example()
    c = Example()

    # Set their name attributes
    #   ( this actually calls <Example>._name.set() )
    a.name = "ThingA"
    b.name = "ThingB"
    c.name = "ThingC"

    # Access and print their name attributes
    #   ( this actually calls <Example>._name.get() )
    print(a.name, b.name, c.name)  # ThingA ThingB ThingC

    # We can't bind like you suggested, but we can change what b._name points to
    #   So here, we change b._name to point to the FakeMutableString a._name points to
    b._name = a._name
    

    # Now when we print the names, we see something different
    print(a.name, b.name, c.name)  # ThingA ThingA ThingC

    # And we can make a change in a, and have it reflected in b
    a.name = "CommonName"
    print(a.name, b.name, c.name)  # CommonName CommonName ThingC

    # And vice-versa
    b.name = "StillCommon"
    print(a.name, b.name, c.name)  # StillCommon StillCommon ThingC

    print(a._can)

品，细细的品。所以还是建议养成通过property方式访问变量

三、辅助
1.通过ui文件生成并整理py文件：
因为遵循qt ui文件名称全小写，但是py文件名使用驼峰命名规则，所以ui文件名称和py文件名称不完全挂钩，因此有了下面的脚本：
#! /bin/bash
# author: chen_xt

ui_src_file_path='/home/xt/Documents/git/GUI/pygui_generate/pyqt_gui_generate/'
ui_dst_file_path='/home/xt/Documents/git/AITrainning/Widget/UI'
py_dst_file_path='/home/xt/Documents/git/AITrainning/Widget/'


# 声明ui和py文件之间对应关系
declare -A arr

arr["aitrainningwidget.ui"]='UI_ATPMainWidget.py'

arr+=( ["atplocalserverconn.ui"]='UI_ATPLogLocalServerWidget.py' \

   ["autoinstanceconfigwidget.ui"]='UI_ATPInstanceDefaultConfigWidget.py' \

   ["basetaskdetailwidget.ui"]='UI_ATPBaseTaskDetailWidget.py' \

   ["createtaskwidget.ui"]='UI_ATPGenerateTask.py' \
......
)

# 执行生成py文件以及移动到指定位置
for key in ${!arr[@]}; do
   ui_file_name=${key}
   py_file_name=${arr[${key}]} 
   total_ui_path=$ui_src_file_path$ui_file_name
   total_py_path=$py_dst_file_path$py_file_name
   
   pyuic5 -x $total_ui_path -o $total_py_path
   echo $total_ui_path $ui_dst_file_path $total_py_path
   cp -rf $total_ui_path $ui_dst_file_path 
    
done

2.pyqt example

3.配色网址

4.qss guild

5.pyqt api doc

6.uml example

7.icon图标网址（张都赞助提供）

8.python exec / pyqtdeployed

9.打包为Python库（简单试用成功，但是多个python文件打包还是没成功后续在试试）

10.markBook




  
