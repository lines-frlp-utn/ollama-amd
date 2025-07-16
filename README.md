# Cómo ejecutar Ollama con Docker en Linux usando GPU AMD

Si tienes una tarjeta gráfica AMD y quieres usarla con Ollama dentro de un contenedor Docker, este artículo te guía paso a paso. Este ejemplo ha sido realizado y probado en un sistema Linux Mint con una placa gráfica AMD Radeon RX 7600.

## 1. Verifica la compatibilidad de tu GPU

Antes de comenzar, asegúrate de que tu tarjeta AMD sea compatible con Ollama. Puedes consultar la lista oficial de GPUs compatibles aquí:

🔗 [Lista de GPUs AMD compatibles con Ollama](https://github.com/ollama/ollama/blob/main/docs/gpu.md)

## 2. Instala Docker

Ollama se ejecutará dentro de un contenedor Docker, así que necesitas tener Docker instalado en tu sistema.

- 📘 [Guía oficial de instalación de Docker](https://docs.docker.com/engine/install/)
- 📚 [Docker en Arch Linux (ArchWiki)](https://wiki.archlinux.org/title/Docker)

## 3. Crea un archivo compose.yml y pega este contenido

```yaml
services:
  ollama:
    container_name: ollama
    image: ollama/ollama:rocm
    volumes:
      - ollama:/root/.ollama
    ports:
      - "11435:11435"
    environment:
      - OLLAMA_DEBUG=true
    restart: ${RESTART_POLICY:-no}
    devices:
      - /dev/kfd
      - /dev/dri

volumes:
  ollama:
    external: true
```

### Estructura principal

**services:** - Define los contenedores que se van a ejecutar. Cada servicio es un contenedor independiente.

**ollama:** - Nombre del servicio. Puedes usar cualquier nombre descriptivo.

### Configuración del contenedor

**container_name: ollama** - Nombre específico que tendrá el contenedor en Docker (opcional, si no se especifica, Docker genera uno automáticamente).

**image: ollama/ollama:rocm** - Imagen base del contenedor. Aquí usa la versión ROCm (para GPUs AMD) de Ollama desde Docker Hub.

**volumes:** - Monta almacenamiento persistente:
- `ollama:/root/.ollama` - Monta el volumen nombrado "ollama" en la carpeta `/root/.ollama` del contenedor

**ports:** - Mapea puertos entre el host y el contenedor:
- `"11435:11435"` - El puerto 11435 del host se conecta al puerto 11435 del contenedort

**environment:** - Variables de entorno para el contenedor:
- `OLLAMA_DEBUG=true` - Activa el modo debug de Ollama

**restart: ${RESTART_POLICY:-no}** - Política de reinicio. Usa la variable de entorno RESTART_POLICY, y si no existe, usa "no" por defecto.

**devices:** - Da acceso a dispositivos del sistema:
- `/dev/kfd` y `/dev/dri` - Necesarios para usar GPUs AMD con ROCm

### Volúmenes

**volumes:** (sección global) - Define volúmenes persistentes:
- `external: true` - Indica que el volumen "ollama" ya existe y fue creado externamente

## 4. Crear un volumen

```bash
docker volume create ollama
```

### ¿Por qué usar volúmenes?

**Persistencia de datos**: Sin volúmenes, cuando eliminas o actualizas el contenedor, pierdes todos los datos. Los volúmenes mantienen la información incluso si el contenedor se destruye.

**Separación de datos y aplicación**: Los datos viven independientemente del contenedor, lo que facilita actualizaciones y mantenimiento.

En nuestro caso, el volumen en `/root/.ollama` guarda:
- Modelos descargados (pueden ser varios GB)
- Configuraciones personalizadas
- Historial de conversaciones

Sin volumen, cada vez que recrees el contenedor tendrías que descargar todos los modelos nuevamente, lo que puede tomar horas dependiendo del tamaño.

## 5. Levantar el servicio

```bash
docker compose -f linux-compose.yml up -d
```

```bash
docker compose -f windows-compose.yml up -d
```

## 6. Ejecutar ollama

```bash
docker exec -it ollama ollama run llama3.2
```