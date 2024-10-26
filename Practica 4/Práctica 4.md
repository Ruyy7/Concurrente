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

## Ejercicio 4
Simular la atención en un locutorio con 10 cabinas telefónicas, el cual tiene un empleado que se encarga de atender a N clientes. Al llegar, cada cliente espera hasta que el empleado le indique a qué cabina ir, la usa y luego se dirige al empleado para pagarle. El empleado atiende a los clientes en el orden en que hacen los pedidos. A cada cliente se le entrega un ticket factura por la operación.

### A
Implemente una solución para el problema descrito.

```java
chan pedido(int);
chan cabinaDesignada[1..N](int);
chan usoCabinas[1..10](int);
chan ticket[1..N](text);

Process Cabina [idCabina: 1..10]{
    int idCliente;
    text ticket;
    while (true){
        receive usoCabinas[idCabina](idCliente);
        ticket = Cobrar(idCliente);
        send ticket[idCliente](ticket);
    }
}

Process Empleado{
    int idCliente, cabinaUsar;
    for (int i = 1 to N){
        receive pedido(idCliente);
        cabinaUsar = //Obtener cabina menos ocupada;
        send cabinaDesignada[idCliente](cabinaUsar);
    }
}

Process Cliente [idCliente: 1..N]{
    int cabinaUsar;
    text factura;
    send pedido(idCliente);
    receive cabinaDesignada[idCliente](cabinaUsar);
    send usoCabinas[cabinaUsar](idCliente);
    receive ticket[idCliente](factura);
}
```

### B
Modifique la solución implementada para que el empleado dé prioridad a los que terminaron de usar la cabina sobre los que están esperando para usarla.

## Ejercicio 5
Resolver la administración de 3 impresoras de una oficina. Las impresoras son usadas por N administrativos, los cuales están continuamente trabajando y cada tanto envían documentos a imprimir. Cada impresora, cuando está libre, toma un documento y lo imprime, de acuerdo con el orden de llegada.

### A
Implemente una solución para el problema descrito. 

```java
chan mailAdministrativo(text);

Process Administrativo [idA: 1..N]{
    text documentos;
    while (true){
        send mailAdministrativo(documentos);
    }
}

Process Impresora [idP: 1..3]{
    text documentoImprimir;
    while (true){
        receive mailAdministrativo(documentoImprimir);
        imprimir(documentoImprimir);
    }
}
```

### B
Modifique la solución implementada para que considere la presencia de un director de oficina que también usa las impresas, el cual tiene prioridad sobre los administrativos.

```java
chan mailAdministrativo(text);
chan mailDirector(text);
chan pedido(int);
chan siguiente[1..3](text);

Process Administrativo [idA: 1..N]{
    text documentos;
    while (true){
        send mailAdministrativo(documentos);
    }
}

Process Director{
    text documentos;
    while (true){
        send mailDirector(documentos);
    }
}

Process Coordinador{
    int idPrinter;
    text documento;
    while (true){
        receive pedido(idPrinter);
        if (!empty(mailDirector)){
            receive mailDirector(documento);
        }
        else if (!empty(mailAdministrativo)){
            receive mailAdministrativo(documento);
        }
        else{
            documento = "Nada";
        }
        send siguiente[idPrinter](documento);
    }
}

Process Impresora [idP: 1..3]{
    text documentoImprimir;
    while (true){
        send pedido(idP);
        receive siguiente(documentoImprimir);
        if (documentoImprimir != "Nada"){
            imprimir(documentoImprimir);
        }
    }
}
```

### C
Modifique la solución (a) considerando que cada administrativo imprime 10 trabajos y que todos los procesos deben terminar su ejecución.

```java
chan mailAdministrativo(text);

Process Administrativo [idA: 1..N]{
    text documentos;
    for (int i = 1 to 10){
        send mailAdministrativo(documentos);
    }
}

Process Impresora [idP: 1..3]{
    text documentoImprimir;
    int totalImpresiones = 10 * N;
    for (int i = 1 to totalImpresiones){
        receive mailAdministrativo(documentoImprimir);
        imprimir(documentoImprimir);
    }
}
```

### D
Modifique la solución (b) considerando que tanto el director como cada administrativo imprimen 10 trabajos y que todos los procesos deben terminar su ejecución.

```java
chan mailAdministrativo(text);
chan mailDirector(text);
chan pedido(int);
chan siguiente[1..3](text);
chan finDirector(bool);
chan finAdministrativo(int);

Process Administrativo [idA: 1..N]{
    text documentos;
    for (int i = 1 to 10){
        send mailAdministrativo(documentos);
    }
    send finAdministrativo(1);
}

Process Director{
    text documentos;
    for (int i = 1 to 10){
        send mailDirector(documentos);
    }
    send finDirector(true);
}

Process Coordinador{
    int idPrinter;
    text documento;
    bool ok = true;
    bool director = false;
    totalAdministrativos = 0;
    int intaux = 0;
    while (ok){
        receive pedido(idPrinter);
        if (!empty(mailDirector)){
            receive mailDirector(documento);
            if (!empty(finDirector)){
                receive finDirector(director);
            }
        }
        else if (!empty(mailAdministrativo)){
            receive mailAdministrativo(documento);
            while (!empty(finAdministrativo)){
                totalAdministrativos += receive finAdministrativo(intaux);
            }
        }
        else{
            documento = "Nada";
        }
        if (director && totalAdministrativos == N){
            ok = false;
        }
        send siguiente[idPrinter](documento);
    }
    send siguiente[idPrinter]("");
}

Process Impresora [idP: 1..3]{
    text documentoImprimir;
    bool ok = true;
    while (ok){
        send pedido(idP);
        receive siguiente(documentoImprimir);
        if (documentoImprimir != "Nada"){
            if (documentoImprimir == ""){
                ok = false;
            } else{
                imprimir(documentoImprimir);
            }
        }
    }
}
```