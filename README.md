
import os

from kivy.app import App
from kivy.core.audio import SoundLoader
from kivy.core.window import Window
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.button import Button
from kivy.uix.label import Label
from kivy.uix.scrollview import ScrollView
from kivy.uix.widget import Widget
from kivy.utils import get_color_from_hex

try:
    import arabic_reshaper
    from bidi.algorithm import get_display

    def rtl(text):
        return get_display(arabic_reshaper.reshape(text))

except ImportError:
    def rtl(text):
        return text


# متن سوره حمد
FATIHA = [
    ("بِسْمِ اللَّهِ الرَّحْمَٰنِ الرَّحِيمِ",
     "به نام خداوند بخشنده مهربان"),
    ("الْحَمْدُ لِلَّهِ رَبِّ الْعَالَمِينَ",
     "ستایش مخصوص خداوند، پروردگار جهانیان است."),
    ("الرَّحْمَٰنِ الرَّحِيمِ",
     "بخشنده و مهربان است."),
    ("مَالِكِ يَوْمِ الدِّينِ",
     "مالک روز جزاست."),
    ("إِيَّاكَ نَعْبُدُ وَإِيَّاكَ نَسْتَعِينُ",
     "تنها تو را می‌پرستیم و تنها از تو یاری می‌جوییم."),
    ("اهْدِنَا الصِّرَاطَ الْمُسْتَقِيمَ",
     "ما را به راه راست هدایت کن."),
    ("صِرَاطَ الَّذِينَ أَنْعَمْتَ عَلَيْهِمْ غَيْرِ الْمَغْضُوبِ عَلَيْهِمْ وَلَا الضَّالِّينَ",
     "راه کسانی که به آنان نعمت دادی؛ نه راه کسانی که مورد خشم قرار گرفتند و نه گمراهان.")
]

# متن سوره اخلاص
IKHLAS = [
    ("قُلْ هُوَ اللَّهُ أَحَدٌ",
     "بگو: اوست خداوند یکتا."),
    ("اللَّهُ الصَّمَدُ",
     "خداوند بی‌نیاز است."),
    ("لَمْ يَلِدْ وَلَمْ يُولَدْ",
     "نه زاده و نه زاده شده است."),
    ("وَلَمْ يَكُن لَّهُ كُفُوًا أَحَدٌ",
     "و هیچ‌کس همتای او نیست.")
]


class QuranApp(App):

    def build(self):
        Window.clearcolor = get_color_from_hex("#F4F0E5")

        self.sound = None
        self.current_surah = None

        self.root = BoxLayout(
            orientation="vertical",
            padding=15,
            spacing=10
        )

        self.show_home()
        return self.root

    def make_label(self, text, size, color="#183D30"):
        return Label(
            text=rtl(text),
            font_size=size,
            color=get_color_from_hex(color),
            halign="center",
            valign="middle",
            size_hint_y=None,
            height=size * 2.2,
            text_size=(None, None)
        )

    def make_button(self, text, callback, height=55):
        button = Button(
            text=rtl(text),
            font_size=19,
            size_hint_y=None,
            height=height,
            background_normal="",
            background_color=get_color_from_hex("#245B45"),
            color=(1, 1, 1, 1)
        )
        button.bind(on_press=callback)
        return button

    def stop_audio(self):
        if self.sound:
            self.sound.stop()
            self.sound = None

    def show_home(self):
        self.stop_audio()
        self.root.clear_widgets()
        self.current_surah = None

        self.root.add_widget(Widget(size_hint_y=0.3))

        self.root.add_widget(
            self.make_label("قرآن کریم", 34)
        )

        self.root.add_widget(
            self.make_label(
                "متن عربی همراه با ترجمه فارسی",
                18,
                "#555555"
            )
        )

        self.root.add_widget(Widget(size_hint_y=0.2))

        self.root.add_widget(
            self.make_button(
                "سوره حمد",
                lambda x: self.show_surah("fatiha")
            )
        )

        self.root.add_widget(
            self.make_button(
                "سوره اخلاص",
                lambda x: self.show_surah("ikhlas")
            )
        )

        self.root.add_widget(Widget(size_hint_y=1))

    def show_surah(self, name):
        self.stop_audio()
        self.root.clear_widgets()
        self.current_surah = name

        if name == "fatiha":
            title = "سوره حمد"
            verses = FATIHA
            audio_file = "fatiha.mp3"
        else:
            title = "سوره اخلاص"
            verses = IKHLAS
            audio_file = "ikhlas.mp3"

        self.root.add_widget(
            self.make_label(title, 30)
        )

        scroll = ScrollView()

        content = BoxLayout(
            orientation="vertical",
            spacing=12,
            padding=10,
            size_hint_y=None
        )

        content.bind(
            minimum_height=content.setter("height")
        )

        for number, (arabic, persian) in enumerate(
            verses, start=1
        ):

            verse_box = BoxLayout(
                orientation="vertical",
                spacing=5,
                size_hint_y=None,
                height=145,
                padding=8
            )

            verse_box.add_widget(
                self.make_label(
                    f"آیه {number}",
                    14,
                    "#A07832"
                )
            )

            verse_box.add_widget(
                self.make_label(arabic, 23)
            )

            verse_box.add_widget(
                self.make_label(
                    persian,
                    17,
                    "#444444"
                )
            )

            content.add_widget(verse_box)

        scroll.add_widget(content)
        self.root.add_widget(scroll)

        self.status = self.make_label(
            "آماده پخش",
            14,
            "#555555"
        )
        self.root.add_widget(self.status)

        self.root.add_widget(
            self.make_button(
                "▶ پخش تلاوت",
                lambda x: self.play_audio(audio_file)
            )
        )

        self.root.add_widget(
            self.make_button(
                "⏸ مکث",
                lambda x: self.pause_audio()
            )
        )

        self.root.add_widget(
            self.make_button(
                "■ توقف",
                lambda x: self.stop_and_update()
            )
        )

        self.root.add_widget(
            self.make_button(
                "بازگشت به صفحه اصلی",
                lambda x: self.show_home(),
                48
            )
        )

    def play_audio(self, filename):
        if self.sound is None:
            base = os.path.dirname(
                os.path.abspath(__file__)
            )
            audio_path = os.path.join(base, filename)

            if not os.path.isfile(audio_path):
                self.status.text = rtl(
                    "فایل صوتی پیدا نشد: " + filename
                )
                return

            self.sound = SoundLoader.load(audio_path)

            if self.sound is None:
                self.status.text = rtl(
                    "پخش این فایل صوتی ممکن نیست."
                )
                return

        self.sound.play()
        self.status.text = rtl("در حال پخش تلاوت")

    def pause_audio(self):
        if self.sound:
            self.sound.stop()
            self.sound = None
            self.status.text = rtl("پخش متوقف شد؛ برای شروع دوباره بزنید")

    def stop_and_update(self):
        self.stop_audio()
        self.status.text = rtl("تلاوت متوقف شد")


if __name__ == "__main__":
    QuranApp().run()
