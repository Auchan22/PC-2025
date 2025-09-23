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

5. En un corralón de materiales se deben atender a N clientes de acuerdo con el orden de llegada. Cuando un cliente es llamado para ser atendido, entrega una lista con los productos que comprará, y espera a que alguno de los empleados le entregue el comprobante de la compra realizada.
    a) Resuelva considerando que el corralón tiene un único empleado.
    ```
    Process Cliente[id: 0..N-1]{
        text lista, comprobante;
        Corralon.llegada();
        Compra.atencion(lista, comprobante);
    }
    Process Empleado{
        text lista;
        int j;
        for j = 0 to N-1 {
            Corralon.proximo();
            Compra.esperarDatos(lista);
            res = //Generar comprobante;
            Compra.entregarComprobante(res);
        }
    }
    Monitor Corralon {
        int esperando = 0;
        bool libre = true;
        cond esperaCliente;

        Procedure llegada(){
            if(not libre){
                esperando++;
                wait(esperaCliente);
            }
            else{
                libre = false;
            }
        }

        Procedure proximo(){
            if(esperando > 0){
                esperando--;
                signal(esperaCliente);
            }else{
                libre = true;
            }
        }
    }
    Monitor Compra{
        cond vcCliente, vcEmpleado;
        text datos, resultados;
        bool listo = false;

        Procedure atencion(D: in text; R: out text){
            datos = D;
            listo = true;
            signal(vcEmpleado);
            wait(vcCliente);
            R = resultados;
            signal(vcEmpleado);
        }

        Procedure esperarDatos(D: out text){
            wait(vcEmpleado);
            D = datos;
        }

        Procedure entregarComprobante(D: in text){
            resultados = D;
            signal(vcCliente);
            wait(vcEmpleado);
        }
    }
    ```
    b) Resuelva considerando que el corralón tiene E empleados (E > 1). Los empleados no
    deben terminar su ejecución.
    ```
    Process Cliente[id: 0..N-1]{
        text lista, comprobante;
        int idE;
        Corralon.llegada(idE);
        Compra[idE].atencion(lista, comprobante);
    }
    Process Empleado[id: 0..E-1]{
        text lista;
        while(true){
            Corralon.proximo(id);
            Compra[id].esperarDatos(lista);
            res = //Generar comprobante;
            Compra[id].entregarComprobante(res);
        }
    }
    Monitor Corralon {
        int esperando = 0, canttLibres = 0;
        cond esperaCliente;
        cola eLibres;

        Procedure llegada(idE: out int){
            if(cantLibres == 0){
                esperando++;
                wait(esperaCliente);
            }
            else{
                cantLibres--;
            }
            pop(eLibres, idE);
        }

        Procedure proximo(idE: in int){
            push(eLibres, idE);
            if(esperando > 0){
                esperando--;
                signal(esperaCliente);
            }else{
                cantLibres++;
            }
        }
    }
    Monitor Compra[id: 0..E-1]{
        cond vcCliente, vcEmpleado;
        text datos, resultados;
        bool listo = false;

        Procedure atencion(D: in text; R: out text){
            datos = D;
            listo = true;
            signal(vcEmpleado);
            wait(vcCliente);
            R = resultados;
            signal(vcEmpleado);
        }

        Procedure esperarDatos(D: out text){
            if(not listo) wait(vcEmpleado);
            D = datos;
        }

        Procedure entregarComprobante(D: in text){
            resultados = D;
            signal(vcCliente);
            wait(vcEmpleado);
            listo= false;
        }
    }
    ```
    c) Modifique la solución (b) considerando que los empleados deben terminar su ejecución
    cuando se hayan atendido todos los clientes.

6. Existe una comisión de 50 alumnos que deben realizar tareas de a pares, las cuales son corregidas por un JTP. Cuando los alumnos llegan, forman una fila. Una vez que están todos en fila, el JTP les asigna un número de grupo a cada uno. Para ello, suponga que existe una función AsignarNroGrupo() que retorna un número “aleatorio” del 1 al 25. Cuando un alumno ha recibido su número de grupo, comienza a realizar su tarea. Al terminarla, el alumno le avisa al JTP y espera por su nota. Cuando los dos alumnos del grupo completaron la tarea, el JTP les asigna un puntaje (el primer grupo en terminar tendrá como nota 25, el segundo 24, y así sucesivamente hasta el último que tendrá nota 1). Nota: el JTP no guarda el número de grupo que le asigna a cada alumno.
    ```
    int N = 50;
    Process Alumno[id: 0..N-1]{
        int nroGrupo, nota;
        Aula.llegada(id); //Aviso que llegue
        Aula.esperarGrupo(id, nroGrupo); //Se me asigna el nro de grupo
        // realizar tarea
        Aula.esperarNota(nroGrupo, nota);
    }
    Process JTP{
        int i, grupo;
        int puntaje = 25;
        int contador[25] = ([25], 0); //Esto para ver que cada almno de su grupo correspondiente llego

        Aula.esperarAlumnos();
        for i: 0..N-1 {
            grupo = asignarNroGrupo();
            Aula.asignarGrupo(grupo);
        }

        for i: 0..50{
            Aula.corregirTarea(grupo);
            //Corrigiendo tarea...
            contador[grupo]++;
            if(contador[grupo] == 2){
                Aula.asignarNota(puntaje, grupo);
                puntaje--;
            }
        }


    }
    Monitor Aula{
        int cant = 0;
        cond esperaAlumno[N], esperaNota[25], inicio, corregir;
        cola fila, filaNotas;
        int nroAsig[N] = ([N] = -1), nota[25];

        Procedure llegada(idA: in int){
            cant++;
            push(fila, idA);
            if(cant == N) signal(inicio);
        }

        Procedure esperarAlumnos(){
            if(cant < N) wait(inicio);
        }

        Procedure asignarGrupo(nroGrupo: int int){
            int idAux;
            pop(fila, idAux)
            nroAsig[idAux] = nroGrupo;
            singal(esperaAlumno[idAux]);
        }

        //El alumno espera que se le asigne un nroGrupo para poder empezar la tarea
        Procedure esperarGrupo(idA: in int; nro: out int) {
            wait(esperaAlumno[idA]);
            nro = nroAsig[idA];
        }

        Procedure esperarNota(nroGrupo: in int; nota: out int){
            push(filaNotas, nroGrupo);
            signal(corregir);
            wait(esperaNota[grupo]);
            nota = nota[grupo];
        }

        Procedure corregirTarea(grupo: out int){
            if(empty(filaNotas)){
                wait(corregir)
            }
            pop(filaNotas, grupo);
        }

        Procedura asignarNota(nota: in int; grupo: in int){
            nota[grupo] = nota;
            signal_all(esperaNota[grupo]);
        }

    }
    ```

7. Se debe simular una maratón con C corredores donde en la llegada hay UNA máquina
expendedoras de agua con capacidad para 20 botellas. Además, existe un repositor encargado de reponer las botellas de la máquina. Cuando los C corredores han llegado al inicio comienza la carrera. Cuando un corredor termina la carrera se dirigen a la máquina expendedora, espera su turno (respetando el orden de llegada), saca una botella y se retira. Si encuentra la máquina sin botellas, le avisa al repositor para que cargue nuevamente la máquina con 20 botellas; espera a que se haga la recarga; saca una botella y se retira. Nota: mientras se reponen las botellas se debe permitir que otros corredores se encolen. 
    ```
    Process Corredor[id: 0..N-1]{
        Carrera.llegada();
        // Corre carrera
        //Finaliza la carrera
        Carrera.accederMaquina();
        Maquina.usar();
        Carrera.dejarMaquina();
    }
    Procesos Repositor{
        while(true){
            Maquina.reponerBottellas();
        }
    }
    Monitor Carrera{
        bool libre = true;
        int cantCorredores = 0, esperando = 0;
        cond inicioCarrera, cond colaMaquina;
    
        Procedure llegada(){
            cantCorredores++;
            if(cantCorredores == N) signal_all(inicioCarrera);
            else wait(inicioCarrera);
        }

        Procedure accederMaquina(){
            if(not libre){
                esperando++;
                wait(colaMaquina);
            }else{
                libre = false;
            }
        }

        Procedure dejarMaquina(){
            if(esperando > 0){
                esperando--;
                signal(colaMaquina);
            }else{
                libre = true;;
            }
        }
    }
    Monitor Maquina {
        int cantBotellas = 20;
        cond esperarBotellas, repositor;

        Procedure usar(){
            if(cantBotellas == 0){
                signal(repositor);
                wait(esperarBotellas);
            }
            cantBotellas--;
        }

        Procedure reponerBotellas(){
            if(cantBotella > 0) wait(repositor);
            cantBotellas = 20;
            signal(esperarBotellas);
        }
    }
    ```

8. En un entrenamiento de fútbol hay 20 jugadores que forman 4 equipos (cada jugador conoce el equipo al cual pertenece llamando a la función DarEquipo()). Cuando un equipo está listo (han llegado los 5 jugadores que lo componen), debe enfrentarse a otro equipo que también esté listo (los dos primeros equipos en juntarse juegan en la cancha 1, y los otros dos equipos juegan en la cancha 2). Una vez que el equipo conoce la cancha en la que juega, sus jugadores se dirigen a ella. Cuando los 10 jugadores del partido llegaron a la cancha comienza el partido, juegan durante 50 minutos, y al terminar todos los jugadores del partido se retiran (no es necesario que se esperen para salir).
    ```
    Process Jugador[id: 0..19]{
        int equipo = DarEquipo();
        int numeroC; //Numero cancha

        Equipo[equipo].llegada(numeroC);
        Cancha[numeroC].llegada();
    }
    Process Partido[id: 0..1]{
        Cancha[id].iniciar();
        delay(50minutos);
        Cancha[id].terminar();
    }
    Monitor Admin {
        int cancha = 0;

        Procedure asignarCancha(c: out int){
            if(cancha == 0){
                c =0;
                cancha++;
            }else{
                c = 1;
                cancha--;
            }
        }
    }
    Monitor Equipo[id: 0..3]{
        int cantJugadores = 0, cancha;
        cond habilitarJugadores;

        Procedure llegada(numeroC: out int){
            cantJugadores++;
            if(cantJugadores < 4) wait(habilitarJugadores);
            else {
                Admin.asignarCancha(cancha);
                signal_all(habilitarJugadores);
            }
            numeroC = cancha;
        }
        
    }
    Monitor Cancha[id: 0..1]{
        int cant = 0;
        cond espera, inicio;

        Procedure llegada(){
            cant++;
            if(cant == 10) signal(inicio);
            wait(espera);
        }

        Procedure iniciar(){
            if(cant < 10) wait(inicio);
        }

        Procedure terminar(){
            signal_all(espera);
        }
    }
    ```

9. En un examen de la secundaria hay un preceptor y una profesora que deben tomar un examen escrito a 45 alumnos. El preceptor se encarga de darle el enunciado del examen a los alumnos cundo los 45 han llegado (es el mismo enunciado para todos). La profesora se encarga de ir corrigiendo los exámenes de acuerdo con el orden en que los alumnos van entregando. Cada alumno al llegar espera a que le den el enunciado, resuelve el examen, y al terminar lo deja para que la profesora lo corrija y le envíe la nota. Nota: maximizar la concurrencia; todos los procesos deben terminar su ejecución; suponga que la profesora tiene una función corregirExamen que recibe un examen y devuelve un entero con la nota. 
    ```
    Process Alumno[id: 0..44]{
        text e;
        int nota;

        Aula.llegada(e);
        // realizando examen
        Aula.entregar(id, e, nota);

    }
    Process Preceptor{
        text e = ...;
        Aula.darEnunciado(e);
    }
    Process Profesora{
        int i = 0, nota, idA;
        text e;

        for i: 0..44 {
            Aula.recibirExamen(idA, e);
            nota = corregirExamen(e);
            Aula.notificarCorregido(idA, nota);
        }
    }
    Monitor Aula{
        int cant = 0, notas[45] = 0;
        cond inicioPreceptor, inicioProfesora, inicioAlumno, inicio;
        text enunciado;
        cola alumnos;

        Procedure llegada(e: out text){
            cant++;
            if(cant == 45) signal(inicioPreceptor);
            wait(inicio);
            e = enunciado;
        }

        Procedure darEnunciado(e: in text){
            if(cant < 45) wait(inicioPreceptor);
            enunciado = e;
            signal_all(inicio);
        }

        Procedure entregar(idA: in int; e: in text; nota: out int){
            push(alumnos, (idA, e));
            signal(inicioProfesora);
            wait(inicioAlumno);
            nota = notas[idA];
        }

        Procedure.recibirExamen(idA: out int; e: out text){
            if(empty(alumnos)) wait(inicioProfesora);
            pop(alumnos, (idA, e));
        }

        Procedure notificarCorregido(idA: in int; nota: in int){
            notas[idA] = nota;
            signal(inicioAlumno);
        }
    }
    ```