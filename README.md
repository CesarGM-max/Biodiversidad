# Biodiversidad: identificación de plantas con Make.com, Telegram e Inteligencia Artificial

- **Alumno:** Cesar Jesus Gamez Mendoza
- **Materia:** Desarrollo Sustentable
- **Profesor:** Mtro. Miguel Ángel Barrón Hernández
- **Institución:** Instituto Tecnológico de Mazatlán

## Descripción

Automatización hecha en Make.com para identificar plantas y ayudar a cuidarlas, como apoyo al reconocimiento de la biodiversidad vegetal. El alumno le manda una foto de una planta (opcionalmente con una pregunta) a un bot de Telegram llamado **"Bot Botánico"**. Make recibe el mensaje, descarga la imagen y se la pasa a un Agente de IA que actúa como botánico experto. El agente responde en segundos con:

- el nombre común y científico probable de la planta,
- si se ve sana o no,
- cada cuánto regarla,
- y un tip de cuidado.

Si la foto no muestra una planta, el bot avisa que no puede identificarla. Si el alumno manda solo texto, el bot le pide que envíe una foto.

## Objetivos de aprendizaje

**Objetivo general:** diseñar y documentar una automatización en Make.com que, con un bot de Telegram y un Agente de IA con visión, identifique plantas a partir de una fotografía y brinde recomendaciones básicas de cuidado, como herramienta de apoyo al reconocimiento de la biodiversidad.

**Objetivos específicos:**

- Configurar un disparador (trigger) de Telegram que reciba en tiempo real los mensajes enviados al bot.
- Enrutar el flujo según el contenido recibido (con foto o solo texto) con un módulo Router y dos filtros.
- Descargar la imagen de Telegram y dársela como entrada al Make AI Agent.
- Redactar un *system prompt* que limite la respuesta a un formato breve, natural y en español, y que filtre imágenes que no sean plantas.
- Responder automáticamente al alumno con el resultado del análisis y recomendaciones de cuidado.
- Reforzar la identificación de especies vegetales como parte del estudio de la biodiversidad.

## Material utilizado

Esta práctica no usa Arduino ni componentes electrónicos. Se utilizó:

- Laptop (para construir el escenario en Make.com)
- Teléfono celular con Telegram (para tomar y enviar las fotos)
- Cuenta de **Make.com** (zona us2.make.com) con el módulo **Make AI Agent**
- Bot de **Telegram** creado con @BotFather ("Bot Botánico")
- Modelo de IA: *Large* de Make (gpt-5-mini, razonamiento bajo)

## Escenario en Make

En lugar de un circuito, se documenta el **escenario de Make** (6 módulos y un Router con 2 rutas).

![Escenario en Make](Imagenes/Escenario%20Biodiversidad.png)

**Filtros del Router:**

- **Mensaje con foto** (entre el Router 2 y el módulo 3): `{{1.message.attachment.file_id}}` — *exist*
- **Mensaje sin foto** (entre el Router 2 y el módulo 6): `{{1.message.chat.id}}` *exist* y `{{1.message.attachment.file_id}}` — *not exist*

## Código

El "código" de esta práctica es el escenario de Make exportado como blueprint:

- [`Codigo/Biodiversidad.blueprint.json`](Codigo/Biodiversidad.blueprint.json): se puede importar en Make con **Import Blueprint**.
- [`Codigo/Explicacion del blueprint.md`](Codigo/Explicacion%20del%20blueprint.md): explicación en texto del blueprint, módulo por módulo.

## Video

[Ver video en YouTube](https://youtube.com/shorts/1LXUDgtPWpU?feature=share)

Link también disponible en [`Video/enlace.txt`](Video/enlace.txt)

## Resultados

[`Resultados/Resultados.pdf`](Resultados/Resultados.pdf)

**Observaciones sobre el comportamiento del sistema:**

- El escenario es instantáneo: Telegram avisa a Make por webhook en cuanto llega la foto, y la respuesta tarda solo unos segundos.
- El Router usa el campo `message.attachment.file_id` para decidir la ruta, así el Agente de IA solo se ejecuta cuando hay una imagen que analizar, lo que ahorra operaciones de Make.
- El *system prompt* hace que la IA primero valide si la imagen es realmente una planta antes de responder, evitando identificaciones erróneas sobre personas, animales u objetos.
- La respuesta siempre sigue el mismo formato breve (nombre, estado de salud, riego y un tip), pensado para leerse rápido en el chat, como si fuera un amigo experto respondiendo.

## Conclusiones

La práctica permitió aplicar la automatización al reconocimiento de la biodiversidad vegetal. Con un bot de Telegram, un Router con filtros y un Agente de IA en Make.com se construyó una herramienta que identifica plantas a partir de una foto y ofrece recomendaciones básicas de cuidado. El diseño del *system prompt* fue clave: además de identificar la especie, el agente valida que la imagen sea realmente una planta antes de responder, lo que evita respuestas fuera de lugar. Este tipo de herramienta acerca a los estudiantes al reconocimiento práctico de la biodiversidad vegetal de su entorno, usando únicamente el teléfono y una app de mensajería.
