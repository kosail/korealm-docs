# Ejercicios
Te dejo 3 ejercicios o problemas a resolver. Los tres son ejercicios cortos para no quitarte demasiado tiempo, pero lo suficientemente pensados como para que sí sirvan de algo.

Los primeros dos son cortos y simples, y el tercero es un poquito más elaborado. En orden, son estos:
1. Records
2. Pattern Matching
3. Lambdas y Streams

## Ejercicio de records: La flota gatuna

Para que no te quedes solo con la teoría, abre tu terminal, mete `jshell` y dale enter, y pegale lo siguiente (o crea un archivo `FlotaGatuna.java` y lo corres manual). Imagina que estás a cargo de la logística de una flota de gatos espaciales traficantes de hierba para gato (catnip, pues). Tenemos dos conceptos básicos:

```java
record Pistola(String modelo, int potencia) {}

record ComandanteGato(String nombre, String rango, Pistola arma, int catnipGramos) {
    public ComandanteGato {
        if (catnipGramos < 0) {
            throw new IllegalArgumentException("Un comandante no puede deber catnip.");
        }
    }
}
```

Ahora sí, tu trabajo:

1. Crea dos instancias independientes con los mismos datos:
   ```java
   ComandanteGato c1 = new ComandanteGato("Michi", "Capitán", new Pistola("Laser", 120), 30);
   ComandanteGato c2 = new ComandanteGato("Michi", "Capitán", new Pistola("Laser", 120), 30);
   ```
   Prueba `c1 == c2` contra `c1.equals(c2)`. ¿Qué te dan ambos de salida?

2. El capitán Michi se fumó 10 gramos de catnip. No existe el método `setCatnipGramos`. ¿Cómo le harías para actualizar los gramos?
3. Intenta instanciar un comandante con `-5` gramos de catnip.
4. Haz `System.out.println(c1);`. Mira como ya no imprime `ComandanteGato@4f023ac`, ¿qué te imprime?

## Ejercicio de pattern matching: Sistema de Soporte

Crea un archivo `SistemaSoporte.java`.

La idea es simple: recibes un `Object` que representa algo que llegó a la mesa de soporte (un ticket, un usuario, una alerta, etc.) y devuelves un `String` describiendo qué hacer con él.


```java
public class SistemaSoporte {

    record Ticket(int id, String prioridad, int horasAbierto, boolean asignado) {}
    record Usuario(String nombre, int nivel, boolean premium) {}
    record Alerta(String servicio, int severidad, boolean resuelta) {}
    record Equipo(String nombre, int miembros) {}

    public static String procesar(Object entrada) {
        // TODO: IMPLEMENTA EL PATTERN MATCHING AQUI
        throw new UnsupportedOperationException("Implementa tu switch aquí");
    }

    public static void main(String[] args) {
        var entradas = new Object[] {
            new Ticket(101, "Crítica", 72, false),
            new Ticket(102, "Alta", 5, true),
            new Ticket(103, "Baja", 1, false),
            new Ticket(104, "Media", 48, true),
            new Alerta("Base de datos", 3, false),
            new Alerta("Cache", 1, true),
            new Usuario("Ana", 5, true),
            new Usuario("Luis", 2, false),
            new Equipo("Backend", 6),
            null
        };

        for (var e : entradas) {
            System.out.println(procesar(e));
        }
    }
}
```

Instrucciones:

1. Tu método recibe un `Object` entrada y debe retornar un `String`. Usa un `return switch (entrada) { ... };` con las siguientes reglas:
2. Ticket crítico sin asignar: Si la prioridad es "Crítica" y no está asignado, avisa que debe escalarse de inmediato a un ingeniero senior.
3. Ticket abierto demasiado tiempo: Si lleva 48 horas o más abierto, avisa que está por incumplir los reglamentos de la empresa.
4. Ticket de prioridad alta ya asignado: Si la prioridad es "Alta" y ya está asignado, avisa que va por buen camino.
5. Cualquier otro ticket: Avisa que está en cola normal con sus horas abiertas.
6. Alerta no `resuelta`: Si `resuelta` es false, avisa que hay que notificar al equipo de guardia según su severidad.
7. Alerta ya `resuelta`: Si no, avisa que se cerró correctamente.
8. Usuario `premium`: Si es `premium`, avisa que su ticket tiene atención prioritaria.
9. Usuario estándar: Si no, avisa que se atenderá en orden de llegada según su nivel.
10. Equipo: Avisa cuántos miembros tiene y que se le asignará la carga correspondiente.
11. case `null`: La entrada se perdió o llegó corrupta.
12. default: Para cualquier otro objeto no reconocido por el sistema.


**Nota 1:** Recuerda que el orden de los casos es importante. El compilador de Java evaluará los casos en el orden en que aparecen. Si un caso más general aparece antes que uno más específico, el compilador lanzará un error. 

**Nota 2:**
En el Java viejo, meter un null en un switch lanzaba NullPointerException de inmediato. Con el switch moderno, case null es un caso más. Ya no explota.

## Ejercicio de lambdas y streams: Stella-1

Trabajas para la NASA, y reciéntemente te pasaron al departamento a cargo del satélite Stella-1. El satélite manda cada tanto una lista de lecturas a la estación de control, y tu trabajo es procesar la telemetría con streams.

Guarda el código en un archivo `Stella.java`. Puedes usar `for` en `main` para imprimir, pero cada método debe resolver su parte con un stream.

```java
import java.util.*;
import java.util.stream.Collectors;

public class Stella {

    record Lectura(String modulo, String sensor, int valor, boolean critica) {}

    public static void main(String[] args) {
        var lecturas = List.of(
            new Lectura("energia", "panel-solar", 82, false),
            new Lectura("energia", "bateria", 17, true),
            new Lectura("termico", "cpu", 91, true),
            new Lectura("termico", "cabina", 22, false),
            new Lectura("termico", "radiador", 110, true),
            new Lectura("navegacion", "estrella-guia", 12, true),
            new Lectura("navegacion", "gyro", 48, false),
            new Lectura("comunicaciones", "antena", 73, false),
            new Lectura("comunicaciones", "laser", 3, true),
            new Lectura("comunicaciones", "transponder", 55, false)
        );

        System.out.println("Alertas: " + alertasCriticas(lecturas));
        System.out.println("Modo seguro: " + hayEmergencia(lecturas));
        System.out.println("Prioridad: " + prioridad(lecturas));
        System.out.println("Criticas: " + contarCriticas(lecturas));
        System.out.println("Modulos afectados: " + modulosAfectados(lecturas));
    }

    static List<String> alertasCriticas(List<Lectura> lecturas) {
        // TODO
        return List.of();
    }

    static boolean hayEmergencia(List<Lectura> lecturas) {
        // TODO
        return false;
    }

    static List<String> prioridad(List<Lectura> lecturas) {
        // TODO
        return List.of();
    }

    static long contarCriticas(List<Lectura> lecturas) {
        // TODO
        return 0;
    }

    static List<String> modulosAfectados(List<Lectura> lecturas) {
        // TODO
        return List.of();
    }
}
```

Tu trabajo:

1. **Alertas críticas:** devuelve una lista con `"modulo/sensor=valor"` por cada lectura crítica, en el orden original.
2. **¿Modo seguro?:** devuelve `true` si alguna lectura crítica tiene un valor mayor que `90`.
3. **Prioridad de atención:** devuelve los nombres de los sensores críticos ordenados de mayor a menor valor.
4. **Conteo:** devuelve cuántas lecturas críticas hay.
5. **Módulos afectados:** devuelve los módulos que tienen al menos una lectura crítica, sin repetir y en orden alfabético.

Cuando termines, la salida debe verse así:

```text
Alertas: [energia/bateria=17, termico/cpu=91, termico/radiador=110, navegacion/estrella-guia=12, comunicaciones/laser=3]
Modo seguro: true
Prioridad: [radiador, cpu, bateria, estrella-guia, laser]
Criticas: 5
Modulos afectados: [comunicaciones, energia, navegacion, termico]
```

**Pistas:**
Este es el ejercicio más complicado de los tres, por lo que te dejo algunas ayudas.

- Existen funciones como `.distinct()`, `.sorted()` que su propio nombre dice para qué sirven.
- `filter` y `map` son operaciones intermedias; `toList`, `count` y `anyMatch` son terminales.
- `anyMatch` termina en cuanto encuentra una coincidencia, no necesita revisar todas las lecturas.
- Para ordenar por valores: `Comparator.comparingInt(Lectura::valor).reversed()`.

**Reto extra:** usa `Collectors.groupingBy` para contar cuántas lecturas hay por módulo. El orden del mapa no importa.

```java
static Map<String, Long> lecturasPorModulo(List<Lectura> lecturas) {
    // TODO: groupingBy + counting
    return Map.of();
}
```



**Prueba algo más:** ¿qué pasa si quitas la operación terminal de `alertasCriticas`? ¿Por qué `anyMatch` puede responder sin revisar las diez lecturas?
