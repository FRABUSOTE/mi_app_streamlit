import streamlit as st
import pdfplumber
import pandas as pd
import re
from io import BytesIO

st.title("📄 Extractor de Datos desde Facturas PDF")

uploaded_files = st.file_uploader(
    "Sube uno o varios PDFs",
    type=["pdf"],
    accept_multiple_files=True
)

def extraer_datos(pdf_bytes, nombre_archivo):
    with pdfplumber.open(pdf_bytes) as pdf:
        texto = ""
        for pagina in pdf.pages:
            texto += pagina.extract_text() + "\n"

    nombre = re.search(r'Nombre/Razón Social:\s*(.+)', texto)
    direccion = re.search(r'Dirección:\s*(.+)', texto)
    orden = re.search(r'Orden de Compra:\s*(\w+)', texto)
    fecha = re.search(r'(\d{2}/\d{2}/\d{4})', texto)
    correo = re.search(r'[\w\.-]+@[\w\.-]+', texto)
    total_num = re.search(r'(?<!\d)(\d{1,3}(?:,\d{3})*\.\d{2})(?!\d)', texto)
    total_letras = re.search(r'DOS MIL.*?SOLES', texto, re.IGNORECASE)

    return {
        "Archivo": nombre_archivo,
        "Nombre/Razón Social": nombre.group(1) if nombre else "",
        "Dirección": direccion.group(1) if direccion else "",
        "Orden de Compra": orden.group(1) if orden else "",
        "Fecha Emisión": fecha.group(1) if fecha else "",
        "Correo": correo.group(0) if correo else "",
        "Total (número)": total_num.group(1) if total_num else "",
        "Total (letras)": total_letras.group(0) if total_letras else ""
    }

if uploaded_files:
    resultados = []

    for archivo in uploaded_files:
        resultados.append(extraer_datos(archivo, archivo.name))

    df = pd.DataFrame(resultados)

    st.success("PDFs procesados correctamente ✔")

    st.dataframe(df)

    # Descargar Excel
    output = BytesIO()
    df.to_excel(output, index=False, sheet_name="Resultados")
    output.seek(0)

    st.download_button(
        "⬇ Descargar Excel",
        data=output,
        file_name="resumen_facturas.xlsx",
        mime="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"
    )
