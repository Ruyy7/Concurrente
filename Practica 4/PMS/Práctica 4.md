# Práctica 4 - PMS

## Ejercicio 1
Suponga que existe un antivirus distribuido que se compone de R procesos robots Examinadores y 1 proceso Analizador. Los procesos Examinadores están buscando continuamente posibles sitios web infectados; cada vez que encuentran uno avisan la dirección y luego continúan buscando. El proceso Analizador se encarga de hacer todas las pruebas necesarias con cada uno de los sitios encontrados por los robots para determinar si están o no infectados.

- a) Analice el problema y defina qué procesos, recursos y comunicaciones serán necesarios/convenientes para resolverlo.
- b) Implemente una solución con PMS sin tener en cuenta el orden de los pedidos.
- c) Modifique el inciso (b) para que el Analizador resuelva los pedidos en el orden en que se hicieron. 

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

## Ejercicio 3
En un examen final hay N alumnos y P profesores. Cada alumno resuelve su examen, lo entrega y espera a que alguno de los profesores lo corrija y le indique la nota. Los profesores corrigen los exámenes respetando el orden en que los alumnos van entregando.

- a) Considerando que P=1.
- b) Considerando que P>1.
- c) Ídem b) pero considerando que los alumnos no comienzan a realizar su examen hasta que todos hayan llegado al aula.

**Nota**: maximizar la concurrencia; no generar demora innecesaria; todos los procesos deben terminar su ejecución.

### A
```java
Process Alumno[idA:1..N]{
    Text examen;
    int nota;
    examen.realiar();
    Admin!entregarExamen(idA,examen);
    Profresor?recibirNota(nota);
}

Process Admin{
    Text examen;
    int idA;
    Cola examenes;
    do 
        Alumno?entregarExamen(idA,examen) -> examenes.push(idA,examen);
        not empty(examenes); Profesor?pedido() -> Profesor!recibirExamen(examenes.pop());
    od
}

Process Profesor{
    int idA,nota;
    Text examen;
    for (int i=1 to N){
        Admin!pedido();
        Admin?recibirExamen(idA,examen);
        nota = examen.corregir();
        Alumno[idA]!recibirNota(nota);
    }
}
```

### B
```java
Process Alumno[idA:1..N]{
    Text examen;
    int nota;
    examen.realiar();
    Admin!entregarExamen(idA,examen);
    Profresor[*]?recibirNota(nota);
}

Process Admin{
    Text examen;
    int idA,idP;
    Cola examenes;
    do 
        Alumno?entregarExamen(idA,examen) -> examenes.push(idA,examen);
        not empty(examenes); Profesor[*]?pedido(idP) -> Profesor[idP]!recibirExamen(examenes.pop());
    od
}

Process Profesor[idP:1..P]{
    int idA,nota;
    Text examen;
    for (int i=1 to N){ // Tengo que ver como hacer que finalice de manera eficiente el profesor.
        Admin!pedido(id);
        Admin?recibirExamen(idA,examen);
        nota = examen.corregir();
        Alumno[idA]!recibirNota(nota);
    }
}
```

### C
```java
Process Alumno[idA:1..N]{
    Text examen;
    int nota;

    Admin!llegaAlumno();
    Admin?comienzaExamen();
    examen.realiar();
    Admin!entregarExamen(idA,examen);
    Profresor[*]?recibirNota(nota);
}

Process Admin{
    Text examen;
    int idA,idP;
    Cola examenes;

    for (int i=1 to N){
        Alumno[*]?llegaAlumno();
    }
    for (int i=1 to n){
        Alumno[i]!comienzaExamen();
    }

    do 
        Alumno?entregarExamen(idA,examen) -> examenes.push(idA,examen);
        not empty(examenes); Profesor[*]?pedido(idP) -> Profesor[idP]!recibirExamen(examenes.pop());
    od
}

Process Profesor[idP:1..P]{
    int idA,nota;
    Text examen;
    for (int i=1 to N){ // Tengo que ver como hacer que finalice de manera eficiente el profesor.
        Admin!pedido(id);
        Admin?recibirExamen(idA,examen);
        nota = examen.corregir();
        Alumno[idA]!recibirNota(nota);
    }
}
```

## Ejercicio 4
En una exposición aeronáutica hay un simulador de vuelo (que debe ser usado con exclusión mutua) y un empleado encargado de administrar su uso. Hay P personas que esperan a que el empleado lo deje acceder al simulador, lo usa por un rato y se retira.

- a) Implemente una solución donde el empleado sólo se ocupa de garantizar la exclusión mutua.
- b) Modifique la solución anterior para que el empleado considere el orden de llegada para dar acceso al simulador. 

**Nota**: cada persona usa sólo una vez el simulador.

### A
```java
Process Persona[idP:1..P]{
    Empleado!acceder(idP);
    Empleado?pasar();
    //Usando simulador;
    Empleado!retirarse();
}

Process Empleado{
    int idP;
    while(true){
        Persona[*]?acceder(idP);
        Persona[idP]!pasar();
        Persona[*]?retirarse(); //Capaz puedo pasar el idP, creo que es lo mismo.
    }
}
```

### B
```java
Process Persona[idP:1..P]{
    Empleado!acceder(idP);
    Empleado?pasar();
    //Usando simulador;
    Empleado!retirarse();
}

Process Empleado{
    int idP;
    Cola personas;
    boolean libre;
    do
        Persona[*]?acceder(idP) --> if (libre){
                                        libre = false;
                                        Persona[idP]!pasar();
                                    }
                                    else{
                                        personas.push(idP);
                                    }
        Persona[*]?retirarse() --> if (empty(pesronas)){
                                    libre = true;
                                }
                                else{
                                    Persona[personas.pop()]!pasar();
                                }
    od
}
```

## Ejercicio 5
En un estadio de fútbol hay una máquina expendedora de gaseosas que debe ser usada por E Espectadores de acuerdo con el orden de llegada. Cuando el espectador accede a la máquina en su turno usa la máquina y luego se retira para dejar al siguiente. 
**Nota**: cada Espectador una sólo una vez la máquina.

```java
Process Espectador[idP:1..P]{
    Maquina!acceder(idP);
    Maquina?pasar();
    //Usando maquina;
    Maquina!retirarse();
}

Process Maquina{
    int idP;
    Cola espectadores;
    boolean libre;
    do
        Espectador[*]?acceder(idP) --> if (libre){
                                        libre = false;
                                        Espectador[idP]!pasar();
                                    }
                                    else{
                                        espectadores.push(idP);
                                    }
        Espectador[*]?retirarse() --> if (empty(espectadores)){
                                    libre = true;
                                }
                                else{
                                    Espectador[espectadores.pop()]!pasar();
                                }
    od
}
```