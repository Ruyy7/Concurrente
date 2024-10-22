# Práctica 4 - PMA

## Ejercicio 1 
Suponga que N clientes llegan a la cola de un banco y que serán atendidos por sus empleados. Analice el problema y defina qué procesos, recursos y canales/comunicaciones serán necesarios/convenientes para resolverlo. Luego, resuelva considerando las siguientes situaciones:

- a. Existe un único empleado, el cual atiende por orden de llegada
- b. Ídem 1) pero considerando que hay 2 empleados para atender, ¿qué debe modificarse en la solución anterior?
- c. Ídem b) pero considerando que, si no hay clientes para atender, los empleados realizan tareas administrativas durante 15 minutos. ¿Se puede resolver sin usar procesos adicionales? ¿Qué consecuencias implicaría?

### A

```java
chan atencion(int);

Process Persona [id:1..N]{
    send atencion(id);
}

Process Empleado{
    int id;
    while (true){
        receive atencion(id);
        atender(id);
    }
}
```

### B
Es igual al anterior, no se debe modificar nada ya que se atiende sin discriminar que empleado atiende a una persona.
```java
chan atencion(int);

Process Persona [id:1..N]{
    send atencion(id);
}

Process Empleado[nroEmpleado:1..2]{
    int id;
    while (true){
        receive atencion(id);
        atender(id);
    }
}
```

### C
```java
chan atencion(int);
chan pedido(int);
chan siguiente[2](int);

Process Persona [id:1..N]{
    send atencion(id);
}

Process Empleado[nroEmpleado:1..2]{
    int id;
    while (true){
        send pedido(nroEmpleado);
        receive siguiente[nroEmpleado](id);
        if (id = 0){
            delay(900);
        }
        else{
            atender(id);
        }
    }
}

Process Coordinador{
    int idPedido;
    int respuesta;
    while (true){
        receive pedido(idPedido);
        if (empty(atencion)){
            respuesta = 0;
        }
        else{
            receive atencion(respuesta)
        }
        send siguiente[idPedido] (respuesta);
    }
}
```