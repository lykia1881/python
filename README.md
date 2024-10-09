import sys
import webbrowser
import requests
from PyQt5.QtWidgets import QApplication, QWidget, QVBoxLayout, QLabel, QLineEdit, QPushButton, QComboBox, QMessageBox, QCheckBox

class UsernameFinder(QWidget):
    def __init__(self):
        super().__init__()
        self.history = []
        self.initUI()

    def initUI(self):
        self.setWindowTitle('CG Username Finder')
        self.setGeometry(100, 100, 350, 400)

        layout = QVBoxLayout()

        self.username_input = QLineEdit(self)
        layout.addWidget(QLabel('Kullanıcı Adı:'))
        layout.addWidget(self.username_input)

        layout.addWidget(QLabel('Platform:'))
        self.platform_urls = {
            'Instagram': 'https://www.instagram.com/',
            'YouTube': 'https://www.youtube.com/@',
            'Twitter': 'https://twitter.com/',
            'Facebook': 'https://www.facebook.com/',
            'Chess.com': 'https://www.chess.com/member/',
            'TikTok': 'https://www.tiktok.com/@',
            'LinkedIn': 'https://www.linkedin.com/in/',
            'Reddit': 'https://www.reddit.com/user/',
            'GitHub': 'https://github.com/',
            'Lichess': 'https://lichess.org/@/',
            'Discord': 'https://discord.com/users/',
            'Snapchat': 'https://www.snapchat.com/add/',
            'Telegram': 'https://t.me/',
            'WeChat': 'https://weixin.qq.com/',
            'VKontakte (VK)': 'https://vk.com/',
            'MySpace': 'https://myspace.com/',
            'Viber': 'viber://chat?number=',
            'Line': 'http://line.me/ti/p/',
            'Quora': 'https://www.quora.com/profile/',
            'Clubhouse': 'https://www.joinclubhouse.com/@',
            'Foursquare': 'https://foursquare.com/user/',
            'Badoo': 'https://badoo.com/en/',
            'MeWe': 'https://mewe.com/i/',
            'Mastodon': 'https://mastodon.social/@',
            'Gab': 'https://gab.com/',
            'Parler': 'https://parler.com/profile/',
            'Nextdoor': 'https://nextdoor.com/',
            'BitChute': 'https://www.bitchute.com/channel/',
            'DLive': 'https://dlive.tv/',
            'Rumble': 'https://rumble.com/user/',
            'Pinterest': 'https://www.pinterest.com/',
            'Tumblr': 'https://www.tumblr.com/',
        }
        self.platform_combo = QComboBox(self)
        self.platform_combo.addItems(self.platform_urls.keys())
        layout.addWidget(self.platform_combo)

        self.url_preview = QLabel('')
        layout.addWidget(self.url_preview)

        self.copy_url_checkbox = QCheckBox('Sadece URL\'yi Kopyala')
        layout.addWidget(self.copy_url_checkbox)

        self.search_button = QPushButton('Hesaba Git', self)
        self.search_button.clicked.connect(self.perform_search)
        layout.addWidget(self.search_button)

        layout.addWidget(QLabel('Geçmiş:'))
        self.history_combo = QComboBox(self)
        layout.addWidget(self.history_combo)

        self.history_button = QPushButton('Geçmişten Git', self)
        self.history_button.clicked.connect(self.open_history)
        layout.addWidget(self.history_button)

        self.clear_history_button = QPushButton('Geçmişi Temizle', self)
        self.clear_history_button.clicked.connect(self.clear_history)
        layout.addWidget(self.clear_history_button)

        self.setLayout(layout)

        self.username_input.textChanged.connect(self.update_preview)

    def update_preview(self):
        username = self.username_input.text().strip()
        platform = self.platform_combo.currentText()
        if username:
            full_url = self.platform_urls[platform] + username
            self.url_preview.setText(f'URL: {full_url}')
        else:
            self.url_preview.setText('')

    def perform_search(self):
        username = self.username_input.text().strip()
        platform = self.platform_combo.currentText()

        if username:
            full_url = self.platform_urls[platform] + username
            if self.check_url_status(full_url):
                if self.copy_url_checkbox.isChecked():
                    self.copy_to_clipboard(full_url)
                    self.show_message('Başarılı', 'URL panoya kopyalandı!')
                else:
                    webbrowser.open(full_url)
                self.update_history(username)
            else:
                self.show_message('Hata', f'{full_url} bulunamadı!')
        else:
            self.show_message('Hata', 'Lütfen geçerli bir kullanıcı adı girin!')

    def open_history(self):
        selected_username = self.history_combo.currentText()
        if selected_username:
            platform = self.platform_combo.currentText()
            full_url = self.platform_urls[platform] + selected_username
            if self.check_url_status(full_url):
                if self.copy_url_checkbox.isChecked():
                    self.copy_to_clipboard(full_url)
                    self.show_message('Başarılı', 'URL panoya kopyalandı!')
                else:
                    webbrowser.open(full_url)
            else:
                self.show_message('Hata', f'{full_url} bulunamadı!')
        else:
            self.show_message('Hata', 'Geçmişten bir kullanıcı adı seçin!')

    def check_url_status(self, url):
        try:
            response = requests.get(url, timeout=5)
            return response.status_code == 200
        except requests.RequestException:
            return False

    def update_history(self, username):
        if username and username not in self.history:
            self.history.append(username)
            self.history_combo.addItem(username)

    def clear_history(self):
        self.history.clear()
        self.history_combo.clear()
        self.show_message('Başarılı', 'Geçmiş temizlendi!')

    def copy_to_clipboard(self, text):
        clipboard = QApplication.clipboard()
        clipboard.setText(text)

    def show_message(self, title, message):
        msg_box = QMessageBox()
        msg_box.setWindowTitle(title)
        msg_box.setText(message)
        msg_box.exec_()

def main():
    app = QApplication(sys.argv)
    finder = UsernameFinder()
    finder.show()
    sys.exit(app.exec_())

if __name__ == '__main__':
    main()
