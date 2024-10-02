import tkinter as tk
from tkinter import messagebox
import serial
import time

def read_usb_data():
    """Liest Daten vom USB-Gerät aus und gibt den ersten empfangenen Text zurück."""
    try:
        ser = serial.Serial('COM3', 9600, timeout=1)
        time.sleep(2)  # Warte auf die Verbindung
        if ser.in_waiting > 0:
            data = ser.readline()  # Lese die Daten als Bytes
            decoded_data = data.decode('utf-8').rstrip()  # Versuche, die Daten als UTF-8 zu dekodieren
            ser.close()  # Schließe die serielle Verbindung
            return decoded_data
        else:
            ser.close()  # Schließe die serielle Verbindung, wenn keine Daten empfangen wurden
            print("Keine Daten empfangen.")  # Debugging-Ausgabe
            return None
    except serial.SerialException as e:
        messagebox.showerror("Verbindungsfehler", f"Fehler beim Verbinden mit dem USB-Gerät: {e}")
        return None
    except Exception as e:
        messagebox.showerror("Fehler", f"Ein unerwarteter Fehler ist aufgetreten: {e}")
        return None

def display_text():
    """Zeigt den eingegebenen Text im Label an oder zeigt eine Fehlermeldung an."""
    entered_text = entry.get()  # Holt den Text aus dem Eingabefeld
    if entered_text:
        label.config(text=entered_text)  # Aktualisiert das Label mit dem eingegebenen Text
        
        # Berechnung der Rechteckbreite basierend auf der Länge des Textes
        rect_width = len(entered_text) * 10  # Beispiel: 10 Pixel pro Zeichen
        canvas.delete("all")  # Löscht vorherige Zeichnungen
        canvas.create_rectangle(10, 50, 10 + rect_width, 100, fill="#4CAF50")  # Zeichnet das Rechteck
    else:
        # Versuche, Daten vom USB-Gerät zu lesen, wenn kein Text eingegeben ist
        usb_data = read_usb_data()
        if usb_data:
            label.config(text=usb_data)  # Zeigt die empfangenen Daten an
            rect_width = len(usb_data) * 10
            canvas.delete("all")
            canvas.create_rectangle(10, 50, 10 + rect_width, 100, fill="#4CAF50")
        else:
            messagebox.showwarning("Eingabefehler", "Bitte gib einen Text ein oder stelle sicher, dass das USB-Gerät verbunden ist.")  # Fehlermeldung

# Hauptfenster erstellen
root = tk.Tk()
root.title("Welcome")
root.geometry("400x300")  # Fenstergröße
root.configure(bg="#f0f8ff")  # Hintergrundfarbe ändern

# Eingabefeld erstellen
entry = tk.Entry(root, font=("Arial", 14), width=30)
entry.grid(row=0, column=0, padx=10, pady=10, columnspan=2)  # Padding und Spaltenanpassung

# Button erstellen
button = tk.Button(root, text="Anzeigen", command=display_text, font=("Arial", 14), bg="#4CAF50", fg="white")
button.grid(row=1, column=0, padx=10, pady=10)

# Label erstellen
label = tk.Label(root, text="", font=("Arial", 16), bg="#f0f8ff")
label.grid(row=2, column=0, padx=10, pady=10, columnspan=2)  # Padding und Spaltenanpassung

# Stil der Labels anpassen
label.configure(borderwidth=2, relief="groove", padx=10, pady=10)  # Rahmen und Padding für das Label

# Canvas erstellen
canvas = tk.Canvas(root, width=380, height=150, bg="#e0f7fa")  # Hintergrundfarbe des Canvas
canvas.grid(row=3, column=0, columnspan=2, padx=10, pady=10)  # Padding und Spaltenanpassung

# Binding der Enter-Taste
root.bind('<Return>', lambda event: display_text())

# Hauptloop starten
root.mainloop()
