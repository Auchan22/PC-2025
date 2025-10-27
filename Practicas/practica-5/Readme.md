# Práctica 5 - Rendezvous (ADA)

1. Se requiere modelar un puente de un único sentido que soporta hasta 5 unidades de peso. El peso de los vehículos depende del tipo: cada auto pesa 1 unidad, cada camioneta pesa 2 unidades y cada camión 3 unidades. Suponga que hay una cantidad innumerable de vehículos (A autos, B camionetas y C camiones). Analice el problema y defina qué tareas, recursos y sincronizaciones serán necesarios/convenientes para resolverlo.
   
  a. Realice la solución suponiendo que todos los vehículos tienen la misma prioridad.

  ```ada
  Process Puente is

  Task Type Vehiculo;

  Task Coordinador is
    Entry PasarAuto();
    Entry PasarCamion();
    Entry PasarCamioneta();
    Entry Salir(P: IN int);
  End Coordinador;

  arrV = ARRAY(1..N) of Vehiculo;

  Task Body Vehiculo is
    tipo: text := ...;
  Begin
    if(tipo == 'A'){
      Coordinador.PasarAuto();
      Coordinador.Salir(1);
    }elseif(tipo == 'C') {
      Coordinador.PasarCamioneta();
      Coordinador.Salir(2);
    }elseif(tipo == 'CA'){
      Coordinador.PasarCamion();
      Coordinador.Salir(3);
    }
  End Vehiculo;

  Task Body Coordinador is
    total: int := 0;
  Begin
    loop
      SELECT
        when (total < 5) =>
          ACCEPT PasarAuto() IS
            total += 1;
          End PasarAuto;
      OR
        when (total < 4) =>
          ACCEPT PasarCamioneta() IS
            total += 2;
          END PasarCamioneta;
      OR
        when (total < 3) =>
          ACCEPT PasarCamion() IS
            total += 3;
          END PasarCamion;
      OR
          ACCEPT Salir(P: IN Int) IS
            total -= P;
          END Salir;
      END Select;
    end loop;
  End Coordinador;
  
  Begin
    null;
  End Puente;
  ```

  b. Modifique la solución para que tengan mayor prioridad los camiones que el resto de los vehículos.

  ```ada
  Process Puente is

  Task Type Vehiculo;

  Task Coordinador is
    Entry PasarAuto();
    Entry PasarCamion();
    Entry PasarCamioneta();
    Entry Salir(P: IN int);
  End Coordinador;

  arrV = ARRAY(1..N) of Vehiculo;

  Task Body Vehiculo is
    tipo: text := ...;
  Begin
    if(tipo == 'A'){
      Coordinador.PasarAuto();
      Coordinador.Salir(1);
    }elseif(tipo == 'C') {
      Coordinador.PasarCamioneta();
      Coordinador.Salir(2);
    }elseif(tipo == 'CA'){
      Coordinador.PasarCamion();
      Coordinador.Salir(3);
    }
  End Vehiculo;

  Task Body Coordinador is
    total: int := 0;
  Begin
    loop
      SELECT
        when (total < 5) and (PasarCamion'count = 0) =>
          ACCEPT PasarAuto() IS
            total += 1;
          End PasarAuto;
      OR
        when (total < 4) and (PasarCamion'count = 0) =>
          ACCEPT PasarCamioneta() IS
            total += 2;
          END PasarCamioneta;
      OR
        when (total < 3) =>
          ACCEPT PasarCamion() IS
            total += 3;
          END PasarCamion;
      OR
          ACCEPT Salir(P: IN Int) IS
            total -= P;
          END Salir;
      END Select;
    end loop;
  End Coordinador;
  
  Begin
    null;
  End Puente;
  ```

2. Se quiere modelar el funcionamiento de un banco, al cual llegan clientes que deben realizar un pago y retirar un comprobante. Existe un único empleado en el banco, el cual atiende de acuerdo con el orden de llegada.
   
   a) Implemente una solución donde los clientes llegan y se retiran sólo después de haber sido atendidos.

   ```ada
   Process Banco is
      Task Type Cliente;

      Task Empleado is
         Entry Pedido(monto: IN integer; comprobante: OUT text);
      End Empleado;

      arrC: array(1..N) of Cliente;

      Task Body Cliente is
         monto: int = ...;
         comprobante: text;
      Begin
         Empleado.Pedido(monto, comprobante);
      End Cliente;

      Task Body Empleado is
      Begin
         loop
            Accept Pedido(m: IN integer; c: OUT text) do
               c = GenerarComprobante(m);
            end Pedido;
         end loop;
      End Empleado;

   Begin
      null;
   End Banco;
   ```

   b) Implemente una solución donde los clientes se retiran si esperan más de 10 minutos para realizar el pago.

   ```ada
   Process Banco is
      Task Type Cliente;

      Task Empleado is
         Entry Pedido(monto: IN integer; comprobante: OUT text);
      End Empleado;

      arrC: array(1..N) of Cliente;

      Task Body Cliente is
         monto: int := ...;
         comprobante: text;
      Begin
         SELECT
            Empleado.Pedido(monto, comprobante);
         OR DELAY 600.0
            NULL;
         END SELECT;
      End Cliente;

      Task Body Empleado is
      Begin
         loop
            Accept Pedido(m: IN integer; c: OUT text) do
               c := GenerarComprobante(m);
            end Pedido;
         end loop;
      End Empleado;

   Begin
      null;
   End Banco;
   ```
   
   c) Implemente una solución donde los clientes se retiran si no son atendidos inmediatamente.

   ```ada
   Process Banco is
      Task Type Cliente;

      Task Empleado is
         Entry Pedido(monto: IN integer; comprobante: OUT text);
      End Empleado;

      arrC: array(1..N) of Cliente;

      Task Body Cliente is
         monto: int := ...;
         comprobante: text;
      Begin
         SELECT
            Empleado.Pedido(monto, comprobante);
         OR
            NULL;
         END SELECT;
      End Cliente;

      Task Body Empleado is
      Begin
         loop
            Accept Pedido(m: IN integer; c: OUT text) do
               c := GenerarComprobante(m);
            end Pedido;
         end loop;
      End Empleado;

   Begin
      null;
   End Banco;
   ```
   
   d) Implemente una solución donde los clientes esperan a lo sumo 10 minutos para ser atendidos. Si pasado ese lapso no fueron atendidos, entonces solicitan atención una vez más y se retiran si no son atendidos inmediatamente.

   ```ada
   Process Banco is
      Task Type Cliente;

      Task Empleado is
         Entry Pedido(monto: IN integer; comprobante: OUT text);
      End Empleado;

      arrC: array(1..N) of Cliente;

      Task Body Cliente is
         monto: int := ...;
         comprobante: text;
      Begin
         SELECT
            Empleado.Pedido(monto, comprobante);
         OR DELAY 600.0
            Empleado.Pedido(monto, comprobante);
         OR DELAY 600.0
            NULL;
         END SELECT;
      End Cliente;

      Task Body Empleado is
      Begin
         loop
            Accept Pedido(m: IN integer; c: OUT text) do
               c := GenerarComprobante(m);
            end Pedido;
         end loop;
      End Empleado;

   Begin
      null;
   End Banco;
   ```

3. Se dispone de un sistema compuesto por 1 central y 2 procesos periféricos, que se comunican continuamente. Se requiere modelar su funcionamiento considerando las siguientes condiciones:

   - La central siempre comienza su ejecución tomando una señal del proceso 1; luego toma aleatoriamente señales de cualquiera de los dos indefinidamente. Al recibir una señal de proceso 2, recibe señales del mismo proceso durante 3 minutos.
     
   - Los procesos periféricos envían señales continuamente a la central. La señal del proceso 1 será considerada vieja (se deshecha) si en 2 minutos no fue recibida. Si la señal del proceso 2 no puede ser recibida inmediatamente, entonces espera 1 minuto y vuelve a mandarla (no se deshecha).
  
   ```ada
   
   ```
