import tkinter as tk
import bluetooth
from tkinter import messagebox
from functools import partial

clipboard_history = []

def get_clipboard_data(device_address):
    try:
        socket = bluetooth.BluetoothSocket(bluetooth.RFCOMM)
        socket.connect((device_address, 1))
        socket.send("GET_CLIPBOARD")
        data = socket.recv(1024)
        clipboard_text = data.decode("utf-8")
        clipboard_history.append(clipboard_text)  # Add clipboard text to history
        clipboard_label.config(text="Clipboard Text: " + clipboard_text)
        socket.close()
    except bluetooth.BluetoothError as e:
        clipboard_label.config(text="Bluetooth Error: " + str(e))

def clear_history():
    if clipboard_history:
        result = messagebox.askquestion("Clear History", "Are you sure you want to clear the history?")
        if result == "yes":
            clipboard_history.clear()
            history_text.delete('1.0', tk.END)
    else:
        messagebox.showinfo("Clear History", "The history is already empty.")

def show_history():
    history_text.delete('1.0', tk.END)
    for i, clipboard_text in enumerate(clipboard_history, start=1):
        copy_button = tk.Button(history_text, text="Copy", command=partial(copy_to_clipboard, clipboard_text), font=("Helvetica", 10))
        history_text.window_create(tk.END, window=copy_button)
        history_text.insert(tk.END, f" {i}. {clipboard_text}\n")
        
def copy_to_clipboard(clipboard_text):
    root.clipboard_clear()
    root.clipboard_append(clipboard_text)

root = tk.Tk()
root.title("Bluetooth Clipboard Access")

# Title label
title_label = tk.Label(root, text="Bluetooth Clipboard Access", font=("Helvetica", 16, "bold"))
title_label.pack(pady=20)

# Discover devices frame
discover_frame = tk.Frame(root)
discover_frame.pack()

# Discover devices button
discover_button = tk.Button(discover_frame, text="Discover Devices", command=discover_devices, font=("Helvetica", 12))
discover_button.pack(side=tk.LEFT, padx=5)

# Connection buttons frame
connection_frame = tk.Frame(root)
connection_frame.pack(pady=10)

# Scrollbar for device list
device_scrollbar = tk.Scrollbar(connection_frame)
device_scrollbar.pack(side=tk.RIGHT, fill=tk.Y)

# Device listbox
device_list = tk.Listbox(connection_frame, selectmode=tk.SINGLE, font=("Helvetica", 12),
                         yscrollcommand=device_scrollbar.set)
device_list.pack(side=tk.LEFT, padx=5)

# Attach scrollbar to device listbox
device_scrollbar.config(command=device_list.yview)

# Connect button
connect_button = tk.Button(root, text="Connect", command=lambda: connect_to_device(device_list.get(tk.ACTIVE)),
                           font=("Helvetica", 12))
connect_button.pack(pady=10)

# Clipboard text label
clipboard_label = tk.Label(root, text="Clipboard Text: ", font=("Helvetica", 14))
clipboard_label.pack(pady=10)

# History frame
history_frame = tk.Frame(root)
history_frame.pack()

# History button
history_button = tk.Button(history_frame, text="View History", command=show_history, font=("Helvetica", 12))
history_button.pack(side=tk.LEFT, padx=5)

# Clear history button
clear_history_button = tk.Button(history_frame, text="Clear History", command=clear_history, font=("Helvetica", 12))
clear_history_button.pack(side=tk.LEFT, padx=5)

# History text box
history_text = tk.Text(root, height=6, width=50, font=("Helvetica", 12))
history_text.pack()

# Separator line
separator_line = tk.Frame(root, height=2, bd=1, relief=tk.SUNKEN)
separator_line.pack(fill=tk.X, padx=20, pady=10)

# Status label
status_label = tk.Label(root, text="Status: Not Connected", font=("Helvetica", 12))
status_label.pack()

# Information label
info_label = tk.Label(root, text="Please select a device and click Connect to access the clipboard.", font=("Helvetica", 12))
info_label.pack(pady=10)

# Help label
help_label = tk.Label(root, text="For assistance, please refer to the user manual or contact support.", font=("Helvetica", 12), fg="blue")
help_label.pack(pady=10)

# Exit button
exit_button = tk.Button(root, text="Exit", command=root.quit, font=("Helvetica", 12), bg="red", fg="white")
exit_button.pack(pady=20, ipadx=10, ipady=5)

# Additional User Experience Enhancements

# UX Enhancement 1: Tooltips
import tkinter.ttk as ttk
tooltip1 = ttk.ToolTip()
tooltip1.set_text("Click this button to discover Bluetooth devices.")
tooltip1.wraplength = 200
tooltip1.configure(style="Info.TTLabel")
discover_button.bind("<Enter>", lambda event: tooltip1.show(discover_button, above=True))
discover_button.bind("<Leave>", lambda event: tooltip1.hide())

# UX Enhancement 2: Changing Cursor
connect_button.config(cursor="hand2")

# UX Enhancement 3: Keyboard Shortcut
root.bind("<Control-q>", lambda event: root.quit())

# UX Enhancement 4: Context Menu
context_menu = tk.Menu(root, tearoff=0)
context_menu.add_command(label="Copy", command=lambda: copy_to_clipboard(history_text.get(tk.SEL_FIRST, tk.SEL_LAST)))
context_menu.add_separator()
context_menu.add_command(label="Clear History", command=clear_history)
history_text.bind("<Button-3>", lambda event: context_menu.post(event.x_root, event.y_root))

# UX Enhancement 5: Styling
history_text.config(bg="lightgray", padx=5, pady=5)

# UX Enhancement 6: Progress Indicator
progress_indicator = ttk.Progressbar(root, mode="indeterminate")
progress_indicator.pack(pady=10)
progress_indicator.start()

# UX Enhancement 7: Keyboard Focus
device_list.focus_set()

# UX Enhancement 8: Dynamic Labels
status_var = tk.StringVar()
status_label.config(textvariable=status_var)
status_var.set("Status: Not Connected")

# UX Enhancement 9: Balloon Help
balloon_help = tk.Label(root, text="Hover over buttons for help", font=("Helvetica", 12))
balloon_help.pack(pady=10)
tooltip2 = tk.Label(root, text="Click this button to connect to the selected device", font=("Helvetica", 10))
tooltip2.pack()
tooltip2.bind("<Enter>", lambda event: balloon_help.config(text="Connect to the device"))
tooltip2.bind("<Leave>", lambda event: balloon_help.config(text="Hover over buttons for help"))

# UX Enhancement 10: Keyboard Navigation
root.option_add("*tearOff", False)
menu_bar = tk.Menu(root)
menu_file = tk.Menu(menu_bar)
menu_file.add_command(label="Connect", command=lambda: connect_to_device(device_list.get(tk.ACTIVE)))
menu_file.add_command(label="Exit", command=root.quit)
menu_bar.add_cascade(label="File", menu=menu_file)
root.config(menu=menu_bar)

root.mainloop()
