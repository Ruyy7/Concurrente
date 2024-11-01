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
    receive personas[id](vuelto,recibo);
}

Process Cajero{
    int idPersona;
    Boletas[x] boletas;
    double dinero,vuelto;
    String recibo;
    while (true){
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
