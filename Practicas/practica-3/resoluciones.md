# Práctica 3 - Monitores

1. Monitor Puente
    ```
    Monitor Puente
        cond cola;
        int cant= 0;

        Procedure entrarPuente ()
            while ( cant > 0) wait (cola);
            cant = cant + 1;
        end;

        Procedure salirPuente ()
            cant = cant – 1;
            signal(cola);
        end;
    End Monitor;

    Process Auto [a:1..M]
        Puente.entrarPuente (a);
        “el auto cruza el puente”
        Puente.salirPuente(a);
    End Process;
    ```
    a. **¿El código funciona correctamente? Justifique su respuesta:** Si, el código planteado propone una solución decente al problema de utilizar el recurso compartido (el puente). Al llegar el 1er auto, ya se bloquea la posibilidad e acceso, por lo que los demás autos se encolan en la "cola de llegada". Cuando un auto termina de pasar, `Puente.salirPuente()` lo que hace es desencolar al proximo proceso que este en la cola.
    
    b. **¿Se podría simplificar el programa? ¿Sin monitor? ¿Menos procedimientos? ¿Sin variable condition? En caso afirmativo, rescriba el código.**: Si, se puede modificar el programa, dado que en este caso necesitamos solo la *exclusión mutua*, ofrecida por default por los monitores. Como en la EP, el monitor debe representar el proceso de cruzar el puente. 
    ```
    Monitor Puente
    
        Procedure cruzarPuente()
            # Auto cruza el puente
        end;
    End Monitor;

    Process Auto [a:1..M]
        Puente.cruzarPuente();
    End Process;
    ```

    c. **¿La solución original respeta el orden de llegada de los vehículos? Si rescribió el código en el punto b), ¿esa solución respeta el orden de llegada?:** No, tanto en el código original como en el inciso b no se respeta el orden de llegada, esto debido a que apenas inician los procesos, pasan a competir por el recurso.

2. Existen N procesos que deben leer información de una base de datos, la cual es administrada por un motor que admite una cantidad limitada de consultas simultáneas.

    a) **Analice el problema y defina qué procesos, recursos y monitores/sincronizaciones serán necesarios/convenientes para resolverlo.** Se va a requerir un monitor `Motor`, el cual permita el uso del recurso compartido (base de datos), y los correspondientes procesos `process[i]`. El monitor en su interior va a tener su variable de condición, que permitira controlar la entrada al recurso (lectura de la base de datos).
    
    b) **Implemente el acceso a la base por parte de los procesos, sabiendo que el motor de base de datos puede atender a lo sumo 5 consultas de lectura simultáneas.**
    ```
    Monitor Motor{
        cond cola;
        int cant_consulta;

        Procedure entrarBase(){
            while(cant_consulta == 5) wait(cola);
            cant_consulta++;
        }

        Procedure salirBase(){
            cant_consulta--;
            signal(cola);
        }
    }

    Process process[id = 0 to N-1]{
        Motor.entrarBase();
        // lee datos de la bd
        Motor.salirBase();
    }
    ```

3. Existen N personas que deben fotocopiar un documento. La fotocopiadora sólo puede ser usada por una persona a la vez. Analice el problema y defina qué procesos, recursos y monitores serán necesarios/convenientes, además de las posibles sincronizaciones requeridas para resolver el problema. Luego, resuelva considerando las siguientes situaciones:

    a) Implemente una solución suponiendo no importa el orden de uso. Existe una función Fotocopiar() que simula el uso de la fotocopiadora.
    ```
    Monitor Fotocopiadora{
        Procedure fotocopiar(Documento d){
            Fotocopiar(d);
        }
    }

    Process Persona[id = 0 to N-1]{
        Documento d;
        Fotocopiadora.fotocopiar(d)
    }
    ```
    | Dado que no importa el orden, un monitor hace exclusión mutua por si solo.

    b) Modifique la solución de (a) para el caso en que se deba respetar el orden de llegada.
    ```
    Monitor Fotocopiadora{
        bool libre = true;
        cond cola;
        int cant = 0;

        Procedure entrar(){
            if(not(libre)){
                cant++;
                wait(cola);
            }else{
                libre = false;
            }
        }

        Procedure salir(){
            if(cant > 0){
                cant--;
                signal(cola);
            }else{
                libre = true;
            }
        }
    }

    Process Persona[id = 0 to N-1]{
        Documento d;
        Fotocopiadora.entrar();
        Fotocopiar(d);
        Fotocopiadora.salir();
    }
    ```

    c) Modifique la solución de (b) para el caso en que se deba dar prioridad de acuerdo con la edad de cada persona (cuando la fotocopiadora está libre la debe usar la persona de mayor edad entre las que estén esperando para usarla).
    ```
    Monitor Fotocopiadora{
        bool libre = true;
        cond cola[N];
        int cant = 0, idAux;
        colaOrdenada fila;

        Procedure entrar(idP, edad: in int){
            if(not(libre)){
                insertar(fila, idP, edad);
                cant++;
                wait(cola[idP]);
            }else{
                libre = false;
            }
        }

        Procedure salir(){
            if(cant > 0){
                cant--;
                sacar(fila, idAux)
                signal(cola[idAux]);
            }else{
                libre = true;
            }
        }
    }

    Process Persona[id = 0 to N-1]{
        Documento d;
        boold edad = ...;
        Fotocopiadora.entrar(id, edad);
        Fotocopiar(d);
        Fotocopiadora.salir();
    }
    ```

    d) Modifique la solución de (a) para el caso en que se deba respetar estrictamente el orden dado por el identificador del proceso (la persona X no puede usar la fotocopiadora hasta que no haya terminado de usarla la persona X-1).
    ```
    Monitor Fotocopiadora{
        cond cola[N];
        int idSiguiente = 0;

        Procedure usar(id: in int){
            if(idSiguiente != id){
                wait(cola[id]);
            }
        }

        Procedure salir(){
            idSiguiente++;
            signal(cola[idSiguiente]);
        }
    }

    Process Persona[id = 0 to N-1]{
        Documento d;
        Fotocopiadora.usar(id);
        Fotocopiar(d);
        Fotocopiadora.salir();
    }
    ```

    e) Modifique la solución de (b) para el caso en que además haya un Empleado que le indica a cada persona cuando debe usar la fotocopiadora.
    ```
    Monitor Fotocopiadora{
        cond fin;
        cond persona;
        cond empleado;
        int esperando = 0;

        Procedure asignar(idP: in int){
            if(esperando == 0){
                wait(empleado);
            }
            esperando--;
            signal(persona[idP]);
            wait(fin);
        
        }

        Procedure usar(){
            signal(empleado);
            esperando++;
            wait(persona);
        }

        Procedure dejar(){
            signal(fin);
        }
    }

    Process Persona[id = 0 to N-1]{
        Documento d;
        Fotocopiadora.usar();
        Fotocopiar(d);
        Fotocopiadora.dejar();
    }

    Process Empleado{
        int i;
        for i = 0 to N-1 {
            Fotocopiadora.asignar(i);
        }
    }
    ```

    f) Modificar la solución (e) para el caso en que sean 10 fotocopiadoras. El empleado le indica a la persona cuál fotocopiadora usar y cuándo hacerlo.
    ```
    Monitor Fotocopiadora{
        cola espera;
        cola impresoras = {...}
        cond fin;
        cond persona;
        cond empleado;
        int esperando = 0;

        Procedure asignar(idP: in int){
            if(esperando == 0){
                wait(empleado);
            }else{
                esperando--;
                signal(persona[idP]);
                wait(fin);
            }
        }

        Procedure usar(){
            signal(empleado);
            esperando++;
            wait(persona);
        }

        Procedure dejar(){
            signal(fin);
        }
    }

    Process Persona[id = 0 to N-1]{
        Documento d;
        Fotocopiadora fotocopiadoraAsignada;
        Fotocopiadora.usar(fotocopiadoraAsignada);
        Fotocopiar(d);
        Fotocopiadora.dejar();
    }

    Process Empleado{
        int i;
        for i = 0 to N-1 {
            Fotocopiadora.asignar(i);
        }
    }
    ```

4. Existen N vehículos que deben pasar por un puente de acuerdo con el orden de llegada. Considere que el puente no soporta más de 50000kg y que cada vehículo cuenta con su propio peso (ningún vehículo supera el peso soportado por el puente).
    ```
    int peso_soportado = 50000;
    Monitor Puente{
        int peso_total = 0;
        cola autos;          // cola FIFO de autos esperando
        cond[N] condAutos;
        int idAux;

        Procedure entrarPuente(peso, idA: in int) {
            // me encolo para respetar orden
            autos.push(idA);

            // espero mi turno y que haya capacidad
            while (peso_total + peso > 50000) {
                wait(condAutos[idA]);
            }

            // entro efectivamente
            peso_total += peso;
            autos.pop();
        }

        Procedure salirPuente(peso: in int) {
            peso_total -= peso;
            idAux = autos.pop();

            // despierto a todos para que reevalúen turno y peso
            signal(condAutos[idAux]);
        }
    }

    Process Auto [a:1..N]{
        int peso = ...;
        Puente.entrarPuente(peso);
        “el auto cruza el puente”
        Puente.salirPuente(peso);
    }
    ```