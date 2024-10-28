# Práctica 4 - PMS

## Ejercicio 1
Suponga que existe un antivirus distribuido que se compone de R procesos robots Examinadores y 1 proceso Analizador. Los procesos Examinadores están buscando continuamente posibles sitios web infectados; cada vez que encuentran uno avisan la dirección y luego continúan buscando. El proceso Analizador se encarga de hacer todas las pruebas necesarias con cada uno de los sitios encontrados por los robots para determinar si están o no infectados.

- a)Analice el problema y defina qué procesos, recursos y comunicaciones serán necesarios/convenientes para resolverlo.
- b)Implemente una solución con PMS sin tener en cuenta el orden de los pedidos.
- c)Modifique el inciso (b) para que el Analizador resuelva los pedidos en el orden en que se hicieron. 

### A y B
```java
Process Examinador [id:1..R]{
text url;
    while (true){
        url = //Buscar sitio web infectado;
        Analizador!pedido(url);
    }
}

Process Analizador {
    text url;
    while (true){
        Examinador[*]?pedido(url);
    }
}
```

### C
```java
Process Examinador [id:1..R]{
text url;
    while (true){
        url = //Buscar sitio web infectado;
        Admin!pedido(url);
    }
}

Process Admin{
    text url;
    Cola links;
    do 
        Examinador[*]?pedido(url) -> links.push(url);
        not empty(links); Analizador?solicitud() -> Analizador!reporte(links.pop());
    od
}

Process Analizador {
    text url;
    while (true){
        Admin!solicitud();
        Admin?reporte(url);
        url = //Analizando URL.
    }
}
```

## Ejercicio 2
En un laboratorio de genética veterinaria hay 3 empleados. El primero de ellos continuamente prepara las muestras de ADN; cada vez que termina, se la envía al segundo empleado y vuelve a su trabajo. El segundo empleado toma cada muestra de ADN preparada, arma el set de análisis que se deben realizar con ella y espera el resultado para archivarlo. Por último, el tercer empleado se encarga de realizar el análisis y devolverle el resultado al segundo empleado.

```java
Process Empleado1{
    ADN muestra;
    while (true){
        muestra = //Preparar muestra;
        Admin!enviarMuestra(muestra);
    }
}

Process Empleado2{
    ADN muestra;
    Set set;
    text resultado;
    Cola resultaos;
    while (true){
        Admin!pedido();
        Admin?recibirMuestra(muestra);
        set = muestra.armarSet();
        Empleado3!enviarSet(set);
        Empleado3?obtenerResultado(resultado);
        resultados.push(resultado); // Archiva el resultado, no tiene uso especial.
    }
}

Process Empleado3{
    Set set;
    text resultado;
    while(true){
        Empleado2?enviarSet(set);
        resultado = set.analizar();
        Empleado2!obtenerResultado(resultado);
    }
}

Process Admin{
    ADN muestra;
    Cola muestras;
    do
        Empleado1?enviarMuestra(muestra) -> muestras.push(muestra);
        not empty(muestras); Empleado2?pedido() -> Empleado2!recibirMuestra(muestras.pop());
    od
}
```