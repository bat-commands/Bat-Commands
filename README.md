<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Archive Ultimate Code Uploads</title>
  <style>
    body { font-family: Arial, sans-serif; margin: 20px; }
    #editor { width: 100%; height: 300px; border: 1px solid #ccc; padding: 10px; white-space: pre-wrap; }
    button, input, select { margin: 5px; padding: 8px; }
    #status { margin-top: 10px; font-weight: bold; color: darkblue; }
    #options { margin-top: 15px; }
  </style>
</head>
<body>

<h1>Gestor Archive (TXT Versions)</h1>
<div id="editor" contenteditable="true"></div>
<br>

<label for="versionName">Nombre base:</label>
<input type="text" id="versionName" placeholder="Ej: Nota, Documento">
<br>
<select id="versionSelector"></select>
<br>
<button onclick="guardar()">Guardar (Bloque Nombre+Fecha)</button>
<button onclick="abrir()">Abrir</button>
<button onclick="eliminar()">Eliminar</button>
<br>

<!-- Nuevo: Upload de archivo externo -->
<input type="file" id="fileUpload" accept=".txt">
<button onclick="subir()">Upload TXT</button>

<div id="status"></div>
<div id="options"></div>

<script>
  window.onload = actualizarSelector;

  function guardar() {
    const contenido = document.getElementById("editor").innerText;
    let nombreBase = document.getElementById("versionName").value.trim();
    if (!nombreBase) nombreBase = "Archivo";

    const fecha = new Date();
    const sello = fecha.getFullYear() + "-" +
                  String(fecha.getMonth()+1).padStart(2,"0") + "-" +
                  String(fecha.getDate()).padStart(2,"0") + "_" +
                  String(fecha.getHours()).padStart(2,"0") + "-" +
                  String(fecha.getMinutes()).padStart(2,"0");

    const nombreFinal = nombreBase + "_" + sello;
    const clave = "txt_" + nombreFinal;

    const existe = localStorage.getItem(clave);
    const estado = existe ? "Existing" : "New";

    const paquete = {
      arcVersion: nombreFinal,
      content: contenido,
      block: sello,
      state: estado
    };

    localStorage.setItem(clave, JSON.stringify(paquete));

    document.getElementById("status").textContent =
      "📝 Estado: " + estado + " → " + nombreFinal;

    generarBoton(nombreFinal);
    actualizarSelector();
  }

  function abrir() {
    const nombre = document.getElementById("versionSelector").value;
    const paquete = JSON.parse(localStorage.getItem("txt_" + nombre));
    if (paquete) {
      const contenidoConBloque =
        "Arc.Version: " + paquete.arcVersion + "\n" +
        "Block: " + paquete.block + "\n" +
        "State: " + paquete.state + "\n\n" +
        paquete.content;

      document.getElementById("editor").innerText = contenidoConBloque;
      document.getElementById("status").textContent =
        "📂 Estado: " + paquete.state + " → " + paquete.arcVersion;
      generarBoton(paquete.arcVersion);
    }
  }

  function descargar(nombre) {
    const paquete = JSON.parse(localStorage.getItem("txt_" + nombre));
    if (paquete) {
      const contenidoTxt = "Arc.Version: " + paquete.arcVersion + "\n" +
                           "Block: " + paquete.block + "\n" +
                           "State: " + paquete.state + "\n\n" +
                           paquete.content;
      const blob = new Blob([contenidoTxt], { type: "text/plain" });
      const enlace = document.createElement("a");
      enlace.href = URL.createObjectURL(blob);
      enlace.download = "Archive_" + nombre + ".txt";
      enlace.click();
      document.getElementById("status").textContent =
        "⬇️ Estado: Downloaded → " + paquete.arcVersion;
    }
  }

  function eliminar() {
    const nombre = document.getElementById("versionSelector").value;
    localStorage.removeItem("txt_" + nombre);
    actualizarSelector();
    document.getElementById("status").textContent =
      "🗑️ Estado: Deleted → " + nombre;
    document.getElementById("options").innerHTML = "";
  }

  function actualizarSelector() {
    const selector = document.getElementById("versionSelector");
    selector.innerHTML = "";
    const keys = [];
    for (let i = 0; i < localStorage.length; i++) {
      const key = localStorage.key(i);
      if (key.startsWith("txt_")) keys.push(key.replace("txt_", ""));
    }
    keys.sort().reverse();
    keys.forEach(nombre => {
      const option = document.createElement("option");
      option.value = nombre;
      option.textContent = nombre;
      selector.appendChild(option);
    });
  }

  function generarBoton(nombre) {
    const optionsDiv = document.getElementById("options");
    optionsDiv.innerHTML = "";
    const btn = document.createElement("button");
    btn.textContent = "Download versión: " + nombre;
    btn.onclick = () => descargar(nombre);
    optionsDiv.appendChild(btn);
  }

  // Nuevo: función para subir archivo externo
  function subir() {
    const fileInput = document.getElementById("fileUpload");
    const file = fileInput.files[0];
    if (!file) {
      alert("⚠️ Selecciona un archivo .txt para subir.");
      return;
    }

    const reader = new FileReader();
    reader.onload = function(e) {
      const contenido = e.target.result;

      const fecha = new Date();
      const sello = fecha.getFullYear() + "-" +
                    String(fecha.getMonth()+1).padStart(2,"0") + "-" +
                    String(fecha.getDate()).padStart(2,"0") + "_" +
                    String(fecha.getHours()).padStart(2,"0") + "-" +
                    String(fecha.getMinutes()).padStart(2,"0");

      const nombreFinal = file.name.replace(".txt","") + "_" + sello;
      const clave = "txt_" + nombreFinal;

      const paquete = {
        arcVersion: nombreFinal,
        content: contenido,
        block: sello,
        state: "Uploaded"
      };

      localStorage.setItem(clave, JSON.stringify(paquete));

      document.getElementById("status").textContent =
        "📤 Estado: Uploaded → " + nombreFinal;

      generarBoton(nombreFinal);
      actualizarSelector();
    };

    reader.readAsText(file);
  }
</script>

</body>
</html>
