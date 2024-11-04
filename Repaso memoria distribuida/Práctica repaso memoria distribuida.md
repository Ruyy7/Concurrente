# Práctica repaso memoria distribuida
## Pasaje de mensajes
## Ejercicio 1
En una oficina existen 100 empleados que envían documentos para imprimir en 5 impresoras compartidas. Los pedidos de impresión son procesados por orden de llegada y se asignan a la primera impresora que se encuentre libre:
- a) Implemente un programa que permita resolver el problema anterior usando PMA.
- b) Resuelva el mismo problema anterior pero ahora usando PMS.

### A
```java
chan impresiones(int,string);
chan documentoImpreso[100](string);
Process Persona [id:1..100]{
    String documento,resultado;
    while(true){
        documento.generar();
        send impresiones(id,documento);
        receive documentoImpreso[id](resultado)
    }
}

Process Impresora[id:1..5]{
    String documento,resultado;
    int idPersona;
    while (true){
        receive impresiones(idPersona,documento);
        resultado = //Imprimir documento;
        send documentoImpreso[idPersona](resultado);
    }
}
```

### B
```java
Process Persona[id:1..100]{
    String documento,resultado;
    while (true){
        documento.generar();
        Admin!enviarDocumento(id,documento);
        Impresora[*]?recibirResultado(resultado);
    }
}

Process Impresora[id:1..5]{
    String documento,resultado;
    int idPersona;
    while (true){
        Admin!pedido(id);
        Admin?siguiente(idPersona,documento);
        resultado = //Imprimir documento;
        Persona[idPersona]!recibirResultado(resultado);
    }
}

Process Admin{
    String documento;
    int idPersona,idImpresora;
    Cola impresiones;
    do
        Persona[*]?enviarDocumento(idPersona,documento) -> impresiones.push(idPersona,documento);
        not empty(impresiones); Impresora[*]?pedido(idImpresora) -> Impresora[idImpresora]!(impresiones.pop())
    od
}
```

## Ejercicio 2
Resolver el siguiente problema con PMS. En la estación de trenes hay una terminal de SUBE que debe ser usada por P personas de acuerdo con el orden de llegada. Cuando la persona accede a la terminal, la usa y luego se retira para dejar al siguiente. **Nota**: cada Persona usa sólo una vez la terminal.

```java
Process Persona[id:1..P}{
    Terminal!llegue(id);
    Terminal?usar();
    //Usando terminal;
    Terminal!retirarse();
}

Process Terminal{
    int idPersona;
    Cola personas;
    boolean libre = true;
    do
        Persona[*]?llegue(idPersona) -> if (libre){
                                            libre = false;
                                            Persona[idPersona]!usar();
                                        }
                                        else{
                                            personas.push(idPersona);
                                        }
        Persona[*]retirarse -> if (empty(personas)){
                                    libre = true;
                                }
                                else{
                                    Persona[personas.pop()]!usar();
                                }
    od
}
```

### Ejercicio 3
Resolver el siguiente problema con PMA. En un negocio de cobros digitales hay P personas que deben pasar por la única caja de cobros para realizar el pago de sus boletas. Las personas son atendidas de acuerdo con el orden de llegada, teniendo prioridad aquellos que deben pagar menos de 5 boletas de los que pagan más. Adicionalmente, las personas embarazadas tienen prioridad sobre los dos casos anteriores. Las personas entregan sus boletas al cajero y el dinero de pago; el cajero les devuelve el vuelto y los recibos de pago.

```java
chan cajaNormal(int);
chan cajaPrioritaria(int);
chan cajaEmbarazadas(int);
chan personas[P](double,String);
chan hayPedido();

Process Persona[id:1..P]{
    int numeroBoletas = //Tiene x boletas;
    boolean embarazada = //Es o no es una embarazada;
    Boletas[x] boletas;
    double dinero,vuelto;
    String recibo;
    if (embarazada){
        send cajaEmbarazadas(id,boletas,dinero);
    }
    else if (numeroBoletas < 5){
        send cajaPrioritaria(id,boletas,dinero);
    }
    else{
        send cajaNormal(id,boletas,dinero);
    }
    send hayPedido();
    receive personas[id](vuelto,recibo);
}

Process Cajero{
    int idPersona;
    Boletas[x] boletas;
    double dinero,vuelto;
    String recibo;
    while (true){
        receive hayPedido();
        if (not empty(cajaEmbarazadas)){
            receive cajaEmbarazadas(idPersona,boletas,dinero);
        }
        else if (not empty(cajaPrioritaria)){
            receive cajaPrioritaria(idPersona,boletas,dinero);
        }
        else if (not empty(cajaNormal)){
            receive cajaNormal(idPersona,boletas,dinero);
        }
        vuelto = //Calcular vueto;
        recibo = //Imprimir recibo;
        send personas[idPersona](vuelto,recibo);
    }
}
```

## ADA
## Ejercicio 1
Resolver el siguiente problema. La página web del Banco Central exhibe las diferentes cotizaciones del dólar oficial de 20 bancos del país, tanto para la compra como para la venta. Existe una tarea programada que se ocupa de actualizar la página en forma periódica y para ello consulta la cotización  de cada uno de los 20 bancos. Cada banco dispone de una API, cuya única función es procesar las solicitudes de aplicaciones externas. La tarea programada consulta de a una API por vez, esperando a lo sumo 5 segundos por su respuesta. Si pasado ese tiempo no respondió, entonces se mostrará vacía la información de ese banco.

```ada
Procedure Cotizaciones
    Task aplicacion;

    Task type Banco is
        Entry solicitarInformacion(informacion:OUT String);
    End Banco;
    Bancos : array (1..20) of Banco;

    Task body Banco is
    Begin
        loop
            accept solicitarInformacion(informacion:OUT String) do
                informacion = cotizacionDolar();
            end solicitarInformacion;
        end loop;
    End Banco;

    Task body aplicacion is
        informacionBanco: array (1..20) of String;
        informacion:String;
    Begin
        loop
            for (int i=1 to 20) loop
                SELECT
                    Banco(i).solicitarInformacion(informacion);
                    informacionBanco(i):= informacion;
                OR DELAY 5
                    informacionBanco(i):= "No data";
                END SELECT;
            end loop;
            mostrar(informacionBanco);
        end loop;
    end aplicacion;
Begin
    null;
End Cotizaciones;
```

## Ejercicio 2
Resolver el siguiente problema. En un negocio de cobros digitales hay P personas que deben pasar por la única caja de cobros para realizar el pago de sus boletas. Las personas son atendidas de acuerdo con el orden de llegada, teniendo prioridad aquellos que deben pagar menos de 5 boletas de los que pagan más. Adicionalmente, las personas ancianas tienen prioridad sobre los dos casos anteriores. Las personas entregan sus boletas al cajero y el dinero de pago; el cajero les devuelve el vuelto y los recibos de pago.

```ada
Procedure negocio
    Task type persona;
    personas: array (1..20) of persona;

    Task cajero is 
        Entry cajaNormal (boletas:IN Boletas[*], dinero:IN real, vuelto:OUT real, recibo:OUT string);
        Entry cajaPrioritaria;
        Entry cajaAncianos;
    end cajero;

    Task body persona is
        boletas: Boletas[*];
        dinero,vuelto: real;
        recibo:string;
    Begin
        if ("Es anciano") then
            cajero.cajaAncianos (boletas,dinero,vuelto,recibo);
        else if (Boletas[*] < 5) then
            cajero.cajaPrioritaria (boletas,dinero,vuelto,recibo);
        else
            cajero.cajaNormal (boletas,dinero,vuelto,recibo);
    End persona;

    Task body cajero is

    Beging
        loop
            SELECT
                accept cajaAncianos (boletas:IN Boletas[*], dinero:IN real, vuelto:OUT real, recibo:OUT string) do
                    vuelto = calcularVuelto(dinero);
                    recibo = //Imprimir recibo;
                end cajaAncianos;
            OR
                when (cajaAncianos'count = 0) =>
                    accept cajaPrioritaria (boletas:IN Boletas[*], dinero:IN real, vuelto:OUT real, recibo:OUT string) do
                        vuelto = calcularVuelto(dinero);
                        recibo = //Imprimir recibo;
                    end cajaPrioritaria;
            OR
                when (cajaAncianos'count = 0 and cajaPrioritaria'count = 0) =>
                    accept cajaNormal (boletas:IN Boletas[*], dinero:IN real, vuelto:OUT real, recibo:OUT string) do
                        vuelto = calcularVuelto(dinero);
                        recibo = //Imprimir recibo;                       
                    end cajaNormal;
            END SELECT
        end loop;
    End cajero;
Begin
    null;
End negocio;
```

## Ejercicio 3
Resolver el siguiente problema. La oficina central de una empresa de venta de indumentaria debe calcular cuántas veces fue vendido cada uno de los artículos de su catálogo. La empresa se compone de 100 sucursales y cada una de ellas maneja su propia base de datos de ventas. La oficina central cuenta con una herramienta que funciona de la siguiente manera: ante la consulta realizada para un artículo determinado, la herramienta envía el identificador del artículo a las sucursales, para que cada una calcule cuántas veces fue vendido en ella. Al final del procesamiento, la herramienta debe conocer cuántas veces fue vendido en total, considerando todas las sucursales. Cuando ha terminado de procesar un artículo comienza con el siguiente (suponga que la herramienta tiene una función generarArtículo() que retorna el siguiente ID a consultar). Nota: maximizar la concurrencia. Existe una función ObtenerVentas(ID) que retorna la cantidad de veces que fue vendido el artículo con identificador ID en la base de la sucursal que la llama.

```ada
Procedure Empresa is
    Task type sucursal is
        Entry ident(idSucursal:IN int);
        Entry continuar;
    end sucursal;
    sucursales: array (1..100) of sucursal;

    Task herramienta is
        Entry enviarArticulo(idA:OUT int);
        Entry ventasSucursalArticulo(cant:IN int);
    End herramienta;

    Task body sucursal is
        id,articuloId,cantVentasArticulo:int;
    Begin
        accept ident(idSucursal:IN int) do
            id:=idSucursal;
        end ident;
        loop
            herramienta.enviarArticulo(articuloId);
            cantVentasArticulo:= ObtenerVentas(articuloId);
            herramienta.ventasSucursalArticulo(id,cantVentasArticulo);
            accept continuar;
        end loop;

    End sucursal;

    Task body herramienta is
        cantArticuloSucursal: array (1..100) of int;
        articuloId,cantTotal:int;
    Begin
        loop
            articuloId:= generarArtículo();
            cantTotal:= 0;
            for (i in 1..200) loop
                SELECT
                    accept enviarArticulo(idA:OUT int) do
                        idA:=articuloId;
                    end enviarArticulo;
                OR
                    accept ventasSucursalArticulo(idSucursal:IN int,cant:IN int) do
                        cantArticuloSucursal[idSucursal]:= cant;
                    end ventasSucursalArticulo;
                END SELECT;
            end loop;
            for (i in 1..100) loop
                writeln("Cantidad de ventas del artículo ",articuloId," en sucursal ",i," ",cantArticuloSucursal(i));
                cantTotal += cantArticuloSucursal(i);
            end loop;
            writeln("Cantidad total de ventas del artículo ",articuloId,": ",cantTotal);
            for (i in 1..100) loop
                sucursales(i).continuar;
            end loop;
        end loop;
        <!-- Supuse que se imprimen los valores de todas las sucursales + el total del articulo y luego continua el ciclo como el EJ8 de la práctica -->
    End herramienta;

Begin
    for (i in 1..100) loop
        sucursal.ident(i);
    end loop;
End Empresa;
```