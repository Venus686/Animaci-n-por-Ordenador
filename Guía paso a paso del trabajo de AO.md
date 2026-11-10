# Guía paso a paso del trabajo de AO

Nov 9, 2026 · @Lucia

En 10 fases pasas de la práctica 7 a un esqueleto que hace un lanzamiento de peso; después de cada fase compilas y compruebas que funciona antes de seguir.

## Cómo usar esta guía

Vas a construir el trabajo en 4 piezas, una detrás de otra, y después de cada pieza compilas y miras la ventana: si lo que ves coincide con lo que dice la guía, sigues; si no, arreglas antes de avanzar.

Resultado final: una ventana titulada "Trabajo" con el suelo verde y un esqueleto de esferas rojas y cilindros azules que, cada 5 segundos, se agacha, se impulsa, gira y lanza el peso con el brazo derecho.

Las 4 piezas, de abajo arriba:

1. **GEBalljoint** (modificar): una articulación que ahora tiene hijas, límites y pose. Es el ladrillo.
2. **GESkeleton** (nueva): lee body.skel y monta el árbol de 19 articulaciones. Para la escena es "una figura más".
3. **GEAnimation** (nueva): guarda las poses clave (keyframes) y calcula la pose de cada instante interpolando.
4. **GEScene** (modificar): crea suelo + esqueleto + animación y, en cada frame, pide a la animación la pose del instante actual.

Cómo leer los bloques de código:

- Cuando la guía dice **"sustituye el fichero entero"**, borra todo lo que tenga y pega el bloque completo.
- Cuando dice **"añade"** o **"cambia"**, solo toca lo indicado.
- Escíbelo tú (o cópialo) pero lee la explicación de antes: en la memoria tendrás que explicar cada método.
- Cada bloque tiene la cabecera de comentario `FUNCIÓN / PROPÓSITO` que usa el profesor; mantenla, es parte del estilo de la asignatura.

Tiempo orientativo: fase 0–3, 1 hora; fase 4, 1 hora; fase 5, 2 horas; fase 6–7, 2 horas; fase 8–9, 1–2 horas más la memoria.

## Fase 0. Abrir la práctica 7 en Visual Studio

Objetivo: ver en tu PC la ventana de la práctica 7 (la cruz azul con dos brazos que giran). Si esto funciona, todo lo demás funcionará.

**Paso 0.1 — Visual Studio 2022.** Si no lo tienes, instala Visual Studio Community 2022. En el instalador marca la carga de trabajo **"Desarrollo para el escritorio con C++"**. El proyecto usa el conjunto de herramientas v143, que es el de VS 2022.

**Paso 0.2 — Vulkan SDK.** El proyecto busca el SDK exactamente en `C:\VulkanSDK\1.4.313.2`. Descarga el instalador de esa versión desde vulkan.lunarg.com (SDK → Windows) e instálalo en la ruta que propone. Si ya tienes otra versión, mira el paso 0.6.

**Paso 0.3 — Colocar los ficheros.** El .rar de la práctica contiene una carpeta `GameEngine` con `Icon`, `Projects`, `Shaders` y `Tools`. Descomprímelo en `C:\` para que quede así:

```
C:\GameEngine\Icon\toy.ico
C:\GameEngine\Tools\GLFW34\...   GLM\...   STBI\...
C:\GameEngine\Projects\Project7\Project7.sln
C:\GameEngine\Projects\Project7\Project7\  (aquí están todos los .cpp y .h)
```

La ruta importa: el proyecto tiene escrito `C:\GameEngine\Tools\...` para encontrar GLFW y GLM y `C:\GameEngine\Icon\toy.ico` para el icono.

**Paso 0.4 — Abrir.** Doble clic en `C:\GameEngine\Projects\Project7\Project7.sln`. En la barra de arriba elige **Release** y **x64** (los dos desplegables junto al botón verde).

**Paso 0.5 — Compilar y ejecutar.** Menú Compilar → Compilar solución (Ctrl+Mayús+B). Abajo, en la ventana Salida, debe acabar en "1 correctos, 0 incorrectos". Después Depurar → Iniciar sin depurar (Ctrl+F5).

Punto de control 0: se abre una consola negra y una ventana "Game Engine" con suelo verde, una cruz azul y dos articulaciones rojas que giran en bucle. Cierra la ventana para terminar.

**Paso 0.6 — Si tienes otra versión del SDK.** Clic derecho sobre el proyecto Project7 (no sobre la solución) → Propiedades. Arriba elige Configuración: **Todas las configuraciones** y Plataforma: **x64**. Cambia `1.4.313.2` por tu versión en dos sitios:

- C/C++ → General → Directorios de inclusión adicionales (`C:\VulkanSDK\<tu versión>\Include`).
- Vinculador → General → Directorios de bibliotecas adicionales (`C:\VulkanSDK\<tu versión>\Lib`).

Si falla aquí, no sigas: mira la sección de errores típicos al final.

## Fase 1. Entender la práctica 7 (no se escribe código)

De las 24 clases solo vas a tocar 2 (GEBalljoint y GEScene) y a imitar otra (GEFigure); el resto es el "motor" de Vulkan y no se toca.

**Qué hace cada fichero**

| Fichero | Para qué sirve | ¿Lo tocas? |
| --- | --- | --- |
| main.cpp | Crea un GEApplication y llama a run() | Solo el comentario de cabecera |
| GEApplication | Crea la ventana con GLFW, los contextos de Vulkan y la escena; bucle principal; teclado; redimensionado | Solo el título de la ventana |
| GEGraphicsContext | Instancia de Vulkan, superficie, GPU física y lógica, colas | No |
| GEDrawingContext | Swapchain, imágenes, semáforos y fences (sincronización), presentación | No |
| GECommandContext | Pool y command buffers (uno por imagen de la swapchain) | No |
| GERenderingContext + GEPipelineConfig | Render pass, framebuffers, buffer de profundidad, layout de descriptores y el pipeline gráfico (carga los shaders) | No |
| GEVertexBuffer, GEIndexBuffer, GEUniformBuffer, GEDescriptorSet, GEDepthBuffer | Envoltorios de buffers y descriptores de Vulkan | No |
| GEVertex.h, GETransform.h, GEMaterial.h, GELight.h | Structs: vértice (posición + normal), matrices para el shader, material Phong, luz direccional | No |
| GEFigure | Una malla dibujable: vértices, índices, matriz `location` y 3 uniform buffers | No, pero GESkeleton copia sus métodos |
| GESphere, GECylinder, GEGround | Hijas de GEFigure que generan la malla de una esfera, un cilindro y el suelo | No |
| GECamera | Posición y dirección de la cámara, movimiento con teclas | No |
| GEBalljoint | Una articulación: esfera roja + cilindro azul | **Sí: fase 4** |
| GEScene | Lo que se ve: cámara, luz, suelo, cruz y dos articulaciones | **Sí: fases 5 y 7** |
| shaders\\shader.vert / .frag (+ .spv) | Iluminación Phong | No |
| Project7.rc + resource.h | Recursos que se meten dentro del .exe: los dos shaders compilados y el icono | **Sí: fase 3** |

**Cómo se ejecuta un frame.** `GEApplication::mainLoop()` repite: `glfwPollEvents()` (teclado) y `draw()`. `draw()` hace 4 cosas: espera a la siguiente imagen de la swapchain, llama a `scene->update(gc, imagen)`, envía los command buffers y presenta la imagen.

La idea clave: los **command buffers se graban una sola vez** en el constructor de la escena (`fillCommandBuffers`: "dibuja el suelo, dibuja la cruz…"). Lo que cambia en cada frame **no son los comandos sino los uniform buffers** (las matrices), que se actualizan en `update`. Por eso para animar basta con cambiar matrices: el esqueleto se graba una vez y después solo movemos matrices.

**Cómo funciona GEFigure.** Cada figura guarda `location` (su matriz de modelo, mat4). Sus métodos:

- `initialize(gc, rc)`: crea los buffers de Vulkan.
- `addCommands(cb, layout, i)`: graba en el command buffer i el dibujo de la figura.
- `update(gc, i, view, projection)`: calcula `MVP = projection · view · location` y lo copia al uniform buffer de la imagen i, junto con el material y la luz.
- `setLocation(m)`, `resetLocation()`, `translate(t)`, `rotate(ángulo, eje)`: cambian `location`.
- `setMaterial(m)`, `setLight(l)`: color y luz.
- `destroy(gc)`: libera los buffers.

Esta es la interfaz que tendrá que tener GESkeleton ("los mismos métodos que GEFigure").

**Cómo funciona GEBalljoint (versión de la práctica 7).** Guarda la longitud del hueso (`length`), su posición (`location`), sus tres ejes propios (`right` = X, `up` = Y, `dir` = Z) y la pose (`angles[3]`, ángulos de Euler en grados). Se dibuja con una `GESphere` (la rótula) y un `GECylinder` (el hueso, que apunta en su eje Z).

Toda la magia está en `ComputeMatrix()`, que construye dos matrices y las multiplica:

- `jointm` (posicionamiento): sus columnas son `right`, `up`, `dir` y `location`. Coloca la articulación en su sitio y la orienta. (En glm, `m[columna][fila]`.)
- `posem` (pose): la matriz de rotación de los ángulos de Euler XYZ que aparece en la página de la práctica, equivalente a Rz·Ry·Rx.
- `matrix = jointm * posem`: primero se gira (pose) en los ejes propios de la articulación y después se coloca en su sitio.

La esfera se pone con `matrix` y el cilindro con `matrix` desplazada `length/2` en Z (porque el cilindro de longitud `length/2` va de −length/2 a +length/2 alrededor de su centro; al desplazarlo, queda de 0 a length).

**Qué le falta para ser un esqueleto.** Una articulación de la práctica 7 está sola: su `location` está en coordenadas de la escena. En un esqueleto el antebrazo cuelga del brazo: si giras el hombro, el codo y la muñeca deben moverse con él. Eso se consigue con una **matriz del padre**:

```latex
M_{\text{hija}} = M_{\text{extremo del padre}} \cdot jointm_{\text{hija}} \cdot posem_{\text{hija}}
```

El "extremo del padre" es la matriz del padre desplazada `length` en su eje Z: así lo dice el enunciado ("la posición (0,0,0) de una articulación hija corresponde al extremo de la articulación").

## Fase 2. Crear tu proyecto del trabajo

Trabaja sobre una copia: así la práctica 7 original sigue intacta para compararla si algo se rompe.

1. Cierra Visual Studio.
2. En el Explorador de Windows, copia la carpeta `C:\GameEngine\Projects\Project7` y pégala en `C:\GameEngine\Projects\`. Renombra la copia a **Trabajo**.
3. Dentro de `C:\GameEngine\Projects\Trabajo\` borra, si existen, las carpetas `.vs` (oculta) y `x64` (son restos de compilaciones; Visual Studio las vuelve a crear). Dentro de `Trabajo\Project7\` borra también `x64` si existe.
4. Abre `C:\GameEngine\Projects\Trabajo\Project7.sln`. Los ficheros internos siguen llamándose Project7: no los renombres, porque la solución, el proyecto y el .rc se referencian entre sí por nombre y es fácil romperlo. Lo que cuenta es la carpeta.
5. Comprueba que trabajas en la copia: en el Explorador de soluciones, clic derecho en cualquier .cpp → "Abrir la carpeta en el Explorador de archivos". La ruta debe contener `Projects\Trabajo`.
6. Cambia el título de la ventana. En **GEApplication.cpp** busca (Ctrl+F) la línea:

```cpp
	GLFWwindow* w = glfwCreateWindow(WIDTH, HEIGHT, "Game Engine", nullptr, nullptr);
```

y cámbiala por:

```cpp
	GLFWwindow* w = glfwCreateWindow(WIDTH, HEIGHT, "Trabajo", nullptr, nullptr);
```

7. En **main.cpp** actualiza el comentario de cabecera (PROYECTO: Trabajo, y una descripción: "Animación de un esqueleto descrito en body.skel realizando un lanzamiento de peso").

Punto de control 2: compila (Ctrl+Mayús+B) y ejecuta (Ctrl+F5). Ves lo mismo que en la fase 0, pero la ventana se llama "Trabajo".

## Fase 3. Añadir body.skel como recurso

body.skel se mete dentro del .exe igual que el profesor mete los shaders; así el programa nunca falla por "no encuentro el fichero" aunque lo ejecutes desde otra carpeta.

**Por qué así.** Mira `GERenderingContext::getFileFromResource(int resource)`: con `FindResource(NULL, MAKEINTRESOURCE(resource), RT_HTML)` busca un recurso de tipo HTML dentro del propio .exe y lo devuelve como bytes. Los shaders son los recursos `IDR_HTML1` (vert.spv) e `IDR_HTML2` (frag.spv). Haremos lo mismo con `IDR_HTML3` = body.skel.

**Paso 3.1 — Crear el fichero.** Abre el Bloc de notas, pega el contenido completo de body.skel del enunciado (empieza por `<skeleton name="body">` y acaba en `</skeleton>`) y guárdalo como `C:\GameEngine\Projects\Trabajo\Project7\body.skel` (en "Tipo" elige "Todos los archivos" para que no añada .txt). Tiene que estar en la misma carpeta que los .cpp.

**Paso 3.2 — Verlo en el proyecto.** En Visual Studio: clic derecho sobre el proyecto Project7 → Agregar → Elemento existente… → body.skel. Ahora puedes editarlo desde Visual Studio.

**Paso 3.3 — Declarar el identificador.** Abre **resource.h** y añade la línea de `IDR_HTML3`; sube también el siguiente valor libre de 104 a 105. Debe quedar así:

```cpp
#define IDR_HTML1                       101
#define IDR_HTML2                       102
#define IDI_ICON1                       103
#define IDR_HTML3                       104
```

```cpp
#define _APS_NEXT_RESOURCE_VALUE        105
```

**Paso 3.4 — Incluirlo en el .rc.** En el Explorador de soluciones abre la carpeta "Archivos de recursos", clic derecho sobre **Project7.rc** → **Ver código** (si te avisa de que está abierto en el editor de recursos, ciérralo y repite). Busca las líneas de los shaders y añade la tercera debajo:

```
IDR_HTML1               HTML                    "shaders\\vert.spv"

IDR_HTML2               HTML                    "shaders\\frag.spv"

IDR_HTML3               HTML                    "body.skel"
```

La ruta es relativa a la carpeta del .rc, por eso basta con "body.skel". Guarda (Ctrl+S).

Punto de control 3: Compilar → **Recompilar solución**. Debe compilar sin errores y la ventana sigue igual (todavía nadie lee el recurso). Si cambias body.skel más adelante, usa siempre Recompilar solución para que se vuelva a meter en el .exe.

## Fase 4. Modificar GEBalljoint

GEBalljoint pasa de ser una articulación suelta a un nodo de un árbol: guarda sus hijas, recibe la matriz de su padre, tiene límites y nombre, y todos sus métodos se repiten en las hijas (recursión).

**Resumen de cambios**

| Qué se añade | Para qué |
| --- | --- |
| `std::string name` | Buscar articulaciones por nombre ("knee\_l") desde la animación |
| `std::vector<GEBalljoint*> children` | Las articulaciones hijas (el árbol) |
| `minAngles[3]`, `maxAngles[3]` | Límites de los grados de libertad |
| `parentMatrix` | Sistema de coordenadas del padre (su extremo) |
| `matrix` | Matriz final de esta articulación, guardada para usarla en `update` |
| Recursión en initialize, update, addCommands, destroy, setLight | "Los métodos deben realizar el recorrido actualizando y dibujando las hijas" |
| Getters/setters | "Getters y setters para incluir las poses, la posición y orientación y para añadir las hijas" |

Un cambio importante de diseño: en la práctica 7, `ComputeMatrix()` colocaba directamente la esfera y el cilindro. Ahora `ComputeMatrix()` se llama también antes de `initialize()` (al leer el fichero, cuando aún no existen esfera ni cilindro), así que solo calcula y guarda `matrix`; la esfera y el cilindro se colocan en `update()`.

### 4.1 GEBalljoint.h — sustituye el fichero entero

```cpp
#pragma once

#include "GEGraphicsContext.h"
#include "GERenderingContext.h"
#include "GESphere.h"
#include "GECylinder.h"
#include "GELight.h"
#include "GEMaterial.h"
#include <glm\glm.hpp>
#include <string>
#include <vector>

//
// CLASE: GEBalljoint
//
// DESCRIPCIÓN: Representa una articulación con 3 grados de libertad.
//              Cada articulación puede tener articulaciones hijas, formando
//              un árbol de articulaciones (esqueleto).
// 
class GEBalljoint {
private:
	std::string name;
	GLfloat length;
	glm::vec3 location;
	glm::vec3 dir;
	glm::vec3 up;
	glm::vec3 right;
	GLfloat angles[3];
	GLfloat minAngles[3];
	GLfloat maxAngles[3];
	glm::mat4 parentMatrix;
	glm::mat4 matrix;
	std::vector<GEBalljoint*> children;
	GESphere* joint;
	GECylinder* bone;
	void ComputeMatrix();

public:
	GEBalljoint(std::string name, float length);
	void initialize(GEGraphicsContext* gc, GERenderingContext* rc);
	void destroy(GEGraphicsContext* gc);
	void addCommands(VkCommandBuffer commandBuffer, VkPipelineLayout pipelineLayout, int index);
	void update(GEGraphicsContext* gc, uint32_t index, glm::mat4 view, glm::mat4 projection);

	void setLight(GELight l);
	void setBoneMaterial(GEMaterial m);

	std::string getName();
	float getLength();

	void setLocation(glm::vec3 loc);
	glm::vec3 getLocation();
	void setOrientation(glm::vec3 nDir, glm::vec3 nUp);
	glm::vec3 getDirection();
	glm::vec3 getUpDirection();

	void setPose(float xrot, float yrot, float zrot);
	glm::vec3 getPose();
	void setLimits(float xmin, float xmax, float ymin, float ymax, float zmin, float zmax);
	glm::vec3 getMinLimits();
	glm::vec3 getMaxLimits();

	void setParentMatrix(glm::mat4 m);
	void addChild(GEBalljoint* child);
	std::vector<GEBalljoint*> getChildren();
	GEBalljoint* findJoint(std::string jname);
};
```

### 4.2 GEBalljoint.cpp — sustituye el fichero entero por los bloques siguientes, en este orden

**Includes.** Los mismos de la práctica 7:

```cpp
#include "GEBalljoint.h"

#include "GEMaterial.h"
#include "GESphere.h"
#include "GECylinder.h"
#include "GELight.h"
#include <glm\glm.hpp>
#include <glm\gtc\matrix_transform.hpp>
```

**Constructor.** Ahora recibe también el nombre. Valores por defecto que pide el enunciado: desplazamiento (0,0,0), orientación del padre (ejes identidad: X, Y, Z), pose (0,0,0). Límites por defecto -180..180, es decir, sin límite. Las matrices empiezan en identidad (`glm::mat4(1.0f)`).

```cpp
//
// FUNCIÓN: GEBalljoint::GEBalljoint(std::string n, float l)
//
// PROPÓSITO: Construye una articulación.
//
// COMENTARIOS:
//     Por defecto el desplazamiento es (0,0,0), la orientación es la del
//     nodo padre, la pose es (0,0,0) y no hay límites en los grados de
//     libertad (-180º a 180º).
//
GEBalljoint::GEBalljoint(std::string n, float l)
{
	name = n;
	length = l;

	location = glm::vec3(0.0f, 0.0f, 0.0f);
	dir = glm::vec3(0.0f, 0.0f, 1.0f);
	up = glm::vec3(0.0f, 1.0f, 0.0f);
	right = glm::vec3(1.0f, 0.0f, 0.0f);

	for (int i = 0; i < 3; i++)
	{
		angles[i] = 0.0f;
		minAngles[i] = -180.0f;
		maxAngles[i] = 180.0f;
	}

	parentMatrix = glm::mat4(1.0f);
	matrix = glm::mat4(1.0f);

	joint = nullptr;
	bone = nullptr;

	ComputeMatrix();
}
```

**ComputeMatrix().** Las dos primeras partes (`jointm` y `posem`) son exactamente las de la práctica 7: cópialas de tu fichero antiguo. Cambia el final: (1) se multiplica también por `parentMatrix`; (2) se calcula la matriz del extremo (`translate` de `length` en Z); (3) se le pasa a cada hija con `setParentMatrix`, que a su vez llama al `ComputeMatrix` de la hija. Así, cambiar el hombro recalcula automáticamente codo y muñeca.

```cpp
//
// FUNCIÓN: GEBalljoint::ComputeMatrix()
//
// PROPÓSITO: Crea la matriz de transformación a partir de la matriz del
//            padre, la posición, la orientación y la pose. Después propaga
//            la posición del extremo de la articulación a las articulaciones
//            hijas.
//
void GEBalljoint::ComputeMatrix()
{
	// Formato glm::mat4[column][row]
	glm::mat4 jointm;
	jointm[0][0] = right.x;
	jointm[0][1] = right.y;
	jointm[0][2] = right.z;
	jointm[0][3] = 0.0;

	jointm[1][0] = up.x;
	jointm[1][1] = up.y;
	jointm[1][2] = up.z;
	jointm[1][3] = 0.0;

	jointm[2][0] = dir.x;
	jointm[2][1] = dir.y;
	jointm[2][2] = dir.z;
	jointm[2][3] = 0.0;

	jointm[3][0] = location.x;
	jointm[3][1] = location.y;
	jointm[3][2] = location.z;
	jointm[3][3] = 1.0;

	float cx = (float)cos(glm::radians(angles[0]));
	float sx = (float)sin(glm::radians(angles[0]));
	float cy = (float)cos(glm::radians(angles[1]));
	float sy = (float)sin(glm::radians(angles[1]));
	float cz = (float)cos(glm::radians(angles[2]));
	float sz = (float)sin(glm::radians(angles[2]));

	glm::mat4 posem;
	posem[0][0] = cz * cy;
	posem[1][0] = -sz * cx + cz * sy * sx;
	posem[2][0] = sz * sx + cz * sy * cx;
	posem[3][0] = 0;

	posem[0][1] = sz * cy;
	posem[1][1] = cz * cx + sz * sy * sx;
	posem[2][1] = -cz * sx + sz * sy * cx;
	posem[3][1] = 0;

	posem[0][2] = -sy;
	posem[1][2] = cy * sx;
	posem[2][2] = cy * cx;
	posem[3][2] = 0;

	posem[0][3] = 0;
	posem[1][3] = 0;
	posem[2][3] = 0;
	posem[3][3] = 1;

	matrix = parentMatrix * jointm * posem;

	// El origen de las articulaciones hijas es el extremo de esta articulación
	glm::mat4 endMatrix = glm::translate(matrix, glm::vec3(0.0f, 0.0f, length));
	for (size_t i = 0; i < children.size(); i++)
	{
		children[i]->setParentMatrix(endMatrix);
	}
}
```

**initialize().** Igual que antes (crea esfera roja y cilindro azul con sus materiales) y, al final, inicializa cada hija. Por eso las hijas deben añadirse ANTES de llamar a `initialize` del padre.

```cpp
//
// FUNCIÓN: GEBalljoint::initialize(GEGraphicsContext* gc, GERenderingContext* rc)
//
// PROPÓSITO: Inicializa las piezas de la figura y de las articulaciones hijas
//
void GEBalljoint::initialize(GEGraphicsContext* gc, GERenderingContext* rc)
{
	GEMaterial jointMat = {};
	jointMat.Ka = glm::vec3(1.0f, 0.0f, 0.0f);
	jointMat.Kd = glm::vec3(1.0f, 0.0f, 0.0f);
	jointMat.Ks = glm::vec3(0.8f, 0.8f, 0.8f);
	jointMat.Shininess = 16.0f;

	joint = new GESphere(10, 20, 0.1f);
	joint->initialize(gc, rc);
	joint->setMaterial(jointMat);

	GEMaterial boneMat = {};
	boneMat.Ka = glm::vec3(0.0f, 0.0f, 0.8f);
	boneMat.Kd = glm::vec3(0.0f, 0.0f, 0.8f);
	boneMat.Ks = glm::vec3(0.8f, 0.8f, 0.8f);
	boneMat.Shininess = 16.0f;

	bone = new GECylinder(2, 10, 0.05f, length / 2);
	bone->initialize(gc, rc);
	bone->setMaterial(boneMat);

	for (size_t i = 0; i < children.size(); i++)
	{
		children[i]->initialize(gc, rc);
	}

	ComputeMatrix();
}
```

**update().** Aquí se colocan la esfera (en `matrix`) y el cilindro (en `matrix` desplazada `length/2`), se actualizan sus uniform buffers y se repite en las hijas.

```cpp
//
// FUNCIÓN: GEBalljoint::update(...)
//
// PROPÓSITO: Coloca las piezas en su posición y actualiza las variables 
//            uniformes de la articulación y de las articulaciones hijas
//
void GEBalljoint::update(GEGraphicsContext* gc, uint32_t index,
	glm::mat4 view, glm::mat4 projection)
{
	joint->setLocation(matrix);
	glm::mat4 mm = glm::translate(matrix, glm::vec3(0.0f, 0.0f, length / 2));
	bone->setLocation(mm);

	joint->update(gc, index, view, projection);
	bone->update(gc, index, view, projection);

	for (size_t i = 0; i < children.size(); i++)
	{
		children[i]->update(gc, index, view, projection);
	}
}
```

**destroy(), addCommands(), setLight(), setBoneMaterial().** Lo mismo que antes + el bucle sobre las hijas. En `destroy` el padre también hace `delete` de las hijas: el padre es su "dueño". `setBoneMaterial` es nuevo: lo usará `GESkeleton::setMaterial`.

```cpp
//
// FUNCIÓN: GEBalljoint::destroy(GEGraphicsContext* gc)
//
// PROPÓSITO: Destruye los buffers de las piezas y las articulaciones hijas
//
void GEBalljoint::destroy(GEGraphicsContext* gc)
{
	for (size_t i = 0; i < children.size(); i++)
	{
		children[i]->destroy(gc);
		delete children[i];
	}
	children.clear();

	joint->destroy(gc);
	bone->destroy(gc);

	delete joint;
	delete bone;
}

//
// FUNCIÓN: GEBalljoint::addCommands()
//
// PROPÓSITO: Añade los comandos de dibujo de la articulación y de sus hijas
//
void GEBalljoint::addCommands(VkCommandBuffer commandBuffer,
	VkPipelineLayout pipelineLayout, int index)
{
	joint->addCommands(commandBuffer, pipelineLayout, index);
	bone->addCommands(commandBuffer, pipelineLayout, index);

	for (size_t i = 0; i < children.size(); i++)
	{
		children[i]->addCommands(commandBuffer, pipelineLayout, index);
	}
}

//
// FUNCIÓN: GEBalljoint::setLight(GELight l)
//
// PROPÓSITO: Asigna las propiedades de la luz
//
void GEBalljoint::setLight(GELight l)
{
	joint->setLight(l);
	bone->setLight(l);

	for (size_t i = 0; i < children.size(); i++)
	{
		children[i]->setLight(l);
	}
}

//
// FUNCIÓN: GEBalljoint::setBoneMaterial(GEMaterial m)
//
// PROPÓSITO: Asigna el material de los huesos (cilindros)
//
void GEBalljoint::setBoneMaterial(GEMaterial m)
{
	bone->setMaterial(m);

	for (size_t i = 0; i < children.size(); i++)
	{
		children[i]->setBoneMaterial(m);
	}
}
```

**Getters y setters de posición y orientación.** Cada setter guarda el valor y llama a `ComputeMatrix()` (como en la práctica 7). `setOrientation` calcula el eje X como `cross(up, dir)`.

```cpp
//
// FUNCIÓN: GEBalljoint::getName()
//
// PROPÓSITO: Obtiene el nombre de la articulación
//
std::string GEBalljoint::getName()
{
	return name;
}

//
// FUNCIÓN: GEBalljoint::getLength()
//
// PROPÓSITO: Obtiene la longitud de la articulación
//
float GEBalljoint::getLength()
{
	return length;
}

//
// FUNCIÓN: GEBalljoint::setLocation(glm::vec3 loc)
//
// PROPÓSITO: Asigna la posición de la articulación (en coordenadas del padre)
//
void GEBalljoint::setLocation(glm::vec3 loc)
{
	location = loc;
	ComputeMatrix();
}

//
// FUNCIÓN: GEBalljoint::getLocation()
//
// PROPÓSITO: Obtiene la posición de la articulación (en coordenadas del padre)
//
glm::vec3 GEBalljoint::getLocation()
{
	return location;
}

//
// FUNCIÓN: GEBalljoint::setOrientation(glm::vec3 nDir, glm::vec3 nUp)
//
// PROPÓSITO: Asigna la orientación de la articulación (con pose 0,0,0).
//
void GEBalljoint::setOrientation(glm::vec3 nDir, glm::vec3 nUp)
{
	dir = nDir;
	up = nUp;
	right = glm::cross(up, dir);
	ComputeMatrix();
}

//
// FUNCIÓN: GEBalljoint::getDirection()
//
// PROPÓSITO: Obtiene el eje Z de la orientación de la articulación
//
glm::vec3 GEBalljoint::getDirection()
{
	return dir;
}

//
// FUNCIÓN: GEBalljoint::getUpDirection()
//
// PROPÓSITO: Obtiene el eje Y de la orientación de la articulación
//
glm::vec3 GEBalljoint::getUpDirection()
{
	return up;
}
```

**Pose y límites.** `setPose` recorta cada ángulo a su rango con `glm::clamp(valor, mínimo, máximo)`: si pides -30º a una rodilla limitada a 0..150, se queda en 0. `setLimits` vuelve a aplicar la pose actual para que respete los nuevos límites.

```cpp
//
// FUNCIÓN: GEBalljoint::setPose()
//
// PROPÓSITO: Asigna la rotación de la articulación, respetando los límites
//            de los grados de libertad
//
void GEBalljoint::setPose(float xrot, float yrot, float zrot)
{
	angles[0] = glm::clamp(xrot, minAngles[0], maxAngles[0]);
	angles[1] = glm::clamp(yrot, minAngles[1], maxAngles[1]);
	angles[2] = glm::clamp(zrot, minAngles[2], maxAngles[2]);
	ComputeMatrix();
}

//
// FUNCIÓN: GEBalljoint::getPose()
//
// PROPÓSITO: Obtiene la rotación de la articulación (ángulos de Euler)
//
glm::vec3 GEBalljoint::getPose()
{
	return glm::vec3(angles[0], angles[1], angles[2]);
}

//
// FUNCIÓN: GEBalljoint::setLimits(...)
//
// PROPÓSITO: Asigna los límites de los grados de libertad (en grados)
//
void GEBalljoint::setLimits(float xmin, float xmax, float ymin, float ymax, float zmin, float zmax)
{
	minAngles[0] = xmin;
	maxAngles[0] = xmax;
	minAngles[1] = ymin;
	maxAngles[1] = ymax;
	minAngles[2] = zmin;
	maxAngles[2] = zmax;
	setPose(angles[0], angles[1], angles[2]);
}

//
// FUNCIÓN: GEBalljoint::getMinLimits()
//
// PROPÓSITO: Obtiene los valores mínimos de los grados de libertad
//
glm::vec3 GEBalljoint::getMinLimits()
{
	return glm::vec3(minAngles[0], minAngles[1], minAngles[2]);
}

//
// FUNCIÓN: GEBalljoint::getMaxLimits()
//
// PROPÓSITO: Obtiene los valores máximos de los grados de libertad
//
glm::vec3 GEBalljoint::getMaxLimits()
{
	return glm::vec3(maxAngles[0], maxAngles[1], maxAngles[2]);
}
```

**El árbol.** `setParentMatrix` lo llama el padre (o el esqueleto) para decirle a la hija dónde está su sistema de coordenadas. `addChild` añade una hija y recalcula para pasársela. `findJoint` busca por nombre recorriendo el subárbol en profundidad: primero se mira a sí misma y luego a cada hija.

```cpp
//
// FUNCIÓN: GEBalljoint::setParentMatrix(glm::mat4 m)
//
// PROPÓSITO: Asigna la matriz de transformación del sistema de coordenadas
//            del padre (extremo de la articulación padre o raíz del esqueleto)
//
void GEBalljoint::setParentMatrix(glm::mat4 m)
{
	parentMatrix = m;
	ComputeMatrix();
}

//
// FUNCIÓN: GEBalljoint::addChild(GEBalljoint* child)
//
// PROPÓSITO: Añade una articulación hija. La articulación padre pasa a ser 
//            responsable de destruirla.
//
void GEBalljoint::addChild(GEBalljoint* child)
{
	children.push_back(child);
	ComputeMatrix();
}

//
// FUNCIÓN: GEBalljoint::getChildren()
//
// PROPÓSITO: Obtiene la lista de articulaciones hijas
//
std::vector<GEBalljoint*> GEBalljoint::getChildren()
{
	return children;
}

//
// FUNCIÓN: GEBalljoint::findJoint(std::string jname)
//
// PROPÓSITO: Busca una articulación por su nombre en el subárbol
//
GEBalljoint* GEBalljoint::findJoint(std::string jname)
{
	if (name == jname) return this;

	for (size_t i = 0; i < children.size(); i++)
	{
		GEBalljoint* found = children[i]->findJoint(jname);
		if (found != nullptr) return found;
	}
	return nullptr;
}
```

### 4.3 Punto de control: un brazo con antebrazo en la escena de la cruz

Antes de construir el esqueleto, prueba el árbol con la escena de la práctica 7. Al compilar ahora dará error en GEScene.cpp porque el constructor ha cambiado (pide nombre); haz estos 3 cambios en **GEScene.cpp**:

1. En el constructor, donde se crea `armL`, dale nombre y añádele un antebrazo ANTES de `initialize`:

```cpp
	armL = new GEBalljoint("armL", 0.4f);
	armL->addChild(new GEBalljoint("forearmL", 0.3f));
	armL->initialize(gc, rc);
```

2. Donde se crea `armR`, solo el nombre:

```cpp
	armR = new GEBalljoint("armR", 0.4f);
```

3. En `update`, después de `armL->setPose(angle, 0.0f, 0.0f);`, mueve también el antebrazo:

```cpp
	armL->findJoint("forearmL")->setPose(0.0f, -angle, 0.0f);
```

No hace falta tocar `destroy`: `armL->destroy(gc)` ya destruye su antebrazo.

Punto de control 4: compila y ejecuta. El brazo izquierdo de la cruz ahora tiene dos tramos (dos esferas rojas seguidas); al girar el brazo, el antebrazo lo acompaña y además se dobla por su cuenta. Si el antebrazo aparece en el centro de la escena en vez de en el extremo del brazo, revisa que `ComputeMatrix` llame a `setParentMatrix` de las hijas.

## Fase 5. Crear GESkeleton

GESkeleton es la raíz del árbol: guarda la posición y orientación del esqueleto en la escena, las articulaciones que cuelgan directamente de la raíz (pelvis, hip\_l, hip\_r) y sabe leer body.skel. Hacia fuera se comporta como un GEFigure.

**Crear los ficheros.** Clic derecho en el proyecto → Agregar → Nuevo elemento → "Archivo de encabezado (.h)" → nombre `GESkeleton.h`. Repite con "Archivo C++ (.cpp)" → `GESkeleton.cpp`. Hazlo siempre así (desde Visual Studio), no creando el fichero a mano en la carpeta: si el .cpp no está en el proyecto, no se compila y aparece el error LNK2019.

**Qué guarda.** Los datos del fichero para el nodo `<skeleton>`: `name`, `position` (el `<offset>`), `dir` (`<zaxis>`), `up` (`<yaxis>`) y `right` (calculado). Además, `location`: una matriz de modelo extra como la de GEFigure, para que `translate`/`rotate` funcionen igual que en cualquier figura. La matriz de la raíz es `location · rootm`, donde `rootm` se construye igual que `jointm` de GEBalljoint.

### 5.1 GESkeleton.h — fichero completo

```cpp
#pragma once

#include "GEGraphicsContext.h"
#include "GERenderingContext.h"
#include "GEBalljoint.h"
#include "GEMaterial.h"
#include "GELight.h"
#include <glm/glm.hpp>
#include <string>
#include <vector>
#include <map>

//
// CLASE: GESkeleton
//
// DESCRIPCIÓN: Clase que describe un esqueleto formado por un árbol de 
//              articulaciones. Tiene los mismos métodos que GEFigure para
//              poder utilizarse como una figura dentro de una escena.
//
class GESkeleton
{
private:
	std::string name;
	glm::mat4 location;
	glm::vec3 position;
	glm::vec3 dir;
	glm::vec3 up;
	glm::vec3 right;
	std::vector<GEBalljoint*> joints;
	void ComputeMatrix();

public:
	GESkeleton(std::string name);
	GESkeleton(int resource);

	// Métodos equivalentes a los de GEFigure
	void initialize(GEGraphicsContext* gc, GERenderingContext* rc);
	void destroy(GEGraphicsContext* gc);
	void addCommands(VkCommandBuffer commandBuffer, VkPipelineLayout pipelineLayout, int index);
	void update(GEGraphicsContext* gc, uint32_t index, glm::mat4 view, glm::mat4 projection);
	void resetLocation();
	void setLocation(glm::mat4 m);
	void translate(glm::vec3 t);
	void rotate(float angle, glm::vec3 axis);
	void setMaterial(GEMaterial m);
	void setLight(GELight l);

	// Getters y setters del esqueleto
	std::string getName();
	void setPosition(glm::vec3 pos);
	glm::vec3 getPosition();
	void setOrientation(glm::vec3 nDir, glm::vec3 nUp);
	glm::vec3 getDirection();
	glm::vec3 getUpDirection();
	void addJoint(GEBalljoint* joint);
	std::vector<GEBalljoint*> getJoints();
	GEBalljoint* getJoint(std::string jname);
	void setPose(std::string jname, float xrot, float yrot, float zrot);

private:
	// Lectura del fichero de descripción del esqueleto
	void parse(std::string text);
	bool readTag(std::string& text, size_t& pos, std::string& tag, std::map<std::string, std::string>& attrs, bool& closing, bool& empty);
	float getAttribute(std::map<std::string, std::string>& attrs, std::string key, float def);
	glm::vec3 getVector(std::map<std::string, std::string>& attrs);
	std::string getTextFromResource(int resource);
};
```

### 5.2 GESkeleton.cpp, parte A: constructores y matriz de la raíz

Dos constructores: uno vacío (para montar un esqueleto a mano con `addJoint`) y otro que lee el recurso y llama a `parse`. `ComputeMatrix` hace con la raíz lo que GEBalljoint hace con una articulación y pasa la matriz a las articulaciones raíz (la raíz no tiene longitud, así que no hay "extremo").

```cpp
#include "GESkeleton.h"

#include "GEBalljoint.h"
#include "GEMaterial.h"
#include "GELight.h"
#include <windows.h>
#include <glm/glm.hpp>
#include <glm/gtc/matrix_transform.hpp>
#include <stdexcept>
#include <cctype>

//
// FUNCIÓN: GESkeleton::GESkeleton(std::string n)
//
// PROPÓSITO: Construye un esqueleto vacío, situado en el origen y con la 
//            orientación del sistema de coordenadas de la escena
//
GESkeleton::GESkeleton(std::string n)
{
	name = n;
	location = glm::mat4(1.0f);
	position = glm::vec3(0.0f, 0.0f, 0.0f);
	dir = glm::vec3(0.0f, 0.0f, 1.0f);
	up = glm::vec3(0.0f, 1.0f, 0.0f);
	right = glm::vec3(1.0f, 0.0f, 0.0f);
}

//
// FUNCIÓN: GESkeleton::GESkeleton(int resource)
//
// PROPÓSITO: Construye un esqueleto a partir de un fichero de descripción
//            (formato .skel) incluido como recurso de la aplicación
//
GESkeleton::GESkeleton(int resource)
{
	name = "";
	location = glm::mat4(1.0f);
	position = glm::vec3(0.0f, 0.0f, 0.0f);
	dir = glm::vec3(0.0f, 0.0f, 1.0f);
	up = glm::vec3(0.0f, 1.0f, 0.0f);
	right = glm::vec3(1.0f, 0.0f, 0.0f);

	std::string text = getTextFromResource(resource);
	parse(text);
}

//
// FUNCIÓN: GESkeleton::ComputeMatrix()
//
// PROPÓSITO: Calcula la matriz de la raíz del esqueleto y la propaga a las
//            articulaciones que dependen de ella
//
void GESkeleton::ComputeMatrix()
{
	// Formato glm::mat4[column][row]
	glm::mat4 rootm;
	rootm[0][0] = right.x;
	rootm[0][1] = right.y;
	rootm[0][2] = right.z;
	rootm[0][3] = 0.0;

	rootm[1][0] = up.x;
	rootm[1][1] = up.y;
	rootm[1][2] = up.z;
	rootm[1][3] = 0.0;

	rootm[2][0] = dir.x;
	rootm[2][1] = dir.y;
	rootm[2][2] = dir.z;
	rootm[2][3] = 0.0;

	rootm[3][0] = position.x;
	rootm[3][1] = position.y;
	rootm[3][2] = position.z;
	rootm[3][3] = 1.0;

	glm::mat4 matrix = location * rootm;

	for (size_t i = 0; i < joints.size(); i++)
	{
		joints[i]->setParentMatrix(matrix);
	}
}
```

### 5.3 GESkeleton.cpp, parte B: los métodos de GEFigure

Cada uno simplemente reenvía la llamada a las articulaciones raíz, y estas la repiten en sus hijas (lo programaste en la fase 4). `translate`, `rotate`, `setLocation` y `resetLocation` modifican `location` igual que en GEFigure.cpp (ábrelo y compara) y recalculan.

```cpp
//
// FUNCIÓN: GESkeleton::initialize(GEGraphicsContext* gc, GERenderingContext* rc)
//
// PROPÓSITO: Crea las piezas de todas las articulaciones del esqueleto
//
void GESkeleton::initialize(GEGraphicsContext* gc, GERenderingContext* rc)
{
	for (size_t i = 0; i < joints.size(); i++)
	{
		joints[i]->initialize(gc, rc);
	}
	ComputeMatrix();
}

//
// FUNCIÓN: GESkeleton::destroy(GEGraphicsContext* gc)
//
// PROPÓSITO: Libera los buffers y las articulaciones del esqueleto
//
void GESkeleton::destroy(GEGraphicsContext* gc)
{
	for (size_t i = 0; i < joints.size(); i++)
	{
		joints[i]->destroy(gc);
		delete joints[i];
	}
	joints.clear();
}

//
// FUNCIÓN: GESkeleton::addCommands(VkCommandBuffer commandBuffer, VkPipelineLayout pipelineLayout, int index)
//
// PROPÓSITO: Añade los comandos de renderizado de todas las articulaciones
//
void GESkeleton::addCommands(VkCommandBuffer commandBuffer, VkPipelineLayout pipelineLayout, int index)
{
	for (size_t i = 0; i < joints.size(); i++)
	{
		joints[i]->addCommands(commandBuffer, pipelineLayout, index);
	}
}

//
// FUNCIÓN: GESkeleton::update(GEGraphicsContext* gc, uint32_t index, glm::mat4 view, glm::mat4 projection)
//
// PROPÓSITO: Actualiza las variables uniformes de todas las articulaciones
//
void GESkeleton::update(GEGraphicsContext* gc, uint32_t index, glm::mat4 view, glm::mat4 projection)
{
	for (size_t i = 0; i < joints.size(); i++)
	{
		joints[i]->update(gc, index, view, projection);
	}
}

//
// FUNCIÓN: GESkeleton::resetLocation()
//
// PROPÓSITO: Resetea la matriz de localización (Model).
//
void GESkeleton::resetLocation()
{
	location = glm::mat4(1.0f);
	ComputeMatrix();
}

//
// FUNCIÓN: GESkeleton::setLocation(glm::mat4 m)
//
// PROPÓSITO: Asigna la matriz de localización (Model).
//
void GESkeleton::setLocation(glm::mat4 m)
{
	location = glm::mat4(m);
	ComputeMatrix();
}

//
// FUNCIÓN: GESkeleton::translate(glm::vec3 t)
//
// PROPÓSITO: Añade un desplazamiento a la matriz de localización (Model).
//
void GESkeleton::translate(glm::vec3 t)
{
	location = glm::translate(location, t);
	ComputeMatrix();
}

//
// FUNCIÓN: GESkeleton::rotate(float angle, glm::vec3 axis)
//
// PROPÓSITO: Añade una rotación a la matriz de localización (Model).
//
void GESkeleton::rotate(float angle, glm::vec3 axis)
{
	location = glm::rotate(location, glm::radians(angle), axis);
	ComputeMatrix();
}

//
// FUNCIÓN: GESkeleton::setMaterial(GEMaterial m)
//
// PROPÓSITO: Asigna el material de los huesos del esqueleto
//
void GESkeleton::setMaterial(GEMaterial m)
{
	for (size_t i = 0; i < joints.size(); i++)
	{
		joints[i]->setBoneMaterial(m);
	}
}

//
// FUNCIÓN: GESkeleton::setLight(GELight l)
//
// PROPÓSITO: Asigna las propiedades de la luz que ilumina el esqueleto
//
void GESkeleton::setLight(GELight l)
{
	for (size_t i = 0; i < joints.size(); i++)
	{
		joints[i]->setLight(l);
	}
}
```

### 5.4 GESkeleton.cpp, parte C: getters y setters

`setPosition`/`setOrientation` son los que pide el enunciado ("asignar la posición del esqueleto, su orientación") y los usará la animación para desplazar y girar el cuerpo entero. `getJoint` busca en las tres ramas con el `findJoint` de la fase 4. `setPose(nombre, x, y, z)` es un atajo: busca la articulación y le pone la pose.

```cpp
//
// FUNCIÓN: GESkeleton::getName()
//
// PROPÓSITO: Obtiene el nombre del esqueleto
//
std::string GESkeleton::getName()
{
	return name;
}

//
// FUNCIÓN: GESkeleton::setPosition(glm::vec3 pos)
//
// PROPÓSITO: Asigna la posición del esqueleto (en coordenadas de la escena)
//
void GESkeleton::setPosition(glm::vec3 pos)
{
	position = pos;
	ComputeMatrix();
}

//
// FUNCIÓN: GESkeleton::getPosition()
//
// PROPÓSITO: Obtiene la posición del esqueleto
//
glm::vec3 GESkeleton::getPosition()
{
	return position;
}

//
// FUNCIÓN: GESkeleton::setOrientation(glm::vec3 nDir, glm::vec3 nUp)
//
// PROPÓSITO: Asigna la orientación del esqueleto (ejes Z e Y)
//
void GESkeleton::setOrientation(glm::vec3 nDir, glm::vec3 nUp)
{
	dir = nDir;
	up = nUp;
	right = glm::cross(up, dir);
	ComputeMatrix();
}

//
// FUNCIÓN: GESkeleton::getDirection()
//
// PROPÓSITO: Obtiene el eje Z de la orientación del esqueleto
//
glm::vec3 GESkeleton::getDirection()
{
	return dir;
}

//
// FUNCIÓN: GESkeleton::getUpDirection()
//
// PROPÓSITO: Obtiene el eje Y de la orientación del esqueleto
//
glm::vec3 GESkeleton::getUpDirection()
{
	return up;
}

//
// FUNCIÓN: GESkeleton::addJoint(GEBalljoint* joint)
//
// PROPÓSITO: Añade una articulación dependiente de la raíz del esqueleto.
//            El esqueleto pasa a ser responsable de destruirla.
//
void GESkeleton::addJoint(GEBalljoint* joint)
{
	joints.push_back(joint);
	ComputeMatrix();
}

//
// FUNCIÓN: GESkeleton::getJoints()
//
// PROPÓSITO: Obtiene las articulaciones dependientes de la raíz
//
std::vector<GEBalljoint*> GESkeleton::getJoints()
{
	return joints;
}

//
// FUNCIÓN: GESkeleton::getJoint(std::string jname)
//
// PROPÓSITO: Busca una articulación del esqueleto por su nombre
//
GEBalljoint* GESkeleton::getJoint(std::string jname)
{
	for (size_t i = 0; i < joints.size(); i++)
	{
		GEBalljoint* found = joints[i]->findJoint(jname);
		if (found != nullptr) return found;
	}
	return nullptr;
}

//
// FUNCIÓN: GESkeleton::setPose(std::string jname, float xrot, float yrot, float zrot)
//
// PROPÓSITO: Asigna la pose (ángulos de Euler) de una articulación
//
void GESkeleton::setPose(std::string jname, float xrot, float yrot, float zrot)
{
	GEBalljoint* joint = getJoint(jname);
	if (joint != nullptr) joint->setPose(xrot, yrot, zrot);
}
```

### 5.5 GESkeleton.cpp, parte D: leer body.skel

Esta es la parte más nueva. Se divide en tres funciones pequeñas.

**readTag: leer una etiqueta.** Busca el siguiente `<`, se salta comentarios `<!-- -->` y declaraciones `<? ?>`, y devuelve el nombre de la etiqueta, sus atributos y dos marcas. Ejemplo con la línea `<offset x="0.05" y="-0.05" z="0.0" />`:

| Salida | Valor |
| --- | --- |
| tag | offset |
| attrs | {x: "0.05", y: "-0.05", z: "0.0"} |
| closing (¿es `</...>`?) | false |
| empty (¿acaba en `/>`?) | true |

Con `</balljoint>` devuelve tag = balljoint y closing = true. `pos` avanza hasta después del `>`, así que la siguiente llamada lee la siguiente etiqueta; cuando no quedan, devuelve false.

**parse: montar el árbol con una pila.** El fichero es un árbol escrito en orden: cada `<balljoint>` se abre, dentro vienen sus datos y sus hijas, y luego se cierra. La pila guarda las articulaciones abiertas; la de arriba es "la actual".

- `<balljoint ...>`: crea la articulación. Si la pila está vacía, cuelga de la raíz (`addJoint`); si no, es hija de la de arriba (`addChild`). Después se apila (salvo que sea `<balljoint ... />`, que no tiene contenido).
- `</balljoint>`: se desapila; volvemos al padre.
- `<offset>`, `<zaxis>`, `<yaxis>`, `<limit>`: se aplican a la de arriba de la pila, o al esqueleto si la pila está vacía.

Con `<zaxis>` se llama a `setOrientation(nuevo Z, Y actual)` y con `<yaxis>` a `setOrientation(Z actual, nuevo Y)`; tras leer las dos, el eje X calculado es el correcto, venga primero la que venga.

Recorrido real de body.skel: `<skeleton>` (pila vacía: datos del esqueleto) → `<balljoint pelvis>` (cuelga de la raíz, pila \[pelvis\]) → `<balljoint spine>` (hija de pelvis, pila \[pelvis, spine\]) → `<balljoint neck>` (hija de spine) → `</balljoint>` (pila \[pelvis, spine\]) → `<balljoint clavicle_l>` (hija de spine)… y así hasta `</skeleton>`.

**getTextFromResource.** Es una copia de `GERenderingContext::getFileFromResource`, pero devuelve un `std::string`. Si el recurso no existe lanza una excepción con un mensaje claro.

```cpp
//
// FUNCIÓN: GESkeleton::parse(std::string text)
//
// PROPÓSITO: Construye el esqueleto a partir del texto del fichero de 
//            descripción. Se recorren las etiquetas en orden, utilizando
//            una pila con las articulaciones abiertas. Las etiquetas offset,
//            zaxis, yaxis y limit se aplican al elemento abierto en ese 
//            momento (la articulación de la cima de la pila o, si la pila
//            está vacía, el propio esqueleto).
//
void GESkeleton::parse(std::string text)
{
	std::vector<GEBalljoint*> stack;
	std::map<std::string, std::string> attrs;
	std::string tag;
	bool closing, empty;
	size_t pos = 0;

	while (readTag(text, pos, tag, attrs, closing, empty))
	{
		if (tag == "skeleton")
		{
			if (!closing) name = attrs["name"];
		}
		else if (tag == "balljoint")
		{
			if (closing)
			{
				if (!stack.empty()) stack.pop_back();
			}
			else
			{
				GEBalljoint* joint = new GEBalljoint(attrs["name"], getAttribute(attrs, "length", 0.0f));
				if (stack.empty()) addJoint(joint);
				else stack.back()->addChild(joint);
				if (!empty) stack.push_back(joint);
			}
		}
		else if (tag == "offset")
		{
			glm::vec3 v = getVector(attrs);
			if (stack.empty()) setPosition(v);
			else stack.back()->setLocation(v);
		}
		else if (tag == "zaxis")
		{
			glm::vec3 v = getVector(attrs);
			if (stack.empty()) setOrientation(v, up);
			else stack.back()->setOrientation(v, stack.back()->getUpDirection());
		}
		else if (tag == "yaxis")
		{
			glm::vec3 v = getVector(attrs);
			if (stack.empty()) setOrientation(dir, v);
			else stack.back()->setOrientation(stack.back()->getDirection(), v);
		}
		else if (tag == "limit")
		{
			if (!stack.empty())
			{
				stack.back()->setLimits(
					getAttribute(attrs, "xmin", -180.0f), getAttribute(attrs, "xmax", 180.0f),
					getAttribute(attrs, "ymin", -180.0f), getAttribute(attrs, "ymax", 180.0f),
					getAttribute(attrs, "zmin", -180.0f), getAttribute(attrs, "zmax", 180.0f));
			}
		}
	}
}

//
// FUNCIÓN: GESkeleton::readTag(...)
//
// PROPÓSITO: Lee la siguiente etiqueta del texto a partir de la posición pos.
//            Devuelve el nombre de la etiqueta, sus atributos, si es una 
//            etiqueta de cierre (</tag>) y si es una etiqueta vacía (<tag />).
//            Ignora los comentarios (<!-- -->) y las declaraciones (<? ?>).
//            Devuelve false cuando no quedan más etiquetas.
//
bool GESkeleton::readTag(std::string& text, size_t& pos, std::string& tag, std::map<std::string, std::string>& attrs, bool& closing, bool& empty)
{
	while (true)
	{
		size_t start = text.find('<', pos);
		if (start == std::string::npos) return false;

		if (text.compare(start, 4, "<!--") == 0)
		{
			size_t end = text.find("-->", start);
			if (end == std::string::npos) return false;
			pos = end + 3;
			continue;
		}
		if (text.compare(start, 2, "<?") == 0)
		{
			size_t end = text.find("?>", start);
			if (end == std::string::npos) return false;
			pos = end + 2;
			continue;
		}

		size_t end = text.find('>', start);
		if (end == std::string::npos) return false;
		std::string content = text.substr(start + 1, end - start - 1);
		pos = end + 1;

		tag.clear();
		attrs.clear();
		closing = false;
		empty = false;

		// Elimina los espacios finales
		while (!content.empty() && isspace((unsigned char)content.back())) content.pop_back();

		if (!content.empty() && content[0] == '/')
		{
			closing = true;
			content = content.substr(1);
		}
		if (!content.empty() && content.back() == '/')
		{
			empty = true;
			content.pop_back();
		}

		size_t i = 0;
		while (i < content.size() && !isspace((unsigned char)content[i])) i++;
		tag = content.substr(0, i);

		// Atributos de la forma nombre="valor"
		while (i < content.size())
		{
			while (i < content.size() && isspace((unsigned char)content[i])) i++;
			if (i >= content.size()) break;

			size_t eq = content.find('=', i);
			if (eq == std::string::npos) break;
			std::string key = content.substr(i, eq - i);
			while (!key.empty() && isspace((unsigned char)key.back())) key.pop_back();

			size_t q1 = content.find_first_of("\"'", eq);
			if (q1 == std::string::npos) break;
			size_t q2 = content.find(content[q1], q1 + 1);
			if (q2 == std::string::npos) break;

			attrs[key] = content.substr(q1 + 1, q2 - q1 - 1);
			i = q2 + 1;
		}
		return true;
	}
}

//
// FUNCIÓN: GESkeleton::getAttribute(...)
//
// PROPÓSITO: Obtiene el valor numérico de un atributo (o el valor por defecto
//            si el atributo no existe)
//
float GESkeleton::getAttribute(std::map<std::string, std::string>& attrs, std::string key, float def)
{
	if (attrs.find(key) == attrs.end()) return def;
	return std::stof(attrs[key]);
}

//
// FUNCIÓN: GESkeleton::getVector(...)
//
// PROPÓSITO: Obtiene el vector descrito por los atributos x, y, z
//
glm::vec3 GESkeleton::getVector(std::map<std::string, std::string>& attrs)
{
	float x = getAttribute(attrs, "x", 0.0f);
	float y = getAttribute(attrs, "y", 0.0f);
	float z = getAttribute(attrs, "z", 0.0f);
	return glm::vec3(x, y, z);
}

//
// FUNCIÓN: GESkeleton::getTextFromResource(int resource)
//
// PROPÓSITO: Extrae el contenido de un fichero de texto incluido como 
//            recurso de la aplicación
//
std::string GESkeleton::getTextFromResource(int resource)
{
	HRSRC handle = FindResource(NULL, MAKEINTRESOURCE(resource), RT_HTML);
	if (handle == NULL)
	{
		throw std::runtime_error("No se encuentra el fichero de descripcion del esqueleto");
	}
	HGLOBAL global = LoadResource(NULL, handle);
	const char* ptr = static_cast<const char*>(LockResource(global));
	DWORD size = SizeofResource(NULL, handle);

	std::string text(ptr, size);
	UnlockResource(global);
	FreeResource(global);
	return text;
}
```

### 5.6 Punto de control: el esqueleto quieto en la escena

Ahora sustituye la cruz por el esqueleto en GEScene (todavía sin animación).

**GEScene.h:** cambia `#include "GEBalljoint.h"` por `#include "GESkeleton.h"` y deja la parte `private` de la clase así (fuera `column`, `cross`, `armL`, `armR` y `angle`):

```cpp
private:
	GERenderingContext* rc;
	GEFigure* ground;
	GESkeleton* skeleton;
	GECamera* camera;
	glm::mat4 projection;
```

**GEScene.cpp:**

1. En los `#include`, cambia `#include "GEBalljoint.h"` por `#include "GESkeleton.h"`.
2. En el constructor, acerca la cámara: `this->camera->setPosition(glm::vec3(0.0f, 1.0f, 6.0f));`
3. Borra todo lo de `column`, `cross`, `armL`, `armR` y `angle = 0.0;`, y en su lugar (después de crear el suelo) pon:

```cpp
	skeleton = new GESkeleton(IDR_HTML3);
	skeleton->initialize(gc, rc);
	skeleton->setLight(light);

	// Pose de ejemplo del enunciado
	skeleton->setPose("shoulder_l", 0.0f, -45.0f, 0.0f);
	skeleton->setPose("elbow_l", -45.0f, 0.0f, 0.0f);
	skeleton->setPose("shoulder_r", -45.0f, -30.0f, -150.0f);
	skeleton->setPose("elbow_r", -90.0f, 0.0f, 0.0f);
	skeleton->setPose("leg_l", -45.0f, 0.0f, 0.0f);
	skeleton->setPose("knee_l", 30.0f, 0.0f, 0.0f);
```

El orden importa: primero `initialize` (crea esferas y cilindros) y después `setLight` (si no, `setLight` usa punteros nulos y el programa se cierra).

4. `destroy` queda así:

```cpp
void GEScene::destroy(GEGraphicsContext* gc)
{
	rc->destroy(gc);
	delete rc;

	ground->destroy(gc);
	skeleton->destroy(gc);

	delete ground;
	delete skeleton;
	delete camera;
}
```

5. En `fillCommandBuffers`, dentro del bucle, deja solo el suelo y el esqueleto:

```cpp
		ground->addCommands(commandBuffers[i], rc->pipelineLayout, i);
		skeleton->addCommands(commandBuffers[i], rc->pipelineLayout, i);
```

6. `update` queda así:

```cpp
void GEScene::update(GEGraphicsContext* gc, uint32_t index)
{
	camera->update();
	glm::mat4 view = camera->getViewMatrix();

	ground->update(gc, index, view, projection);
	skeleton->update(gc, index, view, projection);
}
```

Punto de control 5a: comenta las 6 líneas de `setPose`, compila y ejecuta. Debe verse el esqueleto en reposo con los brazos en cruz, igual que la primera imagen del enunciado (vista desde otro ángulo).

Punto de control 5b: descomenta las 6 líneas. Debe verse la pose de ejemplo del enunciado: brazo izquierdo (a la derecha de la pantalla) estirado con el antebrazo hacia arriba, brazo derecho doblado junto a la cabeza y pierna izquierda levantada hacia delante con la rodilla doblada. Si el esqueleto sale desmontado (huesos que no se tocan), el fallo está en el `endMatrix` de `ComputeMatrix`; si no sale nada, revisa el recurso (fase 3) y que el esqueleto esté en `fillCommandBuffers`.

Referencia (capturas del programa ya hecho): a la izquierda, el punto de control 4 (cruz con brazo y antebrazo); a la derecha, el 5b (pose de ejemplo del enunciado con la cámara en (0, 1, 6)).

&#91;image: Puntos de control 4 y 5b\]

## Fase 6. Crear GEAnimation

GEAnimation guarda unas pocas poses completas en instantes concretos (keyframes) y, para cualquier instante intermedio, calcula la pose mezclando las dos que lo rodean en línea recta (interpolación lineal).

**La idea con números.** Supongamos que la rodilla derecha vale 5º en K0 (t = 0,0 s) y 75º en K1 (t = 0,8 s). En t = 0,2 s:

```latex
\alpha = \frac{t - t_k}{t_{k+1} - t_k} = \frac{0.2 - 0.0}{0.8 - 0.0} = 0.25
```

```latex
v(t) = v_k + \alpha \,(v_{k+1} - v_k) = 5 + 0.25 \cdot (75 - 5) = 22.5^{\circ}
```

Con alfa = 0 estás en K0, con alfa = 1 en K1. Lo mismo se hace con los 3 ángulos de las 19 articulaciones y, además, con la posición y el giro del esqueleto entero (para que el cuerpo pueda agacharse, desplazarse y girar).

**Cómo se guardan los datos.** Una lista de nombres de articulaciones (`jointNames`) y, por cada keyframe: su instante (`times`), la posición del esqueleto (`positions`), su giro alrededor del eje Y en grados (`rotations`) y un vector con un `glm::vec3` de ángulos por articulación (`poses[keyframe][articulación]`). Crea `GEAnimation.h` y `GEAnimation.cpp` como en la fase 5 (Agregar → Nuevo elemento).

### 6.1 GEAnimation.h — fichero completo

```cpp
#pragma once

#include "GESkeleton.h"
#include <glm/glm.hpp>
#include <string>
#include <vector>

//
// CLASE: GEAnimation
//
// DESCRIPCIÓN: Clase que genera la animación de un esqueleto realizando un
//              lanzamiento de peso. La animación se describe mediante una 
//              lista de poses clave (keyframes). Cada keyframe almacena el
//              instante de tiempo, la posición y el giro (eje Y) del 
//              esqueleto y los ángulos de Euler de cada articulación. La pose
//              instantánea se calcula mediante interpolación lineal entre
//              los dos keyframes que rodean al instante solicitado.
//
class GEAnimation
{
private:
	float duration;
	std::vector<std::string> jointNames;
	std::vector<float> times;
	std::vector<glm::vec3> positions;
	std::vector<float> rotations;
	std::vector<std::vector<glm::vec3>> poses;

public:
	GEAnimation();
	float getDuration();
	int getKeyframeCount();
	float getKeyframeTime(int k);
	void animate(GESkeleton* skeleton, float time);

private:
	void createKeyframes();
	int addKeyframe(float time, glm::vec3 position, float rotation);
	void setKeyframePose(int k, std::string joint, float xrot, float yrot, float zrot);
	int getJointIndex(std::string joint);
	int getSegment(float time);
};
```

### 6.2 GEAnimation.cpp, parte A: constructor y herramientas para escribir keyframes

El constructor rellena la lista de las 19 articulaciones (los mismos nombres de body.skel), crea los keyframes y toma como duración el instante del último (5,0 s). `addKeyframe` añade un keyframe con todas las articulaciones a (0,0,0) y devuelve su número; `setKeyframePose` cambia una articulación de ese keyframe. Así, en cada keyframe solo escribes las articulaciones que no están en reposo.

```cpp
#include "GEAnimation.h"

#include "GESkeleton.h"
#include <glm/glm.hpp>
#include <cmath>

//
// FUNCIÓN: GEAnimation::GEAnimation()
//
// PROPÓSITO: Construye la animación del lanzamiento de peso
//
// COMENTARIOS:
//     La lista de articulaciones coincide con las descritas en body.skel.
//     Las poses de cada keyframe toman por defecto el valor (0,0,0).
//
GEAnimation::GEAnimation()
{
	jointNames = {
		"pelvis", "spine", "neck",
		"clavicle_l", "shoulder_l", "elbow_l", "wrist_l",
		"clavicle_r", "shoulder_r", "elbow_r", "wrist_r",
		"hip_l", "leg_l", "knee_l", "ankle_l",
		"hip_r", "leg_r", "knee_r", "ankle_r"
	};

	createKeyframes();
	duration = times.back();
}

//
// FUNCIÓN: GEAnimation::addKeyframe(float time, glm::vec3 position, float rotation)
//
// PROPÓSITO: Añade un keyframe con todas las articulaciones en pose (0,0,0).
//            Devuelve el índice del keyframe.
//
int GEAnimation::addKeyframe(float time, glm::vec3 position, float rotation)
{
	times.push_back(time);
	positions.push_back(position);
	rotations.push_back(rotation);
	poses.push_back(std::vector<glm::vec3>(jointNames.size(), glm::vec3(0.0f, 0.0f, 0.0f)));
	return (int)times.size() - 1;
}

//
// FUNCIÓN: GEAnimation::setKeyframePose(int k, std::string joint, float xrot, float yrot, float zrot)
//
// PROPÓSITO: Asigna los ángulos de Euler de una articulación en un keyframe
//
void GEAnimation::setKeyframePose(int k, std::string joint, float xrot, float yrot, float zrot)
{
	int j = getJointIndex(joint);
	if (j < 0) return;
	poses[k][j] = glm::vec3(xrot, yrot, zrot);
}

//
// FUNCIÓN: GEAnimation::getJointIndex(std::string joint)
//
// PROPÓSITO: Obtiene la posición de una articulación en la lista (-1 si no existe)
//
int GEAnimation::getJointIndex(std::string joint)
{
	for (size_t j = 0; j < jointNames.size(); j++)
	{
		if (jointNames[j] == joint) return (int)j;
	}
	return -1;
}

//
// FUNCIÓN: GEAnimation::getDuration()
//
// PROPÓSITO: Obtiene la duración de la animación (en segundos)
//
float GEAnimation::getDuration()
{
	return duration;
}

//
// FUNCIÓN: GEAnimation::getKeyframeCount()
//
// PROPÓSITO: Obtiene el número de keyframes
//
int GEAnimation::getKeyframeCount()
{
	return (int)times.size();
}

//
// FUNCIÓN: GEAnimation::getKeyframeTime(int k)
//
// PROPÓSITO: Obtiene el instante de tiempo de un keyframe
//
float GEAnimation::getKeyframeTime(int k)
{
	return times[k];
}
```

### 6.3 GEAnimation.cpp, parte B: calcular la pose de un instante

`getSegment(t)` busca el tramo: el índice k tal que `times[k] <= t < times[k+1]`. `animate(skeleton, t)` hace todo el trabajo:

1. `fmod(time, duration)` convierte el tiempo real (que crece sin parar) en un tiempo dentro del ciclo de 5 s: en t = 7,3 s se ve lo mismo que en t = 2,3 s.
2. Busca el tramo k y calcula alfa.
3. Para cada articulación, interpola los tres ángulos (con `glm::vec3` la fórmula se aplica a las tres componentes a la vez) y llama a `skeleton->setPose`.
4. Interpola la posición y el giro del esqueleto. El giro ψ se convierte en el eje Z del esqueleto: (sin ψ, 0, cos ψ). Con ψ = 0 mira a +Z (hacia la cámara inicial), con 90º mira a +X.

```cpp
//
// FUNCIÓN: GEAnimation::getSegment(float time)
//
// PROPÓSITO: Obtiene el índice k del keyframe tal que 
//            times[k] <= time < times[k+1]
//
int GEAnimation::getSegment(float time)
{
	int k = 0;
	while (k < (int)times.size() - 2 && time >= times[k + 1]) k++;
	return k;
}

//
// FUNCIÓN: GEAnimation::animate(GESkeleton* skeleton, float time)
//
// PROPÓSITO: Calcula la pose instantánea del esqueleto en el instante time
//            (en segundos) y la asigna al esqueleto. La animación es cíclica.
//
// COMENTARIOS:
//     Para cada valor (ángulos de cada articulación, posición y giro del 
//     esqueleto) se realiza una interpolación lineal entre los keyframes k 
//     y k+1:  v(t) = v[k] + alpha * (v[k+1] - v[k]),
//     siendo alpha = (t - t[k]) / (t[k+1] - t[k]).
//
void GEAnimation::animate(GESkeleton* skeleton, float time)
{
	float t = (float)fmod(time, duration);
	if (t < 0.0f) t += duration;

	int k = getSegment(t);
	float alpha = (t - times[k]) / (times[k + 1] - times[k]);
	if (alpha < 0.0f) alpha = 0.0f;
	if (alpha > 1.0f) alpha = 1.0f;

	for (size_t j = 0; j < jointNames.size(); j++)
	{
		glm::vec3 angles = poses[k][j] + alpha * (poses[k + 1][j] - poses[k][j]);
		skeleton->setPose(jointNames[j], angles.x, angles.y, angles.z);
	}

	glm::vec3 position = positions[k] + alpha * (positions[k + 1] - positions[k]);
	float rotation = rotations[k] + alpha * (rotations[k + 1] - rotations[k]);
	float rad = glm::radians(rotation);

	skeleton->setPosition(position);
	skeleton->setOrientation(glm::vec3(sin(rad), 0.0f, cos(rad)), glm::vec3(0.0f, 1.0f, 0.0f));
}
```

### 6.4 Qué hace cada eje en cada articulación

Antes de escribir poses necesitas saber hacia dónde gira cada ángulo. Con body.skel y el esqueleto mirando a la cámara (su izquierda queda a la derecha de la pantalla):

| Articulación | X positivo | X negativo | Y | Z |
| --- | --- | --- | --- | --- |
| pelvis, spine | Tronco hacia atrás | Tronco hacia delante (agacharse) | Inclinación lateral (Y+ hacia la derecha del cuerpo) | Gira el tronco (Z+ hacia la izquierda del cuerpo) |
| neck | Cabeza hacia atrás | Cabeza hacia delante | Lateral | Gira la cabeza |
| shoulder\_l | Baja el brazo (90 = pegado al cuerpo) | Sube el brazo | Y− lleva el brazo adelante, Y+ atrás | Gira el brazo sobre sí mismo |
| shoulder\_r | Baja el brazo | Sube el brazo | **Al revés**: Y+ adelante, Y− atrás | Gira el brazo sobre sí mismo |
| elbow\_l / elbow\_r | Dobla en el plano arriba/abajo | Dobla en el otro sentido | Dobla en el plano delante/detrás (mismos signos que su hombro) | Giro del antebrazo |
| leg\_l / leg\_r | Pierna hacia atrás | Pierna hacia delante | Y+ hacia la izquierda del cuerpo (abre la izquierda, cierra la derecha) | Giro de la pierna |
| knee\_l / knee\_r | Dobla la rodilla (positivo, como una rodilla real) | No natural | — | — |
| ankle\_l / ankle\_r | Pie de puntillas | Punta del pie hacia arriba | Lateral | Lateral |

Truco para comprobarlo tú misma: en la escena estática de la fase 5 pon una sola articulación con un solo ángulo (por ejemplo `skeleton->setPose("leg_r", -45.0f, 0.0f, 0.0f);`), ejecuta y mira. Así puedes diseñar cada pose antes de meterla en la animación.

El orden de Euler es el de la práctica 7 (Rz·Ry·Rx): primero se aplica X y después Y sobre el resultado. Por eso en el hombro `(-40, 80, 0)` significa "sube el brazo 40º y luego llévalo 80º hacia delante": brazo estirado hacia delante y arriba.

### 6.5 Las poses del lanzamiento de peso

El lanzamiento va hacia +X de la escena (derecha de la pantalla) con la mano derecha. Fases: el atleta empieza de espaldas, se agacha, se impulsa, queda en posición de fuerza, gira, empuja, suelta y cambia de piernas para no salir del círculo; luego vuelve al inicio para que el ciclo no tenga saltos.

Las posiciones (x, y, z) del esqueleto se ajustaron para que los pies queden sobre el suelo: al doblar rodillas el cuerpo debe bajar (y < 1,0) y al ponerse de puntillas subir (y > 1,0). Si diseñas tus propias poses y un pie se hunde o flota, sube o baja la y de ese keyframe.

Puedes usar estas poses tal cual o cambiarlas. Añade este método al final de GEAnimation.cpp:

```cpp
//
// FUNCIÓN: GEAnimation::createKeyframes()
//
// PROPÓSITO: Define las poses clave del lanzamiento de peso (5 segundos).
//
// COMENTARIOS:
//     El lanzamiento se realiza en la dirección +X de la escena. El giro 
//     del esqueleto se mide alrededor del eje Y: -90º significa que el 
//     esqueleto está de espaldas a la dirección de lanzamiento y 90º que 
//     está de frente a ella. La posición del esqueleto se ha calculado para 
//     que los pies se apoyen sobre el suelo.
//     Ángulos en grados (rotaciones X, Y, Z de los ángulos de Euler).
//
void GEAnimation::createKeyframes()
{
	int k;

	// K0 (0.0 s): Preparación
	//   De espaldas a la dirección de lanzamiento, peso en la pierna derecha,
	//   bola junto al cuello y brazo izquierdo adelantado
	k = addKeyframe(0.0f, glm::vec3(-0.400f, 1.061f, 0.000f), -90.0f);
	setKeyframePose(k, "shoulder_l", 20.0f, -70.0f, 0.0f);
	setKeyframePose(k, "elbow_l", 0.0f, -20.0f, 0.0f);
	setKeyframePose(k, "shoulder_r", 10.0f, 25.0f, 0.0f);
	setKeyframePose(k, "elbow_r", 0.0f, 140.0f, 0.0f);
	setKeyframePose(k, "wrist_r", 0.0f, 30.0f, 0.0f);
	setKeyframePose(k, "leg_l", 10.0f, 0.0f, 0.0f);
	setKeyframePose(k, "knee_l", 15.0f, 0.0f, 0.0f);
	setKeyframePose(k, "knee_r", 5.0f, 0.0f, 0.0f);

	// K1 (0.8 s): Agrupamiento
	//   Tronco flexionado sobre la pierna derecha, rodilla derecha flexionada
	//   y pierna izquierda recogida
	k = addKeyframe(0.8f, glm::vec3(-0.151f, 0.953f, 0.000f), -90.0f);
	setKeyframePose(k, "pelvis", -55.0f, 0.0f, 0.0f);
	setKeyframePose(k, "spine", -15.0f, 0.0f, 0.0f);
	setKeyframePose(k, "neck", 20.0f, 0.0f, 0.0f);
	setKeyframePose(k, "shoulder_l", 60.0f, -50.0f, 0.0f);
	setKeyframePose(k, "elbow_l", 0.0f, -10.0f, 0.0f);
	setKeyframePose(k, "shoulder_r", 10.0f, 25.0f, 0.0f);
	setKeyframePose(k, "elbow_r", 0.0f, 140.0f, 0.0f);
	setKeyframePose(k, "wrist_r", 0.0f, 30.0f, 0.0f);
	setKeyframePose(k, "leg_l", -20.0f, 0.0f, 0.0f);
	setKeyframePose(k, "knee_l", 90.0f, 0.0f, 0.0f);
	setKeyframePose(k, "ankle_l", 10.0f, 0.0f, 0.0f);
	setKeyframePose(k, "leg_r", -50.0f, 0.0f, 0.0f);
	setKeyframePose(k, "knee_r", 75.0f, 0.0f, 0.0f);
	setKeyframePose(k, "ankle_r", -25.0f, 0.0f, 0.0f);

	// K2 (1.5 s): Impulso
	//   La pierna izquierda se lanza hacia la dirección de lanzamiento
	//   y la derecha empuja; el cuerpo se desplaza
	k = addKeyframe(1.5f, glm::vec3(-0.100f, 0.927f, 0.000f), -80.0f);
	setKeyframePose(k, "pelvis", -50.0f, 0.0f, 0.0f);
	setKeyframePose(k, "spine", -10.0f, 0.0f, 0.0f);
	setKeyframePose(k, "neck", 20.0f, 0.0f, 0.0f);
	setKeyframePose(k, "shoulder_l", 40.0f, -60.0f, 0.0f);
	setKeyframePose(k, "elbow_l", 0.0f, -20.0f, 0.0f);
	setKeyframePose(k, "shoulder_r", 10.0f, 25.0f, 0.0f);
	setKeyframePose(k, "elbow_r", 0.0f, 140.0f, 0.0f);
	setKeyframePose(k, "wrist_r", 0.0f, 30.0f, 0.0f);
	setKeyframePose(k, "leg_l", 50.0f, 0.0f, 0.0f);
	setKeyframePose(k, "knee_l", 10.0f, 0.0f, 0.0f);
	setKeyframePose(k, "leg_r", -30.0f, 0.0f, 0.0f);
	setKeyframePose(k, "knee_r", 40.0f, 0.0f, 0.0f);
	setKeyframePose(k, "ankle_r", -10.0f, 0.0f, 0.0f);

	// K3 (2.1 s): Posición de fuerza
	//   Pierna derecha flexionada bajo el cuerpo, pierna izquierda extendida
	//   hacia la dirección de lanzamiento y tronco girado hacia atrás
	k = addKeyframe(2.1f, glm::vec3(0.150f, 0.872f, 0.000f), -45.0f);
	setKeyframePose(k, "pelvis", -35.0f, 0.0f, 0.0f);
	setKeyframePose(k, "spine", -10.0f, 0.0f, -30.0f);
	setKeyframePose(k, "neck", 15.0f, 0.0f, 0.0f);
	setKeyframePose(k, "shoulder_l", 0.0f, -70.0f, 0.0f);
	setKeyframePose(k, "elbow_l", 0.0f, -40.0f, 0.0f);
	setKeyframePose(k, "shoulder_r", 10.0f, 25.0f, 0.0f);
	setKeyframePose(k, "elbow_r", 0.0f, 140.0f, 0.0f);
	setKeyframePose(k, "wrist_r", 0.0f, 30.0f, 0.0f);
	setKeyframePose(k, "leg_l", 35.0f, 30.0f, 0.0f);
	setKeyframePose(k, "knee_l", 5.0f, 0.0f, 0.0f);
	setKeyframePose(k, "leg_r", -35.0f, 0.0f, 0.0f);
	setKeyframePose(k, "knee_r", 60.0f, 0.0f, 0.0f);
	setKeyframePose(k, "ankle_r", -25.0f, 0.0f, 0.0f);

	// K4 (2.8 s): Empuje
	//   Las caderas y el tronco giran hacia la dirección de lanzamiento,
	//   las piernas se extienden y el codo derecho se eleva
	k = addKeyframe(2.8f, glm::vec3(0.486f, 1.062f, 0.308f), 45.0f);
	setKeyframePose(k, "pelvis", -10.0f, 0.0f, 0.0f);
	setKeyframePose(k, "spine", 0.0f, 0.0f, 20.0f);
	setKeyframePose(k, "shoulder_l", 10.0f, 20.0f, 0.0f);
	setKeyframePose(k, "elbow_l", 0.0f, -60.0f, 0.0f);
	setKeyframePose(k, "shoulder_r", -20.0f, 50.0f, 0.0f);
	setKeyframePose(k, "elbow_r", 0.0f, 80.0f, 0.0f);
	setKeyframePose(k, "wrist_r", 0.0f, 20.0f, 0.0f);
	setKeyframePose(k, "leg_l", -12.0f, 18.0f, 0.0f);
	setKeyframePose(k, "knee_l", 5.0f, 0.0f, 0.0f);
	setKeyframePose(k, "leg_r", -15.0f, 0.0f, 0.0f);
	setKeyframePose(k, "knee_r", 25.0f, 0.0f, 0.0f);
	setKeyframePose(k, "ankle_r", 10.0f, 0.0f, 0.0f);

	// K5 (3.2 s): Lanzamiento
	//   Cuerpo extendido de puntillas y brazo derecho completamente extendido
	//   hacia delante y arriba (unos 40º)
	k = addKeyframe(3.2f, glm::vec3(0.754f, 1.178f, 0.340f), 80.0f);
	setKeyframePose(k, "pelvis", 8.0f, 0.0f, 0.0f);
	setKeyframePose(k, "spine", 0.0f, 0.0f, 10.0f);
	setKeyframePose(k, "neck", 5.0f, 0.0f, 0.0f);
	setKeyframePose(k, "shoulder_l", 70.0f, 10.0f, 0.0f);
	setKeyframePose(k, "elbow_l", 0.0f, -60.0f, 0.0f);
	setKeyframePose(k, "shoulder_r", -40.0f, 80.0f, 0.0f);
	setKeyframePose(k, "leg_l", -10.0f, 5.0f, 0.0f);
	setKeyframePose(k, "ankle_l", 35.0f, 0.0f, 0.0f);
	setKeyframePose(k, "leg_r", 10.0f, 0.0f, 0.0f);
	setKeyframePose(k, "ankle_r", 40.0f, 0.0f, 0.0f);

	// K6 (3.8 s): Cambio de piernas
	//   La pierna derecha avanza, la izquierda se eleva por detrás y el
	//   tronco se inclina para recuperar el equilibrio
	k = addKeyframe(3.8f, glm::vec3(0.850f, 0.952f, 0.250f), 90.0f);
	setKeyframePose(k, "pelvis", -35.0f, 0.0f, 0.0f);
	setKeyframePose(k, "spine", -5.0f, 0.0f, 0.0f);
	setKeyframePose(k, "neck", 20.0f, 0.0f, 0.0f);
	setKeyframePose(k, "shoulder_l", 40.0f, 40.0f, 0.0f);
	setKeyframePose(k, "elbow_l", 0.0f, -30.0f, 0.0f);
	setKeyframePose(k, "shoulder_r", 20.0f, 70.0f, 0.0f);
	setKeyframePose(k, "leg_l", 45.0f, 0.0f, 0.0f);
	setKeyframePose(k, "knee_l", 20.0f, 0.0f, 0.0f);
	setKeyframePose(k, "leg_r", -25.0f, 0.0f, 0.0f);
	setKeyframePose(k, "knee_r", 30.0f, 0.0f, 0.0f);
	setKeyframePose(k, "ankle_r", -10.0f, 0.0f, 0.0f);

	// K7 (5.0 s): Vuelta a la posición inicial (igual que K0) para que la
	//   animación sea cíclica
	k = addKeyframe(5.0f, glm::vec3(-0.400f, 1.061f, 0.000f), -90.0f);
	setKeyframePose(k, "shoulder_l", 20.0f, -70.0f, 0.0f);
	setKeyframePose(k, "elbow_l", 0.0f, -20.0f, 0.0f);
	setKeyframePose(k, "shoulder_r", 10.0f, 25.0f, 0.0f);
	setKeyframePose(k, "elbow_r", 0.0f, 140.0f, 0.0f);
	setKeyframePose(k, "wrist_r", 0.0f, 30.0f, 0.0f);
	setKeyframePose(k, "leg_l", 10.0f, 0.0f, 0.0f);
	setKeyframePose(k, "knee_l", 15.0f, 0.0f, 0.0f);
	setKeyframePose(k, "knee_r", 5.0f, 0.0f, 0.0f);
}
```

Punto de control 6: compila (no ejecutes todavía: nadie usa GEAnimation). Debe compilar sin errores. Si da "identificador no declarado" en `createKeyframes`, comprueba que la declaraste en el .h.

## Fase 7. GEScene final: animación en tiempo real

La escena guarda un GEAnimation y un reloj; en cada frame mide cuánto tiempo ha pasado y pide a la animación la pose de ese instante. Es lo que pide el enunciado: "el objeto de animación y un método que actualice la pose del esqueleto a lo largo del tiempo".

**Por qué con tiempo real y no con un contador.** La práctica 7 sumaba 1º por frame (`angle += 1.0f`): la velocidad depende de los FPS de tu PC. El enunciado pide que la animación dure 5 segundos, así que usamos `glfwGetTime()`, que devuelve los segundos desde que arrancó GLFW. Ya está disponible en GEScene.cpp porque GLFW se incluye a través de GEGraphicsContext.h.

**Cómo funciona el reloj con pausa.** `animTime` es el tiempo de la animación. En cada frame se le suma lo que ha pasado desde el frame anterior (`now - lastTime`), salvo en pausa. Para reiniciar basta con `animTime = 0`.

### 7.1 GEScene.h — cambios

Añade `#include "GEAnimation.h"` debajo de `#include "GESkeleton.h"`, cuatro campos nuevos y el método `updatePose`. La clase queda así:

```cpp
class GEScene
{
private:
	GERenderingContext* rc;
	GEFigure* ground;
	GESkeleton* skeleton;
	GEAnimation* animation;
	double animTime;
	double lastTime;
	bool paused;
	GECamera* camera;
	glm::mat4 projection;

public:
	GEScene(GEGraphicsContext* gc, GEDrawingContext* dc, GECommandContext* cc);
	void destroy(GEGraphicsContext* gc);
	void recreate(GEGraphicsContext* gc, GEDrawingContext* dc, GECommandContext* cc);
	void update(GEGraphicsContext* gc, uint32_t index);
	void updatePose();
	void key_action(int key, bool pressed);
	void aspect_ratio(double aspect);

private:
	GEPipelineConfig* createPipelineConfig(VkExtent2D extent);
	void fillCommandBuffers(std::vector<VkCommandBuffer> commandBuffers);
};
```

### 7.2 GEScene.cpp — cambios

**Includes:** añade `#include "GEAnimation.h"` y `#include <cmath>` (para `fmod`).

**Constructor:** tres cambios.

1. Cámara: el lanzamiento va hacia la derecha (+X), así que la ponemos delante, algo a la izquierda y mirando un poco de lado, para ver el gesto casi de perfil con algo de profundidad. `setDirection` recibe el vector que va del objetivo hacia la cámara (la cámara mira hacia −Dir):

```cpp
	this->camera = new GECamera();
	this->camera->setPosition(glm::vec3(-0.92f, 1.15f, 5.38f));
	this->camera->setDirection(glm::vec3(-0.208f, 0.0f, 0.978f), glm::vec3(0.0f, 1.0f, 0.0f));
	this->camera->setMoveStep(0.0f);
```

2. Suelo más grande, para que con esta cámara no se vea su borde: `ground = new GEGround(10.0f, 10.0f);`
3. Quita las 6 líneas de la pose de ejemplo y, después de `skeleton->setLight(light);`, crea la animación y pon la primera pose:

```cpp
	animation = new GEAnimation();
	animTime = 0.0;
	lastTime = glfwGetTime();
	paused = false;
	updatePose();
```

**destroy:** añade `delete animation;` junto a los demás `delete`.

**update:** llama a `updatePose()` antes de actualizar las figuras:

```cpp
void GEScene::update(GEGraphicsContext* gc, uint32_t index)
{
	camera->update();
	glm::mat4 view = camera->getViewMatrix();

	updatePose();

	ground->update(gc, index, view, projection);
	skeleton->update(gc, index, view, projection);
}
```

**updatePose (nuevo):** ponlo detrás de `update`.

```cpp
//
// FUNCIÓN: GEScene::updatePose()
//
// PROPÓSITO: Actualiza la pose del esqueleto a lo largo del tiempo. El tiempo
//            de la animación avanza con el tiempo real (salvo en pausa).
//
void GEScene::updatePose()
{
	double now = glfwGetTime();
	if (!paused) animTime += now - lastTime;
	lastTime = now;

	animation->animate(skeleton, (float)animTime);
}
```

**key\_action:** añade estos tres `case` al final del `switch`, después del de `GLFW_KEY_L`. Se actúa al **soltar** la tecla (`!pressed`) porque GEApplication también llama con `pressed = true` mientras la mantienes pulsada, y la pausa se activaría y desactivaría muchas veces.

- Espacio: pausa / continúa.
- R: vuelve a empezar.
- N: salta al siguiente keyframe y pausa. Recorre los keyframes en orden; tras el último vuelve a K0. Muy útil para las capturas de la memoria.

```cpp
	case GLFW_KEY_SPACE:
		// Pausa / continúa la animación (al soltar la tecla)
		if (!pressed) paused = !paused;
		break;
	case GLFW_KEY_R:
		// Reinicia la animación
		if (!pressed) animTime = 0.0;
		break;
	case GLFW_KEY_N:
		// Avanza (en pausa) hasta el siguiente keyframe
		if (!pressed)
		{
			float duration = animation->getDuration();
			float t = (float)fmod(animTime, duration);
			int k = 0;
			while (k < animation->getKeyframeCount() - 1 && animation->getKeyframeTime(k) <= t + 0.001f) k++;
			animTime = (k < animation->getKeyframeCount() - 1) ? animation->getKeyframeTime(k) : 0.0;
			paused = true;
		}
		break;
```

Punto de control 7: compila y ejecuta. El esqueleto empieza de espaldas a la derecha de la pantalla, se agacha (0,8 s), se impulsa (1,5 s), queda en posición de fuerza (2,1 s), gira y empuja (2,8 s), lanza con el brazo derecho estirado hacia arriba y a la derecha (3,2 s), cambia de piernas (3,8 s) y vuelve al inicio (5,0 s). Se repite sin saltos. Comprueba Espacio, R y N.

Referencia: así se ve cada keyframe con la cámara final (lo que tendrás pulsando N).

&#91;image: Los 8 keyframes del lanzamiento de peso\]

Si va demasiado rápido o lento, mira que uses `glfwGetTime()` y no un contador de frames. Si se mueve "a trompicones" solo al principio, es normal: el primer frame tarda más porque se crean todos los buffers.

## Fase 8. Límites en body.skel

El código de límites ya funciona (fase 4 y lector de la fase 5); falta escribirlos en body.skel para que se usen. El enunciado describe la etiqueta `<limit>` pero no da su formato, así que lo defines tú y lo explicas en la memoria:

```
<limit xmin="..." xmax="..." ymin="..." ymax="..." zmin="..." zmax="..." />
```

Valores en grados. Un atributo que falte vale -180 (mínimo) o 180 (máximo), es decir, sin límite.

**Paso 8.1.** En body.skel añade una etiqueta `<limit>` dentro de cada una de estas articulaciones, justo después de su `<yaxis>` (antes de las articulaciones hijas):

| Articulación | Línea a añadir | Por qué |
| --- | --- | --- |
| neck | `<limit xmin="-60.0" xmax="60.0" ymin="-45.0" ymax="45.0" zmin="-80.0" zmax="80.0" />` | El cuello no da la vuelta |
| knee\_l y knee\_r | `<limit xmin="0.0" xmax="150.0" ymin="0.0" ymax="0.0" zmin="0.0" zmax="0.0" />` | La rodilla es una bisagra: solo se dobla hacia atrás |
| ankle\_l y ankle\_r | `<limit xmin="-45.0" xmax="60.0" ymin="-30.0" ymax="30.0" zmin="-30.0" zmax="30.0" />` | Movilidad normal del tobillo |

Ejemplo de cómo queda la rodilla izquierda:

```
			<balljoint name="knee_l" length="0.4">
				<offset x="0.0" y="0.0" z="0.0" />
				<zaxis x="0.0" y="0.0" z="1.0" />
				<yaxis x="0.0" y="1.0" z="0.0" />
				<limit xmin="0.0" xmax="150.0" ymin="0.0" ymax="0.0" zmin="0.0" zmax="0.0" />
				<balljoint name="ankle_l" length="0.25">
```

Estos rangos respetan todas las poses de la fase 6 y la pose de ejemplo del enunciado (knee\_l 30º en X). Si cambias poses, comprueba que no se salen de los límites: si se salen, se recortan sin avisar y la pose no se verá como esperas.

**Paso 8.2.** Compilar → **Recompilar solución** (body.skel va dentro del .exe).

**Paso 8.3 — Prueba de que los límites funcionan.** En el constructor de GEScene, justo después de `updatePose();`, añade temporalmente:

```cpp
	skeleton->getJoint("knee_r")->setPose(-60.0f, 0.0f, 0.0f);
	glm::vec3 p = skeleton->getJoint("knee_r")->getPose();
	printf("knee_r pide -60, queda en %f\n", p.x);
```

Punto de control 8: al ejecutar, en la consola negra aparece `knee_r pide -60, queda en 0.000000`. Sin el `<limit>` de knee\_r aparecería -60. Haz una captura de la consola para la memoria y después borra esas 3 líneas (si `printf` da error, añade `#include <cstdio>` arriba).

## Fase 9. Pruebas, capturas y memoria

El enunciado pide 4 entregables; esta fase te dice qué poner en cada uno y de dónde sacarlo.

**Paso 9.1 — Capturas.** Ejecuta, pulsa **N** (salta a K1 y pausa), haz la captura de la ventana con Win+Mayús+S o Alt+ImprPant, y repite N para K2…K7. Haz también:

- La captura del punto de control 5a (reposo) y 5b (pose de ejemplo del enunciado), para compararlas con las imágenes del enunciado.
- La captura del punto de control 4 (la cruz con el antebrazo), como prueba del árbol.
- La captura de la consola del punto de control 8 (límites).
- Opcional: con Espacio pausa en instantes intermedios (por ejemplo entre K4 y K5) para mostrar que la interpolación crea poses que no son keyframes.

**Paso 9.2 — Índice recomendado de la memoria**

1. **Introducción**: objetivo (esqueleto de body.skel + lanzamiento de peso de 5 s por interpolación lineal de keyframes) y punto de partida (práctica 7).
2. **Formato del fichero y lectura**: resumen del formato .skel, la decisión sobre `<limit>` (fase 8) y cómo funciona el lector con la pila (fase 5.5). Incluye el código de `parse` y `readTag`.
3. **Clase GEBalljoint (modificada)**: qué se añade y por qué (tabla de la fase 4), la fórmula de la matriz con el padre, y el código de los métodos modificados: constructor, ComputeMatrix, initialize, update, destroy, addCommands, setLight, setPose, y los nuevos.
4. **Clase GESkeleton (nueva)**: código completo y explicación (la raíz del árbol, los métodos de GEFigure, los getters/setters).
5. **Clase GEAnimation (nueva)**: código completo, la fórmula de interpolación con el ejemplo numérico de la fase 6 y por qué se interpola también la posición y el giro del cuerpo.
6. **Clase GEScene (modificada)**: código de los métodos modificados (constructor, destroy, update, fillCommandBuffers, key\_action) y del nuevo updatePose; por qué se usa `glfwGetTime()`.
7. **Descripción de los keyframes** (entregable explícito del enunciado): una tabla K0…K7 con instante, fase del lanzamiento, posición y giro del cuerpo y descripción; otra tabla con los ángulos de cada articulación; y las capturas de cada keyframe.
8. **Pruebas realizadas**: cada punto de control como una prueba (qué se probó, cómo, resultado, captura): árbol con la cruz (4), lectura del fichero en reposo (5a), pose del enunciado (5b), animación (7), límites (8).
9. **Conclusiones**: problemas encontrados y cómo los resolviste.

**Paso 9.3 — Código fuente y compilado.** Comprime la carpeta `C:\GameEngine\Projects\Trabajo` (sin la carpeta oculta `.vs` ni `Project7\x64`, que son archivos intermedios y pesan mucho) y añade aparte el ejecutable compilado: está en `C:\GameEngine\Projects\Trabajo\x64\Release\Project7.exe`. Como body.skel y los shaders van dentro del .exe, el ejecutable funciona solo (en un PC con Vulkan).

**Paso 9.4 — Repaso final con el enunciado**

- [ ] GESkeleton guarda toda la información de body.skel (nombre, posición, ejes, articulaciones)
- [ ] GESkeleton tiene los métodos de GEFigure (initialize, destroy, addCommands, update, resetLocation, setLocation, translate, rotate, setMaterial, setLight)
- [ ] GESkeleton tiene getters/setters de posición, orientación y para añadir articulaciones
- [ ] GEBalljoint tiene hijas, límites y pose, con sus getters/setters
- [ ] Los métodos de GEBalljoint recorren las hijas
- [ ] GEAnimation calcula la pose instantánea por interpolación lineal de keyframes
- [ ] Brazos y piernas se mueven coordinados; la animación dura 5 s
- [ ] GEScene tiene suelo, esqueleto, objeto de animación y un método que actualiza la pose con el tiempo
- [ ] Memoria con código, keyframes, pruebas y capturas

## Errores típicos y cómo arreglarlos

Busca el mensaje que te sale en la primera columna; casi todos vienen de un paso saltado.

| Síntoma | Causa probable | Solución |
| --- | --- | --- |
| `No se puede abrir el archivo de inclusión: 'vulkan/vulkan.h'` o `LNK1104 vulkan-1.lib` | El SDK no está en `C:\VulkanSDK\1.4.313.2` | Instala esa versión o cambia las rutas (paso 0.6) |
| `No se puede abrir 'GLFW/glfw3.h'` o `glm/glm.hpp` | La carpeta GameEngine no está en `C:\` | Revisa el paso 0.3 |
| `LNK2019: símbolo externo sin resolver ... GESkeleton::...` (o GEAnimation) | El .cpp no está añadido al proyecto | Clic derecho en el proyecto → Agregar → Elemento existente → el .cpp |
| `LNK2019 ... GEBalljoint::setBoneMaterial` (o cualquier método) | Declarado en el .h pero no escrito en el .cpp, o escrito con otro nombre/parámetros | Compara la firma del .h con la del .cpp letra a letra |
| `C2661` / `C2664` en `new GEBalljoint(0.4f)` | El constructor ahora pide nombre | `new GEBalljoint("nombre", 0.4f)` |
| `IDR_HTML3: identificador no declarado` | Falta el `#define` en resource.h o falta `#include "resource.h"` | Paso 3.3 |
| Al arrancar: excepción "No se encuentra el fichero de descripcion del esqueleto" | El recurso no está en el .exe | Revisa la línea de body.skel en el .rc (paso 3.4) y usa Recompilar solución |
| El programa se cierra al arrancar sin mensaje, en `setLight` | `setLight` antes de `initialize` (esfera y cilindro aún son `nullptr`) | Orden: crear → `initialize` → `setLight` |
| Se cierra en `findJoint(...)->setPose` | El nombre no existe y `findJoint` devuelve `nullptr` | Revisa el nombre exacto (minúsculas, `_l` / `_r`) |
| No se ve el esqueleto (solo el suelo) | No está en `fillCommandBuffers` o no se llama a su `update` | Fase 5.6, pasos 5 y 6 |
| El esqueleto sale "desmontado" (huesos separados o todos en el mismo punto) | Falta `parentMatrix` en `matrix = parentMatrix * jointm * posem` o falta pasar `endMatrix` a las hijas | Revisa ComputeMatrix (fase 4.2) |
| Las piernas aparecen colgando de la cabeza o de los hombros | Fallo en la pila del lector: no se desapila en `</balljoint>` | Revisa el caso `closing` en `parse` |
| La pose no cambia aunque llamas a `setPose` | Estás fuera de los límites (se recorta) o la animación la sobrescribe en el siguiente frame | Revisa los `<limit>`; las pruebas de poses hazlas sin animación (fase 5.6) |
| La animación va muy rápida o muy lenta | Se usa un contador de frames | Usa `glfwGetTime()` (fase 7) |
| Los acentos de los comentarios salen raros | Codificación del fichero | No afecta a la compilación; Archivo → Guardar como → flecha de Guardar → "Guardar con codificación" y elige UTF-8 con firma |
| Se cierra al crear el pipeline (`vkCreateGraphicsPipelines`) | Error del código base: `GERenderingContext::createPipelineVertexInputStateCreateInfo()` guarda punteros a dos vectores locales que se destruyen al salir | Raro en Windows (la práctica 7 funciona igual). Si te pasa, añade `static` delante de las declaraciones de `bindingDescriptions` y `attributeDescriptions` en esa función |

Si algo no está en la tabla, vuelve al último punto de control que funcionó y compara tu código con el de esa fase: el fallo está en lo que cambiaste desde entonces.
