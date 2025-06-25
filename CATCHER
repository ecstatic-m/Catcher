import lzma
import platform
import subprocess
import psutil
import cpuinfo
import socket
import json
import os
import ctypes
import sys
import pickle
import tkinter as tk
from tkinter import scrolledtext, messagebox, simpledialog
# importation de l'outil Nmap
try:
    import nmap  # assuré vous que nmap est installé en utilisant la commande : pip install python-nmap
except ImportError:
    print("Module 'nmap' not installed. Install it with 'pip install python-nmap'.")

# fonction qui vérifie si le script est exécuté avec les privilèges administratifs
def is_admin():
    if platform.system() == "Windows": #si le OS ciblé et un windows
        try:
            return ctypes.windll.shell32.IsUserAnAdmin()
        except:
            return False
    elif platform.system() in ["Linux", "Darwin"]: #si le os ciblé est un linux ou mac OS
        return os.geteuid() == 0
    return False

# relance le script avec des privilèges d'administrateur si nécessaire
if not is_admin():
    if platform.system() == "Windows":
        ctypes.windll.shell32.ShellExecuteW(None, "runas", sys.executable, " ".join(sys.argv), None, 1)
        sys.exit()
    elif platform.system() in ["Linux", "Darwin"]:
        try:
            subprocess.run(["pkexec", sys.executable] + sys.argv)
        except FileNotFoundError:
            print("pkexec not available, trying with sudo.")
            subprocess.run(["sudo", sys.executable] + sys.argv)
        sys.exit()


# Fonction pour détecter les informations du système d'exploitation
def osdetect():
    return f"OS: {platform.system()}\nOS version: {platform.version()}\nMachine type: {platform.machine()}\n\n"

# Fonction pour détecter les périphériques connectés
def periph():
    result = "Peripherals:\n"
    if platform.system() == "Linux":
        try:

            result += subprocess.check_output("lspci", shell=True).decode() + "\n"
            result += subprocess.check_output("lsblk -o NAME,SIZE,TYPE,MOUNTPOINT", shell=True).decode() + "\n"
            result += subprocess.check_output("ip link show", shell=True).decode()
        except subprocess.CalledProcessError as e:
            result += f"Error in executing command: {e}\n"
    elif platform.system() == "Windows":
        import win32com.client
        wmi = win32com.client.GetObject('winmgmts:')
        for device in wmi.InstancesOf('Win32_PnPEntity'):
            result += f"Name: {device.Name}, ID: {device.DeviceID}\n"
        for disk in wmi.InstancesOf('Win32_LogicalDisk'):
            result += f"Disk: {disk.DeviceID}, Type: {'Removable' if disk.DriveType == 2 else 'Local'}, Size: {int(disk.Size) / (1024 ** 3):.2f} GB\n"
    return result + "\n"

# Fonction pour détecter les informations sur le processeur
def cpudetect():
    cpu_info = f"CPU: {platform.processor()} {cpuinfo.get_cpu_info().get('brand_raw', 'Unknown')}\n" #model du processeur
    cpu_info += f"Logical cores: {psutil.cpu_count(logical=True)}, Physical cores: {psutil.cpu_count(logical=False)}\n" #nombre core et de thread
    cpu_freq = psutil.cpu_freq()
    cpu_info += f"Frequency: {cpu_freq.current} MHz, Max: {cpu_freq.max} MHz\n\n" #la frequence du cpu
    return cpu_info

# Fonction pour détecter les informations sur la ram
def memory_info():
    mem = psutil.virtual_memory()
    return f"Memory: Total: {mem.total / 1024 ** 3:.2f} GB, Used: {mem.used / 1024 ** 3:.2f} GB\n\n"  #affiche la mémoire totale et utilisée







def get_root_directory():
    system = platform.system()  # Retourne 'Linux', 'Darwin', 'Windows', etc.

    if system == 'Linux' or system == 'Darwin':  # Linux ou macOS (Darwin)
        return '/'
    elif system == 'Windows':  # Windows
        return 'C:\\'  # Généralement, le répertoire racine sur Windows est C:
    else:
        raise Exception(f"Système {system} non pris en charge")







def release_lock():
    # Supprimer le fichier de verrouillage à la fin du programme
    if os.path.exists(os.path.join(get_root_directory(),'server.lock')):
        os.remove(os.path.join(get_root_directory(),'server.lock'))





# Fonction pour détecter les informations sur la batterie

def battery_info():
    battery = psutil.sensors_battery()
    if battery:
        info = f"Battery: Charge: {battery.percent}%, Plugged in: {'Yes' if battery.power_plugged else 'No'}\n" #affiche le pourcentage de la batterie et si le secteur et branché
        if battery.secsleft not in (psutil.POWER_TIME_UNLIMITED, psutil.POWER_TIME_UNKNOWN):
            hours, minutes = divmod(battery.secsleft // 60, 60) #si le secteur n'est pas branché il affiche le temps retant avant decharge
            info += f"Time left until discharge: {hours}H {minutes}Mn\n\n"
    else:
        info = "Battery information not available.\n\n" #si les informations sont pas dispobile
    return info

#la fonction qui sert a envoyé les commandes a la machine cible
def send_command(target_ip, server_port, command):  #la fonction prend 3 parametres l'@ ip de la cible le port serveur et la commande
    print("sending",command)
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as client_socket:
        client_socket.connect((target_ip, server_port)) #on se connect au client en utilisant les socket
        client_socket.send(command.encode()) #la commande est envoyé a la cible
        if command=='Devices':
            data = b"" #Remplacer "" par "FIN" si vous voulez faire sur la MEME machine autrement laissez comme ca
            while True:
                chunk = client_socket.recv(1024)
                print("received",chunk)
                if chunk == b"":
                    print(chunk)
                    break
                data += chunk

            # Désérialiser les données pour obtenir la liste

            print("avant return")
            print(data)
            return data
        else:
         data = client_socket.recv(4096).decode('utf-8', errors='ignore')
    if not data:
        print("not data")
        raise ValueError("Aucune donnée reçue du serveur.")
    return json.loads(data)

# Fonction du scan local en utilisant linterface graphique tkinter
def run_local_scan(scan_type):
    output_text.delete(1.0, tk.END)
    output_text.insert(tk.END, f"Local System Scan - {scan_type}:\n")

    if scan_type == 'OS':
        output_text.insert(tk.END, osdetect())
    elif scan_type == 'Peripherals':
        output_text.insert(tk.END, periph())
    elif scan_type == 'CPU':
        output_text.insert(tk.END, cpudetect())
    elif scan_type == 'Memory':
        output_text.insert(tk.END, memory_info())
    elif scan_type == 'Battery':
        output_text.insert(tk.END, battery_info())

#fonction pour le scan global a distance
def run_global_scan():
    target_ip = simpledialog.askstring("Global Scan", "Enter Target IP:") #introduir l'@ ip de la cible
    server_port = simpledialog.askinteger("Global Scan", "Enter Server Port:") #le port serveur utilisé dans notre cas ça sera 12345

    if target_ip and server_port: #si les informations requises sont introduits et la connexion est etablit tout les commande sont envoyer
        output_text.delete(1.0, tk.END)
        output_text.insert(tk.END, f"Global System Scan for {target_ip}:\n")
        try:
            send_command(target_ip,server_port,'CREATE_LOCK')
            cpu_info = send_command(target_ip, server_port, 'CPU_INFO')
            if cpu_info:
                output_text.insert(tk.END, "CPU and OS Information:\n")
                for key, value in cpu_info.items():
                    output_text.insert(tk.END, f"\t{key}: {value}\n")
            memory = send_command(target_ip, server_port, 'MEM_INFO')
            if memory:
                output_text.insert(tk.END, "Memory Info:\n")
                output_text.insert(tk.END,
                                   f"\tTotal: {memory[0] / 1024 ** 3:.2f} GB, Used: {memory[3] / 1024 ** 3:.2f} GB\n")


            devices = send_command(target_ip, server_port, 'Devices')
            if devices:
                output_text.insert(tk.END, "All Devices:\n")
                decompress=lzma.decompress(devices)
                data=pickle.loads(decompress)
                for res in data:
                 output_text.insert(tk.END, f"\tDevice: {res}\n")

            battery_info = send_command(target_ip, server_port, 'BATTERY_INFO')
            if battery_info:
                output_text.insert(tk.END, "Battery Info:\n")
                for key, value in battery_info.items():
                    output_text.insert(tk.END, f"\t{key}: {value}\n")

            external_devices=send_command(target_ip,server_port,'STORAGE_UNITS')
            if external_devices:
                output_text.insert(tk.END,"External devices and disks:\n")
                for de in external_devices:
                    output_text.insert(tk.END,f"\t{de}:{external_devices[de]}\n")
            send_command(target_ip, server_port, 'DEL_LOCK')
        except Exception as e:
            messagebox.showerror("Error", f"Communication Error: {e}")


# Creation de l'interface graphique avec Tkinter
root = tk.Tk()
root.title("Catcher System Scanner")
root.geometry("750x600")
root.configure(bg="#1f2e2e")

header_frame = tk.Frame(root, bg="#34495e", padx=10, pady=10)
header_frame.pack(fill="x")

title_label = tk.Label(header_frame, text="Catcher System Scanner", font=("Arial", 18, "bold"), fg="white",
                       bg="#34495e")
title_label.pack(pady=5)

scan_options_frame = tk.Frame(root, bg="#1f2e2e")
scan_options_frame.pack(pady=10)

local_scan_label = tk.Label(scan_options_frame, text="Select Local Scan Option:", font=("Arial", 14, "bold"),
                            fg="white", bg="#1f2e2e")
local_scan_label.grid(row=0, column=0, columnspan=5, pady=(5, 10))

btn_os = tk.Button(scan_options_frame, text="OS Info", command=lambda: run_local_scan('OS'), width=12,
                   font=("Arial", 10), bg="#3498db", fg="white")
btn_os.grid(row=1, column=0, padx=5, pady=5)

btn_cpu = tk.Button(scan_options_frame, text="CPU Info", command=lambda: run_local_scan('CPU'), width=12,
                    font=("Arial", 10), bg="#3498db", fg="white")
btn_cpu.grid(row=1, column=1, padx=5, pady=5)

btn_memory = tk.Button(scan_options_frame, text="Memory Info", command=lambda: run_local_scan('Memory'), width=12,
                       font=("Arial", 10), bg="#3498db", fg="white")
btn_memory.grid(row=1, column=2, padx=5, pady=5)

btn_battery = tk.Button(scan_options_frame, text="Battery Info", command=lambda: run_local_scan('Battery'), width=12,
                        font=("Arial", 10), bg="#3498db", fg="white")
btn_battery.grid(row=1, column=3, padx=5, pady=5)

btn_periph = tk.Button(scan_options_frame, text="Peripheral Info", command=lambda: run_local_scan('Peripherals'),
                       width=12, font=("Arial", 10), bg="#3498db", fg="white")
btn_periph.grid(row=1, column=4, padx=5, pady=5)

global_scan_label = tk.Label(scan_options_frame, text="Global Scan (IP-based):", font=("Arial", 14, "bold"), fg="white",
                             bg="#1f2e2e")
global_scan_label.grid(row=2, column=0, columnspan=5, pady=(15, 10))

btn_global_scan = tk.Button(scan_options_frame, text="Run Global Scan", command=run_global_scan, width=15,
                            font=("Arial", 10, "bold"), bg="#e74c3c", fg="white")
btn_global_scan.grid(row=3, column=0, columnspan=5, pady=5)

output_text = scrolledtext.ScrolledText(root, width=80, height=20, font=("Arial", 10), bg="#34495e", fg="white")
output_text.pack(pady=10)

root.mainloop()
