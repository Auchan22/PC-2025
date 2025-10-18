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

5. 
