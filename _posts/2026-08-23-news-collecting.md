
# [단계 1] 상세 내용 다이얼로그 창 클래스 (`DetailDialog`)

* **설명**: 메인 화면에서 뉴스 목록의 **[내용]** 버튼을 눌렀을 때 새롭게 팝업창으로 뜨는 상세 보기 화면입니다. 사용자가 뉴스의 제목과 긴 본문을 한눈에 읽기 편하도록 `QTextEdit` 창을 읽기 전용으로 배치하여 보여줍니다. 창이 이미 열려 있는 상태에서 다른 뉴스의 내용을 누르면 창을 새로 띄우지 않고 내용만 갱신(`update_content`)해 주는 스마트한 역할도 함께 담당합니다.

```python
import sys
from PyQt5.QtWidgets import *
from PyQt5 import uic
import pymysql.cursors

form = uic.loadUiType("hello.ui")[0]


class DetailDialog(QWidget):

  def __init__(self, data):
    super().__init__()
    # 뉴스 제목을 팝업창의 상단 타이틀바 이름으로 설정합니다.
    self.setWindowTitle(str(data.get("jemok", "뉴스 내용")))
    self.resize(500, 400)

    layout = QVBoxLayout()

    # 본문을 보여줄 텍스트 편집 상자를 만들고 수정하지 못하도록 읽기 전용으로 설정합니다.
    self.text_neyong = QTextEdit()
    self.text_neyong.setReadOnly(True)
    self.text_neyong.setText(str(data.get("neyong", "")))

    layout.addWidget(self.text_neyong)
    self.setLayout(layout)

  def update_content(self, data):
    # 이미 켜져 있는 상세 창일 경우, 새로운 뉴스 내용으로 제목과 본문을 갈아끼웁니다.
    self.setWindowTitle(str(data.get("jemok", "뉴스 내용")))
    self.text_neyong.setText(str(data.get("neyong", "")))
```



# [단계 2] 메인 윈도우 초기화 및 화면 세팅 (`MyWindow`의 `__init__`)

* **설명**: 프로그램을 켜자마자 가장 먼저 준비 운동을 하는 생성자 단계입니다. `hello.ui` 화면 파일을 불러와 기본 틀을 잡고, 한글 카테고리 이름을 네이버 뉴스 번호 코드로 바꿔주기 위한 사전(`category_map`)을 세팅합니다. 또한 뉴스를 띄울 표(`tableWidget`)의 크기 조절 방식을 맞추고, 사용자가 콤보박스나 달력(캘린더)을 건드릴 때마다 데이터를 새로고침하도록 CCTV(시그널)를 연결합니다.

```python
class MyWindow(QMainWindow, form):

  def __init__(self):
    super().__init__()
    self.setupUi(self)
    self.resize(800, 600)

    # 한글 카테고리 이름과 네이버 뉴스 번호 코드를 1:1로 짝지어 둔 매핑 사전입니다.
    self.category_map = {
        "정치": "100",
        "경제": "101",
        "사회": "102",
        "세계": "104",
    }

    # 뉴스 목록을 띄울 테이블의 열 개수를 4개로 지정하고 마지막 열의 이름을 '내용'으로 정합니다.
    self.tableWidget.setColumnCount(4)
    self.tableWidget.setHorizontalHeaderItem(3, QTableWidgetItem("내용"))

    # 표의 칸 크기가 화면에 맞게 예쁘게 늘어나고 줄어들도록 설정합니다.
    header = self.tableWidget.horizontalHeader()
    header.setSectionResizeMode(0, QHeaderView.Stretch)
    header.setSectionResizeMode(1, QHeaderView.ResizeToContents)
    header.setSectionResizeMode(2, QHeaderView.ResizeToContents)
    header.setSectionResizeMode(3, QHeaderView.ResizeToContents)

    self.detail_window = None
    self.current_result = []

    # 네이버 원본 뉴스로 바로 갈 수 있는 링크를 보여줄 라벨을 설정합니다.
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

    # 콤보박스나 캘린더를 조작하면 자동으로 load_data 함수가 실행되도록 신호를 연결합니다.
    self.comboBox.currentIndexChanged.connect(self.load_data)
    self.calendarWidget.clicked.connect(self.load_data)
    self.load_data()
```



# [단계 3] 데이터 조회 및 화면 갱신 (`load_data`)

* **설명**: 사용자가 콤보박스에서 카테고리를 바꾸거나 캘린더에서 특정 날짜를 고를 때마다 작동하는 핵심 기능입니다. 선택한 날짜와 카테고리를 바탕으로 네이버 뉴스 원본 주소(URL)를 만들어 하단에 띄워주고, 원격 MySQL 데이터베이스에 접속해서 조건에 딱 맞는 뉴스 데이터(제목, 수정일, 감성 분석, 내용)를 싹 긁어와 표(Table)에 한 줄씩 예쁘게 채워 넣습니다. 각 행의 끝에는 상세 창을 띄울 [내용] 버튼도 함께 생성해 줍니다.

```python
  def load_data(self):
    selected_category_name = self.comboBox.currentText()
    category_code = self.category_map.get(selected_category_name)

    selected_date = self.calendarWidget.selectedDate().toString("yyyy.MM.dd")
    url_date = self.calendarWidget.selectedDate().toString("yyyyMMdd")

    # 선택된 정보가 있다면 사용자가 직접 볼 수 있는 네이버 원본 링크를 만들어 화면에 보여줍니다.
    if category_code and selected_date:
      target_url = f"[https://news.naver.com/main/list.naver?mode=LS2D&sid1=](https://news.naver.com/main/list.naver?mode=LS2D&sid1=){category_code}&mid=shm&date={url_date}&page=1"
      self.url_label.setText(
          f'직접 찾아보기: <a href="{target_url}">{target_url}</a>'
      )
      self.url_label.setVisible(True)

    try:
      # 원격 데이터베이스 서버에 안전하게 접속합니다.
      conn = pymysql.connect(
          host="***.***.***.***",
          user="****",
          password="**********",
          db="*********",
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

        # 조회해온 뉴스 개수만큼 표의 행 개수를 맞추고 데이터를 하나씩 집어넣습니다.
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

          # 각 행마다 '내용' 버튼을 만들어서 클릭 시 상세 창 함수가 연결되도록 합니다.
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


# [단계 4] 상세 창 띄우기 제어 (`open_detail_window`)

* **설명**: 표에 있는 [내용] 버튼을 눌렀을 때 실제로 어떤 동작을 할지 결정해 주는 함수입니다. 몇 번째 행의 버튼을 눌렀는지 확인한 뒤, 아까 1단계에서 만든 `DetailDialog` 창을 새로 띄우거나 이미 열려 있는 창이 있다면 그 창을 맨 앞으로 가져와서 내용만 바꿔 띄워줍니다.

```python
  def open_detail_window(self, row, column):
    if row < len(self.current_result):
      data = self.current_result[row]
      # 상세 창이 아예 안 켜져 있거나 닫혀 있다면 새로 만들어서 띄웁니다.
      if self.detail_window is None or not self.detail_window.isVisible():
        self.detail_window = DetailDialog(data)
        self.detail_window.show()
      else:
        # 이미 켜져 있다면 새로 만들지 않고 내용만 싹 바꾼 뒤 화면 앞으로 불러옵니다.
        self.detail_window.update_content(data)
        self.detail_window.activateWindow()
```
