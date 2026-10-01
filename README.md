# 2D Gaussian Splatting — Entorno de Proyecto Final de Carrera I (Daniela Perales)

Este fork documenta la instalación y puesta en funcionamiento de 2D Gaussian Splatting (Huang et al., SIGGRAPH 2024) como base para la implementación del curso de Proyecto Final de Carrera I, que posteriormente integrará las técnicas de NPGS (Enhancing Sparse-View 3DGS with Guidance of Normals Priors and Dense Point Initialization): normales estimadas con Metric3D v2 y nube de puntos densa generada con RoMa.

Repositorio original: https://github.com/hbb1/2d-gaussian-splatting

## Hardware y software de referencia

- GPU: NVIDIA GeForce RTX 4060 Laptop (Ada Lovelace, compute capability 8.9 / sm_89)
- Driver NVIDIA: 580.95.05, CUDA soportado hasta 13.0
- CUDA Toolkit instalado a nivel de sistema: 13.0 (/usr/local/cuda)
- OS: Ubuntu 24.04 (Noble)
- Python: 3.12.3
- PyTorch: 2.14.0+cu130

## Instalación

### 1. Submódulos
git submodule update --init --recursive

### 2. Entorno virtual (venv nativo)
sudo apt install python3.12-venv python3.12-dev -y
python3 -m venv venv
source venv/bin/activate
pip install --upgrade pip

### 3. PyTorch (build para CUDA 13.0, compatible con Ada/sm_89)
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu130

### 4. Dependencias adicionales
El environment.yml original esta desactualizado (pensado para CUDA 11.6 / PyTorch 1.12, incompatible con GPUs Ada). Dependencias reales instaladas via pip, congeladas en requirements-instalacion.txt:

pip install plyfile tqdm opencv-python trimesh matplotlib mediapy
pip install "open3d==0.19.0"   # ver nota de bug más abajo, NO usar 0.20.0

### 5. Compilación de submódulos CUDA
export TORCH_CUDA_ARCH_LIST="8.9"
pip install --no-build-isolation submodules/diff-surfel-rasterization
pip install --no-build-isolation submodules/simple-knn

## Problemas encontrados y soluciones

- Error "namespace std has no member uintptr_t" al compilar diff-surfel-rasterization: falta #include <cstdint> en cuda_rasterizer/rasterizer_impl.h (compiladores nuevos con CUDA 13 ya no lo incluyen implícitamente). Solución: agregar ese include al inicio del archivo.
- Error "fatal error: Python.h: No existe el archivo o el directorio": falta el paquete de cabeceras de desarrollo de Python. Solución: sudo apt install python3.12-dev
- Error "ModuleNotFoundError: No module named torch" al compilar submódulos: pip usa un entorno de build aislado que no ve el venv activo. Solución: usar pip install --no-build-isolation
- Malla extraída con 0 vértices en render.py pese a mapas de profundidad válidos: bug de compatibilidad del pipeline legacy ScalableTSDFVolume con Open3D 0.20.0. Solución: usar open3d==0.19.0 (la 0.18.0 no tiene wheel para Python 3.12)

## Flujo de entrenamiento (validado con DTU scan24)

source venv/bin/activate

python train.py -s <ruta_dataset>/scan24 -m <ruta_output>/scan24 -r 2 --depth_ratio 1

python render.py -m <ruta_output>/scan24 -s <ruta_dataset>/scan24 -r 2 --depth_ratio 1 --skip_test --skip_train

Resultado de referencia (scan24, 30k iteraciones, ~29 min en RTX 4060 8GB): PSNR train 36.1, malla extraída (fuse_post.ply) con 305678 vértices y 10569 clusters.

## Próximos pasos

- Integrar Metric3D v2 para generar normales como prior
- Integrar RoMa para inicialización densa de puntos (DFTri)
