import sys
import ollama
from PySide6.QtWidgets import QApplication, QWidget, QVBoxLayout, QTextEdit, QLineEdit, QPushButton, QMainWindow
from PySide6.QtCore import Qt, QThread, Signal
from PySide6.QtGui import QTextCursor, QTextBlockFormat, QFont

class OllamaWorker(QThread):
    response_ready = Signal(str)

    def __init__(self, user_message):
        super().__init__()
        self.user_message = user_message

    def run(self):
        try:
            response = ollama.chat(model="phi4", messages=[{"role": "user", "content": self.user_message}])
            reply_text = response.get("message", {}).get("content", "Error: No response received.")
        except Exception as e:
            reply_text = f"Error: {str(e)}"

        self.response_ready.emit(reply_text)

class OOPChat(QMainWindow):
    def __init__(self):
        super().__init__()

        self.setWindowTitle("JARVIS Chat")
        self.setGeometry(100, 100, 500, 600)
        self.current_theme = 'light'
        self.create_central_widget()
        self.apply_theme()
        self.show()

    def create_central_widget(self):
        central_widget = QWidget(self)
        layout = QVBoxLayout()

        self.chat_display = QTextEdit()
        self.chat_display.setReadOnly(True)
        self.chat_display.setFont(QFont("Arial", 12))
        layout.addWidget(self.chat_display)

        self.input_field = QLineEdit()
        self.input_field.setPlaceholderText("Type your message here...")
        self.input_field.setFont(QFont("Arial", 12))
        self.input_field.returnPressed.connect(self.send_message)
        layout.addWidget(self.input_field)

        self.send_button = QPushButton("Send")
        self.send_button.clicked.connect(self.send_message)
        layout.addWidget(self.send_button)

        theme_button = QPushButton("Switch theme")
        theme_button.clicked.connect(self.toggle_theme)
        layout.addWidget(theme_button)

        central_widget.setLayout(layout)
        self.setCentralWidget(central_widget)

    def apply_theme(self):
        if self.current_theme == 'dark':
            self.setStyleSheet("background-color: #000000; color: gold;")
        else:
            self.setStyleSheet("background-color: white; color: black;")

    def toggle_theme(self):
        self.current_theme = 'dark' if self.current_theme == 'light' else 'light'
        self.apply_theme()

    def send_message(self):
        user_text = self.input_field.text().strip()
        if not user_text:
            return

        self.display_message(user_text, Qt.AlignRight)
        self.input_field.setDisabled(True)
        self.send_button.setDisabled(True)
        self.chat_display.append("JARVIS กำลังพิมพ์...")
        self.chat_display.repaint()

        self.worker = OllamaWorker(user_text)
        self.worker.response_ready.connect(self.display_response)
        self.worker.start()

    def display_response(self, response):
        self.chat_display.undo()
        self.display_message(f"JARVIS: {response}", Qt.AlignLeft)
        self.input_field.setEnabled(True)
        self.send_button.setEnabled(True)
        self.input_field.clear()
        self.input_field.setFocus()

    def display_message(self, message, alignment):
        cursor = self.chat_display.textCursor()
        cursor.movePosition(QTextCursor.End)
        block_format = QTextBlockFormat()
        block_format.setAlignment(alignment)
        cursor.insertBlock(block_format)
        cursor.insertText(message)
        self.chat_display.setTextCursor(cursor)
        self.chat_display.append("")

if __name__ == "__main__":
    app = QApplication(sys.argv)
    window = OOPChat()
    sys.exit(app.exec())
