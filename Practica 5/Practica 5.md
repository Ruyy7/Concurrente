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
        Entry Pedido(idCliente:IN int; comprobante:OUT int);
    End empleado;

    Task type cliente;

    arrayClientes: array (1..N) of cliente;

    Task body Cliente is
        int id;
    Begin
        Empleado.Pedido(id,comprobante);
    End Cliente;

    Task body Empleado is

    Begin
        loop
            accept Pedido (idCliente:IN int; comprobante:OUT in)
                comprobante = generarComprobante(idCliente);
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
        Entry Pedido(idCliente:IN int; comprobante:OUT int);
    End empleado;

    Task type cliente;

    arrayClientes: array (1..N) of cliente;

    Task body Cliente is
        int id;
    Begin
        SELECT
            Empleado.Pedido(id,comprobante);
        OR DELAY 600
            null;
        END SELECT;
    End Cliente;

    Task body Empleado is

    Begin
        loop
            accept Pedido (idCliente:IN int; comprobante:OUT in)
                comprobante = generarComprobante(idCliente);
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
        Entry Pedido(idCliente:IN int; comprobante:OUT int);
    End empleado;

    Task type cliente;

    arrayClientes: array (1..N) of cliente;

    Task body Cliente is
        int id;
    Begin
        SELECT
            Empleado.Pedido(id,comprobante);
        ELSE
            null;
        END SELECT;
    End Cliente;

    Task body Empleado is

    Begin
        loop
            accept Pedido (idCliente:IN int; comprobante:OUT in)
                comprobante = generarComprobante(idCliente);
            end Pedido;
        end loop;
    End Empleado;
End Banco1;
```

### D
```ada
Procedure Banco1 is
    Task empleado is
        Entry Pedido(idCliente:IN int; comprobante:OUT int);
    End empleado;

    Task type cliente;

    arrayClientes: array (1..N) of cliente;

    Task body Cliente is
        int id; boolean atendido;
    Begin
        atendido = false;
        SELECT
            Empleado.Pedido(id,comprobante);
            atendido = true;
        OR DELAY 600
            null;
        END SELECT;
        If (atendido == false) then
            SELECT
                Empleado.Pedido(id,comprobante);
            OR ELSE
                null;
            END SELECT;
        end if;
    End Cliente;

    Task body Empleado is

    Begin
        loop
            accept Pedido (idCliente:IN int; comprobante:OUT in)
                comprobante = generarComprobante(idCliente);
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