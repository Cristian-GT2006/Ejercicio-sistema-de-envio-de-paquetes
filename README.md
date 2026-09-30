# Ejercicio-sistema-de-envio-de-paquetes
# 📦 Arquitectura de Pruebas y Lógica Comercial para Sistemas de Envío

Este repositorio implementa un motor logístico interactivo en consola para la cotización de envíos postales. El desarrollo sigue los principios de **TDD (Test-Driven Development)**, definiendo una matriz exhaustiva de validaciones límites y prioridades comerciales antes de escribir una sola línea de código fuente.

**Autor:** Cristian Gómez T.  
**Materia:** Programación / Control de Calidad de Software (QA)  

---

## 📋 1. Definición de Límites y Prioridades

Para garantizar que el software nunca entre en un estado indeterminado o calcule tarifas erróneas, se han establecido las siguientes directrices estructurales:

### ⏱️ Matriz de Prioridades de Ejecución (Orden de Validación)
1.  **Prioridad 1 (Filtro Geográfico):** Verificar la existencia de la zona destino (`1` o `2`). Si falla, detiene el programa inmediatamente.
2.  **Prioridad 2 (Integridad Física del Paquete):** Evaluar si el peso se encuentra estrictamente en la cota admisible `[0.1 kg - 30.0 kg]`.
3.  **Prioridad 3 (Nivel de Servicio):** Validar la modalidad logística seleccionada (`Normal` o `Exprés`).
4.  **Prioridad 4 (Tasación Base):** Cruzar la zona, el peso y el servicio dentro de la matriz de precios.
5.  **Prioridad 5 (Penalizaciones y Bonificaciones):** Calcular los recargos por tonelaje excesivo (`> 10 kg`) y, finalmente, deducir el beneficio del cliente frecuente sobre la suma total de los costos acumulados.

---

## 🧪 2. Plan de Pruebas Unitarias (10 Casos de Prueba Obligatorios)

La siguiente tabla representa las especificaciones de control de calidad diseñadas previamente al desarrollo del código:

| Test | Escenario de Prueba | Zona (Entrada) | Peso (Entrada) | Servicio (Entrada) | Frecuente (Entrada) | Regla Evaluada | Salida / Total Esperado | Estado |
| :---: | :--- | :---: | :---: | :---: | :---: | :--- | :--- | :---: |
| **1** | Cotización Base Local Mínima | `1` (Local) | `3.5 kg` | `1` (Normal) | `N` | Tarifa Plana Rango 1 | **$3.00** | ✅ Pass |
| **2** | Local con Entrega Exprés y Descuento | `1` (Local) | `4.0 kg` | `2` (Exprés) | `S` | Fidelización del 10% | **$4.50** *(5.00 - 0.50)* | ✅ Pass |
| **3** | Rango de Peso Medio Local | `1` (Local) | `8.2 kg` | `1` (Normal) | `N` | Rango de peso (5 a 10 kg) | **$5.00** | ✅ Pass |
| **4** | Sobrepeso Crítico Local con Exprés | `1` (Local) | `15.0 kg` | `2` (Exprés) | `S` | Recargo por exceso de peso | **$17.55** *(12.00 base + 7.50 recargo - 10% desc)* | ✅ Pass |
| **5** | Cotización Nacional Estándar | `2` (Nacional)| `2.0 kg` | `1` (Normal) | `N` | Tarifa Nacional Rango 1 | **$6.00** | ✅ Pass |
| **6** | Nacional Exprés con Fidelidad | `2` (Nacional)| `9.5 kg` | `2` (Exprés) | `S` | Matriz nacional peso medio | **$13.50** *(15.00 - 1.50)* | ✅ Pass |
| **7** | Máximo Tonelaje Nacional Permitido | `2` (Nacional)| `25.0 kg` | `1` (Normal) | `S` | Peso máximo admisible | **$41.85** *(24.00 base + 22.50 recargo - 10% desc)* | ✅ Pass |
| **8** | Rechazo por Exceso de Capacidad | `1` (Local) | `45.0 kg` | `1` (Normal) | `N` | Control de límites (R2) | **ERROR:** Fuera de rango (0.1 a 30kg) | ✅ Pass |
| **9** | Rechazo por Destino Inexistente | `3` (Inter.) | `5.0 kg` | `2` (Exprés) | `N` | Control de límites (R1) | **ERROR:** Zona no válida | ✅ Pass |
| **10**| Rechazo por Carga Nula/Negativa | `2` (Nacional)| `0.05 kg`| `1` (Normal) | `S` | Límite inferior de carga | **ERROR:** Fuera de rango (0.1 a 30kg) | ✅ Pass |

---

## 💻 3. Pseudocódigo

```text
Algoritmo SistemaEnvioPaquetes
    Definir zona Como Entero 
    Definir tipoServicio Como Entero 
    Definir esFrecuenteRespuesta Como Caracter
    Definir esClienteFrecuente Como Logico
    Definir peso, tarifaBase, recargoExceso, subtotal, descuento, costoTotal Como Real
    Definir datosValidos Como Logico
    
    datosValidos <- Verdadero
    tarifaBase <- 0
    recargoExceso <- 0
    descuento <- 0
    
    Escribir "=== SISTEMA DE CALCULO DE ENVIOS DE PAQUETES ==="
    
    // 1. Prioridad: Validación de Zona
    Escribir "Seleccione la Zona del envio (1: Local, 2: Nacional):"
    Leer zona
    Si zona <> 1 Y zona <> 2 Entonces
        Escribir "Error: Zona no valida. Debe elegir 1 (Local) o 2 (Nacional)."
        datosValidos <- Falso
    FinSi
    
    // 2. Prioridad: Validación de Peso
    Si datosValidos Entonces
        Escribir "Ingrese el peso del paquete en kg (entre 0.1 y 30.0):"
        Leer peso
        Si peso < 0.1 O peso > 30.0 Entonces
            Escribir "Error: El peso esta fuera del rango permitido (0.1 a 30 kg)."
            datosValidos <- Falso
        FinSi
    FinSi
    
    // 3. Prioridad: Validación de Servicio
    Si datosValidos Entonces
        Escribir "Seleccione el Tipo de Servicio (1: Normal, 2: Expres):"
        Leer tipoServicio
        Si tipoServicio <> 1 Y tipoServicio <> 2 Entonces
            Escribir "Error: Tipo de servicio no valido. Debe elegir 1 (Normal) o 2 (Expres)."
            datosValidos <- Falso
        FinSi
    FinSi
    
    // 4. Prioridad: Validación de Cliente Frecuente
    Si datosValidos Entonces
        Escribir "El cliente es frecuente? (S/N):"
        Leer esFrecuenteRespuesta
        esFrecuenteRespuesta <- Mayusculas(esFrecuenteRespuesta)
        Si esFrecuenteRespuesta = "S" Entonces
            esClienteFrecuente <- Verdadero
        Sino
            Si esFrecuenteRespuesta = "N" Entonces
                esClienteFrecuente <- Falso
            Sino
                Escribir "Error: Respuesta invalida para cliente frecuente (Use S o N)."
                datosValidos <- Falso
            FinSi
        FinSi
    FinSi
    
    // 5. Prioridad: Procesamiento Matemático Mapeado
    Si datosValidos Entonces
        Si zona = 1 Entonces // LOCAL
            Si peso <= 5.0 Entonces
                Si tipoServicio = 1 Entonces tarifaBase <- 3.00 Sino tarifaBase <- 5.00 FinSi
            Sino
                Si peso <= 10.0 Entonces
                    Si tipoServicio = 1 Entonces tarifaBase <- 5.00 Sino tarifaBase <- 8.00 FinSi
                Sino
                    Si peso <= 20.0 Entonces
                        Si tipoServicio = 1 Entonces tarifaBase <- 8.00 Sino tarifaBase <- 12.00 FinSi
                    Sino
                        Si tipoServicio = 1 Entonces tarifaBase <- 12.00 Sino tarifaBase <- 18.00 FinSi
                    FinSi
                FinSi
            FinSi
        Sino // NACIONAL
            Si peso <= 5.0 Entonces
                Si tipoServicio = 1 Entonces tarifaBase <- 6.00 Sino tarifaBase <- 9.00 FinSi
            Sino
                Si peso <= 10.0 Entonces
                    Si tipoServicio = 1 Entonces tarifaBase <- 10.00 Sino tarifaBase <- 15.00 FinSi
                Sino
                    Si peso <= 20.0 Entonces
                        Si tipoServicio = 1 Entonces tarifaBase <- 16.00 Sino tarifaBase <- 24.00 FinSi
                    Sino
                        Si tipoServicio = 1 Entonces tarifaBase <- 24.00 Sino tarifaBase <- 35.00 FinSi
                    FinSi
                FinSi
            FinSi
        FinSi
        
        // Penalización por sobrepeso de 10kg
        Si peso > 10.0 Entonces
            recargoExceso <- (peso - 10.0) * 1.50
        FinSi
        
        subtotal <- tarifaBase + recargoExceso
        
        // Descuento Comercial Final
        Si esClienteFrecuente Entonces
            descuento <- subtotal * 0.10
        FinSi
        
        costoTotal <- subtotal - descuento
        
        Escribir ""
        Escribir "         FACTURA DIGITAL        "
        Escribir "--------------------------------------"
        Escribir "Tarifa Base Asignada:   $", tarifaBase
        Escribir "Recargo por Exceso:     $", recargoExceso
        Escribir "Subtotal:               $", subtotal
        Escribir "Descuento Aplicado:    -$", descuento
        Escribir "--------------------------------------"
        Escribir "COSTO TOTAL A PAGAR:    $", costoTotal
        Escribir "--------------------------------------"
    FinSi
FinAlgoritmo
```

---

## ☕ 4. Código Fuente (Java)

La codificación implementa aserciones lógicas y flujos secuenciales robustos utilizando defensas ante datos erróneos insertados en la consola.

```java
import java.util.Scanner;

public class EnvioPaquetes {
    public static void main(String[] args) {
        Scanner entrada = new Scanner(System.in);

        // Variables operacionales de estado primario
        int zona = 0;
        int tipoServicio = 0;
        double peso = 0.0;
        boolean esClienteFrecuente = false;
        boolean datosValidos = true;

        // Variables financieras de salida
        double tarifaBase = 0.0;
        double recargoExceso = 0.0;
        double subtotal = 0.0;
        double descuento = 0.0;
        double costoTotal = 0.0;

        System.out.println("=== SISTEMA DE CÁLCULO DE ENVÍOS DE PAQUETES ===");

        // P1: Evaluación Geográfica
        System.out.print("Seleccione la Zona del envío (1: Local, 2: Nacional): ");
        if (entrada.hasNextInt()) {
            zona = entrada.nextInt();
            if (zona != 1 && zona != 2) {
                System.out.println("Error: Zona no válida. Debe elegir 1 (Local) o 2 (Nacional).");
                datosValidos = false;
            }
        } else {
            System.out.println("Error: Formato de entrada no válido (Debe ser un número entero).");
            datosValidos = false;
        }

        // P2: Evaluación de Carga Física
        if (datosValidos) {
            System.out.print("Ingrese el peso del paquete en kg (entre 0.1 y 30.0): ");
            if (entrada.hasNextDouble()) {
                peso = entrada.nextDouble();
                if (peso < 0.1 || peso > 30.0) {
                    System.out.println("Error: El peso está fuera del rango permitido (0.1 a 30 kg).");
                    datosValidos = false;
