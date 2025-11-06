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
   Process Sistema IS

      Task ProcesoA;
      Task ProcesoB;
      
      Task Central IS
         Entry SignalA(SA: IN int);
         Entry SignalB(SB: IN int);
         Entry OutOfTime();
      END Central;

      Task Contador IS
         Entry start();
      END Contador;
      TASK BODY Contador IS
      BEGIN
         ACCEPT start();
         DELAY 180.0;
         Central.OutOfTime();
      END Contador;

      TASK BODY ProcesoA IS
         SA: int;
      BEGIN
         LOOP
            SA := crearSignal();
            SELECT
               Central.SiganlA(SA);
            OR DELAY 120.0
               NULL;
            END SELECT;
         END LOOP;
      END ProcesoA;

      TASK BODY ProcesoB IS
         SB: int;
      BEGIN
         LOOP
            SB := crearSignal();
            SELECT
               Central.SignalB(SB);
               SB := crearSignal();
            ELSE
               DELAY 60.0;
            END SELECT;
         END LOOP;
      END ProcesoA;

      Task Body Central IS
         menos_tres_min: bool := true;
      BEGIN
         ACCEPT SignalA(SA);

         loop
            SELECT
               ACCEPT SignalA(SA);
             OR
               ACCEPT SignalB(SB);
               Contador.start();
               menos_tres_min := false;
               WHILE(not(menos_tres_min)){
                  SELECT
                     WHEN(OutOfTime'Count = 0) =>
                        ACCEPT SignalB(SB);
                  OR
                     ACCEPT OutOfTime() DO
                        menos_tres_min := true;
                     END OutOfTime;
                  END SELECT;
               }
            END SELECT;
         END LOOP;
      END Central;

   BEGIN
      null;
   END;
   ```

4. En una clínica existe un médico de guardia que recibe continuamente peticiones de atención de las E enfermeras que trabajan en su piso y de las P personas que llegan a la clínica ser atendidos. Cuando una persona necesita que la atiendan espera a lo sumo 5 minutos a que el médico lo haga, si pasado ese tiempo no lo hace, espera 10 minutos y vuelve a requerir la atención del médico. Si no es atendida tres veces, se enoja y se retira de la clínica. Cuando una enfermera requiere la atención del médico, si este no lo atiende inmediatamente le hace una nota y se la deja en el consultorio para que esta resuelva su pedido en el momento que pueda (el pedido puede ser que el médico le firme algún papel). Cuando la petición ha sido recibida por el médico o la nota ha sido dejada en el escritorio, continúa trabajando y haciendo más peticiones. El médico atiende los pedidos dándole prioridad a los enfermos que llegan para ser atendidos. Cuando atiende un pedido, recibe la solicitud y la procesa durante un cierto tiempo. Cuando está libre aprovecha a procesar las notas dejadas por las enfermeras.

```ada
Process Clinica

   TASK Medico IS
      ENTRY pedidoPaciente(pedido: IN text);
      ENTRY pedidoEnfermera(pedido: IN text);
   END Medico;
   
   TASK Escritorio IS
      ENTRY dejarNota(nota: IN text);
      ENTRY recibirNotas(nota: IN OUT text);
   END Escritorio;
   
   TASK TYPE Enfermera;
   
   TASK TYPE Persona;

   arrEnfermeras: array(1..E) of Enfermera;
   arrPersonas: array(1..P) of Persona;

   TASK BODY Medico IS
      trabajo: text;
   BEGIN
      LOOP
         SELECT
            ACCEPT pedidoPaciente(pedido: IN text) DO
               trabajo := AtenderPaciente(pedido);
            END pedidoPaciente;
         OR
            WHEN (pedidoPaciente'count = 0) =>
               ACCEPT pedidoEnfermera(pedido: IN text) DO
                  trabajo := AtenderEnfermera(pedido);
               END pedidoEnfermera;
         ELSE
            SELECT
               Escritorio.recibirNotas(trabajo);
               trabajo := ProcesarNota(trabajo);
            OR
               null;
            END SELECT;
         END SELECT;
      END LOOP;
   END Medico;

   TASK BODY Escritorio IS
      trabajos: cola;
   BEGIN
      LOOP
         SELECT
            ACCEPT dejarNota(nota: IN text) DO
               trabajos.push(nota);
            END dejarNota;
         OR
            WHEN (not(trabajos.empty())) =>
               ACCEPT recibirNotas(nota: IN OUT text) DO
                  nota := trabajos.pop();
               END recibirNotas;
         END SELECT;
      END LOOP;
   END Escritorio;

   TASK BODY Enfermera IS
      pedido: text;
   BEGIN
      LOOP
         pedido := GenerarPedido();
         SELECT
            Medico.pedidoEnfermera(pedido);
         OR
            Escritorio.dejarNota(pedido);
         END SELECT;
      END LOOP;
   END Enfermera;

   TASK BODY Persona IS
      cantIntentos: int := 0;
      continuar: bool := true;
      pedido: text;
   BEGIN
      pedido := GenerarPedido();
      WHILE (cantIntentos < 3) AND (continuar) LOOP
         SELECT
            Medico.pedidoPaciente(pedido);
            continuar := false;
         OR DELAY 300.0
            cantIntentos := cantIntentos + 1;
            IF (cantIntentos < 3) THEN
               DELAY 600.0;
            END IF;
         END SELECT;
      END LOOP;
   END Persona;

BEGIN
   null;
END;
```

5. En una playa hay 5 equipos de 4 personas cada uno (en total son 20 personas donde cada una conoce previamente a que equipo pertenece). Cuando las personas van llegando esperan con los de su equipo hasta que el mismo esté completo (hayan llegado los 4 integrantes), a partir de ese momento el equipo comienza a jugar. El juego consiste en que cada integrante del grupo junta 15 monedas de a una en una playa (las monedas pueden ser de 1, 2 o 5 pesos) y se suman los montos de las 60 monedas conseguidas en el grupo. Al finalizar cada persona debe conocer el grupo que más dinero junto. Nota: maximizar la concurrencia. Suponga que para simular la búsqueda de una moneda por parte de una persona existe una función Moneda() que retorna el valor de la moneda encontrada.

```ada
PROCESS Playa IS
   TASK Admin IS
   BEGIN
      ENTRY CalcularGanador(idEquipo: IN integer; monto: IN integer);
      ENTRY IdentificarGanador(ganador: OUT integer);
   END Admin;

   TASK TYPE Equipo IS
   BEGIN
      ENTRY recibirId(id: IN integer);
      ENTRY llegada;
      ENTRY comenzar;
      ENTRY finalizar(monto: IN integer);
   END Equipo;

   TASK TYPE Persona;

   arrEquipos: ARRAY(1..5) of Equipo;
   arrPersonas: ARRAY(1..20) of Persona;

   TASK BODY Admin IS
      max: integer := -1;
      maxEquipo: integer;
   BEGIN
      FOR i IN 1..5 LOOP
         ACCEPT CalcularGanador(idEquipo: IN integer; monto: IN integer) DO
            IF (monto > max) THEN
               max := monto;
               maxEquipo := idEquipo;
            END IF;
         END CalcularGanador;
      END LOOP;
      FOR i IN 1..20 LOOP
         ACCEPT IdentificarGanador(ganador: OUT integer) DO
            ganador := maxEquipo;
         END IdentificarGanador;
      END LOOP;
   END Admin;

   TASK BODY Equipo IS
      idEquipo: integer;
      montoTotal: integer := 0;
   BEGIN
      ACCEPT recibirId(id: IN integer) DO
         idEquipo := id;
      END recibirId;
      FOR i IN 1..4 LOOP
         ACCEPT llegada;
      END LOOP;
      FOR i IN 1..4 LOOP
         ACCEPT comenzar;
      END LOOP;
      FOR i IN 1..4 LOOP
         ACCEPT finalizar(monto: IN integer) DO
            montoTotal := montoTotal + monto;
         END finalizar;
      END LOOP;
      Admin.CalcularGanador(idEquipo, montoTotal);
   END Equipo;

   TASK BODY Persona IS
      equipo: integer := ObtenerNroGrupo();
      monto: integer := 0;
      ganador: integer;
   BEGIN
      Equipo(equipo).llegada;
      Equipo(equipo).comenzar;
      FOR i IN 1..15 LOOP
         monto := monto + Moneda();
      END LOOP;
      Equipo(equipo).finalizar(monto);
      Admin.IdentificarGanador(ganador);
   END Persona;

BEGIN
   FOR i IN 1..5 LOOP
      arrEquipos(i).recibirId(i);
   END LOOP;
END;
```

6. Se debe calcular el valor promedio de un vector de 1 millón de números enteros que se encuentra distribuido entre 10 procesos Worker (es decir, cada Worker tiene un vector de 100 mil números). Para ello, existe un Coordinador que determina el momento en que se debe realizar el cálculo de este promedio y que, además, se queda con el resultado. Nota: maximizar la concurrencia; este cálculo se hace una sola vez.

```ada
PROCESS Calculo IS

   TASK TYPE Worker;

   TASK Coordinador IS
      ENTRY comenzar;
      ENTRY finalizar(valor: IN integer);
   END Coordinador;

   arrW: array(1..10) of Worker;

   TASK BODY Worker IS
      arrN: array(1..100000) of integer := InicializarVector();
      valor: integer := 0;
   BEGIN
      Coordinador.comenzar;
      FOR i IN 1..100000 LOOP
         valor := valor + arrN(i);
      END LOOP;
      Coordinador.finalizar(valor);
   END Worker;

   TASK BODY Coordinador IS
      avg: float;
      total: integer := 0;
   BEGIN
      FOR i IN 1..20 LOOP
         SELECT
            ACCEPT comenzar;
         OR
            ACCEPT finalizar(valor: IN integer) DO
               total := total + valor;
            END finalizar;
         END SELECT;
      END LOOP;
      avg := (total / 1000000);
   END Coordinador;
BEGIN
   null;
END Calculo;
```

7. Hay un sistema de reconocimiento de huellas dactilares de la policía que tiene 8 Servidores para realizar el reconocimiento, cada uno de ellos trabajando con una Base de Datos propia; a su vez hay un Especialista que utiliza indefinidamente. El sistema funciona de la siguiente manera: el Especialista toma una imagen de una huella (TEST) y se la envía a los servidores para que cada uno de ellos le devuelva el código y el valor de similitud de la huella que más se asemeja a TEST en su BD; al final del procesamiento, el especialista debe conocer el código de la huella con mayor valor de similitud entre las devueltas por los 8 servidores. Cuando ha terminado de procesar una huella comienza nuevamente todo el ciclo. Nota: suponga que existe una función Buscar(test, código, valor) que utiliza cada Servidor donde recibe como parámetro de entrada la huella test, y devuelve como parámetros de salida el código y el valor de similitud de la huella más parecida a test en la BD correspondiente. Maximizar la concurrencia y no generar demora innecesaria.
