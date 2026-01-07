# Validación de Optimización Paralela con FiberSCIP

Este repositorio documenta la validación funcional del solver **FiberSCIP** en un entorno multi-núcleo.

El objetivo principal es demostrar que la instalación actual soporta correctamente la ejecución paralela. A través de la resolución de la instancia `glass4`, se verifica que el sistema es capaz de instanciar múltiples solvers y distribuir la carga de trabajo (Branch-and-Bound) entre los procesadores lógicos disponibles, confirmando que el entorno está habilitado para computación concurrente.

## Sobre UG (Ubiquity Generator)

UG es un framework genérico diseñado para paralelizar solvers basados en *Branch-and-Bound* (por ejemplo, MIP, MINLP, ExactIP) tanto en entornos de memoria distribuida como compartida. Su arquitectura permite explotar el rendimiento de solvers de estado del arte base (como SCIP o Xpress) sin necesidad de reescribir o paralelizar el código interno del solver base.

<img width="915" height="300" alt="image" src="https://github.com/user-attachments/assets/a321a7c1-1500-43a9-a8ed-ad675a80559a" />

## Concepto del Framework de Paralelización: Paradigma Supervisor-Trabajador

UG implementa un modelo de paralelización de tareas de alto nivel basado en el paradigma **Supervisor-Trabajador** (Supervisor-Worker), diferenciándose del modelo clásico Maestro-Trabajador (Master-Worker).

En esta arquitectura:
*   **LoadCoordinator (Supervisor):** Gestiona el balanceo de carga dinámico entre los solvers.
*   **Solvers (Workers):** Instancias del solver base (SCIP) que procesan sub-problemas (Tasks) de alta granularidad.

### Comparativa de Paradigmas

**1. Paradigma Maestro-Trabajador (Master-Worker)**
En este modelo clásico, la comunicación es simple ("Task" y "Result"). El Maestro gestiona todos los nodos abiertos del árbol de búsqueda. Aunque es adecuado para computación de alto rendimiento (High-Throughput), puede presentar cuellos de botella en HPC a gran escala debido a la enorme cantidad de nodos transferidos.

<img width="1109" height="862" alt="image" src="https://github.com/user-attachments/assets/330036bd-28c1-4e82-850f-a2a4f1568f4c" />

**2. Paradigma Supervisor-Trabajador (Implementado en UG)**
UG define un protocolo de paso de mensajes flexible entre el inicio ("Task") y la finalización ("Completion") de una tarea. Mensajes como "Solution" (nueva solución incumbente), "InCollecting" (solicitud de nuevas tareas) o "Interrupt" permiten una coordinación más eficiente. El **LoadCoordinator** mantiene solo los nodos raíz de los sub-problemas activos, delegando la gestión profunda del árbol a los Workers, lo cual optimiza el uso de memoria y ancho de banda en la comunicación.

<img width="1104" height="863" alt="image" src="https://github.com/user-attachments/assets/8ff0e152-f788-4ebb-86cc-f796a23823dc" />

## Definición del Problema: glass4

Se utilizó la instancia `glass4` de MIPLIB 2010/2017, un problema clásico de "Nesting" (anidamiento de piezas) caracterizado por su alta complejidad combinatoria.

| Propiedad | Valor | Descripción |
|-----------|-------|-------------|
| **Nombre** | `glass4.mps` | Instancia de corte de vidrio (Glass Cutting) |
| **Variables** | 322 | 302 Binarias, 20 Continuas |
| **Restricciones** | 396 | Partitioning y Precedencia |
| **Densidad** | ~1.42% | Matriz dispersa |
| **Objetivo** | Minimizar | Mejor solución conocida: ~1.2000126e+09 |

## Entorno de Pruebas

La validación se ejecutó en una estación de trabajo de alto rendimiento con las siguientes especificaciones:

| Componente | Especificación |
|------------|----------------|
| **Procesador** | 6 Procesadores Lógicos (Base 2.38 GHz) |
| **Arquitectura** | x64 con Virtualización Activada |
| **Caché L1/L2/L3** | 384 kB / 3.0 MB / 8.0 MB |
| **Solver** | SCIP Optimization Suite 10.0.0 (FiberSCIP) |
| **Algoritmo** | Parallel Branch-and-Bound (Racing Ramp-up) |

---

## Guía de Inicio Rápido (Quickstart)

Esta sección detalla los pasos para reproducir los resultados obtenidos.

### 1. Prerrequisitos
Instalación de **SCIP Optimization Suite** (versión 8.0 o superior). El ejecutable `fscip` debe estar accesible.

### 2. Configuración (default.prm)
El archivo de parámetros controla el comportamiento del framework UG. Configuración utilizada:

```ini
# Nivel de verbosidad (4 para ver todo el movimiento paralelo)
OutputParaParams = 2

# No silenciar la salida
Quiet = FALSE
```

> Notar que el número de solvers se ajusta automáticamente o mediante argumentos de línea de comandos si no se especifica explicitamente en el archivo de parámetros.

### 3. Ejecución
Comando para iniciar la resolución:

```bash
"ruta/a/fscip.exe" default.prm glass4.mps
```

---

## Resultados de la Validación

La ejecución concluyó exitosamente. A continuación se presentan los fragmentos relevantes del log de ejecución que demuestran la convergencia y la optimalidad.

### Convergencia al Óptimo

El solver logró cerrar el "Gap" al **0.00%**, encontrando la solución óptima comprobada.

```text
...
*    1525        2806177        3402         6  1200012600.0000   800006321.7489     50.00%   800006321.7489     50.00%
     1527        2807218         941         6  1200012600.0000   800006321.7489     50.00%   800006321.7489     50.00%
     1532        2809095          48         6  1200012600.0000   800006321.7489     50.00%   800006321.7489     50.00%
     1538        2809103          42         6  1200012600.0000   800006321.7489     50.00%   800006321.7489     50.00%
     1544        2809112          36         6  1200012600.0000   828577747.1684     44.83%   828577747.1684     44.83%
     1549        2809230          34         5  1200012600.0000   833339690.7874     44.00%   833339690.7874     44.00%
     1554        2809512           8         5  1200012600.0000  1043908033.1194     14.95%  1043908033.1194     14.95%
     1560        2809562           0         2  1200012600.0000  1066673980.0339     12.50%  1066673980.0339     12.50%
     1561        2809568           0         0  1200012600.0000  1200012600.0000      0.00%
SCIP Status        : problem is solved
Total Time         : 1561.36
  solving          : 1561.36
  presolving       : 0.00 (included in solving)
B&B Tree           : 
  nodes (total)    : 2809558
Solution           : 
  Solutions found  : 55
  Primal Bound     : +1.20001259999999e+09
  Dual Bound       : +1.20001259999999e+09
Gap                : 0.00000 %
* Warning: the number of nodes (total) including nodes solved by interrupted Solvers is 2809568
```

*   **Total Time:** 1561.36s (~26 minutos), confirmando la robustez para problemas de larga duración.
*   **Gap 0.00%:** Prueba matemática de que no existe una solución mejor.

### Evidencia de Trazas (Logs de Paralelismo)

El log final del **Load Coordinator** confirma el funcionamiento del sistema Supervisor-Trabajador:

```log
######### The number of nodes solved in all solvers: 2809558 : 2809568 #########
######### LoadCoordinator Rank = 0 is terminated. #########
#=== The number of ParaNodes received = 357
#=== The number of ParaNodes sent = 91
#=== The number of ParaNodes deleted in LoadCoordinator = 208
#=== Elapsed time to terminate this LoadCoordinator  = 1561.36
```

**Interpretación de la Prueba de Paralelismo:**

*   **ParaNodes:** Representan sub-problemas o ramas del árbol de decisión.
*   **Sent = 91:** Indica que el coordinador **movió trabajo activamente** de un procesador ocupado a otro libre en 91 ocasiones. Si el sistema no hubiera funcionado en paralelo, este valor sería 0, ya que no habría habido redistribución de carga.
*   Esto valida que múltiples núcleos contribuyeron a la resolución del problema de manera cooperativa y balanceada.

---

## Referencias y Documentación

*   **SCIP Optimization Suite:** [https://scipopt.org/](https://scipopt.org/)
*   **UG (Ubiquity Generator) Home:** [https://ug.zib.de](https://ug.zib.de)
*   **Conceptos de UG:** [https://ug.zib.de/doc/html/CONCEPT.php](https://ug.zib.de/doc/html/CONCEPT.php)
*   **MIPLIB 2017 (Instancia glass4):** [https://miplib.zib.de/instance_details_glass4.html](https://miplib.zib.de/instance_details_glass4.html)
