# Práctica 4

### Pasaje de Mensajes Asíncronos

1. Suponga que N clientes llegan a la cola de un banco y que serán atendidos por sus empleados. Analice el problema y defina qué procesos, recursos y canales/comunicaciones serán necesarios/convenientes para resolverlo. Luego, resuelva considerando las siguientes
situaciones:
    1. Existe un único empleado, el cual atiende por orden de llegada.
       ```
       chan cola(text);
       
       Process Persona[id: 0..N-1]{
         text solicitud = ...;
         send cola(solicitud);
       }

       Process Empleado{
         text solicitud;
         for int i= 0 to N-1 {
           receive cola(solicitud);
           //Atendiendo solicitud
         }
       }
       ```
    1. Ídem a) pero considerando que hay 2 empleados para atender, ¿qué debe modificarse en la solución anterior?
       ```
       chan cola(text);
       
       Process Persona[id: 0..N-1]{
         text solicitud = ...;
         send cola(solicitud);
       }

       Process Empleado[id: 0..1]{
         text solicitud;
         for int i= 0 to N-1 {
           receive cola(solicitud);
           //Atendiendo solicitud
         }
       }
       ```
    1. Ídem b) pero considerando que, si no hay clientes para atender, los empleados realizan tareas administrativas durante 15 minutos. ¿Se puede resolver sin usar procesos adicionales? ¿Qué consecuencias implicaría?
       
2.Se desea modelar el funcionamiento de un banco en el cual existen 5 cajas para realizar pagos. Existen P clientes que desean hacer un pago. Para esto, cada una selecciona la caja donde hay menos personas esperando; una vez seleccionada, espera a ser atendido. En cada caja, los clientes son atendidos por orden de llegada por los cajeros. Luego del pago, se les entrega un comprobante. Nota: maximizar la concurrencia.
```
chan hayAtencion[5](int, text);
chan esperaComprobante[P](text);
chan esperarCaja[P](int);
chan solicitarCaja(int);
chan hayPedido(bool);
chan liberarCaja(int);

Process Administrador {
  int vectorCajas[5] = ([5] 0);
  int idCliente, nroCaja, minCaja;
  bool pedido;
  while(true){
    receive hayPedido(pedido);    
    if(not empty(liberarCaja)){
      receive liberarCaja(nroCaja);
      vectorCajas[nroCaja]--
    }elseif(not empty(solicitarCaja) && empty(liberarCaja)){
      receive solicitarCaja(idCliente);
      minCaja = min(vectorCajas);
      vectorCajas[minCaja]++;
      send esperarCaja[idCliente](minCaja);
    }
  }
}
Process Cliente[id: 0..P-1]{
  text pago, comprobante;
  int nroCaja;
  send solicitarCaja(id);
  send hayPedido(true);
  receive esperarCaja[id](nroCaja);
  send hayAtencion[nroCaja](id, pago);
  receive esperaComprobante[id](comprobante);
  send liberarCaja(nroCaja);
  send hayPedido(true);
}
Process Caja[id: 0..4]{
  int idCliente;
  text pago, comprobante;
  while(true){
    receive hayAtencion[id](idCliente, pago);
    comprobante = _GenerarComprobante(pago)_;
    send esperaComprobante[idCliente](comprobante);
  }
}
```


3. Se debe modelar el funcionamiento de una casa de comida rápida, en la cual trabajan 2 cocineros y 3 vendedores, y que debe atender a C clientes. El modelado debe considerar que:
    - Cada cliente realiza un pedido y luego espera a que se lo entreguen.
    - Los pedidos que hacen los clientes son tomados por cualquiera de los vendedores y se lo pasan a los cocineros para que realicen el plato. Cuando no hay pedidos para atender, los vendedores aprovechan para reponer un pack de bebidas de la heladera (tardan entre 1 y 3 minutos para hacer esto).
    - Repetidamente cada cocinero toma un pedido pendiente dejado por los vendedores, lo cocina y se lo entrega directamente al cliente correspondiente.
*Nota*: maximizar la concurrencia.
```
chan solicitudPedido(int, text); // Este es por donde el cliente solicita
chan pedidoResuelto[C](text); // Este por donde el cliente lo recibe
chan hayPedido(int); // El vendedor avisa que esta disponible para atender un pedido
chan siguiente[2](int, text); // Dado que hay varios receptores, lo utiliza el coordinador para decidir a cual vendedor ira el pedido
chan pedidoPorHacer(int, text); // Por medio de este, el vendedor notifica al cocinero que hay un pedido para hacer

Process Cliente[id: 0..C-1]{
    text ped = ..., res;
    send solicitudPedido(id, ped);
    receive pedidoResuelto[id](res);
}
Process Coordinador{
    int idV, idC;
    text ped;
    while(true){
        receive hayPedido(idV);
        if(empty(solicitudPedido)){
            ped = "VACIO";
            idC = -1;
        }else{
            receive solicitudPedido(idC, ped);
        }
        send siguiente[idV](idC, ped);
    }
}
Process Cocinero[id: 0..2]{
    int idC;
    text ped, res;

    while(true){
        receive pedidoPorHacer(idC, ped);
        res = _Resolver(ped)_;
        send pedidoResuelto[idC](res);
    }
}
Process Vendedor[id: 0..1]{
    text ped;
    int idC;
    while(true){
        send hayPedido(id);
        receive siguiente[id](idC, ped);
        if(ped <> "VACIO"){
            send pedidoPorHacer(idC, ped);
        }else{
            delay(Random(121)+60);
        }
    }
}
```


4. Simular la atención en un locutorio con 10 cabinas telefónicas, el cual tiene un empleado que se encarga de atender a N clientes. Al llegar, cada cliente espera hasta que el empleado le indique a qué cabina ir, la usa y luego se dirige al empleado para pagarle. El empleado atiende a los clientes en el orden en que hacen los pedidos. A cada cliente se le entrega un ticket factura por la operación.
   
    a) Implemente una solución para el problema descrito.
   
    ```
    chan solicitarCabina(int); // El empleado informa que quiere una cabina
    chan esperarCabina[C](int);
    chan hayPedido(bool); // Esto para evitar el busy waiting
    chant esperarTicket[C](text);
    Process Cliente[id: 0..C-1]{
        int cabina;
        text ticket;
        send solicitarCabina(true);
        send hayPedido(true);
        receive esperarCabina(id)(cabina);
        _UsarCabina(cabina);_
        send pagar(id, cabina);
        send hayPedido(true);
        receive esperarTicket[id](ticket);
    }
    Process Empleado{
        int idC, cabina;
        bool p;
        bool cabinasOcupadas[10] = ([10] false);
        while(true){
            receive hayPedido(p);
            // Le da prioridad a los que la terminaron de usar
            if(not empty(pagar)){
                receive pagar(idC, cabina);
                cabinasOcupadas[cabina] = false;
                text ticket = Cobrar(idC);
                send esperarTicket[idC](ticket);
            }
            else if(not empty(solicitarCabina)){
                if(not HayCabinasOcupadas(cabinasOcupadas)){
                    receive pagar(idC, cabina);
                    cabinasOcupadas[cabina] = false;
                    text ticket = Cobrar(idC);
                    send esperarTicket[idC](ticket)
                }
                receive solicitarCabina(idC);
                cabina = encontrarCabinaLibre(cabinasOcupadas);
                cabinasOcupadas[cabina] = true;
                send esperarCabina[idCliente](cabina);
            }
        }
    }
    ```

    b) Modifique la solución implementada para que el empleado dé prioridad a los que terminaron de usar la cabina sobre los que están esperando para usarla.

    **Nota**: maximizar la concurrencia; suponga que hay una función Cobrar() llamada por el empleado que simula que el empleado le cobra al cliente.

5. Resolver la administración de 3 impresoras de una oficina. Las impresoras son usadas por N administrativos, los cuales están continuamente trabajando y cada tanto envían documentos a imprimir. Cada impresora, cuando está libre, toma un documento y lo imprime, de acuerdo con el orden de llegada.
   
    a) Implemente una solución para el problema descrito.
   
       ```
       chan documentos(text);
       Process Administrativo[id: 0..N]{
           while(true){
               text doc = ...;
               send documentos(doc);
           }
       }

       Process Impresora[id: 0..2]{
            while(true){
               text doc;
               receive documentos(doc)
               _Imprimir(doc)_;
           }
       }
       ```
   
    b) Modifique la solución implementada para que considere la presencia de un director de oficina que también usa las impresas, el cual tiene prioridad sobre los administrativos.

       ```
       chan documentosAdmin(text);
       chan documentosDir(text);
       chan aviso(bool);
       Process Administrativo[id: 0..N]{
           while(true){
               text doc = ...;
               send documentosAdmin(text);
               send aviso(true);
           }
       }

       Process Director{
           while(true){
               text doc = ...;
               send documentosDir(text);
               send aviso(true);
           }
       }

       Process Impresora[id: 0..2]{
           bool ok;
            while(true){
               receive aviso(ok);
               text doc;
               if(not empty(documentosDir)){
                    receive documentosDir(doc);
               }else if (not empty(documentosAdmin) and (empty(documentosDir))){
                    receive documentosAdmin(doc);
               }
               _Imprimir(doc)_;
           }
       }
       ```
   
    c) Modifique la solución (a) considerando que cada administrativo imprime 10 trabajos y que todos los procesos deben terminar su ejecución.

       ```
       chan documentos(text);
       chan hayDocumento(bool);
       chan esperaImpresion(text);
       
       Process Administrativo[id: 0..N]{
           for int i:0..9{
               text doc = ...;
               send documentos(doc);
               send hayDocumento(true); 
           }
       }

       Process Coordinador{
           bool ok;
           text doc;
           for int i: 0..((N*10)-1){
               receive hayDocumento(ok);
               receive documentos(doc);
               send esperaImpresion(doc);
           }
           for i: 0..2 {
                send esperaImpresion(null)
           }
       }

       Process Impresora[id: 0..2]{
           text doc;
           receive esperaImpresion(doc);
           while(doc != null){
               Imprimir(doc);
               receive esperaImpresion(doc);
           }
       }
       ```
   
    d) Modifique la solución (b) considerando que tanto el director como cada administrativo imprimen 10 trabajos y que todos los procesos deben terminar su ejecución.

       ```
       chan documentosAdmin(text);
       chan documentosDir(text);
       chan hayDocumento(bool);
       chan esperaImpresion(text);
       
       Process Administrativo[id: 0..N]{
           for int i:0..9{
               text doc = ...;
               send documentosAdmin(doc);
               send hayDocumento(true); 
           }
       }

       Process Director{
           for int i:0..9{
               text doc = ...;
               send documentosDir(doc);
               send hayDocumento(true); 
           }
       }

       Process Coordinador{
           bool ok;
           text doc;
           for int i: 0..((N*10)-1){
               receive hayDocumento(ok);
               text doc;
               if(not empty(documentosDir)){
                    receive documentosDir(doc);
               }else if (not empty(documentosAdmin) and (empty(documentosDir))){
                    receive documentosAdmin(doc);
               }
               send esperaImpresion(doc);
           }
           for i: 0..2 {
                send esperaImpresion(null)
           }
       }

       Process Impresora[id: 0..2]{
           text doc;
           receive esperaImpresion(doc);
           while(doc != null){
               Imprimir(doc);
               receive esperaImpresion(doc);
           }
       }
       ```
   
    e) Si la solución al ítem d) implica realizar Busy Waiting, modifíquela para evitarlo.

    Nota: ni los administrativos ni el director deben esperar a que se imprima el documento.

### Pasaje de Mensajes Síncronicos

1. Suponga que existe un antivirus distribuido que se compone de R procesos robots Examinadores y 1 proceso Analizador. Los procesos Examinadores están buscando continuamente posibles sitios web infectados; cada vez que encuentran uno avisan la dirección y luego continúan buscando. El proceso Analizador se encarga de hacer todas las pruebas necesarias con cada uno de los sitios encontrados por los robots para determinar si están o no infectados.
   
    a) Analice el problema y defina qué procesos, recursos y comunicaciones serán necesarios/convenientes para resolverlo.
   
    b) Implemente una solución con PMS sin tener en cuenta el orden de los pedidos.

    ```
    Process Examinador[id: 0..R-1]{
        text sitio;
        while true {
            sitio = BuscarSitio();
            Analizador?avisar(sitio);
        }
    }
        
    Process Analizador{
        text sitio;
        while(true){
            Examinador[*]!avisar(sitio);
            Analizar(sitio);
        }
    }
    
    ```
   
    c) Modifique el inciso (b) para que el Analizador resuelva los pedidos en el orden en que se hicieron.

    ```
    Process Examinador[id: 0..R-1]{
        text sitio;
        while(true){
            sitio = BuscarSitio();
            Administrador!find(sitio);
        }
    }
        
    Process Analizador{
        text sitio;
        while(true){
            Administrador!pedido();
            Administrador?send(sitio);
            Analizar(sitio);
        }
    }
    
    Process Administrador{
        cola Fila;
        text sitio;
        do Examinador[*]?find(sitio) -> push(Fila, sitio);
        [] (not empty(Buffer)); Analizador?pedido() -> {
            Fila.pop(sitio);
            Analizador!send(sitio);
        };
    }
    ```

2. En un laboratorio de genética veterinaria hay 3 empleados. El primero de ellos continuamente prepara las muestras de ADN; cada vez que termina, se la envía al segundo empleado y vuelve a su trabajo. El segundo empleado toma cada muestra de ADN preparada, arma el set de análisis que se deben realizar con ella y espera el resultado para archivarlo. Por último, el tercer empleado se encarga de realizar el análisis y devolverle el resultado al segundo empleado.

Dado que si usamos comunicación entre procesos, el proceso E1 debe esperar a que el E2 termine de recibir, por lo que estaría limitando su concurrencia. Se debe usar un Admin, que tenga un buffer.

```
Process E1 {
    text muestra;
    while (true) {
        muestra = PrepararMuestra();
        Admin!muestraLista(muestra);
    }
}

Process E2 {
    text m, set, analisis;
    while(true){
        Admin!aviso();
        Admin?recibirMuestra(m);
        set = ArmarSet(m);
        E3!recibirSet(set);
        E3?recibirAnalisis(analisis);
        GuardarResultado(analisis);
    }
}

Process E3 {
    text s, a;
    while(true){
        E2?recibirSet(s);
        a = ArmarAnalisis(s);
        E2!recibirAnalisis(a);
    }
}

Process Admin {
    cola Buffer;
    text m;

    do E1?muestraLista(m) -> push(Buffer, m);
    [] (not empty(Buffer)); E2?aviso(); -> E2!recibirMuestra(Buffer.pop());
    od;
}
```

3. En un examen final hay N alumnos y P profesores. Cada alumno resuelve su examen, lo entrega y espera a que alguno de los profesores lo corrija y le indique la nota. Los profesores corrigen los exámenes respetando el orden en que los alumnos van entregando.
      
    a) Considerando que P=1.

    ```
    Process Alumno[id: 0..N-1]{
        text e = ...;
        int n;
        text r = ResolverExamen(e);
        Admin!entregaExamen(e, id);
        Profesor?recibirNota(n);
    }

    Process Profesor{
        text e;
        int idA, n;
        for int i: 0..N-1{
            Admin!aviso();
            Admin?corregir(e, idA);
            n = CorregirExamen(e);
            Alumno[idA]!recobirNota(n);
        }
    }

    Process Admin{
        cola Fila;
        text e;
        int idA;
        do Alumno[*]?entregaExamen(e, idA); -> push(Fila, (e, idA));
        [] (not (empty(Fila))); Profesor?aviso(); -> {
            Fila.pop(e, idA);
            Profesor!corregir(e, idA);
        }
        od;
    }
    ```
   
    b) Considerando que P>1.

    ```
    Process Alumno[id: 0..N-1]{
        text e = ...;
        int n;
        text r = ResolverExamen(e);
        Admin!entregaExamen(e, id);
        Profesor[*]?recibirNota(n);
    }

    Process Profesor[id: 0..P-1]{
        text e;
        int idA, n;
        Admin!aviso(id);
        Admin?corregir(e, idA);
        while(e != null){
            n = CorregirExamen(e);
            Alumno[idA]!recibirNota(n);
            Admin!aviso(id);
            Admin?corregir(e, idA);
        }
    }

    Process Admin{
        cola Fila;
        text e;
        int idA, idP;
        do Alumno[*]?entregaExamen(e, idA); -> push(Fila, (e, idA));
        [] (not (empty(Fila))); Profesor[*]?aviso(idP); -> {
            Fila.pop(e, idA);
            Profesor[idP]!corregir(e, idA);
        }
        od;
        for int i: 0..P-1{
            Profesor[*]?aviso(idP)
            Profespr[idP]!corregir(null, -1);
        }
    }
    ```
   
    c) Ídem b) pero considerando que los alumnos no comienzan a realizar su examen hasta que todos hayan llegado al aula.

    ```
    Process Alumno[id: 0..N-1]{
        text e = ...;
        int n;
        Barrera!llegada();
        Barrera?empezar();
        text r = ResolverExamen(e);
        Admin!entregaExamen(e, id);
        Profesor[*]?recibirNota(n);
    }

    Process Profesor[id: 0..P-1]{
        text e;
        int idA, n;
        Admin!aviso(id);
        Admin?corregir(e, idA);
        while(e != null){
            n = CorregirExamen(e);
            Alumno[idA]!recibirNota(n);
            Admin!aviso(id);
            Admin?corregir(e, idA);
        }
    }

    Process Admin{
        cola Fila;
        text e;
        int idA, idP;
        do Alumno[*]?entregaExamen(e, idA); -> push(Fila, (e, idA));
        [] (not (empty(Fila))); Profesor[*]?aviso(idP); -> {
            Fila.pop(e, idA);
            Profesor[idP]!corregir(e, idA);
        }
        od;
        for int i: 0..P-1{
            Profesor[*]?aviso(idP)
            Profespr[idP]!corregir(null, -1);
        }
    }

    Process Barrera {
        for int i: 0..N-1{
            Alumno[*]?llegada();
        }
        for int i: 0..N-1{
            Alumno[i]!empezar();
        }
    }
    ```

   Nota: maximizar la concurrencia; no generar demora innecesaria; todos los procesos deben terminar su ejecución

4. En una exposición aeronáutica hay un simulador de vuelo (que debe ser usado con exclusión mutua) y un empleado encargado de administrar su uso. Hay P personas que esperan a que el empleado lo deje acceder al simulador, lo usa por un rato y se retira.
   
   a) Implemente una solución donde el empleado sólo se ocupa de garantizar la exclusión mutua (sin importar el orden).

    ```
    Process Persona[id: 0..P-1]{
        Empleado!llegar(id);
        Empleado?usarMaquina();
        UsarMaquina();
        Empleado!avisarFin();
    }
    Process Empleado{
        int idP;
        for int i: 0..P-1{
            Persona[*]?llegar(idP);
            Persona[idP]!usarMaquina();
            Persona[idP]?avisarFin();
        }
    }
    ```
   
   b) Modifique la solución anterior para que el empleado los deje acceder según el orden de su identificador (hasta que la persona i no lo haya usado, la persona i+1 debe esperar).

   ```
   Process Persona[id: 0..P-1]{
       Empleado!llegar();
       Empleado?usarMaquina();
       UsarMaquina();
       Empleado!avisarFin();
   }
   Process Empleado{
       int idP;
       for int i: 0..P-1{
           Persona[i]?llegar();
           Persona[i]!usarMaquina();
           Persona[i]?avisarFin();
       }
   }
    ```
   
   c) Modifique la solución a) para que el empleado considere el orden de llegada para dar acceso al simulador.

   ```
    Process Persona[id: 0..P-1]{
       Admin!llegar(id);
       Empleado?usarMaquina();
       UsarMaquina();
       Empleado!avisarFin();
    }
    Process Admin{
       int idP;
       cola Fila;
       int c = 0;
       do (c < P); Persona[*]?llegar(idP); -> push(Fila, idP);
       [] (not empty(Fila)); Empleado?libre(); -> {
           Empleado!avisar(Fila.pop());
           c++;
       }
    }
    Process Empleado {
       int idP;
       for int i: 0..P-1{
           Admin!libre();
           Admin?avisar(idP);
           Persona[idP]!usarMaquina();
           Persona[idP]?avisarFin();
       }
    }
    ```

   Nota: cada persona usa sólo una vez el simulador. 

5. En un estadio de fútbol hay una máquina expendedora de gaseosas que debe ser usada por E Espectadores de acuerdo con el orden de llegada. Cuando el espectador accede a la máquina en su turno usa la máquina y luego se retira para dejar al siguiente. Nota: cada Espectador una sólo una vez la máquina.

```
Process Espectador[id: 0..E-1]{
    Admin!llegar(id);
    Admin?usarMaquina();
    UsarMaquina();
    Admin?salir();
}

Process Admin{
    bool libre = true;
    int idE;
    cola c;
    
    while(true){
        if(libre); Espectador[*]?llegar(idE); -> {
            libre = false;
            Espectador[idE]!usarMaquina();
        }
        [] (not libre); Espectador[*]?llegar(idE); -> c.push(idE);
        [] Espectador[idE]?salir(); -> {
            if(empty(c)){
                libre = true;
            }else{
                c.pop(idE);
                Espectado[idE]!usarMaquina();
            }
        }
    }
}
```
