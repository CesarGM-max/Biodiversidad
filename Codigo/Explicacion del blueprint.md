# Explicación del código (blueprint) — Escenario "Biodiversidad"

El "código" de esta práctica es el archivo **`Biodiversidad.blueprint.json`**, que Make.com genera al exportar el escenario (**⋯ → Export Blueprint**). Es un archivo JSON que describe cada módulo, cómo está configurado y cómo se conecta con los demás. Si se importa en Make con **Import Blueprint**, el escenario se vuelve a armar igual.

---

## 1. Estructura general del archivo

El JSON tiene tres partes principales:

- **`name`**: el nombre del escenario, `"Biodiversidad"`.
- **`flow`**: la lista de módulos, en el orden en que se ejecutan.
- **`metadata`**: la configuración general del escenario.

Cada módulo tiene los mismos campos: **`id`** (número único, se usa para leer sus datos desde otros módulos, por ejemplo `{{1.message.chat.id}}`), **`module`** (la app y acción que usa), **`parameters`** (la conexión o webhook, sin tokens), **`mapper`** (los valores configurados) y **`metadata`** (posición en el lienzo y nombres visibles para el editor de Make).

---

## 2. Módulo por módulo

### Módulo 1 — Telegram Bot: Watch Updates (disparador)

```json
"id": 1,
"module": "telegram:WatchUpdates",
"parameters": { "__IMTHOOK__": 2871689 }
```

Es el **inicio del escenario**. Cada vez que alguien le escribe al bot de Telegram ("Bot Botánico"), Telegram avisa a Make por este webhook y el escenario se ejecuta. Entrega datos como `message.chat.id` (para responder al alumno) y `message.attachment.file_id` (si el mensaje trae una foto).

### Módulo 2 — Router

```json
"id": 2,
"module": "builtin:BasicRouter"
```

Divide el escenario en **dos rutas** según si el mensaje trae foto o no. Qué ruta se sigue lo decide el **filtro** del primer módulo de cada ruta.

---

### Ruta 1: el mensaje trae foto

#### Módulo 3 — Telegram Bot: Download a File

```json
"id": 3,
"module": "telegram:DownloadFile",
"filter": {
  "name": "Mensaje con foto",
  "conditions": [[ { "a": "{{1.message.attachment.file_id}}", "o": "exist" } ]]
},
"mapper": { "fileId": "{{1.message.attachment.file_id}}" }
```

- **`filter`**: el filtro **"Mensaje con foto"**. Si `1.message.attachment.file_id` **existe**, el flujo sigue por esta ruta.
- **`mapper.fileId`**: toma el identificador de la foto que envió el alumno.
- **Resultado:** descarga la imagen y entrega `fileOutput` (los datos del archivo) y `fileName`.

#### Módulo 4 — Make AI Agent: Run an agent

```json
"id": 4,
"module": "ai-local-agent:RunLocalAIAgent",
"mapper": {
  "files": [ { "data": "{{3.fileOutput}}", "fileName": "{{3.fileName}}" } ],
  "message": "{{if(1.message.caption; 1.message.caption; \"¿Qué planta es y cómo la cuido?\")}}",
  "defaultModel": "large",
  "outputType": "text",
  "tokenLimit": "50",
  "systemPrompt": "..."
}
```

Es el **módulo central**: la IA analiza la foto de la planta.

- **`files`**: le pasa al agente la imagen descargada en el módulo 3.
- **`message`**: usa el texto que el alumno haya escrito junto con la foto (`1.message.caption`); si no escribió nada, usa la pregunta por defecto `"¿Qué planta es y cómo la cuido?"`.
- **`systemPrompt`**: las instrucciones del agente. Le indican que actúe como un experto botánico amigable: primero revisa si la imagen realmente muestra una planta (si no, responde con un mensaje de error); si sí es una planta, identifica su nombre común y científico, dice si se ve sana, qué vigilar, cada cuánto regarla y un tip útil — todo en máximo 200 palabras, en tono natural y siempre en español.
- **`defaultModel: "large"`**: usa el modelo *Large* de Make (gpt-5-mini, razonamiento bajo), pensado para tareas de razonamiento más completas que el modelo *Medium*.
- **`tokenLimit: "50"`**: limita la longitud de la respuesta al 50 % del máximo permitido.
- **Resultado:** entrega `response`, el texto que generó la IA.

#### Módulo 5 — Telegram Bot: Send a Text Message or a Reply

```json
"id": 5,
"module": "telegram:SendReplyMessage",
"mapper": {
  "chatId": "{{1.message.chat.id}}",
  "text": "{{4.response}}"
}
```

Envía la **respuesta de la IA** al alumno, en el mismo chat de donde llegó la foto.

---

### Ruta 2: el mensaje no trae foto

#### Módulo 6 — Telegram Bot: Send a Text Message or a Reply

```json
"id": 6,
"module": "telegram:SendReplyMessage",
"filter": {
  "name": "Mensaje sin foto",
  "conditions": [[
    { "a": "{{1.message.chat.id}}", "o": "exist" },
    { "a": "{{1.message.attachment.file_id}}", "o": "notexist" }
  ]]
},
"mapper": {
  "text": "🌱 ¡Hola! Envíame una foto de tu planta (puedes agregar una pregunta en el texto de la foto) y te ayudo a identificarla y a saber cómo cuidarla."
}
```

- **`filter`**: el filtro **"Mensaje sin foto"**. Si hay un chat (`message.chat.id` existe) pero **no** hay foto (`message.attachment.file_id` no existe), el flujo sigue por aquí.
- **`mapper.text`**: es un mensaje fijo que le pide al alumno una foto. En esta ruta no se usa la IA, lo que ahorra operaciones.

---

## 3. Configuración general del escenario (`metadata`)

```json
"metadata": {
  "instant": true,
  "scenario": { "roundtrips": 1, "maxErrors": 3, "sequential": false, ... },
  "zone": "us2.make.com"
}
```

- **`instant: true`**: el escenario es **instantáneo**, se ejecuta en cuanto llega el aviso del webhook de Telegram.
- **`maxErrors: 3`**: si falla 3 veces seguidas, Make desactiva el escenario.
- **`sequential: false`**: puede procesar varios mensajes al mismo tiempo.
- **`zone: "us2.make.com"`**: el servidor de Make donde está la cuenta.

---

## 4. Resumen del funcionamiento

1. El alumno le manda una foto de una planta al bot ("Bot Botánico") → **módulo 1** la recibe por webhook.
2. El **Router (2)** revisa si el mensaje trae foto.
3. **Con foto:** el **módulo 3** la descarga → el **módulo 4** (IA botánica) la analiza e identifica la planta → el **módulo 5** envía la respuesta al alumno.
4. **Sin foto:** el **módulo 6** le pide al alumno que mande una foto.
