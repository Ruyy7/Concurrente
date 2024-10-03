# Ejercicio 5
## Inciso A
```java
Process Cliente [id:1..N]
    string listaProductos; string comprobante;

    Atencion.llegada();
    Recepcion.atender(listaProductos,comprobante);
End Process;

Process Empleado
    string datos;
    while (true){
        Atencion.proximo();
        Recepcion.esperarDatos(datos);
        string resultado = generar comprobante;
        Recepcion.enviarDatos(resultado);
    }
End Process;

Monitor Atencion
    cond esperaC;
    int esperando = 0;
    boolean libre = true;
    
    Procedure Llegada(){
        if (not libre){
            esperando++;
            wait(esperaC);
        }
        else{libre = false}
    }

    Procedure Proximo(){
        if (esperando > 0 ){
            esperando--;
            signal(esperaC);
        }
        else {libre = true}
    }
End Monitor;

Monitor Recepcion
    cond empleado; cond cliente;
    boolean listo = false;
    String datos; String resultados;

    Procedure atender(String in listaProductos; String out comprobante){
        datos = listaProductos; //Datos de la lista de productos del cliente
        listo = true; //Aviso que ya hay datos para que el empleado los procese
        signal(empleado);
        wait(cliente);
        comprobante = resultados; // Realizamos la copia del comprobante que realizo el empleado
        signal(empleado); // Volvemos a despertar al empelado para que continue con los demas clientes
    }

    Procedure esperarDatos(String out listaProductos){
        if (not listo){ 
            wait(empleado);
        }
        // Si el empleado llego primero y no hay datos esperamos a que el cliente nos proporcione los datos, sino, si hay datos (es decir el cliente llego primero) copiamos la lista de productos que proporciono el cliente
        // en el procedimiento atender.
        listaProductos = datos;
    }

    Procedure enviarDatos (String in comprobante){
        resultados = comprobante; // El empleado devuelve el comprobante
        signal(cliente); // Despierto al cliente para que copie el comprobante
        wait(empleado); // Espero a que el cliente copie el comprobante
        listo = false; // Reseteamos para el proximo cliente
    }

End Monitor;
```

## Inciso B

```java
Process Cliente [idC:1..N]
    string listaProductos; string comprobante; int idE;

    Atencion.llegada(idE); // Recibimos el empleado que nos va a atender
    Recepcion[idE].atender(listaProductos,comprobante); //Le pedimos al empleado asignado (idE) que nos realice el comprobante
End Process;

Process Empleado [idE:1..E]
    string datos;
    while (true){
        Atencion.proximo(idE); //Enviamos el id del empleado para agregarlo en la cola de empleados disponibles
        Recepcion[idE].esperarDatos(datos); //En el monitor que le corresponde espera los datos del cliente
        string resultado = generar comprobante;
        Recepcion[idE].enviarDatos(resultado); //En el monitor que le corresponde envia el comprobante
    }
End Process;

Monitor Atencion
    cond esperaC;
    Cola empleados; 
    int esperando = 0;
    int libre = 0;
    
    Procedure Llegada(int out idE){
        if (libre == 0){
            esperando++;
            wait(esperaC);
        }
        libre--;
        idE = empleados.pop();
    }

    Procedure Proximo(int in idE){
        empelados.push(idE);
        if (esperando > 0 ){
            esperando--;
            signal(esperaC);
        }
        else {libre++}
    }
End Monitor;

Monitor Recepcion [id:1..E]
    cond empleado; cond cliente;
    boolean listo = false;
    String datos; String resultados;

    Procedure atender(String in listaProductos; String out comprobante){
        datos = listaProductos; //Datos de la lista de productos del cliente
        listo = true; //Aviso que ya hay datos para que el empleado los procese
        signal(empleado);
        wait(cliente);
        comprobante = resultados; // Realizamos la copia del comprobante que realizo el empleado
        signal(empleado); // Volvemos a despertar al empelado para que continue con los demas clientes
    }

    Procedure esperarDatos(String out listaProductos){
        if (not listo){ 
            wait(empleado);
        }
        // Si el empleado llego primero y no hay datos esperamos a que el cliente nos proporcione los datos, sino, si hay datos (es decir el cliente llego primero) copiamos la lista de productos que proporciono el cliente
        // en el procedimiento atender.
        listaProductos = datos;
    }

    Procedure enviarDatos (String in comprobante){
        resultados = comprobante; // El empleado devuelve el comprobante
        signal(cliente); // Despierto al cliente para que copie el comprobante
        wait(empleado); // Espero a que el cliente copie el comprobante
        listo = false; // Reseteamos para el proximo cliente
    }

End Monitor;
```

## Inciso C
```java
Process Cliente [idC:1..N]
    string listaProductos; string comprobante; int idE;

    Atencion.llegada(idE); // Recibimos el empleado que nos va a atender
    Recepcion[idE].atender(listaProductos,comprobante); //Le pedimos al empleado asignado (idE) que nos realice el comprobante
End Process;

Process Empleado [idE:1..E]
    string datos; boolean ok = true; int clientesRestanes = 0;
    while (ok){
        Atencion.atendidos(clientesRestantes); // Verificamos la cantidad de clientes restantes por atender
        if (clientesRestantes == 0){
            ok = false;
        }
        else{
            Atencion.proximo(idE); //Enviamos el id del empleado para agregarlo en la cola de empleados disponibles
            Recepcion[idE].esperarDatos(datos); //En el monitor que le corresponde espera los datos del cliente
            string resultado = generar comprobante;
            Recepcion[idE].enviarDatos(resultado); //En el monitor que le corresponde envia el comprobante
        }

    }
End Process;

Monitor Atencion
    cond esperaC;
    Cola empleados; 
    int esperando = 0;
    int libre = 0;
    int clientesRestantes = N;

    Procedure Llegada(int out idE){
        if (libre == 0){
            esperando++;
            wait(esperaC);
        }
        libre--;
        idE = empleados.pop();
    }

    Procedure Proximo(int in idE){
        empelados.push(idE);
        if (esperando > 0 ){
            esperando--;
            signal(esperaC);
        }
        else {libre++}
    }

    Procedure atendidos(int out cliRestantes){
        cliRestantes = clientesRestantes;
    }
End Monitor;

Monitor Recepcion [id:1..E]
    cond empleado; cond cliente;
    boolean listo = false;
    String datos; String resultados;

    Procedure atender(String in listaProductos; String out comprobante){
        datos = listaProductos; //Datos de la lista de productos del cliente
        listo = true; //Aviso que ya hay datos para que el empleado los procese
        signal(empleado);
        wait(cliente);
        comprobante = resultados; // Realizamos la copia del comprobante que realizo el empleado
        signal(empleado); // Volvemos a despertar al empelado para que continue con los demas clientes
    }

    Procedure esperarDatos(String out listaProductos){
        if (not listo){ 
            wait(empleado);
        }
        // Si el empleado llego primero y no hay datos esperamos a que el cliente nos proporcione los datos, sino, si hay datos (es decir el cliente llego primero) copiamos la lista de productos que proporciono el cliente
        // en el procedimiento atender.
        listaProductos = datos;
    }

    Procedure enviarDatos (String in comprobante){
        resultados = comprobante; // El empleado devuelve el comprobante
        signal(cliente); // Despierto al cliente para que copie el comprobante
        wait(empleado); // Espero a que el cliente copie el comprobante
        listo = false; // Reseteamos para el proximo cliente
    }

End Monitor;
```

# Ejercicio 6

```java
Process Alumno [id:1..50]
    int nroGrupo;

    Tarea.formarFila(id);
    Tarea.recibirNumeroGrupo(id,nroGrupo);
    // Realizar tarea
    Tarea.esperarNota(nota);
End Process;

Process JTP
    int nroGrupoAux = 0;
    Tarea.esperarAlumnos();
    for (int i= 1 to 50){
        nroGrupoAux = AsignarNroGrupo();
        Tarea.asignarNumeroGrupo(nroGrupoAux);
    }
End Process;

Monitor Tarea
    Cola fila;
    cond profesor;
    int restantes = 50;
    cond[50] esperandoNumero;
    int[50] alumnmos;


    Procedure formarFila(int in idA){
        restantes--;
        if (restantes > 0){
            fila.push(idA);
        }
        else{
            signal(profesor);
        }
    }

    Procedure esperarAlumnos(){
        if (restantes > 0){
            wait(profesor);
        }
    }

    Procedure recibirNumeroGrupo(int in id; int out nroGrupo){
        wait(esperandoNumero[id]);
        nroGrupo = alumnos[idAlumnoAux];
    }

    Procedure asignarNumeroGrupo (int in nroGrupo){
        idAlumnoAux = fila.pop();
        alumnos[idAlumnoAux] = nroGrupo;
        signal(esperandoNumero[idAlumnoAux]);
    }


End Monitor;
```

# Ejercicio 7
```java
    Process Corredor[id:1..C]
        Carrera.llegada();
        //Corriendo carrera
        Carrera.agarrarBotella();
    End Process;

    Process Repositor
        while(true){
            Carrera.esperarAvisoReFill();
            Carrera.reponerBotellas();
        }
    End Process;

    Monitor Carrera
        int restantes = C;
        int botellas = 20;
        int corredoresEnFila = 0;
        cond corredor;
        cond filaBotellas;
        cond repositor;
        cond corredorSinBotellas;
        boolean maquinaLibre = true;

        Procedure llegada(){
            restantes--;
            if (restantes > 0){
                wait(corredor);
            }
            else{
                signal_all(corredor);
            }
        }

        Procedure agarrarBotella(){
            if (not maquinaLibre){
                corredoresEnFila++;
                wait(filaBotellas);
            }
            else{
                maquinaLibre = false;
            }
            if (botellas == 0 ){
                signal(repositor);
                wait(corredorSinBotellas);
            }
            botellas--;
            if (corredoresEnFila > 0){
                corredoresEnFila--;
                signal(filaBotellas);
            }
            else {maquinaLibre = true}
        }

        Procedure esperarAvisoReFill(){
            if (botellas > 0){
                wait(repositor);
            }
        }

        Procedure reponerBotellas(){
            botellas = 20;
            signal(corredorSinBotellas);
        }


    End Monitor;
```

# Ejercicio 8
```java
Process Jugador [id:1..20]
    int equipo = DarEquipo();
    int cancha = 0;

    Equipo[id].llegada(cancha); // Aviso a mi equipo que llegue.
    Juego[cancha].entrar(); // Entro a la cancha que me corresponde.
End Process;

Process Partido [id:1..2]
    Juego[id].iniciarPartido(); // Cuando estan los 2 equipos arranca el partido.
    delay(50min); // El delay se realiza aca asi todos los jugadores juegan el mismo partido.
    Juego[id].terminarPartido(); // Avisa a todos los jugadores que terminó el partido.
End Process;

Monitor Equipo [id:1..4]
    int presentes = 0;
    int cancha = 0;
    cond esperandoJugadores;

    Procedure llegada (int out canchaAsignada){
        presentes++;
        if (presentes == 5){
            Cancha.asignarCancha(cancha);
            signal_all(esperandoJugadores);
        }
        else{
            wait(esperandoJugadores);
        }
        canchaAsignada = cancha;
    }
    // Mientras no este todo el equipo espero como jugador a que estemos todos, una vez que estamos todos pedimos que nos asignen una cancha.
End Monitor;

Monitor Cancha
    int equiposPresentes = 0;

    Procedure asignarCancha(int out cancha){
        equiposPresentes++;
        if (equiposPresentes < 3 ){
            cancha = 1;
        }
        else{
            cancha = 2;
        }
    }
    // En base a la cantidad de equipos presentes elijo cancha 1 o cancha 2.
End Monitor;

Monitor Juego [id:1..2]
    cond partido;
    cond jugador;
    int presentes = 0;

    Procedure entrar(){
        presentes ++;
        if (presentes == 10){
            signal(partido);
        }
        wait(jugador);
    }
    // Una vez que esten todos los jugadores en la cancha despertamos al proceso partido asi empieza a correr el tiempo, los jugadores se "duermen" porque quien maneja el tiempo del juego es el partido, asi una vez termine el partido todos los jugadores se retiran en simultaneo.

    Procedure iniciarPartido(){
        if (jugadores < 10){
            wait(partido);
        }
    }

    Procedure terminarPartido(){
        signal_all(jugador); // Avisamos a todos los jugadores que terminó el partido.
    }
End Monitor;
```

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