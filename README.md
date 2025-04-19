# Scraping
scraping basico

    
    import requests
    from bs4 import BeautifulSoup
    
    # URL de la página que quieres scrapear
    url = "https://example.com"
    
    # Hacer la solicitud HTTP
    response = requests.get(url)
    if response.status_code == 200:  # Verificar que la página responde correctamente
        soup = BeautifulSoup(response.text, "html.parser")
    
        # Extraer títulos de la página (por ejemplo, etiquetas <h1>)
        titulos = soup.find_all("h1")
        
        for titulo in titulos:
            print(titulo.text)  # Mostrar los títulos en la página
    
    else:
        print(f"Error: No se pudo acceder a la página ({response.status_code})")



## Scraping algo más dificil: sin apis, que descargue pdf y pase a la sgte pagina.

    import requests
    from bs4 import BeautifulSoup
    import os
    
    # URL inicial de la página web
    base_url = "https://example.com/pagina"  # Reemplázala con la URL real
    
    # Carpeta donde se guardarán los PDFs
    os.makedirs("pdfs_descargados", exist_ok=True)
    
    def descargar_pdf(url_pdf):
        """ Descarga un PDF y lo guarda en la carpeta local. """
        pdf_nombre = url_pdf.split("/")[-1]  # Obtiene el nombre del archivo
        ruta_pdf = os.path.join("pdfs_descargados", pdf_nombre)
    
        response = requests.get(url_pdf)
        if response.status_code == 200:
            with open(ruta_pdf, "wb") as f:
                f.write(response.content)
            print(f"✅ PDF descargado: {pdf_nombre}")
    
    def extraer_pdfs_y_siguiente(url_actual):
        """ Extrae enlaces de PDF y encuentra el botón 'Siguiente' para avanzar. """
        response = requests.get(url_actual)
        if response.status_code != 200:
            print(f"🚨 Error al acceder a la página: {url_actual}")
            return None
    
        soup = BeautifulSoup(response.text, "html.parser")
    
        # Extraer enlaces a archivos PDF
        enlaces_pdf = [a["href"] for a in soup.find_all("a") if a["href"].endswith(".pdf")]
        if not enlaces_pdf:
            print("⚠ No se encontraron PDFs en esta página.")
    
        for enlace in enlaces_pdf:
            if enlace.startswith("http"):
                descargar_pdf(enlace)  # Si el enlace es completo, lo descarga
            else:
                descargar_pdf(base_url + enlace)  # Si es relativo, lo completa con la base URL
    
        # Buscar el botón "Siguiente" para avanzar a la próxima página
        siguiente_pagina = soup.find("a", text="Siguiente")  # Ajusta según el nombre del botón
        if siguiente_pagina:
            nueva_url = siguiente_pagina["href"]
            if nueva_url.startswith("http"):
                extraer_pdfs_y_siguiente(nueva_url)
            else:
                extraer_pdfs_y_siguiente(base_url + nueva_url)
    
    # Ejecutar el proceso
    extraer_pdfs_y_siguiente(base_url)
