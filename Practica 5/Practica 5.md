# Práctica 5 - ADA

## Ejercicio 1
Se requiere modelar un puente de un único sentido que soporta hasta 5 unidades de peso. El peso de los vehículos depende del tipo: cada auto pesa 1 unidad, cada camioneta pesa 2 unidades y cada camión 3 unidades. Suponga que hay una cantidad innumerable de vehículos (A autos, B camionetas y C camiones). Analice el problema y defina qué tareas, recursos y sincronizaciones serán necesarios/convenientes para resolverlo. 

- a) Realice la solución suponiendo que todos los vehículos tienen la misma prioridad.
- b) Modifique la solución para que tengan mayor prioridad los camiones que el resto de los vehículos. 

### A
```ada
Procedure Puente1 is
    Task puente is
        Entry Auto(peso:IN int);
        Entry Camioneta(peso:IN int);
        Entry Camion(peso:IN int);
    End puente;

    Task type vehiculo;

    arrAutos: array (1..A) of vehiculo;
    arrCamioneta: array (1..B) of vehiculo;
    arrCamiones: array (1..C) of vehiculo;


    Task Body vehiculo is
    Begin
        if ("Es auto") then
            Puente.Auto(1);
        else if ("Es Camioneta") then
            Puente.Camioneta(2);
        else
            Puente.Camion(3);
        end if;
    End vehiculo;

    Task Body puente is
    Begin
        loop
            SELECT
                
        end loop;
    end puente;
End Puente1;
```