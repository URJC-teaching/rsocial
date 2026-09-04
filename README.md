# rsocial

El repositorio contiene nodos Python, comunicación entre nodos, TF, sensores, navegación, máquinas de estados, árboles de comportamiento e interacción humanx-robot.

## Requisitos

- Ubuntu 24.04 con ROS 2 Jazzy instalado (la imagen Docker del repositorio ya incluye este entorno).
- `git`, `python3`, `python3-rosdep`, `python3-vcstool`, `python3-colcon-common-extensions` y [`uv`](https://docs.astral.sh/uv/).
- Para los ejemplos de movimiento y navegación: un robot Kobuki o una simulación que publique los topics y TF necesarios.

En cada terminal desde la que se ejecute ROS hay que cargar ROS y este workspace:

```bash
source /opt/ros/jazzy/setup.bash
source ~/rsocial/install/setup.bash
```

## Instalación desde cero

```bash
mkdir -p ~/rsocial/src
git clone https://github.com/rodperex/rsocial.git ~/rsocial/src/rsocial
cd ~/rsocial/src
vcs import < rsocial/src/thirdparty.repos
cd ~/rsocial

rosdep update
rosdep install --from-paths src --ignore-src -r -y \\
	--skip-keys="ament_python rclpy_lifecycle"

# Las dependencias Python de los third parties no las instala rosdep.
# Crear el entorno con los paquetes de ROS visibles desde el venv.
uv venv --system-site-packages .venv
source .venv/bin/activate
uv pip install -r src/thirdparty/yolo_ros/requirements.txt
uv pip install -r src/thirdparty/simple_hri/simple_hri/requirements.txt

# Necesario para sound_play (reproducción y síntesis de audio).
sudo apt install -y gstreamer1.0 gstreamer1.0-alsa \\
  gstreamer1.0-plugins-base gstreamer1.0-plugins-good \\
  gstreamer1.0-plugins-ugly python3-gi

colcon build --symlink-install
source install/setup.bash
```

En cada terminal que ejecute nodos hay que activar también `.venv` después de
cargar ROS y el workspace:

```bash
source /opt/ros/jazzy/setup.bash
source ~/rsocial/.venv/bin/activate
source ~/rsocial/install/setup.bash
```

`yolo_ros` necesita instalarse explícitamente con `uv`; si se omite ese paso,
`rosdep` puede terminar correctamente pero el nodo fallará al importar
`ultralytics`, `cv2` o `lap`. La restricción `numpy<2` de `yolo_ros` es
intencionada: evita incompatibilidades entre `cv_bridge`/ROS 2 Jazzy y NumPy
2. No se debe sustituir por la versión global de NumPy sin comprobar antes
que la cámara y la conversión de imágenes siguen funcionando.

### Portabilidad de `simple_hri`

`simple_hri` incluye servicios locales y servicios que requieren credenciales.
El entorno virtual contiene las dependencias Python y ve los paquetes ROS del
sistema gracias a `--system-site-packages`, incluido el wheel de WebRTC VAD.
Los launchers
locales no necesitan claves, pero sí descargan modelos la primera
vez y requieren micrófono y salida de audio accesibles desde el sistema:

```bash
ros2 launch simple_hri local_simple_hri.launch.py
```

Si el nodo no arranca fuera del equipo original, comprueba primero el
dispositivo de audio y las importaciones desde la misma terminal donde se
cargaron ROS y el entorno virtual:

```bash
python3 -c "import numpy, torch, transformers, whisper, sounddevice, webrtcvad; print('Dependencias Python OK')"
python3 -c "import sounddevice as sd; print(sd.query_devices())"
```

En sistemas sin micrófono o servidor de audio, se pueden probar los servicios
de texto, pero los servicios STT/TTS locales no podrán grabar o reproducir
audio hasta configurar ALSA/PulseAudio o el equivalente del sistema.

## Ejemplos básicos

Los comandos siguientes se ejecutan en terminales separadas cuando se lanzan varios nodos.

### Publicación, suscripción y ciclo de vida

```bash
ros2 launch node_programming pubsub.launch.py
ros2 launch node_programming lc_pubsub.launch.py
```

También están disponibles los nodos individuales `publisher_node`, `subscriber_node`, `logger_node`, `lifecycle_publisher_node`, `lifecycle_subscriber_node`, `simple_node_creation`, `simple_node_logging`, `simple_node_publishing` y `simple_callback`.

### Servicios y acciones

Ejecuta un servidor y su cliente en terminales distintas:

```bash
ros2 run comms_demo service_server
ros2 run comms_demo service_client

ros2 run comms_demo action_server
ros2 run comms_demo action_client
```

### Movimiento y TF

Estos ejemplos necesitan un robot o simulador que acepte `cmd_vel`:

```bash
ros2 run square_motion square_move
ros2 run tf_square_motion tf_square
ros2 run tf_square_motion tf_square2
ros2 launch tf_seeker tf_seeker.launch.py
```

## Sensores

### Láser

Con un `LaserScan` publicado por el robot:

```bash
ros2 launch laser laser.launch.py
```

Los nodos individuales son `obstacle_detector_node` y `obstacle_detector_node_no_tf`.

### Cámara y YOLO

Primero inicia una cámara. Para una cámara OAK-D:

```bash
ros2 launch oak_d_camera camera.launch.py \\
	use_disparity:=False use_lr_raw:=False use_pointcloud:=False
```

Después inicia YOLO con los topics de la OAK-D y transforma sus detecciones:

```bash
ros2 launch yolo_bringup yolo.launch.py \\
	input_image_topic:=/color/image \\
	input_depth_topic:=/stereo/depth \\
	input_depth_info_topic:=/stereo/camera_info \\
	target_frame:=oak-d_frame

ros2 launch camera yolo_to_standard2d.launch.py
```

Para detecciones 3D usa `yolo_to_standard3d.launch.py`. Con otra cámara hay que sustituir los topics y `target_frame` por los que publique ese dispositivo.

## Control y navegación

El control VFF puede ejecutarse en 2D o 3D. Los launch completos incluyen los nodos de detección necesarios para el escenario configurado:

```bash
ros2 launch vff_control full_vff_2d.launch.py
ros2 launch vff_control full_vff_3d.launch.py
```

También se pueden lanzar por separado `vff_2d.launch.py`, `vff_3d.launch.py`, `yolo_class_2d.launch.py`, `yolo_class_3d.launch.py` y `obstacle_detector.launch.py`.

Ejemplos de navegación:

```bash
ros2 run nav2_example simple_navigation_app
ros2 launch fsm_nav fsm_nav.launch.py
```

La navegación requiere Nav2 activo, mapa/localización y los topics/TF del robot. Los puntos de la FSM se configuran en `src/fsm_nav/config/waypoints.yaml`.

## Máquinas de estados y árboles de comportamiento

```bash
ros2 launch fsm_bumpgo bumpgo.launch.py
ros2 launch bt_bumpgo bumpgo.launch.py
ros2 launch bt_bumpgo side_bumpgo.launch.py
ros2 launch bt_bumpgo groot_bumpgo.launch.py
```

Los ejemplos didácticos de `py_trees` también se pueden ejecutar directamente:

```bash
ros2 run bt_examples sequence
ros2 run bt_examples reactive_sequence
ros2 run bt_examples fallback
ros2 run bt_examples reactive_fallback
ros2 run bt_examples decorator
```

Para editar árboles con Groot, instala el puente una vez:

```bash
python3 -m pip install --user \\
	git+https://github.com/narcispr/py_trees_meet_groot.git
```

## Interacción humano-robot

Los ejemplos usan los servicios de `simple_hri`. Para modelos locales:

```bash
ros2 launch hri_examples hri_dependencies.launch.py
ros2 run hri_examples say
ros2 run hri_examples repeat
ros2 launch hri_examples generate_response.launch.py
```

Para los servicios en la nube, inicia el launch correspondiente de `simple_hri` y configura antes sus credenciales como indique ese paquete:

```bash
ros2 launch simple_hri simple_hri.launch.py
ros2 run hri_examples hri_example
```

Otros ejecutables disponibles son `hri_example2`, `hri_example3`, `nao_hri_example` y sus clientes.

## Docker

Para trabajar con el entorno Docker, sigue la guía específica [docker/howto.md](docker/howto.md). Ahí se explica cómo:

- construir y arrancar las imágenes Jazzy y Lyrical;
- acceder al escritorio ROS 2 desde el navegador;
- usar el workspace que ya viene compilado;
- clonar el workspace desde cero o copiar tus repositorios al contenedor;
- recompilar y detener/eliminar el contenedor.

La guía Docker es la referencia para todo lo relacionado con contenedores; el resto de este README describe los ejemplos y sus comandos ROS 2.

## Licencia

[Apache License 2.0](LICENSE)