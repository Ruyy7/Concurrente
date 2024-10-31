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
        end if;

        if ("Es auto") then
            Puente.SaleAuto;
        else if ("Es Camioneta") then
            Puente.SaleCamioneta;
        else
            Puente.SaleCamion;
        end if;
    End vehiculo;

    Task Body puente is
        pesoTotal:int;
    Begin
        loop
            SELECT
                WHEN (pesoTotal + 1 <= 5) =>
                    accept Auto do
                        pesoTotal += 1;
                    end Auto;
            OR
                WHEN (pesoTotal + 2 <= 5) =>
                    accept Camioneta do
                        pesoTotal += 2;
                    end Auto;
            OR
                WHEN (pesoTotal + 3 <= 5) =>
                    accept Camion do
                        pesoTotal += 3;
                    end Camion;
            OR
                accept SaleAuto do
                    pesoTotal -= 1;
                end SaleAuto;
            OR
                accept SaleCamioneta do
                    pesoTotal -= 2;
                end SaleCamioneta;
            OR
                accept SaleCamion do
                    pesoTotal -= 3;
                end SaleCamion;
        end loop;
    end puente;
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
        end if;

        if ("Es auto") then
            Puente.SaleAuto;
        else if ("Es Camioneta") then
            Puente.SaleCamioneta;
        else
            Puente.SaleCamion;
        end if;
    End vehiculo;

    Task Body puente is
        pesoTotal:int;
    Begin
        loop
            SELECT
                WHEN (pesoTotal + 1 <= 5 and Camion'count = 0) =>
                    accept Auto do
                        pesoTotal += 1;
                    end Auto;
            OR
                WHEN (pesoTotal + 2 <= 5 and Camion'count = 0) =>
                    accept Camioneta do
                        pesoTotal += 2;
                    end Auto;
            OR
                WHEN (pesoTotal + 3 <= 5) =>
                    accept Camion do
                        pesoTotal += 3;
                    end Camion;
            OR
                accept SaleAuto do
                    pesoTotal -= 1;
                end SaleAuto;
            OR
                accept SaleCamioneta do
                    pesoTotal -= 2;
                end SaleCamioneta;
            OR
                accept SaleCamion do
                    pesoTotal -= 3;
                end SaleCamion;
            END SELECT;
        end loop;
    end puente;
Begin Puente 1
    null;
End Puente1;
```

## Ejercicio 2
Se quiere modelar el funcionamiento de un banco, al cual llegan clientes que deben realizar un pago y retirar un comprobante. Existe un único empleado en el banco, el cual atiende de acuerdo con el orden de llegada.

- a) Implemente una solución donde los clientes llegan y se retiran sólo después de haber sido atendidos.
- b) Implemente una solución donde los clientes se retiran si esperan más de 10 minutos para realizar el pago.
- c) Implemente una solución donde los clientes se retiran si no son atendidos inmediatamente.
- d) Implemente una solución donde los clientes esperan a lo sumo 10 minutos para ser atendidos. Si pasado ese lapso no fueron atendidos, entonces solicitan atención una vez más y se retiran si no son atendidos inmediatamente.

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
            end Pedido;
        end loop;
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
        END SELECT;
    End Cliente;

    Task body Empleado is

    Begin
        loop
            accept Pedido (dinero:IN int; comprobante:OUT in)
                comprobante = generarComprobante();
            end Pedido;
        end loop;
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
        END SELECT;
    End Cliente;

    Task body Empleado is

    Begin
        loop
            accept Pedido (dinero:IN int; comprobante:OUT in)
                comprobante = generarComprobante();
            end Pedido;
        end loop;
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
        int dinero,comprobante; boolean atendido;
    Begin
        atendido = false;
        SELECT
            Empleado.Pedido(dinero,comprobante);
            atendido = true;
        OR DELAY 600
            null;
        END SELECT;
        If (atendido == false) then
            SELECT
                Empleado.Pedido(dinero,comprobante);
            OR ELSE
                null;
            END SELECT;
        end if;
    End Cliente;

    Task body Empleado is

    Begin
        loop
            accept Pedido (dinero:IN int; comprobante:OUT in)
                comprobante = generarComprobante();
            end Pedido;
        end loop;
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
    end timer;

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
                            end StopTimer;
                        END SELECT;
                    end loop;
                end señal2;
            END SELECT;
        end loop;
    end central;

    Task body proceso1 is
    Begin
        loop
            // Generar señal;
            SELECT
                Central.señal1;
            OR DELAY 120
                null;
            END SELECT;
        end loop;
    end proceso1;

    Task body proceso2 is
    Begin
        // Generar señal;
        loop
            SELECT
                Central.señal2;
                // Generar señal;
            OR DELAY 60
                null;
            END SELECT;
        end loop;
    end proceso2;

    Task body Timer is
    Begin
        loop
            accept Iniciar;
            delay(120);
            Central.StopTimer;
        end loop;
    end Timer;

Begin
    null;
End SistemaCompuesto
```

### Ejercicio 4
En una clínica existe un médico de guardia que recibe continuamente peticiones de atención de las E enfermeras que trabajan en su piso y de las P personas que llegan a la clínica ser atendidos.  
Cuando una persona necesita que la atiendan espera a lo sumo 5 minutos a que el médico lo haga, si pasado ese tiempo no lo hace, espera 10 minutos y vuelve a requerir la atención del médico. Si no es atendida tres veces, se enoja y se retira de la clínica. 
Cuando una enfermera requiere la atención del médico, si este no lo atiende inmediatamente le hace una nota y se la deja en el consultorio para que esta resuelva su pedido en el momento que pueda (el pedido puede ser que el médico le firme algún papel). Cuando la petición ha sido recibida por el médico o la nota ha sido dejada en el escritorio, continúa trabajando y haciendo más peticiones.
El médico atiende los pedidos dándole prioridad a los enfermos que llegan para ser atendidos. Cuando atiende un pedido, recibe la solicitud y la procesa durante un cierto tiempo. Cuando está libre aprovecha a procesar las notas dejadas por las enfermeras. 

```ada
Procedure Hospital is
    Task medico is
        Entry Enfermera;
        Entry Persona;
    end medico;

    Task archivo is
        Entry notaSolicitudEnfermera(pedido:IN string);
        Entry recibirNota(pedido:OUT string);
    end archivo;

    Task type persona;
    Task type enfermera;

    arrPersona: array (1..P) of personas;
    arrEnfermeras: array (1..E) of enfermeras;

    Task body persona is
        boolean atendido;
    Begin
        atendido = false;
        SELECT
            Medico.Persona;
            atendido = true;
        OR DELAY 300;
            null;
        END SELECT;
        if (atendido == false) then
        while (contador < 3 and atendido == false) do begin
            SELECT
                Medico.Persona;
                atendido = true;
            OR DELAY 600;
                contador++;
            END SELECT;
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
        END SELECT;
    End enfermera;

    Task body medico is
        String peticion;
    Begin
        loop
            SELECT
                accept Persona do
                    //Atiende persona;
                end Persona;
            OR
                when (Persona'count = 0) => accept Enfermera do
                    //Atiende enfermera;
                end Enfermera;
            ELSE
                SELECT
                    Archivo.recibirnota(peticion);
                    //Atiende peticion;
                OR ESLE
                    null;
                END SELECT;
            END SELECT;
        end loop;
    End medico;

    Task body archivo is
        Cola pedidos;
    Begin
        loop
            SELECT
                when (notaSolicitudEnfermera'count > 0) => 
                    accept notaSolicitudEnfermera (pedido:IN string) do
                        pedidos.push(pedido);
                    end notaSolicitudEnfermera;
            OR
                when (recibirNota'count > 0 and not empty(pedidos)) => 
                    accept recibirNota (pedido:OUT string) do
                        pedido = pedidos.pop();
                    end recibirNota;
            END SELECT
        end loop;
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
    end Servidor;

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
            END SELECT;
        end loop;
    end usuario;

    Task body Servidor is

    Begin
        loop
            accept Pedido (documento:IN String, respuesta:OUT String) do
                respuesta = documento.analizar();
            end Pedido;
        end loop;
    end;

Begin
    null;
end Universidad;
```

### Ejercicio 6
Se debe calcular el valor promedio de un vector de 1 millón de números enteros que se encuentra distribuido entre 10 procesos Worker (es decir, cada Worker tiene un vector de 100 mil números). Para ello, existe un Coordinador que determina el momento en que se debe realizar el cálculo de este promedio y que, además, se queda con el resultado. **Nota**: maximizar la concurrencia; este cálculo se hace una sola vez.