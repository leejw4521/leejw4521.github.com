# 🎥 PyQt5와 OpenCV를 활용한 실시간 웹캠 영상 처리 프로그램

* **설명**: 웹캠으로부터 실시간 영상을 받아와 처리하고, PyQt5 창에 띄울 수 있도록 이미지 신호를 쏘아주는 핵심 백그라운드 클래스입니다. `QObject`를 상속받아 시그널(`pyqtSignal`)을 사용할 수 있게 만들어졌으며, 무한 루프를 돌며 웹캠 프레임을 읽어와 OpenCV의 BGR 색상을 RGB로 변환한 뒤 신호로 방출합니다. 또한 `canny` 메서드를 통해 Canny 필터(외곽선 추출) 효과를 켜고 끌 수 있는 플래그 제어 기능도 포함하고 있습니다.

```python
import cv2
import sys
from PyQt5 import QtCore
from PyQt5 import QtGui
from PyQt5 import QtWidgets


class ShowVideo(QtCore.QObject):
  flag = 0

  camera = cv2.VideoCapture(0)

  ret, image = camera.read()
  height, width = image.shape[:2]

  VideoSignal1 = QtCore.pyqtSignal(QtGui.QImage)
  VideoSignal2 = QtCore.pyqtSignal(QtGui.QImage)

  def __init__(self, parent=None):
    super(ShowVideo, self).__init__(parent)

  @QtCore.pyqtSlot()
  def startVideo(self):
    global image

    run_video = True
    while run_video:
      ret, image = self.camera.read()
      color_swapped_image = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)

      qt_image1 = QtGui.QImage(
          color_swapped_image.data,
          self.width,
          self.height,
          color_swapped_image.strides[0],
          QtGui.QImage.Format_RGB888,
      )
      self.VideoSignal1.emit(qt_image1)

      if self.flag:
        img_gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
        img_canny = cv2.Canny(img_gray, 50, 100)

        qt_image2 = QtGui.QImage(
            img_canny.data,
            self.width,
            self.height,
            img_canny.strides[0],
            QtGui.QImage.Format_Grayscale8,
        )

        self.VideoSignal2.emit(qt_image2)

      loop = QtCore.QEventLoop()
      QtCore.QTimer.singleShot(25, loop.quit)  # 25 ms
      loop.exec_()

  @QtCore.pyqtSlot()
  def canny(self):
    self.flag = 1 - self.flag
```


# [단계 2] 이미지 화면 렌더링 클래스 (`ImageViewer`)

* **설명**: `ShowVideo` 클래스로부터 전달받은 이미지 데이터(`QImage`)를 실제 PyQt5 화면 위에 그려주는(렌더링하는) 위젯 클래스입니다. `paintEvent`를 오버라이드하여 `QPainter`로 이미지를 창에 직접 그리며, `setImage` 슬롯 함수를 통해 새로운 이미지가 들어올 때마다 창의 크기를 이미지에 맞게 조절하고 화면을 갱신(`self.update()`)합니다.

```python
class ImageViewer(QtWidgets.QWidget):

  def __init__(self, parent=None):
    super(ImageViewer, self).__init__(parent)
    self.image = QtGui.QImage()
    self.setAttribute(QtCore.Qt.WA_OpaquePaintEvent)

  def paintEvent(self, event):
    painter = QtGui.QPainter(self)
    painter.drawImage(0, 0, self.image)
    self.image = QtGui.QImage()

  def initUI(self):
    self.setWindowTitle("Test")

  @QtCore.pyqtSlot(QtGui.QImage)
  def setImage(self, image):
    if image.isNull():
      print("Viewer Dropped frame!")

    self.image = image
    if image.size() != self.size():
      self.setFixedSize(image.size())
    self.update()
```



# [단계 3] 메인 애플리케이션 및 스레드 조립 (`if __name__ == '__main__':`)

* **설명**: 프로그램이 실행되는 진입점입니다. 무한 루프로 영상을 계속 읽어오는 무거운 작업이 메인 GUI 화면을 멈추게(버벅이게) 하지 않도록 별도의 백그라운드 **스레드(`QThread`)**를 생성하여 `ShowVideo` 객체를 그 안으로 이동시킵니다. 그 후 원본 영상과 Canny 필터 영상을 보여줄 두 개의 뷰어와 제어 버튼들을 가로·세로 레이아웃으로 예쁘게 배치한 뒤 최종 메인 윈도우에 띄우고 이벤트 루프를 시작합니다.

```python
if __name__ == "__main__":
  app = QtWidgets.QApplication(sys.argv)

  thread = QtCore.QThread()
  thread.start()
  vid = ShowVideo()
  vid.moveToThread(thread)

  image_viewer1 = ImageViewer()
  image_viewer2 = ImageViewer()

  vid.VideoSignal1.connect(image_viewer1.setImage)
  vid.VideoSignal2.connect(image_viewer2.setImage)

  push_button1 = QtWidgets.QPushButton("Start")
  push_button2 = QtWidgets.QPushButton("Canny")
  push_button1.clicked.connect(vid.startVideo)
  push_button2.clicked.connect(vid.canny)

  vertical_layout = QtWidgets.QVBoxLayout()
  horizontal_layout = QtWidgets.QHBoxLayout()
  horizontal_layout.addWidget(image_viewer1)
  horizontal_layout.addWidget(image_viewer2)
  vertical_layout.addLayout(horizontal_layout)
  vertical_layout.addWidget(push_button1)
  vertical_layout.addWidget(push_button2)

  layout_widget = QtWidgets.QWidget()
  layout_widget.setLayout(vertical_layout)

  main_window = QtWidgets.QMainWindow()
  main_window.setCentralWidget(layout_widget)
  main_window.show()
  sys.exit(app.exec_())
```
