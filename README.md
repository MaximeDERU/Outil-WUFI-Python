# Outil-WUFI-Python
Cet outil permet de, à partir d'un fichier météo et de certaines informations sur le local étudier, déterminer les conditions hygrothermiques intérieures du local conformément au guide PACTE

# Veuillez éxécuter le programme et suivre les instructions

#Importation des bibliothèques
import pandas as pd
import chardet
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.dates as mdates
import tkinter as tk
from tkinter import ttk
from tkinter import filedialog
import pvlib
from pvlib.iotools import read_epw
import os

# Determination de constantes, ne pas modifier
R = 8.314462618
Tcons = 22.0
HRcons = 0.5
Tbf = 0.0

# Création de la fonction permettant d'afficher la fenêtre pour le choisir le fichier météo

def select_input_file():
    # Initialisation de l'interface Tkinter
    root = tk.Tk()
    root.title("Sélection du fichier météo")
    root.geometry("400x200")
    root.attributes('-topmost', True)

    # Ajout de l'instruction écrite
    instruction = tk.Label(root, text="Veuillez sélectionner un fichier météo au format .epw")
    instruction.pack(pady=20)

    file_path = None

    # Création de la fonction permettant d'aller chercher le fichier météo au format .epw
    def open_file_dialog():
        nonlocal file_path
        # Ouvrir la boîte de dialogue de sélection de fichier
        file_path = filedialog.askopenfilename(
            title="Sélectionner un fichier",
            filetypes=(("Tous les fichiers", "*.*"), ("Fichiers texte", "*.txt"))
        )
        root.destroy()  # Ferme la fenêtre Tkinter après la sélection du fichier

    # Ajout d'un bouton pour ouvrir la boîte de dialogue de sélection de fichier
    select_button = tk.Button(root, text="Choisir un fichier", command=open_file_dialog)
    select_button.pack(pady=20)

    root.mainloop()

    return file_path

# Affichage du nom du fichier selectionné afin de verifier que le script c'est éxécuté correctement
if __name__ == "__main__":
    input_epw_filepath = select_input_file()
    if input_epw_filepath:
        print(f"Fichier sélectionné : {input_epw_filepath}")
    else:
        print("Aucun fichier sélectionné.")

# Création de la fenêtre pour le choisir le dossier où seront importés les résultats

def select_output_folder():
    # Initialisation de l'interface Tkinter
    root = tk.Tk()
    root.title("Sélectionner du dossier")
    root.geometry("400x200")
    root.attributes('-topmost', True)

    # Ajout d'une instruction écrite
    instruction = tk.Label(root, text="Veuillez sélectionner le dossier dans lequel vous voulez exporter les résultats")
    instruction.pack(pady=20)

    folder_path = None

    def open_folder_dialog():
        nonlocal folder_path
        # Ouvrir la boîte de dialogue de sélection de fichier
        folder_path = filedialog.askdirectory()
        root.destroy()  # Ferme la fenêtre Tkinter après la sélection

    # Ajout d'un bouton pour ouvrir la boîte de dialogue de sélection de fichier
    select_button = tk.Button(root, text="Choisir un fichier", command=open_folder_dialog)
    select_button.pack(pady=20)

    root.mainloop()

    return folder_path

# Exécution du script pour sélectionner un fichier
if __name__ == "__main__":
    Output_folder = select_output_folder()
    if Output_folder:
        print(f"Fichier sélectionné : {Output_folder}")
    else:
        print("Aucun fichier sélectionné.")

# Enregistrement du fichier météo contenant les conditions intérieures
additional_file_name_1 = "Conditions_Intérieures.epw"
Chemin_Fichier_epw = os.path.join(Output_folder, additional_file_name_1 )
with open(Chemin_Fichier_epw, 'w') as file:
    file.write("Contenu pour file4.epw\n")

pd.set_option('display.max_columns', None)

def read_epw_with_pvlib(input_epw_filepath):
    # Utiliser la fonction pvlib pour lire le fichier EPW
    weather_data, metadata = pvlib.iotools.read_epw(input_epw_filepath)
    
    # La fonction read_epw de pvlib renvoie un DataFrame et un dictionnaire de métadonnées
    df = pd.DataFrame(weather_data)
    
    return df, metadata

# Exemple d'utilisation - chemin d'accès ici

df, metadata = read_epw_with_pvlib(input_epw_filepath)

df = df.rename(columns={'temp_air': 'Text_0', 'relative_humidity': 'HRext'})

print("Premières lignes du DataFrame :")
print(df.head())

def write_epw_file(df, metadata, additional_file_name_1):

# Ecrit un Dataframe dans un fichier EPW.

    # Créaton de l'en-tête du fichier EPW
    header_lines = []
    for key, value in metadata.items():
        if isinstance(value, list):
            value = ",".join(map(str, value))
        header_lines.append(f"{key.upper()} = {value}")

    header = "\n".join(header_lines)
    
    # Create the data part of the EPW file
    data_lines = df.to_csv(index=False, header=False)
    
    # Combine header and data
    epw_content = header + "\n" + data_lines
    
    # Write to file
    with open(additional_file_name, 'w') as file:
        file.write(epw_content)

# Création du graphique des températures et humidité relative extérieures
dates = pd.date_range(start='2005-01-01', periods=8760, freq='H')

# Tracer le graphique
plt.figure(figsize=(14, 7))
plt.plot(np.arange(len(df)), df['Text_0'], label='Température extérieure en °C', color = "r")
plt.plot(np.arange(len(df)), df['HRext'], label='Humidité relative en %', color = "b")

# Définir l'échelle des abscisses
plt.xticks(np.linspace(0, len(df)-1, num=13), np.linspace(0, 8760, num=13, dtype=int))

plt.xlabel("Temps en nombre d'heure depuis le début de l'année, 1 mois = 730h")
plt.ylabel("Valeurs")
plt.yticks(np.linspace(0, 100, num = 11))
plt.ylim(0, 100)
plt.legend()
plt.title("Température et humidité extérieures")
plt.grid()

plt.tight_layout()
#plt.show()

figure_path = os.path.join(Output_folder, "Climat_Extérieur.png")

plt.savefig(figure_path)
print(f"Courbe sauvegardée avec succès dans {figure_path}.")

# Réalisation de l'interface graphique pour choisir les caractéristiques de traitement de l'air du local
def set_constants():
    global Tcons, HRcons, Tbf
    
    # Initialiser les variables globales avec des valeurs par défaut
    if air_treatment.get() == "Climatisé":
        try:
            Tcons = float(entry_Tcons.get())
            Tbf = float(entry_Tbf.get())
        except ValueError:
            print("Veuillez entrer des valeurs valides pour Tcons et Tbf")
            return
    elif air_treatment.get() == "Déshumidifié":
        try:
            HRcons = float(entry_HRcons.get())
        except ValueError:
            print("Veuillez entrer une valeur valide pour HRcons")
            return

    print(f"Tcons: {Tcons}, HRcons: {HRcons}, Tbf: {Tbf}")
    root.quit()
    root.destroy()

def update_entries(event):
    treatment = air_treatment.get()
    if treatment == "Climatisé":
        label_Tcons.grid(row=2, column=0, padx=10, pady=10, sticky="e")
        entry_Tcons.grid(row=2, column=1, padx=10, pady=10)
        label_Tbf.grid(row=3, column=0, padx=10, pady=10, sticky="e")
        entry_Tbf.grid(row=3, column=1, padx=10, pady=10)
        label_HRcons.grid_forget()
        entry_HRcons.grid_forget()
    elif treatment == "Déshumidifié":
        label_HRcons.grid(row=2, column=0, padx=10, pady=10, sticky="e")
        entry_HRcons.grid(row=2, column=1, padx=10, pady=10)
        label_Tcons.grid_forget()
        entry_Tcons.grid_forget()
        label_Tbf.grid_forget()
        entry_Tbf.grid_forget()
    elif treatment == "Non traité":
        label_Tcons.grid_forget()
        entry_Tcons.grid_forget()
        label_HRcons.grid_forget()
        entry_HRcons.grid_forget()
        label_Tbf.grid_forget()
        entry_Tbf.grid_forget()

# Création de la fenêtre principale
root = tk.Tk()
root.title("Choix des constantes")
root.geometry("500x300")
root.attributes('-topmost', True)

# Instructions
instructions = tk.Label(root, text="Remplissez les valeurs requises puis cliquez sur Set Constants")
instructions.grid(row=0, column=0, columnspan=2, padx=10, pady=10)

# Variable associée au choix du traitement de l'air
air_treatment = tk.StringVar(value="Climatisé")

# Création du label et du menu déroulant pour le traitement de l'air
label_treatment = tk.Label(root, text="Comment est traité l'air au sein du local ?")
label_treatment.grid(row=1, column=0, padx=10, pady=10, sticky="e")
combo_treatment = ttk.Combobox(root, textvariable=air_treatment, values=["Climatisé", "Déshumidifié", "Non traité"], state="readonly")
combo_treatment.grid(row=1, column=1, padx=10, pady=10)
combo_treatment.bind("<<ComboboxSelected>>", update_entries)

# Variables associées aux entrées
Tcons_var = tk.StringVar(value="22")
HRcons_var = tk.StringVar(value="0.5")
Tbf_var = tk.StringVar(value="0")

# Création des labels et des entrées pour Tcons, HRcons, et Tbf
label_Tcons = tk.Label(root, text="Tcons:")
entry_Tcons = ttk.Entry(root, textvariable=Tcons_var)

label_HRcons = tk.Label(root, text="HRcons:")
entry_HRcons = ttk.Entry(root, textvariable=HRcons_var)

label_Tbf = tk.Label(root, text="Tbf:")
entry_Tbf = ttk.Entry(root, textvariable=Tbf_var)

# Bouton pour définir les constantes
button_set = ttk.Button(root, text="Set Constants", command=set_constants)
button_set.grid(row=4, column=0, columnspan=2, pady=10)

# Mise à jour des entrées en fonction du choix initial
update_entries(None)

root.mainloop()

# Utilisation des constantes dans les calculs
print(f"Tcons: {Tcons}, HRcons: {HRcons}, Tbf: {Tbf}")

# Création de la moyenne glissante des valeurs de la température extérieure sur 24h - Guide PACTE page 11

# Ajout des 23 dernières valeurs de 'Text(0)' au début du DataFrame pour l'étendre
extended_values_24 = pd.concat([df['Text_0'].iloc[-23:], df['Text_0']])

# Calcul de la moyenne glissante sur 24 valeurs de la colonne étendue 'Text(0)'
rolling_mean_24 = extended_values_24.rolling(window=24).mean()

# Suppression des 23 premières lignes du résultat pour obtenir la taille originale du DataFrame
df['Text_24h'] = rolling_mean_24.iloc[23:].values

# Création de la moyenne glissante des valeurs de la température extérieure sur 168h - Guide PACTE page 11

# Ajout des 167 dernières valeurs de 'Text(0)' au début du DataFrame pour l'étendre
extended_values_168 = pd.concat([df['Text_0'].iloc[-167:], df['Text_0']])

# Calcul de la moyenne glissante sur 168 valeurs de la colonne étendue 'Text(0)'
rolling_mean_168 = extended_values_168.rolling(window=168).mean()

# Suppression les 167 premières lignes du résultat pour obtenir la taille originale du DataFrame
df['Text_168h'] = rolling_mean_168.iloc[167:].values

# Affichage des premières lignes du DataFrame avec la moyenne glissante
# print(df.head(2))

# Création de la courbe de la température intérieure en fonction de la moyenne glissante de la température extérieure sur 24h
# Guide PACTE page 9
df['Tint_0'] = np.where(
    df['Text_24h'] <= 10,
    20,
    np.where(df['Text_24h'] <= 26, 
    0.5 * df['Text_24h'] + 15,
    28)
)

# Création de la courbe permettant de determiner dw en fonction de la moyenne glissante de la température extérieure sur 168h pour une classe d'hygrométrie faible
# Guide PACTE Page 9
df['dW_1'] = np.where(
    df['Text_168h'] <= 0,
    2.5,
    np.where(
        df['Text_168h'] <= 20,
        - 0.075 * df['Text_168h'] + 2.5,
        1))

# Création de la courbe permettant de determiner dw en fonction de la moyenne glissante de la température extérieure sur 168h pour une classe d'hygrométrie moyenne
df['dW_2'] = np.where(
    df['Text_168h'] <= 0,
    5,
    np.where(
        df['Text_168h'] <= 20,
        - 0.2 * df['Text_168h'] + 5,
        1))

# Création de la courbe permettant de determiner dw en fonction de la moyenne glissante de la température extérieure sur 168h pour une classe d'hygrométrie forte
df['dW_3'] = np.where(
    df['Text_168h'] <= 0,
    7.5,
    np.where(
        df['Text_168h'] <= 20,
        - 0.325 * df['Text_168h'] + 7.5,
        1))

# Choix du type de local permettant de mettre à jour automatiquemet dW en fonction des classes d'hygrométries associées
# Création de la fenêtre principale
root = tk.Tk()
root.geometry('950x500')
root.attributes('-topmost', True)

# Dictionnaire de mappage des classes d'hygrométrie aux colonnes du DataFrame
choix_local = {
    "Immeubles de bureaux non conditionnés": 'dW_1',
    "Externats scolaires": 'dW_1',
    "Logements équipés de ventilations mécaniques contrôlées et de systèmes propres à évacuer les pointes de production de vapeur d eau dès qu elles se produisent (hottes etc…)": 'dW_1', 
    "Bâtiments industriels à usage de stockage":'dW_1', 
    "Ateliers mécaniques sans production de vapeur d'eau": 'dW_1', 
    "Locaux sportifs sans public(sauf piscine ou patinoires)": 'dW_1',
    "Bâtiments d’habitation, y compris les cuisines et salles d’eau, correctement chauffés et ventilés, sans suroccupation": 'dW_2',
    "Locaux avec forte concentration humaine ou animale (bâtiments d’élevage, manèges couverts de chevaux, certains ateliers etc…": 'dW_3',
    "Locaux à atmosphère humide contrôlée pour les besoins de la fabrication des produits (boulangeries et pâtisseries industrielles, imprimeries, tannage des cuirs etc…)": 'dW_3',
    "Locaux avec forte production de vapeur d’eau (piscines, conserveries, laiteries, ateliers de lavage de bouteilles, brasseries, cuisines collectives, blanchisseries, etc…)": 'dW_3',
    "Locaux chauffés par panneaux radiants à combustible gaz": 'dW_3',
    "Locaux spéciaux tels que locaux nécessitant le maintien d’une humidité relativement élevée, locaux sanitaires de collectivités d’utilisation très fréquente": 'dW_3'
}

def action(event):
    select = listecombo.get()
    df['dW'] = df[choix_local[select]]
    print("Vous avez sélectionné :", select)
    print("df['dW'] a été mis à jour :\n", df['dW'])
    root.quit()
    root.destroy()

# Création du label qui indique à l'utilisateur de faire un choix
labelchoix = tk.Label(root, text="Choisissez le type de local")
labelchoix.pack()

# Création de la liste des éléments du combobox
TypeLocal = ["Immeubles de bureaux non conditionnés", "Externats scolaires",
"Logements équipés de ventilations mécaniques contrôlées et de systèmes propres à évacuer les pointes de production de vapeur d eau dès qu elles se produisent (hottes etc…)", 
"Bâtiments industriels à usage de stockage", "Ateliers mécaniques sans production de vapeur d'eau", 
"Locaux sportifs sans public(sauf piscine ou patinoires)", 
"Bâtiments d’habitation, y compris les cuisines et salles d’eau, correctement chauffés et ventilés, sans suroccupation",
"Bâtiments d’habitation médiocrement ventilés et suroccupés", 
"Locaux avec forte concentration humaine ou animale (bâtiments d’élevage, manèges couverts de chevaux, certains ateliers etc…",
"Locaux à atmosphère humide contrôlée pour les besoins de la fabrication des produits (boulangeries et pâtisseries industrielles, imprimeries, tannage des cuirs etc…)",
"Locaux avec forte production de vapeur d’eau (piscines, conserveries, laiteries, ateliers de lavage de bouteilles, brasseries, cuisines collectives, blanchisseries, etc…)",
"Locaux chauffés par panneaux radiants à combustible gaz",
"Locaux spéciaux tels que locaux nécessitant le maintien d’une humidité relativement élevée, locaux sanitaires de collectivités d’utilisation très fréquente"]

# Création de l'objet combobox
listecombo = ttk.Combobox(root, values=TypeLocal, width=150)
listecombo.current(0)
listecombo.pack()
listecombo.bind("<<ComboboxSelected>>", action)

# Lancer la boucle principale de l'interface graphique
root.mainloop()

# Détermination de la température intérieure selon une température de consigne dans le cas d'un bâtiment climatisé
# Guide PACTE Page 11
df['Tint_clim'] = np.where(
    df['Tint_0'] > Tcons,
    Tcons,
    df['Tint_0']
)
if air_treatment.get() == "Climatisé":
    df['Tint'] = df['Tint_clim']
else:
    df['Tint'] = df['Tint_0']

# Calcul de la pression de vapeur saturante extérieure 'Pvs' en Pa
# Guide PACTE Page 12
df['pvs_ext'] = np.where(
    df['Text_0'] < 0,
    610.5 * np.exp((21.875 * df['Text_0']) / (df['Text_0'] + 265.5)),
    610.5 * np.exp((17.269 * df['Text_0']) / (df['Text_0'] + 237.3))
)

# Calcul de la pression de vapeur extérieure 'Pv_ext' en Pa
df['pv_ext'] = (df['HRext']/100) * df['pvs_ext']
#print(df)

# Calcul de W_ext(h)
df['W_ext(h)'] = df['pvs_ext'] * (df['HRext']/100) * 18 / (R * (df['Text_0'] + 273.15))

# Calcul de W_ext_168h
# Ajouter les 23 dernières valeurs de 'Text_0' au début du DataFrame pour l'étendre
extended_values_Wext_168 = pd.concat([df['W_ext(h)'].iloc[-167:], df['W_ext(h)']])

# Calcul de la moyenne glissante sur 168 valeurs de la colonne étendue 'Text_0'
rolling_mean_Wext_168 = extended_values_Wext_168.rolling(window=168).mean()

# Supprimer les 167 premières lignes du résultat pour obtenir la taille originale du DataFrame
df['W_ext_168h'] = rolling_mean_Wext_168.iloc[167:].values

# Calcul de Wint_0
df['Wint_0'] = df['W_ext_168h'] + df['dW']

# Calcul de l'humidité relative intérieure sans modification
# 1 - Calcul de la pression de vapeur saturante intérieure
# Guide PACTE Page 12

# 1- Calcul de pvs_Tint
df['pvs_Tint'] = np.where(
    df['Tint'] < 0,
    610.5 * np.exp(21.875 * df['Tint'] / (265.5 + df['Tint'])),
    610.5 * np.exp(17.269 * df['Tint'] / (237.3 + df['Tint']))
)

# 2- Calcul de HRint_0 - Page 12 équation 9
df['HRint_0'] = (df['Wint_0'] / 1000) * R * (df['Tint'] + 273.15) / ((18 / 1000) * df['pvs_Tint'])

# Réalisation de l'algorithme de calcul de HRint 
# Guide PACTE page 13

# Choix de l'humidité relative consigne à partir duquel la déshumidification ou climatisation entre en marche
# Calcul de HRint avec déshumidificateur
df['HRint_deshum'] = np.where(
    df['HRint_0'] > HRcons,
    HRcons,
    df['HRint_0']
)

# Si HRint non régulée (cas d'une clim)
# Calcul de la température de soufflage
df['Tsouf'] = np.where(
    df['Tint'] < df['Tint_0'],
    Tbf,
    df['Tint']
)
# Calcul de pvsat de tsouf
df['pvs_Tsouf'] = np.where(
    df['Tsouf'] < 0,
    610.5 * np.exp(21.875 * df['Tsouf'] / (265.5 + df['Tsouf'])),
    610.5 * np.exp(17.269 * df['Tsouf'] / (237.3 + df['Tsouf']))
)
# Calcul de Wsat de Tsouf
df['Wsat_Tsouf'] = df['pvs_Tsouf'] * 1 * 18 / (R * (df['Tsouf'] + 273.15))
# Calcul de Wint dans le cas d'une clim
df['Wint'] = np.where(
    df['Tint'] < df['Tint_0'],
    ((df['Wint_0'] - df['Wsat_Tsouf']) / (df['Tint_0'] - df['Tsouf'])) * (df['Tint'] - df['Tsouf']) + df['Wsat_Tsouf'],
    df['Wint_0']
)
# Calcul de HRint à partir de Wint et Tint
df['HRint_clim'] = (df['Wint'] / 1000) * R * (df['Tint'] + 273.15) / ((18 / 1000) * df['pvs_Tint'])

if air_treatment.get() == "Climatisé":
    df['HRint'] = df['HRint_clim']
if air_treatment.get() == "Déshumidifié":
    df['HRint'] = df['HRint_deshum']
if air_treatment.get() == "Non traité":
    df['HRint'] = df['HRint_0']
    
plt.figure(figsize = (14, 7))
plt.plot(np.arange(len(df)), df['HRint_clim'] * 100, label = "HRint_clim", linewidth = 1, color = 'blue', linestyle = 'solid')
plt.plot(np.arange(len(df)), df['HRint_deshum'] * 100, label = "HRint_deshum", linewidth = 1, color = 'green', linestyle = 'dotted')
plt.plot(np.arange(len(df)), df['HRint_0'] * 100, label = "HRint_0", linewidth = 1, color = 'red', linestyle = 'dashed')
plt.plot(np.arange(len(df)), df['HRint'] * 100, label = "HRint", linewidth = 1, color = 'black', linestyle = 'dashdot')

plt.title("Courbes d'évolution annuelle des humidités relatives intérieures selon le scénario choisi")
plt.xlabel('Heures (Un mois = 730 h)')
plt.ylabel('HR (%)')
plt.legend()

# Conversion de HR en %
df['HRint'] = df['HRint'] * 100

# Vérification du filtre HR
plt.figure(figsize = (14, 7))
plt.xticks(np.linspace(0, len(df)-1, num=13), np.linspace(0, 8760, num=13, dtype=int))
plt.plot(np.arange(len(df)), df['Tint_clim'], label = "Tint_clim", linewidth = 1, color = 'blue', linestyle = 'solid')
plt.plot(np.arange(len(df)), df['Tint_0'], label = "Tint_0", linewidth = 1, color = 'green', linestyle = 'dotted')
plt.plot(np.arange(len(df)), df['Tint'], label = "Tint", linewidth = 2, color = 'red', linestyle = 'dotted')
plt.title("Courbes d'évolution annuelle des humidités relatives intérieures selon le scénario choisi")
plt.xlabel('Heures (Un mois = 730 h)')
plt.ylabel('T (°C)')
plt.legend()

# Création du graphique contenant les conditions intérieures
dates = pd.date_range(start='2005-01-01', periods=8760, freq='H')

# Tracé du graphique
plt.figure(figsize=(14, 7))
plt.plot(np.arange(len(df)), df['Tint'], label='Température Intérieure en °C', color = 'r')
plt.plot(np.arange(len(df)), df['HRint'], label='Humidité relative en %', color = 'b')

# Définition l'échelle des abscisses
plt.xticks(np.linspace(0, len(df)-1, num=13), np.linspace(0, 8760, num=13, dtype=int))
plt.yticks(np.linspace(0, 100, num = 11))
plt.ylim(0, 100)
plt.xlabel("Temps en nombre d'heure depuis le début de l'année, 1 mois = 730h")
plt.ylabel("Valeurs")
plt.legend()
plt.title("Température et humidité intérieures")
plt.grid()

plt.tight_layout()

# Téléchargement de la courbe
figure_path = os.path.join(Output_folder, "Climat_Intérieur.png")

plt.savefig(figure_path)
print(f"Courbe sauvegardée avec succès dans {figure_path}.")

# Arrondi les résultats des conditions intérieures afin qu'ils aient autant de chiffres significatifs que dans un fichier .epw ce qui rend le fichier compaptible avec WUFI
df['Tint'] = round(df['Tint'], 1)
df['HRint'] = round(df['HRint'])

# Lire le fichier EPW d'entrée
df_epw, metadata = read_epw_with_pvlib(input_epw_filepath)

# Assurez-vous que df['Tint'] et df['HRint'] sont déjà calculés dans votre script principal
# Par exemple, df['Tint'] et df['HRint'] devraient exister ici

# Remplacer les colonnes 'temp_air' et 'relative_humidity' par 'Tint' et 'HRint' respectivement
df_epw['temp_air'] = df['Tint']
df_epw['relative_humidity'] = df['HRint']

# Convertir les données modifiées en CSV sans index et sans en-têtes
data_lines = df_epw.to_csv(index=False, header=False).split('\n')

# Lire le fichier EPW original ligne par ligne pour préserver le format exact
with open(input_epw_filepath, 'r') as file:
    original_lines = file.readlines()

# Localiser la ligne où commencent les données (en ignorant les lignes de métadonnées)
start_index = 0
for i, line in enumerate(original_lines):
    if line.strip() and line[0].isdigit():
        start_index = i
        break

# Fusionner les lignes de métadonnées et les lignes de données modifiées
new_epw_content = original_lines[:start_index] + [line + '\n' for line in data_lines if line.strip()]

# Écrire le nouveau contenu dans le fichier de sortie
with open(Chemin_Fichier_epw, 'w') as file:
    file.writelines(new_epw_content)

#  Création du fichier CSV contenant les conditions extérieures et intérieures
csv_path = os.path.join(Output_folder, "Conditions_T_HR.csv")

# Sélectionner les colonnes à exporter
cols_to_export = ['Text_0', 'HRext', 'Tint', 'HRint']

# Exporter les données vers un fichier CSV sans index
df[cols_to_export].to_csv(csv_path, index=False)
