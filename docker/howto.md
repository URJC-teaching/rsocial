docker build -t ros2_docker jazzy
docker run -d --name robd -p 6080:6080 ros2_docker
http://localhost:6080/  
http://127.0.0.1:6080/

El escritorio y la terminal se ejecutan con el usuario `ubuntu`.
La contraseña inicial es `ubuntu`; el usuario puede elevar privilegios con `sudo` cuando sea necesario.

Nota: la imagen `lyrical` incluye `rsocial` y `audio_common` precompilados, pero no `kobuki` (su repositorio y algunas de sus dependencias ROS aún no están disponibles para la distro Lyrical).

## 1. Workspace ya incluido en la imagen

El workspace `~/ros2_ws` ya incluye `rsocial`, `kobuki` (con sus componentes de terceros) y `audio_common` clonados y compilados. Para recompilar tras modificar el código:

```bash
cd ~/ros2_ws
colcon build --symlink-install
```

Si necesitas volver a resolver dependencias (por ejemplo, tras actualizar los repositorios):

```bash
cd ~/ros2_ws
sudo apt update
rosdep update
sudo rosdep install --from-paths src --ignore-src -r -y --skip-keys="ament_python rclpy_lifecycle"
colcon build --symlink-install
```

## 2. Clonar los repositorios desde cero

Si prefieres empezar de nuevo, borra o renombra `~/ros2_ws/src` y clona los repositorios manualmente:

```bash
mkdir -p ~/ros2_ws/src
git clone https://github.com/URJC-teaching/rsocial.git ~/ros2_ws/src/rsocial
git clone --branch jazzy https://github.com/IntelligentRoboticsLabs/kobuki.git ~/ros2_ws/src/kobuki
git clone https://github.com/mgonzs13/audio_common.git ~/ros2_ws/src/audio_common
cd ~/ros2_ws/src
vcs import < kobuki/thirdparty.repos
cd ~/ros2_ws
sudo apt update
rosdep update
sudo rosdep install --from-paths src --ignore-src -r -y --skip-keys="ament_python rclpy_lifecycle"
colcon build --symlink-install
```

## 3. Copiar los repositorios desde tu ordenador

Si ya tienes los repositorios clonados en local, cópialos al contenedor antes de importar los componentes y compilar:

```bash
docker cp /ruta/a/rsocial robd:/home/ubuntu/ros2_ws/src/rsocial
docker cp /ruta/a/kobuki robd:/home/ubuntu/ros2_ws/src/kobuki
docker cp /ruta/a/audio_common robd:/home/ubuntu/ros2_ws/src/audio_common
```

Después, dentro del contenedor:

```bash
cd ~/ros2_ws/src
vcs import < kobuki/thirdparty.repos
cd ~/ros2_ws
sudo apt update
rosdep update
sudo rosdep install --from-paths src --ignore-src -r -y --skip-keys="ament_python rclpy_lifecycle"
colcon build --symlink-install
```

docker stop robd
docker start robd

* Parar el docker: docker stop robd
* Eliminar el container: docker rm robd
* Eliminar la imagen: docker rm ros2_docker