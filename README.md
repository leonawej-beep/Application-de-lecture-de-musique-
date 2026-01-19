import customtkinter as ctk
import pygame
from tkinter import filedialog, messagebox
import os

pygame.mixer.init()

root = ctk.CTk()
root.title("Lecteur de Musique - Under Leo")
root.geometry("400x550")

# Variables globales
current_song = None
song_list = []
current_song_index = -1
paused = False

# ================= FONCTIONS =================

def load_song():
    global current_song, current_song_index, song_list
    songs = filedialog.askopenfilenames(
        filetypes=[
            ("Audio Files", "*.mp3 *.wav"),
            ("MP3 Files", "*.mp3"),
            ("WAV Files", "*.wav"),
            ("All Files", "*.*")
        ],
        title="Sélectionner une ou plusieurs musiques"
    )
    
    if songs:
        song_list = list(songs)
        current_song_index = 0
        load_song_by_index(current_song_index)
        status_label.configure(text=f"Chargé: {len(song_list)} musique(s)")

def load_song_by_index(index):
    global current_song, current_song_index, paused
    if 0 <= index < len(song_list):
        try:
            song_path = song_list[index]
            pygame.mixer.music.load(song_path)
            current_song = song_path
            current_song_index = index
            song_label.configure(text=f"{os.path.basename(song_path)}\n({index + 1}/{len(song_list)})")
            playlist_label.configure(text=f"Playlist: {len(song_list)} musique(s)")
            paused = False
        except Exception as e:
            messagebox.showerror("Erreur", f"Impossible de charger la musique:\n{str(e)}")

def play_song():
    global paused
    if current_song is None:
        messagebox.showwarning("Erreur", "Aucune musique chargée !")
        return
    
    if paused:
        pygame.mixer.music.unpause()
        paused = False
    else:
        pygame.mixer.music.play()
    
    status_label.configure(text="Lecture...")
    update_song_info()

def pause_song():
    global paused
    pygame.mixer.music.pause()
    paused = True
    status_label.configure(text="En pause")

def stop_song():
    global paused
    pygame.mixer.music.stop()
    paused = False
    status_label.configure(text="Arrêté")

def next_song():
    global current_song_index
    if not song_list:
        return
    
    if current_song_index < len(song_list) - 1:
        current_song_index += 1
    else:
        current_song_index = 0  # Retour au début
    
    load_song_by_index(current_song_index)
    if not paused:
        play_song()

def previous_song():
    global current_song_index
    if not song_list:
        return
    
    if current_song_index > 0:
        current_song_index -= 1
    else:
        current_song_index = len(song_list) - 1  # Aller à la fin
    
    load_song_by_index(current_song_index)
    if not paused:
        play_song()

def set_volume(value):
    pygame.mixer.music.set_volume(float(value))
    volume_label.configure(text=f"Volume: {int(float(value) * 100)}%")

def update_song_info():
    if current_song and pygame.mixer.music.get_busy():
        root.after(1000, update_song_info)

# Vérifier si une musique est terminée
def check_music_end():
    if not pygame.mixer.music.get_busy() and current_song and not paused:
        # Si la musique est terminée et non en pause, passer à la suivante
        next_song()
    root.after(100, check_music_end)

# ================= INTERFACE =================

# Titre
ctk.CTkLabel(root, text="Music Player - Under Leo", font=("Arial", 20, "bold")).pack(pady=10)

# Affichage de la musique actuelle
song_label = ctk.CTkLabel(root, text="Aucune musique chargée", font=("Arial", 14), wraplength=350)
song_label.pack(pady=5)

# Information playlist
playlist_label = ctk.CTkLabel(root, text="Playlist: 0 musique(s)", font=("Arial", 12))
playlist_label.pack(pady=2)

# Status
status_label = ctk.CTkLabel(root, text="Status: Arrêté", font=("Arial", 12))
status_label.pack(pady=2)

# Boutons de contrôle
button_frame = ctk.CTkFrame(root)
button_frame.pack(pady=15)

ctk.CTkButton(button_frame, text="Précédent", command=previous_song, width=80).grid(row=0, column=0, padx=5)
ctk.CTkButton(button_frame, text="Suivant", command=next_song, width=80).grid(row=0, column=1, padx=5)

# Boutons principaux
ctk.CTkButton(root, text="Charger Musique(s)", command=load_song, width=250).pack(pady=5)
ctk.CTkButton(root, text="Lecture", command=play_song, width=250, fg_color="green").pack(pady=5)
ctk.CTkButton(root, text="Pause", command=pause_song, width=250).pack(pady=5)
ctk.CTkButton(root, text="Stop", command=stop_song, width=250, fg_color="red").pack(pady=5)

# Contrôle du volume
volume_frame = ctk.CTkFrame(root)
volume_frame.pack(pady=15)

ctk.CTkLabel(volume_frame, text="Contrôle du Volume", font=("Arial", 14)).pack(pady=5)

volume_label = ctk.CTkLabel(volume_frame, text="Volume: 50%")
volume_label.pack()

volume_slider = ctk.CTkSlider(
    volume_frame,
    from_=0,
    to=1,
    command=set_volume,
    width=250
)
volume_slider.set(0.5)
volume_slider.pack(pady=10)

# Informations
info_label = ctk.CTkLabel(root, text="Utilisez les boutons 'Précédent' et 'Suivant' \npour naviguer dans votre playlist", 
                          font=("Arial", 11), text_color="gray")
info_label.pack(pady=10)

# Démarrer la vérification de fin de musique
root.after(100, check_music_end)

root.mainloop()
