
# [단계 1] 프로그램 초기화 및 UI 로드 (`__init__`)

* **설명**: 프로그램을 실행하면 가장 먼저 호출되는 생성자 단계입니다. `hello.ui` 파일을 불러와 기본 창을 구성하고, 카테고리 매핑 정보와 테이블 컬럼의 크기 조절 방식을 설정합니다. 또한 콤보박스나 캘린더가 변경될 때 데이터를 불러오는 함수가 실행되도록 시그널을 연결합니다.

```python
import sys
from PyQt5.QtWidgets import *
from PyQt5 import uic
import pymysql.cursors

form = uic.loadUiType("hello.ui")[0]


class MyWindow(QMainWindow, form):

  def __init__(self):
    super().__init__()
    self.setupUi(self)
    self.resize(800, 600)

    self.category_map = {
        "정치": "100",
        "경제": "101",
        "사회": "102",
        "세계": "104",
    }

    self.tableWidget.setColumnCount(4)
    self.tableWidget.setHorizontalHeaderItem(3, QTableWidgetItem("내용"))

    header = self.tableWidget.horizontalHeader()
    header.setSectionResizeMode(0, QHeaderView.Stretch)
    header.setSectionResizeMode(1, QHeaderView.ResizeToContents)
    header.setSectionResizeMode(2, QHeaderView.ResizeToContents)
    header.setSectionResizeMode(3, QHeaderView.ResizeToContents)

    self.detail_window = None
    self.current_result = []

    self.url_label = QLabel(self)
    self.url_label.setOpenExternalLinks(True)
    self.url_label.setVisible(False)
    self.url_label.resize(800, 50)

    central_widget = self.centralWidget()
    if central_widget:
      if central_widget.layout():
        central_widget.layout().addWidget(self.url_label)
      else:
        layout = QVBoxLayout(central_widget)
        layout.addWidget(self.url_label)

    self.comboBox.currentIndexChanged.connect(self.load_data)
    self.calendarWidget.clicked.connect(self.load_data)
    self.load_data()
```

# [단계 2] 데이터 조회 및 화면 갱신 (`load_data`)

* **설명**: 사용자가 콤보박스에서 카테고리를 바꾸거나 캘린더에서 날짜를 선택할 때 실행됩니다. 선택된 조건에 맞춰 네이버 뉴스 원본 링크를 생성해 상단에 보여주고, 원격 MySQL 데이터베이스에 접속하여 조건에 맞는 뉴스 데이터를 조회한 뒤 테이블 위젯에 출력합니다.

```python
  def load_data(self):
    selected_category_name = self.comboBox.currentText()
    category_code = self.category_map.get(selected_category_name)

    selected_date = self.calendarWidget.selectedDate().toString("yyyy.MM.dd")
    url_date = self.calendarWidget.selectedDate().toString("yyyyMMdd")

    if category_code and selected_date:
      target_url = f"[https://news.naver.com/main/list.naver?mode=LS2D&sid1=](https://news.naver.com/main/list.naver?mode=LS2D&sid1=){category_code}&mid=shm&date={url_date}&page=1"
      self.url_label.setText(
          f'직접 찾아보기: <a href="{target_url}">{target_url}</a>'
      )
      self.url_label.setVisible(True)

    try:
      conn = pymysql.connect(
          host="183.100.182.169",
          user="root",
          password="swhacademy!",
          db="leejunwon",
          charset="utf8",
          cursorclass=pymysql.cursors.DictCursor,
      )

      with conn.cursor() as cursor:
        sql = """
                    SELECT jemok, sujeongil, gamsung, neyong
                    FROM nyusu
                    WHERE tag = %s AND sujeongil LIKE %s
                """
        cursor.execute(sql, (category_code, f"{selected_date}%"))
        result = cursor.fetchall()

        self.current_result = result

        self.tableWidget.setRowCount(len(result))
        for row_idx, row in enumerate(result):
          self.tableWidget.setItem(
              row_idx, 0, QTableWidgetItem(str(row["jemok"]))
          )
          self.tableWidget.setItem(
              row_idx, 1, QTableWidgetItem(str(row["sujeongil"]))
          )
          self.tableWidget.setItem(
              row_idx, 2, QTableWidgetItem(str(row["gamsung"]))
          )

          btn = QPushButton("내용")
          btn.clicked.connect(
              lambda checked, r=row_idx: self.open_detail_window(r, 3)
          )
          self.tableWidget.setCellWidget(row_idx, 3, btn)

    except Exception as e:
      print(f"DB 조회 에러: {e}")
    finally:
      if "conn" in locals() and conn.open:
        conn.close()
```


# [단계 3] 상세 내용 창 제어 및 다이얼로그 정의 (`open_detail_window` & `DetailDialog`)

* **설명**: 테이블의 [내용] 버튼을 누르면 호출됩니다. 뉴스의 전체 본문을 보여주기 위한 `DetailDialog` 창을 띄우거나, 이미 창이 열려있는 경우 기존 창의 내용만 업데이트하여 화면 앞으로 활성화합니다.

```python
class DetailDialog(QWidget):

  def __init__(self, data):
    super().__init__()
    self.setWindowTitle(str(data.get("jemok", "뉴스 내용")))
    self.resize(500, 400)

    layout = QVBoxLayout()

    self.text_neyong = QTextEdit()
    self.text_neyong.setReadOnly(True)
    self.text_neyong.setText(str(data.get("neyong", "")))

    layout.addWidget(self.text_neyong)
    self.setLayout(layout)

  def update_content(self, data):
    self.setWindowTitle(str(data.get("jemok", "뉴스 내용")))
    self.text_neyong.setText(str(data.get("neyong", "")))


  def open_detail_window(self, row, column):
    if row < len(self.current_result):
      data = self.current_result[row]
      if self.detail_window is None or not self.detail_window.isVisible():
        self.detail_window = DetailDialog(data)
        self.detail_window.show()
      else:
        self.detail_window.update_content(data)
        self.detail_window.activateWindow()
```


# [단계 4] 애플리케이션 실행 (`if __name__ == "__main__":`)

* **설명**: 파이썬 스크립트가 직접 실행될 때 `QApplication` 객체를 생성하고 메인 윈도우를 화면에 띄운 뒤 이벤트 루프를 시작합니다.

```python
if __name__ == "__main__":
  app = QApplication(sys.argv)
  window = MyWindow()
  window.show()
  sys.exit(app.exec_())
```

