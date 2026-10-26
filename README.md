import datetime
import threading
import tkinter as tk
from PIL import Image, ImageTk
import pyttsx3
import speech_recognition as sr

# إعداد محرك تحويل النص إلى كلام
engine = pyttsx3.init()
engine.setProperty('rate', 150)  # سرعة الكلام

def speak(text):
    """دالة لجعل التطبيق ينطق النص بصوت مسموع"""
    engine.say(text)
    engine.runAndWait()

class App:
    def __init__(self, root):
        self.root = root
        self.name = ""
        self.named = False
        
        # إعداد نافذة التطبيق
        root.title("المساعد الذكي - تيمو")
        root.geometry("400x650")
        root.config(bg="#111111")
        
        # عنوان التطبيق
        tk.Label(
            root, text="المساعد الصوتي: تيمو", 
            font=("Arial", 16, "bold"), fg="white", bg="#111111"
        ).pack(pady=10)
        
        # محاولة عرض الصورة الشخصية لتيمو
        try:
            self.img = ImageTk.PhotoImage(
                Image.open("1000018313.png").resize((200, 200), Image.Resampling.LANCZOS)
            )
            tk.Label(root, image=self.img, bg="#111111").pack(pady=10)
        except Exception:
            # في حال عدم وجود الصورة، يتخطى الخطوة دون توقف التطبيق
            pass
            
        # منطقة عرض النصوص والحوارات
        self.lbl = tk.Label(
            root, text="اضغط وتحدث...", 
            font=("Arial", 12), fg="#00ffff", bg="#111111", wraplength=350, justify="center"
        );
        self.lbl.pack(pady=20)
        
        # زر الاستماع والتحدث
        tk.Button(
            root, text="🎙️ تحدث الآن", font=("Arial", 14, "bold"), 
            fg="white", bg="#0084ff", activebackground="#005bb5",
            command=lambda: threading.Thread(target=self.listen).start()
        ).pack(pady=10)
        
        # رسالة الترحيب الأولية عند فتح التطبيق
        greet = "أهلاً بك! أنا تيمو، مساعدك الشخصي. ما هو اسمك؟"
        self.lbl.config(text=greet)
        threading.Thread(target=speak, args=(greet,)).start()

    def listen(self):
        """دالة الاستماع لصوت المستخدم وتحويله إلى نص ومعالجته"""
        rec = sr.Recognizer()
        with sr.Microphone() as src:
            self.lbl.config(text="جاري الاستماع... تفضل بالكلام")
            try:
                # الاستماع للصوت مع تحديد وقت أقصى للانتظار
                audio = rec.listen(src, timeout=5, phrase_time_limit=5)
                msg = rec.recognize_google(audio, language="ar-DZ")
                self.lbl.config(text=f"أنت: {msg}")
                
                # التحقق مما إذا كان التطبيق يعرف اسم المستخدم أم لا
                if not self.named:
                    self.name = msg.strip()
                    self.named = True
                    resp = f"عاش من شافك يا {self.name}! كيف يمكنني مساعدتك اليوم؟"
                else:
                    m = msg.lower()
                    # إضافة ميزات ذكية جديدة للردود
                    if "الساعة" in m or "الوقت" in m:
                        current_time = datetime.datetime.now().strftime('%H:%M')
                        resp = f"الساعة الآن هي {current_time}."
                    elif "التاريخ" in m or "اليوم" in m:
                        current_date = datetime.datetime.now().strftime('%Y-%m-%d')
                        resp = f"تاريخ اليوم هو {current_date}."
                    elif "كيف حالك" in m:
                        resp = f"أنا بصحة جيدة يا {self.name}، شكراً لسؤالك!"
                    else:
                        resp = f"لقد سمعتك يا {self.name}, وتقول: {msg}."
                
                # تحديث النص ونطق الرد
                self.lbl.config(text=f"تيمو: {resp}")
                speak(resp)
                
            except sr.WaitTimeoutError:
                self.lbl.config(text="انتهى وقت الانتظار، لم أسمع شيئاً.")
            except sr.UnknownValueError:
                self.lbl.config(text="عفواً، لم أتمكن من فهم ما قلت. عاود المحاولة.")
            except Exception as e:
                self.lbl.config(text="حدث خطأ في الاتصال أو الميكروفون.")

if __name__ == "__main__":
    root = tk.Tk()
    app = App(root)
    root.mainloop()
