# Mejoras simplementadas

## 2026-10-09 — Optimizar aplicación para Acer AN515-57

- Descargar los archivos de los modelos manualmente de https://huggingface.co/ggml-org/Qwen3-TTS-12Hz-1.7B-Base-GGUF/tree/main, y almacenarlos en \modelo

- El tamaño del GGUF no equivale al consumo total de VRAM. llama.cpp tiene además un problema conocido al estimar cuánto cabe cuando hay un mmproj: el ajuste del modelo principal puede consumir la memoria disponible sin contabilizar correctamente la memoria que necesitará posteriormente el projector.

    - Ajustando "capas_gpu": 8 ha sido un punto en el que el modelo deja espacio suficiente para el mmproj y sus buffers. Esto controla cuántas capas del modelo principal se descargan/offload a la GPU.

- Además, hay un issue muy reciente de llama.cpp dedicado exactamente a Qwen3-TTS 1.7B: si llama-tts no recibe -c, utiliza el contexto del modelo de 32768 tokens, provocando una caché KV de alrededor de 3.5 GB por sí sola.

    - Agregando "-c", "4096", la KV cache baja aproximadamente de 3584 MiB a 448 MiB, y en la prueba reportada el consumo total bajó a alrededor de 2.5 GB sin penalización de velocidad.