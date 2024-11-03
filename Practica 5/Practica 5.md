# Práctica 5 - ADA

## Ejercicio 1
Se requiere modelar un puente de un único sentido que soporta hasta 5 unidades de peso. El peso de los vehículos depende del tipo: cada auto pesa 1 unidad, cada camioneta pesa 2 unidades y cada camión 3 unidades. Suponga que hay una cantidad innumerable de vehículos (A autos, B camionetas y C camiones). Analice el problema y defina qué tareas, recursos y sincronizaciones serán necesarios/convenientes para resolverlo. 

- a) Realice la solución suponiendo que todos los vehículos tienen la misma prioridad.
- b) Modifique la solución para que tengan mayor prioridad los camiones que el resto de los vehículos. 

### A
```ada
Procedure Puente1 is
    Task puente is
        Entry Auto;
        Entry Camioneta;
        Entry Camion;
        Entry SaleAuto;
        Entry SaleCamioneta;
        Entry SaleCamion;
    End puente;

    Task type vehiculo;

    arrAutos: array (1..A) of vehiculo;
    arrCamioneta: array (1..B) of vehiculo;
    arrCamiones: array (1..C) of vehiculo;


    Task Body vehiculo is
    Begin
        if ("Es auto") then
            Puente.Auto;
        else if ("Es Camioneta") then
            Puente.Camioneta;
        else
            Puente.Camion;
        End if;

        if ("Es auto") then
            Puente.SaleAuto;
        else if ("Es Camioneta") then
            Puente.SaleCamioneta;
        else
            Puente.SaleCamion;
        End if;
    End vehiculo;

    Task Body puente is
        pesoTotal:int;
    Begin
        loop
            SELECT
                WHEN (pesoTotal + 1 <= 5) =>
                    accept Auto do
                        pesoTotal += 1;
                    End Auto;
            OR
                WHEN (pesoTotal + 2 <= 5) =>
                    accept Camioneta do
                        pesoTotal += 2;
                    End Auto;
            OR
                WHEN (pesoTotal + 3 <= 5) =>
                    accept Camion do
                        pesoTotal += 3;
                    End Camion;
            OR
                accept SaleAuto do
                    pesoTotal -= 1;
                End SaleAuto;
            OR
                accept SaleCamioneta do
                    pesoTotal -= 2;
                End SaleCamioneta;
            OR
                accept SaleCamion do
                    pesoTotal -= 3;
                End SaleCamion;
        End loop;
    End puente;
Begin Puente 1
    null;
End Puente1;
```

### B
```ada
Procedure Puente1 is
    Task puente is
        Entry Auto;
        Entry Camioneta;
        Entry Camion;
        Entry SaleAuto;
        Entry SaleCamioneta;
        Entry SaleCamion;
    End puente;

    Task type vehiculo;

    arrAutos: array (1..A) of vehiculo;
    arrCamioneta: array (1..B) of vehiculo;
    arrCamiones: array (1..C) of vehiculo;


    Task Body vehiculo is
    Begin
        if ("Es auto") then
            Puente.Auto;
        else if ("Es Camioneta") then
            Puente.Camioneta;
        else
            Puente.Camion;
        End if;

        if ("Es auto") then
            Puente.SaleAuto;
        else if ("Es Camioneta") then
            Puente.SaleCamioneta;
        else
            Puente.SaleCamion;
        End if;
    End vehiculo;

    Task Body puente is
        pesoTotal:int;
    Begin
        loop
            SELECT
                WHEN (pesoTotal + 1 <= 5 and Camion'count = 0) =>
                    accept Auto do
                        pesoTotal += 1;
                    End Auto;
            OR
                WHEN (pesoTotal + 2 <= 5 and Camion'count = 0) =>
                    accept Camioneta do
                        pesoTotal += 2;
                    End Auto;
            OR
                WHEN (pesoTotal + 3 <= 5) =>
                    accept Camion do
                        pesoTotal += 3;
                    End Camion;
            OR
                accept SaleAuto do
                    pesoTotal -= 1;
                End SaleAuto;
            OR
                accept SaleCamioneta do
                    pesoTotal -= 2;
                End SaleCamioneta;
            OR
                accept SaleCamion do
                    pesoTotal -= 3;
                End SaleCamion;
            End SELECT;
        End loop;
    End puente;
Begin Puente 1
    null;
End Puente1;
```

## Ejercicio 2
Se quiere modelar el funcionamiento de un banco, al cual llegan clientes que deben realizar un pago y retirar un comprobante. Existe un único empleado en el banco, el cual atiende de acuerdo con el orden de llegada.

- a) Implemente una solución donde los clientes llegan y se retiran sólo después de haber sido atEndidos.
- b) Implemente una solución donde los clientes se retiran si esperan más de 10 minutos para realizar el pago.
- c) Implemente una solución donde los clientes se retiran si no son atEndidos inmediatamente.
- d) Implemente una solución donde los clientes esperan a lo sumo 10 minutos para ser atEndidos. Si pasado ese lapso no fueron atEndidos, entonces solicitan atención una vez más y se retiran si no son atEndidos inmediatamente.

### A
```ada
Procedure Banco1 is
    Task empleado is
        Entry Pedido(dinero:IN int; comprobante:OUT int);
    End empleado;

    Task type cliente;

    arrayClientes: array (1..N) of cliente;

    Task body Cliente is
        int dinero,comprobante;
    Begin
        Empleado.Pedido(dinero,comprobante);
    End Cliente;

    Task body Empleado is

    Begin
        loop
            accept Pedido (dinero:IN int; comprobante:OUT in)
                comprobante = generarComprobante();
            End Pedido;
        End loop;
    End Empleado;
Begin Banco1
    null;
End Banco1;
```

### B
```ada
Procedure Banco1 is
    Task empleado is
        Entry Pedido(dinero:IN int; comprobante:OUT int);
    End empleado;

    Task type cliente;

    arrayClientes: array (1..N) of cliente;

    Task body Cliente is
        int dinero,comprobante;
    Begin
        SELECT
            Empleado.Pedido(dinero,comprobante);
        OR DELAY 600
            null;
        End SELECT;
    End Cliente;

    Task body Empleado is

    Begin
        loop
            accept Pedido (dinero:IN int; comprobante:OUT in)
                comprobante = generarComprobante();
            End Pedido;
        End loop;
    End Empleado;
Begin Banco 1
    null;
End Banco1;
```

### C
```ada
Procedure Banco1 is
    Task empleado is
        Entry Pedido(dinero:IN int; comprobante:OUT int);
    End empleado;

    Task type cliente;

    arrayClientes: array (1..N) of cliente;

    Task body Cliente is
        int dinero,comprobante;
    Begin
        SELECT
            Empleado.Pedido(dinero,comprobante);
        ELSE
            null;
        End SELECT;
    End Cliente;

    Task body Empleado is

    Begin
        loop
            accept Pedido (dinero:IN int; comprobante:OUT in)
                comprobante = generarComprobante();
            End Pedido;
        End loop;
    End Empleado;
End Banco1;
```

### D
```ada
Procedure Banco1 is
    Task empleado is
        Entry Pedido(dinero:IN int; comprobante:OUT int);
    End empleado;

    Task type cliente;

    arrayClientes: array (1..N) of cliente;

    Task body Cliente is
        int dinero,comprobante; boolean atEndido;
    Begin
        atEndido = false;
        SELECT
            Empleado.Pedido(dinero,comprobante);
            atEndido = true;
        OR DELAY 600
            null;
        End SELECT;
        If (atEndido == false) then
            SELECT
                Empleado.Pedido(dinero,comprobante);
            OR ELSE
                null;
            End SELECT;
        End if;
    End Cliente;

    Task body Empleado is

    Begin
        loop
            accept Pedido (dinero:IN int; comprobante:OUT in)
                comprobante = generarComprobante();
            End Pedido;
        End loop;
    End Empleado;
Begin Banco 1
    null;
End Banco1;
```

## Ejercicio 3
Se dispone de un sistema compuesto por 1 central y 2 procesos periféricos, que se comunican continuamente. Se requiere modelar su funcionamiento considerando las siguientes condiciones:

- La central siempre comienza su ejecución tomando una señal del proceso 1; luego toma aleatoriamente señales de cualquiera de los dos indefinidamente. Al recibir una señal de proceso 2, recibe señales del mismo proceso durante 3 minutos.
- Los procesos periféricos envían señales continuamente a la central. La señal del proceso 1 será considerada vieja (se deshecha) si en 2 minutos no fue recibida. Si la señal del proceso 2 no puede ser recibida inmediatamente, entonces espera 1 minuto y vuelve a mandarla (no se deshecha).

```ada
Procedure SistemaCompuesto is
    Task central is
        Entry señal1;
        Entry señal2;
        Entry StopTimer;
    End central;

    Task proceso1;
    Task proceso2;

    Task timer is
        Entry Iniciar;
    End timer;

    Task body central is
        boolean ok;
    Begin
        timer = 180;
        accept señal1;
        loop
            SELECT
                accept señal1;
            OR
                accept señal2 do
                    while (ok) loop
                        SELECT
                            when (StopTimer'count = 0) =>
                                accept señal2;
                        OR
                            accept StopTimer do
                                ok = false;
                            End StopTimer;
                        End SELECT;
                    End loop;
                End señal2;
            End SELECT;
        End loop;
    End central;

    Task body proceso1 is
    Begin
        loop
            // Generar señal;
            SELECT
                Central.señal1;
            OR DELAY 120
                null;
            End SELECT;
        End loop;
    End proceso1;

    Task body proceso2 is
    Begin
        // Generar señal;
        loop
            SELECT
                Central.señal2;
                // Generar señal;
            OR DELAY 60
                null;
            End SELECT;
        End loop;
    End proceso2;

    Task body Timer is
    Begin
        loop
            accept Iniciar;
            delay(120);
            Central.StopTimer;
        End loop;
    End Timer;

Begin
    null;
End SistemaCompuesto
```

### Ejercicio 4
En una clínica existe un médico de guardia que recibe continuamente peticiones de atención de las E enfermeras que trabajan en su piso y de las P personas que llegan a la clínica ser atEndidos.  
Cuando una persona necesita que la atiEndan espera a lo sumo 5 minutos a que el médico lo haga, si pasado ese tiempo no lo hace, espera 10 minutos y vuelve a requerir la atención del médico. Si no es atEndida tres veces, se enoja y se retira de la clínica. 
Cuando una enfermera requiere la atención del médico, si este no lo atiende inmediatamente le hace una nota y se la deja en el consultorio para que esta resuelva su pedido en el momento que pueda (el pedido puede ser que el médico le firme algún papel). Cuando la petición ha sido recibida por el médico o la nota ha sido dejada en el escritorio, continúa trabajando y haciEndo más peticiones.
El médico atiende los pedidos dándole prioridad a los enfermos que llegan para ser atEndidos. Cuando atiende un pedido, recibe la solicitud y la procesa durante un cierto tiempo. Cuando está libre aprovecha a procesar las notas dejadas por las enfermeras. 

```ada
Procedure Hospital is
    Task medico is
        Entry Enfermera;
        Entry Persona;
    End medico;

    Task archivo is
        Entry notaSolicitudEnfermera(pedido:IN string);
        Entry recibirNota(pedido:OUT string);
    End archivo;

    Task type persona;
    Task type enfermera;

    arrPersona: array (1..P) of personas;
    arrEnfermeras: array (1..E) of enfermeras;

    Task body persona is
        boolean atEndido;
    Begin
        atEndido = false;
        SELECT
            Medico.Persona;
            atEndido = true;
        OR DELAY 300;
            null;
        End SELECT;
        if (atEndido == false) then
        while (contador < 3 and atEndido == false) do begin
            SELECT
                Medico.Persona;
                atEndido = true;
            OR DELAY 600;
                contador++;
            End SELECT;
        End while;
        //Me enojo y me voy;
    End persona;

    Task body enfermera is
        String pedido;
    Begin
        SELECT
            Medico.Enfermera;
        OR ELSE
            Archivo.notaSolicitudEnfermera(pedido); // Deja la nota con el pedido a realizar
        End SELECT;
    End enfermera;

    Task body medico is
        String peticion;
    Begin
        loop
            SELECT
                accept Persona do
                    //atiende persona;
                End Persona;
            OR
                when (Persona'count = 0) => accept Enfermera do
                    //atiende enfermera;
                End Enfermera;
            ELSE
                SELECT
                    Archivo.recibirnota(peticion);
                    //atiende peticion;
                OR ESLE
                    null;
                End SELECT;
            End SELECT;
        End loop;
    End medico;

    Task body archivo is
        Cola pedidos;
    Begin
        loop
            SELECT
                when (notaSolicitudEnfermera'count > 0) => 
                    accept notaSolicitudEnfermera (pedido:IN string) do
                        pedidos.push(pedido);
                    End notaSolicitudEnfermera;
            OR
                when (recibirNota'count > 0 and not empty(pedidos)) => 
                    accept recibirNota (pedido:OUT string) do
                        pedido = pedidos.pop();
                    End recibirNota;
            End SELECT
        End loop;
    End archivo;

Begin
    null;
End Hospital;
```

## Ejercicio 5
En un sistema para acreditar carreras universitarias, hay UN Servidor que atiende pedidos de U Usuarios de a uno a la vez y de acuerdo con el orden en que se hacen los pedidos. Cada usuario trabaja en el documento a presentar, y luego lo envía al servidor; espera la respuesta de este que le indica si está todo bien o hay algún error. Mientras haya algún error, vuelve a trabajar con el documento y a enviarlo al servidor. Cuando el servidor le responde que está todo bien, el usuario se retira. Cuando un usuario envía un pedido espera a lo sumo 2 minutos a que sea recibido por el servidor, pasado ese tiempo espera un minuto y vuelve a intentarlo (usando el mismo documento). 

```ada
Procedure Universidad
    Task Servidor is
        Entry Pedido(documento:IN String, respuesta:OUT String);
    End Servidor;

    Task type usuario;
    arrayUsuarios: array (1..N) of usuario;

    Task body usuario is
        String documento,respuesta;
    Begin
        documento = //Trabajando con el documento;
        while (respuesta != "Todo bien")loop
            SELECT
                Servidor.Pedido(documento,respuesta);
                if (respuesta == "Error") then
                    documento = //Corregir error/es;
            OR DELAY 120
                delay(60);
            End SELECT;
        End loop;
    End usuario;

    Task body Servidor is

    Begin
        loop
            accept Pedido (documento:IN String, respuesta:OUT String) do
                respuesta = documento.analizar();
            End Pedido;
        End loop;
    End;

Begin
    null;
End Universidad;
```

### Ejercicio 6
En una playa hay 5 equipos de 4 personas cada uno (en total son 20 personas donde cada una conoce previamente a que equipo pertenece). Cuando las personas van llegando esperan con los de su equipo hasta que el mismo esté completo (hayan llegado los 4 integrantes), a partir de ese momento el equipo comienza a jugar. El juego consiste en que cada integrante del grupo junta 15 monedas de a una en una playa (las monedas pueden ser de 1, 2 o 5 pesos) y se suman los montos de las 60 monedas conseguidas en el grupo. Al finalizar cada persona debe conocer el grupo que más dinero junto. **Nota**: maximizar la concurrencia. Suponga que para simular la búsqueda de una moneda por parte de una persona existe una función Moneda() que retorna el valor de la moneda encontrada.

```ada
Procedure Playa
    Task type Persona is
        Entry comenzar;
    end Persona;
    arrayPersonas: array (1..20) of persona;

    Task type Equipo is
        Entry marcarPresente;
        Entry identificarme(idEquipo:OUT int);
        Entry personaTermina(totalRecolectadoPersona:IN int);
        Entry equipoGanador(idGrupoGanador:OUT int);
    End Equipo;
    Equipos: array (1..5) of Equipo;

    Task AdministradorEvento is
        Entry equipoTermina(idEquipo:IN int, totalRecolectadoEquipo:IN int);
        Entry equipoGanador(idEquipoGanador:OUT int);
    End AdministradorEvento;

    Task body Persona is
        miEquipo:int = equipoAsignado;
        totalRecolectado,idEquipoGanador:int;
    Begin
        totalRecolectado = 0;
        Equipos(miEquipo).marcarPresente;
        accept comenzar;
        for (int i = 1 to 15) loop
            //Junta una moneda
            totalRecolectado += Moneda();
        end loop;
        Equipos(miEquipo).personaTermina(totalRecolectado);
        Equipos(miEquipo).equipoGanador(idEquipoGanador);
    End Persona;

    Task body Equipo is
        idEquipo,totalRecolectado,idEquipoGanador:int;
        idsPersonas: array (1..4) of int;
    Begin
        accept (idEquipo:OUT int);
        for (int i = 1 to 4) do accept marcarPresente;
        for (idPersona in idPersonas) do idPersona.Comenzar;
        for (int i = 1 to 4) loop
            accept personaTermina(totalRecolectadoPersona:IN int) do totalRecolectado += totalRecolectadoPersona;
        end loop;
        AdministradorEvento.equipoTermina(idEquipo,totalRecolectado);
        AdministradorEvento.equipoGanador(idEquipoGanador);
        for (int i = 1 to 4) loop
            accept equipoGanador(idGrupoGanador:OUT int) do
                idGrupoGanador = idEquipoGanador;
            end equipoGanador;
        end loop;
    End Equipo;

    Task body AdministradorEvento is
        grupoMax,maxRecolectado:int;
    Begin
        grupoMax = 0; maxRecolectado = 0;
        for (int i = 1 to 5) loop
            accept equipoTermina(idEquipo:IN int, totalRecolectadoEquipo:IN int) do
                if (totalRecolectadoEquipo > maxRecolectado) then
                    maxRecolectado = totalRecolectadoEquipo;
                    grupoMax = idEquipo;
                end if;
            end equipoTermina;
        end loop;
        for (int i = 1 to 5) loop
            accept equipoGanador(idEquipoGanador:OUT int) do
                idEquipoGanador = grupoMax;
            end equipoGanador;
        end loop;
    End AdministradorEvento;

Begin
    for (int i = 1 to 5) loop
        Equipos(i).identificarme(i);
    end loop;
End Playa;
```

### Ejercicio 7
Se debe calcular el valor promedio de un vector de 1 millón de números enteros que se encuentra distribuido entre 10 procesos Worker (es decir, cada Worker tiene un vector de 100 mil números). Para ello, existe un Coordinador que determina el momento en que se debe realizar el cálculo de este promedio y que, además, se queda con el resultado. **Nota**: maximizar la concurrencia; este cálculo se hace una sola vez.

```ada
Procedure Promedio is
    Task Coordinador is
        Entry iniciar;
        Entry resultado(calculo:IN int);
    End Coordinador;    

    Task type Worker;
    arrayWorkers: array (1..10) of Worker;

    Task body Worker is
        numeros:array[1..100000] of int = InicializarVector;
        total:int;
    Begin
        Admin.iniciar;
        for (int i = 1 to 100000) loop
            total += numeros[i];
        End loop;
        Admin.Resultado(total);
    End Worker;

    Task body Coordinador is
        total,calculo:int;
        promedio:real;
    Begin
        total = 0;
        for (int i = 1 to 20) loop
            SELECT
                accept iniciar;
            OR
                accept resultado(calculo:IN int) do
                    total += calculo;
                End resultado;
            End SELECT;
        End loop;
        promedio = total/1000000;
    End Coordinador;
Begin
    null;
End Promedio;
```

## Ejercicio 8
Hay un sistema de reconocimiento de huellas dactilares de la policía que tiene 8 Servidores para realizar el reconocimiento, cada uno de ellos trabajando con una Base de Datos propia; a su vez hay un Especialista que utiliza indefinidamente. El sistema funciona de la siguiente manera: el Especialista toma una imagen de una huella (TEST) y se la envía a los servidores para que cada uno de ellos le devuelva el código y el valor de similitud de la huella que más se asemeja a TEST en su BD; al final del procesamiento, el especialista debe conocer el código de la huella con mayor valor de similitud entre las devueltas por los 8 servidores. 
Cuando ha terminado de procesar una huella comienza nuevamente todo el ciclo. Nota: suponga que existe una función Buscar(test, código, valor) que utiliza cada Servidor donde recibe como parámetro de entrada la huella test, y devuelve como parámetros de salida el código y el valor de similitud de la huella más parecida a test en la BD correspondiente. Maximizar la concurrencia y no generar demora innecesaria. 

```ada
Procedure Policia is
    Task Especialista is
        Entry enviarHuella (test:OIT Huella);
        Entry recibirResultado (codigo:IN int, valor:IN int);
        Entry listo;
    End Especialista;

    Task type Servidor;
    arrayServidores: array (1..8) of Servidor;

    Task body servidor is
        Huella test;
        int codigo,valor;
    Begin
        loop
            Especialista.enviarHuella(test);
            Buscar(test,codigo,valor);
            Especialista.recibirResultado(codigo,valor);
            Especialista.listo;
        End loop;
    End Servidor;

    Task body Especialista is
        Huella test;
        codigo, valor, codigomax:int;
    Begin
        loop
            test = //Especialista toma imagen de una huella;
            valormax = 0; codigomax = 0;
            for (int i= 1 to 16) loop
                SELECT
                    accept enviarHuella(test:OUT Huella);
                OR
                    accept recibirResultado (codigo:IN int, valor:IN int) do
                        if (valor > valormax) then begin
                            valormax = valor;
                            codigomax = codigo;
                        end if;
                    end recibirResultado;
                END SELECT;
            end loop;
            writeln ("Codigo con mayor similitud: ",codigomax);
            for (int j = 1 to 8) loop
                accept listo;
            end loop;
        end loop;
    End Especialista;

Begin
    null;
End Policia;
```

### Ejercicio 9
Una empresa de limpieza se encarga de recolectar residuos en una ciudad por medio de 3 camiones. Hay P personas que hacen reclamos continuamente hasta que uno de los camiones pase por su casa. Cada persona hace un reclamo y espera a lo sumo 15 minutos a que llegue un camión; si no pasa, vuelve a hacer el reclamo y a esperar a lo sumo 15 minutos a que llegue un camión; y así sucesivamente hasta que el camión llegue y recolecte los residuos. Sólo cuando un camión llega, es cuando deja de hacer reclamos y se retira. Cuando un camión está libre la empresa lo envía a la casa de la persona que más reclamos ha hecho sin ser atendido. **Nota**: maximizar la concurrencia.

```ada
Procedure Ciudad is
    Task type camion;
    arrayCamiones: array(1..3) of camion;


    Task type persona;
    Personas: array(1..P) of persona;
    Task persona is
        Entry identificador(id:OUT int);
        Entry recolectar;
    End persona;

    Task Empresa is 
        Entry pedido(idPersona:IN int);
        Entry siguiente(idPersona:OUT int);
    end Empresa;

    Task body persona is
        id: int;
        atendido:boolean;
    Begin
        accept identificador(id:OUT int);
        while (atendido == false) loop
            Empresa.pedido(id);
            SELECT
                accept recolectar;
                atendido = true;
            OR DELAY 900
                null;
            END SELECT;
        end loop;
    End persona;

    Task body camion is
        idPersona:int;
    Begin
        loop
            Empresa.siguiente(idPersona);
            Persona(idPersona).recolectar;
        end loop;
    End camion;

    Task body Empresa is
        reclamos:array(1..P) of int;
    Begin
        for (int i = 1 to P) loop
            reclamos(i) = 0;
        end loop;
        loop
            SELECT
                accept pedido(idPersona:IN int)
                    reclamos(idPersona)++;
                end pedido;
            OR
                when alguienPendiente(reclamos) => accept siguiente(idPersona:OUT int) do
                    idPersona = reclamos.obtenerMaximoPedidos();
                    reclamos(idPersona) = -1; //Finaliza contador de reclamos de la persona
                End siguiente;
            END SELECT;
        end loop;
    End Empresa;

Begin
    for (int i = 1 to P) loop
        Personas(i).identificador(i);
    end loop;
End Ciudad;
```