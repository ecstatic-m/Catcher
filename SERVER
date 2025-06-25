#importation des modules necessaires
import lzma
import socket
import json
import platform
import subprocess
import sys
import zlib

import psutil
import ctypes
import os
import cpuinfo
import pickle
# fonction qui vérifie si le script est exécuté avec les privilèges administratifs
def is_admin():
    if platform.system() == "Windows":
        try:
            return ctypes.windll.shell32.IsUserAnAdmin()
        except:
            return False
    elif platform.system() in ["Linux", "Darwin"]:
        # Sous Linux et macOS, vérifier si l'ID utilisateur est 0 (root)
        return os.geteuid() == 0
    else:
        # Pour les autres systèmes, on retourne False par défaut
        return False



if not is_admin():
    if platform.system() == "Windows":
        # Sur Windows, relance le script avec des privilèges administratifs
        ctypes.windll.shell32.ShellExecuteW(None, "runas", sys.executable, " ".join(sys.argv), None, 1)
        sys.exit()
    elif platform.system() in ["Linux", "Darwin"]:
        # Sous Linux et macOS, utilise pkexec ou sudo pour demander les droits administratifs
        try:
            # Tentative avec pkexec (plus sûr et offre une interface graphique)
            subprocess.run(["pkexec", sys.executable] + sys.argv)
        except FileNotFoundError:
            # Si pkexec n'est pas disponible, utilise sudo
            print("pkexec non disponible, tentative avec sudo.")
            subprocess.run(["sudo", sys.executable] + sys.argv)
        sys.exit()

HOST = '0.0.0.0'  # Écoute sur toutes les interfaces réseau
PORT = 12345 # Port utilisé par le serveur

# Fonction pour lister les périphériques USB
import subprocess
import platform


def storaged():
    devices = {}

    # Windows
    if platform.system() == "Windows":
        import win32com.client
        wmi = win32com.client.GetObject('winmgmts:')
        for disk in wmi.InstancesOf('Win32_LogicalDisk'):
            if disk.Size is not None:
                type_disk = "Disque amovible" if disk.DriveType == 2 else "Disque local"
                devices[disk.DeviceID] = f"Taille: {int(disk.Size) / (1024 ** 3):.2f} Go"

    # Linux
    elif platform.system() == "Linux":
        try:
            # Utilisation de la commande df pour récupérer les informations de disque
            result = subprocess.check_output(['df', '-h']).decode('utf-8')
            lines = result.splitlines()[1:]  # Ignorer la première ligne d'en-tête
            for line in lines:
                parts = line.split()
                device = parts[0]
                size = parts[1]
                devices[device] = f"Taille: {size}"
        except Exception as e:
            print(f"Erreur lors de la récupération des informations sous Linux: {e}")

    # macOS (Darwin)
    elif platform.system() == "Darwin":
        try:
            # Utilisation de la commande df pour récupérer les informations de disque sur macOS
            result = subprocess.check_output(['df', '-h']).decode('utf-8')
            lines = result.splitlines()[1:]  # Ignorer la première ligne d'en-tête
            for line in lines:
                parts = line.split()
                device = parts[0]
                size = parts[1]
                devices[device] = f"Taille: {size}"
        except Exception as e:
            print(f"Erreur lors de la récupération des informations sous macOS: {e}")

    return devices
def list_usb_devices():
    devices = []
    if platform.system() == "Windows":
        try:
            result = subprocess.check_output("wmic path CIM_LogicalDevice where \"Description like 'USB%'\" get /value",
                                             shell=True)
            result = result.decode().strip().split("\n\n")
            for device in result:
                device_info = {}
                for line in device.split("\n"):
                    if "=" in line:
                        key, value = line.split("=", 1)
                        device_info[key.strip()] = value.strip()
                devices.append(device_info)
        except subprocess.CalledProcessError as e:
            print(f"Error executing WMIC command: {e}")

    elif platform.system() == "Linux":
        try:
            result = subprocess.check_output("lsusb", shell=True)
            result = result.decode().strip().split("\n")
            for line in result:
                parts = line.split()
                device_info = {
                    'Bus': parts[1],
                    'Device': parts[3].strip(':'),
                    'ID': parts[5],
                    'Description': ' '.join(parts[6:])
                }
                devices.append(device_info)
        except subprocess.CalledProcessError as e:
            print(f"Error executing lsusb command: {e}")

    elif platform.system() == "Darwin":  # macOS
        try:
            result = subprocess.check_output("system_profiler SPUSBDataType", shell=True)
            result = result.decode().strip().split('\n\n')
            for device in result:
                device_info = {}
                for line in device.split('\n'):
                    if ':' in line:
                        key, value = line.split(':', 1)
                        device_info[key.strip()] = value.strip()
                devices.append(device_info)
        except subprocess.CalledProcessError as e:
            print(f"Error executing system_profiler command: {e}")

    print(devices)
    if len(devices) == 0 or all(len(d) == 0 for d in devices):
        devices = []
    return devices


def memory_catch():
    print("-------------------------------------")
    print("Memory:")
    mem = psutil.virtual_memory()
    print(mem)
    return mem
# Fonction pour obtenir les informations sur le CPU
def get_cpu_info():
    cpu_info = {
        'architecture': platform.architecture()[0],
        'processor': platform.processor(),
        'processor_name':f"{platform.processor()} {cpuinfo.get_cpu_info().get('brand_raw', 'Inconnu')}",
        'machine': platform.machine(),
        'system': platform.system(),
        'version': platform.version(),
        'platform': platform.platform()
    }
    return cpu_info



def battery():
    batteryresp={}
    battery = psutil.sensors_battery()
    if battery:
        batteryresp['Battery left charge']=f"{battery.percent}%"
        batteryresp['Battery plugged in']=f"{'Yes' if battery.power_plugged else 'No'}"
        print(f"\tBattery left charge: {battery.percent}%")
        print(f"\tBattery plugged in: {'Yes' if battery.power_plugged else 'No'}")
        if battery.secsleft not in (psutil.POWER_TIME_UNLIMITED, psutil.POWER_TIME_UNKNOWN):
            hours, minutes = divmod(battery.secsleft // 60, 60)
            batteryresp["Time left until discharge"]=f"{hours}H {minutes}Mn"
            print(f"\tTime left until discharge: {hours}H {minutes}Mn")
    return batteryresp
# Fonction pour gérer les clients


def get_connected_device_names():
    os_type = platform.system()

    if os_type == "Windows":
        command = "wmic path Win32_PnPEntity get Name"
    elif os_type == "Linux":
        command = "lsblk -o NAME"
    elif os_type == "Darwin":  # macOS
        command = "system_profiler SPUSBDataType | grep 'Product ID' -B 1 | grep 'Product:'"
    else:
        return {"error": "Unsupported OS"}

    try:
        result = subprocess.run(command, shell=True, capture_output=True)
        output = result.stdout.decode('utf-8', errors='ignore')
        device_names = {line.strip(): "Description ou ID" for line in output.splitlines() if line.strip()}
        return device_names
    except Exception as e:
        return {"error": str(e)}


def handle_client(client_socket):
    try:
        command = client_socket.recv(1024).decode()
        print(f"Commande reçue: {command}")

        if command == 'LIST_USB':
            usb_devices = list_usb_devices()  # Récupère la liste des périphériques USB
            if len(usb_devices)==0:  # Vérifie si la liste est vide
                response = json.dumps({"error": "Aucun périphérique USB trouvé."})
            else:
                response = json.dumps(usb_devices)  # Convertit la liste en JSON
            print(f"Réponse à envoyer: {response}")  # Afficher la réponse avant l'envoi
            client_socket.send(response.encode())  # Envoie la réponse au client
        elif command == 'CPU_INFO':
            cpu_info = get_cpu_info()  # Récupère les informations sur le CPU
            print("apres exec")
            response = json.dumps(cpu_info)
            print(f"Réponse à envoyer: {response}")  # Afficher la réponse avant l'envoi
            client_socket.send(response.encode())  # Envoie la réponse au client
        elif command == 'CREATE_LOCK':
            create_lock()
            client_socket.send(json.dumps({"success":"success"}).encode())
        elif command =='DEL_LOCK':
            os.remove(os.path.join(get_root_directory(), LOCK_FILE))
            client_socket.send(json.dumps({"success": "success"}).encode())
        elif command == 'MEM_INFO':
            memory=memory_catch()
            response = json.dumps(memory)
            print(f"Réponse à envoyer: {response}")  # Afficher la réponse avant l'envoi
            client_socket.send(response.encode())  # Envoie la réponse au client
        elif command=='BATTERY_INFO':
            response=json.dumps(battery())
            print(f"Réponse à envoyer: {response}")  # Afficher la réponse avant l'envoi
            client_socket.send(response.encode())  # Envoie la réponse au client
        elif command=='STORAGE_UNITS':
            response=json.dumps(storaged())
            print(f"Réponse à envoyer: {response}")
            client_socket.send(response.encode())
        elif command=='Devices':
            periphs=get_connected_device_names()
            print("Reponse a envoyer:",periphs)
            serialized_data = pickle.dumps(periphs)

            # Compresser les données avant de les envoyer
            compressed_data = lzma.compress(serialized_data)
            print(compressed_data)

            chunk_size=1024
            # Découper la liste en morceaux de `chunk_size`
            for i in range(0, len(compressed_data), chunk_size):
                chunk = compressed_data[i:i + chunk_size]
                client_socket.send(chunk)  # Envoyer chaque morceau
                print('sent',chunk)


            # Envoyer un signal de fin pour indiquer la fin de l'envoi
            client_socket.send(b"FIN")
            print('sent fin')

    except Exception as e:
        print(f"Erreur: {e}")
    finally:
        client_socket.close()  # Ferme la connexion avec le client
        release_lock()


# Fonction pour démarrer le serveur
def start_server():
    check_lock()
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)  # Permettre la réutilisation de l'adresse
    server_socket.bind((HOST, PORT))
    server_socket.listen(5)
    print(f"Serveur en écoute sur {HOST}:{PORT}")

    try:
        while True:
            client_socket, addr = server_socket.accept()
            print(f"Connexion de {addr}")
            handle_client(client_socket)
    except Exception:
        print("\nServeur interrompu. Fermeture...")
    finally:
        server_socket.close()
        release_lock()




def disable_battery_conditions(task_name):
    import win32com.client
    scheduler = win32com.client.Dispatch('Schedule.Service')
    scheduler.Connect()
    rootFolder = scheduler.GetFolder('\\')

    try:
        task = rootFolder.GetTask(task_name)
        definition = task.Definition
        settings = definition.Settings

        # Modifier les conditions de la tâche
        settings.StopIfGoingOnBatteries = False
        settings.DisallowStartIfOnBatteries = False

        # Enregistrer les modifications
        rootFolder.RegisterTaskDefinition(
            task_name,
            definition,
            6,  # TASK_CREATE_OR_UPDATE
            None,  # Utilisateur
            None,  # Mot de passe
            3  # TASK_LOGON_INTERACTIVE_TOKEN
        )
        print("Les conditions de la tâche ont été modifiées avec succès.")
    except Exception as e:
        print(f"Erreur : {e}")


def create_Linux():

    # Définir la commande à exécuter
    script_path = os.path.abspath(sys.argv[0])
    command = f"@reboot python3 {script_path}"

    # Ajouter la tâche cron
    subprocess.run(f'(crontab -l ; echo "{command}") | crontab -', shell=True, check=True)
    print("Tâche cron ajoutée avec succès.")


def create_darwin():

    # Définir le chemin du fichier plist
    plist_path = os.path.expanduser("~/Library/LaunchAgents/com.example.task.plist")

    # Contenu du fichier plist
    plist_content = f"""
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
    <dict>
        <key>Label</key>
        <string>com.example.task</string>
        <key>ProgramArguments</key>
        <array>
            <string>/usr/bin/python3</string>
            <string>{os.path.abspath(sys.argv[0])}</string>
        </array>
        <key>RunAtLoad</key>
        <true/>
    </dict>
    </plist>
    """

    # Écrire le fichier plist
    with open(plist_path, "w") as file:
        file.write(plist_content)

    # Charger le fichier plist avec launchctl
    subprocess.run(["launchctl", "load", plist_path], check=True)
    print("Tâche launchd ajoutée avec succès.")
    # Charger le fichier plist avec launchctl
    subprocess.run(["launchctl", "load", plist_path], check=True)
    print("Tâche launchd ajoutée avec succès.")


def check_task_exists_windows(task_name):
    try:
        # Exécute la commande pour vérifier la tâche
        subprocess.run(["schtasks", "/query", "/tn", task_name], check=True, stdout=subprocess.PIPE,
                       stderr=subprocess.PIPE)
        print(f"La tâche '{task_name}' existe.")
        return True
    except subprocess.CalledProcessError:
        print(f"La tâche '{task_name}' n'existe pas.")
        return False



#------------------------------------------------------------------------------------------------------------------------------------------



def create_task_windows(task_name):
    if (check_task_exists_windows(task_name) == False):
        pythonw_path = os.path.join(os.path.dirname(sys.executable), "pythonw.exe")

        # Chemin vers ton script
        script_path = os.path.abspath(sys.argv[0])


        # Commande pour créer la tâche planifiée
        command = [
            "schtasks",
            "/create",
            "/tn",f"{task_name}",
            "/tr", f"{script_path}",
            "/sc", "onlogon",
            "/rl","highest",
            "/it",
            "/f"
        ]

        # Exécuter la commande
        subprocess.run(command)
        disable_battery_conditions(task_name)
    else:
        print("La tache existe deja")


def get_root_directory():
    system = platform.system()  # Retourne 'Linux', 'Darwin', 'Windows', etc.

    if system == 'Linux' or system == 'Darwin':  # Linux ou macOS (Darwin)
        return '/'
    elif system == 'Windows':  # Windows
        return 'C:\\'  # Généralement, le répertoire racine sur Windows est C:
    else:
        raise Exception(f"Système {system} non pris en charge")



LOCK_FILE = "serveur.lock"  # Nom du fichier de verrouillage
def check_lock():
    path=os.path.join(get_root_directory(),LOCK_FILE)
    if os.path.exists(path):
        print("Processus existant")
        sys.exit(-1)
def create_lock():
    path=os.path.join(get_root_directory(),LOCK_FILE)
    # Si le fichier de verrouillage existe, cela signifie qu'une instance du programme est déjà en cours

    with open(path, 'w') as lock:
        lock.write("Server is running")

def release_lock():
    # Supprimer le fichier de verrouillage à la fin du programme
    if os.path.exists(LOCK_FILE):
        os.remove(os.path.join(get_root_directory(),LOCK_FILE))









#---------------------------------------------------------------------------------------------------------------------------


if __name__ == "__main__":
    if platform.system() == "Windows":
        create_task_windows("SERVEUR")
    elif platform.system() == "Linux":
        create_Linux()
    elif platform.system() == "Darwin":
        create_darwin()
    #subprocess.run(["schtasks","/create","/tn","SERVEUR","/tr",os.path.abspath(sys.argv[0]),"/sc","onlogon","/rl","highest","/f","/it"])
    start_server()
