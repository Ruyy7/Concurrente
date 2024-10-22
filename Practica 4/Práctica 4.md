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

## Ejercicio 2
Se desea modelar el funcionamiento de un banco en el cual existen 5 cajas para realizar pagos. Existen P clientes que desean hacer un pago. Para esto, cada una selecciona la caja donde hay menos personas esperando; una vez seleccionada, espera a ser atendido. En cada caja, los clientes son atendidos por orden de llegada por los cajeros. Luego del pago, se les entrega un comprobante. Nota: maximizar la concurrencia.

```java
chan caja[5](id,pago)
chan comprobante[P](text)

Process Persona [id:1..N]{
    int cajaMin = //Obtener caja menos ocupada;
    text pago, comprobante;
    send caja[cajamin](id,pago);
    receive comprobante[id](comprobante);
}

Process Caja [id:1..5]{
    int idPersona;
    text pago, comprobante;
    while (true){
        receive caja[id](idPersona,pago);
        comprobante = //Generar comprobante;
        send comprobante[idPersona](comprobante);
    }
}
```

## Ejercicio 3
Se debe modelar el funcionamiento de una casa de comida rápida, en la cual trabajan 2 cocineros y 3 vendedores, y que debe atender a C clientes. El modelado debe considerar que:
- Cada cliente realiza un pedido y luego espera a que se lo entreguen.
- Los pedidos que hacen los clientes son tomados por cualquiera de los vendedores y se lo pasan a los cocineros para que realicen el plato. Cuando no hay pedidos para atender, los vendedores aprovechan para reponer un pack de bebidas de la heladera (tardan entre 1 y 3 minutos para hacer esto).
- Repetidamente cada cocinero toma un pedido pendiente dejado por los vendedores, lo cocina y se lo entrega directamente al cliente correspondiente.
**Nota**: maximizar la concurrencia.

```java
chan pedidos(int,text);
chan solcitudPedido(int);
chan siguiente [1..3](int,text);
chan preparacion(int,text);
chan pedidoHecho[C](text);

Process Cliente [idC:1..C]{
    text pedido,comida;
    send pedidos(id,pedido);
    receive pedidoHecho[idC](comida);
}

Process Vendedor [idV:1..3]{
    int idPedido;
    text pedido;
    while (true){
        send solcitudPedido(idV);
        receive siguiente[idV](idPedido,pedido);
        if (idPedido = 0){
            delay(60..180);
        }
        else{
            send preparacion(idPedido,pedido);
        }
    }
}

Process Coordinador {
    int idVendedor, idPedido;
    text pedido;
    while (true){
        receive solicitudPedido(idVendedor);
        if (empty(pedidos)){
            idPedido = 0;
        }
        else{
            receive pedidos(idPedido,pedido);
        }
        send siguiente[idVendedor](idPedido,pedido);
    }
}

Process Cocinero [idCocinero: 1..2]{
    int idPedido;
    text pedido, pedidoHecho;
    while (true)}{
        receive preparacion(idPedido,pedido);
        pedidoHecho = cocinar(idPedido,pedido);
        send pedidoHecho[idPedido](pedidoHecho);
    }
}
```
