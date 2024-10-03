# Ejercicio 9
En un examen de la secundaria hay un preceptor y una profesora que deben tomar un examen 
escrito a 45 alumnos. El preceptor se encarga de darle el enunciado del examen a los alumnos 
cundo los 45 han llegado (es el mismo enunciado para todos). La profesora se encarga de ir 
corrigiendo los exámenes de acuerdo con el orden en que los alumnos van entregando. Cada 
alumno al llegar espera a que le den el enunciado, resuelve el examen, y al terminar lo deja 
para que la profesora lo corrija y le envíe la nota. Nota: maximizar la concurrencia; todos los 
procesos deben terminar su ejecución; suponga que la profesora tiene una función 
corregirExamen que recibe un examen y devuelve un entero con la nota. 

```java
Process Alumno [id:1..45]
    String examen; int nota;
    Aula.llegar();
    Examen.solicitarEnunciado(examen);
    // Alumno resuelve examen
    Examen.devolucionNota(examen,nota);
End Process;

Process Preceptor
    String enunciado;
    Examen.esperarAlumnos();
    Examen.repatir(enunciado);
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
    cond alumno;

    Process llegar(){
        presentes++;
        if (presentes == 45){
            signal_all(alumno);
        }
        else{
            wait(alumno);
        }
    }
End Monitor;

Monitor Examen
    cond preceptor; cond profesora; cond alumno;
    String enunciadoExamen;
    boolean primero = true; 
    int[45] notas;
    Cola examenes;

    
    Procedure esperarAlumnos(){
        wait(preceptor);
    }

    Procedure solicitarEnunciado(String out examen){
        if (primero){
            signal(preceptor);
            primero = false;
        }
        examen = enunciadoExamen;
    }

    Procedure repartir(String in enunciado){
        enunciadoExamen = enunciado;
    }

    Procedure devolucionNota (int in idA, String in examen, int out nota){
        examenes.push(idA,examen);
        signal(profesora);
        wait(alumno);
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
        signal(alumno);
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

```java
Process Personas [id:1..N]
    Juego.llegar();
    Juego.usar();
End Process;

Process Empleado
    boolean ok = true; int restantes;
    while (ok){
        Juego.cantRestantes(restantes);
        if (restantes == 0){
            ok = false;
        }
        else{
            Juego.desinfectar();
            delay(10min);
            Juego.liberarJuego();
        }
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
    }

    Procedure usar(){
        signal(empleado);
        wait (personaAjugar);
        if(enEspera > 0){
            enEspera--;
            signal(persona);
        }
        else{
            libre = true;
        }
        restantes--;
    }

    Procedure desinfectar(){
        if (libre){
            wait(empleado);
        }
        Desinfectar_Juego();
    }

    Procedure liberarJuego(){
        signal(personaAjugar);
    }

    Procedure cantRestantes (int out personasRestantes){
        personasRestantes = restantes;
    }
End Monitor;
```