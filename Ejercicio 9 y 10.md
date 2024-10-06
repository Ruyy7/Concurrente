# Ejercicio 9
En un examen de la secundaria hay un preceptor y una profesora que deben tomar un examen 
escrito a 45 alumnos. El preceptor se encarga de darle el enunciado del examen a los alumnos 
cundo los 45 han llegado (es el mismo enunciado para todos). 
- ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png) `Como el preceptor le da el enunciado a los alumnos cuando llegaron todos, lo ideal sería que esa sincronización ocurra directamente en el monitor de llegada (Aula).`
La profesora se encarga de ir corrigiendo los exámenes de acuerdo con el orden en que los alumnos van entregando. Cada  alumno al llegar espera a que le den el enunciado, resuelve el examen, y al terminar lo deja para que la profesora lo corrija y le envíe la nota.
- ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png) `En el monitor Examen solo ocurre la sincronización entre alumnos entregando el examen y la profesora corrigiéndolos (y devolviendo la nota al alumno).`
Nota: maximizar la concurrencia; todos los procesos deben terminar su ejecución; suponga que la profesora tiene una función corregirExamen que recibe un examen y devuelve un entero con la nota. 


- ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png) `El proceso Preceptor hace dos invocaciones sucesivas al monitor Examen (uno haciendo una barrera y otro solo para que se setee el enunciado). Esto seguro que se podía (y es mejor) resolver en una sola invocación ya que no hay ningún procesamiento activo en el medio.`
```java
Process Alumno [id:1..45]
    String examen; int nota;
    Aula.llegar();
    Aula.solicitarEnunciado(examen);
    // Alumno resuelve examen
    Examen.devolucionNota(examen,nota);
End Process;

Process Preceptor
    String enunciado;
    Aula.repatir(enunciado);
End Process;

Process Profesora
    int idA;
    for (int i= 1 to 45){
        Examen.siguiente(idA,examen);
        int nota = corregirExamen(examen);
        Examen.entregarNota(idA,nota);
    }
End Process;

Monitor Aula
    int presentes = 0;
    cond alumno; cond preceptor;
    String enunciado;

    Procedure llegar(){
        presentes++;
        if (presentes == 45){
            signal(preceptor);
            wait(alumno);
        }
        else{
            wait(alumno);
        }
    }

    Procedure repartir(String in enunciadoExamen){
        wait(preceptor);
        enunciado = enunciadoExamen;
        signal_all(alumno);
    }

    Procedure solicitarEnunciado(String out enunciadoExamen){
        enunciadoExamen = enunciado;
    }


End Monitor;

Monitor Examen
    cond preceptor; cond profesora; cond[45] alumno;
    String enunciadoExamen;
    int[45] notas;
    Cola examenes;

    Procedure devolucionNota (int in idA, String in examen, int out nota){
        examenes.push(idA,examen);
        signal(profesora);
        wait(alumno[idA]);
        nota = notas[idA];
    }

    Procedure siguiente(int out idA, String out examen){
        if (empty(examenes)){
            wait(profesora);
        }
        examenes.pop(idA,examen);
    }

    Procedure entregarNota(int in idA, int in nota){
        notas[idA] = nota;
        signal(alumno[idA]);
    }

End Monitor;
```

# Ejercicio 10
En un parque hay un juego para ser usada por N personas de a una a la vez y de acuerdo al 
orden en que llegan para solicitar su uso. Además, hay un empleado encargado de desinfectar el 
juego durante 10 minutos antes de que una persona lo use. Cada persona al llegar espera hasta 
que el empleado le avisa que puede usar el juego, lo usa por un tiempo y luego lo devuelve. 
Nota: suponga que la persona tiene una función Usar_juego que simula el uso del juego; y el 
empleado una función Desinfectar_Juego que simula su trabajo. Todos los procesos deben 
terminar su ejecución.

- ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png) `El proceso Personas realiza dos invocaciones sucesivas sin procesamiento activo en el medio al mismo monitor. Esto es indicio de que se podía resolver todo en una sola invocación.`

- ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png) `El proceso Empleado ya podría saber en la invocación de liberarJuego si está habilitando a la última persona y con eso interrumpir sin hacer una invocación adicional a cantRestantes. Esta invocación funciona junto con desinfectar como dos llamadas sucesivas al mismo monitor y se debe evitar.`
```java
Process Personas [id:1..N]
    Juego.llegar();
End Process;

Process Empleado
    for (int i = 1 to N){
        Juego.chequearEstado();
        Desinfectar_Juego(delay(10min));
        Juego.liberarJuego(int restantes);
    }
End Process;

Monitor Juego
    int restantes = N;
    cond empleado; cond persona; cond personaAjugar;
    int enEspera = 0;
    boolean libre;

    Procedure llegar(){
        if(not libre){
            enEspera++;
            wait(persona);
        }
        else{
            libre = false;
        }
        signal(empleado);
        wait (personaAjugar);
        usar_juego();
        if(enEspera > 0){
            enEspera--;
            signal(persona);
        }
        else{
            libre = true;
        }
        restantes--;
    }

    Procedure chequearEstado(){
        if (libre){
            wait(empleado);
        }
    }

    Procedure liberarJuego(){
        signal(personaAjugar);
    }

End Monitor;
```