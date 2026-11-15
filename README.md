import os
import socket
import threading
import customtkinter as ctk
from PIL import Image


class RegistrationWindow(ctk.CTk):
    def __init__(self):
        super().__init__()

        self.title("Реєстрація")
        self.geometry("420x340")
        self.resizable(False, False)

        self.eval('tk::PlaceWindow . center')

        self.main_font = ("Segoe Script", 14)

        self.title_label = ctk.CTkLabel(
            self,
            text="Вхід в LogiTalk",
            font=("Segoe Script", 20, "bold"),
            text_color="lightblue"
        )
        self.title_label.pack(pady=(15, 10))

        self.name_entry = ctk.CTkEntry(
            self,
            placeholder_text="Введіть ім'я",
            placeholder_text_color="#8E8E93",
            height=36,
            width=340,
            corner_radius=4,
            fg_color="#F8F8FA",
            border_width=1,
            border_color="#D1D1D6",
            text_color="black",
            font=self.main_font
        )
        self.name_entry.pack(pady=6)

        self.host_entry = ctk.CTkEntry(
            self,
            placeholder_text="Введіть ХОСТ сервера",
            placeholder_text_color="#8E8E93",
            height=36,
            width=340,
            corner_radius=4,
            fg_color="#F8F8FA",
            border_width=1,
            border_color="#D1D1D6",
            text_color="black",
            font=self.main_font
        )
        self.host_entry.insert(0, "localhost")
        self.host_entry.pack(pady=6)

        self.port_entry = ctk.CTkEntry(
            self,
            placeholder_text="Введіть порт сервера",
            placeholder_text_color="#8E8E93",
            height=36,
            width=340,
            corner_radius=4,
            fg_color="#F8F8FA",
            border_width=1,
            border_color="#D1D1D6",
            text_color="black",
            font=self.main_font
        )
        self.port_entry.insert(0, "8080")
        self.port_entry.pack(pady=6)

        self.register_btn = ctk.CTkButton(
            self,
            text="Зареєструватися",
            height=38,
            width=340,
            corner_radius=4,
            fg_color="#229ED9",
            hover_color="#1D89BD",
            font=self.main_font,
            command=self.on_register
        )
        self.register_btn.pack(pady=(10, 15))

        self.user_data = None

    def on_register(self):
        name = self.name_entry.get().strip()
        host = self.host_entry.get().strip()
        port = self.port_entry.get().strip()

        if not name:
            name = "Гість"

        if not host:
            host = "localhost"

        try:
            port = int(port)
        except ValueError:
            port = 8080

        self.user_data = {"name": name, "host": host, "port": port}
        self.destroy()


class MessengerWindow(ctk.CTk):
    def __init__(self, user_data):
        super().__init__()

        self.user_data = user_data

        self.title("LogiTalk")
        self.geometry("900x550")
        self.minsize(800, 500)
        self.resizable(True, True)

        self.main_font = ("Segoe Script", 15)
        self.chat_font = ("Segoe Script", 15)

        self.profile_panel_open = False

        self.grid_columnconfigure(0, weight=0, minsize=280)
        self.grid_columnconfigure(1, weight=1)
        self.grid_rowconfigure(0, weight=1)

        self.left_container = ctk.CTkFrame(self, fg_color="#F8F9FA", corner_radius=0, width=280)
        self.left_container.grid(row=0, column=0, sticky="nsew")
        self.left_container.grid_propagate(False)
        self.left_container.grid_columnconfigure(0, weight=1)
        self.left_container.grid_rowconfigure(2, weight=1)

        avatar_img = None
        if os.path.exists("images.png"):
            avatar_img = ctk.CTkImage(light_image=Image.open("images.png"), size=(35, 35))

        self.avatar_btn = ctk.CTkButton(
            self.left_container,
            text="",
            image=avatar_img,
            width=45,
            height=45,
            corner_radius=0,
            fg_color="transparent",
            hover_color="#E5E5EA",
            command=self.toggle_profile_panel
        )
        self.avatar_btn.grid(row=0, column=0, sticky="nw", padx=15, pady=15)

        self.profile_frame = ctk.CTkFrame(self.left_container, fg_color="#EFEFF4", corner_radius=0)

        self.name_entry = ctk.CTkEntry(
            self.profile_frame,
            placeholder_text="введіть ім'я",
            placeholder_text_color="#8E8E93",
            height=35,
            corner_radius=0,
            fg_color="white",
            text_color="black",
            border_width=0,
            font=self.main_font
        )
        self.name_entry.pack(fill="x", padx=10, pady=10)
        self.name_entry.insert(0, self.user_data["name"])

        self.input_frame = ctk.CTkFrame(self.left_container, fg_color="transparent", corner_radius=0)
        self.input_frame.grid(row=3, column=0, sticky="ew", padx=15, pady=15)
        self.input_frame.grid_columnconfigure(0, weight=1)

        self.msg_entry = ctk.CTkEntry(
            self.input_frame,
            placeholder_text="Введіть повідомлення...",
            placeholder_text_color="#8E8E93",
            height=40,
            corner_radius=0,
            fg_color="white",
            text_color="black",
            border_width=1,
            border_color="#D1D1D6",
            font=self.main_font
        )
        self.msg_entry.grid(row=0, column=0, sticky="ew", padx=(0, 8))
        self.msg_entry.bind("<Return>", lambda event: self.send_message())

        send_img = None
        if os.path.exists("telegram.jfif"):
            send_img = ctk.CTkImage(light_image=Image.open("telegram.jfif"), size=(22, 22))

        self.send_btn = ctk.CTkButton(
            self.input_frame,
            text="",
            image=send_img,
            width=40,
            height=40,
            corner_radius=0,
            fg_color="#229ED9",
            hover_color="#1D89BD",
            command=self.send_message
        )
        self.send_btn.grid(row=0, column=1, sticky="e")

        self.right_container = ctk.CTkFrame(self, fg_color="#FFE5D0", corner_radius=0)
        self.right_container.grid(row=0, column=1, sticky="nsew")
        self.right_container.grid_columnconfigure(0, weight=1)
        self.right_container.grid_rowconfigure(0, weight=1)

        self.chat_box = ctk.CTkScrollableFrame(
            self.right_container,
            fg_color="#FFF3EB",
            scrollbar_button_color="#8B4513",
            scrollbar_button_hover_color="#5C2E0B"
        )
        self.chat_box.grid(row=0, column=0, sticky="nsew", padx=10, pady=10)
        self.chat_box.grid_columnconfigure(0, weight=1)

        self.connect_to_server()

    def toggle_profile_panel(self):
        if self.profile_panel_open:
            self.profile_frame.grid_forget()
            self.profile_panel_open = False
        else:
            self.profile_frame.grid(row=1, column=0, sticky="ew", padx=15, pady=(0, 10))
            self.profile_panel_open = True

    def add_message(self, message):
        message_frame = ctk.CTkFrame(self.chat_box, fg_color="white", corner_radius=10)
        message_frame.pack(fill="x", pady=5, anchor="w")
        message_frame.grid_columnconfigure(0, weight=1)

        wraplength_size = 900 - 280 - 40
        msg_label = ctk.CTkLabel(
            message_frame,
            text=message,
            text_color="#8A3B00",
            font=self.chat_font,
            justify="left",
            wraplength=wraplength_size
        )
        msg_label.pack(padx=10, pady=5, anchor="w")

    def connect_to_server(self):
        try:
            host = self.user_data["host"]
            port = self.user_data["port"]
            name = self.user_data["name"]

            self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            self.sock.connect((host, port))
            self.sock.send(name.encode())

            threading.Thread(target=self.recv_message, daemon=True).start()
        except Exception as e:
            self.add_message(f"Не вдалося підключитися до сервера: {e}")

    def recv_message(self):
        while True:
            try:
                message = self.sock.recv(1024).decode()
                if message:
                    self.after(0, lambda: self.add_message(message))
            except:
                break

    def send_message(self):
        message = self.msg_entry.get()
        name = self.name_entry.get() if hasattr(self, "name_entry") else self.user_data["name"]

        if message:
            full_message = f"{name}: {message}"
            try:
                self.sock.send(full_message.encode())
                self.add_message(full_message)
                self.msg_entry.delete(0, "end")
            except Exception as e:
                print(f"Помилка відправки: {e}")


if __name__ == "__main__":
    reg_window = RegistrationWindow()
    reg_window.mainloop()

    if reg_window.user_data:
        app = MessengerWindow(reg_window.user_data)
        app.mainloop()
