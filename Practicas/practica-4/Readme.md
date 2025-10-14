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
