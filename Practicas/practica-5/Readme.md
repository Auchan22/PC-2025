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
