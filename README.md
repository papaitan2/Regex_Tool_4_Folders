# Regex_Tool_4_Folders
Regex simplifica la búsqueda de patrones complejos en tus archivos.



---
¡Presentamos **RegexTool**: La herramienta definitiva para búsquedas avanzadas en archivos!
Desarrollada con PyQt5 en el entorno Qt, Regex simplifica la búsqueda de patrones complejos en tus archivos.

Funcionalidades:

Búsqueda con expresiones regulares: Encuentra patrones específicos en archivos de texto utilizando expresiones regulares.
Opciones de búsqueda avanzadas: Personaliza tu búsqueda con opciones como la sensibilidad a mayúsculas y minúsculas, búsqueda multilínea, y más.
Visualización de resultados: Muestra los resultados de la búsqueda en una interfaz gráfica intuitiva, con la opción de guardarlos en un archivo Markdown.
Generación de enlaces: Crea enlaces directos a las líneas donde se encuentran las coincidencias para una navegación rápida.
Resaltado de coincidencias: Resalta las coincidencias en el contexto para una fácil identificación.
Copiado de archivos: Copia los archivos que contienen coincidencias a una subcarpeta para un análisis posterior.

Características:

Interfaz gráfica de usuario (GUI) intuitiva: Facilita la interacción con el programa, incluso para usuarios sin experiencia en programación.
Opciones de personalización: Adapta la búsqueda a tus necesidades con opciones de configuración flexibles.
Generación de archivos Markdown: Crea informes de búsqueda con formato profesional para compartir o documentar tus hallazgos.
Beneficios:

Ahorra tiempo: Encuentra rápidamente la información que necesitas sin tener que revisar manualmente cada archivo.
Mejora la precisión: Evita errores humanos al buscar patrones complejos.
Facilita el análisis: Organiza y visualiza los resultados de la búsqueda para un análisis más eficiente.
¡Simplifica tus búsquedas y aumenta tu productividad con Regex!



---
<br>
<br>
<br>
<br>


![Pasted image 20241016074959](https://github.com/user-attachments/assets/afc58eab-8659-483a-9bb6-768d95f0a728)



---
<br>
<br>
<br>
<br>

![Pasted image 20241016074346](https://github.com/user-attachments/assets/6ad9de5a-4574-44a8-8973-00e4b7d50ff7)



---
<br>
<br>
<br>
<br>

...

![regex-1](https://github.com/user-attachments/assets/dc92d608-92f1-4307-b16a-b94d7f15728a)

...

---
<br>
<br>
<br>
<br>


```python

import os
import re
import shutil
import subprocess
from datetime import datetime

from PyQt5.QtWidgets import (
    QApplication,
    QWidget,
    QVBoxLayout,
    QLabel,
    QLineEdit,
    QPushButton,
    QTextEdit,
    QFileDialog,
    QRadioButton,
    QHBoxLayout,
)
from PyQt5.QtCore import Qt


def guardar_resultados_busqueda(resultados, ruta_carpeta, extension):
    """
    Escribe los resultados de la búsqueda en un archivo Markdown en la carpeta especificada.
    """
    try:
        fecha_hora = datetime.now().strftime("%d%m_%H%M")
        nombre_archivo = os.path.join(ruta_carpeta, f"{fecha_hora}-resultados_busqueda.md")
        with open(nombre_archivo, "w", encoding="utf-8") as f:
            for ruta_archivo, resultados_archivo in resultados.items():
                f.write(f"# {ruta_archivo}\n")
                for num_linea, contexto, coincidencias in resultados_archivo:
                    f.write(f"## Línea {num_linea}:\n")
                    contexto = contexto.replace("<", "$<").replace(">", ">$")

                    # Formatear las coincidencias en el contexto
                    for coincidencia in coincidencias:
                        contexto = contexto.replace(coincidencia, f"**{coincidencia}**")

                    # Imprimir la línea completa como título 3
                    f.write(f"### {contexto}\n")  # Imprimir la línea completa
                    f.write("_________________________\n")

                    # Imprimir la línea completa como título 3 con enlace al archivo.
                    enlace_archivo = f"file:///{ruta_archivo}"  # Crear enlace al archivo
                    f.write(f"### [{ruta_archivo}]({enlace_archivo})\n")  # Imprimir línea con enlace
                    f.write("_________________________\n")


                    # Dividir el contexto en líneas.
                    lineas_contexto = contexto.splitlines()


                    # Formato de bloque de código con lenguaje especificado___
                    if extension == "lsp":
                        extTxtParaBloque = "lisp"
                    elif extension == "md":
                        extTxtParaBloque = "markdown"
                    elif extension == "py":
                        extTxtParaBloque = "python"
                    else:
                        extTxtParaBloque = extension

                    # Primer bloque de código (3 primeras líneas)
                    f.write("```" + extTxtParaBloque + "\n")
                    lc = int(len(lineas_contexto)/2)
                    if len(lineas_contexto) >= lc :
                        f.write("\n".join(lineas_contexto[:lc]) + "\n")
                    else:
                        f.write("no hay\n")
                    f.write("```\n")

                    # Salto de línea
                    f.write("\n")

                    # Segundo bloque de código (3 últimas líneas)
                    f.write("```" + extTxtParaBloque + "\n")
                    if len(lineas_contexto) >= lc:
                        f.write("\n".join(lineas_contexto[-lc:]) + "\n")
                    else:
                        f.write("no hay\n")
                    f.write("```\n")

    except Exception as e:
        fecha_hora = datetime.now().strftime("%d%m_%H%M")
        nombre_archivo_log = os.path.join(ruta_carpeta, f"{fecha_hora}-error_log.txt")
        with open(nombre_archivo_log, "a") as log_file:
            log_file.write(f"Error al escribir los resultados en el archivo Markdown: {e}\n")





def copiar_archivos_positivos(archivos_positivos, ruta_carpeta):
    """
    Copia los archivos que dan positivo a una subcarpeta en la carpeta especificada.
    """
    if archivos_positivos:
        fecha_hora = datetime.now().strftime("%d%m_%H%M")
        carpeta_destino = os.path.join(ruta_carpeta, f"{fecha_hora}-resultados_positivos")
        os.makedirs(carpeta_destino, exist_ok=True)
        for archivo in archivos_positivos:
            try:
                shutil.copy2(archivo, carpeta_destino)
            except Exception as e:
                fecha_hora = datetime.now().strftime("%d%m_%H%M")
                nombre_archivo_log = os.path.join(ruta_carpeta, f"{fecha_hora}-error_log.txt")
                with open(nombre_archivo_log, "a") as log_file:
                    log_file.write(f"Error al copiar el archivo {archivo}: {e}\n")







class MainWindow(QWidget):
    def __init__(self):
        super().__init__()

        self.setWindowTitle("Búsqueda Regex")
        self.layout = QVBoxLayout()

        # Ruta de la carpeta
        self.ruta_label = QLabel("Ruta de la carpeta:")
        self.ruta_input = QLineEdit()
        # self.ruta_input.setText("C:\\Users\\ejemplo\\ruta")  # Valor predeterminado ----------------------
        self.ruta_input.setText("E:\\test")  # Valor predeterminado
        self.layout.addWidget(self.ruta_label)
        self.layout.addWidget(self.ruta_input)

        # Tipo de condición
        self.tipo_condicion_label = QLabel("Tipo de condición:")
        self.tipo_condicion_layout = QHBoxLayout()
        self.radio_and = QRadioButton("and")
        self.radio_or = QRadioButton("or")
        self.radio_and.setChecked(True)  # Valor predeterminado
        self.tipo_condicion_layout.addWidget(self.radio_and)
        self.tipo_condicion_layout.addWidget(self.radio_or)
        self.layout.addWidget(self.tipo_condicion_label)
        self.layout.addWidget(QLabel())  # Espacio en blanco
        self.layout.addLayout(self.tipo_condicion_layout)

        # Número de condiciones
        self.num_condiciones_label = QLabel("Número de condiciones:")
        self.num_condiciones_input = QLineEdit()
        self.num_condiciones_input.setText("2")  # Valor predeterminado
        self.layout.addWidget(self.num_condiciones_label)
        self.layout.addWidget(self.num_condiciones_input)

        # Líneas de contexto
        self.lineas_contexto_label = QLabel("Número de líneas de contexto:")
        self.lineas_contexto_input = QLineEdit()
        self.lineas_contexto_input.setText("3")  # Valor predeterminado
        self.layout.addWidget(self.lineas_contexto_label)
        self.layout.addWidget(self.lineas_contexto_input)

        # Extensión de los archivos
        self.extension_label = QLabel("Extensión de los archivos (o 'todos'):")
        self.extension_input = QLineEdit()
        # self.extension_input.setText("todos")  # Valor predeterminado -----------------------
        self.extension_input.setText("lsp")  # Valor predeterminado
        self.layout.addWidget(self.extension_label)
        self.layout.addWidget(self.extension_input)

        # Patrones regex
        self.patrones_label = QLabel("Patrones regex (separados por comas):")
        self.patrones_input = QLineEdit()
        # self.patrones_input.setText("patron1, patron2")  # Valor predeterminado
        self.patrones_input.setText("def,EE")  # Valor predeterminado
        self.layout.addWidget(self.patrones_label)
        self.layout.addWidget(self.patrones_input)

        # Opciones adicionales
        self.opciones_layout = QVBoxLayout()
        self.regex_multilinea_label = QLabel("Regex multilínea:")
        self.radio_multilinea_si = QRadioButton("Sí")
        self.radio_multilinea_no = QRadioButton("No")
        self.radio_multilinea_no.setChecked(True)  # Valor predeterminado
        self.regex_multilinea_layout = QHBoxLayout()
        self.regex_multilinea_layout.addWidget(self.radio_multilinea_si)
        self.regex_multilinea_layout.addWidget(self.radio_multilinea_no)
        self.opciones_layout.addWidget(self.regex_multilinea_label)
        self.opciones_layout.addLayout(self.regex_multilinea_layout)

        self.regex_case_sensitive_label = QLabel("Regex sensible a mayúsculas:")
        self.radio_case_sensitive_si = QRadioButton("Sí")
        self.radio_case_sensitive_no = QRadioButton("No")
        self.radio_case_sensitive_no.setChecked(True)  # Valor predeterminado
        self.regex_case_sensitive_layout = QHBoxLayout()
        self.regex_case_sensitive_layout.addWidget(self.radio_case_sensitive_si)
        self.regex_case_sensitive_layout.addWidget(self.radio_case_sensitive_no)
        self.opciones_layout.addWidget(self.regex_case_sensitive_label)
        self.opciones_layout.addLayout(self.regex_case_sensitive_layout)

        self.copiar_archivos_label = QLabel("Copiar archivos que dan positivo:")
        self.radio_copiar_si = QRadioButton("Sí")
        self.radio_copiar_no = QRadioButton("No")
        self.radio_copiar_si.setChecked(True)  # Valor predeterminado
        self.copiar_archivos_layout = QHBoxLayout()
        self.copiar_archivos_layout.addWidget(self.radio_copiar_si)
        self.copiar_archivos_layout.addWidget(self.radio_copiar_no)
        self.opciones_layout.addWidget(self.copiar_archivos_label)
        self.opciones_layout.addLayout(self.copiar_archivos_layout)

        self.buscar_nombres_archivos_label = QLabel("Buscar en nombres de archivos:")
        self.radio_buscar_nombres_si = QRadioButton("Sí")
        self.radio_buscar_nombres_no = QRadioButton("No")
        self.radio_buscar_nombres_no.setChecked(True)  # Valor predeterminado
        self.buscar_nombres_archivos_layout = QHBoxLayout()
        self.buscar_nombres_archivos_layout.addWidget(self.radio_buscar_nombres_si)
        self.buscar_nombres_archivos_layout.addWidget(self.radio_buscar_nombres_no)
        self.opciones_layout.addWidget(self.buscar_nombres_archivos_label)
        self.opciones_layout.addLayout(self.buscar_nombres_archivos_layout)

        self.layout.addLayout(self.opciones_layout)

        # Botones
        self.boton_buscar = QPushButton("Buscar")
        self.boton_buscar.clicked.connect(self.realizar_busqueda)
        self.layout.addWidget(self.boton_buscar)

        self.boton_guardar = QPushButton("Guardar Resultados")
        self.boton_guardar.clicked.connect(self.guardar_resultados)
        self.layout.addWidget(self.boton_guardar)

        # Salida
        self.salida = QTextEdit()
        self.salida.setReadOnly(True)
        self.layout.addWidget(self.salida)

        self.setLayout(self.layout)








    def realizar_busqueda(self):

        ruta_carpeta = self.ruta_input.text()
        tipo_condicion = "and" if self.radio_and.isChecked() else "or"
        try:
            num_condiciones = int(self.num_condiciones_input.text())
        except ValueError:
            self.salida.append("El número de condiciones debe ser un número entero.")
            return
        try:
            lineas_contexto = int(self.lineas_contexto_input.text())
        except ValueError:
            self.salida.append("El número de líneas de contexto debe ser un número entero.")
            return
        extension = self.extension_input.text().lower()
        patrones_str = self.patrones_input.text()

        flags = 0
        if self.radio_multilinea_si.isChecked():
            flags |= re.MULTILINE
        if self.radio_case_sensitive_no.isChecked():
            flags |= re.IGNORECASE


        try:
            patrones = [re.compile(p.strip(), flags) for p in patrones_str.split(",")]

            resultados = {}
            archivos_positivos = []
            for ruta_raiz, _, archivos in os.walk(ruta_carpeta):
                for nombre_archivo in archivos:
                    if extension == "todos" or nombre_archivo.lower().endswith(f".{extension}"):
                        ruta_archivo = os.path.join(ruta_raiz, nombre_archivo)
                        try:
                            with open(ruta_archivo, "r", encoding="utf-8") as f:
                                lineas = f.readlines()
                                for i, linea in enumerate(lineas):
                                    coincidencias = []
                                    for patron in patrones:
                                        coincidencias.extend(patron.findall(linea))

                                    if (tipo_condicion == "and" and all(p.search(linea) for p in patrones)) or \
                                    (tipo_condicion == "or" and any(p.search(linea) for p in patrones)):
                                        
                                        inicio = max(0, i - lineas_contexto)
                                        fin = min(len(lineas), i + lineas_contexto + 1)
                                        contexto = "".join(lineas[inicio:fin])

                                        # Guardar cada coincidencia como una tupla separada
                                        for coincidencia in coincidencias:
                                            resultados.setdefault(ruta_archivo, []).append(
                                                (i + 1, contexto, [coincidencia])  # Guardar la coincidencia individual
                                            )

                                        if ruta_archivo not in archivos_positivos:
                                            archivos_positivos.append(ruta_archivo)

                        except UnicodeDecodeError:
                            mensaje = f"Error al leer el archivo {ruta_archivo}: No se puede decodificar con UTF-8."
                            print(mensaje)
                            self.salida.append(mensaje)
                            with open("error_log.txt", "a") as log_file:
                                log_file.write(f"{mensaje}\n")

            
            # Mostrar resultados en la interfaz
            self.salida.clear()
            if resultados:  # Si hay resultados, mostrarlos
                try:
                    for nombre_archivo, resultados_archivo in resultados.items():
                        self.salida.append(f"# {nombre_archivo}\n")
                        for num_linea, contexto, coincidencias in resultados_archivo:  # Desempaquetar tres valores
                            # print(resultados_archivo)   # ----------------------------------
                            self.salida.append(f"## Línea {num_linea}:\n")
                            self.salida.append(contexto + "\n")
                            # Aquí puedes mostrar las coincidencias si lo deseas
                except ValueError as e:
                    mensaje = f"Error al mostrar los resultados: {e}"
                    print(mensaje)
                    self.salida.append(mensaje)
                    with open("error_log.txt", "a") as log_file:
                        log_file.write(f"{mensaje}\n")
            else:
                self.salida.append("No se encontraron coincidencias.")

            # Copiar los archivos que dan positivo a una subcarpeta (si se seleccionó la opción)
            if self.radio_copiar_si.isChecked():
                copiar_archivos_positivos(archivos_positivos, ruta_carpeta)

        except Exception as e:
            mensaje = f"Error en la búsqueda: {e}"
            print(mensaje)
            self.salida.append(mensaje)
            with open("error_log.txt", "a") as log_file:
                log_file.write(f"{mensaje}\n")


















    def guardar_resultados(self):
        """
        Guarda los resultados de la búsqueda en un archivo Markdown y copia los archivos
        que dan positivo a una subcarpeta (si se seleccionó la opción).
        """
        try:
            ruta_carpeta = self.ruta_input.text()

            tipo_condicion = "and" if self.radio_and.isChecked() else "or"
            try:
                num_condiciones = int(self.num_condiciones_input.text())
            except ValueError:
                self.salida.append("El número de condiciones debe ser un número entero.")
                return
            try:
                lineas_contexto = int(self.lineas_contexto_input.text())
            except ValueError:
                self.salida.append("El número de líneas de contexto debe ser un número entero.")
                return
            extension = self.extension_input.text().lower()
            patrones_str = self.patrones_input.text()

            flags = 0
            if self.radio_multilinea_si.isChecked():
                flags |= re.MULTILINE
            if self.radio_case_sensitive_no.isChecked():
                flags |= re.IGNORECASE

            patrones = [re.compile(p.strip(), flags) for p in patrones_str.split(",")]

            resultados = {}
            archivos_positivos = []
            for ruta_raiz, _, archivos in os.walk(ruta_carpeta):
                for nombre_archivo in archivos:
                    if extension == "todos" or nombre_archivo.lower().endswith(f".{extension}"):
                        ruta_archivo = os.path.join(ruta_raiz, nombre_archivo)
                        try:
                            with open(ruta_archivo, "r", encoding="utf-8") as f:
                                lineas = f.readlines()
                                for i, linea in enumerate(lineas):
                                    coincidencias = []
                                    for patron in patrones:
                                        coincidencias.extend(patron.findall(linea))
                                    if (tipo_condicion == "and" and all(p.search(linea) for p in patrones)) or \
                                    (tipo_condicion == "or" and any(p.search(linea) for p in patrones)):
                                        inicio = max(0, i - lineas_contexto)
                                        fin = min(len(lineas), i + lineas_contexto + 1)
                                        contexto = "".join(lineas[inicio:fin])
                                        
                                        # Guardar cada coincidencia como una tupla separada -+++++++++++
                                        for coincidencia in coincidencias:
                                            resultados.setdefault(ruta_archivo, []).append(
                                                (i + 1, contexto, [coincidencia])  # Guardar la coincidencia individual
                                            )

                                        if ruta_archivo not in archivos_positivos:
                                            archivos_positivos.append(ruta_archivo)
                        except UnicodeDecodeError:
                            mensaje = f"Error al leer el archivo {ruta_archivo}: No se puede decodificar con UTF-8."
                            print(mensaje)
                            self.salida.append(mensaje)
                            fecha_hora = datetime.now().strftime("%d%m_%H%M")
                            nombre_archivo_log = os.path.join(ruta_carpeta, f"{fecha_hora}-error_log.txt")
                            with open(nombre_archivo_log, "a") as log_file:
                                log_file.write(f"{mensaje}\n")

                # Guardar resultados en archivo Markdown
                guardar_resultados_busqueda(resultados, ruta_carpeta, extension)





            # Guardar la lista de archivos positivos en la carpeta ruta_carpeta.
            ruta_archivo_positivos = os.path.join(ruta_carpeta, "lista_resultados_positivos.txt")  # Incluir ruta completa
            with open(ruta_archivo_positivos, "w", encoding="utf-8") as f:
                for archivo in archivos_positivos:
                    f.write(archivo + "\n")


            # Copiar archivos a subcarpeta (si se seleccionó la opción)
            if self.radio_copiar_si.isChecked():
                copiar_archivos_positivos(archivos_positivos, ruta_carpeta)


        except Exception as e:
            fecha_hora = datetime.now().strftime("%d%m_%H%M")
            nombre_archivo_log = os.path.join(ruta_carpeta, f"{fecha_hora}-error_log.txt")
            with open(nombre_archivo_log, "a") as log_file:
                log_file.write(f"Error al guardar los resultados: {e}\n")




if __name__ == "__main__":
    app = QApplication([])
    window = MainWindow()
    window.show()
    app.exec_()












```






...





